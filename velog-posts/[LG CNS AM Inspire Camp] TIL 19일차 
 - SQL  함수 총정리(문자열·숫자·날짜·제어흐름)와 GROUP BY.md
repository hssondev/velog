<blockquote>
<p><strong>한 줄 요약</strong></p>
<p>함수를 &quot;단일행&quot;(문자열·숫자·날짜·제어흐름)과 &quot;복수행&quot;(집계)으로 나누고, 각 카테고리를 실습한 뒤 &quot;춘 기술대학교&quot; 문제 15개로 실전 응용까지 이어서 정리했다.</p>
</blockquote>
<hr />
<h1 id="📚-오늘-학습한-커리큘럼">📚 오늘 학습한 커리큘럼</h1>
<pre><code>함수란? — 단일행 함수 vs 복수행(집계) 함수
        ↓
문자열 함수 (LENGTH, SUBSTRING, INSTR, PAD, TRIM, REPLACE ...)
        ↓
CAST — 타입 강제 변환
        ↓
숫자 함수 (ROUND, TRUNCATE, CEILING, FLOOR, GREATEST/LEAST)
        ↓
날짜 함수 (NOW, ADDDATE, DATEDIFF, YEAR/MONTH/DAY ...)
        ↓
제어흐름 함수 (IF, IFNULL, CASE WHEN)
        ↓
집계 함수 + GROUP BY / HAVING / ORDER BY
        ↓
실전 문제 15개 (SELECT ~ GROUP BY ~ JOIN ~ ROLLUP)</code></pre><h1 id="📌-오늘의-학습-키워드">📌 오늘의 학습 키워드</h1>
<ul>
<li>단일행 함수: 문자열 / 숫자 / 날짜 / 제어흐름</li>
<li><code>CAST</code></li>
<li>복수행(집계) 함수: <code>COUNT</code>, <code>MIN</code>, <code>MAX</code>, <code>AVG</code>, <code>SUM</code></li>
<li><code>GROUP BY</code>, <code>HAVING</code>, <code>ORDER BY</code></li>
<li><code>CREATE TABLE</code>, <code>INSERT INTO</code></li>
</ul>
<hr />
<h1 id="0-개념-정리-키워드-사전">0. 개념 정리 (키워드 사전)</h1>
<blockquote>
<p><strong>한 줄 요약</strong></p>
<p>함수는 반복되는 계산·가공 로직을 미리 만들어둔 &quot;부품&quot;이고, 오늘 배운 함수는 크게 문자열/숫자/날짜/제어흐름(모두 단일행)과 집계(복수행) 다섯 갈래로 나뉜다.</p>
</blockquote>
<p><strong>문자열 함수</strong></p>
<table>
<thead>
<tr>
<th>함수</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td><code>LENGTH</code> / <code>CHAR_LENGTH</code></td>
<td>바이트 수 / 글자 수</td>
</tr>
<tr>
<td><code>CONCAT</code></td>
<td>문자열 이어붙이기</td>
</tr>
<tr>
<td><code>SUBSTRING</code> (<code>SUBSTR</code>)</td>
<td>특정 위치부터 N글자 잘라내기</td>
</tr>
<tr>
<td><code>LEFT</code> / <code>RIGHT</code></td>
<td>앞/뒤에서부터 N글자 잘라내기</td>
</tr>
<tr>
<td><code>INSTR</code></td>
<td>특정 문자가 몇 번째에 있는지 찾기</td>
</tr>
<tr>
<td><code>SUBSTRING_INDEX</code></td>
<td>구분자를 기준으로 N번째 조각까지 잘라내기</td>
</tr>
<tr>
<td><code>REPLACE</code></td>
<td>특정 문자열을 다른 문자열로 치환</td>
</tr>
<tr>
<td><code>UPPER</code> / <code>LOWER</code></td>
<td>대문자/소문자로 변환</td>
</tr>
<tr>
<td><code>TRIM</code> / <code>LTRIM</code> / <code>RTRIM</code></td>
<td>양쪽/왼쪽/오른쪽 공백 제거</td>
</tr>
<tr>
<td><code>LPAD</code> / <code>RPAD</code></td>
<td>왼쪽/오른쪽을 지정한 문자로 채워 길이 맞추기</td>
</tr>
<tr>
<td><code>REPEAT</code></td>
<td>문자열을 N번 반복</td>
</tr>
</tbody></table>
<p><strong>숫자 함수</strong></p>
<table>
<thead>
<tr>
<th>함수</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td><code>ABS</code></td>
<td>절댓값</td>
</tr>
<tr>
<td><code>CEILING</code></td>
<td>올림</td>
</tr>
<tr>
<td><code>FLOOR</code></td>
<td>내림</td>
</tr>
<tr>
<td><code>ROUND(값, N)</code></td>
<td>N번째 자리까지 반올림 (N이 음수면 정수 자리 반올림)</td>
</tr>
<tr>
<td><code>TRUNCATE(값, N)</code></td>
<td>N번째 자리까지 자르기(버림)</td>
</tr>
<tr>
<td><code>GREATEST</code> / <code>LEAST</code></td>
<td>여러 값 중 최댓값/최솟값</td>
</tr>
</tbody></table>
<p><strong>날짜 함수</strong></p>
<table>
<thead>
<tr>
<th>함수</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td><code>NOW()</code> / <code>SYSDATE()</code></td>
<td>현재 날짜+시간</td>
</tr>
<tr>
<td><code>CURDATE()</code> / <code>CURTIME()</code></td>
<td>현재 날짜만 / 현재 시간만</td>
</tr>
<tr>
<td><code>ADDDATE()</code> / <code>SUBDATE()</code></td>
<td>날짜에 기간을 더하기/빼기</td>
</tr>
<tr>
<td><code>DATEDIFF(날짜1, 날짜2)</code></td>
<td>두 날짜 사이의 일수 차이</td>
</tr>
<tr>
<td><code>YEAR()</code>/<code>MONTH()</code>/<code>DAY()</code>/<code>HOUR()</code>/<code>MINUTE()</code>/<code>SECOND()</code></td>
<td>날짜·시간에서 각 구성요소만 추출</td>
</tr>
<tr>
<td><code>WEEKDAY()</code> / <code>DAYOFWEEK()</code></td>
<td>요일을 숫자로 반환</td>
</tr>
</tbody></table>
<p><strong>제어흐름 / 집계</strong></p>
<table>
<thead>
<tr>
<th>함수</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td><code>IF(조건, 참, 거짓)</code></td>
<td>조건 하나를 검사해 값을 고름</td>
</tr>
<tr>
<td><code>IFNULL(값, 대체값)</code></td>
<td>값이 NULL이면 대체값 반환</td>
</tr>
<tr>
<td><code>CASE WHEN ~ THEN ~ ELSE ~ END</code></td>
<td>여러 조건을 순서대로 검사</td>
</tr>
<tr>
<td><code>COUNT</code>/<code>MIN</code>/<code>MAX</code>/<code>AVG</code>/<code>SUM</code></td>
<td>여러 행을 하나(또는 그룹별)의 값으로 집계</td>
</tr>
</tbody></table>
<h3 id="코드로-다시-보기">코드로 다시 보기</h3>
<pre><code class="language-sql">SELECT LENGTH('임정섭'), CHAR_LENGTH('임정섭');
-- LENGTH: 9 (한글 3글자 × 3바이트)   CHAR_LENGTH: 3 (글자 수 그대로)</code></pre>
<hr />
<h1 id="1-문자열-함수--자르고-찾고-채우고-바꾸기">1. 문자열 함수 — 자르고, 찾고, 채우고, 바꾸기</h1>
<p><strong>SQL</strong> · 이메일 아이디를 두 가지 방법으로 추출</p>
<pre><code class="language-sql">SELECT  EMAIL,
        LEFT(EMAIL, INSTR(EMAIL, '@') - 1),   -- ①
        SUBSTRING_INDEX(EMAIL, '@', 1)          -- ②
FROM    employee;</code></pre>
<ul>
<li>① <code>INSTR(EMAIL, '@')</code>로 <code>@</code>의 위치를 찾고, <code>LEFT(EMAIL, 그위치-1)</code>로 그 앞부분만 잘라낸다. <strong>&quot;위치를 찾고 → 그 위치까지 자른다&quot;</strong>는 2단계 조합.</li>
<li>② <code>SUBSTRING_INDEX(EMAIL, '@', 1)</code>는 <strong>한 번에</strong> &quot;구분자 <code>@</code> 기준 1번째 조각&quot;을 반환한다 — 같은 결과를 훨씬 짧게 얻는 방법. <strong>함수는 이렇게 여러 개를 중첩해서 쓸 수도 있고, 처음부터 그 일을 전담하는 함수를 골라 쓸 수도 있다</strong>는 걸 두 방식을 비교하며 배웠다.</li>
</ul>
<p><strong>SQL</strong> · 자릿수 맞추기(<code>LPAD</code>)</p>
<pre><code class="language-sql">SELECT LPAD('5', 3, ' '), LENGTH(LPAD('5', 3, ' '));
-- '  5' , 3</code></pre>
<ul>
<li><code>LPAD(문자열, 목표길이, 채울문자)</code>: 문자열이 목표 길이보다 짧으면, <strong>왼쪽</strong>을 지정한 문자로 채워 길이를 맞춘다. 표 형태로 숫자를 정렬해서 보여줄 때 자주 쓰인다.</li>
</ul>
<p><strong>SQL</strong> · 마지막 조각만 꺼내기 (음수 인덱스)</p>
<pre><code class="language-sql">SELECT SUBSTRING_INDEX('WWW.LGCNS.COM', '.', -1);
-- COM</code></pre>
<ul>
<li><code>SUBSTRING_INDEX</code>에 <strong>음수</strong>를 넣으면, 앞이 아니라 <strong>뒤에서부터</strong> 센 조각을 반환한다. <code>-1</code>이면 마지막 조각.</li>
</ul>
<p><strong>SQL</strong> · 문자열 치환</p>
<pre><code class="language-sql">SELECT REPLACE('오늘은 즐거운 함수 수업', '함수', '메서드');
-- 오늘은 즐거운 메서드 수업</code></pre>
<hr />
<h1 id="2-날짜-함수--근속년수-계산해보기">2. 날짜 함수 — 근속년수 계산해보기</h1>
<p><strong>SQL</strong> · 근속년수가 30년 이상인 사원 찾기</p>
<pre><code class="language-sql">SELECT EMP_NAME, HIRE_DATE,
       ROUND(DATEDIFF(CURDATE(), HIRE_DATE) / 365) AS `근속년수`
FROM employee
WHERE DATEDIFF(CURDATE(), HIRE_DATE) / 365 &gt;= 30;</code></pre>
<ul>
<li><code>DATEDIFF(날짜1, 날짜2)</code>: 두 날짜 사이의 <strong>일수 차이</strong>를 정수로 반환한다.</li>
<li>그 일수를 <code>365</code>로 나누면 대략적인 &quot;년수&quot;가 나온다. <code>ROUND</code>로 반올림해서 보기 좋은 정수로 표시했다.</li>
</ul>
<p>⚠️ <strong>윤년 보정</strong>: 매 4년마다 366일인 윤년이 끼기 때문에, 정확히는 <code>365</code>가 아니라 <strong><code>365.25</code></strong>로 나눠야 오차가 줄어든다는 걸 배웠다.</p>
<pre><code class="language-sql">WHERE DATEDIFF(CURDATE(), HIRE_DATE) / 365.25 &gt;= 30</code></pre>
<p>경계값 근처(정확히 30년에 가까운 사람)에서는 <code>365</code>냐 <code>365.25</code>냐에 따라 포함 여부가 달라질 수 있다 — 날짜 계산은 &quot;대략 맞다&quot;가 아니라 이런 미세한 기준까지 신경 써야 한다는 걸 느꼈다.</p>
<p><strong>SQL</strong> · 날짜에서 각 구성요소만 뽑기</p>
<pre><code class="language-sql">SELECT HIRE_DATE,
       CAST(YEAR(HIRE_DATE) AS CHAR),
       CAST(MONTH(HIRE_DATE) AS CHAR),
       CAST(DAY(HIRE_DATE) AS CHAR)
FROM employee;</code></pre>
<ul>
<li><code>YEAR()</code>/<code>MONTH()</code>/<code>DAY()</code>는 날짜 타입 컬럼에서 각각 연/월/일 숫자만 뽑아낸다. 뒤에 <code>CAST(... AS CHAR)</code>를 붙인 건, 숫자 그대로 두면 다른 문자열과 이어붙이거나 표시 형식을 맞추기 까다로워서 문자열로 바꿔둔 것이다.</li>
</ul>
<hr />
<h1 id="3-cast--타입을-강제로-바꾸기">3. <code>CAST</code> — 타입을 강제로 바꾸기</h1>
<p><strong>SQL</strong> · 잘못된 형변환과 올바른 형변환</p>
<pre><code class="language-sql">SELECT INT(AVG(AMOUNT)) FROM buytbl;              -- ❌ 에러! INT는 함수가 아니라 타입 이름
SELECT CAST(AVG(AMOUNT) AS SIGNED INTEGER) FROM buytbl;  -- ✅ 정상</code></pre>
<ul>
<li><code>INT(...)</code>처럼 타입 이름을 함수처럼 바로 호출할 수는 없다. 타입을 바꾸고 싶을 땐 반드시 <code>CAST(값 AS 타입)</code> 문법을 써야 한다는 걸 에러를 직접 겪고 확인했다.</li>
</ul>
<p><strong>SQL</strong> · 문자열과 숫자를 섞어 계산</p>
<pre><code class="language-sql">SELECT NUM AS `구매번호`,
       CONCAT(CAST(PRICE AS VARCHAR(10)), ' * ', CAST(AMOUNT AS VARCHAR(10)), ' = ') AS `총 금액`,
       PRICE * AMOUNT AS `구매액`
FROM buytbl;</code></pre>
<ul>
<li><code>CONCAT</code>은 문자열끼리 이어붙이는 함수라서, 숫자 컬럼(<code>PRICE</code>, <code>AMOUNT</code>)을 그 안에 넣으려면 먼저 <code>CAST(... AS VARCHAR(10))</code>로 문자열로 바꿔줘야 한다.</li>
<li>반대로 <code>PRICE * AMOUNT</code>처럼 <strong>진짜 곱셈</strong>을 할 때는 숫자 타입 그대로 써야 한다 — &quot;보여주기 위한 문자열 조합&quot;과 &quot;실제 계산&quot;을 구분해서 타입을 다뤄야 한다는 걸 이 예제로 정리했다.</li>
</ul>
<hr />
<h1 id="4-숫자-함수--반올림-올림-내림-비교">4. 숫자 함수 — 반올림, 올림, 내림 비교</h1>
<pre><code class="language-sql">SELECT ROUND(4153.415354, 2),      -- 4153.42 (반올림)
       TRUNCATE(4153.415354, 2),   -- 4153.41 (그냥 자름)
       ROUND(4153.415354, -2),     -- 4200    (백의 자리에서 반올림)
       CEILING(4.1),                -- 5 (무조건 올림)
       FLOOR(4.7);                  -- 4 (무조건 내림)</code></pre>
<ul>
<li><code>ROUND</code>와 <code>TRUNCATE</code>는 둘 다 &quot;N번째 자리까지 남긴다&quot;는 점은 같지만, <code>ROUND</code>는 반올림하고 <code>TRUNCATE</code>는 그 뒤를 <strong>그냥 잘라버린다</strong>(반올림 없음)는 차이가 있다.</li>
<li><code>ROUND(..., -2)</code>처럼 <strong>음수</strong>를 넣으면, 소수점이 아니라 <strong>정수 자리</strong>(십의 자리, 백의 자리 ...)에서 반올림한다.</li>
<li><code>CEILING</code>은 소수점이 있으면 무조건 올리고, <code>FLOOR</code>는 무조건 내린다 — 둘 다 반올림 규칙(4 이하/5 이상)과 무관하게 방향이 고정되어 있다.</li>
</ul>
<hr />
<h1 id="5-제어흐름-함수--if-vs-case-when">5. 제어흐름 함수 — <code>IF</code> vs <code>CASE WHEN</code></h1>
<p><strong>SQL</strong> · 성별 판별 (세 가지 방식)</p>
<pre><code class="language-sql">SELECT EMP_NAME, EMP_NO,
       IF(SUBSTRING(EMP_NO, 8, 1) IN ('1', '3'), '남자', '여자') AS GENDER1,   -- ①

       CASE WHEN SUBSTRING(EMP_NO, 8, 1) IN ('1', '3') THEN '남자'              -- ②
            WHEN SUBSTRING(EMP_NO, 8, 1) IN ('2', '4') THEN '여자'
            ELSE '?'
       END AS GENDER2,

       CASE SUBSTRING(EMP_NO, 8, 1)                                              -- ③
            WHEN '1' THEN '남자'  WHEN '2' THEN '여자'
            WHEN '3' THEN '남자'  WHEN '4' THEN '여자'
            ELSE '?'
       END AS GENDER3
FROM employee
WHERE DEPT_ID = '50';</code></pre>
<ul>
<li>① <code>IF</code>: 조건 하나로 딱 두 갈래(참/거짓)만 나눌 때 가장 짧다.</li>
<li>② <strong>검색형 <code>CASE WHEN</code></strong>: <code>WHEN</code> 뒤에 매번 <strong>조건식</strong>(<code>... IN (...)</code>)을 쓴다. 조건이 복잡하거나 서로 다른 컬럼을 비교해야 할 때 쓴다.</li>
<li>③ <strong>단순형 <code>CASE</code></strong>: <code>CASE 컬럼명</code> 뒤에 <code>WHEN 값 THEN 결과</code>만 나열한다. <strong>비교 대상이 컬럼 하나의 &quot;값 일치 여부&quot;뿐일 때</strong> ②보다 더 간결하게 쓸 수 있는 변형이다.</li>
</ul>
<p><strong>SQL</strong> · NULL 대체 — <code>IFNULL</code></p>
<pre><code class="language-sql">SELECT EMP_NAME, EMP_ID,
       IFNULL(MGR_ID, '관리자') AS 사수번호
FROM employee
WHERE JOB_ID = 'J4';</code></pre>
<ul>
<li><code>IFNULL(값, 대체값)</code>: 값이 <code>NULL</code>이면 대체값을, 아니면 원래 값을 그대로 반환한다. 어제 정리한 <code>IS NULL</code>이 &quot;조건으로 걸러내는&quot; 용도였다면, <code>IFNULL</code>은 &quot;걸러내지 않고 그 자리에서 바로 대체값으로 바꿔치기&quot;한다는 차이가 있다.</li>
<li>⚠️ 원래 시도했던 <code>IF(CHAR_LENGTH(MGR_ID) &lt;&gt; 0, MGR_ID, '관리자')</code>는, <code>MGR_ID</code>가 <code>NULL</code>이 아니라 <strong>빈 문자열(<code>''</code>)</strong>로 저장돼 있는 데이터에 맞춘 방식이었다. <code>NULL</code>과 <code>''</code>(빈 문자열)은 <strong>서로 다른 상태</strong>라서, 데이터가 어느 쪽으로 들어있는지에 따라 <code>IFNULL</code>을 쓸지 <code>IF(... &lt;&gt; '' ...)</code>를 쓸지 골라야 한다는 걸 배웠다.</li>
</ul>
<hr />
<h1 id="6-create-table--insert-into--ddl과-dml-맛보기">6. <code>CREATE TABLE</code> / <code>INSERT INTO</code> — DDL과 DML 맛보기</h1>
<pre><code class="language-sql">CREATE TABLE COUPON_TBL (
    CREATE_AT DATE,
    END_AT    DATE
);

INSERT INTO COUPON_TBL (CREATE_AT, END_AT)
VALUES (NOW(), ADDDATE(NOW(), INTERVAL 7 DAY));</code></pre>
<ul>
<li><code>CREATE TABLE</code>(DDL)로 테이블의 뼈대(컬럼과 타입)를 정의하고, <code>INSERT INTO ~ VALUES</code>(DML)로 실제 데이터를 채워넣는다 — 어제 정리한 DDL/DML 구분이 실제 코드로 등장한 부분.</li>
<li><code>ADDDATE(NOW(), INTERVAL 7 DAY)</code>: 지금 시각에 7일을 더한 값을 만들어서, &quot;쿠폰 발급일 + 7일 뒤 만료&quot;라는 데이터를 한 줄로 표현했다.</li>
</ul>
<hr />
<h1 id="7-복수행집계-함수--group-by의-규칙">7. 복수행(집계) 함수 + <code>GROUP BY</code>의 규칙</h1>
<pre><code class="language-sql">SELECT COUNT(*), COUNT(BONUS_PCT), COUNT(IFNULL(BONUS_PCT, 0)),
       MIN(SALARY), MAX(SALARY), AVG(SALARY), SUM(SALARY)
FROM employee;</code></pre>
<ul>
<li><code>COUNT(*)</code>는 전체 행 수, <code>COUNT(BONUS_PCT)</code>는 그중 <code>BONUS_PCT</code>가 <code>NULL</code>이 아닌 행만 센다 — 같은 <code>COUNT</code>라도 <strong>무엇을 세는지</strong>에 따라 결과가 다를 수 있다.</li>
</ul>
<p>⚠️ <strong>오늘 강조된 규칙 두 가지</strong></p>
<ul>
<li><code>WHERE</code> 절에는 집계 함수를 쓸 수 없다 — 집계는 여러 행을 다 모은 뒤에 계산되는데, <code>WHERE</code>는 아직 행을 거르는 단계라 그 결과를 알 수 없기 때문이다. (그래서 집계 결과로 거르려면 어제 배운 <code>HAVING</code>을 써야 한다.)</li>
<li><code>SELECT</code> 절에서 집계 함수와 <strong>일반 컬럼을 그냥 같이 쓸 수는 없다</strong> — <code>GROUP BY</code> 없이 <code>SELECT EMP_NAME, COUNT(*)</code>처럼 쓰면, <code>COUNT(*)</code>는 &quot;전체를 하나로 합친 값&quot;인데 <code>EMP_NAME</code>은 &quot;행마다 다른 값&quot;이라 둘의 결과 개수가 안 맞아 문제가 된다. 그룹별로 집계하고 싶다면 <code>GROUP BY</code>로 기준을 명시해야 한다.</li>
</ul>
<hr />
<h1 id="8-order-by에-별칭alias-쓰기">8. <code>ORDER BY</code>에 별칭(alias) 쓰기</h1>
<pre><code class="language-sql">SELECT EMP_NO, EMP_NAME, SALARY AS S,
       CASE WHEN SALARY &lt;= 3000000 THEN '초급'
            WHEN SALARY &lt;= 4000000 THEN '중급'
            ELSE '고급'
       END AS `GRADE`
FROM employee
ORDER BY GRADE;</code></pre>
<ul>
<li><code>SELECT</code>에서 <code>AS</code>로 붙인 별칭(<code>GRADE</code>)을 <code>ORDER BY</code>에서 그대로 컬럼명처럼 쓸 수 있다. <code>CASE WHEN</code>으로 만든, 원래 테이블에는 없는 계산된 값 기준으로도 정렬할 수 있다는 걸 확인했다.</li>
</ul>
<hr />
<h1 id="9-실전-문제-15개--오늘-배운-함수들의-응용">9. 실전 문제 15개 — 오늘 배운 함수들의 응용</h1>
<p>&quot;춘 기술대학교&quot; 예제 DB로 아래 문제들을 풀며 오늘 배운 함수를 실전에 적용했다. (자세한 라인별 설명은 GROUP BY/HAVING/JOIN/ROLLUP 관련 문제는 앞선 글에서 이미 다뤘으므로, 여기서는 각 문제가 어떤 함수를 응용했는지만 표로 정리한다.)</p>
<table>
<thead>
<tr>
<th>번호</th>
<th>문제 요지</th>
<th>사용한 함수/문법</th>
</tr>
</thead>
<tbody><tr>
<td>1</td>
<td>특정 학과 학생을 입학년도순 정렬</td>
<td><code>WHERE</code>, <code>ORDER BY</code></td>
</tr>
<tr>
<td>2</td>
<td>이름이 3글자가 아닌 교수 찾기</td>
<td><code>CHAR_LENGTH</code></td>
</tr>
<tr>
<td>3</td>
<td>남자 교수의 만 나이 계산</td>
<td><code>SUBSTR</code>, <code>CONCAT</code>, <code>YEAR(CURDATE())</code></td>
</tr>
<tr>
<td>4</td>
<td>성을 뺀 이름만 추출</td>
<td><code>RIGHT</code></td>
</tr>
<tr>
<td>5</td>
<td>재수생 입학자 찾기</td>
<td><code>LEFT</code>, 날짜/문자 혼합 연산</td>
</tr>
<tr>
<td>6</td>
<td>특정 날짜의 요일</td>
<td><code>DAYNAME</code>, <code>REPLACE</code></td>
</tr>
<tr>
<td>7</td>
<td><code>YY</code> vs <code>RR</code> 연도 해석 차이 (개념)</td>
<td><code>TO_DATE</code>(오라클 전용)</td>
</tr>
<tr>
<td>8</td>
<td>2000년 이전 학번 찾기</td>
<td><code>LEFT</code></td>
</tr>
<tr>
<td>9</td>
<td>특정 학생의 총 평점</td>
<td><code>AVG</code>, <code>ROUND</code></td>
</tr>
<tr>
<td>10</td>
<td>학과별 학생 수</td>
<td><code>COUNT</code>, <code>GROUP BY</code></td>
</tr>
<tr>
<td>11</td>
<td>지도교수 미배정 학생 수</td>
<td><code>IS NULL</code>, <code>COUNT</code></td>
</tr>
<tr>
<td>12</td>
<td>학생의 년도별 평점</td>
<td><code>LEFT</code>, <code>GROUP BY</code>, <code>AVG</code></td>
</tr>
<tr>
<td>13</td>
<td>학과별 휴학생 수(0명 포함)</td>
<td><code>LEFT JOIN</code>, <code>COUNT</code></td>
</tr>
<tr>
<td>14</td>
<td>동명이인 찾기</td>
<td><code>GROUP BY</code>, <code>HAVING</code></td>
</tr>
<tr>
<td>15</td>
<td>년도·학기별 평점 + 소계·총계</td>
<td><code>GROUP BY ~ WITH ROLLUP</code></td>
</tr>
</tbody></table>
<hr />
<h1 id="✨-새롭게-알게-된-점">✨ 새롭게 알게 된 점</h1>
<ul>
<li>같은 결과를 얻는 방법이 함수마다 여러 개 있을 수 있다(이메일 아이디 추출: <code>INSTR</code>+<code>LEFT</code> 조합 vs <code>SUBSTRING_INDEX</code> 하나) — 더 짧고 명확한 쪽을 고르는 감각이 필요하다.</li>
<li>근속년수처럼 날짜를 &quot;일수 ÷ 365&quot;로 계산할 때는 윤년 때문에 <code>365.25</code>로 나눠야 더 정확하다.</li>
<li><code>NULL</code>과 빈 문자열(<code>''</code>)은 다른 상태라서, <code>IFNULL</code>을 쓸지 <code>IF(... &lt;&gt; '' ...)</code>를 쓸지는 데이터가 실제로 어떻게 저장되어 있는지에 달려있다.</li>
<li><code>WHERE</code>에는 집계 함수를 못 쓰고, <code>GROUP BY</code> 없이 집계 함수와 일반 컬럼을 같이 쓰면 안 된다는 규칙을 명확히 정리했다.</li>
<li><code>ORDER BY</code>에는 <code>SELECT</code>에서 만든 별칭을 그대로 쓸 수 있다.</li>
</ul>
<hr />
<h1 id="🤔-헷갈렸던-점">🤔 헷갈렸던 점</h1>
<ul>
<li><strong>검색형 <code>CASE WHEN</code> vs 단순형 <code>CASE 컬럼</code></strong>: 언제 어느 걸 쓰는 게 더 적절한지 아직 완전히 손에 익지 않았다.</li>
<li><strong><code>CAST</code>를 문자열 조합과 숫자 계산에서 다르게 써야 하는 것</strong>: <code>CONCAT</code> 안에서는 문자열로, 실제 곱셈에서는 숫자로 — 같은 컬럼인데 상황마다 타입을 오가야 한다는 게 아직 헷갈린다.</li>
</ul>
<hr />
<h1 id="다음에-할-일">다음에 할 일</h1>
<ul>
<li><input disabled="" type="checkbox" /> <code>NULLIF</code> 함수를 아직 안 써봐서 직접 예제로 확인해보기</li>
<li><input disabled="" type="checkbox" /> <code>WEEKDAY()</code>와 <code>DAYOFWEEK()</code>의 반환값 차이(0부터 시작 vs 1부터 시작) 직접 비교해보기</li>
<li><input disabled="" type="checkbox" /> <code>TRUNCATE</code>와 <code>ROUND</code>를 같은 값에 여러 자리수로 적용해보며 차이 표로 정리하기</li>
</ul>
<hr />
<h1 id="💡-오늘의-회고">💡 오늘의 회고</h1>
<p>오늘은 함수 종류가 정말 많았는데, 크게 &quot;문자열/숫자/날짜/제어흐름(단일행)&quot;과 &quot;집계(복수행)&quot; 다섯 갈래로 나눠보니 각 함수가 왜 그 카테고리에 속하는지 감이 잡혔다. 특히 이메일 아이디를 <code>INSTR</code>+<code>LEFT</code> 조합으로도, <code>SUBSTRING_INDEX</code> 하나로도 뽑아낼 수 있다는 걸 비교해보면서, SQL도 자바처럼 &quot;같은 결과를 여러 방식으로 짤 수 있고, 그중 더 명확한 쪽을 고르는 게 실력&quot;이라는 걸 느꼈다.</p>