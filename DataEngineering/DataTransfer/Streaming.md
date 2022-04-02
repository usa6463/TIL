# Streaming Data Transfer

메시지 배송(message delivery)
- 웹브라우저, 모바일 앱, 디바이스 등 다수의 클라이언트에서 계속해서 작은 데이터가 전송되는 방식
- 전송되는 데이터양에 비해 통신을 위한 오버헤드가 커서, 이를 처리하는 서버는 높은 성능을 요구

메시지 저장 방법
- NoSQL DB : 작은 데이터 쓰기에 적합
- 메시지 큐, 메시지 브로커 등의 중계 시스템
- 기록된 데이터는 일정한 간격으로 꺼내고 모아서 함께 분산 스토리지에 저장

메시지 배송 방법
- 서버에서 임시 파일에 데이터를 축적해 두고 Logstash나 Fluentd 같은 서버 상주형 로그 수집 소프트웨어를 사용하여 메시지 브로커에 전달
- MQTT(MQ Telemetry Transport) 프로토콜을 사용하여 Pub/Sub형 메시지 배송 구조를 만든다.

## 참고
- [도서]빅데이터를 지탱하는 기술