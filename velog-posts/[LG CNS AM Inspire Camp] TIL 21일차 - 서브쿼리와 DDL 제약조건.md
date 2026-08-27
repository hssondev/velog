<blockquote>
<p><strong>한 줄 요약</strong></p>
<p>쿼리 안에 쿼리를 넣는 서브쿼리(단일행/다중행, <code>ANY</code>/<code>ALL</code>)와 인라인 뷰, 상관관계 서브쿼리, <code>RANK() OVER</code>, <code>LIMIT</code>을 배우고, <code>CREATE TABLE</code>에 제약조건(PK/FK/NOT NULL/UNIQUE/CHECK/DEFAULT)을 걸어보며 DDL을 실습했다.</p>
</blockquote>
<hr />
<h1 id="📚-오늘-학습한-커리큘럼">📚 오늘 학습한 커리큘럼</h1>
<pre><code>서브쿼리란? — 위치별(SELECT/FROM/WHERE), 행수별(단일행/다중행) 분류
        ↓
단일행 서브쿼리 — 값 하나와 비교
        ↓
집계 함수를 중첩할 수 없는 문제 → 인라인 뷰(FROM 서브쿼리)로 해결
        ↓
다중행 서브쿼리 — IN, ANY, ALL
        ↓
상관관계 서브쿼리 — 바깥 쿼리 값을 안쪽 쿼리가 참조
        ↓
RANK() OVER — 윈도우 함수로 순위 매기기
        ↓
LIMIT / OFFSET — Top-N 조회
        ↓
DDL — CREATE TABLE과 제약조건(PK, FK, NOT NULL, UNIQUE, CHECK, DEFAULT)</code></pre><h1 id="📌-오늘의-학습-키워드">📌 오늘의 학습 키워드</h1>
<ul>
<li>스칼라 서브쿼리 / 인라인 뷰 / (일반) 서브쿼리</li>
<li>단일행 서브쿼리 vs 다중행 서브쿼리</li>
<li><code>IN</code>, <code>ANY</code>, <code>ALL</code></li>
<li>상관관계 서브쿼리(Correlated Subquery)</li>
<li><code>RANK() OVER (PARTITION BY ~ ORDER BY ~)</code></li>
<li><code>LIMIT</code>, <code>OFFSET</code></li>
<li><code>PRIMARY KEY</code>, <code>FOREIGN KEY</code>, <code>NOT NULL</code>, <code>UNIQUE</code>, <code>CHECK</code>, <code>DEFAULT</code></li>
</ul>
<hr />
<h1 id="0-개념-정리-키워드-사전">0. 개념 정리 (키워드 사전)</h1>
<blockquote>
<p><strong>한 줄 요약</strong></p>
<p>서브쿼리는 &quot;쿼리 안에 들어간 또 다른 쿼리&quot;이고, 어디에 위치하는지(SELECT/FROM/WHERE)와 결과가 몇 행인지(단일행/다중행)에 따라 쓸 수 있는 연산자가 달라진다.</p>
</blockquote>
<table>
<thead>
<tr>
<th>키워드</th>
<th>한 줄 정의</th>
<th>비고 / 예시</th>
</tr>
</thead>
<tbody><tr>
<td><strong>스칼라 서브쿼리</strong></td>
<td><code>SELECT</code> 절 안에 들어가는 서브쿼리, 결과가 <strong>값 하나</strong>여야 함</td>
<td></td>
</tr>
<tr>
<td><strong>인라인 뷰</strong></td>
<td><code>FROM</code> 절 안에 들어가는 서브쿼리, 마치 <strong>임시 테이블</strong>처럼 취급됨</td>
<td><code>FROM (SELECT ...) V</code></td>
</tr>
<tr>
<td><strong>(WHERE) 서브쿼리</strong></td>
<td><code>WHERE</code> 절 안에서 조건값으로 쓰이는 서브쿼리</td>
<td></td>
</tr>
<tr>
<td><strong>단일행 서브쿼리</strong></td>
<td>결과가 <strong>딱 한 행</strong>만 나오는 서브쿼리</td>
<td><code>=</code>, <code>&gt;</code>, <code>&lt;</code> 등 일반 비교연산자 사용 가능</td>
</tr>
<tr>
<td><strong>다중행 서브쿼리</strong></td>
<td>결과가 <strong>여러 행</strong>으로 나올 수 있는 서브쿼리</td>
<td><code>IN</code>, <code>ANY</code>, <code>ALL</code> 같은 전용 연산자 필요</td>
</tr>
<tr>
<td><strong><code>ANY</code></strong></td>
<td>서브쿼리 결과 중 <strong>하나라도</strong> 조건을 만족하면 참</td>
<td><code>&lt; ANY</code>는 &quot;가장 큰 값보다 작으면&quot; 참</td>
</tr>
<tr>
<td><strong><code>ALL</code></strong></td>
<td>서브쿼리 결과 <strong>전부</strong>가 조건을 만족해야 참</td>
<td><code>&lt; ALL</code>은 &quot;가장 작은 값보다도 작으면&quot; 참</td>
</tr>
<tr>
<td><strong>상관관계 서브쿼리</strong></td>
<td>서브쿼리 안에서 <strong>바깥 쿼리의 컬럼을 참조</strong>하는 서브쿼리</td>
<td>바깥 쿼리의 각 행마다 서브쿼리가 다시 실행됨</td>
</tr>
<tr>
<td><strong><code>RANK() OVER (...)</code></strong></td>
<td>그룹 내 순위를 매겨주는 윈도우 함수</td>
<td><code>PARTITION BY</code>로 그룹 나누고 <code>ORDER BY</code>로 정렬 기준 지정</td>
</tr>
<tr>
<td><strong><code>LIMIT N</code></strong></td>
<td>결과에서 상위 N개만 가져옴</td>
<td><code>LIMIT 3 OFFSET 0</code>처럼 시작 위치 지정도 가능</td>
</tr>
<tr>
<td><strong>제약조건(Constraint)</strong></td>
<td>테이블에 들어올 수 있는 데이터의 규칙을 미리 정해두는 것</td>
<td>PK, FK, NOT NULL, UNIQUE, CHECK</td>
</tr>
</tbody></table>
<hr />
<h1 id="1-서브쿼리의-기본--where-절에서-조건값으로-쓰기">1. 서브쿼리의 기본 — <code>WHERE</code> 절에서 조건값으로 쓰기</h1>
<pre><code class="language-sql">SELECT STUDENT_NAME
FROM tb_student S
WHERE ABSENCE_YN = 'Y'
AND DEPARTMENT_NO = (
    SELECT DEPARTMENT_NO
    FROM TB_DEPARTMENT
    WHERE DEPARTMENT_NAME = '국어국문학과'
)
AND SUBSTRING(S.STUDENT_SSN, 8, 1) = '2';</code></pre>
<ul>
<li>괄호 안의 <code>SELECT DEPARTMENT_NO ... WHERE DEPARTMENT_NAME = '국어국문학과'</code>가 서브쿼리다. 이 서브쿼리는 <strong>딱 한 개의 학과코드</strong>만 반환하므로(학과 이름은 유일하니까), 바깥 쿼리에서 <code>= (서브쿼리)</code>처럼 <strong>일반 비교연산자</strong>로 바로 쓸 수 있다 — 이게 <strong>단일행 서브쿼리</strong>다.</li>
<li>학과 이름을 코드로 미리 조회해서 변수처럼 써넣는 대신, 서브쿼리로 그 자리에서 바로 조회해 쓰는 방식이라 <strong>두 단계로 나눠 짤 필요가 없다</strong>는 게 서브쿼리의 장점이다.</li>
</ul>
<hr />
<h1 id="2-단일행-서브쿼리--같은-직급-더-높은-급여인-사람-찾기">2. 단일행 서브쿼리 — 같은 직급, 더 높은 급여인 사람 찾기</h1>
<pre><code class="language-sql">SELECT *
FROM employee
WHERE JOB_ID = (SELECT JOB_ID FROM employee WHERE EMP_NAME = '나승원')
AND SALARY &gt; (SELECT SALARY FROM employee WHERE EMP_NAME = '나승원');</code></pre>
<ul>
<li>서브쿼리 두 개를 각각 <code>=</code>와 <code>&gt;</code>로 비교했다. &quot;나승원의 직급&quot;과 &quot;나승원의 급여&quot;라는 두 개의 단일 값을 각각 별도의 서브쿼리로 구해서, 바깥 쿼리의 조건에 동시에 사용했다.</li>
</ul>
<hr />
<h1 id="3-집계-함수는-중첩할-수-없다--인라인-뷰로-해결하기">3. 집계 함수는 중첩할 수 없다 — 인라인 뷰로 해결하기</h1>
<p><strong>SQL</strong> · 처음 시도했다가 에러가 난 쿼리</p>
<pre><code class="language-sql">-- ❌ ERROR
SELECT D.DEPT_NAME, MAX(SUM(E.SALARY))
FROM department D
JOIN employee E ON (D.DEPT_ID = E.DEPT_ID)
GROUP BY DEPT_NAME;</code></pre>
<ul>
<li>부서별 급여 총합(<code>SUM</code>) 중 <strong>가장 큰 값</strong>(<code>MAX</code>)을 구하고 싶었는데, <code>MAX(SUM(...))</code>처럼 <strong>집계 함수 안에 집계 함수를 바로 중첩해서 쓸 수는 없다.</strong> <code>SUM</code>이 각 그룹(부서)별로 하나씩 값을 만들어내는 중인데, 그 결과가 다 모이기도 전에 <code>MAX</code>가 그 위에서 동시에 계산될 수 없기 때문이다.</li>
</ul>
<p><strong>SQL</strong> · 인라인 뷰로 감싸서 해결</p>
<pre><code class="language-sql">SELECT V.DEPT_NAME, MAX(V.SUM)
FROM (
    SELECT D.DEPT_NAME, SUM(E.SALARY) AS `SUM`
    FROM department D
    JOIN employee E ON (D.DEPT_ID = E.DEPT_ID)
    GROUP BY DEPT_NAME
) V;</code></pre>
<ul>
<li>먼저 <code>GROUP BY</code>로 부서별 급여 합계를 구하는 쿼리를 <strong>완성된 결과 하나</strong>로 만든 뒤, 그 결과를 <code>V</code>라는 이름의 <strong>임시 테이블(인라인 뷰)</strong>처럼 취급해서 바깥에서 다시 <code>MAX(V.SUM)</code>으로 감쌌다. <strong>&quot;먼저 그룹별 합계를 다 구하고, 그다음에 그 합계들 중 최댓값을 구한다&quot;</strong>는 두 단계를 <code>FROM</code> 절 안의 서브쿼리로 순서를 명확히 나눈 것이다.</li>
</ul>
<p><strong>SQL</strong> · 같은 문제를 <code>HAVING</code> + 서브쿼리로도 해결</p>
<pre><code class="language-sql">SELECT D.DEPT_NAME, SUM(E.SALARY)
FROM department D
JOIN employee E ON (D.DEPT_ID = E.DEPT_ID)
GROUP BY DEPT_NAME
HAVING SUM(E.SALARY) = (
    SELECT MAX(SUM_SALARY)
    FROM (
        SELECT SUM(SALARY) AS SUM_SALARY
        FROM department D JOIN employee E ON (D.DEPT_ID = E.DEPT_ID)
        GROUP BY DEPT_NAME
    ) V
);</code></pre>
<ul>
<li>이번엔 부서명·합계를 그대로 두고, <code>HAVING</code>으로 &quot;이 그룹의 합계가, 모든 그룹 합계 중 최댓값과 같은가&quot;를 확인했다. 인라인 뷰 방식과 결과는 같지만, <strong>&quot;최고 부서 하나만 골라서 보여주는&quot; 관점(<code>HAVING</code>)</strong>과 <strong>&quot;최댓값을 먼저 구하고 그걸로 필터링하는&quot; 관점(인라인 뷰)</strong>의 차이로 이해했다.</li>
</ul>
<hr />
<h1 id="4-limit--더-간단하게-top-1-구하기">4. <code>LIMIT</code> — 더 간단하게 Top-1 구하기</h1>
<pre><code class="language-sql">SELECT D.DEPT_NAME, SUM(E.SALARY) AS S
FROM employee E
JOIN department D ON (E.DEPT_ID = D.DEPT_ID)
GROUP BY E.DEPT_ID
ORDER BY S DESC
LIMIT 1;</code></pre>
<ul>
<li>3번에서 서브쿼리로 복잡하게 풀었던 &quot;급여 합계가 가장 높은 부서 찾기&quot;를, <code>ORDER BY 합계 DESC</code>(내림차순 정렬) 후 <code>LIMIT 1</code>(첫 번째 행만)로 훨씬 간단하게 구할 수 있다. <strong>동점(공동 1위)이 있어도 딱 한 행만 나온다</strong>는 차이는 있지만, 대부분의 상황에서는 이 방식이 훨씬 간결하다.</li>
</ul>
<pre><code class="language-sql">-- OFFSET: 몇 번째 행부터 가져올지 지정
SELECT E.DEPT_ID, SUM(SALARY)
FROM employee E
GROUP BY DEPT_ID
ORDER BY 2 DESC
LIMIT 3 OFFSET 0;</code></pre>
<ul>
<li><code>LIMIT 3 OFFSET 0</code>: 0번째(맨 처음)부터 시작해서 3개만 가져온다. <code>OFFSET</code>을 조정하면 &quot;다음 페이지&quot;처럼 이어서 조회하는 페이징에도 쓸 수 있다.</li>
</ul>
<hr />
<h1 id="5-다중행-서브쿼리--any-all">5. 다중행 서브쿼리 — <code>ANY</code>, <code>ALL</code></h1>
<pre><code class="language-sql">SELECT E.EMP_NAME, E.SALARY
FROM employee E
JOIN job J ON (E.JOB_ID = J.JOB_ID)
WHERE J.JOB_TITLE = '대리'
AND SALARY &lt; ALL (
    SELECT E.SALARY
    FROM employee E
    JOIN job J ON (E.JOB_ID = J.JOB_ID)
    WHERE J.JOB_TITLE = '과장'
);</code></pre>
<ul>
<li>&quot;과장 직급보다 많은 급여를 받는 대리&quot;를 찾는 문제인데, 과장은 <strong>여러 명</strong>이라 서브쿼리 결과가 여러 행(다중행)이다. 이럴 땐 <code>=</code>나 <code>&gt;</code> 같은 일반 연산자를 바로 못 쓰고, <code>ANY</code>/<code>ALL</code>을 써야 한다.</li>
<li><code>SALARY &lt; ALL(...)</code>은 &quot;이 급여가 과장들 급여 <strong>전부보다</strong> 작다&quot;는 뜻이다. 문제(더 많이 받는 대리 찾기)의 반대 방향처럼 보일 수 있는데, 실제로는 &quot;과장 중 최솟값보다도 작은 대리&quot;를 찾는 것과 같다 — <code>ALL</code> 조건은 &quot;가장 낮은 기준(또는 가장 높은 기준)까지 넘어서야 만족&quot;하는 서술이라는 걸 잘 새겨야 한다.</li>
</ul>
<table>
<thead>
<tr>
<th>연산자</th>
<th>의미</th>
</tr>
</thead>
<tbody><tr>
<td><code>&gt; ANY (...)</code></td>
<td>서브쿼리 결과 중 <strong>하나보다만</strong> 커도 참 → 사실상 &quot;최솟값보다 크다&quot;</td>
</tr>
<tr>
<td><code>&gt; ALL (...)</code></td>
<td>서브쿼리 결과 <strong>전부보다</strong> 커야 참 → 사실상 &quot;최댓값보다 크다&quot;</td>
</tr>
<tr>
<td><code>&lt; ANY (...)</code></td>
<td>서브쿼리 결과 중 <strong>하나보다만</strong> 작아도 참 → 사실상 &quot;최댓값보다 작다&quot;</td>
</tr>
<tr>
<td><code>&lt; ALL (...)</code></td>
<td>서브쿼리 결과 <strong>전부보다</strong> 작아야 참 → 사실상 &quot;최솟값보다 작다&quot;</td>
</tr>
</tbody></table>
<hr />
<h1 id="6-상관관계-서브쿼리--바깥-값을-안쪽에서-참조하기">6. 상관관계 서브쿼리 — 바깥 값을 안쪽에서 참조하기</h1>
<p><strong>SQL</strong> · 자기 직급의 평균 급여를 받는 직원 찾기 (세 가지 풀이 비교)</p>
<pre><code class="language-sql">-- CASE 03: 상관관계 서브쿼리
SELECT E.EMP_NAME, J.JOB_TITLE, E.SALARY
FROM job J
JOIN employee E ON (J.JOB_ID = E.JOB_ID)
WHERE E.SALARY = (
    SELECT TRUNCATE(AVG(SALARY), -5)
    FROM employee EE
    WHERE EE.JOB_ID = E.JOB_ID     -- ← 바깥 쿼리의 E.JOB_ID를 그대로 참조!
);</code></pre>
<ul>
<li>서브쿼리 안의 <code>EE.JOB_ID = E.JOB_ID</code>에서 <code>E</code>는 <strong>바깥 쿼리에 있는 테이블 별칭</strong>이다. 이렇게 서브쿼리가 자기 자신만으로 계산을 끝내지 못하고, <strong>바깥 쿼리의 &quot;지금 보고 있는 행&quot;의 값을 참조</strong>하는 걸 <strong>상관관계 서브쿼리</strong>라고 한다.</li>
<li>동작 방식이 독특한데, 이 쿼리는 <strong>바깥 쿼리가 한 행씩 넘어갈 때마다, 서브쿼리가 그 행의 <code>JOB_ID</code> 기준으로 매번 다시 계산</strong>된다. &quot;전체 직원 중 자기 직급 평균과 같은 사람&quot;을 찾는 게 아니라, &quot;이 사람 직급의 평균과, 이 사람의 급여가 같은가&quot;를 <strong>직원 한 명 한 명마다 새로</strong> 확인하는 방식이다.</li>
</ul>
<p>같은 문제를 CASE 01(다중행 서브쿼리로 <code>(JOB_ID, SALARY) IN (...)</code>)과 CASE 02(인라인 뷰로 미리 직급별 평균을 구해두고 조인)로도 풀어봤는데, 셋 다 결과는 같지만 <strong>&quot;언제, 몇 번 계산이 일어나는가&quot;</strong>가 다르다는 걸 <code>EXPLAIN</code>으로 실행계획을 확인하며 비교했다.</p>
<hr />
<h1 id="7-rank-over--윈도우-함수로-순위-매기기">7. <code>RANK() OVER</code> — 윈도우 함수로 순위 매기기</h1>
<pre><code class="language-sql">SELECT SUB.EMP_NAME, SUB.DEPT_NAME, SUB.SALARY
FROM (
    SELECT E.EMP_NAME, D.DEPT_NAME, E.SALARY,
           RANK() OVER (PARTITION BY E.DEPT_ID ORDER BY E.SALARY) AS RNK
    FROM employee E
    JOIN department D ON (E.DEPT_ID = D.DEPT_ID)
) SUB
WHERE SUB.RNK = 1;</code></pre>
<ul>
<li><code>RANK() OVER (PARTITION BY E.DEPT_ID ORDER BY E.SALARY)</code>: <strong>부서(<code>DEPT_ID</code>)별로 그룹을 나누고(<code>PARTITION BY</code>)</strong>, 그 안에서 급여 기준으로 순위를 매긴다(<code>ORDER BY</code>). <code>GROUP BY</code>처럼 여러 행을 하나로 합치는 게 아니라, <strong>각 행에 순위 번호를 매긴 채로 원래 행 개수 그대로</strong> 결과가 나온다는 점이 <code>GROUP BY</code>와의 큰 차이다.</li>
<li>이 값을 바로 <code>WHERE</code>에 쓸 수 없어서(윈도우 함수 결과는 같은 레벨의 <code>WHERE</code>에서 바로 참조 불가), 인라인 뷰로 한 번 감싼 뒤 바깥에서 <code>RNK = 1</code>로 필터링했다 — 3번에서 배운 &quot;집계 결과를 다시 걸러야 하면 인라인 뷰로 감싼다&quot;는 패턴이 여기서도 반복된다.</li>
</ul>
<hr />
<h1 id="8-ddl--create-table과-제약조건">8. DDL — <code>CREATE TABLE</code>과 제약조건</h1>
<p>오늘 <code>TB_BLOG</code> 테이블을 여러 번 다시 만들어보며, 제약조건이 왜 필요한지 하나씩 겪었다.</p>
<p><strong>1단계: 제약조건 없이</strong></p>
<pre><code class="language-sql">CREATE TABLE TB_BLOG (
    BLOG_ID    VARCHAR(50),
    TITLE      VARCHAR(100),
    CONTENT    VARCHAR(100),
    EMAIL      VARCHAR(100),
    CREATE_AT  DATE
);</code></pre>
<p><strong>2단계: <code>PRIMARY KEY</code> 추가</strong></p>
<pre><code class="language-sql">CREATE TABLE TB_BLOG (
    BLOG_ID    VARCHAR(50),
    TITLE      VARCHAR(100),
    CONTENT    VARCHAR(100),
    EMAIL      VARCHAR(100),
    CREATE_AT  DATE,
    PRIMARY KEY (BLOG_ID)
);</code></pre>
<ul>
<li><code>PRIMARY KEY(BLOG_ID)</code>처럼 테이블 정의 맨 아래에 따로 지정하는 걸 <strong>테이블 제약조건</strong>이라 하고, 컬럼 옆에 바로 붙이는 방식(<code>BLOG_ID VARCHAR(50) PRIMARY KEY</code>, 주석 처리됐던 부분)은 <strong>컬럼 제약조건</strong>이라고 부른다. 둘은 표기 위치만 다를 뿐 같은 역할을 한다.</li>
<li><code>PRIMARY KEY</code>는 그 컬럼 값이 각 행마다 <strong>고유해야 하고, <code>NULL</code>도 허용하지 않는다.</strong> 그래서 아래처럼 <code>BLOG_ID</code>에 <code>NULL</code>을 넣으려 하면 에러가 난다.</li>
</ul>
<pre><code class="language-sql">-- ❌ ERROR — PRIMARY KEY 컬럼에는 NULL을 넣을 수 없음
INSERT INTO TB_BLOG (BLOG_ID, TITLE, CONTENT, EMAIL)
VALUES (NULL, '휴', '하루 마무리', 'jslim9413@naver.com');</code></pre>
<p><strong>3단계: <code>DEFAULT</code> 추가</strong></p>
<pre><code class="language-sql">CREATE TABLE TB_BLOG (
    BLOG_ID    VARCHAR(50),
    ...
    CREATE_AT  DATE DEFAULT SYSDATE(),
    PRIMARY KEY (BLOG_ID)
);

INSERT INTO TB_BLOG (BLOG_ID, TITLE, CONTENT, EMAIL)
VALUES ('1', '휴', '하루 마무리', 'jslim9413@naver.com');
-- CREATE_AT을 안 넣어도, 자동으로 오늘 날짜가 채워짐</code></pre>
<ul>
<li><code>DEFAULT SYSDATE()</code>: <code>INSERT</code> 시 이 컬럼에 값을 아예 안 넣으면, 자동으로 현재 날짜·시간이 채워진다. 매번 작성일을 직접 넣지 않아도 되게 해주는 편의 기능.</li>
</ul>
<p><strong>4단계: <code>CHECK</code>로 값의 범위 제한</strong></p>
<pre><code class="language-sql">CREATE TABLE TB_BLOG (
    BLOG_ID    VARCHAR(50),
    TITLE      VARCHAR(100),
    CONTENT    VARCHAR(100),
    EMAIL      VARCHAR(100),
    CATEGORY   VARCHAR(100) CHECK (CATEGORY IN ('전체', '취미', '일상', '코딩')),
    CREATE_AT  DATE DEFAULT SYSDATE(),
    PRIMARY KEY (BLOG_ID)
);

INSERT INTO TB_BLOG (BLOG_ID, CATEGORY) VALUES ('1', '전체');   -- ✅ 통과
-- INSERT INTO TB_BLOG (BLOG_ID, CATEGORY) VALUES ('2', '음악'); -- ❌ CHECK 위반으로 거부됨</code></pre>
<ul>
<li><code>CHECK (조건)</code>: 그 컬럼에 들어올 수 있는 값의 범위를 <strong>DB 차원에서 강제</strong>한다. <code>CATEGORY</code>가 미리 정해둔 네 가지(<code>'전체'</code>, <code>'취미'</code>, <code>'일상'</code>, <code>'코딩'</code>) 중 하나가 아니면 <code>INSERT</code> 자체가 거부된다. 애플리케이션 코드(자바)에서 값을 검증하기 전에, <strong>DB 자체가 한 번 더 걸러주는 안전장치</strong>라는 걸 배웠다.</li>
</ul>
<p><strong>오늘 정리한 제약조건 전체</strong></p>
<table>
<thead>
<tr>
<th>제약조건</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td><code>PRIMARY KEY</code></td>
<td>각 행을 유일하게 구분, <code>NULL</code> 불가</td>
</tr>
<tr>
<td><code>FOREIGN KEY</code></td>
<td>다른 테이블의 PK를 참조, 참조 무결성 보장 (18일차에 개념만 다뤘던 것)</td>
</tr>
<tr>
<td><code>NOT NULL</code></td>
<td>그 컬럼에 <code>NULL</code>을 허용하지 않음</td>
</tr>
<tr>
<td><code>UNIQUE</code></td>
<td>값이 중복되면 안 됨(단, <code>NULL</code>은 허용)</td>
</tr>
<tr>
<td><code>CHECK</code></td>
<td>지정한 조건을 만족하는 값만 허용</td>
</tr>
<tr>
<td><code>DEFAULT</code></td>
<td>값을 안 넣으면 자동으로 채워질 기본값 지정</td>
</tr>
</tbody></table>
<hr />
<h1 id="✨-새롭게-알게-된-점">✨ 새롭게 알게 된 점</h1>
<ul>
<li>집계 함수는 바로 중첩(<code>MAX(SUM(...))</code>)할 수 없고, 인라인 뷰로 한 번 감싸서 &quot;먼저 집계 → 그 결과를 다시 집계&quot;하는 두 단계로 나눠야 한다.</li>
<li><code>ANY</code>/<code>ALL</code>은 이름만 보면 헷갈리지만, <code>&lt; ALL</code>은 &quot;최솟값보다 작다&quot;, <code>&gt; ALL</code>은 &quot;최댓값보다 크다&quot;로 바꿔 생각하면 명확해진다.</li>
<li>상관관계 서브쿼리는 바깥 쿼리의 행 하나하나마다 서브쿼리가 다시 계산된다는 점에서, 미리 계산해두고 조인하는 방식과 동작 원리가 다르다.</li>
<li><code>RANK() OVER</code>는 <code>GROUP BY</code>처럼 행을 합치지 않고, 각 행에 순위만 매겨서 그대로 보여준다.</li>
<li><code>CHECK</code> 제약조건으로 애플리케이션 코드 이전에 DB 단에서부터 잘못된 값을 막을 수 있다.</li>
</ul>
<hr />
<h1 id="🤔-헷갈렸던-점">🤔 헷갈렸던 점</h1>
<ul>
<li><strong>인라인 뷰 vs <code>HAVING</code> + 서브쿼리 vs <code>LIMIT</code></strong>: 같은 &quot;최댓값 그룹 찾기&quot; 문제를 오늘 세 가지 방식으로 풀어봤는데, 실무에서 어느 걸 우선 고려해야 하는지는 아직 감이 부족하다(일단 <code>LIMIT</code>이 가장 간단해 보이지만, 동점 처리가 필요하면 다른 방식이 나을 수도 있을 것 같다).</li>
<li><strong>상관관계 서브쿼리의 실행 순서</strong>: &quot;바깥이 먼저 도는지 안쪽이 먼저 도는지&quot;가 일반 서브쿼리와 다르다는 게 아직 완전히 손에 잡히진 않는다.</li>
</ul>
<hr />
<h1 id="다음에-할-일">다음에 할 일</h1>
<ul>
<li><input disabled="" type="checkbox" /> <code>EXPLAIN</code>으로 오늘 짠 여러 방식(인라인 뷰 vs 상관관계 서브쿼리 vs LIMIT)의 실행계획을 비교해서 성능 차이 확인해보기</li>
<li><input disabled="" type="checkbox" /> <code>FOREIGN KEY</code> 제약조건을 실제로 걸어보고, 참조 무결성 위반 시 에러를 직접 재현해보기</li>
<li><input disabled="" type="checkbox" /> <code>UNIQUE</code> 제약조건도 <code>TB_BLOG</code> 예제에 추가해서 중복 시도해보기</li>
</ul>
<hr />
<h1 id="💡-오늘의-회고">💡 오늘의 회고</h1>
<p>오늘은 &quot;같은 문제를 여러 방식으로 풀어보고 비교하는&quot; 하루였다. 부서별 최고 급여 부서 찾기를 인라인 뷰, <code>HAVING</code>+서브쿼리, <code>LIMIT</code> 세 가지로, 자기 직급 평균급여 받는 직원 찾기를 다중행 서브쿼리, 인라인 뷰, 상관관계 서브쿼리 세 가지로 풀어보면서, &quot;정답은 여러 개일 수 있고 각자 장단점이 있다&quot;는 걸 체감했다.</p>
<p><code>MAX(SUM(...))</code> 에러를 직접 만나보고 나서야 왜 인라인 뷰가 필요한지 이해가 됐고, DDL 실습에서는 <code>TB_BLOG</code>를 계속 다시 만들면서 <code>PRIMARY KEY</code> 없이 시작 → <code>NULL</code> 에러 겪기 → <code>DEFAULT</code>로 편의성 추가 → <code>CHECK</code>로 값 제한까지, 제약조건이 하나씩 필요해지는 과정을 직접 겪어서 왜 이런 제약들이 존재하는지 이해가 훨씬 잘 됐다.</p>