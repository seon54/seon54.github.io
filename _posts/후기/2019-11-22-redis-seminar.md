---
layout: post
comments: true
title: 우아한 테크 세미나 <우아한 Redis> 후기
category: 후기
shortinfo: 11월 우아한 테크 세미나 <우아한 Redis> 후기
tags:
- 행사
- database

---



이전에도 몇 번 신청했지만 한 번도 되지 않았던 우아한 테크 세미나에 가게 되었다. redis를 찾다보면 강대명님의 블로그는 한 번은 보게 되는데 이번에 발표를 하셔서 꼭 들어보고 싶었던 세미나여서 참가 메일을 받고 매우 기뻤다. 장소에 도착하니 우아한 형제들에서 음료와 샌드위치도 제공해주셔서 여러모로 감사한 마음이 들었다. 앞으로도 유익한 세미나 많이 준비해주시고 참가할 수 있기를! 



##### 목차
- Redis 소개
- 왜 Collection이 중요한가
- Redis 사용
- Redis Collections
- Redis 운영
- Redis Failover
- Redis 데이터 분산



## Redis 소개

- In-Memory Data Structure Store
- Open source(BSD License)
- Supported data structure
  - Strings, set, sorted-set, hashes, lists
  - Hyperloglog, bitmap, geospatial index
  - stream
- Only 1 committer(Salvatore Sanfilippo)

#### Cache
- Factorial: 이전에 계산한 값을 저장해서 사용
- 접근속도가 다름
  ```
  **CPU Cache**(disk로 갈수록 capacity 높음, core로 갈수록 속도 빠름)
  Disk
  Memory
  L3 cache
  L2 cache
  L1 cache
  Core
  ```
- 전체 요청의 80%는 20%의 사용자

##### Cache 구조 1. Look aside
1. Web server는 데이터가 존재하는지 cache 먼저 확인
2. Cache에 데이터가 있으면 cache에서 가져온다
3. Cache에 데이터가 없다면 DB에서 읽어온다.

##### Cache 구조 2. Write back
1. Web server는 모든 데이터를 cache에만 저장
2. Cache에 특정 시간 동안의 데이터 저장
3. Cache에 있는 데이터를 DB에 저장
4. DB에 저장된 데이터 삭제
5. 단점: 처음에 cache에 저장했기 때문에 리부팅 하게 될 경우(장애 발생) 데이터가 날아감

## 왜 Collection이 중요한가

- 개발의 편의성
  - 랭킹 서버 구현
  - 유저에 score 저장하고 order by로 하면 데이터가 많을 경우 되지 않을 수도 있음
  - Redis의 sorted set 이용하면 간단하게 구현 가능(replication도 가능)
- 개발의 난이도
  - 친구 리스트
  - key/value로 저장할 때 트랜잭션 과정에서 순서 문제가 생길 수 있음
  - Redis 자료구조는 atomic 하므로 race condition을 피할 수 있음(잘못 짜면 발생 가능)
- 외부 콜렉션을 잘 이용하는 것으로 개발 시간을 단축 시키고 문제를 줄여줄 수 있음
- Memcached는 제공되지 않음

## Redis 사용

- Remote data store
  - A서버, B서버, C서버에서 데이터를 공유하고 싶을 때
- 한 대에서만 필요하면 전역 변수를 쓰면 되지 않을까?
  - Redis 자체가 atomic 보장(single thread)
- 주로 많이 쓰는 곳들
  - 인증 토큰 저장(strings, hash)
  - Ranking board로 사용(sorted set)
  - 유저 API limit
  - Job queue(lists)

## Redis collections

##### Strings
- 기본 사용법 
  - SET key value / GET key
  - 예) SET token:123 abcd / GET token:123

##### List
- 기본 사용법
  - LPUSH(RPUSH) key elements
  - LPOP(RPOP) key

##### Sets
- 기본 사용법
  - SADD key member
  - SISMEMBER key member 
  - SMEMBERS key
- SMEMBERS : 모든 values를 가져오므로 데이터가 많을 경우 주의해서 사용

##### Sorted set
- Score: double 타입이므로 값이 정확하지 않을 수 있음
  - 컴퓨터에서는 실수가 표현할 수 없는 정수값 존재

##### Hahses
- 기본 사용법
  - HSET key field value
  - HGET key field

##### Collection 주의 사항
- 하나의 collection에 너무 많은 아이템을 담으면 좋지 않다
  - 1만개 이하 몇 천개 수준으로 유지하는 게 좋음
- Expire는 collection의 아이템 개별로 걸리지 않고 전체 collection에 대해서만 걸림
  - 즉 해당 1만개의 아이템을 가진 콜렉션에 expire가 걸려있다면 그 시간 후에 1만개의 아이템이 모두 삭제됨

## Redis 운영

##### 메모리 관리를 잘하자
- Redis는 in-memory data store
- Physical memory 이상을 사용하면 문제가 발생
  - swap이 있다면 swap 사용으로 해당 메모리 page 접근시마다 늦어짐
  - swap이 없다면?
- Maxmemory를 설정할 때 이를 초과할 경우 랜덤하게 데이터 삭제?
- Maxmemory를 설정하더라도 이보다 더 사용할 가능성이 크며 memory allocator에 따라 다를 수 있음
- RSS 값을 모니터링 해야함
- 많은 업체가 현재 메모리를 사용해서 swap을 쓰고 있다는 것을 모르는 경우가 있음
- 큰 메모리를 사용하는 인스턴스 하나보다는 적은 메모리를 사용하는 여러 인스턴스가 안전함
- 레디스는 메모리 파편화가 발생할 수 있음. 4.x대부터 메모리 파편화를 줄이도록 jemalloc에 힌트를 주는 기능이 들어갔으나 jemalloc 버전에 따라 다르게 동작할 수 있음

메모리가 부족할 때
- 좀 더 많은 장비로 migration
  - 메모리가 빡빡하면 migration중에 문제가 생길 수 있음
- 있는 데이터 줄이기
  - 데이터를 일정 수준에서만 사용하도록 특정 데이터 줄임

메모리를 줄이기 위한 설정
- 기본적으로 콜렉션들은 아래의 자료 구조 사용
  - Hashes → hashtable을 하나 더 사용
  - Sorted set → skiplist와 hashtable 이용
  - Set → hashtable 사용
  - 해당 자료구조들은 메모리 많이 사용
- ziplist 이용하기

ziplist 구조
- In-memory 특성 상, 적은 개수라면 선형 탐색을 하더라도 빠름
- Lists, hashes, sorted sets 등을 ziplist로 대체해서 처리하는 설정 존재


##### O(N) 관련 명령어는 주의하자
- Redis는 싱글 스레드
  - Redis가 동시에 여러 개의 명령을 처리할 수 있을까?
  - 단순한 get/set의 경우, 초당 10만 tps 이상 가능(CPU 속도 영향)
  - packet으로 하나의 command가 완성되면 processCommand에서 실제로 실행
- 한 번에 하나의 명령어 수행 가능
  - 긴 시간이 필요한 명령이 필요한 경우?
  - 망함
- 대표적인 O(N) 명령어들
  - KEYS
  - FLUSHALL, FLUSHDB
  - Delete collections(1-2초 소요)
  - Get all collections
- 대표적 실수 사례
  - key가 백만개 이상인데 확인을 위해 keys 명령을 사용하는 경우
    - 모니터링 스크립트가 일초에 한 번씩 keys 호출하는 경우도
  - 아이템이 몇 만개 든 hash, sorted set, set에서 모든 데이터 가져오는 경우
  - 예전의 spring security oauth redis TokenStore
- KEYS를 어떻게 대체할 것인가?
  - SCAN 명령을 사용하는 것으로 하나의 긴 명령을 짧은 여러 번의 명령으로 바꿀 수 있음
- Collection의 모든 아이템을 가져와야 할 때
  - Collection 일부만 가져오기(sorted set)
  - 큰 collection을 작은 여러 개의 collection으로 나눠서 저장
- Spring security oauth redis tokenstore 이슈
  - access token의 저장을 list(O(N)) 자료구조를 통해 이루어짐
  - 검색, 삭제 시 모든 item을 매번 찾아봐야 함
  - 현재는 set(O(1))을 이용해서 검색, 삭제하도록 수정됨
- O(1) 명령어와 O(N) 명령어를 구분해서 사용해야 함

##### Replication
- async replication
  - replication lag 발생할 수 있음
- 'replicaof(>=5.0.0)' or 'slaveof' 명령으로 설정 가능
  - replicaof hostname port
- DBMS로 보면 statement replication이 유사
- 설정 과정
  - secondary에 replicaof or slaveof 명령 전달
  - secondary는 primary에 sync 명령 전달
  - primary는 현재 메모리 상태를 저장하기 위해 fork
  - fork한 프로세서는 현재 메모리 정보를 disk에 dump
  - 해당 정보를 secondary에 전달
  - fork 이후의 데이터를 secondary에 계속 전달
- 주의 사항
  - 과정에서 fork 발생하므로 메모리 부족 발생 가능
  - redis-cli --rdb 명령은 현재 상태의 메모리 스냅샷을 가져오므로 같은 문제 발생시킴
  - aws나 클라우드의 redis는 좀 다르게 구현되어 있어서 좀 더 안정적(좀 더 느림)
  - 많은 대수의 레디스 서버가 replica를 두고 있ㅇ면
    - 네트워크 이슈나 사람의 작업으로 동시에 replication이 재시도 되도록 하면 문제 발생 가능
    - 예) 같은 네트워크 안에서 30gb를 쓰는 레디스 마스터 100대 정도가 리플리케이션을 동시에 재시작할 경우

##### 권장 설정 TIP
- maxclient 설정 50000
- RDB/AOF 설정 off
- 특정 commands disable
  - KEYS
  - aws의 ElasticCache는 이미 하고 있음
- 전체 장애의 90% 이상이 KEYS와 save 설정을 사용해서 발생
- 적절한 ziplist 설정

## Redis 데이터 분산

1. Consistent hashing
2. Sharding 
  - 데이터를 어떻게 나눌 것인가?
  - 데이터를 어떻게 찾을 것인가?
  - 상황마다 sharding 전략이 달라짐
  - Range
    - 그냥 특정 range를 정의하고 해당 range에 속하면 거기에 저장
    - 서버에 불균형이 생길 수 있음
    - 상황에 따라 노는 서버와 바쁜 서버가 차이가 날 수 있음
    - 확장은 편리함
  - Modular
    - 추가할 때 2배로 해야함
    - 데이터 이동이 정해져있음
    - 서버 한 대가 추가될 때 재분배 양이 많아짐
  - Indexed
    - 해당 key가 어디에 저장되어야 할 지 관리하는 서버가 따로 존재
3. Redis cluster
  - Hash 기반으로 slot 16384로 구분
    - Hash 알고리즘은 CRC16 사용
    - Slot : crc16(key) % 16384
    - 특정 REDIS 서버는 이 slot range를 가지고 있고, 데이터 migratons은 이 slot 단위의 데이터를 다른 서버로 전달(migrateCommand 이용)
  - 라이브러리 의존성 있음
  - 장점
    - 자체적인 primary, secondary failover
    - Slot 단위의 데이터 관리
  - 단점
    - 메모리 사용량이 더 많음
    - Migration 자체는 관리자가 시점을 결정해야 함
    - Library 구현 필요

## Redis Failover

##### Coordinator 기반 failover
- zookeeper, etcd, consul 등의 coordinator 사용
- coordinator 기반으로 설정을 관리한다면 동일한 방식으로 관리 가능
- 해당 기능을 이용하도록 개발 필요

##### VIP(virtual ip)/DNS 기반 failover
- 클라이언트에 추가적 구현이 필요없음
- VIP 기반은 외부로 서비스를 제공해야하는 서비스 업자에 유리
- DNS 기반은 DNS Cache TTL을 관리해야 함
  - 사용하는 언어별 dns 캐싱 정책을 잘 알아야 함
  - 툴에 따라 한 번 가져온 dns 정보를 다시 호출하지 않는 경우도 존재
- redis cluster 사용

##### Mornitoring factor
- redis info를 통한 정보
  - RSS
  - Used Memory
  - Connection 수
  - 초당 처리 요청 수
- System
  - CPU
  - Disk
  - Network rx/tx
- CPU가 100%를 칠 경우
  - 처리량이 매우 많다면?
    - 좀 더 cpu 성능이 좋은 서버로 이전
    - 실제 cpu 성능에 영향을 받지만,  단순 get/set은 초당 10만 이상 처리 가능
  - O(N) 계열의 특정 명령이 많은 경우
    - monitor 명령을 통해 특정 패턴을 파악하는 것이 필요
    - monitor 잘못 쓰면 부하로 해당 서버에 더 큰 문제를 일으킬 수 있음(짧게 쓰는 게 좋음)

## 결론

- 기본적으로 좋은 툴
- 메모리를 빡빡하게 쓸 경우, 관리 어려움
  - 32기가 장비라면 24기가 이상 사용하면 장비 증설을 고려하는 것이 좋음
  - write가 heavy 할 때는 migration도 매우 주의해야 함
- client-output-buffer-limit 설정 필요

##### Redis as Cache
- cache일 경우는 문제가 적게 발생
  - redis가 문제 있을 때 db 등의 부하가 어느 정도 증가하는지 확인 필요
  - consistent hashing도 실제 부하를 아주 균등하게 나누지는 않음. adaptavie consistent hashing을 이용해 볼 수도 있음

##### Redis as Persistent store
- 무조건 primary/secondary 구조로 구성이 필요함
- 메모리를 절대로 빡빡하게 사용하면 안됨
  - 정기적인 migration 필요
  - 가능하면 자동화 툴을 만들어서 이용
- RDB/AOF가 필요하다면 secondary에서만 구동
  - RDB보단 AOF가 조금 더 안정적
- 백업은 secondary에서



