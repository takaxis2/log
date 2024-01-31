### **4. 데이터 모델링의 3단계 진행**

앞에서 라이브러리 캐시 최적화와 데이터베이스 Call 최소화를 통한 성능 개선 방법을 알아보았다. 본 절에서는 데이터베이스 I/O 효율화 및 버퍼캐시 최적화 방법을 이해하는데 필요한 기본 개념과 원리를 소개한다. 데이터베이스 I/O 튜닝을 위해서는 인덱스, 조인, 옵티마이저 원리, 소트 원리 등에 관한 종합적인 이해가 필요한데, 이에 대한 자세한 내용은 3~5장에서 다룬다.

### **1. 블록 단위 I/O**

Oracle을 포함한 모든 DBMS에서 I/O는 블록(SQL Server 등 다른 DBMS는 페이지라는 용어를 사용) 단위로 이루어진다. 즉 하나의 레코드를 읽더라도 레코드가 속한 블록 전체를 읽는다. SQL 성능을 좌우하는 가장 중요한 성능지표는 액세스하는 블록 개수이며, 옵티마이저의 판단에 가장 큰 영향을 미치는 것도 액세스해야 할 블록 개수다. 블록 단위 I/O는 버퍼 캐시와 데이터 파일 I/O 모두에 적용된다.

- 데이터 파일에서 DB 버퍼 캐시로 블록을 적재할 때
- 데이터 파일에서 블록을 직접 읽고 쓸 때
- 버퍼 캐시에서 블록을 읽고 쓸 때
- 버퍼 캐시에서 변경된 블록을 다시 데이터 파일에 쓸 때

### **2. 메모리 I/O vs. 디스크I/O**

### **가. I/O 효율화 튜닝의 중요성**

디스크를 경유한 데이터 입출력은 디스크의 액세스 암(Arm)이 움직이면서 헤드를 통해 데이터를 읽고 쓰기 때문에 느린 반면, 메모리를 통한 입출력은 전기적 신호에 불과하기 때문에 디스크를 통한 I/O에 비해 비교할 수 없을 정도로 빠르다. 모든 DBMS는 읽고자 하는 블록을 먼저 버퍼 캐시에서 찾아보고, 없을 경우에만 디스크에서 읽어 버퍼 캐시에 적재한 후 읽기/쓰기 작업을 수행한다. 물리적인 디스크 I/O가 필요할 때면 서버 프로세스는 시스템에 I/O Call을 하고 잠시 대기 상태에 빠진다. 디스크 I/O 경합이 심할수록 대기 시간도 길어진다. ([그림 Ⅲ-1-12] 참조)

[![](https://dataonair.or.kr/publishing/img/knowledge/SQL_272.jpg)](https://dataonair.or.kr/publishing/img/knowledge/SQL_272.jpg)

모든 데이터를 메모리에 올려 놓고 사용할 수 있다면 좋겠지만 비용과 기술 측면에 한계가 있다. 메모리는 물리적으로 한정된 자원이므로, 결국 디스크 I/O를 최소화하고 버퍼 캐시 효율을 높이는 것이 데이터베이스 I/O 튜닝의 목표가 된다.

### **나. 버퍼 캐시 히트율(Buffer Cache Hit Ratio)**

버퍼 캐시 효율을 측정하는 지표로서, 전체 읽은 블록 중에서 메모리 버퍼 캐시에서 찾은 비율을 나타낸다. 즉, 버퍼 캐시 히트율(Buffer Cache Hit Ratio, 이하 BCHR)은 물리적인 디스크 읽기를 수반하지 않고 곧바로 메모리에서 블록을 찾은 비율을 말한다. Direct Path Read 방식 이외의 모든 블록 읽기는 버퍼 캐시를 통해 이뤄진다. 읽고자 하는 블록을 먼저 버퍼 캐시에서 찾아보고, 없을 때만 디스크로부터 버퍼 캐시에 적재한 후 읽어 들인다.

BCHR = (버퍼 캐시에서 곧바로 찾은 블록 수 / 총 읽은 블록 수) × 100

BCHR은 주로 시스템 전체적인 관점에서 측정하지만, 개별 SQL 측면에서 구해볼 수도 있는데 이 비율이 낮은 것이 SQL 성능을 떨어뜨리는 주원인이라고 할 수 있다

call count cpu elapsed disk query current rows ------ ---- ----- ------ ---- ----- ------ ---- Parse 15 0.00 0.08 0 0 0 0 Execute 44 0.03 0.03 0 0 0 0 Fetch 44 0.01 0.13 18 822 0 44 ------ ---- ----- ------ ---- ----- ------ ---- total 103 0.04 0.25 18 822 0 44

위에서 Disk 항목이 디스크를 경유한 블록 수를 의미하며, 버퍼 캐시에서 읽은 블록 수는 Query와 Current 항목을 더해서 구하게 된다. 따라서 위 샘플에서 BCHR은 98%다. 즉 100개 블록읽기를 요청하면 98개는 메모리에서 찾고, 나머지 2개는 디스크 I/O를 발생시켰다는 뜻이다.

- 총 읽은 블록 수 = 822
- 버퍼 캐시에서 곧바로 찾은 블록 수 = 822 - 18 = 804
- CHR = (822 - 18) / 822 = 97.8%

모든 블록 읽기는 버퍼 캐시를 경유하며, 디스크 I/O가 수반되더라도 먼저 버퍼 캐시에 적재한 후 읽는다고 했다. 총 읽은 블록 수(Query + Current)가 디스크로부터 읽은 블록 수를 이미 포함하므로, 총 읽은 블록 수를 840개(Disk + Query + Current)로 잘못 해석하지 않도록 주의하기 바란다. 논리적인 블록 요청 횟수를 줄이고, 물리적으로 디스크에서 읽어야 할 블록 수를 줄이는 것이 I/O 효율화 튜닝의 핵심 원리다. 같은 블록을 반복적으로 액세스하는 형태의 SQL은 논리적인 I/O 요청이 비효율적으로 많이 발생함에도 불구하고 BCHR은 매우 높게 나타난다. 이는 BCHR이 성능지표로서 갖는 한계점이라 할 수 있다. 예를 들어 NL Join에서 작은 Inner 테이블을 반복적으로 룩업(Lookup)하는 경우가 그렇다. 작은 테이블을 반복 액세스하면 모든 블록이 메모리에서 찾아져 BCHR은 높겠지만 일량이 작지 않고, 블록을 찾는 과정에서 래치(Latch) 경합과 버퍼 Lock 경합까지 발생한다면 메모리 I/O 비용이 디스크 I/O 비용보다 커질 수 있다. 따라서 논리적으로 읽어야 할 블록 수의 절대량이 많다면 반드시 튜닝을 통해 논리적인 블록 읽기를 최소화해야 한다.

### **다. 네트워크, 파일시스템 캐시가 I/O 효율에 미치는 영향**

대용량 데이터를 읽고 쓰는 데 다양한 네트워크 기술(DB서버와 스토리지 간에 NAS 서버나 SAN을 사용)이 사용됨에 따라 네트워크 속도도 SQL 성능에 크게 영향을 미치고 있다. 이에 하드웨어나 DBMS 벤더는 네트워크를 통한 데이터 전송속도를 향상시키려고 노력하고 있지만, 네트워크 전송량이 많을 수 밖에 없도록 SQL을 작성한다면 결코 좋은 성능을 기대할 수 없다. 따라서 SQL을 작성할 때는 다양한 I/O 튜닝 기법을 사용해서 네트워크 전송량을 줄이려고 노력하는 것이 중요하다. RAC 같은 클러스터링(Clustering) 데이터베이스 환경에선 인스턴스 간 캐시된 블록을 공유하므로 메모리 I/O 성능에도 네트워크 속도가 지대한 영향을 미치게 되었다. 같은 양의 디스크 I/O가 발생하더라도 I/O 대기 시간이 크게 차이 날 때가 있다. 디스크 경합 때문일 수도 있고, OS에서 지원하는 파일 시스템 버퍼 캐시와 SAN 캐시 때문일 수도 있다. SAN 캐시는 크다고 문제될 것이 없지만, 파일 시스템 버퍼캐시는 최소화해야 한다. 데이터베이스 자체적으로 캐시 영역을 갖고 있으므로 이를 위한 공간을 크게 할당하는 것이 더 효과적이다. 네트워크 문제이든, 파일시스템 문제이든 I/O 성능에 관한 가장 확실하고 근본적인 해결책은 논리적인 블록 요청 횟수를 최소화하는 것이다.

### **3. Sequential I/O vs. Random I/O**

[![](https://dataonair.or.kr/publishing/img/knowledge/SQL_273.jpg)](https://dataonair.or.kr/publishing/img/knowledge/SQL_273.jpg)

Sequential 액세스는 레코드간 논리적 또는 물리적인 순서를 따라 차례대로 읽어 나가는 방식이다. 인덱스 리프 블록에 위치한 모든 레코드는 포인터를 따라 논리적으로 연결돼 있고, 이 포인터를 따라 스캔하는 것([그림 Ⅲ-1-13]에서 ⑤번)은 Sequential 액세스 방식이다. 테이블 레코드 간에는 포인터로 연결되지 않지만 테이블을 스캔할 때는 물리적으로 저장된 순서대로 읽어 나가므로 이것 또한 Sequential 액세스 방식이다. Random 액세스는 레코드간 논리적, 물리적인 순서를 따르지 않고, 한 건을 읽기 위해 한 블록씩 접근하는 방식을 말한다.([그림 Ⅲ-1-13]에서 ①, ②, ③, ④, ⑥번) 블록 단위 I/O를 하더라도 한번 액세스할 때 Sequential 방식으로 그 안에 저장된 모든 레코드를 읽는다면 비효율은 없다. 반면, 하나의 레코드를 읽으려고 한 블록씩 Random 액세스한다면 매우 비효율적이라고 할 수 있다. 여기서 I/O튜닝의 핵심 원리 두 가지를 발견할 수 있다.

- Sequential 액세스에 의한 선택 비중을 높인다.
- Random 액세스 발생량을 줄인다

### **가. Sequential 액세스에 의한 선택 비중 높이기**

Sequential 액세스 효율성을 높이려면, 읽은 총 건수 중에서 결과집합으로 선택되는 비중을 높여야 한다. 즉, 같은 결과를 얻기 위해 얼마나 적은 레코드를 읽느냐로 효율성을 판단할 수 있다. 테스트를 통해 살펴보자.

- - 테스트용 테이블 생성 SQL> create table t 2 as 3 select * from all_objects 4 order by dbms_random.value; -- 테스트용 테이블 데이터 건수 : 49,906 SQL> select count(*) from t; COUNT(*) -------- 49906

T 테이블에는 49,906건의 레코드가 저장돼 있다.

select count(*) from t where owner like 'SYS%' Rows Row Source Operation ---- ------------------------------ 1 SORT AGGREGATE (cr=691 pr=0 pw=0 time=13037 us) 24613 TABLE ACCESS FULL T (cr=691 pr=0 pw=0 time=98473 us)

위 SQL은 24,613개 레코드를 선택하려고 49,906개 레코드를 읽었으므로 49%가 선택되었다. Table Full Scan에서 이 정도면 나쁘지 않다. 읽은 블록 수는 691개였다.

select count(*) from t where owner like 'SYS%' and object_name = 'ALL_OBJECTS' Rows Row Source Operation ---- ------------------------------ 1 SORT AGGREGATE (cr=691 pr=0 pw=0 time=7191 us) 1 TABLE ACCESS FULL T (cr=691 pr=0 pw=0 time=7150 us)

위 SQL은 49,906개 레코드를 스캔하고 1개 레코드를 선택했다. 선택 비중이 0.002% 밖에 되지 않으므로 Table Full Scan 비효율이 크다. 여기서도 읽은 블록 수는 똑같이 691개다. 이처럼 테이블을 스캔하면서 읽은 레코드 중 대부분 필터링되고 일부만 선택된다면 아래 처럼 인덱스를 이용하는 게 효과적이다.

create index t_idx on t(owner, object_name); select /*+ index(t t_idx) */ count(*) from t where owner like 'SYS%' and object_name = 'ALL_OBJECTS' Rows Row Source Operation ---- ------------------------------ 1 SORT AGGREGATE (cr=76 pr=0 pw=0 time=7009 us) 1 INDEX RANGE SCAN T_IDX (cr=76 pr=0 pw=0 time=6972 us)(Object ID 55337)

위 SQL에서 참조하는 칼럼이 모두 인덱스에 있으므로 인덱스만 스캔하고 결과를 구할 수 있었다. 하지만 1개의 레코드를 읽기 위해 76개의 블록을 읽어야 했다. 테이블뿐만 아니라 인덱스를 Sequential 액세스 방식으로 스캔할 때도 비효율이 나타날 수 있고, 조건절에 사용된 칼럼과 연산자 형태, 인덱스 구성에 의해 효율성이 결정된다. 아래는 인덱스 구성 칼럼의 순서를 변경한 후에 테스트한 결과다.

drop index t_idx; create index t_idx on t(object_name, owner); select /*+ index(t t_idx) */ count(*) from t where owner like 'SYS%' and object_name = 'ALL_OBJECTS' Rows Row Source Operation ---- ------------------------------ 1 SORT AGGREGATE (cr=2 pr=0 pw=0 time=44 us) 1 INDEX RANGE SCAN T_IDX (cr=2 pr=0 pw=0 time=23 us)(Object ID 55338)

루트와 리프, 단 2개의 인덱스 블록만 읽었다. 한 건을 얻으려고 읽은 건수도 한 건일 것이므로 가장 효율적인 방식으로 Sequential 액세스를 수행했다.

### **나. Random 액세스 발생량 줄이기**

Random 액세스 발생량을 낮추는 방법을 살펴보자. 인덱스에 속하지 않는 칼럼(object_id)을 참조하도록 쿼리를 변경함으로써 테이블 액세스가 발생하도록 할 것이다.

drop index t_idx; create index t_idx on t(owner); select object_id from t where owner = 'SYS' and object_name = 'ALL_OBJECTS' Rows Row Source Operation ---- ------------------------------ 1 TABLE ACCESS BY INDEX ROWID T (cr=739 pr=0 pw=0 time=38822 us) 22934 INDEX RANGE SCAN T_IDX (cr=51 pr=0 pw=0 time=115672 us)(Object ID 55339)

인덱스로부터 조건을 만족하는 22,934건을 읽어 그 횟수만큼 테이블을 Random 액세스하였다. 최종적으로 한 건이 선택된 것에 비해 너무 많은 Random 액세스가 발생했다. 아래는 인덱스를 변경하여 테이블 Random 액세스 발생량을 줄인 결과다.

drop index t_idx; create index t_idx on t(owner, object_name); select object_id from t where owner = 'SYS' and object_name = 'ALL_OBJECTS' Rows Row Source Operation ---- ------------------------------ 1 TABLE ACCESS BY INDEX ROWID T (cr=4 pr=0 pw=0 time=67 us) 1 INDEX RANGE SCAN T_IDX (cr=3 pr=0 pw=0 time=51 us)(Object ID 55340)

인덱스로부터 1건을 출력했으므로 테이블을 1번 방문한다. 실제 발생한 테이블 Random 액세스도 1(=4-3)번이다. 같은 쿼리를 수행했는데 인덱스 구성이 바뀌자 테이블 Random 액세스가 대폭 감소한 것이다. 지금까지의 테스트 결과가 쉽게 이해되지 않을 수도 있다. 만약 그렇다면, 세부적인 인덱스 튜닝 원리를 설명한 4장을 읽고서 다시 학습하기 바란다.

### **4. Single Block I/O vs. MultiBlock I/O**

Single Block I/O는 한번의 I/O Call에 하나의 데이터 블록만 읽어 메모리에 적재하는 방식이다. 인덱스를 통해 테이블을 액세스할 때는, 기본적으로 인덱스와 테이블 블록 모두 이 방식을 사용한다. MultiBlock I/O는 I/O Call이 필요한 시점에, 인접한 블록들을 같이 읽어 메모리에 적재하는 방식이다. Table Full Scan처럼 물리적으로 저장된 순서에 따라 읽을 때는 인접한 블록들을 같이 읽는 것이 유리하다. ‘인접한 블록’이란, 한 익스텐트(Extent)내에 속한 블록을 말한다. 달리 말하면, MultiBlock I/O 방식으로 읽더라도 익스텐트 범위를 넘어서까지 읽지는 않는다. 인덱스 스캔 시에는 Single Block I/O 방식이 효율적이다. 인덱스 블록간 논리적 순서(이중 연결 리스트 구조로 연결된 순서)는 데이터 파일에 저장된 물리적인 순서와 다르기 때문이다. 물리적으로 한 익스텐트에 속한 블록들을 I/O Call 시점에 같이 메모리에 올렸는데, 그 블록들이 논리적 순서로는 한참 뒤쪽에 위치할 수 있다. 그러면 그 블록들은 실제 사용되지 못한 채 버퍼 상에서 밀려나는 일이 발생한다. 하나의 블록을 캐싱하려면 다른 블록을 밀어내야 하는데, 이런 현상이 자주 발생한다면 앞에서 소개한 버퍼 캐시 효율만 떨어뜨리게 된다. 대량의 데이터를 MultiBlock I/O 방식으로 읽을 때 Single Block I/O 보다 성능상 유리한 이유는 I/O Call 발생 횟수를 줄여주기 때문이다. 아래 예제를 통해 Single Block I/O 방식과 MultiBlock I/O 방식의 차이점을 설명해 보자.

create table t as select * from all_objects; alter table t add constraint t_pk primary key(object_id); select /*+ index(t) */ count(*) from t where object_id > 0 call count cpu elapsed disk query current rows ----- ---- ---- ------ ---- ---- ----- ---- Parse 1 0.00 0.00 0 0 0 0 Execute 1 0.00 0.00 0 0 0 0 Fetch 2 0.26 0.25 64 65 0 1 ----- ---- ---- ------ ---- ---- ----- ---- total 4 0.26 0.25 64 65 0 1 Rows Row Source Operation ---- ------------------------------ 1 SORT AGGREGATE (cr=65 r=64 w=0 time=256400 us) 31192 INDEX RANGE SCAN T_PK (cr=65 r=64 w=0 time=134613 us) Elapsed times include waiting on following events: Event waited on Times Max. Wait Total Waited ------------------------------- Waited -------- --------- SQL*Net message to client 2 0.00 0.00 db file sequential read 64 0.00 0.00 SQL*Net message from client 2 0.05 0.05

위 실행 결과를 보면 64개 인덱스 블록을 디스크에서 읽으면서 64번의 I/O Call(db file sequential read 대기 이벤트)이 발생했다. 아래는 같은 양의 인덱스 블록을 MultiBlock I/O 방식으로 수행한 결과다.

- - 디스크 I/O가 발생하도록 버퍼 캐시 Flushing alter system flush buffer_cache; -- Multiblock I/O 방식으로 인덱스 스캔 select /*+ index_ffs(t) */ count(*) from t where object_id > 0 call count cpu elapsed disk query current rows ----- ---- ---- ------ ---- ----- ------ ---- Parse 1 0.00 0.00 0 0 0 0 Execute 1 0.00 0.00 0 0 0 0 Fetch 2 0.26 0.26 64 69 0 1 ----- ---- ---- ------ ---- ----- ------ ---- total 4 0.26 0.26 64 69 0 1 Rows Row Source Operation ---- ------------------------------ 1 SORT AGGREGATE (cr=69 r=64 w=0 time=267453 us) 31192 INDEX FAST FULL SCAN T_PK (cr=69 r=64 w=0 time=143781 us) Elapsed times include waiting on following events: Event waited on Times Max. Wait Total Waited ------------------------------ Waited ------- --------- SQL*Net message to client 2 0.00 0.00 db file scattered read 9 0.00 0.00 SQL*Net message from client 2 0.35 0.36

똑같이 64개 블록을 디스크에서 읽었는데, I/O Call이 9번(db file scattered read 대기 이벤트)에 그쳤다. 참고로, 위 테스트는 Oracle 9i에서 수행한 것이다. Oracle 10g부터는 Index Range Scan 또는 Index Full Scan 일 때도 Multiblock I/O 방식으로 읽는 경우가 있는데, 위처럼 테이블 액세스 없이 인덱스만 읽고 처리할 때가 그렇다. 인덱스를 스캔하면서 테이블을 Random 액세스할 때는 9i 이전과 동일하게 테이블과 인덱스 블록을 모두 Single Block I/O 방식으로 읽는다. Single Block I/O 방식으로 읽은 블록들은 LRU 리스트 상 MRU 쪽(end)으로 위치하므로 한번 적재되면 버퍼 캐시에 비교적 오래 머문다. 반대로 MultiBlock I/O 방식으로 읽은 블록들은 LRU 리스트 상 LRU 쪽(end)으로 연결되므로 적재된 지 얼마 지나지 않아 1순위로 버퍼캐시에서 밀려난다.

### **5. I/O 효율화 원리**

논리적인 I/O 요청 횟수를 최소화하는 것이 I/O 효율화 튜닝의 핵심 원리다. I/O 때문에 시스템 성능이 낮게 측정될 때 하드웨어적인 방법을 통해 I/O 성능을 향상 시킬 수도 있다. 하지만 SQL 튜닝을 통해 I/O 발생 횟수 자체를 줄이는 것이 더 근본적이고 확실한 해결 방안이다. 애플리케이션 측면에서의 I/O 효율화 원리는 다음과 같이 요약할 수 있다.

- 필요한 최소 블록만 읽도록 SQL 작성
- 최적의 옵티마이징 팩터 제공
- 필요하다면, 옵티마이저 힌트를 사용해 최적의 액세스 경로로 유도

### **가. 필요한 최소 블록만 읽도록 SQL 작성**

데이터베이스 성능은 I/O 효율에 달렸고, 이를 달성하려면 동일한 데이터를 중복 액세스하지 않고, 필요??령을 사용자는 최소 일량을 요구하는 형태로 논리적인 집합을 정의하고, 효율적인 처리가 가능하도록 작성하는 것이 무엇보다 중요하다. 아래는 비효율적인 중복 액세스를 없애고 필요한 최소 블록만 액세스하도록 튜닝한 사례다.

select a.카드번호 , a.거래금액 전일_거래금액 , b.거래금액 주간_거래금액 , c.거래금액 전월_거래금액 , d.거래금액 연중_거래금액 from ( -- 전일거래실적 select 카드번호, 거래금액 from 일별카드거래내역 where 거래일자 = to_char(sysdate-1,'yyyymmdd') ) a , ( -- 전주거래실적 select 카드번호, sum(거래금액) 거래금액 from 일별카드거래내역 where 거래일자 between to_char(sysdate-7,'yyyymmdd') and to_char(sysdate-1,'yyyymmdd') group by 카드번호 ) b , ( -- 전월거래실적 select 카드번호, sum(거래금액) 거래금액 from 일별카드거래내역 where 거래일자 between to_char(add_months(sysdate,-1),'yyyymm') || '01' and to_char(last_day(add_months(sysdate,-1)),'yyyymmdd') group by 카드번호 ) c , ( -- 연중거래실적 select 카드번호, sum(거래금액) 거래금액 from 일별카드거래내역 where 거래일자 between to_char(add_months(sysdate,-12),'yyyymmdd') and to_char(sysdate-1,'yyyymmdd') group by 카드번호 ) d where b.카드번호 (+) = a.카드번호 and c.카드번호 (+) = a.카드번호 and d.카드번호 (+) = a.카드번호

위 SQL은 어제 거래가 있었던 카드에 대한 전일, 주간, 전월, 연중 거래 실적을 집계하고 있다. 논리적인 전체 집합은 과거 1년치인데, 전일, 주간, 전월 데이터를 각각 액세스한 후 조인한 것을 볼 수 있다. 전일 데이터는 총 4번을 액세스한 셈이다. SQL을 아래와 같이 작성하면 과거 1년치 데이터를 한번만 읽고 전일, 주간, 전월 결과를 구할 수 있다. 즉 논리적인 집합 재구성을 통해 액세스해야 할 데이터 양을 최소화 할 수 있다.

select 카드번호 , sum( case when 거래일자 = to_char(sysdate-1,'yyyymmdd') then 거래금액 end ) 전일_거래금액 , sum( case when 거래일자 between to_char(sysdate-7,'yyyymmdd') and to_char(sysdate-1,'yyyymmdd') then 거래금액 end ) 주간_거래금액 , sum( case when 거래일자 between to_char(add_months(sysdate,-1),'yyyymm') || '01' and to_char(last_day(add_months(sysdate,-1)),'yyyymmdd') then 거래금액 end ) 전월_거래금액 , sum( 거래금액 )연중_거래금액 from 일별카드거래내역 where 거래일자 between to_char(add_months(sysdate,-12),'yyyymmdd') and to_char(sysdate-1,'yyyymmdd') group by 카드번호 having sum( case when 거래일자 = to_char(sysdate-1,'yyyymmdd') then 거래금액 end ) > 0

### **나. 최적의 옵티마이징 팩터 제공**

옵티마이저가 블록 액세스를 최소화하면서 효율적으로 처리할 수 있도록 하려면 최적의 옵티마이징 팩터를 제공해 주어야 한다.

- 략적인 인덱스 구성전략적인 인덱스 구성은 가장 기본적인 옵티마이징 팩터다.
- DBMS가 제공하는 기능 활용인덱스 외에도 DBMS가 제공하는 다양한 기능을 적극적으로 활용한다. 인덱스, 파티션, 클러스터, 윈도우 함수 등을 적극 활용해 옵티마이저가 최적의 선택을 할 수 있도록 한다.
- 옵티마이저 모드 설정옵티마이저 모드(전체 처리속도 최적화, 최초 응답속도 최적화)와 그 외 옵티마이저 행동에 영향을 미치는 일부 파라미터를 변경해 주는 것이 도움이 될 수 있다.
- 통계정보옵티마이저에게 정확한 정보를 제공한다.

### **다. 필요하다면, 옵티마이저 힌트를 사용해 최적의 액세스 경로로 유도**

최적의 옵티마이징 팩터를 제공했다면 가급적 옵티마이저 판단에 맡기는 것이 바람직하지만 옵티마이저가 생각만큼 최적의 실행계획을 수립하지 못하는 경우가 종종 있다. 그럴 때는 어쩔 수 없이 힌트를 사용해야 한다. 아래는 옵티마이저 힌트를 이용해 실행계획을 제어하는 방법을 예시하고 있다.

[예제] Oracle select /*+ leading(d) use_nl(e) index(d dept_loc_idx) */ * from emp e, dept d where e.deptno = d.deptno and d.loc = 'CHICAGO' [예제] SQL Server select * from dept d with (index(dept_loc_idx)), emp e where e.deptno = d.deptno and d.loc = 'CHICAGO' option (force order, loop join)

옵티마이저 힌트를 사용할 때는 의도한 실행계획으로 수행되는지 반드시 확인해야 한다.

CBO 기술이 고도로 발전하고 있긴 하지만 여러 가지 이유로 옵티마이저 힌트의 사용은 불가피하다. 따라서 데이터베이스 애플리케이션 개발자라면 인덱스, 조인, 옵티마이저의 기본 원리를 이해하고, 그것을 바탕으로 최적의 액세스 경로로 유도할 수 있는 능력을 필수적으로 갖추어야 한다. 3장부터 그런 원리들을 하나씩 학습하게 될 것이다.