#RDD Programming Guide

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
    