# Observerbility.md

## 구성
- log: 시간이 지남에 따라 발생한 개별 이벤트에 대한 변경 불가능한 타임스탬프 기록
- metric: 시간 간격에 따라 측정된 데이터의 숫자 표현 
- trace: 분산 시스템을 통한 종단 간 request 흐름을 인코딩하는, 인과 관계가 있는 일련의 분산 이벤트를 나타냅니다.

## 솔루션
- opentelemetry: OpenTelemetry is a collection of tools, APIs, and SDKs. Use it to instrument, generate, collect, and export telemetry data (metrics, logs, and traces) to help you analyze your software’s performance and behavior.
     - telemetry 데이터 수집 목적
  

- prometheus: Prometheus is an open-source systems monitoring and alerting toolkit originally built at SoundCloud. Prometheus collects and stores its metrics as time series data, i.e. metrics information is stored with the timestamp at which it was recorded, alongside optional key-value pairs called labels.
    - metric 데이터 적재 및 alert 목적
  

- grafana: Grafana allows you to query, visualize, alert on and understand your metrics no matter where they are stored.
  - metric 쿼리 및 시각화 목적
  