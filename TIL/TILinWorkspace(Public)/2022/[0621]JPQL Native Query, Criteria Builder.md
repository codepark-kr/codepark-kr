## JPQL Native Query, Criteria Builder
`2022.06.21-2022.06.28.`

## Introduction
해양 오염 분석 프로젝트의 일부로, Optional 조건에 따른 위성 영상 기본 정보 조회 및 미분석 모델 구동을 위한 API를 설계하고, 생성한다.  

## Task
* [x] 스프링 부트 기반 경량 API 서버 환경 신규 구축
* [x] 위성 영상 기본 데이터와 오염 정보 시퀀스 간 차집합을 구한 후 복수의 건으로 반환하는 기반 API 생성
* [x] 복수의 시퀀스 결과값을 파싱한 후 파싱된 결과물을 파라미터로 전달하여 API를 호출하는 로직의 작성
* [x] 위성 기본 정보 조회 API의 신규 생성
* [x] Swagger 기초 설정 작성
* [x] ArgoCD workflow와 구현된 API의 연동
* [x] 미분석 모델의 재구동을 위한 로직의 대상을 Satellite-1 기준으로 한정
* [x] 기존 미분석 모델의 조회 로직을 max-index 기반으로 추출 후 1회 트랜젝션 로직으로 변경

---

### Task #1 위성 영상 기본 데이터와 오염 정보 시퀀스 간 차집합을 구한 후 복수의 건으로 반환하는 기반 API 생성
위성 기본 정보 - 오염 정보 간 시퀀스의 차집합을 구하는 API 신규 생성을 위해, 우선 Entity를 우선적으로 생성한다. 
대상이 되는 테이블은 기본적으로 2종, 1. 위성 기본 정보에 해당하는 객체와 2. 1의 위성 정보의 분석 결과, 즉 탐지된 오염 정보를 가지고 있는 테이블이다. 
다수의 컬럼을 가지고 있는 테이블의 경우, 1차적으로 빠른 생성을 위해 IntelliJ 내 Persistence 메뉴를 통해 자동 생성 해 준다.


![img.png](img.png)  
생성은 Persistence -> Generate Persistence Mapping -> By Database Schema 메뉴를 선택하면 된다.  
Persistence 메뉴의 사용을 위해서는, 물론 일차적으로 해당 Database를 IntelliJ상으로 매핑해야 한다.  

여기서 주의할 점은, IntelliJ Persistence 메뉴를 통한 entity 자동 생성은 컬럼명을 다음과 같이 기본 선언한다:  
```java
private String useYn;
...
@Basic
@Column(name = "use_yn")
public String getUseYn() { return useYn; }
public void setUseYn(String useYn) { this.useYn = useYn; }
```
위의 코드를 보면 `@Column` annotation의 선언 위치가 getter 메소드 위에 붙어있는 것을 확인 할 수 있는데, 
이는 native query가 아닌 JPQL 형식으로 쿼리를 작성하는 경우 해당 컬럼에 대응되는 올바른 변수명을 찾을 수 없는 문제를 야기한다.  
즉: Repository에 다음과 같은 형식으로 JPQL을 작성하는 경우 :  

```java
    @Query(value = "select b.useYn from SatelliteBaseInfo b")
    List<String> findAllUseYn();
```

`useYn` 이라는 `private String useYn`이라는 변수를 찾을 수 없다는 에러가 발생하게 되는 것이다.  
이에 따라, 해당 annotation의 위치를 적절하게 바꿔주어야 한다. 다음과 같이 :  
```java
@Column(name = "useYn") // annotation의 위치를 변수명 윗 라인으로 바꿔준다.
private String useYn;

public String getUseYn() { return useYn; }
public void setUseYn(String useYn) { this.useYn = useYn; }
```

Entity 생성, Repository 생성이 완료되면 Service, ServiceImpl을 순차적으로 작성한다. 
구현 대상인 로직은 위성 기본 정보 테이블의 시퀀스, 즉 PK Id로부터 오염 정보(분석 결과) 테이블의 시퀀스 간 차집합을 구하는 것이므로, 
우선 2회의 트랜젝션이 발생할 것을 염두하고 각 시퀀스 만을 단순 조회한 후 이들의 차집합을 stream을 통해 필터링 하기로 한다.  

**BaseInfoRepository (위성 기본 정보 Repository)**
```java
    @Query(value = "select b.satImgSeq from SatellitelBaseInfo b")
    List<Long> findAllIds();
```

**PollutionRepository (오염 정보 Repository)**
```java
    @Query(value = "select p.satImgSeq from PollutionInfo p")
    List<Long> findAllIds();
```

**SatelliteServiceImpl**
```java
@Override
public List<Long> findAllDirty() {
    List<Long> origin = baseInfoRepository.findAllIds();
    List<Long> analyzed = pollutionRepository.findAllIds();
    return origin.stream()
        .sorted()
        .filter(e -> !analyzed.contains(e))
        .collect(Collectors.toList());
}
```

---

























