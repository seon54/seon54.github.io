---
layout: post
comments: true
title: NHN FORWARD 2019 후기
category: 후기
shortinfo: NHN FORWARD에서 들은 세션 정리
tags:
- 행사
---



2018년에 처음 기술 컨퍼런스를 개최하고 올해도 두번째를 이어가고 있는 NHN FORWARD에 참가하게 되었다. 듣고 싶은 세션들이 있었는데 추첨 메일을 받았을 때 정말 기뻤다. 대부분의 큰 컨퍼런스는 행사가 진행된 삼성역에서 가까운 코엑스에서 많이 열리곤 하는데 파르나스 호텔에서 진행하면서 식사와 다과도 호텔에서 제공해주는 점이 매우 마음에 들었다. 식사는 800명 한정으로 제공해주는 거였는데 다행히 일찍 가서 식사권을 받을 수 있었다.:smile: 오늘 참가하면서 내년의 NHN FORWARD도 기대하게 되었다. 

대부분 들을 세션을 정해놨는데 알고리즘에 대한 발표인 CPU full load 개선기는 내가 이해할 수 있을까라는 걱정에 조금 고민하였지만 정말 재밌게 들은 세션 중 하나였다. 마지막 세션에는 피곤하여 듣고 싶던 세션을 포기하고 일찍 집에 왔다. 나중에 발표 자료가 올라오면 찾아볼 예정이다.

| 발표명                                                       | 발표자         |
| ------------------------------------------------------------ | -------------- |
| '깃'깔나는 Git 워크플로 알아보기                             | 신승엽/NHN Edu |
| PAYCO 쇼핑 마이크로서비스 아키텍처(MSA) 전환기               | 이한진/NHN     |
| HTTP API 설계, 후회, 고민                                    | 이경환/NHN     |
| Spring JPA의 사실과 오해                                     | 신동민/NHN     |
| 쿠폰듀스X101: 가장 좋은 쿠폰을 픽하려다 만난 CPU full load 개선기 | 황도영/NHN     |



## '깃'깔나는 Git 워크플로 알아보기(트랙 1)

### 주요 git 워크 플로우 살펴보기

##### Git Flow
- A successful git branching model(Vincent Driessen 2010.1.5)
- 본인의 프로젝트에 성공적으로 사용했다며 소개
- Main branch
  - master branch: 배포된 코드를 모아두며 tag 사용
  - develop branch: 다음에 배포할 코드 모아두고 후에 master 브랜치에 merge
- Supporting branch
  - 병렬 업무 가능
  - 필요할 때 생성, 역할을 완료하면 삭제
  - feature branch: 하나의 기능을 개발하기 위한 브랜치. develop에서 생성 후 완료하면 다시 merge. fast forward 하지 않도록 주의
  - release branch: 소프트웨어 배포를 하기 위해 준비하는 브랜치. 배포 전 버그 등 수정. 배포 준비가 완료되면 master와 develop에 merge
  - hotfix branch: 배포 버전에 문제가 생길 경우 사용. master에서 생성. 완료되면 master와 develop에 merge. 

##### GitHub Flow
- GitHub Flow(scott chacon)
- Deploying at GitHub(jake douglas)
- git flow는 대부분의 케이스에서 너무 복잡
- 워크 플로우를 이해하기 쉽게 되어 실수가 없어지고 더이상 헤매지 않게 됌
- GitHub 의 개발 철학도 함께 설명
- master: 항상 stable 해야함. 
- topic branch
  - feature branch와 동일하며 기능 개발
  - 브랜치 이름을 기능을 설명하는 명확한 이름으로 짓도록 함
  - 커밋은 업무가 완료되지 않았어도 꾸준히 push 하도록 하여 구성원 모두를 끊임없이 커뮤니케이션 할 수 있도록 함. 브랜치 모델을 보면 업무를 파악할 수 있음
  - 기능을 구현하면서 pull request 개설
  - 코드 리뷰, 논의 진행
  - 개발이 완료되면 배포에 사용하도록 함(bot 이용)
    - CI 빌드 통과
    - lock 가능
    - master 브랜치 최신 커밋 존재
    - 과정 거친 후 배포 시작
  - 배포 후 예외, 오류 확인
  - 배포 후 이상 없으면 merge. pull request 이용

##### GitLab Flow
- GitLab Flow(sytse sijbrandij)
- Git Flow는 너무 복잡
- GitHub Flow는 너무 간단
- GitLab Flow는 여러가지 케이스에서의 모범 사례 소개
- 지속적 배포가 어려울 때
  - master
  - production
    - 배포할 코드
    - 자동 배포하도록 설정하면 정확한 시각 파악 가능
    - tag 생성하면 배포 시간 파악 용이
- 환경별 배포가 필요할 때
  - 각 환경에서 테스트를 통과한 경우만 다음 단계로 merge 가능
  - master
    - staging에 자동 배포
    - pre-production에 merge
  - pre-production
    - production에 merge
  - production
- release software일 때
  - master
    - stable version 브랜치 생성, tag로 버전 표기
    - bug-fix 브랜치 생성 후 merge
- 개발 단계에서 지킬 규칙
  - 개발이 완료되지 않더라도 중간 결과를 팀 내에 공유
  - 커밋을 자주하고 푸시도 자주하라
  - merge 전에 테스트

### 우리는 이렇게 해요!

##### 브랜치 전략
- 상황
  - 단기간의 배포 일정
  - 장기간의 배포 일정
    - 코드 네임 부여해서 관리
  - 같은 시간에 여러 배포 일정의 작업 필요
- Git Flow 기반으로 develop 브랜치를 단기간, 장기간 브랜치로 나누어서 진행
- 각 개발 일정 브랜치에서 각 feature 브랜치 생성
- feature 브랜치는 pull request 생성하고 리뷰 후 merge
- 개발 일정 브랜치가 완료되면 QA진행 
- QA완료 후, 개발 일정 브랜치를 master와 develop에 각각 merge 하고 master에는 tag 생성
- 나머지 개발일정 브랜치를 rebase 해줌. merge 커밋 유지
- 각 feature 브랜치도 rebase 진행
- hotfix가 필요할 때는 gitflow의 절차 따름. 완료 후 develop와 master에 각각 merge. pull request 상에서 코드 리뷰와 테스트만 진행. Git Flow 도구를 이용하여 merge
- hotfix merge 후, 각 개발일정과 feature 브랜치 rebase 

##### 개발 플로우(Github 기반)
1. 업무 생성
2. 개발 진행
3. 개발 완료 후 pull request 생성
   - 몇 가지 조건을 통과해야만 merge 할 수 있도록 설정
     - 코드 리뷰   
     - 테스트
4. 장애 발생 시 처리
5. jenkins를 이용한 자동 테스트   
   - GitHub pull request builder 플러그인
6. Sonar Qube 이용한 정적 분석
   - 버그, 코드 취약점 등을 탐지하는 자동 리뷰 툴   
   - 사용하지 않는 변수나 import 등 탐지
7. pull request merge
8. GitHub PR + jenkins + Sonar Qube
   - 기계적인 테스트, 코드 리뷰 자동가능
   - 사람은 좀 더 비즈니스 로직에 집중하여 리뷰 가능

## PAYCO 쇼핑 마이크로서비스 아키텍처(MSA) 전환기(트랙 5)

### PAYCO shopping?
##### 기본 구조
1. 파트너 쇼핑몰로 상품 수집(색인 서버)
2. 검색 서버를 통해 페이코쇼핑 웹 서버로 상품 질의
3. payco id 인증 후 파트너 쇼핑몰 이용
4. payco 결제
5. 주문 수집 서버에서 주문

##### 문제 상황
3. NCP(NHN Commerce Platform)
   - SaaS 지향   
   - admin, FE 제공   
   - 클라우드 기반의 경우, 오토 스케일 안정적으로 제공   
   - 프로젝트 모듈(Java 8, Spring, Gradle 기반)
4. 추가 기능   
   - 추가되면서 common module이 커짐
5. 빌드 시간 증가
6. 생산성 감소
7. 서비스 분리 결정
   - 새로은 REPO 생성 후 복사 붙이기   
   - 배포시간? 섞여있는 소스? 시스템 업그레이드?   
   - MSA 결정   
     - 모듈 간 낮은 커플링   
     - 독립적 배포
     - 아키텍처의 유연함   
   - 새로운 언어 선택: 코틀린

### MSA 시작
- 현재 상황 파악 및 계획
  -  범위, 도입 방법
- 기술 스택 조사
  - 트렌드, 맞는 기술 스택
- 현재 사용하고 있는 기능의 전수 조사
  - 기능 기준 RESTful API 목록 추출
  - 도메인 구별
  - 결과적으로 4개의 MICROSERVICE로 분할
- API Gateway
  - Netflix ZUUL
- Service Discovery
  - Netflix EUREKA
- Spring Cloud Netflix
- Spring Cloud gateway
  - Netflix ZUUL에서 변경
- apache zookeeper
  - Netflix EUREKA에서 변경
- Spring Cloud config server
  - cloud configuration
  - github에 자동 반영
  - refresh API call
- containerize
  - docker
    - nexus repository manager 3 이용
    - docker internal registry(private) 사용 가능
    - docker proxy registry 사용 가능
  - jenkins
  - ansible
- monitoring
  - nSight
  - Scouter - Zipkin
  - Grafana
  - Scouter - xlog
- 문서화
  - swagger based documentation
  - swagger annotation
    - 장점
      - api 문서의 현행화가 쉬움
      - 이해하기 쉽고 개발도 쉬움
      - 주석을 대체할 수 있음
    - 단점
      - 개발 로직과 상관없는 코드
      - spec file만 별도로 생성할 수 없음
      - 분산 환경에서의 문서 통합이 어려움
      - functional endpoint 방식에서 api 문서화 불가능
  - 보완 대책
    - Spring REST docs 
    - OpenAPI 3.0 
    -  swagger UI

## HTTP API 설계, 후회, 고민(트랙 1)

### RESTful API
- 당시에 따른 규칙
  - 슬래스 구분자(/)는 계층 관계를 나타내는 데  사용
  - 도큐먼트 이름으로는 단수 명사를 사용
  - 컬렉션 이름으로는 복수 명사
  - 컨트롤러 이름은 동사나 동사구
  - crud 기능을 나타내는 것은 uri에 사용하지 않음
  - put 메서드는 리소스를 삽입하거나 저장된 리소스를 갱신하는 데 사용  
- 리소스 계층도
  - 리소스 계층도에 맞게 url 설계
- file
  - content negotiation header 사용  
- 메일 읽음 표시
  - preview라는 query string보냄(표준은 아님)
  - 표준 방식은 put으로 보내기
- PUT vs. PATCH
  - PUT을 PARTIAL UPDATE 가능하게 정의하여 사용 중
    - NULL이 의미를 가지는 필드 업데이트 등에 어려움이 있음
  - PATCH를 잘 살펴보지 않은 것을 후회중
    - PARTIAL UPDATE
    - RFC 6902
    - RFC 7386 
- 미리 잘 정해두면 좋은 것
  - 리소스에 어울리는 단어들 - 비슷한 뜻을 가진 단어들이 혼용되는 것을 방지
  - flag성 이름 규칙 정하기
  - 날짜, 시간 필드명
  - 날짜 시간값 포맷
  - query parameter 이름도 가능한 정확한 표현으로 짓는 것이 좋음

## Spring JPA의 사실과 오해  (트랙 1)

### Entity 매핑, 연관관계 매핑

- Entity 매핑
  - Entity : JPA 를 이용해서 데이터베이스 테이블과 매핑할 클래스
  - Entity 매핑: Entity 클래스에 데이터베이스 테이블과 컬럼, 기본 키, 외래 키 등을 설정하는 것
- 연관관계 매핑
  - Entity들은 대부분의 경우 다른 Entity들과 연관관계를 가짐
  - 데이터베이스 테이블은 외래키로 join을 이용해서 관계 테이블 참조
  - 다중성
    - @OneToOne
    - @OneToMany
    - @ManyToOne
    - @ManyToMany
  - 방향성
    - 단방향
    - 양방향
  - 단방향 vs 양방향
    - 사실상 단방향 매핑만으로 연관관계 매핑은 이미 완료
    - 단방향 매핑에 비해 양방향 매핑은 복잡하고 객체에서 양쪽 방향 모두 관리해야 함
    - 양방향 매핑은 단방향 매핑에 비해 반대 방향으로의 객체 그래프 탐색 기능이 추가된 것 뿐
    - 대개의 경우 단방향으로도 충분
  - 영속성 전이
    - Entity의 영속성상태 변화를 연관된 Entity에도 함께 적용하는 것
    - 연관관계의 다중성 지정시 cascade 속성으로 설정
    - PERSIST, REMOVE, DETACH, REFRESH, MERGE, ALL
  - 오해 - 양방향 매핑보다 단방향 매핑이 좋다
    - 일대다 단방향 연관관계 매핑에서 영속성 전이를 통한 INSERT 시
      - 일대다 관계의 외래키 지정을 위해 추가적인 UPDATE 쿼리가 발생하는 문제
      - 이 경우 오히려 일대다 양방향 연관관계로 변경하면 추가적 UPDATE 쿼리가 없어짐
  - FETCH 전략
    - FetchType.EAGER
    - FetchType.LAZY
  - 기본 FETCH 전략
    - *-ToOne: FetchType.EAGER
    - *-ToMany: FetchType.LAZY
  - N + 1 문제
    - ENTITY에 대해 하나의 쿼리로 N개의 레코드를 가져올 때 연관관계 Entity  를 가져오기 위해 쿼리를 N번 추가적으로 수행하는 문제
    - 해결방법
      - Fetch Join
      - Entity Graph
  - Spring Data JPA Repository
    - Spring Data project에서 제공
  - 오해 - JPA Repository 메서드와 JOIN
    - JPA는 데이터베이스 테이블 간의 관계를 Entity  클래스의 속성으로 모델링
    - JPA Repository 메서드는 언더스코어(_)를 통해 내부 속서앖에 접근 가능
  - JPA Repository 메서드에서도 다양한 DTO Projection 지원

## 쿠폰듀스X101: 가장 좋은 쿠폰을 픽하려다 만난 CPU full load 개선기(트랙 2)

### 무엇이 문제일까?
##### Heap graph
- 단순 트래픽 문제인지 알 수 없음
##### CPU graph
- 가끔 cpu가 매우 바쁨
- 50% 또는 100% 사용
##### Thread dump
- 무언가가 반복 호출됨
### 왜 느릴까?
1. NCP는 다양한 쿠폰 기능 제공
2. 여러 조건과 상황이 발생
3. 상품-쿠폰 매칭 이슈   
   - 탐욕 알고리즘 적용이 어려움
4. 다양한 쿠폰들로 발생되는 이슈
   - 각각 다른 종류의 쿠폰은 변치않은 판매 정가에 의존함   
   - 각각의 쿠폰은 서로에게 영향을 미치지 않으며 독립적인 관계   
   - 상품 할인가에 의존하는 쿠폰은 다른 쿠폰에게 영향을 미치며 의존적인 관계   
   - DFS - 사용자 입력에 너무나 큰 성능 편차를 보임   
   - 상품 개수가 늘어나면서 연산 횟수가 폭발적으로 증가
### 해결해보자!
##### 아이디어 1 - 쿠폰들 의존관계 끊기
1. 쿠폰들이 서로 의존적이라 문제가 너무 복잡함
2. 가장 좋은 것을 찾는 것이 아닌 적당히 좋은 것을 찾는 문제가 됌
3. O(M^P)로 개선 (M 쿠폰 수, P 제품 수)
##### 아이디어 2 - M^p 알고리즘 개선
1. 할당 문제
2. 헝가리안 알고리즘 O(K^3)사용
   1. K x K 정방 행렬로 만들어준다.
   2. 행렬의 각 행과 열에서 최소값을 각각 뺀다
   3. 최소 라인으로 0의 성분의 셀을 지운다. 이 라인이 k개 라면 답이 존재
   4. 남은 수 중 0이 아닌 가장 작은 수로 각각 뺀다
   5. 음수 발생 시, 0 이상의 수가 되도록 다시 더한다
   6. 서로 겹치지 않는 0의 위치가 해가 된다.
### 개선 후 결과
- 100개의 제품과 쿠폰으로 적용한 결과 짧은 시간 내에 완료

