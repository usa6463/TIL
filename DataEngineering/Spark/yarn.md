- yarn이란? 
  - 하둡 에코 시스템에서 리소스 관리, 작업 스케줄링, 모니터링을 담당하는 시스템
  - 하둡 2.0부터 사용 시작

- 구조
  - Resource Manager
    - 모든 응용 프로그램에 리소스를 할당해주는 역할
  - Node Manager
    - 컨테이너 담당
    - 리소스 사용량 모니터링 하여 RM에 보고 
  - Application Manager
    - RM에 리소스를 협상하고 NM과 협업하여 작업을 실행 및 모니터링