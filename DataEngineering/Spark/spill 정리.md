- spill: 파티션이 너무 커서 익스큐터의 메모리를 넘을 때, 넘는 분 만큼 디스크에 저장했다가 (직렬화) 다시 램으로 불러와(역직렬화) 처리하는 행위
  - 비싸고 느린 작업
- 스파크 클러스터의 메모리 구조
  - 워커노드 기준
    - overhead memory
      - 기본적으로 Max(executor.memory*0.10 , 384mb)
      - 메모리 오버헤드는 Java NIO 다이렉트 버퍼, 스레드 스택, 공유 네이티브 라이브러리 또는 메모리 매핑 파일에 사용됩니다
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
        - GC collection 오버헤드가 없음.
      - JVM 메모리 외의 영역에서 SPARK가 사용할 메모리 지정 
      - 보완적인 메모리 
      - spark.conf.set(spark.memory.offHeap.enabled, true)를 설정해야 사용가능
      - spark.memory.offHeap.enabled 옵션을 통해 활성화 한다면 Spark 에서는 Storage Memory, Execution Memory 를 On-heap 이외에도 Off-heap 까지 활용
      - off-heap에서 데이터를 읽어와 on-heap에 쓰는 과정(역직렬화) 필요
        - 역직렬화에 대한 오버헤드 존재 
      
  - 참고로 storage, execution 영역은 Unified되어 있어서 메모리가 공유되고, 서로 공간을 빌릴 수 있음.
    - Dynamic Occupancy Mechanism 라고 함. 
    
- on heap memory 사이즈 계산 예시
  - Let’s do some simple calculations on 14 GBs (14,336 MBs) of the memory executor node and see the size of On-Heap Memory in each area.
    Reserved Memory = 300 MBs
    User Memory = (14,336 MBs — 300 MBs) * spark.memory.fraction
    = (14,036 MBs) * 0.6 = 8,421 MBs = 8.2 GBs
    Storage Memory = (14,336 MBs — 300 MBs) * spark.memory.fraction * spark.memory.storageFraction = 14,036 MBs* 0.6*0.5
    = 4,210 MBs = 4.1 GBs
    Execution Memory = (14,336 MBs — 300 MBs) * spark.memory.fraction * (1- spark.memory.storageFraction) = 14,036 MBs* 0.6*0.5
    = 4,210 MBs = 4.1 GBs
  - default config
    - spark.memory.fraction = 0.6
      spark.memory.storageFraction = 0.5

- spill 발생 시기
  - 전체 데이터 크기가 execution과 storage의 크기 합보다 클 경우 발생
  - spark.sql.shuffle.partitions 가 작아서 파티션당 사이즈가 커질 경우
  - join이나 crossJoin 실행하는 경우
  - spark.sql.files.maxPartitionBytes을 높게 잡아서 파티션당 사이즈가 커질 경우
  - 여러 컬럼들에 대해 explode를 실행하는 경우
  - skewed 데이터의 결과
  
- spill(memory), spill(disk)
  - spill(memory) : spill될 때 메모리에서 역직렬화된 형태의 데이터 크기
  - spill(disk) : spill된 이후 디스크에서 직렬화된 형태의 데이터 크기
  - spill이 발생하면 memory에 있는 데이터를 직렬화해서 디스크에 저장하고, 처리가 완료되면 디스크에 저장되어 있던 데이터를 역직렬화해서 메모리에 다시 올린 후 처리

- spill 완화 방법
  - 결국 각 executor의 메모리에 맞는 적절한 크기의 task가 부여되어야 함. 

[참고](https://selectfrom.dev/spark-performance-tuning-spill-7318363e18cb)

[참고](https://spidyweb.tistory.com/335)