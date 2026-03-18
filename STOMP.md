# STOMP
### STOMP란?
STOMP(Simple Text Oriented Messaging Protocol)  
: WebSocket위에서 동작하는 메세징 프로토콜  
  
WebSocket은 "양방향 연결 통로"를 뚫어주기만 하는 것이고, 그 위에 어떤 형식(프로토콜)로 메세지를 주고받을지 정해주는게 STOMP이다.

### 발행/구독 (Pub/Sub)모델
- 클라이언트가 특정 토픽(채널)을 구독함.
- 서버가 그 토픽으로 메세지를 발행(publish)하면 구독자 전원이 수신



