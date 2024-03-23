# Spark Study

## 2주차

---

페이지 : 89~119

3.2 Dataset API

Data Set API

---

[https://spark.apache.org/docs/latest/api/java/org/apache/spark/sql/Dataset.html](https://spark.apache.org/docs/latest/api/java/org/apache/spark/sql/Dataset.html)

Class Dataset<T> 

dataset은 동적 타입언어(python)도 사용하는 스파크의 타입 안정성을 지원하기 위한 API

Object를 상속 받으며 컬렉션임

Lazy Evaluation

**Spark SQL, DataFrames and Datasets Guide**

---

[https://spark.apache.org/docs/latest/sql-programming-guide.html](https://spark.apache.org/docs/latest/sql-programming-guide.html)

Spark SQL

Spark SQL은 구조화된 데이터 처리를 위한 Spark 모듈

RDD API보다 더 많은 정보를 제공하여 내부적으로 최적화를 수행

SQL 쿼리 실행 및 Hive 데이터 읽기 등의 기능을 제공

결과는 Dataset/DataFrame으로 반환

Dataset

분산 데이터 컬렉션으로 RDD의 강점과 Spark SQL의 최적화된 실행 엔진을 결합

DataFrame

명된 열로 구성된 Dataset으로 구조화된 데이터 처리에 적합함

Python과 R은 Dataset API를 지원하지 않는다.

그렇지만 Python의 동적 성격으로 인해 유사한 이점을 얻을 수 있음

DataFrame API는 Scala, Java, Python 및 R에서 사용 가능하다.

Scala API → Dataset[Row]

Java API → Dataset<Row>

**RDD vs Dataset vs DataFrame**

---

**RDD**

장점

1. 탄력성 : 내결함성을 가지고 있어 실패나 데이터 손실에 대응가능
    
    → 데이터 : 여러 복사본 / 연산 : 재계산 통해 복구 
    
2. 다양한 데이터 소스 : 정형 / 비정형(벌크 or File) 읽어 올 수 있음
3. 유연한 연산 지원

단점

1. 낮은 성능 : 
    
    변경 불가능한 데이터 구조 → 연산 수행할 때마다 새로운 RDD 생성 & 복사본 유지에도 많은 자원이 사용됨
    
2. 저수준 API : 
    
    개발자가 직접 연산을 구현해야 하는 경우도 생김 → 생산성 떨어짐
    

**Dataset**

장점 

1. Type Safety : 
    
    정적타입을 지원하여 Compile시에 타입 오류를 방지해줌
    
2. 고성능 : 
    
    RDD와 유사한 성능을 제공하면서 타입 안정성도 유지함.
    

단점

1. 직렬화 및 역직렬화 오버헤드 : 
    
    타입이 복잡해 질 수록 높은 오버헤드가 발생한다.
    

**DataFrame**

장점 

1. 고수준 추상화 : 
    
    구조적 데이터 처리를 통해 추상화를 제공
    
2. 최적화된 실행 : 
    
    옵티마이저를 통해 실행 계획을 최척화 할 수 있음
    

단점

1. 비정형 데이터 처리 : 
    
    비정형 데이터나 복잡한 데이터 구조를 처리하는데 적합하지 않다.
    

---

Scala 는 NA → 결측값을 나타내는 자료형이 있다.

Null 은 참조되지 않음을 나타내고 NA는 결측치를 나타냄

## 4주차

---

페이지 : 150 ~ 180

DataFrame 관련 공식문서 : 

[https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/dataframe.html](https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/dataframe.html)

**5.4**

---

**16 . 로우 수 제안하기**

**`DataFrame.limit`(*num)***

[https://spark.apache.org/docs/3.1.3/api/python/reference/api/pyspark.sql.DataFrame.limit.html](https://spark.apache.org/docs/3.1.3/api/python/reference/api/pyspark.sql.DataFrame.limit.html)

1. **repartition & coalesce**

![출처 : [https://dongza.tistory.com/18](https://dongza.tistory.com/18)](/spark-study-week04/Untitled.png)

출처 : [https://dongza.tistory.com/18](https://dongza.tistory.com/18)

**`repartition`**과 **`coalesce`**는 두 가지 데이터 파티션을 조정하는 메서드

데이터의 분산을 변경하여 성능을 최적화할 수 있음

**공통점**

1. **데이터 파티션 조정**: 둘 다 데이터를 새로운 파티션으로 분할하거나 합치는데 사용
2. **재분배 :** 데이터를 다시 분산
3. **최적화**: 처리 성능을 향상시키기 위해 사용됨

**차이점**

1. **repartition**:
    - **`repartition`**은 주어진 파티션 수에 맞게 데이터를 재분배한다.
    - 셔플 작업을 수행하여 비용이 더 많이 소요된다.
    - 파티션 수를 늘리거나 줄일 수 있다.
2. **coalesce**:
    - **`coalesce`**는 파티션 수를 줄이려고 한다.
    - 재분배를 수행하지만, 셔플 작업을 최소화한다.
    - 파티션 수를 줄일 때만 효율적으로 작동한다.

따라서, **`repartition`**은 파티션 수를 늘리거나 줄일 때 사용되며, 셔플 작업이 필요하다. 

반면에, **`coalesce`**는 특히 파티션 수를 줄일 때 효과적이며, 셔플 작업을 최소화하여 성능을 향상시킨다.

[https://dongza.tistory.com/18](https://dongza.tistory.com/18)

[https://stackoverflow.com/questions/31610971/spark-repartition-vs-coalesce](https://stackoverflow.com/questions/31610971/spark-repartition-vs-coalesce)

1. **드라이버로 로우 데이터 수집하기 (collect)**

전체 데이터 셋에 반복 처리를 위해 로우 단위로 모을 때 사용됨

List of rows

계획을 실행시킬 때 사용됨 → action

[https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/api/pyspark.sql.DataFrame.collect.html#](https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/api/pyspark.sql.DataFrame.collect.html#)

### Chapter 6 다양한 데이터 타입 다루기

**6.1 API는 어디서 찾을까**

---

[https://sparkbyexamples.com/](https://sparkbyexamples.com/)

**column**

[https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/column.html](https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/column.html)

**DataFrame**

[https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/dataframe.html](https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/dataframe.html)

**6.2 스파크 데이터 타입으로 변환하기**

---

데이터 타입 변환은 lit 함수를 이용한다

Creates a **`[Column](https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/api/pyspark.sql.Column.html#pyspark.sql.Column)`** of literal value.

why?

명시적인 값을 스파크에 전달해야함

프로그래밍 언어의 리터럴을 스파크가 이해할 수 있는 값으로 변환해야 한다.

**6.3 불리언 데이터 타입 다루기**

---

- 불리언(boolean)은 모든 필터링 작업의 기반이므로 데이터 분석 사용된다.
- 불리언 구문은 `and`, `or`, `true`, `false`로 구성됨
- 불리언 구문을 사용해 `true` 또는 `false`로 평가되는 논리 문법을 만든다.
- 스파크에서 동등 여부를 판별해 필터링하려면 일치를 나타내는 `===`이나 불일치를 나타내는 `=!=`를 사용해야 한다. 그렇다면 비교 대상이 객체인가????
- 따라서, `not`함수나 `equalTo` 메서드를 사용한다. → 참조 동등 비교 ( 객체에 대한 비교 )

Scala 에서 `==` : 동등성 / `===` : 참조 동등성

- Where 조건 뿐만 아니라 Boolean 칼럼을 이용하여 DataFrame을 필터링 할수 있음

[https://sparkbyexamples.com/spark/spark-filter-contains-like-rlike-examples/](https://sparkbyexamples.com/spark/spark-filter-contains-like-rlike-examples/)

[https://sparkbyexamples.com/spark/spark-isin-is-not-in-operator-example/](https://sparkbyexamples.com/spark/spark-isin-is-not-in-operator-example/)

**6.4 수치형 데이터 타입 다루기**

---

- 카운트 : count(”정수값”)
- 제곱 : a^b → `pow( a, b )`
- 반올림, 내림 : round, bround
- 피어슨 상관 계수 : `df**.**stat**.**corr("칼럼명", "칼럼명")`
- 요약 통계 : `df**.**describe()**.**show()`

- 난수 생성 : `rand(), randn()`

**6.5 문자열 데이터 타입 다루기**

---

로그파일에 **정규 표현식**을 사용해 데이터 추출, 치환, 문자열 존재 여부, **대/소문자 변환 처리**

- 대/소문자 변환
    - lower : 전체 소문자로 변경
    - upper : 전체 대문자로 변경

- 공백제거
    
    **`from** pyspark.sql.functions **import** lit, ltrim, rtrim, rpad, lpad, trim`
    
    - lpad : 왼쪽부터 자릿수 만큼 채워 넣기 → `lpad("문자열", 자릿수," ")**.**alias("lp")`
    - ltrim : 왼쪽 공백 제거
    - rpad : 오른쪽부터 자릿수 만큼 채워 넣기
    - rtrim : 오른쪽 공백 제거
    - trim : (양쪽) 공백 제거
    - `lpad` 함수나 `rpad` 함수의 문자열 길이보다 작은 숫자를 넘기면 문자열의 오른쪽부터 제거된다.

- 정규 표현식
    - 문자열의 존재 여부를 확인, 패턴확인, 교환
    - `regexp_replace` 함수를 사용해 description 컬럼의 값을 COLOR로 치환하는 예제
        
        ```python
        import org.apache.spark.sql.functions.regexp_replace
        
        val simpleColors = Seq("black", "white", "red", "green", "blue")
        val regexString = simpleColors.map(_.toUpperCase).mkString("|")
        df.select(
          regexp_replace(col("Description"), regexString, "COLOR").alias("color_clean"),
          col("Description")
        ).show(2)
        ```
        
        ```
        +--------------------+--------------------+
        |         color_clean|         Description|
        +--------------------+--------------------+
        |COLOR HANGING HEA...|WHITE HANGING HEA...|
        | COLOR METAL LANTERN| WHITE METAL LANTERN|
        +--------------------+--------------------+
        only showing top 2 rows
        
        ```
        
    - `translate` 주어진 문자를 다른 문자로 치환하는 예제
        
        ```python
        from pyspark.sql.functionsimport translate
        df.select(translate(col("Description"), "LEET", "1337"), \
                  col("Description")).show(2)
        ```
        
        ```
        +----------------------------------+--------------------+
        |translate(Description, LEFT, 1337)|         Description|
        +----------------------------------+--------------------+
        |              WHI73 HANGING H3A...|WHITE HANGING HEA...|
        |               WHI73 M37A1 1AN73RN| WHITE METAL LANTERN|
        +----------------------------------+--------------------+
        only showing top 2 rows
        
        ```
        
    
    - `contains` 값 추출 없이 값의 존재 여부만 파악할 때
        - boolean 타입으로 반환
        - python & SQL에서는 `instr(”칼럼명”, “패턴”)` 사용
        
    - `locate` 패턴에서 해당 문자의 순서 반환 → 없을 시 0 반환
        
        ```
        >>> df = spark.createDataFrame([('abcd',)], ['s',])
        >>> df.select(locate('b', df.s, 1).alias('s')).collect()
        [Row(s=2)]
        ```
        

**6.6 날짜와 타임스탬프 데이터 타입 다루기**

---

- Date : 달력 형태의 날짜
- timestamp : 날짜와 시간정

inferSchema 옵션이 활성화 된 경우

→ 스파크는 특정 날짜 포맷을 명시하지 않아도 자체적으로 식별한다.

[비활성화시 퍼포먼스가 좋아짐 (csv import 경우)]

- 스파크는 날짜와 시간을 최대한 올바른 형태로 읽기 위해 노력한다.
    - 특이한 포맷의 날짜와 시간 데이터를 어쩔 수 없이 다뤄야 한다면 각 단계별로 어떤 데이터 타입과 포맷을 유지하는지 정확히 알고 트랜스포메이션을 적용해야 한다.
    
- `current_date()` : 현재 날짜를 date 타입으로 반환
- `current_timestamp()`  : 현재의 날짜 및 시간을 timestamp 타입으로 반환
- `date_sub(date, num)` : 컬럼과 뺄 날짜 수를 인수로 전달
- **`date_add(date, num)`** : 컬럼과 더할 날짜 수를 인수로 전달
- `datediff(date_1, date_2)` **:** 두 날짜의 일 수 차이를 반환
- **`months_between(date_1, date_2)` :**  두 날짜의 개월 수를 반환
- **`to_date()` :** string to date
- `to_timestamp()` : string to timestamp

**6.7 null값 다루기**

---

- DataFrame에서 빠져있거나 비어있는 데이터를 표현할 때는 항상 null값을 이용하는 것이 좋다.
- Spark 에서 빈 문자열이나 대체 값 대신 null을 이용해야 최적화를 수행할 수 있다.
- null 값을 다루는 두 가지 방법이 있다.
    - 명시적으로 null값을 제거
    - 전역 또는 컬럼 단위로 null 값을 특정 값으로 치환

1. coalesce

인수로 지정한 여러 칼럼 중 Null이 아닌 첫번째 값을 반환
