### **1. 집계 함수(Aggregate Function)**

여러 행들의 그룹이 모여서 그룹당 단 하나의 결과를 돌려주는 다중행 함수 중 집계 함수(Aggregate Function)의 특성은 다음과 같다.

- 여러 행들의 그룹이 모여서 그룹당 단 하나의 결과를 돌려주는 함수이다. - GROUP BY 절은 행들을 소그룹화 한다. - SELECT 절, HAVING 절, ORDER BY 절에 사용할 수 있다.

ANSI/ISO에서 데이터 분석 기능으로 분류한 함수 중 기본적인 집계 함수는 본 절에서 설명하고, ROLLUP, CUBE, GROUPING SETS 같은 GROUP 함수는 2장 5절에서, 다양한 분석 기능을 가진 WINDOW 함수는 2장 6절에서 설명한다.

집계 함수명 ( [DISTINCT | ALL] 칼럼이나 표현식 ) - ALL : Default 옵션이므로 생략 가능함 - DISTINCT : 같은 값을 하나의 데이터로 간주할 때 사용하는 옵션임

자주 사용되는 주요 집계 함수들은 다음과 같다. 집계 함수는 그룹에 대한 정보를 제공하므로 주로 숫자 유형에 사용되지만, MAX, MIN, COUNT 함수는 문자, 날짜 유형에도 적용이 가능한 함수이다.

[![](https://dataonair.or.kr/publishing/img/knowledge/SQL_193.jpg)](https://dataonair.or.kr/publishing/img/knowledge/SQL_193.jpg)

[예제] 일반적으로 집계 함수는 GROUP BY 절과 같이 사용되지만 아래와 같이 테이블 전체가 하나의 그룹이 되는 경우에는 GROUP BY 절 없이 단독으로도 사용 가능하다.

[예제] SELECT COUNT(*) "전체 행수", COUNT(HEIGHT) "키 건수", MAX(HEIGHT) 최대키, MIN(HEIGHT) 최소키, ROUND(AVG(HEIGHT),2) 평균키 FROM PLAYER;

[실행 결과] 전체 행수 키 건수 최대키 최소키 평균키 ------ ----- ---- ---- ----- 480 447 196 165 179.31 1개의 행이 선택되었다.

실행 결과를 보면 COUNT(HEIGHT)는 NULL값이 아닌 키(HEIGHT) 칼럼의 건수만 출력하므로 COUNT(*)의 480보다 작은 것을 볼 수 있다. 그 이유는 COUNT(*) 함수에 사용된 와일드카드(*)는 전체 칼럼을 뜻하는데 전체 칼럼이 NULL인 행은 존재할 수 없기 때문에 결국 COUNT(*)는 전체 행의 개수를 출력한 것이고, COUNT(HEIGHT)는 HEIGHT 칼럼 값이 NULL인 33건은 제외된 건수의 합이다.

### **2. GROUP BY 절**

WHERE 절을 통해 조건에 맞는 데이터를 조회했지만 테이블에 1차적으로 존재하는 데이터 이외의 정보, 예를 들면 각 팀별로 선수가 몇 명인지, 선수들의 평균 신장과 몸무게가 얼마나 되는지, 또는 각 팀에서 가장 큰 키의 선수가 누구인지 등의 2차 가공 정보도 필요하다. GROUP BY 절은 SQL 문에서 FROM 절과 WHERE 절 뒤에 오며, 데이터들을 작은 그룹으로 분류하여 소그룹에 대한 항목별로 통계 정보를 얻을 때 추가로 사용된다.

SELECT [DISTINCT] 칼럼명 [ALIAS명] FROM 테이블명 [WHERE 조건식] [GROUP BY 칼럼(Column)이나 표현식] [HAVING 그룹조건식] ;

GROUP BY 절과 HAVING 절은 다음과 같은 특성을 가진다.

- GROUP BY 절을 통해 소그룹별 기준을 정한 후, SELECT 절에 집계 함수를 사용한다. - 집계 함수의 통계 정보는 NULL 값을 가진 행을 제외하고 수행한다. - GROUP BY 절에서는 SELECT 절과는 달리 ALIAS 명을 사용할 수 없다. - 집계 함수는 WHERE 절에는 올 수 없다. (집계 함수를 사용할 수 있는 GROUP BY 절보다 WHERE 절이 먼저 수행된다) - WHERE 절은 전체 데이터를 GROUP으로 나누기 전에 행들을 미리 제거시킨다. - HAVING 절은 GROUP BY 절의 기준 항목이나 소그룹의 집계 함수를 이용한 조건을 표시할 수 있다. - GROUP BY 절에 의한 소그룹별로 만들어진 집계 데이터 중, HAVING 절에서 제한 조건을 두어 조건을 만족하는 내용만 출력한다. - HAVING 절은 일반적으로 GROUP BY 절 뒤에 위치한다.

일부 데이터베이스의 과거 버전에서 데이터베이스가 GROUP BY 절에 명시된 칼럼의 순서대로 오름차순 정렬을 자동으로 실시(비공식적인 지원이었음)하는 경우가 있었으나, 원칙적으로 관계형 데이터베이스 환경에서는 뒤에서 언급할 ORDER BY 절을 명시해야 데이터 정렬이 수행된다. ANSI/ISO 기준에서도 데이터 정렬에 대한 내용은 ORDER BY 절에서만 언급되어있지, GROUP BY 절에는 언급되어 있지 않다.

[예제] K-리그 선수들의 포지션별 평균키는 어떻게 되는가란 요구 사항을 접수하였다. GROUP BY 절을 사용하지 않고 집계 함수를 사용했을 때 어떤 결과를 보이는지 포지션별 평균키를 구해본다.

[예제 및 실행 결과] SELECT POSITION 포지션, AVG(HEIGHT) 평균키 FROM PLAYER; SELECT POSITION 포지션, AVG(HEIGHT) 평균키 * 1행에 오류: ERROR: 단일 그룹의 집계 함수가 아니다.

GROUP BY 절에서 그룹 단위를 표시해 주어야 SELECT 절에서 그룹 단위의 칼럼과 집계 함수를 사용할 수 있다. 그렇지 않으면 예제와 같이 에러를 발생하게 된다.

[예제] SELECT 절에서 사용된 포지션이라는 한글 ALIAS를 GROUP BY 절의 기준으로 사용해본다.

[예제 및 실행 결과] SELECT POSITION 포지션, AVG(HEIGHT) 평균키 FROM PLAYER GROUP BY POSITION 포지션; GROUP BY POSITION 포지션 * 3행에 오류: ERROR: SQL 명령어가 올바르게 종료되지 않았다.

실행 결과를 살펴보면 GROUP BY 절에 “포지션”이라고 표시된 부분에 에러가 발생했다는 것을 알 수 있다. 칼럼에 대한 ALIAS는 SELECT 절에서 정의하고 ORDER BY 절에서는 재활용할 수 있지만, GROUP BY 절에서는 ALIAS 명을 사용할 수 없다는 것을 보여 주는 사례이다.

[예제] 포지션별 최대키, 최소키, 평균키를 출력한다. (포지션별이란 소그룹의 조건을 제시하였기 때문에 GROUP BY 절을 사용한다.)

[예제] SELECT POSITION 포지션, COUNT(*) 인원수, COUNT(HEIGHT) 키대상, MAX(HEIGHT) 최대키, MIN(HEIGHT) 최소키, ROUND(AVG(HEIGHT),2) 평균키 FROM PLAYER GROUP BY POSITION;

[실행 결과] 포지션 인원수 키대상 최대키 43 43 196 174 186.26 DF 172 142 190 170 180.21 FW 100 100 194 168 179.91 MF 162 162 189 165 176.31 5개의 행이 선택되었다.

실행 결과를 보면 포지션별로 평균키 외에도 인원수, 키대상 인원수, 최대키, 최소키가 제대로 출력된 것을 확인할 수 있다. ORDER BY 절이 없기 때문에 포지션 별로 정렬은 되지 않았다. 추가로 포지션과 키 정보가 없는 선수가 3명이라는 정보를 얻을 수 있으며, 포지션이 DF인 172명 중 30명은 키에 대한 정보가 없는 것도 알 수 있다. GK, DF, FW, MF의 최대키, 최소키, 평균키를 구할 때 키 값이 NULL인 경우는 계산 대상에서 제외된다. 즉, 포지션 DF의 최대키, 최소키, 평균키 결과는 키 값이 NULL인 30명을 제외한 142명을 대상으로 수행한 통계 결과이다.

### **3. HAVING 절**

[예제] K-리그 선수들의 포지션별 평균키를 구하는데, 평균키가 180 센티미터 이상인 정보만 표시하라는 요구 사항이 접수되었으므로 WHERE 절과 GROUP BY 절을 사용해 SQL 문장을 작성한다.

[예제 및 실행 결과] SELECT POSITION 포지션, ROUND(AVG(HEIGHT),2) 평균키 FROM PLAYER WHERE AVG(HEIGHT) >= 180 GROUP BY POSITION; WHERE AVG(HEIGHT) >= 180 * 3행에 오류: ERROR: 집계 함수는 허가되지 않는다.

실행 결과에서 WHERE 절의 집계 함수 AVG(HEIGHT) 부분에서 “집계 함수는 허가되지 않는다”는 에러 메시지가 출력되었다. 즉, WHERE 절에는 AVG()라는 집계 함수는 사용할 수 없다. WHERE 절은 FROM 절에 정의된 집합(주로 테이블)의 개별 행에 WHERE 절의 조건절이 먼저 적용되고, WHERE 절의 조건에 맞는 행이 GROUP BY 절의 대상이 된다. 그런 다음 결과 집합의 행에 HAVING 조건절이 적용된다. 결과적으로 HAVING 절의 조건을 만족하는 내용만 출력된다. 즉, HAVING 절은 WHERE 절과 비슷하지만 그룹을 나타내는 결과 집합의 행에 조건이 적용된다는 점에서 차이가 있다.

[예제] HAVING 조건절에는 GROUP BY 절에서 정의한 소그룹의 집계 함수를 이용한 조건을 표시할 수 있으므로, HAVING 절을 이용해 평균키가 180 센티미터 이상인 정보만 표시한다.

[예제] SELECT POSITION 포지션, ROUND(AVG(HEIGHT),2) 평균키 FROM PLAYER GROUP BY POSITION HAVING AVG(HEIGHT) >= 180;

[예제] 포지션 평균키 ------ ------ GK 186.26 DF 180.21 2개의 행이 선택되었다.

실행 결과에서 전체 4개 포지션 중에서 평균 키가 180cm가 넘는 2개의 데이터만 출력된 것을 확인할 수 있다.

[예제] SQL 문장은 GROUP BY 절과 HAVING 절의 순서를 바꾸어서 수행한다.

[예제] SELECT POSITION 포지션, AVG(HEIGHT) 평균키 FROM PLAYER HAVING AVG(HEIGHT) >= 180 GROUP BY POSITION;

[실행 결과] 포지션 평균키 ----- ---- GK 186.26 DF 180.21 2개의 행이 선택되었다.

GROUP BY 절과 HAVING 절의 순서를 바꾸어서 수행하더라도 문법 에러도 없고 결과물도 동일한 결과를 출력한다. 그렇지만, SQL 내용을 보면, 포지션이란 소그룹으로 그룹핑(GROUPING)되어 통계 정보가 만들어지고, 이후 적용된 결과 값에 대한 HAVING 절의 제한 조건에 맞는 데이터만을 출력하는 것이므로 논리적으로 GROUP BY 절과 HAVING 절의 순서를 지키는 것을 권고한다.

[예제] K-리그의 선수들 중 삼성블루윙즈(K02)와 FC서울(K09)의 인원수는 얼마인가란 요구 사항이 접수되었다. WHERE 절과 GROUP BY 절을 사용한 SQL과 GROUP BY 절과 HAVING 절을 사용한 SQL을 모두 작성한다.

[예제 및 실행 결과] SELECT TEAM_ID 팀ID, COUNT(*) 인원수 FROM PLAYER WHERE TEAM_ID IN ('K09', 'K02') GROUP BY TEAM_ID; 팀ID 인원수 ---- ----- K02 49 K09 49 2개의 행이 선택되었다.

[예제 및 실행 결과] SELECT TEAM_ID 팀ID, COUNT(*) 인원수 FROM PLAYER GROUP BY TEAM_ID HAVING TEAM_ID IN ('K09', 'K02'); 팀ID 인원수 ----- ----- K02 49 K09 49 2개의 행이 선택되었다.

GROUP BY 소그룹의 데이터 중 일부만 필요한 경우, GROUP BY 연산 전 WHERE 절에서 조건을 적용하여 필요한 데이터만 추출하여 GROUP BY 연산을 하는 방법과, GROUP BY 연산 후 HAVING 절에서 필요한 데이터만 필터링 하는 두 가지 방법을 사용할 수 있다. 같은 실행 결과를 얻는 두 가지 방법 중 HAVING 절에서 TEAM_ID 같은 GROUP BY 기준 칼럼에 대한 조건을 추가할 수도 있으나, 가능하면 WHERE 절에서 조건절을 적용하여 GROUP BY의 계산 대상을 줄이는 것이 효율적인 자원 사용 측면에서 바람직하다.

[예제] 포지션별 평균키만 출력하는데, 최대키가 190cm 이상인 선수를 가지고 있는 포지션의 정보만 출력한다.

[예제] SELECT POSITION 포지션, ROUND(AVG(HEIGHT),2) 평균키 FROM PLAYER GROUP BY POSITION HAVING MAX(HEIGHT) >= 190;

[실행 결과] 포지션 평균키 ------ ----- GK 186.26 DF 180.21 FW 179.91 3개의 행이 선택되었다.

SQL을 보면 SELECT 절에서 사용하지 않는 MAX 집계 함수를 HAVING 절에서 조건절로 사용한 사례이다. 즉, HAVING 절은 SELECT 절에 사용되지 않은 칼럼이나 집계 함수가 아니더라도 GROUP BY 절의 기준 항목이나 소그룹의 집계 함수를 이용한 조건을 표시할 수 있다. 이 부분은 1장 8절의 SELECT 문장의 실행 순서에서 추가 설명한다. 여기서 주의할 것은 WHERE 절의 조건 변경은 대상 데이터의 개수가 변경되므로 결과 데이터 값이 변경될 수 있지만, HAVING 절의 조건 변경은 결과 데이터 변경은 없고 출력되는 레코드의 개수만 변경될 수 있다. 실행 결과를 보면 다른 결과 값의 변경 없이 MAX(HEIGHT)가 189cm로 190cm 미만인 MF 포지션의 데이터만 HAVING 조건에 의해 누락된 것을 확인할 수 있다. (다른 포포지션의 통계 정보는 다음과 같ss=bg_gray>포지션 인원수 키대상 최대키 최소키 평균키 ----- ----- ----- ----- ---- ---- MF 162 162 189 165 176.31

### **4. CASE 표현을 활용한 월별 데이터 집계**

“집계 함수(CASE( ))~GROUP BY” 기능은, 모델링의 제1정규화로 인해 반복되는 칼럼의 경우 구분 칼럼을 두고 여러 개의 레코드로 만들어진 집합을, 정해진 칼럼 수만큼 확장해서 집계 보고서를 만드는 유용한 기법이다. 부서별로 월별 입사자의 평균 급여를 알고 싶다는 고객의 요구사항이 있는데, 입사 후 1년마다 급여 인상이나 보너스 지급과 같은 일정이 정기적으로 잡힌다면 업무적으로 중요한 정보가 될 수 있다.

STEP1. 개별 데이터 확인

[예제] 먼저 개별 입사정보에서 월별 데이터를 추출하는 작업을 진행한다. 이 단계는 월별 정보가 있다면 생략 가능하다.

[예제] Oracle SELECT ENAME, DEPTNO, EXTRACT(MONTH FROM HIREDATE) 입사월, SAL FROM EMP;

[예제] SQL Server SELECT ENAME, DEPTNO, DATEPART(MONTH, HIREDATE) 입사월, SAL FROM EMP;

[예제] SQL Server SELECT ENAME, DEPTNO, MONTH(HIREDATE) 입사월, SAL FROM EMP;

[실행 결과] ENAME DEPTNO 입사월 SAL ------- ------- ------ ----- SMITH 20 12 800 ALLEN 30 2 1600 WARD 30 2 1250 JONES 20 4 2975 MARTIN 30 9 1250 BLAKE 30 5 2850 CLARK 10 6 2450 SCOTT 20 7 3000 KING 10 11 5000 TURNER 30 9 1500 ADAMS 20 7 1100 JAMES 30 12 950 FORD 20 12 3000 MILLER 10 1 1300 14개의 행이 선택되었다.

STEP2. 월별 데이터 구분

[예제] 추출된 MONTH 데이터를 Simple Case Expression을 이용해서 12개의 월별 칼럼으로 구분한다. 실행 결과에서 보여 주는 ENAME 칼럼은 최종 리포트에서 요구되는 데이터는 아니지만, 정보의 흐름을 이해하기 위해 부가적으로 보여 주는 임시 정보이다. FROM 절에서 사용된 인라인 뷰는 2장 4절에서 설명한다.

[예제] SELECT ENAME, DEPTNO, CASE MONTH WHEN 1 THEN SAL END M01, CASE MONTH WHEN 2 THEN SAL END M02, CASE MONTH WHEN 3 THEN SAL END M03, CASE MONTH WHEN 4 THEN SAL END M04, CASE MONTH WHEN 5 THEN SAL END M05, CASE MONTH WHEN 6 THEN SAL END M06, CASE MONTH WHEN 7 THEN SAL END M07, CASE MONTH WHEN 8 THEN SAL END M08, CASE MONTH WHEN 9 THEN SAL END M09, CASE MONTH WHEN 10 THEN SAL END M10, CASE MONTH WHEN 11 THEN SAL END M11, CASE MONTH WHEN 12 THEN SAL END M12 FROM (SELECT ENAME, DEPTNO, EXTRACT(MONTH FROM HIREDATE) MONTH, SAL FROM EMP);

[실행 결과] ENAME DEPTNO M01 M02 M03 M04 M05 M06 M07 M08 M09 M10 M11 M12 ------ ----- --- --- --- --- --- --- --- --- --- --- --- --- SMITH 20 　　　　　　　　　　　 800 ALLEN 30 　 1600 　　　　　　　　　　 WARD 30 　 1250 　　　　　　　　　　 JONES 20 　　　 2975 　　　　　　　　 MARTIN 30 　　　　　　　　 1250 　　　 BLAKE 30 　　　　 2850 　　　　　　　 CLARK 10 　　　　　 2450 　　　　　　 SCOTT 20 　　　　　　 3000 　　　　　 KING 10 　　　　　　　　　　 5000 　 TURNER 30 　　　　　　　　 1500 　　　 ADAMS 20 　　　　　　 1100 　　　　　 JAMES 30 　　　　　　　　　　　 950 FORD 20 　　　　　　　　　　　 3000 MILLER 10 1300 　　　　　　　　　　　 14개의 행이 선택되었다.

STEP3. 부서별 데이터 집계

[예제] 최종적으로 보여주는 리포트는 부서별로 월별 입사자의 평균 급여를 알고 싶다는 요구사항이므로 부서별 평균값을 구하기 위해 GROUP BY 절과 AVG 집계 함수를 사용한다. 직원 개인에 대한 정보는 더 이상 필요 없으므로 제외한다. ORDER BY 절을 사용하지 않았기 때문에 부서번호별로 정렬이 되지는 않았다.

[예제] SELECT DEPTNO, AVG(CASE MONTH WHEN 1 THEN SAL END) M01, AVG(CASE MONTH WHEN 2 THEN SAL END) M02, AVG(CASE MONTH WHEN 3 THEN SAL END) M03, AVG(CASE MONTH WHEN 4 THEN SAL END) M04, AVG(CASE MONTH WHEN 5 THEN SAL END) M05, AVG(CASE MONTH WHEN 6 THEN SAL END) M06, AVG(CASE MONTH WHEN 7 THEN SAL END) M07, AVG(CASE MONTH WHEN 8 THEN SAL END) M08, AVG(CASE MONTH WHEN 9 THEN SAL END) M09, AVG(CASE MONTH WHEN 10 THEN SAL END) M10, AVG(CASE MONTH WHEN 11 THEN SAL END) M11, AVG(CASE MONTH WHEN 12 THEN SAL END) M12 FROM (SELECT ENAME, DEPTNO, EXTRACT(MONTH FROM HIREDATE) MONTH, SAL FROM EMP) GROUP BY DEPTNO ;

[실행 결과] DEPTNO M01 M02 M03 M04 M05 M06 M07 M08 M09 M10 M11 M12 ------ --- --- --- --- --- --- --- --- --- --- --- --- 30 　 1425 　　 2850 　　　 1375 　　 950 20 　　　 2975 　　 2050 　　　　 1900 10 1300 　　　　 2450 　　　　 5000 　 3개의 행이 선택되었다.

하나의 데이터에 여러 번 CASE 표현을 사용하고 집계 함수가 적용되므로 SQL 처리 성능 측면에서 나쁜 것이 아니냐는 생각을 할 수도 있다. 그렇지만, 같은 기능을 하는 리포트를 작성하기 위해?을 활용하면 복잡한 프로그램이 아닌 하나의 SQL 문장으로 처리 가능하므로 DBMS 자원 활용이나 처리 속도에서 훨씬 효율?? 차이는 더 크질 수 있다. 개발자들은 가능한 하나의 SQL 문장으로 비즈니스적인 요구 사항을 처리할 수 있도록 노력해야 한다.

[예제] Simple Case Expression으로 표현된 위의 SQL과 같은 내용으로 Oracle의 DECODE 함수를 사용한 SQL 문장을 작성한다.

[예제] SELECT DEPTNO, AVG(DECODE(MONTH, 1,SAL)) M01, AVG(DECODE(MONTH, 2,SAL)) M02, AVG(DECODE(MONTH, 3,SAL)) M03, AVG(DECODE(MONTH, 4,SAL)) M04, AVG(DECODE(MONTH, 5,SAL)) M05, AVG(DECODE(MONTH, 6,SAL)) M06, AVG(DECODE(MONTH, 7,SAL)) M07, AVG(DECODE(MONTH, 8,SAL)) M08, AVG(DECODE(MONTH, 9,SAL)) M09, AVG(DECODE(MONTH,10,SAL)) M10, AVG(DECODE(MONTH,11,SAL)) M11, AVG(DECODE(MONTH,12,SAL)) M12 FROM (SELECT ENAME, DEPTNO, EXTRACT(MONTH FROM HIREDATE) MONTH, SAL FROM EMP) GROUP BY DEPTNO ;

DECODE 함수를 사용함으로써 SQL 문장이 조금 더 짧아졌다. CASE 표현과 Oracle의 DECODE 함수는 표현상 서로 장단점이 있으므로 어떤 기능을 선택할 지는 사용자의 몫이다.

### **5. 집계 함수와 NULL**

리포트의 빈칸을 NULL이 아닌 ZERO로 표현하기 위해 NVL(Oracle)/ISNULL(SQL Server) 함수를 사용하는 경우가 많은데, 다중 행 함수를 사용하는 경우는 오히려 불필요한 부하가 발생하므로 굳이 NVL 함수를 다중 행 함수 안에 사용할 필요가 없다. 다중 행 함수는 입력 값으로 전체 건수가 NULL 값인 경우만 함수의 결과가 NULL이 나오고 전체 건수 중에서 일부만 NULL인 경우는 NULL인 행을 다중 행 함수의 대상에서 제외한다. 예를 들면 100명 중 10명의 성적이 NULL 값일 때 평균을 구하는 다중 행 함수 AVG를 사용하면 NULL 값이 아닌 90명의 성적에 대해서 평균값을 구하게 된다. CASE 표현 사용시 ELSE 절을 생략하게 되면 Default 값이 NULL이다. NULL은 연산의 대상이 아닌 반면, SUM(CASE MONTH WHEN 1 THEN SAL ELSE 0 END)처럼 ELSE 절에서 0(Zero)을 지정하면 불필요하게 0이 SUM 연산에 사용되므로 자원의 사용이 많아진다. 같은 결과를 얻을 수 있다면 가능한 ELSE 절의 상수값을 지정하지 않거나 ELSE 절을 작성하지 않도록 한다. 같은 이유로 Oracle의 DECODE 함수는 4번째 인자를 지정하지 않으면 NULL이 Default로 할당된다. 많이 실수하는 것 중에 하나가 Oracle의 SUM(NVL(SAL,0)), SQL Server의 SUM(ISNULL (SAL,0)) 연산이다. 개별 데이터의 급여(SAL)가 NULL인 경우는 NULL의 특성으로 자동적으로 SUM 연산에서 빠지는 데, 불필요하게 NVL/ISNULL 함수를 사용해 0(Zero)으로 변환시켜 데이터 건수만큼의 연산이 일어나게 하는 것은 시스템의 자원을 낭비하는 일이다. 리포트 출력 때 NULL이 아닌 0을 표시하고 싶은 경우에는 NVL(SUM(SAL),0)이나, ISNULL(SUM(SAL),0)처럼 전체 SUM의 결과가 NULL인 경우(대상 건수가 모두 NULL인 경우)에만 한 번 NVL/ISNULL 함수를 사용하면 된다.

[예제] 팀별 포지션별 FW, MF, DF, GK 포지션의 인원수와 팀별 전체 인원수를 구하는 SQL 문장을 작성한다. 데이터가 없는 경우는 0으로 표시한다.

[예제] SIMPLE_CASE_EXPRESSION 조건 - Oracle SELECT TEAM_ID, NVL(SUM(CASE POSITION WHEN 'FW' THEN 1 ELSE 0 END),0) FW, NVL(SUM(CASE POSITION WHEN 'MF' THEN 1 ELSE 0 END),0) MF, NVL(SUM(CASE POSITION WHEN 'DF' THEN 1 ELSE 0 END),0) DF, NVL(SUM(CASE POSITION WHEN 'GK' THEN 1 ELSE 0 END),0) GK, COUNT(*) SUM FROM PLAYER GROUP BY TEAM_ID;

[예제] SIMPLE_CASE_EXPRESSION 조건 - Oracle CASE 표현의 ELSE 0, ELSE NULL 문구는 생략 가능하므로 다음과 같이 조금 더 짧게 SQL 문장을 작성할 수 있다. Default 값인 NULL이 적용됨. SELECT TEAM_ID, NVL(SUM(CASE POSITION WHEN 'FW' THEN 1 END),0) FW, NVL(SUM(CASE POSITION WHEN 'MF' THEN 1 END),0) MF, NVL(SUM(CASE POSITION WHEN 'DF' THEN 1 END),0) DF, NVL(SUM(CASE POSITION WHEN 'GK' THEN 1 END),0) GK, COUNT(*) SUM FROM PLAYER GROUP BY TEAM_ID;

[예제] SEARCHED_CASE_EXPRESSION 조건 - Oracle SELECT TEAM_ID, NVL(SUM(CASE WHEN POSITION = 'FW' THEN 1 END), 0) FW, NVL(SUM(CASE WHEN POSITION = 'MF' THEN 1 END), 0) MF, NVL(SUM(CASE WHEN POSITION = 'DF' THEN 1 END), 0) DF, NVL(SUM(CASE WHEN POSITION = 'GK' THEN 1 END), 0) GK, COUNT(*) SUM FROM PLAYER GROUP BY TEAM_ID;

[예제] SEARCHED_CASE_EXPRESSION 조건 - SQL Server SELECT TEAM_ID, ISNULL(SUM(CASE WHEN POSITION = 'FW' THEN 1 END), 0) FW, ISNULL(SUM(CASE WHEN POSITION = 'MF' THEN 1 END), 0) MF, ISNULL(SUM(CASE WHEN POSITION = 'DF' THEN 1 END), 0) DF, ISNULL(SUM(CASE WHEN POSITION = 'GK' THEN 1 END), 0) GK, COUNT(*) SUM FROM PLAYER GROUP BY TEAM_ID;

[실행 결과] TEAM_ID FW MF DF GK SUM ------ --- --- --- --- --- K14 0 1 1 0 2 K06 11 11 20 4 46 K13 1 0 1 1 3 K15 1 1 1 0 3 K02 10 18 17 4 49 K12 1 0 1 0 2 K04 13 11 18 4 46 K03 6 15 23 5 49 K07 9 22 16 4 51 K05 10 19 17 5 51 K08 8 15 15 4 45 K11 1 1 1 0 3 K01 12 15 13 5 45 K10 5 15 13 3 36 K09 12 18 15 4 49 15개의 행이 선택되었다.

4개의 예제 SQL 문장은 같은 실행 결과를 출력한다. ORDER BY 절이 적용?다. TEAM_ID 'K08'의 경우 POSITION이 NULL인 3건이 포지션별 분류에는 빠져 있지만 COUNT(*) SUM에는 추가되어 있다.

[예제] GROUP BY 절 없이 전체 선수들의 포지션별 평균 키 및 전체 평균 키를 출력할 수도 있다.

[예제] SELECT ROUND(AVG(CASE WHEN POSITION = 'MF' THEN HEIGHT END),2) 미드필더, ROUND(AVG(CASE WHEN POSITION = 'FW' THEN HEIGHT END),2) 포워드, ROUND(AVG(CASE WHEN POSITION = 'DF' THEN HEIGHT END),2) 디펜더, ROUND(AVG(CASE WHEN POSITION = 'GK' THEN HEIGHT END),2) 골키퍼, ROUND(AVG(HEIGHT),2) 전체평균키 FROM PLAYER;

[실행 결과] 미드필더 포워드 디펜더 골키퍼 전체평균키 ----- ----- ----- ----- -------- 176.31 179.91 180.21 186.26 179.31 1개의 행이 선택되었다.