# Google Places!
### 문제상황 1 api 요청 비용을 줄여보자.
- 구글 API를 매번 호출하면, 돈이 나간다.
- 그렇다고 모든 유저가 보낼때 독립적인 api 호출을 하게 되면 너무나 많은 요청이 필요하다.
- 따라서 "캐싱 전략"이 필요하다.
> 데이터베이스에 Place 정보를 캐시처럼 사용해보자.

### Place정보를 캐시처럼 사용해도 된다고 생각한 이유
실제로 DB 용량은 정해지지 않고, 테이블들이 알아서 남은 디스크 공간을 사용합니다.
하지만, 최악의 경우를 가정하여, places 테이블의 크기가 5GB정도까지 늘어날 수 있다고 생각을해봅시다.

그리고 하나의 데이터 row 크기를 현재 테이블 기준으로 계산해보면
| 컬럼 | 타입 | 예상 크기 |
|------|------|-----------|
| id | BIGINT | 8 bytes |
| google_place_id | VARCHAR(300) | 실제 ~27자 = 27 bytes |
| name | VARCHAR(200) | 평균 ~50자 = 50 bytes |
| address | VARCHAR(500) | 평균 ~100자 = 100 bytes |
| location | GEOGRAPHY(POINT) | 고정 = 32 bytes |
| category | VARCHAR(50) | 평균 ~10자 = 10 bytes |
| rating | DECIMAL(2,1) | 고정 = 4 bytes |
| photo_reference | VARCHAR(500) | 실제 ~100자 = 100 bytes |
| last_synced_at | TIMESTAMP | 고정 = 8 bytes |
| created_at | TIMESTAMP | 고정 = 8 bytes |
| updated_at | TIMESTAMP | 고정 = 8 bytes |

대충 355바이트 정도 됩니다.  
그리고 postgreSQL이 각 row에 붙히는 메타데이터(오버헤드)로 23byte가 붙는다고 합니다.  
그리고 인덱스 데이터 까지 합친다고 해도, 200byte이내일 것이라서,
실제 사용되는건 600~700byte로 1kb에 못미칩니다.

1kb라고 생각하면,  
5GB / 1kb = 5 * 1024 *1024 KB /KB = > 약 5,000,000  
약 500만의 데이터를 저장할 수 있다는 결론에 이르게됩니다.

1kb가 아닌 5kb라고 넉넉하게 잡더라도, 약 100만의 데이터를 저장할 수 있게 됩니다. 

현재 예상 유저 규모를 생각해봤을때, DB가 영원히 차지 않는 수준입니다. 따라서 용량 걱정없이 DB에 Place정보를 캐시처럼 써도 된다는 결론을 내려봤습니다.

---

### 문제상황 2. PlaceId 외에, 다른 정보를 저장해도 되는가?
https://developers.google.com/maps/documentation/places/web-service/policies?hl=ko   
placeId는 영구저장가능, 캐싱제한 면제

https://cloud.google.com/maps-platform/terms/maps-service-terms?hl=ko   
위도 경도는 30일 캐싱정책이 있음.  
Google Places API 웹 서비스를 사용하는 애플리케이션은 Google Maps API 서비스 약관을 준수해야 합니다. 약관의 10.1.3 조항에는 약관에 명시된 제한 조건을 제외하고 어떠한 콘텐츠도 프리페치, 캐싱 또는 저장해서는 안된다고 기술되어 있습니다.

> 그렇다면, 처음에 계획한대로, 구글 api로 불러온 정보들을 그대로 가져다가 사용하는 방식은 안된다. placeID만 저장가능하기 때문이다.

> 생각 1. 애초에 유저가 많지 않을 것이라 가정한 상황이니까, 그냥 독립적인 api 형식으로 값을 받아와야될까?
