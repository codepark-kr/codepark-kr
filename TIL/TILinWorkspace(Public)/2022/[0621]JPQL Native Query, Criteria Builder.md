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
* [x] 미분석 모델의 재구동을 위한 로직의 대상을 Sentinel-1 기준으로 한정
* [x] 기존 미분석 모델의 조회 로직을 max-index 기반으로 추출 후 1회 트랜젝션 로직으로 변경



### Detail
프로젝트 초기 설정 및 위성 기본 정보 - 오염 정보 간 시퀀스의 차집합을 구하는 API 신규 생성을 위해, 우선 Entity를 우선적으로 생성한다. 
대상이 되는 테이블은 기본적으로 2종, 1. 위성 기본 정보에 해당하는 객체와 2. 1의 위성 정보의 분석 결과, 탐지된 오염 정보를 가지고 있는 테이블이다. 
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

(WIP)