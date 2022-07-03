#RDD Programming Guide

## Overview

- resilient distributed dataset (RDD)
    - 클러스터 노드들에 파티셔닝되어 저장된 요소들의 집합.
    - 병렬적으로 처리 가능
    - 메모리에 유지 
    - node failure 로 부터 자동 복구 가능
    
-  shared variable
    - 다른 노드들에 있는 태스크 집합으로서 함수들을 병렬적으로 실행할 때, 각 태스크에게 함수에서 사용되는 변수들의 복사본을 전달하는 역할
    - broadcast variables: 모든 노드의 메모리에 특정 값을 캐싱하는데 사용
    - accumulators: 카운터나 sum처럼 더할 수만 있는 변수
    
