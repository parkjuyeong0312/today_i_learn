# Spring Security

## Spring Security란?

Spring 프레임워크에서 **인증(Authentication)** 과 **인가(Authorization)** 를 담당하는 보안 하위 프레임워크다.

HTTP 요청을 가로채어 **Dispatcher Servlet보다 먼저** 처리하며, Filter Chain 기반으로 동작한다.

> 인증 : 해당 사용자가 누구인지 확인 (로그인)
> 인가 : 해당 사용자가 특정 자원에 접근할 권한이 있는지 확인

---

## 전체 구조

![Spring Security 구조](/images/springSecurity.png)

요청은 다음 순서로 처리된다.

```
HTTP 요청 → Filter Chain → Dispatcher Servlet → Controller
```

Filter Chain 내부의 핵심 흐름:

```
Filter → AuthenticationManager → AuthenticationProvider → UserDetailsService → DB
```

| 구성요소 | 역할 |
|---|---|
| `FilterChain` | 여러 Filter를 순서대로 실행 |
| `AuthenticationManager` | 어떤 Provider에게 인증을 위임할지 결정 (interface) |
| `AuthenticationProvider` | 실제 인증 로직 수행 (구현체) |
| `UserDetailsService` | DB에서 사용자 정보를 불러오는 서비스 |
| `SecurityContextHolder` | 인증된 사용자 정보를 스레드별로 보관 |

---

## 로그인 방식

### 1. 세션(Session) 방식

#### 쿠키란?

서버가 클라이언트 브라우저에 저장하는 문자열 데이터다.
클라이언트가 **직접 수정 가능**하기 때문에, 중요한 인증 정보를 쿠키에 그대로 저장하면 보안 문제가 생긴다.

#### 세션이란?

로그인 정보를 **서버(메모리 또는 DB)에 저장**하고, 쿠키에는 **Session ID만** 담아 주고받는 방식이다.

```
[클라이언트] ←→ 쿠키에 Session ID만 저장
[서버]       ←→ Session ID를 키로 사용자 정보 저장 (Session DB)
```

클라이언트가 쿠키를 수정해도 Session ID만 바뀔 뿐, 서버의 실제 데이터는 변조할 수 없다.

#### 세션 방식의 단점

- **Stateful**: 서버가 모든 로그인 사용자의 정보를 유지해야 함
- 사용자가 많을수록 서버 메모리 부하 증가
- 요청마다 Session DB 조회 발생
- 수평 확장(Scale-out) 시 세션 공유 문제 (Sticky Session 또는 Redis 필요)
- Session ID 탈취 시 사칭 가능 (HTTPS + HttpOnly 쿠키로 완화)

---

### 2. JWT 토큰 방식

세션의 Stateful 문제를 해결하기 위해 등장한 **Stateless** 방식이다.
서버는 사용자 정보를 저장하지 않고, 토큰 자체를 검증하여 인증한다.

#### JWT(JSON Web Token) 구조

JWT는 `.`으로 구분된 3개의 파트로 구성된다.

```
eyJhbGciOiJIUzI1NiJ9 . eyJ1c2VybmFtZSI6InRlc3QifQ . SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV
      Header                      Payload                         Signature
```

| 파트 | 내용 |
|---|---|
| **Header** | 토큰 타입(JWT), 해싱 알고리즘(HS256 등) |
| **Payload** | 사용자 정보(Claims). 누구나 디코딩 가능하므로 민감 정보 금지 |
| **Signature** | `Base64(Header) + Base64(Payload) + 서버 비밀키`를 해싱한 값 |

Signature 덕분에 Header나 Payload가 **1비트라도 변조되면** 검증에 실패한다.

#### JWT 방식의 단점

- 토큰이 탈취되면 만료 전까지 막을 방법이 없음 → 짧은 만료 시간 + Refresh Token으로 완화
- Payload는 암호화되지 않으므로 민감 정보 저장 불가
- 토큰 크기가 세션 ID보다 크므로 매 요청마다 전송 비용 존재

---

## Spring Security 인증 흐름

### 세션 방식 vs JWT 방식 공통 흐름 (최초 로그인)

```
1. HTTP 요청 (username + password)
2. Filter가 UsernamePasswordAuthenticationToken (미인증 임시 객체) 생성
3. AuthenticationManager → 적절한 AuthenticationProvider로 위임
4. AuthenticationProvider
   └─ UserDetailsService.loadUserByUsername() 호출
   └─ DB에서 사용자 정보 조회
   └─ 비밀번호 검증 (BCrypt 등)
5. 검증 성공 → 인증된 Authentication 객체 반환
```

---

### JWT 방식 상세 흐름

#### 최초 로그인

```
요청 (username, password)
 → UsernamePasswordAuthenticationFilter
 → AuthenticationManager → AuthenticationProvider
 → UserDetailsService → DB 조회 → 비밀번호 검증
 → 인증 성공 → JWT 토큰 생성
 → Response Header에 JWT 담아 반환
```

#### 이후 요청 (JWT 보유 시)

```
요청 (Authorization: Bearer <token>)
 → JwtAuthenticationFilter
 → Header + Payload를 서버 비밀키로 Signature 재계산 → 검증
 → 검증 성공 → Payload에서 사용자 정보 추출
 → UsernamePasswordAuthenticationToken 생성 (인증 완료 상태)
 → SecurityContextHolder에 저장
 → 요청 처리 → 응답 완료 후 SecurityContext 소멸
```

> JWT 방식에서는 Signature 검증만으로 인증이 완료되므로
> **AuthenticationManager / AuthenticationProvider / DB 조회 과정이 생략된다.**

---

### 세션 방식 상세 흐름

#### 최초 로그인

```
요청 (username, password)
 → 위와 동일하게 인증 수행
 → 인증 성공 → Session에 Authentication 객체 저장
 → 클라이언트에 Session ID 반환 (Set-Cookie)
```

#### 이후 요청 (Session 보유 시)

```
요청 (Cookie: JSESSIONID=xxx)
 → SessionManagementFilter
 → Session에서 Authentication 객체 조회
 → SecurityContextHolder에 저장
 → 요청 처리
```

---

## SecurityContext 구조

```
SecurityContextHolder
 └─ SecurityContext  (요청/스레드 단위)
     └─ Authentication
         ├─ Principal  : 사용자 식별 정보 (username 또는 UserDetails)
         ├─ Credentials: 비밀번호 (인증 후 보통 null 처리)
         └─ Authorities: 권한 목록 (ROLE_USER, ROLE_ADMIN 등)
```

| 구성요소 | 범위 |
|---|---|
| `SecurityContextHolder` | JVM 전체 (기본 전략: ThreadLocal) |
| `SecurityContext` | 요청(스레드) 단위 |
| `Authentication` | Context당 하나 |

응답이 완료되면 `SecurityContext`와 `Authentication`은 소멸한다.

---

## 정리 비교

| | 세션 방식 | JWT 방식 |
|---|---|---|
| 상태 | Stateful | Stateless |
| 서버 저장 | 필요 (Session DB) | 불필요 |
| 확장성 | 어려움 (세션 공유 문제) | 용이 |
| 토큰 무효화 | 즉시 가능 | 어려움 (만료 대기) |
| 보안 위협 | Session Hijacking | 토큰 탈취 |
| 주로 사용 | 전통적인 웹 애플리케이션 | REST API, MSA |
