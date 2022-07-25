## Criteria Builder -> JPQL 로직 변경 및 리팩토링
`2022.07.08`

## Introduction
해양 오염 분석 프로젝트의 일부로, 기존 CriteriaBuilder + SpecificationExecutor를 통해 작성했던 
optional 변수 기반 위성 자료 조회 API를 JPQL 기반으로 로직 변경 및 리팩토링한다.

## Task
* [x] CriteriaBuilder + SpecificationExecutor -> JPQL 로직 변경
* [x] Entity 전반 등 객체지향 설계에 맞게 리팩토링
* [x] ST_AsText, ST_PolygonFromText 등 사용자 식별 가능한 형태로의 geometry(Polygon, 4326) WKT 변환

---

### Task #1 CriteriaBuilder + SpecificationExecutor to JPQL 로직 변경

기존 CriteriaBuilder + SpecificationExecutor를 통해 작성했던 optional 조건 기반의 위성 자료 조회 API 로직의 변경이 필요하게 되었다. 
첫 번째로 PostGIS의 **geometry(Polygon, 4326) 조회값을 WKT(Well-Known-Text)로 변환 후 반환이 요구**되었고, 두 번째로 CriteriaBuilder를
사용한 방법이 가독성이 떨어진다는 피드백이 있었기 때문이다.

geometry(Polygon, 4326)을 WKT로 변환 후 반환한다는 것이 어떤 의미이냐? 이는 postgresql(PostGIS) 테이블에 저장된 geometry 값을 
직접 확인 해 보면 알 수 있다. 테이블의 생성문을 확인 해 보면 해당 컬럼의 데이터 타입이 `geometry(Polygon, 4326)`로 정의된 것을 
볼 수 있는데, 이는 PostGIS의 geometry 타입(MULTISTRING, POLYGON, MULTIPOLYGON ...) 중 Polygon 타입으로, SRID 4326으로 값을
저장한다는 의미이다. 즉, geometry 값은 이런 식으로 저장되어 있다: 
```
0103000020E61000000100000005000000F9669B1BD3816040CAC16C020C9F41405CE674594C8D60403EB0E3BF407E42402315C61682E66040BFD7101C974B424037FDD98F14D9604019E76F42216C4140F9669B1BD3816040CAC16C020C9F4140
```

길다.  
이러한 데이터의 타입을 조회해 보면 bytea 타입이며 ( java의 String 형식으로 저장할 수 없다!), API의 실 사용자가 개발자 또는 GIS 연구원이라고 할지라도 해당 값만 보고는 정확히
어떠한 데이터인지 식별하기 어렵다. 이러한 geometry값은 `POLYGON((126.261231084735 31.5051737447582,126.296464503952 32.4941417271344,125.128858847044 32.5193348720882,125.106076243533 31.529422327836,126.261231084735 31.5051737447582))`
과 같은 문자열, 즉 Well-Known Text 형식으로 변환해주어야 식별이 가능할 것이다. 좋은 소식은, 이러한 geometry bytea to WKT 변환은 
PostGIS의 `ST_AsText` 함수를 사용하면 쉽게 변환이 가능하다.  
다음과 같이 :  

```sql
SELECT ST_AsText(geom) FROM SatelliteBaseInfo;
```
나쁜 소식은, 위와 같이 직접 함수를 태워 변환된 값을 JPA를 통해 조회 및 반환하게 하려면 일반적인 메소드 형식의 쿼리 작성(e.g. findByGeom) 및
CriteriaBuilder를 통한 자동 생성으로는 불가능하기 때문에 직접 쿼리를 작성하는 JPQL, native query 형식을 통해야 한다는 것이다. 이에 따라, 
기존에 작성했던 CriteriaBuilder + SpecificationExecutor 형식의 모든 로직을 들어내고 새로 작성하기로 했다. 

기왕 로직을 변경하는거, 최대한 우아하게 작성해 보자: 

가장 먼저, 쿼리를 작성한다.  
단, 이전 TIL에서 작성한 바와 마찬가지로 4종의 optional parameter를 받아서 조회해야 한다. 
요컨대, 센서명(sensorName : String) 변수를 사용자로부터 받는다면, 해당 센서명에 해당하는 데이터만 조회하여야 한다. 
반대로, 사용자가 센서명을 명시하지 않고 조회를 실행한다면, 모든 센서명에 해당되는 데이터를 조회하여야 한다. 
CriteriaBuilder + SpecificationExecutor 방식을 선택했던 이유가 바로 이 것인데, 직접 쿼리를 작성하여 위와 비슷한 효과를 낼 수 있는
효율적인 방식이 무엇이 있을까 리서칭 해보니 아래와 같은 흥미로운 방식을 발견했다 :  

```sql
SELECT
    sat.satellite_sensor_name, 
    sat.satellite_shoot_date, 
    ST_AsText(sat.geometry) AS geom
FROM
    satellite_base_info sat
WHERE ('POLYGON((0 0, 0 0, 0 0, 0 0, 0 0))' IS NULL OR 
        ST_AsText(sat.geometry) = 'POLYGON((0 0, 0 0, 0 0, 0 0, 0 0))');
```

여기서 `'POLYGON((0 0, 0 0, 0 0, 0 0, 0 0))'`은 사용자가 전달한 geometry 파라미터의 값이다.  
쿼리의 실행 순서(F-W-G-H-S-O)에 따라 위의 조회문을 해석해보자. 바로 이런 뜻이다 :  

> [ FROM ] 위성 기본 자료 테이블로부터  
> [ WHERE ] _**사용자가 입력한 geometry 값이 NULL이 아니라면 WKT 변환된 geometry가 사용자 입력값과 같은 데이터의**_  
> [ SELECT ] 위성 센서명, 위성영상 촬영일, WKT로 변환된 geometry 값을 조회한다.  

다른 경우도 생각해 보자.  
이번에는 사용자가 geometry 파라미터를 전달하지 않았다 :  

```sql
SELECT
    sat.satellite_sensor_name, 
    sat.satellite_shoot_date, 
    ST_AsText(sat.geometry) AS geom
FROM
    satellite_base_info sat
WHERE (null IS NULL OR 
        ST_AsText(sat.geometry) = null);
```

왜 이 방식이 흥미로운 것인지 발견했는가?  
**SQL에서의 조건(where) 절은 OR로 엮였을 때 먼저 선언된 조건문을 충족시키지 않으면 후술되는 조건문을 타지 않는다!**  
사용자가 geometry 파라미터에 값을 전달하지 않으면(null) `null IS NULL OR` 조건에 걸리게 된다. 또한 후술되는 조건과 선행 조건 사이 
`OR`가 붙어있기 때문에 선행 조건인 `null IS NULL`을 만족하게 되면 후술 조건을 타지 않고 넘어가게 되므로 모든 geometry를 조회하게 된다. 
반대로, 사용자가 geometry 파라미터에 값을 전달하면('POLYGON((0 0, 0 0, 0 0, 0 0, 0 0))') 첫 조건인 `'POLYGON((0 0, 0 0, 0 0, 0 0, 0 0))' IS NULL`을 
만족시키지 못하므로 후술 조건으로 넘어가게 된다. 즉, 사용자가 전달한 geometry와 satellite_base_info 테이블의 geometry 컬럼을 대조하게 된다.

복수의 optional 파라미터를 대입하는 경우에도 마찬가지이다.  
OR을 통한 `선행조건을 만족하면 후행조건을 타지 않는다` || `선행조건을 만족하지 않으면 후행 조건을 탄다` 를 한 쌍으로 묶어, 다음과 같이 작성하면 된다 :   

```sql
SELECT
    sat.satellite_sensor_name, 
    sat.satellite_shoot_date, 
    ST_AsText(sat.geometry) AS geom
FROM
    satellite_base_info sat
WHERE  (null IS NULL OR 
       ST_AsText(sat.geometry) = null);
    AND
       ('SENTINEL-1' IS NULL OR
       sat.satellite_sensor_name = 'SENTINEL-1');
```

이러한 방식이라면 1번의 트랜젝션으로 모든 optional parameter 조건문을 포함시킬 수 있고, PostGIS`ST_AsText(geometry)` 함수를 통한
geometry 값 대조 및 반환 또한 가능하다. 이에 따라, 기작성한 SpecificationExecutor Repository는 삭제하고, 기존 SatelliteRepository에 
새로운 쿼리를 추가시켜 주자. 

```java
    @Query(nativeQuery = true, value =
            "SELECT " +
            "    sat.satellite_sensor_name" +
            "    sat.satellite_shoot_date,\n" +
            "    ST_AsText(sat.geometry) AS geom,\n" +
            "FROM satellite_base_info sat \n" +
            "WHERE (:geometry = '' OR ST_AsText(sat.geometry) = :geometry) \n" +
                "AND (:sensorName = '' OR sat.satellite_sensor_name LIKE (:sensorName)) \n" +
                "AND sat.satellite_shoot_date > :from \n"  +
                "AND sat.satellite_shoot_date < :until")
    List<SatBaseInfo> filterBaseInfoByOptions(
            @Param("geometry") String geometry,
            @Param("sensorName") String sensorName,
            @Param("from") LocalDateTime from,
            @Param("until") LocalDateTime until
    );
```
지극히 당연하게도, PostGIS의 자체 함수를 사용해야 하므로 nativeQuery의 값은 반드시 true로 선언하여야 한다.  






### Task #2 Entity 전반 등 객체지향 설계에 맞게 리팩토링

### Task #3 사용자 식별 가능한 형태로의 geometry(Polygon, 4326) to WKT 변환


---
