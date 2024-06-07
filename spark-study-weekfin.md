# 18 모니터링과 디버깅

## 18.1 모니터링 범위

#### 모니터링 대상
- spark application job
 디버깅을 한다면, SparkUI와 spark log를 확인해야함 
  - 개념적 수준의 정보를 제공(e.g. RDD, 쿼리 실행 계획)
- JVM
  - spark executor는 개별 jvm위에서 실행되기 때문에, jvm을 모니터링 해야하는 것은 당연함
    - jstack, jmap, jstat jconsole 등
    - SparkUI에서도 jvm에 대한 일부 정보를 확인 할 수 있으나, 저수준 디버깅을 해야한다면 결국 JVM 도구를 사용해야함
- OS
  - cpu, network, memory 등. 클러스터 수준의 모니터링에서 확인 가능
- 클러스터
  - YARN, 메소스, 스탠드 얼론 클러스터 매니저의 상태를 모니터링 해야함
    - 강글리아 또는 프로메테우스 사용

<br/>

## 18.2 모니터링 대상
- spark application 프로세스(cpu, 메모리 사용률 등)
- 프로세스 내부에서의 쿼리 실행 과정(job, task)

<br/>

### 18.2.1 driver, executor process
- driver에는 application의 상태를 갖고 있음
- spark의 경우 `$SPARK_HOME/conf/metriecs.properties`파일을 이용하여, 모니터링 솔루션으로 export 할 수 있다.
  - system resource usage
  - task execution time
  - GC

<br/>

### 18.2.2 query, job, task
- **특정 쿼리에서 무슨 일이 중요한지 알아야함**
- spark는 query, job stage, task 개념을 갖고 있음
  - query
  - job: RDD or DataFrame 상에서, action이 호출 될 때, spark는 task set을 구성하는 것
  - task: 1task == 1partition
  - stage: 여러개의 task를 구성하며, 이들은 병렬로 실행된다. stage는 shuffle 연산으로 구분한다
    - shuffle 연산을 만나면 partition을 재분배해야하기 때문(1task=1partiton)
    - 병렬적으로, 독립적으로 stage 실행
    - 장애 허용
    - locality

<br/>

## 18.3 Spark log
- application log, spark 자체 log를 확인하는 것은 중요
- python의 경우 spark의 java기반 logging 라이브러리를 사용할 수 없기 때문에, logging 모듈 또는 `print`사용

<br/>

## 18.4 SparkUI
 UI를 보면 탭이 보이는데, 각 항목이 의미하는 다음과 같음
- Jobs: spark job에 대한 정보 제공
- Stages: 개별 stage의 정보 제공
- Storage: spark application에 캐싱된 정보 제공
- Environment: spark app config 정보 제공. 뿐만 아니라 실행 환경(e.g. java version, spark version 등) 정보도 제공
- Executors: executor에 대한 상세정보 제공
- SQL: SQL, DataFrame 등 구조적 API 쿼리 정보 제공

<br/>

### 18.4.2 SparkUI history server
- 정상 또는 비정상적으로 종료된 applcation의 log를 확인하기 위해 사용

<br/>

`spark.eventLog.enabled=true`

<br/>

## 18.5 디버깅 및 Spark 응급 처치

### 18.5.1 spark app이 시작되지 않는 경우

징후
- spark job 실행 x
- SparkUI가 driver node 제외 나머지 정보 표시 안함
- SparkUI가 잘못된 정보 표시

<br/>

대응
- 클러스터 또는 app에 자원 할당이 적절하게 되었는지 확인
- driver, executor 간 통신 확인
- app이 클러스터 매니저에게 매니저가 갖고 있는 유휴 자원 이상을 요청하는 경우, 대기

<br/>

### 18.5.2 spark 실행 전에, 오류 발생

징후
- 명령 실행되지 않고, 오류 메세지 출력
- SparkUI에서 job, stage, task 정보 확인 불가

<br/>

대응
- column 명, 오탈자 확인
- 오류에서 출력하는 메세지 확인
- classpath 확인

<br/>

### 18.5.3 spark app 실행 중에 오류가 발생한 경우

징후
- 하나의 spark job이 성공하지만, 다음 job은 실패하는 경우
- 쿼리의 특정 단계 실패
- 어제 정상 동작했지만, 오늘은 실패하는 경우
- 해석하기 어려운 오류 메세지

<br/>

대응
- schema, 코드 확인

<br/>

### 18.5.4 느리거나 뒤쳐지는 task
징후
- 소수의 남아있는 task가 오래 수행하는 경우
- 머신 수, 자원을 늘려도 해결되지 않음
- 특정 executor가 다른 executor에 비해, 데이터를 읽거나 쓰고 있음

> 이런 현상은 낙오자(straggler)라고 한다.

<br/>

대응
- 파티션 개수를 늘린다
- 코드 개선
- 메모리 증가
- 투기적 실행(speculative execution)
- Dataset를 꼭 써야하는가? GC 때문에..

<br/>


### 18.5.5 느린 집계 속도

징후
- 느린 groupBy
- 집계 처리 이후의 job도 느릴 때

<br/>

대응
- 집계 연산 전에 파티션 개수를 늘린다
- 메모리 증가
- 집계 후에도 느리다면 re-partition
- **필요한 데이터를 사전에 선별**
- null 대신 다른 표현을 사용하고 있는지
- 태생적으로 느린 함수(e.g. collect_list, collect_set)

<br/>

### 18.5.6 느린 조인 속도
징후
- join 처리 시, 느림
- join 전후는 정상 처리됨

<br/>

대응
- join type 변경
- join 순서 변경
- prejoin partitioning
- broadcast join

<br/>

### 18.5.7 느린 읽기와 쓰기 속도

대응
- 투기적 실행. 이 때, eventually-consistent 기반의 스토리지에서 사용하면 안됨(중복 저장)
  - 한 task의 복사본을 다른 node에서 실행 
- 네트워크 대역폭 확인
- 단일 클러스터에서 spark, hdfs 구성시, 동일한 hostname을 인식 여부 확인

<br/>

### 18.5.8 driver OOM

징후
- 메모리 사용량이 많은 때
- OOM 발생

<br/>

대응
- 대용량 dataset을 대상으로 collect 사용 지양
- braodcast join으로 사용할 데이터의 용량을 줄이자
- executor memory 값을 늘린다

<br/>

### 18.5.9 executor OOM

### 18.5.10 의도하지 않은 null 값이 있는 결과 데이터

징후
- transformation 결과에 의도치 않은 null 값 발생
- 운영 환경에서 잘 돌아가고 있었는데, 예약 작업이 더 이상 동작하지 않거나, 정확한 결과를 생성하지 않음

<br/>

대응
- 쿼리 실행 계획 확인
- 애큐뮬레이터 이용

<br/>

### 18.5.11 디스크 공간 없음 오류

대응
- 저장할 storage의 용량을 늘린다
- 파티션 재분배(특정 노드에 저장되는 데이터 용량이 많을 수 있음)
- 오래된 로그 파일 삭제

<br/>

### 직렬화 오류
UDF, RDD의 사용을 최대한 피한다
- 인클로징 객체 사용을 피한다. spark는 외부 객체도 직렬화를 하려고하기 때문
