- spill: 파티션이 너무 커서 익스큐터의 메모리를 넘을 때, 넘는 분 만큼 디스크에 저장했다가 (직렬화) 다시 램으로 불러와(역직렬화) 처리하는 행위
  - 비싸고 느린 작업
- 스파크 클러스터의 메모리 구조
  - 워커노드 기준
    - overhead memory
    - on-heap memory
      - spill과 직접적인 연관이 있는 부분
      - reserved memory
        - spark system을 위해 예약된 메모리
      - user memory
        - RDD 변환을 위해 정의된 데이터 구조 저장(UDF등)
      - storage memory
        - 캐싱과 브로드캐스팅 데이터를 위한 공간
      - execution memory
      - shuffle, join, aggregation, sort를 위한 공간 
    - off-heap memory
      - OS에 의해 관리
      - 보완적인 메모리 
      - spark.conf.set(spark.memory.offHeap.enabled, true)를 설정해야 사용가능
      - off-heap에서 데이터를 읽어와 on-heap에 쓰는 과정(역직렬화)에 사용
  - 참고로 storage, execution 영역은 Unified되어 있어서 메모리가 공유되고, 서로 공간을 빌릴 수 있음.
    - Dynamic Occupancy Mechanism 라고 함. 
    
- on heap memory 사이즈 계산 예시
  - Let’s do some simple calculations on 14 GBs (14,336 MBs) of the memory executor node and see the size of On-Heap Memory in each area.
    Reserved Memory = 300 MBs
    User Memory = (14,336 MBs — 300 MBs) * spark.memory.fraction
    = (14,036 MBs) * 0.6 = 8,421 MBs = 8.2 GBs
    Storage Memory = (14,336 MBs — 300 MBs) * spark.memory.fraction * (1- spark.memory.storageFraction) = 14,036 MBs* 0.6*0.5
    = 4,210 MBs = 4.1 GBs
    Execution Memory = (14,336 MBs — 300 MBs) * spark.memory.fraction * (1- spark.memory.storageFraction) = 14,036 MBs* 0.6*0.5
    = 4,210 MBs = 4.1 GBs
  - default config
    - spark.memory.fraction = 0.6
      spark.memory.storageFraction = 0.5

- spill 발생 시기
  - 전체 데이터 크기가 execution과 storage의 크기 합보다 클 경우 발생 

[참고](https://selectfrom.dev/spark-performance-tuning-spill-7318363e18cb)