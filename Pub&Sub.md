# Pub/Sub란?
![alt text](/images/pub_sub.png)
- 메세지 지향 미들웨어(MOM) 에서 사용하는 비동기 메세징 패턴
- pub(publisher)가 topic에 메세지를 보내면 해당 topic을 구독해놓은 sub(subscriber) 모두에게 메세지가 전송되면서 데이터 교환이 이루어지는 방식
- 브로커(Broker) 또는 토픽(Topic)을 통해 통신하는 것이 핵심

### Pub/Sub의 핵심 구성 요소
- Publisher : 특정 수신자에게 보내는 게 아니라, 어떤 Topic에 메세지를 게시함.
- Topic : 메세지가 전달되는 중간 통로, 게시판 역할
- Subscriber : 관심있는 토픽을 구독함. 해당 토픽에 새로운 메세지가 올라오면 브로커로부터 메세지를 전달받음.
- Broker : 발행자와 구독자 사이에서 메세지를 필터링하고 배달해주는 중재자 역할
    - Kafka, RabbitMQ, Redis Pub/Sub  

