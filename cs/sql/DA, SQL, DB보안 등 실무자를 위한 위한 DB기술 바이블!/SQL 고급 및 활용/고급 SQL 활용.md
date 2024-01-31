장에서 설명한 것처럼 데이터베이스 Call을 반복적으로 일으키는 프로그램을 One-SQL로 통합했을 때 얻는 성능개선 효과는 매우 극적이다. 본 절에서는 복잡한 처리절차를 One-SQL로 구현하는 데 적용할 수 있는 몇가지 유용한 기법들을 소개하고자 한다.

### **1. CASE문 활용**

[그림 Ⅲ-5-1] 왼쪽에 있는 월별납입방법별집계 테이블을 읽어 오른쪽 월요금납부실적과 같은 형태로 가공하려고 한다.

[![](https://dataonair.or.kr/publishing/img/knowledge/SQL_500.jpg)](https://dataonair.or.kr/publishing/img/knowledge/SQL_500.jpg)

위와 같은 업무를 아래와 같은 SQL로 구현하는 개발자가 있을까 싶겠지만, 의외로 아주 많다.

INSERT INTO 월별요금납부실적 (고객번호, 납입월, 지로, 자동이체, 신용카드, 핸드폰, 인터넷) SELECT K.고객번호, '200903' 납입월 , A.납입금액 지로 , B.납입금액 자동이체 , C.납입금액 신용카드 , D.납입금액 핸드폰 , E.납입금액 인터넷 FROM 고객 K ,(SELECT 고객번호, 납입금액 FROM 월별납입방법별집계 WHERE 납입월 = '200903' AND 납입방법코드 = 'A') A ,(SELECT 고객번호, 납입금액 FROM 월별납입방법별집계 WHERE 납입월 = '200903' AND 납입방법코드 = 'B') B ,(SELECT 고객번호, 납입금액 FROM 월별납입방법별집계 WHERE 납입월 = '200903' AND 납입방법코드 = 'C') C ,(SELECT 고객번호, 납입금액 FROM 월별납입방법별집계 WHERE 납입월 = '200903' AND 납입방법코드 = 'D') D ,(SELECT 고객번호, 납입금액 FROM 월별납입방법별집계 WHERE 납입월 = '200903' AND 납입방법코드 = 'E') E WHERE A.고객번호(+) = K.고객번호 AND B.고객번호(+) = K.고객번호 AND C.고객번호(+) = K.고객번호 AND D.고객번호(+) = K.고객번호 AND E.고객번호(+) = K.고객번호 AND NVL(A.납입금액,0)+NVL(B.납입금액,0)+NVL(C.납입금액,0)+NVL(D.납입금액,0)+NVL(E.납입금액,0) > 0

효율을 고려하지 않은 One-SQL은 누구나 작성할 수 있다. One-SQL로 작성하는 자체가 중요한 것이 아니라 어떻게 I/O 효율을 달성할 지가 중요한데, 이는 동일 레코드를 반복 액세스하지 않고 얼마만큼 블록 액세스 양을 최소화할 수 있느냐에 달렸다. I/O 효율을 고려한다면 SQL을 아래와 같이 작성해야 한다.

INSERT INTO 월별요금납부실적 (고객번호, 납입월, 지로, 자동이체, 신용카드, 핸드폰, 인터넷) SELECT 고객번호, 납입월 , NVL(SUM(CASE WHEN 납입방법코드 = 'A' THEN 납입금액 END), 0) 지로 , NVL(SUM(CASE WHEN 납입방법코드 = 'B' THEN 납입금액 END), 0) 자동이체 , NVL(SUM(CASE WHEN 납입방법코드 = 'C' THEN 납입금액 END), 0) 신용카드 , NVL(SUM(CASE WHEN 납입방법코드 = 'D' THEN 납입금액 END), 0) 핸드폰 , NVL(SUM(CASE WHEN 납입방법코드 = 'E' TJEM 납입금액 END), 0) 인터넷 FROM 월별납입방법별집계 WHERE 납입월 = '200903' GROUP BY 고객번호, 납입월 ;

참고로, SQL Server에선 2005 버전부터 Pivot 구문을 지원하고, Oracle도 11g부터 지원하기 시작했으므로 앞으론 이것을 쓰면 된다. 그렇지만, 위와 같이 CASE문이나 DECODE 함수를 활용하는 기법은 IFELSE 같은 분기조건을 포함한 복잡한 처리절차를 One-SQL로 구현하는 데 반드시 필요하고, 다른 비슷한 업무에도 응용할 수 있으므로 반드시 숙지하기 바란다.

### **2. 데이터 복제 기법 활용**

SQL을 작성하다 보면 데이터 복제 기법을 활용해야 할 때가 많다. 전통적으로 많이 쓰던 방식은 아래와 같은 복제용 테이블(copy_t)을 미리 만들어두고 이를 활용하는 것이다.

create table copy_t ( no number, no2 varchar2(2) ); insert into copy_t select rownum, lpad(rownum, 2, '0') from big_table where rownum <= 31; alter table copy_t add constraint copy_t_pk primary key(no); create unique index copy_t_no2_idx on copy_t(no2);

이 테이블과 아래와 같이 조인절 없이 조인(Cross Join)하면 카티션 곱(Cartesian Product)이 발생해 데이터가 2배로 복제된다. 3배로 복제하려면 no <= 3 조건으로 바꿔주면 된다.

select * from emp a, copy_t b where b.no <= 2;

Oracle 9i부터는 dual 테이블을 사용하면 편하다. 아래와 같이 dual 테이블에 start with절 없는 connect by 구문을 사용하면 두 레코드를 가진 집합이 자동으로 만들어진다. (9i에서는 버그가 있어 아래 쿼리를 인라인 뷰에 담을 때만 데이터 복제가 일어난다.)

SQL> select rownum no from dual connect by level <= 2; NO -------- 1 2

아래는 dual 테이블을 이용해 emp 테이블을 2배로 복제하는 예시다.

SQL> select * from emp a, (select rownum no from dual connect by level <= 2) b;

이런 데이터 복제 기법은 다양한 업무 처리에 응용할 수 있다. 아래는 카드상품분류와 고객등급 기준으로 거래실적을 집계하면서 소계까지 한번에 구하는 방법을 예시한 것이다.

SQL> break on 카드상품분류 SQL> select a.카드상품분류 2 ,(case when b.no = 1 then a.고객등급 else '소계' end) as 고객등급 3 , sum(a.거래금액) as 거래금액 4 from (select 카드.카드상품분류 as 카드상품분류 5 , 고객.고객등급 as 고객등급 6 , sum(거래금액) as 거래금액 7 from 카드월실적, 카드, 고객 8 where 실적년월 = '201008' 9 and 카드.카드번호 = 카드월실적.카드번호 10 and 고객,고객번호 = 카드.고객번호 11 group by 카드.카드상품분류, 고객.고객등급) a 12 , copy_t b 13 where b.no <= 2 14 group by a.카드상품분류, b.no, (case when b.no = 1 then a.고객등급 else '소계' end) 카드상품분류 고객등급 거래금액 -------- ------ --------- 상품분류A VIP 500000000 일반 300000000 소계0000000

상단에 있는 break 명령어는 카드상품분류가 반복적으로 출력되지 않도록 하기 위한 것으로서, Oracle SQL*Plus에서만 사용 가능하다.

### **3. Union All을 활용한 M:M 관계의 조인**

M:M 관계의 조인을 해결하거나 Full Outer Join을 대체하는 용도로 Union All을 활용할 수 있다. [그림 Ⅲ-5-2]처럼 부서별판매계획과 채널별판매실적 테이블이 있다. 이 두 테이블을 이용해 월별로 각 상품의 계획 대비 판매 실적을 집계하려고 한다. 그런데 상품과 연월을 기준으로 볼 때 두 테이블은 M:M 관계이므로 그대로 조인하면 카티션 곱(Cartesian Product)이 발생한다.

[![](https://dataonair.or.kr/publishing/img/knowledge/SQL_501.jpg)](https://dataonair.or.kr/publishing/img/knowledge/SQL_501.jpg)

아래와 같이 상품, 연월 기준으로 group by를 먼저 수행하고 나면 두 집합은 1:1 관계가 되므로 Full Outer Join을 통해 원하는 결과집합을 얻을 수 있다.

select nvl(a.상품, b.상품) as 상품 , nvl(a.계획연월, b.판매연월) as 연월 , nvl(계획수량, 0) 계획수량 , nvl(판매수량, 0) 판매수량 from ( select 상품, 계획연월, sum(계획수량) 계획수량 from 부서별판매계획 where 계획연월 between '200901' and '200903' group by 상품, 계획연월 ) a full outer join ( select 상품, 판매연월, sum(판매수량) 판매수량 from 채널별판매실적 where 판매연월 between '200901' and '200903' group by 상품, 판매연월 ) b on a.상품 = b.상품 and a.계획연월 = b.판매연월

하지만, DBMS와 버전에 따라 Full Outer Join을 아래와 같이 비효율적으로 처리하기도 한다. 한 테이블을 두 번씩 액세스하는 것을 확인하기 바란다.

Execution Plan ------------------------------------------------------------- 0 SELECT STATEMENT Optimizer=CHOOSE (Cost=14 Card=8 Bytes=352) 1 0 VIEW (Cost=14 Card=8 Bytes=352) 2 1 UNION-ALL 3 2 HASH JOIN (OUTER) (Cost=8 Card=7 Bytes=308) 4 3 VIEW (Cost=4 Card=7 Bytes=154) 5 4 SORT (GROUP BY) (Cost=4 Card=7 Bytes=98) 6 5 TABLE ACCESS (FULL) OF '부서별판매계획' (Cost=2 Card=7 Bytes=98) 7 3 VIEW (Cost=4 Card=6 Bytes=132) 8 7 SORT (GROUP BY) (Cost=4 Card=6 Bytes=84) 9 8 TABLE ACCESS (FULL) OF '채널별판매실적' (Cost=2 Card=6 Bytes=84) 10 2 SORT (GROUP BY) (Cost=4 Card=1 Bytes=14) 11 10 FILTER 12 11 TABLE ACCESS (FULL) OF '채널별판매실적' (Cost=2 Card=1 Bytes=14) 13 11 SORT (GROUP BY NOSORT) (Cost=2 Card=1 Bytes=14) 14 13 FILTER 15 14 TABLE ACCESS (FULL) OF '부서별판매계획' (Cost=2 Card=1 Bytes=14)

좀 더 효과적인 방법을 찾기 위해, 우선 두 테이블을 이어서 출력해 보자.

select '계획' as 구분, 상품, 계획연월 as 연월, 판매부서, null as 판매채널 , 계획수량, to_number(null) as 실적수량 from 부서별판매계획 where 계획연월 between '200901' and '200903' union all select '실적', 상품, 판매연월 as 연월, null as 판매부서, 판매채널 , to_number(null) as 계획수량, 판매수량 from 채널별판매실적 where 판매연월 between '200901' and '200903' 구분 ---- 계획 계획 계획 계획 계획 계획 계획 상품 ---- 상품A 상품A 상품A 상품B 상품B 상품C 상품C 연월 ---- 200901 200902 200903 200901 200902 200901 200903 판매부서 ------ 10 20 10 10 30 30 20 판매채널 ------ 계획수량 ------ 10000 5000 20000 20000 15000 15000 20000 실적수량 ----- 실적 실적 실적 실적 실적 실적 상품A 상품A 상품B 상품B 상품C 상품C 200901 200903 200902 200903 200901 200902 대리점 온라인 온라인 위탁 대리점 위탁 7000 8000 12000 19000 13000 18000

이렇게 두 집합을 함께 출력하고 보니 의외로 쉽게 방법이 찾아진다. 방금 출력한 전체 집합을 상품, 연월 기준으로 group by하면서 계획수량과 실적수량을 집계해 보자. 그러면 아래와 같이 월별 판매계획과 실적을 대비해서 보여줄 수 있다.

select 상품, 연월, nvl(sum(계획수량), 0) as 계획수량, nvl(sum(실적수량), 0) as 실적수량 from ( select 상품, 계획연월 as 연월, 계획수량, to_number(null) as 실적수량 from 부서별판매계획 where 계획연월 between '200901' and '200903' union all select 상품, 판매연월 as 연월, to_number(null) as 계획수량, 판매수량 from 채널별판매실적 where 판매연월 between '200901' and '200903' ) a group by 상품, 연월 ; 상품 연월 계획수량 판매수량 ---- ------ ------- ------ 상품A 200901 10000 7000 상품A 200902 5000 0 상품A 200903 20000 8000 상품B 200901 20000 0 상품B 200902 15000 12000 상품B 200903 0 19000 상품C 200901 15000 13000 상품C 200902 0 18000 상품C 200903 20000 0

이처럼 Union All을 이용하면 M:M 관계의 조인이나 Full Outer Join을 쉽게 해결할 수 있다. SQL Server에선 nvl 대신 isnull 함수를 사용하고, to_number 대신 cast 함수를 사용하기 바란다.

### **4. 페이징 처리**

1장에서 데이터베이스 Call과 네트워크 부하를 설명하면서 페이징 처리 활용의 중요성을 강조하였다. 조회할 데이터가 일정량 이상이고 수행빈도가 높다면 반드시 페이징 처리를 해야 한다는 것이 결론이었다. 그러면 어떻게 페이징 처리를 구현하는 것이 효과적인지, 지금부터 살펴보기로 하자. 페이징 처리는 출력방식에 대한 사용자 요건과 애플리케이션 아키텍처, 그리고 인덱스 구성 등에 따라 다양한 방법이 존재하므로 여기서 소개한 기본 패턴을 바탕으로 각 개발 환경에 맞게 응용하기 바란다. [그림 Ⅲ-5-3]에 있는 시간별종목거래 테이블을 예로 들어 설명해 보자.

[![](https://dataonair.or.kr/publishing/img/knowledge/SQL_502.jpg)](https://dataonair.or.kr/publishing/img/knowledge/SQL_502.jpg)

### **가. 일반적인 페이징 처리용 SQL**

아래는 관심 종목에 대해 사용자가 입력한 거래일시 이후 거래 데이터를 페이징 처리 방식으로 조회하는 SQL이다.

SELECT * FROM ( SELECT ROWNUM NO, 거래일시, 체결건수 , 체결수량, 거래대금, COUNT(*) OVER () CNT ……………………………………… ① FROM ( SELECT 거래일시, 체결건수, 체결수량, 거래대금 FROM 시간별종목거래 WHERE 종목코드 = :isu_cd -- 사용자가 입력한 종목코드 AND 거래일시 >= :trd_time -- 사용자가 입력한 거래일자 또는 거래일시 ORDER BY 거래일시 ……………………………………………………………………………………… ② ) WHERE ROWNUM <= :page*:pgsize+1 ………………………………………………………………… ③ ) WHERE NO BETWEEN (:page-1)*:pgsize+1 AND :pgsize*:page …………………………… ④ Execution Plan ------------------------------------------------------------- 0 SELECT STATEMENT Optimizer=ALL_ROWS (Cost=5 Card=1 Bytes=75) 1 0 FILTER 2 1 VIEW (Cost=5 Card=1 Bytes=75) 3 2 WINDOW (BUFFER) (Cost=5 Card=1 Bytes=49) 4 3 COUNT (STOPKEY) 5 4 VIEW (Cost=5 Card=1 Bytes=49) 6 5 TABLE ACCESS (BY INDEX ROWID) OF '시간별종목거래' (TABLE) (Card=1 Bytes=56) 7 6 INDEX (RANGE SCAN) OF '시간별종목거래_PK' (INDEX (UNIQUE)) (Card=1)

:pgsize 변수에는 사용자가 ‘다음(▶)’ 버튼을 누를 때마다 Fetch해 올 데이터 건수를 입력하고, :page 변수에는 그때 출력하고자 하는 페이지 번호를 입력하면 된다.

① ‘다음’ 페이지에 읽을 데이터가 더 있는지 확인하는 용도다. 결과집합에서 CNT 값을 읽었을 때 :pgsize*:page 보다 크면 ‘다음’ 페이지에 출력할 데이터가 더 있음을 알 수 있다. 전체 건수를 세지 않고도 ‘다음’ 버튼을 활성화할지를 판단할 수 있어 유용하다. 이 기능이 필요치 않을 때는 ③번 라인에서 +1을 제거하면 된다. ② [종목코드 + 거래일시]순으로 정렬된 인덱스가 있을 때는 자동으로 Sort 오퍼레이션이 생략된다. NOSORT를 위해 활용 가능한 인덱스가 없으면 결과집합 전체를 읽는 비효율은 어쩔 수 없지만, TOP-N 쿼리 알고리즘이 작동하므로 SORT 부하만큼은 최소화할 수 있다. TOP-N 쿼리 알고리즘은 잠시 후 2절 5항에서 설명한다. ③ :pgsize = 10 이고 :page = 3 일 때, 거래일시 순으로 31건만 읽는다. ④ :pgsize = 10 이고 :page = 3 일 때, 안쪽 인라인 뷰에서 읽은 31건 중 21~30번째 데이터 즉, 3 페이지만 리턴한다.

성능과 I/O 효율을 위해서는 [종목코드 + 거래일시] 순으로 구성된 인덱스가 필요하며, 이 인덱스의 도움을 받을 수만 있다면 정렬작업을 수행하지 않아도 되므로 전체 결과집합이 아무리 크더라도 첫 페이지만큼은 가장 최적의 수행 속도를 보인다. 따라서 사용자가 주로 앞쪽 일부 데이터만 조회할 때 아주 효과적인 구현방식이다. 실제 대부분 업무에서 앞쪽 일부만 조회하므로 표준적인 페이징 처리 구현 패턴으로 가장 적당하다고 하겠다.

### **나. 뒤쪽 페이지까지 자주 조회할 때**

만약 사용자가 ‘다음’ 버튼을 계속 클릭해서 뒤쪽으로 많이 이동하는 업무라면 위 쿼리는 비효율적이다. 인덱스 도움을 받아 NOSORT 방식으로 처리하더라도 앞에서 읽었던 레코드들을 계속 반복적으로 액세스해야 하기 때문이다. 인덱스마저 없다면 전체 조회 대상 집합을 매번 반복적으로 액세스하게 된다. 뒤쪽의 어떤 페이지로 이동하더라도 빠르게 조회되도록 구현해야 한다면? 앞쪽 레코드를 스캔하지 않고 해당 페이지 레코드로 바로 찾아가도록 구현해야 한다. 아래는 첫 번째 페이지를 출력하고 나서 ‘다음’ 버튼을 누를 때의 구현 예시다. 한 페이지에 10건씩 출력하는 것으로 가정하자.

SELECT 거래일시, 체결건수, 체결수량, 거래대금 FROM ( SELECT 거래일시, 체결건수, 체결수량, 거래대금 FROM 시간별종목거래 A WHERE :페이지이동 = 'NEXT' AND 종목코드 = :isu_cd AND 거래일시 >= :trd_time ORDER BY 거래일시 ) WHERE ROWNUM <= 11 Execution Plan ------------------------------------------------------------- 0 SELECT STATEMENT Optimizer=ALL_ROWS (Cost=5 Card=1 Bytes=49) 1 0 COUNT (STOPKEY) 2 1 VIEW (Cost=5 Card=1 Bytes=49) 3 2 FILTER 4 3 TABLE ACCESS (BY INDEX ROWID) OF '시간별종목거래' (TABLE) (Card=1 Bytes=56) 5 4 INDEX (RANGE SCAN) OF '시간별종목거래_PK' (INDEX (UNIQUE)) (Card=1)

첫 화면에서는 :trd_time 변수에 사용자가 입력한 거래일자 또는 거래일시를 바인딩한다. 사용자가 ‘다음(▶)’ 버튼을 눌렀을 때는 ‘이전’ 페이지에서 출력한 마지막 거래일시를 입력한다.

ORDER BY 절이 사용됐음에도 실행계획에 소트 연산이 전혀 발생하지 않음을 확인하기 바란다. COUNT(STOPKEY)는 [종목코드 + 거래일시] 순으로 정렬된 인덱스를 스캔하다가 11번째 레코드에서 멈추게 됨을 의미한다. 사용자가 ‘이전(◀)’ 버튼을 클릭했을 때는 아래 SQL을 사용하며, :trd_time 변수에는 이전 페이지에서 출력한 첫 번째 거래일시를 바인딩하면 된다.

SELECT 거래일시, 체결건수, 체결수량, 거래대금 FROM ( SELECT 거래일시, 체결건수, 체결수량, 거래대금 FROM 시간별종목거래 A WHERE :페이지이동 = 'PREV' AND 종목코드 = :isu_cd AND 거래일시 <= :trd_time ORDER BY 거래일시 DESC ) WHERE ROWNUM <= 11 ORDER BY 거래일시 Execution Plan ------------------------------------------------------------- 0 SELECT STATEMENT Optimizer=ALL_ROWS (Cost=1 Card=1 Bytes=49) 1 0 SORT (ORDER BY) (Cost=1 Card=1 Bytes=49) 2 1 COUNT (STOPKEY) 3 2 VIEW (Cost=5 Card=1 Bytes=49) 4 3 FILTER 5 4 TABLE ACCESS (BY INDEX ROWID) OF '시간별종목거래' (TABLE) (Card=1 Bytes=56) 6 5 INDEX (RANGE SCAN DESCENDING) OF '시간별종목거래_PK' (INDEX (UNIQUE)) (Card=1)

여기서는 ‘SORT (ORDER BY)’가 나타났지만, ‘COUNT (STOPKEY)’ 바깥 쪽에 위치했으므로 조건절에 의해 선택된 11건에 대해서만 소트 연산을 수행한다. 인덱스를 거꾸로 읽었지만 화면에는 오름차순으로 출력되게 하려고 ORDER BY를 한번 더 사용한 것이다. 옵티마이저 힌트를 사용하면 SQL을 더 간단하게 구사할 수 있지만 인덱스 구성이 변경될 때 결과가 틀려질 위험성이 있다. 될 수 있으면 힌트를 이용하지 않고도 같은 방식으로 처리되도록 SQL을 조정하는 것이 바람직하다. SQL Server에선 Top N 구문을 이용해 아래와 같이 작성하면 된다.

< 첫 화면이거나, '다음(▶)' 버튼을 클릭했을 때 > SELECT TOP 11 거래일시, 체결건수, 체결수량, 거래대금 FROM 시간별종목거래 A WHERE :페이지이동 = 'NEXT' AND 종목코드 = :isu_cd AND 거래일시 >= :trd_time ORDER BY 거래일시 ; < '이전(◀)' 버튼을 클릭했을 때 > SELECT 거래일시, 체결건수, 체결수량, 거래대금 FROM ( SELECT TOP 11 거래일시, 체결건수, 체결수량, 거래대금 FROM 시간별종목거래 A WHERE :페이지이동 = 'PREV' AND 종목코드 = :isu_cd AND 거래일시 <= :trd_time ORDER BY 거래일시 DESC ) ORDER BY 거래일시 ;

### **다. Union All 활용**

방금 설명한 방식은 사용자가 어떤 버튼(조회, 다음, 이전)을 눌렀는지에 따라 별도의 SQL을 호출하는 방식이다. Union All을 활용하면 아래와 같이 하나의 SQL로 처리하는 것도 가능하다.

SELECT 거래일시, 체결건수, 체결수량, 거래대금 FROM ( SELECT 거래일시, 체결건수, 체결수량, 거래대금 FROM 시간별종목거래 WHERE :페이지이동 = 'NEXT' -- 첫 페이지 출력 시에도 'NEXT' 입력 AND 종목코드 = :isu_cd AND 거래일시 >= :trd_time ORDER BY 거래일시 ) WHERE ROWNUM <= 11 UNION ALL SELECT 거래일시, 체결건수, 체결수량, 거래대금 FROM ( SELECT 거래일시, 체결건수, 체결수량, 거래대금 FROM 시간별종목거래 WHERE :페이지이동 = 'PREV' AND 종목코드 = :isu_cd AND 거래일시 <= :trd_time ORDER BY 거래일시 DESC ) WHERE ROWNUM <= 11 ORDER BY 거래일시

### **5. 윈도우 함수 활용**

초기 RDBMS에서는 행(Row) 간 연산을 할 수 없다는 제약 때문에 복잡한 업무를 집합적으로 처리하는 데 한계가 많았다. 이 때문에 앞서 소개한 데이터 복제 기법을 이용해 SQL을 복잡하고 길게 작성해야 했고, 이마저도 어려울 땐 절차적 방식으로 프로그래밍 하곤 했다. 물론 지금도 행 간 연산을 지원하지 않지만 윈도우 함수(Window Function)가 도입되면서 복잡한 SQL을 어느 정도 단순화할 수 있게 되었다. Oracle에 의해 처음 소개된 윈도우 함수(Oracle에서는 ‘분석 함수(Analytic Function)’라고 함)가 지금은 ANSI 표준으로 채택돼 대부분 DBMS에서 지원하고 있다. 분석함수에 대해서는 2과목에서 이미 설명하였으므로 여기서는 이를 활용한 사례를 간단히 살펴보기로 하자. [그림 Ⅲ-5-4] 좌측처럼 장비측정 결과를 저장하는 테이블이 있다. 일련번호를 1씩 증가시키면서 측정값을 입력하고, 상태코드는 장비상태가 바뀔 때만 저장한다.

[![](https://dataonair.or.kr/publishing/img/knowledge/SQL_503.jpg)](https://dataonair.or.kr/publishing/img/knowledge/SQL_503.jpg)

그런데 장비측정 결과를 조회할 땐, 사용자가 [그림 Ⅲ-5-4] 우측과 같이 출력해 주길 원한다. 즉, 상태코드가 NULL이면 가장 최근에 상태코드가 바뀐 레코드의 값을 보여주는 식이다. 이를 구현하기 위해 가장 쉽게 생각할 수 있는 방법은 다음과 같다.

select 일련번호, 측정값 ,(select max(상태코드) from 장비측정 where 일련번호 <= o.일련번호 and 상태코드 is not null) 상태코드 from 장비측정 o order by 일련번호

위 쿼리가 빠르게 수행되려면 최소한 일련번호에 인덱스가 있어야 하고, [일련번호 + 상태코드]로 구성된 인덱스가 있으면 가장 최적이다. 좀 더 빠르게 수행되도록 아래와 같이 작성하는 것도 고려해 볼 수 있다.

select 일련번호, 측정값 ,(select /*+ index_desc(장비측정 장비측정_idx) */ 상태코드 from 장비측정 where 일련번호 <= o.일련번호 and 상태코드 is not null and rownum <= 1) 상태코드 from 장비측정 o order by 일련번호

부분범위처리 방식으로 앞쪽 일부만 보다가 멈춘다면 위 쿼리가 가장 최적이다. 만약 전체결과를 다 읽어야 한다면(예를 들어, 파일로 다운로드) 어떻게 쿼리하는 것이 최적일까? 여러 가지 방법을 생각해 볼 수 있지만, 아래와 같이 윈도우 함수를 이용하면 가장 쉽다.

select 일련번호, 측정값 , last_value(상태코드 ignore nulls) over(order by 일련번호 rows between unbounded preceding and current row) 상태코드 from 장비측정 order by 일련번호

### **6. With 구문 활용**

With 구문을 Oracle은 9i 버전부터, SQL Server는 2005 버전부터 지원하기 시작했다. With 절을 처리하는 DBMS 내부 실행 방식에는 아래 2가지가 있다.

- Materialize 방식 : 내부적으로 임시 테이블을 생성함으로써 반복 재사용
- Inline 방식 : 물리적으로 임시 테이블을 생성하지 않으며, 참조된 횟수만큼 런타임 시 반복 수행. SQL문에서 반복적으로 참조되는 집합을 미리 선언함으로써 코딩을 단순화하는 용도(인라인 뷰와는, 메인 쿼리에서 여러 번 참조가 가능하다는 점에서 다름)

Oracle은 위 2가지 방식을 모두 지원하지만, SQL Server는 Inline 방식으로만 실행된다. Oracle의 경우 실행방식을 상황에 따라 옵티마이저가 결정하며, 필요하다면 사용자가 힌트(materialize, inline)로써 지정할 수도 있다. Materialize 방식의 With절을 통해 생성된 임시 데이터는 영구적인 오브젝트가 아니어서, With절을 선언한 SQL문이 실행되는 동안만 유지된다. With절을 2개 이상 선언할 수 있으며, With절 내에서 다른 With절을 참조할 수도 있다. 배치 프로그램에서 특정 데이터 집합을 반복적으로 사용하거나, 전체 처리 흐름을 단순화시킬 목적으로 임시 테이블을 자주 활용하곤 하는데, Materialize 방식의 With 절을 이용하면 명시적으로 오브젝트를 생성하지 않고도 같은 처리를 할 수 있다. 아래는 With 절을 이용해 대용량 데이터를 빠르게 처리한 튜닝 사례다. 고객 테이블에는 2천만 건 이상, 카드 테이블에는 1억t>

with 위험고객카드 as ( select 카드.카드번호객여부 = 'Y' and 고객.고객번호 = 카드발급.고객번호 ) select v.* from ( select a.카드번호 as 카드번호 , sum(a.거래금액) as 거래금액 , null as 현금서비스잔액 , null as 해외거래금액 from 카드거래내역 a , 위험고객카드 b where 조건 group by a.카드번호 union all select a.카드번호 as 카드번호 , null as 현금서비스잔액 , sum(amt) as 현금서비스금액 , null as 해외거래금액 from ( select a.카드번호 as 카드번호 , sum(a.거래금액) as amt from 현금거래내역 a , 위험고객카드 b where 조건 group by a.카드번호 union all select a.카드번호 as 카드번호 , sum(a.결재금액) * -1 as amt from 현금결재내역 a , 위험고객카드 b where 조건 group by a.카드번호 ) a group by a.카드번호 union all select a.카드번호 as 카드번호 , null as 현금서비스잔액 , null as 현금서비스금액 , sum(a.거래금액) as 해외거래금액 from 해외거래내역 a , 위험고객카드 b where 조건 group by a.카드번호 ) v Execution Plan ------------------------------------------------------------- TEMP TABLE TRANSFORMATION → 임시테이블 생성 LOAD AS SELECT VIEW (Cost=94K Card=5K Bytes=345K) UNION-ALL SORT (GROUP BY) (Cost=57K Card=1 Bytes=120) HASH JOIN (Cost=57K Card=1 Bytes=120) PARTITION RANGE (SINGLE) PARTITION HASH (ALL) TABLE ACCESS (FULL) OF '카드거래내역' VIEW (Cost=50 Card=833K Bytes=13M) TABLE ACCESS (FULL) OF 'SYS.SYS_TEMP_0FD9D6B4E_4C0C42BA' → 임시 테이블 사용 SORT (GROUP BY) (Cost=36K Card=746 Bytes=20K) VIEW (Cost=36K Card=746 Bytes=20K) UNION-ALL SORT (GROUP BY) (Cost=34K Card=1 Bytes=74) HASH JOIN (Cost=34K Card=1 Bytes=74) PARTITION RANGE (ITERATOR) PARTITION HASH (ALL) TABLE ACCESS (FULL) OF '현금거래내역' (Cost=34K Card=1 Bytes=58) VIEW (Cost=50 Card=833K Bytes=13M) TABLE ACCESS (FULL) OF 'SYS.SYS_TEMP_0FD9D6B4E_4C0C42BA' → 임시 테이블 사용 SORT (GROUP BY) (Cost=2K Card=745 Bytes=38K) HASH JOIN (Cost=2K Card=746 Bytes=38K) …

고객 테이블은 2천만 건이 넘고, 카드 테이블은 1억 건이 넘지만 위험고객여부 = ‘Y’ 조건을 만족하는 위험고객카드는 그리 큰 집합이 아니다. 만약 materialize 방식의 With절을 이용할 수 없다면 아래쪽 메인 쿼리에서 위험고객카드 집합을 얻기 위해 매번 고객과 카드 테이블을 반복해서 읽어야 하고, 그것이 성능상 문제가 된다면 임시 테이블을 물리적으로 미리 생성해 두는 수밖에 없다.