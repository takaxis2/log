### **1. 인덱스 유지 비용**

테이블 데이터를 변경하면 관련된 인덱스에도 변경이 발생한다. 변경할 인덱스 레코드를 찾아가는 비용에 Redo, Undo를 생성하는 비용까지 더해지므로 인덱스 개수가 많을수록 DML 성능이 나빠지는 것은 당연하다. Update를 수행할 때, 테이블 레코드는 직접 변경하지만 인덱스 레코드는 Delete & Insert 방식으로 처리된다. 인덱스는 항상 정렬된 상태를 유지해야 하기 때문이며, 인덱스 유지를 위한 Undo 레코드도 2개씩 기록된다. 따라서 변경 칼럼과 관련된 인덱스 개수에 따라 Update 성능이 좌우된다. Insert나 Delete 문일 때는 인덱스 모두에(Oracle에서는 인덱스 칼럼이 모두 Null인 경우는 제외) 변경을 가해야 하므로 총 인덱스 개수에 따라 성능이 크게 달라진다. 이처럼 인덱스 개수가 DML 성능에 큰 영향을 미치므로 대량의 데이터를 입력/수정/삭제할 때는 인덱스를 모두 Drop하거나 Unusable 상태로 변경한 다음에 작업하는 것이 빠를 수 있다. 인덱스를 재생성하는 시간까지 포함하더라도 그냥 작업할 때보다 더 빠를 수 있기 때문이다.

### **2. Insert 튜닝**

### **가. Oracle Insert 튜닝**

Insert 속도를 향상시키는 방법에 대해 Oracle부터 살펴보자.

- Direct Path Insert

IOT(index-organized table)는 정해진 키(Key) 순으로 정렬하면서 값을 입력하는 반면, 일반적인 힙 구조 테이블(heap-organized table)은 순서 없이 Freelist로부터 할당받은 블록에 무작위로 값을 입력한다. Freelist는 HWM(High-Water Mark) 아래쪽에 위치한 블록 중 어느 정도(테이블에 지정한 pctfree와 pctused 파라미터에 의해 결정됨) 빈 공간을 가진 블록 리스트를 관리하는 자료구조다. Freelist에서 할당받은 블록을 버퍼 캐시에서 찾아보고, 없으면 데이터 파일에서 읽어 캐시에 적재한 후에 데이터를 삽입한다. 일반적인 트랜잭션을 처리할 때는 빈 공간부터 찾아 채워 나가는 위 방식이 효율적이다. 하지만, 대량의 데이터를 Bulk로 입력할 때는 매우 비효율적이다. 빈 블록은 얼마 지나지 않아 모두 채워지고 이후부터는 순차적으로 뒤쪽에만 데이터를 쌓게 될 텐데도 건건이 Freelist를 조회하면서 입력하기 때문이다. Freelist를 거치지 않고 HWM 바깥 영역에, 그것도 버퍼 캐시를 거치지 않고 데이터 파일에 곧바로 입력하는 Direct Path Insert 방식을 사용하면 대용량 Insert 속도를 크게 향상시킬 수 있다. 이 방식을 사용할 때 Undo 데이터를 쌓지 않는 점도 속도 향상의 주요인이다. 사용자가 커밋할 때만 HWM를 상향 조정하면 되기 때문에 Undo 데이터가 불필요하다. 아래는 Oracle에서 Direct Path Insert 방식으로 데이터를 입력하는 방법이다.

- insert select 문장에 /*+ append */ 힌트 사용
- 병렬 모드로 insert
- direct 옵션을 지정하고 SQL*Loader(sqlldr)로 데이터를 로드
- CTAS(create table … as select) 문장을 수행
- nologging 모드 Insert

Oracle에서 아래와 같이 테이블 속성을 nologging으로 바꿔주면 Redo 로그까지 최소화(데이터 딕셔너리 변경사항만 로깅)되므로 더 빠르게 insert 할 수 있다. 이 기능은 Direct Path Insert 일 때만 작동하며, 일반 insert문을 로깅하지 않도록 하는 방법은 없다.

alter table t NOLOGGING;

주의할 것은, Direct Path Insert 방식으로 데이터를 입력하면 Exclusive 모드 테이블 Lock이 걸린다는 사실이다. 즉, 작업이 수행되는 동안 다른 트랜잭션은 해당 테이블에 DML을 수행하지 못하게 된다. 따라서 트랜잭션이 빈번한 주간에 이 옵션을 사용하는 것은 절대 금물이다. nologging 상태에서 입력한 데이터는 장애가 발생했을 때 복구가 불가능하다는 사실도 반드시 기억하기 바란다. 이 옵션을 사용해 데이터를 insert한 후에는 곧바로 백업을 실시해야 한다. 또는 언제든 재생 가능한 데이터를 insert할 때만 사용해야 한다. 예를 들면, 배치 프로그램에서 중간 단계의 임시 테이블을 만들 때가 대표적이다. DW 시스템에 읽기 전용 데이터를 적재할 때도 유용하다. DW성 데이터는 OLTP로부터 언제든 재현해 낼 수 있기 때문이다. 물론 가용성 요건과 운영 환경이 시스템마다 다르므로 상황에 맞게 적용하기 바란다.

### **나. SQL Server Insert 튜닝**

- 최소 로깅(minimal nologging

SQL Server에서 최소 로깅 기능을 사용하려면, 우선 해당 데이터베이스의 복구 모델(Recovery model)이 ‘Bulk-logged’ 또는 ‘Simple’로 설정돼 있어야 한다.

alter database SQLPRO set recovery SIMPLE

첫 번째로, 아래와 같이 파일 데이터를 읽어 DB로 로딩하는 Bulk Insert 구문을 사용할 때, With 옵션에 TABLOCK 힌트를 추가하면 최소 로깅 모드로 작동한다.

BULK INSERT AdventureWorks.Sales.SalesOrderDetail FROM 'C:\orders\lineitem.txt' WITH ( DATAFILETYPE = 'CHAR', FIELDTERMINATOR = ' |', ROWTERMINATOR = ' |\n', TABLOCK )

두 번째로, Oracle CTAS와 같은 문장이 select into 인데, 복구 모델이 ‘Bulk-logged’ 또는 ‘Simple’로 설정된 상태에서 이 문장을 사용하면 최소 로깅 모드로 작동한다.

select * into target from source ;

세 번째로, SQL Server 2008 버전부터 최소 로깅 기능을 일반 Insert문에서 활용할 수 있게 되었다. 힙(Heap) 테이블에 Insert할 땐 아래와 같이 간단히 TABLOCK 힌트를 사용하면 된다. 이때, X 테이블 Lock 때문에 여러 트랜잭션이 동시에 Insert 할 수 없게 된다는 사실을 기억하기 바란다.

insert into t_heap with (TABLOCK) select * from t_source

B*Tree 구조 테이블(클러스터형 인덱스)에 Insert 할 때도 최소 로깅이 가능한데, 가장 기본적인 전제 조건은 소스 데이터를 목표 테이블 정렬(클러스터형 인덱스 정렬?? 데이터베이스의 복구 모델(Recovery model)은 ‘Bulk-logged’ 또는 ‘Simple’로 설정돼 있어야 한다. 최소 로깅을 위해 필요한 다른 조건은 다음과 같다.

- 비어있는 B*Tree 구조에서 TABLOCK 힌트 사용
- 비어있는 B*Tree 구조에서 TF-610을 활성화
- 비어 있지 않은 B*Tree 구조에서 TF-610을 활성화하고, 새로운 키 범위만 입력

위 조건에서 보듯, B*Tree 구조 테이블에 최소 로깅 모드로 Insert 할 때는 TABLOCK 힌트가 반드시 필요하지 않다. 따라서 입력하는 소스 데이터의 값 범위가 중복되지 않는다면 동시 Insert도 가능하다. 아래는 B*Tree 구조 테이블에 최소 로깅 모드로 Insert하는 예시다. 목표 테이블 정렬 순서와 같게 하려고 order by 절을 사용한 것을 확인하기 바란다.

use SQLPRO go alter database SQLPRO set recovery SIMPLE DBCC TRACEON(610); insert into t_idx select * from t_source order by col1 → t_idx 테이블의 클러스터형 인덱스 키 순 정렬

SQL Server에서도 최소 로깅 작업을 수행한 다음에는 차등(Differential) 백업을 수행해 줘야 한다는 사실을 기억하자.

### **3. Update 튜닝**

### **가. Truncate & Insert 방식 사용**

아래는 1999년 12월 31일 이전 주문 데이터의 상태코드를 모두 변경하는 Update문이다.

update 주문 set 상태코드 = '9999' where 주문일시 < to_date('20000101', 'yyyymmdd')

대량의 데이터를 위와 같이 일반 Update문으로 갱신하면 상당히 오랜 시간이 소요될 수 있다. 다음과 같은 이유 때문이며, Delete문일 때도 마찬가지다.

- 테이블 데이터를 갱신하는 본연의 작업
- 인덱스 데이터까지 갱신
- 버퍼 캐시에 없는 블록를 디스크에서 읽어 버퍼 캐시에 적재한 후에 갱신
- 내부적으로 Redo와 Undo 정보 생성
- 블록에 빈 공간이 없으면 새 블록 할당(→ Row Migration 발생)

따라서 대량의 데이터를 갱신할 때는 Update문을 이용하기보다 아래와 같이 처리하는 것이 더 빠를 수 있다.

- - Oracle create table 주문_임시 as select * from 주문; -- SQL Server select * into \#emp_temp from emp; alter table emp drop constraint 주문_pk; drop index [주문.]주문_idx1; → [] : SQL Server truncate table 주문; insert into 주문(고객번호, 주문일시, , 상태코드) select 고객번호, 주문일시, ,(case when 주문일시 >= to_date('20000101', 'yyyymmdd') then '9999' else status end) 상태코드 from 주문_임시; alter table 주문 add constraint 주문_pk primary key(고객번호, 주문일시); create index 주문_idx1 on 주문(주문일시, 상태코드);

인덱스가 하나도 없는 상태에서 테스트해 봐도 대략 20% 수준에서 손익분기점이 결정되는 것을 알 수 있고, 만약 인덱스까지 여러 개 있다면 손익분기점은 더 낮아진다. Oracle의 경우 위 CTAS 문장에 nologging 옵션을 사용하고서 Insert 문장에 append 힌트까지 사용하면 손익분기점은 더 낮아진다. 아래는 1999년 12월 31일 이전 주문 데이터의 상태코드를 모두 지우는 Delete문이다.

delete from 주문 where 주문일시 < to_date('20000101', 'yyyymmdd')

대량의 데이터를 Delete 할 때도 아래와 같이 처리하는 것이 빠를 수 있다.

create table 주문_임시 as select * from 주문 where 주문일시 >= to_date('20000101', 'yyyymmdd'); alter table emp drop constraint 주문_pk; drop index 주문_idx1; truncate table 주문; insert into 주문 select * from 주문_임시; alter table 주문 add constraint 주문_pk primary key(고객번호, 주문일시); create index 주문_idx1 on 주문(주문일시, 상태코드);

### **나. 조인을 내포한 Update 튜닝**

조인을 내포한 Update 문을 수행할 때는 Update 자체의 성능보다 조인 과정에서 발생하는 비효율 때문에 성능이 느려지는 경우가 더 많다. 그 원인과 튜닝 방안에 대해 살펴보자.

- 전통적인 방식의 Update문

다른 테이블과 조인을 통해 Update를 수행할 때, 아래와 같이 일반적인 Update문을 사용하면 비효율이 발생한다. Update를 위해 참조하는 테이블을 2번 액세스해야 하기 때문이다.

update 고객 set (최종거래일시, 최근거래금액) = ( select max(거래일시), sum(거래금액) from 거래 where 고객번호 = 고객.고객번호 and 거래일시 >= trunc(add_months(sysdate,-1)) ) where exisis ( select 'x' from 거래 where 고객번호 = 고객.고객번호 and 거래일시 >= trunc(add_months(sysdate,-1) ) );

위 Update를 위해서는 기본적으로 거래 테이블에 [고객번호+거래일시] 인덱스가 있어야 한다. 인덱스가 그렇게 구성돼 있어도 고객 수가 많고 고객별 거래 데이터가 많다면 위 쿼리는 결코 빠르게 수행될 수 없는데, Random 액세스 방식으로 조인을 수행하기 때문이다. 그럴 때는 서브쿼리에 unnest와 함께 hash_sj 힌트를 사용해 해시 세미 조인(Semi Join) 방식으로 유도하는 것이 효과적이다. 해시 세미 조인 방식으로 수행하면 Random 액세스는 상당히 줄일 수 있지만 거래 테이블을 2번 액세스하는 비효율은 여전히 남는다. 이 문제를 해결하기 위한 확장 Update 문장이 DBMS마다 조금씩 다른 형태로 제공되는데, 지금부터 이에 대해 살펴보자.

- SQL Server 확장 Update문 활용

SQL Server에서는 아래와 같은 확장 Update문을 활용함으로써 방금 설명한 비효율을 쉽게 제거할 수 있다.

update 고객 set 최종거래일시 = b.거래일시, 최근거래금액 = b.거래금액 from 고객 a inner join ( select 고객번호, max(거래일시) 거래일시, sum(거래금액) 거래금액 from 거래 where 거래일시 >= dateadd(mm,-1,convert(datetime,convert(char(8),getdate(),112),112)) group by 고객번호 ) b on a.고객번호 = b.고객번호

- Oracle 수정 가능 조인 뷰 활용

Oracle에서는 아래와 같이 수정 가능 조인 뷰(Updatable Join View)를 활용할 수 있다.

update /*+ bypass_ujvc */ (select c.최종거래일시, c.최근거래금액, t.거래일시, t.거래금액 from (select 고객번호, max(거래일시) 거래일시, sum(거래금액) 거래금액 from 거래 where 거래일시 >= trunc(add_months(sysdate,-1)) group by 고객번호) t , 고객 c where c.고객번호 = t.고객번호 ) set 최종거래일시 = 거래일시 , 최근거래금액 = 거래금액

위 update 문에 사용된 bypass_ujvc 힌트에 대해서는 설명이 필요하다. ‘조인 뷰’는 from절에 두 개 이상 테이블을 가진 뷰를 말하며, 조인 뷰를 통해 원본 테이블에 입력, 수정, 삭제가 가능하다. 여기에 한가지 제약사항이 있는데, 키-보존 테이블에만 입력, 수정, 삭제가 허용된다는 사실이다. 키-보존 테이블(Key-Preserved Table)이란, 조인된 결과집합을 통해서도 중복 없이 Unique하게 식별이 가능한 테이블을 말한다. 이를 위해선 Unique한 집합과 조인되어야 하는데, 옵티마이저는 조인되는 테이블에 Unique 인덱스가 있는지를 보고 Unique 집합 여부를 판단한다. 결국, Unique 인덱스가 없는 테이블과 조인된 테이블에는 입력, 수정, 삭제가 허용되지 않는다. 방금 본 Update문이 제대로 수행되려면 고객 테이블이 키-보존 테이블이어야 한다. 그런데 거래 데이터를 집계한 인라인 뷰에 Unique 인덱스가 존재할 수 없으므로 Oracle은 고객 테이블을 키-보존 테이블로 인정하지 않는다. 고객번호로 group by한 집합의 고객번호에 중복 값이 있을 수 없다는 사실을 옵티마이저도 충분히 인지할 수 있는데도 말이다. 집합적으로 Unique성이 보장됨에도 불구하고 Unique 인덱스를 찾을 수 없다는 이유로 옵티마이저가 필요 이상의 제약을 가한 셈인데, 다행히 이를 피해갈 수 있는 bypass_ujvc 힌트가 제공된다. 참고로, 이 힌트는 ‘Bypass Updatable Join View Check’를 축약해 만든 것이다. 이 힌트는 Update를 위해 참조하는 집합에 중복 레코드가 없음이 100% 보장될 때만 사용할 것을 당부한다. 10g부터는 바로 이어서 설명할 Merge Into 구문을 활용하는 것이 바람직하다.

- Oracle Merge문 활용

merge into문을 이용하면 하나의 SQL 안에서 insert, update, delete 작업을 한번에 처리할 수 있다. 이 기능은 Oracle 9i부터 제공되기 시작했고, delete 작업까지 처리할 수 있게 된 것은 10g부터다. SQL Server도 2008 버전부터 이 문장을 지원하기 시작했다. merge into는 기간계 시스템으로부터 읽어온 신규 및 변경분 데이터를 DW 시스템에 반영하고자 할 때 사용하면 효과적이다. 아래는 merge문을 이용해 insert, update를 동시에 처리하는 예시다.

merge into 고객 t using 고객변경분 s on (t.고객번호 = s.고객번호) when matched then update set t.고객번호 = s.고객번호, t.고객명 = s.고객명, t.이메일 = s.이메일, when not matched then insert (고객번호, 고객명, 이메일, 전화번호, 거주지역, 주소, 등록일시) values (s.고객번호, s.고객명, s.이메일, s.전화번호, s.거주지역, s.주소, s.등록일시);

Oracle 10g부터는 아래와 같이 update와 insert를 선택적으로 처리할 수 있다.

merge into 고객 t using 고객변경분 s on (t.고객번호 = s.고객번호) when matched then update set t.고객번호 = s.고객번호, t.고객명 = s.고객명, t.이메일 = s.이메일, ; merge into 고객 t using 고객변경분 s on (t.고객번호 = s.고객번호) when not matched then insert (고객번호, 고객명, 이메일, 전화번호, 거주지역, 주소, 등록일시) values (s.고객번호, s.고객명, s.이메일, s.전화번호, s.거주지역, s.주소, s.등록일시);

이 확장 기능을 통해 Updatable Join View 기능을 대체할 수 있게 되었다. 앞에서 bypass_ujvc 힌트를 사용했던 update 문장을 예로 들면, 아래와 같이 merge문으로 처리를 할 수 있게 되었다.

merge into 고객 c using (select 고객번호, max(거래일시) 거래일시, sum(거래금액) 거래금액 from 거래 where 거래일시 >= trunc(add_months(sysdate,-1)) group by 고객번호) t on (c.고객번호 = t.고객번호) when matched then update set c.최종거래일시 = t.거래일시, c.최근거래금액 = t.거래금액