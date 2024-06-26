### **1. ORDER BY 정렬**

ORDER BY 절은 SQL 문장으로 조회된 데이터들을 다양한 목적에 맞게 특정 칼럼을 기준으로 정렬하여 출력하는데 사용한다. ORDER BY 절에 칼럼(Column)명 대신에 SELECT 절에서 사용한 ALIAS 명이나 칼럼 순서를 나타내는 정수도 사용 가능하다. 그리고 별도로 정렬 방식을 지정하지 않으면 기본적으로 오름차순이 적용되며, SQL 문장의 제일 마지막에 위치한다.

SELECT 칼럼명 [ALIAS명] FROM 테이블명 [WHERE 조건식] [GROUP BY 칼럼(Column)이나 표현식] [HAVING 그룹조건식] [ORDER BY 칼럼(Column)이나 표현식 [ASC 또는 DESC]] ; ASC(Ascending) : 조회한 데이터를 오름차순으로 정렬한다.(기본 값이므로 생략 가능) DESC(Descending) : 조회한 데이터를 내림차순으로 정렬한다.

[예제] ORDER BY 절의 예로 선수 테이블에서 선수들의 이름, 포지션, 백넘버를 출력하는데 사람 이름을 내림차순으로 정렬하여 출력한다.

[예제] SELECT PLAYER_NAME 선수명, POSITION 포지션, BACK_NO 백넘버 FROM PLAYER ORDER BY PLAYER_NAME DESC;

[실행 결과] 선수명 포지션 백넘버 ----- ----- ---- 히카르도 MF 10 황철민 MF 35 황연석 FW 16 황승주 DF 98 홍종하 MF 32 홍인기 DF 35 홍성요 DF 28 홍복표 FW 19 홍명보 DF 20 홍도표 MF 9 홍광철 DF 4 호제리오 DF 3 480개의 행이 선택되었다.

[예제] ORDER BY 절의 예로 선수 테이블에서 선수들의 이름, 포지션, 백넘버를 출력하는데 선수들의 포지션 내림차순으로 출력한다. 칼럼명이 아닌 ALIAS를 이용한다.

[예제] SELECT PLAYER_NAME 선수명, POSITION 포지션, BACK_NO 백넘버 FROM PLAYER ORDER BY 포지션 DESC;

[실행 결과] Oracle 선수명 포지션 백넘버 키 ------ ------ ----- --- 정학범 173 차상광 186 안익수 174 백영철 MF 22 173 조태용 MF 7 192 올리베 MF 29 190 김리네 MF 26 188 쟈스민 MF 33 186 480개의 행이 선택되었다.

실행 결과에서 포지션에 아무 것도 없는 값들이 있다. 현재 선수 테이블에서 포지션 칼럼에 NULL이 들어 있는데 포지션의 내림차순에서 NULL 값이 앞에 출력되었다는 것은 Oracle이 NULL 값을 가장 큰 값으로 취급했다는 것을 알 수 있다. 반면 SQL Server는 반대의 정렬 순서를 가진다. ORDER BY 절 사용 특징은 아래와 같다.

- 기본적인 정렬 순서는 오름차순(ASC)이다. - 숫자형 데이터 타입은 오름차순으로 정렬했을 경우에 가장 작은 값부터 출력된다. - 날짜형 데이터 타입은 오름차순으로 정렬했을 경우 날짜 값이 가장 빠른 값이 먼저 출력된다. 예를 들어 ‘01-JAN-2012’는 ‘01-SEP-2012’보다 먼저 출력된다. - Oracle에서는 NULL 값을 가장 큰 값으로 간주하여 오름차순으로 정렬했을 경우에는 가장 마지막에, 내림차순으로 정렬했을 경우에는 가장 먼저 위치한다. - 반면, SQL Server에서는 NULL 값을 가장 작은 값으로 간주하여 오름차순으로 정렬했을 경우에는 가장 먼저, 내림차순으로 정렬했을 경우에는 가장 마지막에 위치한다.

[예제] 한 개의 칼럼이 아닌 여러 가지 칼럼(Column)을 기준으로 정렬해본다. 먼저 키가 큰 순서대로, 키가 같은 경우 백넘버 순으로 ORDER BY 절을 적용하여 SQL 문장을 작성하는데, 키가 NULL인 데이터는 제외한다.

[예제] SELECT PLAYER_NAME 선수이름, POSITION 포지션, BACK_NO 백넘버, HEIGHT 키 FROM PLAYER WHERE HEIGHT IS NOT NULL ORDER BY HEIGHT DESC, BACK_NO;

[실행 결과] 선수명 포지션 백넘버 키 ----- ------ ----- --- 서동명 GK 21 196 권정혁 GK 1 195 김석 FW 20 194 정경두 GK 41 194 이현 GK 1 192 황연석 FW 16 192 미트로 FW 19 192 김대희 GK 31 192 조의손 GK 44 192 김창민 GK 1 191 우성용 FW 22 191 최동석 GK 1 190 샤샤 FW 10 190 447개의 행이 선택되었다.

실행 결과를 보면 키가 192cm인 선수가 5명 있는데, ORDER BY 절에서 키가 큰 순서대로 출력하고, 키가 같으면 백넘버 순으로 정렬하라는 조건에 따라서 백넘버 순으로 정렬되어 있는 것을 확인할 수 있다. 칼럼명이나 ALIAS 명을 대신해서 SELECT 절의 칼럼 순서를 정수로 매핑하여 사용할 수도 있다. SELECT 절의 칼럼명이 길거나 정렬 조건이 많을 경우 편리하게 사용할 수 있으나 향후 유지보수성이나 가독성이 떨어지므로 가능한 칼럼명이나 ALIAS 명을 권고한다. ORDER BY 절에서 칼럼명, ALIAS명, 칼럼 순서를 같이 혼용하는 것도 가능하다.

[예제] ORDER BY 절의 예로 선수 테이블에서 선수들의 이름, 포지션, 백넘버를 출력하는데 선수들의 백넘버 내림차순, 백넘버가 같은 경우 포지션, 포지션까지 같은 경우 선수명 순서로 출력한다. BACK_NO가 NULL인 경우는 제외하고, 칼럼명이나 ALIAS가 아닌 칼럼 순서를 매핑하여 사용한다.

[예제] SELECT PLAYER_NAME 선수명, POSITION 포지션, BACK_NO 백넘버 FROM PLAYER WHERE BACK_NO IS NOT NULL ORDER BY 3 DESC, 2, 1;

[실행 결과] 선수명 포지션 백넘버 ------ ---- ----- 뚜따 FW 99 쿠키 FW 99 황승주 DF 98 무스타파 MF 77 다보 FW 63 다오 DF 61 김충호 GK 60 최동우 GK 60 최주호 GK 51 안동원 DF 49 오재진 DF 49 ... ... .. 439개의 행이 선택되었다.

[예제] DEPT 테이블 정보를 부서명, 지역, 부서번호 내림차순으로 정렬해서 출력한다. 아래의 SQL 문장은 출력되는 칼럼 레이블은 다를 수 있지만 결과는 모두 같다.

Case1. 칼럼명 사용 ORDER BY 절 사용

[예제] SELECT DNAME, LOC, DEPTNO FROM DEPT ORDER BY DNAME, LOC, DEPTNO DESC;

[실행 결과] DNAME LOC DEPTNO ----------- --------- ------ ACCOUNTING NEW YORK 10 OPERATIONS BOSTON 40 RESEARCH DALLAS 20 SALES CHICAGO 30 4개의 행이 선택되었다.

Case2. 칼럼명 + ALIAS 명 사용 ORDER BY 절 사용

[예제] SELECT DNAME DEPT, LOC AREA, DEPTNO FROM DEPT ORDER BY DNAME, AREA, DEPTNO DESC;

[실행 결과] DEPT AREA DEPTNO ---------- --------- ------ ACCOUNTING NEW YORK 10 OPERATIONS BOSTON 40 RESEARCH DALLAS 20 SALES CHICAGO 30 4개의 행이 선택되었다.

Case3. 칼럼 순서번호 + ALIAS 명 사용 ORDER BY 절 사용

[예제] SELECT DNAME, LOC AREA, DEPTNO FROM DEPT ORDER BY 1, AREA, 3 DESC;

[실행 결과] DNAME AREA DEPTNO ----------- ---------- ------ ACCOUNTING NEW YORK 10 OPERATIONS BOSTON 40 RESEARCH DALLAS 20 SALES CHICAGO 30 4개의 행이 선택되었다.

### **2. SELECT 문장 실행 순서**

GROUP BY 절과 ORDER BY가 같이 사용될 때 SELECT 문장은 6개의 절로 구성이 되고, SELECT 문장의 수행 단계는 아래와 같다.

5. SELECT 칼럼명 [ALIAS명] 1. FROM 테이블명 2. WHERE 조건식 3. GROUP BY 칼럼(Column)이나 표현식 4. HAVING 그룹조건식 6. ORDER BY 칼럼(Column)이나 표현식;

1. 발췌 대상 테이블을 참조한다. (FROM) 2. 발췌 대상 데이터가 아닌 것은 제거한다. (WHERE) 3. 행들을 소그룹화 한다. (GROUP BY) 4. 그룹핑된 값의 조건에 맞는 것만을 출력한다. (HAVING) 5. 데이터 값을 출력/계산한다. (SELECT) 6. 데이터를 정렬한다. (ORDER BY)

위 순서는 옵티마이저가 SQL 문장의 SYNTAX, SEMANTIC 에러를 점검하는 순서이기도 하다. 예를 들면 FROM 절에 정의되지 않은 테이블의 칼럼을 WHERE 절, GROUP BY 절, HAVING 절, SELECT 절, ORDER BY 절에서 사용하면 에러가 발생한다. 그러나 ORDER BY 절에는 SELECT 목록에 나타나지 않은 문자형 항목이 포함될 수 있다. 단, SELECT DISTINCT를 지정하거나 SQL 문장에 GROUP BY 절이 있거나 또는 SELECT 문에 UNION 연산자가 있으면 열 정의가 SELECT 목록에 표시되어야 한다. 이 부분은 관계형 데이터베이스가 데이터를 메모리에 올릴 때 행 단위로 모든 칼럼을 가져오게 되므로, SELECT 절에서 일부 칼럼만 선택하더라도 ORDER BY 절에서 메모리에 올라와 있는 다른 칼럼의 데이터를 사용할 수 있다.

[예제] SELECT 절에 없는 EMP 칼럼을 ORDER BY 절에 사용한다.

[예제] SELECT EMPNO, ENAME FROM EMP ORDER BY MGR;

[실행 결과] EMPNO ENAME ----- ------- 7902 FORD 7788 SCOTT 7900 JAMES 7499 ALLEN 7521 WARD 7844 TURNER 7654 MARTIN 7934 MILLER 7876 ADAMS 7698 BLAKE 7566 JONES 7782 CLARK 7369 SMITH 7839 KING 14개의 행이 선택되었다.

실행 결과에서 ORDER BY 절에서 SELECT 절에서 정의하지 않은 칼럼을 사용해도 문제없음을 확인할 수 있다.

[예제] 인라인 뷰에 정의된 SELECT 칼럼을 메인쿼리에서 사용한다.

[예제 및 실행 결과] SELECT EMPNO FROM (SELECT EMPNO, ENAME FROM EMP ORDER BY MGR); 14개의 행이 선택되었다.

실행 결과에서 2장에서 배울 인라인 뷰의 SELECT 절에서 정의한 칼럼은 메인쿼리에서도 사용할 수 있는 것을 확인할 수 있다.

[예제] 인라인 뷰에 미정의된 칼럼을 메인쿼리에서 사용해본다.

[예제 및 실행 결과] SELECT MGR FROM (SELECT EMPNO, ENAME FROM EMP ORDER BY MGR); SELECT MGR FROM ; * ERROR: "MGR": 부적합한 식별자

그러나 서브쿼리의 SELECT 절에서 선택되지 않은 칼럼들은 계속 유지되는 것이 아니라 서브쿼리 범위를 벗어나면 더 이상 사용할 수 없게 된다. (인라인 뷰도 동일함) GROUP BY 절에서 그룹핑 기준을 정의하게 되면 데이터베이스는 일반적인 SELECT 문장처럼 FROM 절에 정의된 테이블의 구조를 그대로 가지고 가는 것이 아니라, GROUP BY 절의 그룹핑 기준에 사용된 칼럼과 집계 함수에 사용될 수 있는 숫자형 데이터 칼럼들의 집합을 새로 만든다. GROUP BY 절을 사용하게 되면 그룹핑 기준에 사용된 칼럼과 집계 함수에 사용될 수 있는 숫자형 데이터 칼럼들의 집합을 새로 만드는데, 개별 데이터는 필요 없으므로 저장하지 않는다. GROUP BY 이후 수행 절인 SELECT 절이나 ORDER BY 절에서 개별 데이터를 사용하는 경우 에러가 발생한다. 결과적으로 SELECT 절에서는 그룹핑 기준과 숫자 형식 칼럼의 집계 함수를 사용할 수 있지만, 그룹핑 기준 외의 문자 형식 칼럼은 정할 수 없다.

[예제] GROUP BY 절 사용시 SELECT 절에 일반 칼럼을 사용해본다.

[예제 및 실행 결과] SELECT JOB, SAL FROM EMP GROUP BY JOB HAVING COUNT(*) > 0 ORDER BY SAL; SELECT JOB, SAL ; * ERROR: GROUP BY 표현식이 아니다.

[예제] GROUP BY 절 사용시 ORDER BY 절에 일반 칼럼을 사용해본다.

[예제 및 실행 결과] SELECT JOB FROM EMP GROUP BY JOB HAVING COUNT(*) > 0 ORDER BY SAL; ORDER BY SAL; * ERROR: GROUP BY 표현식이 아니다.

[예제] GROUP BY 절 사용시 ORDER BY 절에 집계 칼럼을 사용해본다.

[예제] SELECT JOB FROM EMP GROUP BY JOB HAVING COUNT(*) > 0 ORDER BY MAX(EMPNO), MAX(MGR), SUM(SAL), COUNT(DEPTNO), MAX(HIREDATE);

[실행 결과] JOB --------- MANAGER PRESIDENT SALESMAN ANALYST CLERK 5개의 행이 선택되었다.

SELECT SQL에서 GROUP BY 절이 사용되었기 때문에 SELECT 절에 정의하지 않은 MAX, SUM, COUNT 집계 함수도 ORDER BY 절에서 사용할 수 있는 것을 실행 결과에서 확인할 수 있다.

### **3. Top N 쿼리**

- ROWNUM

Oracle에서 순위가 높은 N개의 로우를 추출하기 위해 ORDER BY 절과 WHERE 절의 ROWNUM 조건을 같이 사용하는 경우가 있는데 이 두 조건으로는 원하는 결과를 얻을 수 없다. Oracle의 경우 정렬이 완료된 후 데이터의 일부가 출력되는 것이 아니라, 데이터의 일부가 먼저 추출된 후(ORDER BY 절은 결과 집합을 결정하는데 관여하지 않음) 데이터에 대한 정렬 작업이 일어나므로 주의해야 한다.

[예제] 사원 테이블에서 급여가 높은 3명만 내림차순으로 출력하고자 하는데, 잘못 사용된 SQL의 사례이다.

[예제] SELECT ENAME, SAL FROM EMP WHERE ROWNUM < 4 ORDER BY SAL DESC;

[실행 결과] ENAME SAL ------ ---- ALLEN 1600 WARD 1250 SMITH 800 3개의 행이 선택되었다.

실행 결과의 3명은 급여가 상위인 3명을 출력한 것이 아니라, 급여 순서에 상관없이 무작위로 추출된 3명에 한해서 급여를 내림차순으로 정렬한 결과이므로 원하는 결과를 출력한 것이 아니다.

[예제] ORDER BY 절이 없으면 ORACLE의 ROWNUM 조건과 SQL SERVER의 TOP 절은 같은 결과를 보인다. 그렇지만, ORDER BY 절이 사용되는 경우 ORACLE은 ROWNUM 조건을 ORDER BY 절보다 먼저 처리되는 WHERE 절에서 처리하므로, 정렬 후 원하는 데이터를 얻기 위해서는 2장 4절에서 배울 인라인 뷰에서 먼저 데이터 정렬을 수행한 후 메인쿼리에서 ROWNUM 조건을 사용해야 한다.

[예제] SELECT ENAME, SAL FROM (SELECT ENAME, SAL FROM EMP ORDER BY SAL DESC) WHERE ROWNUM < 4 ;

[실행 결과] ENAME SAL ------ ---- KING 5000 SCOTT 3000 FORD 3000 3개의 행이 선택되었다.

위 사례에서는 인라인 뷰를 사용하여 추출하고자 하는 접합을 정렬한 후 ROWNUM을 적용시킴으로써 결과에 참여하는 순서와 추출되는 로우 순서를 일치시킴으로써 Top N 쿼리의 결과를 만들어내었다. 실행 결과를 보면 EMP 테이블의 데이터를 급여가 많은 순서부터 정렬을 수행한 후 상위 3건의 데이터를 출력한 것을 알 수 있다. 추가로, 원하는 추출 결과와 동일한 순서로 정렬된 인덱스가 존재한다면 그 인덱스를 사용하여 동일한 결과를 얻을 수도 있다.

- TOP ( )

반면 SQL Server는 TOP 조건을 사용하게 되면 별도 처리 없이 관련 Order By 절의 데이터 정렬 후 원하는 일부 데이터만 쉽게 출력할 수 있다.

TOP (Expression) [PERCENT] [WITH TIES]

TOP 절을 사용하여 결과 집합으로 반환되는 행 수를 제한할 수 있다. WITH TIES 옵션은 ORDER BY 절의 조건 기준으로 TOP N의 마지막 행으로 표시되는 추가 행의 데이터가 같을 경우 N+ 동일 정렬 순서 데이터를 추가 반환하도록 지정하는 옵션이다.

[예제] 사원 테이블에서 급여가 높은 2명을 내림차순으로 출력하고자 한다.

[예제] SELECT TOP(2) ENAME, SAL FROM EMP ORDER BY SAL DESC;

[실행 결과] ENAME SAL ------ ---- KING 5000 SCOTT 3000 2개의 행이 선택되었다.

[예제] 사원 테이블에서 급여가 높은 2명을 내림차순으로 출력하는데 같은 급여를 받는 사원이 있으면 같이 출력한다.

[예제] SELECT TOP(2) WITH TIES ENAME, SAL FROM EMP ORDER BY SAL DESC;

[실행 결과] ENAME SAL ----- --- KING 5000 SCOTT 3000 FORD 3000 3개의 행이 선택되었다.

TOP(2) WITH TIES 옵션은 동일 수치의 데이터를 추가로 더 추출하는 것으로, SCOTT과 FORD의 급여가 공동 2위이므로 TOP(2) WITH TIES의 실행 결과는 3건의 데이터가 출력된다.