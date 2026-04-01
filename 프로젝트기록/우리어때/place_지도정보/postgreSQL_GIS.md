# PostGis란? : PostgreSQL의 공간데이터 확장 기능
PostGIS는 PostgreSQL 데이터베이스에서 공간(지리) 데이터를 저장, 조회, 분석할 수 있도록 확장해주는 공간데이터 확장 모듈입니다.

### PostGIS의 주요 특징
#### PostgreSQL 기반의 공간 데이터 확장
- PostgreSQL의 익스텐션으로 설치 되고, SQL을 사용해서 공간데이터(Geography, Geometry)를 저장할 수 있음.
- PostgreSQL의 기능(트랜잭션,인덱싱,JSON)을 그대로 활용하며, 공간 연산 및 분석기능을 추가할 수 있음.

#### PostGIS의 타입
1. Geometry
- 카티시안 좌표계 공간 데이터, 2D,3D
2. Geography
- 경도,위도 기반 데이터

#### 기능
- 공간 연산 기능을 제공합니다.
- 공간 검색 속도를 높이기 위한 공간인덱스(GiST)를 지원합니다.

#### 저장되는 흐름
- Google API는 위도, 경도 값을 응답으로 줍니다.
- Java 서버는 이 값을 이용해 Point 객체를 생성합니다.
- Hibernate Spatial 라이브러리가 Point 객체를 자동으로
  GEOGRAPHY 타입으로 변환하여 DB에 저장합니다.
- GEOGRAPHY는 바이너리 형태로 저장되며, DB 내에서
  위치 정보 연산(예: 50m 이내 장소 찾기)에 활용됩니다.



https://kimhongsi.tistory.com/entry/GIS-PostGIS%EB%9E%80-PostgreSQL%EC%9D%98-%EA%B3%B5%EA%B0%84%EB%8D%B0%EC%9D%B4%ED%84%B0-%ED%99%95%EC%9E%A5-%EA%B8%B0%EB%8A%A5#google_vignette
