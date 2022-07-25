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
저장한다는 의미이다. 실제 geometry 값은 이런 식으로 저장되어 있다: 
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
SELECT ST_AsText(geometry) FROM satellite_base_info;
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
또한 ServiceImpl 또한 아래와 같은 형식으로 변경한다.

**기존**
```java
public Specification<SatBaseInfo> getBaseInfoByOptionalCondition(SatBaseInfoRequest request) {
    List<Predicate> predicates = new ArrayList<>();
    return (root, query, criteriaBuilder) -> {
        if(request.getSensorName() != null) {
            predicates.add(criteriaBuilder.equal(root.get("satImgSnsrNm"), request.getSensorName()));
        }
        if(request.getStartDate() != null && request.getEndDate() != null) {
            predicates.add(criteriaBuilder.between(root.get("satImgShtDt"),
            stringToLocalDateTime(request.getStartDate()), stringToLocalDateTime(request.getEndDate())));
        }
        if(request.getGeometry() != null){
            predicates.add(criteriaBuilder.equal(root.get("geom"), request.getGeometry()));
        }
        return criteriaBuilder.and(predicates.toArray(new Predicate[0]));
    };
}


...
```

**개선 후**
```java
@Override
public List<SatBaseInfo> getSatBaseInfo(SatBaseInfoRequest request) {
    SatBaseInfoRequest converted = setBaseInfoRequestByConverted(request);
    return baseInfoRepository.filterBaseInfoByOptions(
        converted.getGeometry(),
        converted.getSensorName(), 
        converted.getFrom(),
        converted.getUntil()
    );
}
```
훨씬 간결하고, 가독성이 높아졌다.  
Repository에 신규로 추가한 조회문을 태우기 위한 변수들의 형변환 역시 기존에는 ServiceImpl 내에서 처리하였지만, 
보다 객체지향적인 설계에 맞도록 DTO 내부에서 형변환을 완료한 후, ServiceImpl에서 그 결과를 파싱하여 바로 Repository에 
넘겨만 주면 되도록 설계를 수정하도록 하자.

---

### Task #2 Entity 전반 등 객체지향 설계에 맞게 리팩토링
[객체지향의 사실과 오해 - 역할, 책임, 협력 관점에서 본 객체지향](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=60550259) 
서적에서 반복적으로 강조하는 것 중 하나는 이 것이다 :  
**"각 객체는 독립성을 가지고 있어야 한다. 모든 객체는 각자의 역할을 충실히 수행할 책임을 가지고 있으며, 그러한 객체 간의 협력이 바로 객체 지향적 설계의 핵심이다."**
요컨대, 책에서 자주 언급하는 앨리스의 비유가 그러하다. 트럼프 카드 여왕이 주관하는 법정에서 말이다, 여왕은 목격자인 토끼에게 그가 본 것을 
진술하라는 요청(Request)을 보낸다. 이 때, 목격한 것을 진술하는 역할을 가진 토끼는 그의 책임을 다하기 위해 그가 본 것을 법정에서 응답(Response)하여야 한다. 
**단, 어떠한 수단과 어떠한 방법으로 진술할지에 대한 방식은 토끼(객체) 자신이 스스로 선택하여야 한다. 그와 협력하고 있는 또 다른 객체 - 진술을 요구한 트럼프 여왕과 
피고인인 앨리스에게는 토끼에게 진술(역할을 수행하는) 방법을 강제할 권한이 없다.** 

즉, 각 객체는 자신의 맡은 바 역할에 충실한다. 역할을 수행하는 방식에 대해서는 각 객체가 독립적으로 선택하고 수행한다. 협력관계에 있는 다른 객체들은 
그 객체가 어떠한 방식으로 역할에 충실하는지 알 수 없고, 알아선 안 된다. - 는 것이다. 이러한 면에서, 기존의 API 구조는 Entity, DTO, Repository, Service, Controller간 그들의 역할이 독립적으로 분리되고 있었다고 보기 어렵다. 
대표적으로 사용자로부터 요청받은 데이터(DTO)의 형변환 처리를 Service 단에서 수행하고 있는 경우가 바로 그러하다.

**ServiceImpl**  
```java
...
    if(request.getStartDate() != null && request.getEndDate() != null) {
    predicates.add(ifTheValue.between(fromDB.get("satImgShtDt"),stringToLocalDateTime(request.getStartDate()), stringToLocalDateTime(request.getEndDate())));
}
...

public LocalDateTime stringToLocalDateTime(String input){
        DateTimeFormatter df = DateTimeFormatter.ofPattern("yyyyMMddHHmm");
        return LocalDateTime.parse(input.replace("-", "") + "0000", df)
        .atZone(ZoneId.of(ZoneId.of("Asia/Seoul").getId())).toLocalDateTime();
    }
}
```

객체지향적 설계 관점에서, 사용자로부터 받은 데이터를 형변환하여 질의문에 대입할 수 있는 형태로 반환하는 것은 Service단이 아닌
DTO(Data Transfer Object) 단에서 처리되어야 할 문제이다. 이에 따라, 기존 Service 단에서 작성했던 형변환 메소드 등을 제거하고, 
DTO 내부에서 처리하여 Service단으로 넘겨준 후 Service단은 Repository에 파라미터를 파싱 후 바로 요청할 수 있게 리팩토링한다. 
또한, 무분별한 @Builder, @Data 등의 annotation은 제거하고, builder 패턴을 직접 생성 후 활용할 수 있도록 한다. *@Builder
annotation은 생성자를 자동 생성하는 사이드 이펙트를 유발하여, 선언이 지양된다. 

**기존 RequestDTO**
```java
@Builder
@NoArgsConstructor(access = AccessLevel.PUBLIC)
@Data
public class SatBaseInfoRequest {

    private String geometry;
    private String sensorName;
    private String startDate;
    private String endDate;

}
```

**개선된 RequestDTO**
```java
@NoArgsConstructor(access = AccessLevel.PRIVATE)
@Getter
@Setter
public class SatBaseInfoRequest {

    private String geometry;
    private String sensorName;
    private String startDate;
    private String endDate;

    private LocalDateTime from;

    private LocalDateTime until;

    public SatBaseInfoRequest(String geometry, String sensorName, LocalDateTime from, LocalDateTime until) {
        this.geometry = geometry;
        this.sensorName = sensorName;
        this.from = from;
        this.until = until;
    }

    public static Builder builder() { return new Builder(); }

    public static class Builder {
        private String geometry = "";
        private String sensorName = "";
        private LocalDateTime from = LocalDateTime.now().minusDays(6);
        private LocalDateTime until = LocalDateTime.now();

        public Builder geometry(String geometry) {
            if(geometry != null) { this.geometry = geometry; }
            return this;
        }

        public Builder sensorName(String sensorName) {
            if(sensorName != null) { this.sensorName = sensorName+"%"; }
            return this;
        }

        public Builder from(String startDate) {
            if(startDate != null) this.from = stringToLocalDateTime(startDate);
            return this;
        }

        public Builder until(String endDate) {
            if(endDate != null) this.until = stringToLocalDateTime(endDate);
            return this;
        }

        public SatBaseInfoRequest build() {
            return new SatBaseInfoRequest(
                    geometry,
                    sensorName,
                    from,
                    until
            );
        }

        public static SatBaseInfoRequest setBaseInfoRequestByConverted(SatBaseInfoRequest request) {
            return SatBaseInfoRequest.builder()
                    .geometry(request.getGeometry())
                    .sensorName(request.getSensorName())
                    .from(request.getStartDate())
                    .until(request.getEndDate())
                    .build();
        }

        public LocalDateTime stringToLocalDateTime(String input){
            DateTimeFormatter df = DateTimeFormatter.ofPattern("yyyyMMddHHmm");
            return LocalDateTime.parse(input.replace("-", "") + "0000", df)
                    .atZone(ZoneId.of(ZoneId.of("Asia/Seoul").getId())).toLocalDateTime();
        }
    }
}
```

요청 파라미터(Request)의 형변환, 기본값 대입, builder의 직접 생성, 와일드카드 추가, NoArgsConstructor의 접근제한자 설정이 완료되었다. 


첫 번째로, 사용자로부터 받아오는 날짜, 위성 영상 촬영일의 조회 시작일, 조회 종료일은 String(YYYYMMDD) 형식으로 받게되므로 이를 본래 형태인
LocalDateTime으로 변환이 필요하다. 단, 해당 API는 모든 조회 인자가 optional인 점을 고려하여, 지나치게 많은 데이터를 조회하는 위험
(geometry 컬럼에 저장된 값은 단수의 각 값당 MB 단위의 용량을 가진다)을 감수하지 않도록 default startDate, endDate를 
대입할 수 있도록 한다. 이는 위의 코드에서 `from`, `until`에 해당하는 변수로, 요청 현 시점으로부터 6일 전(LocalDateTime.now().minusDays(6)),
요청 현 시점(LocalDateTime.now()) - 즉 요청일까지 일주일 간의 데이터를 조회하도록 설정하였다. 또한, String to LocalDateTime의 형변환 
메소드를 기존 ServiceImpl에서 RequestDTO로 이동하였다. 

둘 째로, 센서명의 LIKE 조회를 위한 조건부 '%'의 append, geometry, 센서명의 기본값 설정이다.  
한 가지 유념해야 할 점은, 센서명(String), WKT-geometry(String)의 경우 사용자가 값을 전달하지 않아 null이 전달된다면
null(bytea) `IS NOT(!=)` String(requested)의 상이한 타입간 비교 실행 불가로 에러가 발생하게 된다. 이에 따라, 사용자가 이 두 값에 대한
파라미터를 추가하지 않는 경우 빈 String을 전달하도록 하고, 비교 구문 역시 `IS NOT NULL` 이 아닌 `!= ''` 으로 기본값을 대입하였다. 

현업에서 조건부 검색 및 조회 등을 위해 LIKE을 사용하는 것은 조회 속도면에서 치명적으로 느리기 때문에 지양해야 하는 패턴임은 
알고 있으나, 조회 조건이 `Sentinell1 /  Sentinell2`인 경우, Sentinell1A과 Sentinell1B /
Sentinell2A와 Sentinell2B를 모두 조회하도록 하고, 예를 들어 ZXY라거나 ABC와 같은 그 외의 경우는 해당되는 단일의 결과값만 반환해야 한다는
특수한 요구조건이 존재하였으므로 불가피하게 사용하게 되었다. - 이 또한 사용자가 센서명에 해당되는 조회 파라미터를 전달한 경우에만 '%'을 붙일 수 
있도록, 조건부 처리하였다. 

세 번째로, `@NoArgsConstructor(AccessLevel.PRIVATE)`의 접근제한자 설정은, 
[조슈아 블로크의 이펙티브 자바 3/A](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=171196410) 를 읽으며 배운 것으로, 
해당 객체의 무분별한 생성 및 호출(e.g. SatInfoRequest satInfoRequest = new SatInfoRequest();)와 같은 케이스를 방지하기 위한 
것으로, 일반적으로 PROTECTED 또는 PRIVATE으로 설정한다.

네 번째 Builder 패턴의 직접 생성은, 상기한 바와 같이 AllArgsConstructor를 자동 생성하는 사이트 이펙트를 방지하기 위해, builder 패턴을
직접 생성하고 - 사용자로부터 받아온 startDate / endDate : String을 변환한 분리된 LocalDateTime 변수 - from, until을 자동으로 
변환하여 대입할 수 있도록 설정하였다.

마지막으로, 사용자로부터 받아온 all-String 값들을 상기한 형변환, 기본값 대입, 와일드 카드 추가 등의 과정을 거쳐서 완성된 DTO 객체를 
생성할 수 있도록 별도의 메소드를 추가하였다. 이제 이 메소드를 Service 단에서 호출한 뒤, 각 변수를 Repository에 전달(협력)하는 것으로
Service단은 그의 역할에만 충실할 수 있게 된다.

---

### Task #3 사용자 식별 가능한 형태로의 geometry(Polygon, 4326) to WKT 변환


---
