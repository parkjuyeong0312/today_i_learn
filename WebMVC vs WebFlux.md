# Spring WebMVC vs WebFlux
스프링에는 WebMVC와 WebFlux, 2가지 웹 프레임워크가 있음.
WebMVC : 전통적인 멀티 쓰레드 기반의 웹 프레임워크
WebFlux : 리액티브 스택 기반의 웹 프레임 워크

## 용어 정리
- 동기
    - 호출과 응답이 동시에 이루어짐
- 비동기
    - 호출과 응답이 동시에 이루어지지 않는 것
- 블로킹
    - 함수를 call 했을 때 응답을 받기 위해 멈춰 있는 상태.
    - 함수가 종료되어야 다음줄을 실행함.
- 논블로킹
    - 함수를 call 했을 때 결과를 받기 위해 멈춰있지 않고 다음줄을 실행하는 상태

=> Spring WebMVC : 동기적  
=> WebFlux : 비동기적 

## Spring WebMVC
**동기적 블로킹** 방식
- 사용자 요청마다 스레드를 생성(스레드 풀)
- 많은 사용자가 동시요청을 보내면 요청을 처리하지 못함.(Thread Poll Hell)
- 시스템 트래픽을 설정해서 **스레드 풀 크기**를 잘 조정해야함.

: springMVC 방식은 사용자의 요청을 대량으로 받아내는데 한계가 있다. WebFlux가 이를 해결 가능

## WebFlux
**비동기적 논블로킹** 방식  
- **리액티브 프로그래밍**
- **Event loop**
- 모든 코드가 non-blocking 하게 동작해야한다.


참고 문헌
https://pearlluck.tistory.com/726#google_vignette
