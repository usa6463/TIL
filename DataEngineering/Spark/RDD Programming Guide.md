#RDD Programming Guide
- [doc](https://spark.apache.org/docs/3.3.0/rdd-programming-guide.html)

## Overview

- Overview
    - RDD
        - 클러스터 노드들에 파티셔닝되어 저장된 요소들의 집합.
        - 병렬적으로 처리 가능
        - 메모리에 유지 
        - node failure 로 부터 자동 복구 가능
    
    -  shared variable
        - 다른 노드들에 있는 태스크 집합으로서 함수들을 병렬적으로 실행할 때, 각 태스크에게 함수에서 사용되는 변수들의 복사본을 전달하는 역할
        - broadcast variables: 모든 노드의 메모리에 특정 값을 캐싱하는데 사용
        - accumulators: 카운터나 sum처럼 더할 수만 있는 변수
    
- Initializing Spark
    - SparkContext: 스파크가 클러스터를 어떻게 제어할지 얘기하는 객체
    - SparkConf: 애플리케이션의 정보를 가지는 객체. SparkContext 생성에 필요
    
- resilient distributed dataset (RDD)
    - 병렬로 실행 가능하고 fault-tolerant한 컬렉션
    - 생성방법
        - 기존에 존재하던 드라이버 프로그램의 컬렉션을 parallelizing 한다.
        - 외부 스토리지 시스템의 데이터셋을 참조(HDFS, Hbase, ...)
    

- Parallelized Collections
    - 예시
        ```scala
        val data = Array(1, 2, 3, 4, 5)
        val distData = sc.parallelize(data, 10)
        ```
    - parallelize 함수 두번째 파라미터로 파티션 갯수 지정 가능
    
- External Datasets
    - 로컬 파일 시스템, HDFS, 카산드라, HBase, aws S3 등 하둡에서 지원하는 모든 스토리지 소스에서 데이터셋 생성 가능
    - text file, sequence file 및 기타 Hadoop input format 지원
    - 예시
        ```scala
        val distFile = sc.textFile("data.txt")
        ```
      
    - 주의사항
        - 로컬 파일 시스템 경로 사용시, worker에서도 동일한 경로 파일에 접근 가능해야함. 
        - 경로에 wildcard 지원, 압축파일 지원
        ```scala
        textFile("/my/directory")
        textFile("/my/directory/*.txt")
        textFile("/my/directory/*.gz")
        ```
        - textFile 메서드는 두번째 파라미터로 파티션 수 제어 가능. 
            - 디폴트는 파일의 각 블록 당 파티션 1개(HDFS 디폴트 블록크기 128MB)
            - 블록 수 보다 더 적은 파티션 수 가질 수 없음. 블록 수 이상은 가질 수 있음
        - SparkContext.wholeTextFiles
            - 여러 개의 작은 텍스트 파일이 포함된 디렉토리를 읽고 각 파일을 (파일 이름, 내용) 쌍으로 반환
        - RDD.saveAsObjectFile
            - 직렬화된 Java 객체로 구성된 RDD 저장
            - AVRO 같은 형식에 비하면 효율적이지 않지만 모든 RDD를 저장하는 쉬운 방법 제공
        - SparkContext’s sequenceFile[K, V]
            - sequence file 대상으론 해당 함수 사용
            - key-value 형태
        - SparkContext.hadoopRDD
            - 다른 Hadoop InputFormats을 위해 사용
    
- RDD Operations
    - 2개 타입의 operation 제공
        - transformation: 기존 데이터셋으로 부터 새로 데이터셋 생성
        - action: 데이터셋에서 연산 수행 후 하나의 값을 드라이버 프로그램에 반환 
    - lazy
        - transformation은 lazy한데 그 결과를 바로 반환하는게 아니라 action이 필요할 때 연산이 발생
        - 이런 디자인은 스파크가 더 효율적으로 동작하게 함
        - 예를들어, map 결과가 reduce에서 사용되는걸 알 수 있고, 크기가 큰 map 결과를 바로 반환하는게 아니라 reduce된 결과를 반환할 수 있게 됨.
    
    - cache
        - 기본적으로 RDD는 action이 실행될 때 마다 재연산됨.
        - RDD를 메모리에 유지할 수 있는데, persist나 cache 메소드 사용하면 됨. 
        - 디스크에 유지하거나 여러 노드에 복제될 수 있게 하는 기능도 지원
    
- Passing Functions to Spark
    - SPARK에서 함수를 전달하는 추천 방법 2가지 존재
        - 익명함수
        - 싱글톤 오브젝트의 static 메소드
    - 인스턴스의 메소드를 넘기는 방법도 가능하긴 함(추천되진 않는듯?)
        - 이 방법은 객체 전체를 클러스터에 보내는 것과 동일하다고 함. 
        ```scala
        class MyClass {
            def func1(s: String): String = { ... }
            def doStuff(rdd: RDD[String]): RDD[String] = { rdd.map(func1) }
        }
        ```
        - 위와 비슷하게 필드를 참조하는 것도 객체 전체를 클러스터에 보내게 됨.
        ```scala
        def doStuff(rdd: RDD[String]): RDD[String] = {
          val field_ = this.field
          rdd.map(x => field_ + x)
        }
        ```
        - 객체 전체를 보내는 것을 피하려면 필드 값을 local variable에 복사하는 것이 좋다 

- Understanding closures
    - spark의 어려운점: variable과 method의 scope과 life cycle
    - 스파크는 RDD 연산의 과정을 여러 태스크로 나눈다.
        - 태스크는 익스큐터에서 실행된다
        - execution 이전에 스파크는 태스크의 closure를 계산한다
    - closure
        - RDD의 연산 실행을 위해 명시 되어야 하는 변수와 메소드들
        - 예를들면, rdd.foreach 내 명시된 부분
        - closure는 직렬화되어 각 익스큐터에 전송된다. 
    - closure내 변수는 익스큐터에 보내질 때 값이 카피되어 보내진다
    - 예를들면 이 코드에서 closure 외부의 변수를 참조하고 있는데 이 값은 여러 익스큐더에 복사되어 보내지므로 counter로서의 역할을 못한다
        - 이럴떈 Accumulator를 사용해야 함. 
    ```scala
    var counter = 0
    var rdd = sc.parallelize(data)
    
    // Wrong: Don't do this!!
    rdd.foreach(x => counter += x)
    
    println("Counter value: " + counter)
    ```
    - Printing elements of an RDD
        - rdd.foreach(println) or rdd.map(println): 각 익스큐터의 stdout에 출력이 찍힘
        - rdd.collect().foreach(println): driver에 모아서 찍긴하지만, 전체 RDD를 한 머신에 모으는 것이므로 OOM 발생할 수 있음
        - rdd.take(100).foreach(println): 일부만 샘플로 보는 목적이라면 take를 쓰는 방법이 좋음. 
    
- Working with Key-Value Pairs
    - 대부분의 spark 연산은 모든 타입에 대해 동작하지만, key-value pair에서만 동작하는 일부 연산 존재
    - Scala 기준
        - Tuple2 객체를 가지는 RDD 대상
        - key-value pair 연산은 PairRDDFunctions 클래스에 있음. 이 클래스는 자동으로 tuple형태 RDD를 wrapping 함
    - 예시
        - reduceByKey
        - sortByKey
    
- Transformations
    - 주의해야할 메소드만 기록
    - groupByKey([numPartitions])
      - V에 있는 값들을 iterable 클래스에 모으는 메소드
      - 모든 익스큐터에서 함수를 적용하기전에 해당 키와 관련된 모든 값을 메모리로 읽어야 함.
      - skew한 키가 있다면 일부 파티션이 많은 양의 값을 가질 수 있음.(OOM)
    - reduceByKey(func, [numPartitions])
      - V에 있는 값들을 func 내용에 따라 집계
      - 각 파티션에서 리듀스 작업을 수행하기 때문에 groupByKey보다 안정적이며 모든 값을 메모리에 유지하지 않아도 됨.
      - 최종 리듀스 과정 제외한 모든 작업은 개별 워커에서 처리하기 때문에 연산중에 셔플이 발생하지 않음.
      - 정리하면 groupByKey는 일단 특정 파티션에 모든 K의 V를 다 모으고 시작하는거고, reduceByKey는 각 파티션에서 같은 K에 대한 aggregate를 먼저 진행하고 최종적으로 K별로 reduce를 한번 더 하는것
    - aggregateByKey(zeroValue)(seqOp, combOp, [numPartitions])
      - zeroValue : seqOp의 첫번째 매개변수로 들어가는 값. U 타입
      - seqOp: K, V 타입에서 V를 U 타입으로 바꾸는 함수. 
      - combOp: U타입으로 바뀐 값들을 집계하는 함수
      - reduceByKey와의 차이점은 파티션 단위에서 실행할 함수와(seqOp) 전체 파티션(드라이버?) 단위에서 실행함 함수(combOp)를 다르게 할 수 있음.
    - combineByKey
      - aggregateByKey 보다 좀 더 세밀하게 집계를 실행하는 함수로 보임
      - value를 변환, 파티션 단위 병합, 여러 컴바이너 결과값을 병합 하는 3개의 단계에 대해 함수를 커스텀하게 전달 가능
    - foldByKey
        - 결합함수와 항등원 값을 이용해 각 키의 값을 병합
        - 항등원 값은 결과에 여러번 사용될 수 있으나 결과 변경불가 (덧셈에선 0, 곱셈에선 1)
    - cogroup(otherDataset, [numPartitions])
        - (K,V) 타입과 (K, W) 타입이 매개변수로 주어지면 (K,(Iterable<V>,Iterable<W>)) tuple을 생성함
    - coalesce(numPartitions)
        - 파티션수를 줄일 때 사용. 큰 데이터셋을 필터링 하고난 후 연산을 효율적으로 수행하는데 유용. 리셔플이 일어나지 않음. 
    - repartition(numPartitions)
        - 파티션수를 늘리거나 줄일 때 사용. 데이터를 랜덤하게 리셔플한다.
    - repartitionAndSortWithinPartitions(partitioner)
        - 리파티셔닝 하면서 각 파티션에서 레코드가 소팅되도록 한다. 리파티션하고 각 파티션에서 소팅하는 것보다 효율적. 소팅을 셔플 기계로 넣을 수 있기 때문
    
- Actions
    - reduce(func)
        - aggregation 연산 수행하는 액션        
        - 정상적으로 병렬 실행 되기 위해선 commutative(가환성:교환법칙이 성립하게 하는 성질), associative(결합법칙)한 함수를 실행해야 한다
            - 교환법칙과 결합법칙이 만족되는 경우는 실수에선 더하기, 곱하기
    - collect()
        - 데이터셋의 요소들을 배열 형태로 드라이버 프로그램에 반환.
    - foreach(func)
        - 각 요소에 대해 func 수행.
        - 사이드 이펙트를 위해 사용(Accumulator나 외부 스토리지 시스템을 업데이트하는 것과 같은)

- Shuffle operations
    - 배경지식
        - 셔플이란
            - reduceByKey 기준        
            - 하나의 태스크는 하나의 파티션에서 수행됨.
            - reduce를 수행하려면 하나의 파티션에서 동일한 키의 튜플들을 모두 알아야함. 
            - 하지만 일반적으로 데이터들은 특정 작업에 필요한 위치에 있기 위해 파티션 분할되지 않는다. 
            - 결국 spark에서 all-to-all operation을 수행해야 함. 모든 키에 대한 모든 값을 찾기위해 모든 파티션을 읽고 값을 가져와야 함.  
            - 위 과정이 셔플
        - 셔플 되고나면 파티션들의 순서는 정렬되어 있지만 파티션 내 요소들은 정렬되어 있지 않음. 그리고 각 파티션 내 값 집합은 결정적임.
        - 파티션 내 요소 정렬 방법
            - mapPartitions 와 .sorted
            - repartitionAndSortWithinPartitions 
            - sortBy
    - Performance Impact
        - 셔플은 비싼 연산: disk I/O, data serialization, network I/O
        - map 연산 결과
            - 메모리에 저장된다 (꽉 차기 전까지)
            - 타겟 파티션 기준으로 정렬되며 하나의 파일에 저장된다.
            - reduce시 관련 블록을 읽는다(파일 말하는듯?)
        - 셔플은 상당한 양의 힙 메모리 사용
            - in-memory 데이터 구조를 사용하여 전송 전후에 레코드를 구성하기 때문에
            - 데이터가 메모리 크기를 넘는다면 디스크로 보냄. 
            - 이 작업은 Disk I/O 증가시키고 가비지 컬렉션을 증가시킴. 
        - 디스크에 많은 양의 중간 파일 생성
            - Spark 1.3부터 이러한 파일은 해당 RDD가 더 이상 사용되지 않고 가비지 컬렉션 될 때 까지 유지
            - 이는 lineage 가 다시 계산되는 경우 셔플 파일 다시 만들 필요 없도록 함. 
            - 이로 인해 장기 수행되는 Spark job은 많은 양의 디스크 공간 사용할 수 있음. 
            - 임시 저장소는 spark context에서 spark.local.dir 설정 파라미터로 지정 가능

- RDD Persistence
    - persist or cache 사용시 데이터셋을 메모리에 유지 가능.
    - spark의 캐시는 내결함성을 가짐. RDD의 어떤 파티션이 lost되면 자동적으로 다시 transformation 하여 재계산된다. 
    - persist 쓸 시 각 RDD 스토리지 레벨 설정 가능
    - cache()는 default 스토리지 레벨 사용하는 축약형임.(deserialized object를 메모리에 저장)
    - MEMORY_ONLY_2, MEMORY_AND_DISK_2 : 2개의 클러스터 노드에 각 파티션을 복제한다. 뒤에 3이 붙으면 3개의 클러스터도 지원. 4부턴 없는듯
    - 셔플 연산시 스파크가 자동으로 중간 데이터를 persist 한다. 셔플 중 노드 장애시 인풋 전체에 대한 재연산 피하기 위함.
    
    - Which Storage Level to Choose?
        - MEMORY_ONLY(default): 가장 CPU 효율적. 가능한한 빠른 연산 제공 
        - MEMORY_ONLY_SER : 직렬화를 통해 메모리 용량을 덜 차지하지만 조금 느려짐.
        - 연산이 비싸거나, 많은 양의 데이터를 필터링 하기 전에는 disk에 쓰지마라. 디스크로부터 읽는 것보다 파티션 재연산이 더 빠를 수 있다.
        - 복제된 저장소 수준은 손실된 파티션 재연산보다 빠른 오류 복구를 제공함. 
    - Removing Data
        - 스파크는 캐시 사용량을 모니터링 하고, LRU 방식으로 오래된 데이터 파티션을 드롭한다.
        - 수동으로 드롭 시키고 싶다면 `RDD.unpersist()` 사용
        - `RDD.unpersist()`는 기본적으로 차단 되지 않는데, 리소스가 해제될 때 까지 차단하려면 `blocking=true` 옵션을 명시해라
    
- Shared Variables
    - 클러스터 노드들에 복사되어 사용됨
    - 2가지 종류 존재
    - Broadcast Variables
        - 각 머신에 저장된 read-only 변수 
        - spark의 action은 여러개의 stage로 나눠지는데, 셔플 연산에 의해 나눠짐.
        - 사용하기 좋은 상황        
            - 태스크가 여러 stage에 걸쳐 동일한 데이터가 필요할 때
            - 역직렬화된 형태의 데이터를 캐싱하기 원할 때
      
        - 캐싱된 브로드캐스트 제거
            - `.unpersist()`: 이후 브로드캐스트 변수가 다시 사용되면 re-broadcast 된다
            - `.destroy()`: 호출 이후 브로드캐스트 변수 사용 불가
      
    - Accumulators
        - 병렬적으로 더하기만 지원됨. 그래서 가환성 및 결합법칙 성립해야 함.
        - Spark UI를 통해 확인 가능
        - 생성 방법
            - `SparkContext.longAccumulator()`
            - `SparkContext.doubleAccumulator()`
        - 각 태스크들은 add 연산 가능하지만, 읽는 건 드라이버 프로그램에서만 가능
        - AccumulatorV2를 상속하여 커스텀 타입 생성 가능 
        - action 단계에선 Accumulators에 값 업데이트 되는게 한번만 되는게 보장
        - 반대로 transformations 단계에선 re-excuted 될 때마다 값 업데이트가 발생

- mapPartition, foreachPartition
    - mapPartition: 개별 파티션에 대해 map 연산 수행하고 결과를 반환
    - foreachPartition: 개별 파티션에 대해 순회, 결과를 반환하진 않음. 
    
- 조인
    - 내부조인
        - 2개의 RDD 필요        
        - 출력 파티션수 설정 가능 `KVcharacters.join(keyedChars, outputPartitions).count()`
        - 조인 종류
            - fullOuterJoin: 조인 조건에 맞지 않더라도 양측의 RDD에 존재하는 row가 최소한 1개는 존재하도록 함. 
            - leftOuterJoin
            - rightOuterJoin
            - cartesian
    - zip
        - 동일한 길이, 동일한 수의 파티션인 2개의 RDD 필요
        - 파티션 수 다른데 zip 사용하면 exception 발생
        