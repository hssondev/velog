<blockquote>
<p><strong>한 줄 요약</strong></p>
<p>Database의 정의와 DDL/DML/DCL, 참조 무결성 같은 개념을 먼저 잡고, <code>SELECT</code>문의 기본 구조와 <code>WHERE</code>/<code>LIKE</code>/<code>CONCAT</code>을 &quot;춘 기술대학교&quot; 예제 DB로 10개 문제를 풀며 실습했다.</p>
</blockquote>
<hr />
<h1 id="📚-오늘-학습한-커리큘럼">📚 오늘 학습한 커리큘럼</h1>
<pre><code>Database란? — DDL / DML / DCL, 참조 무결성
        ↓
SELECT 문의 기본 구조
        ↓
WHERE — 조건으로 행 제한하기
        ↓
LIKE / NOT LIKE — 패턴 검색
        ↓
CONCAT — 문자열 연결
        ↓
IN, BETWEEN, IS NULL — 다양한 조건 표현
        ↓
DISTINCT, ORDER BY — 중복 제거와 정렬</code></pre><h1 id="📌-오늘의-학습-키워드">📌 오늘의 학습 키워드</h1>
<ul>
<li>DDL(정의어), DML(조작어), DCL(제어어)</li>
<li>참조 무결성(Referential Integrity)</li>
<li><code>SELECT ~ FROM ~ WHERE ~ GROUP BY ~ HAVING ~ ORDER BY</code></li>
<li><code>LIKE</code>, <code>NOT LIKE</code>, <code>CONCAT</code>, <code>IN</code>, <code>BETWEEN</code>, <code>IS NULL</code>, <code>DISTINCT</code></li>
</ul>
<hr />
<h1 id="0-개념-정리-키워드-사전">0. 개념 정리 (키워드 사전)</h1>
<blockquote>
<p><strong>한 줄 요약</strong></p>
<p>DDL은 &quot;그릇(테이블)을 만들고 바꾸는 말&quot;, DML은 &quot;그 그릇 안의 데이터를 다루는 말&quot;, DCL은 &quot;누가 그 그릇을 만질 수 있는지 정하는 말&quot;이다.</p>
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
<td><strong>DDL (Data Definition Language)</strong></td>
<td>테이블 등 데이터베이스 구조 자체를 정의하는 명령어</td>
<td><code>CREATE</code>, <code>ALTER</code>, <code>DROP</code></td>
</tr>
<tr>
<td><strong>DML (Data Manipulation Language)</strong></td>
<td>테이블 안의 데이터를 조회·조작하는 명령어</td>
<td><code>SELECT</code>, <code>INSERT</code>, <code>UPDATE</code>, <code>DELETE</code></td>
</tr>
<tr>
<td><strong>DCL (Data Control Language)</strong></td>
<td>데이터베이스에 대한 접근 권한을 제어하는 명령어</td>
<td><code>GRANT</code>, <code>REVOKE</code></td>
</tr>
<tr>
<td><strong>참조 무결성</strong></td>
<td>서로 연결된 테이블 사이의 데이터가 항상 앞뒤가 맞도록 보장하는 제약</td>
<td>예: 학생의 <code>DEPARTMENT_NO</code>는 반드시 학과 테이블에 실제로 존재하는 값이어야 함</td>
</tr>
<tr>
<td><strong><code>LIKE</code> / <code>NOT LIKE</code></strong></td>
<td>문자열이 특정 패턴을 포함/미포함하는지 검사</td>
<td><code>%</code>(0글자 이상), <code>_</code>(정확히 1글자) 와일드카드 사용</td>
</tr>
<tr>
<td><strong><code>CONCAT</code></strong></td>
<td>여러 문자열(또는 컬럼 값)을 하나로 이어붙임</td>
<td><code>CONCAT(a, ' 님, 안녕')</code></td>
</tr>
<tr>
<td><strong><code>IN</code></strong></td>
<td>여러 값 중 하나와 일치하는지 검사</td>
<td><code>WHERE no IN ('A1','A2','A3')</code></td>
</tr>
<tr>
<td><strong><code>BETWEEN</code></strong></td>
<td>지정한 두 값 <strong>사이(포함)</strong>인지 검사</td>
<td><code>WHERE capacity BETWEEN 20 AND 30</code></td>
</tr>
<tr>
<td><strong><code>IS NULL</code> / <code>IS NOT NULL</code></strong></td>
<td>값이 비어있는지(NULL) 여부를 검사</td>
<td><code>= NULL</code>이 아니라 반드시 <code>IS NULL</code>로 써야 함</td>
</tr>
<tr>
<td><strong><code>DISTINCT</code></strong></td>
<td>조회 결과에서 중복된 값을 제거</td>
<td><code>SELECT DISTINCT category</code></td>
</tr>
</tbody></table>
<h3 id="코드로-다시-보기">코드로 다시 보기</h3>
<pre><code class="language-sql">SELECT      DISTINCT 컬럼명 | 컬럼명 | * | 표현식 및 함수 | [AS] 별칭
FROM        테이블 이름
[WHERE]     행의 제한
[GROUP BY]  데이터를 그룹으로 묶을 때
[HAVING]    그룹에 대한 조건
[ORDER BY]  정렬(ASC, DESC)</code></pre>
<hr />
<h1 id="1-select-기본-구조와-별칭as">1. <code>SELECT</code> 기본 구조와 별칭(<code>AS</code>)</h1>
<p><strong>SQL</strong> · 학과 이름과 계열 조회</p>
<pre><code class="language-sql">SELECT a.DEPARTMENT_NAME AS '학과 명', a.CATEGORY AS '계열'
FROM TB_DEPARTMENT a;</code></pre>
<ul>
<li><code>SELECT</code> 뒤에 조회할 컬럼을 나열하고, <code>AS</code>로 결과 화면에 표시될 <strong>헤더 이름(별칭)</strong>을 따로 지정할 수 있다. 원래 컬럼명(<code>DEPARTMENT_NAME</code>)이 아니라 문제에서 요구한 이름(<code>'학과 명'</code>)으로 헤더를 바꿔 출력했다.</li>
<li><code>a</code>는 <code>TB_DEPARTMENT</code> 테이블에 붙인 별칭이다. 지금은 테이블이 하나뿐이라 꼭 필요하진 않지만, 여러 테이블을 조인할 때 <code>a.컬럼명</code>처럼 &quot;어느 테이블의 컬럼인지&quot; 명확히 구분하기 위해 습관적으로 붙인다.</li>
</ul>
<p><strong>SQL</strong> · 문자열을 조합해서 문장으로 출력</p>
<pre><code class="language-sql">SELECT CONCAT(a.DEPARTMENT_NAME, '의 정원은 ', a.CAPACITY, '명 입니다.') AS '학과별 정원'
FROM TB_DEPARTMENT a;</code></pre>
<ul>
<li><code>CONCAT(...)</code> 안에 컬럼과 문자열 리터럴을 순서대로 나열하면, 그걸 전부 이어붙인 하나의 문자열로 만들어준다. <code>&quot;국어국문학과의 정원은 40명 입니다.&quot;</code>처럼 사람이 읽기 좋은 문장 형태로 데이터를 가공해서 보여줄 때 쓴다.</li>
</ul>
<hr />
<h1 id="2-where--like로-조건에-맞는-행-찾기">2. <code>WHERE</code> + <code>LIKE</code>로 조건에 맞는 행 찾기</h1>
<p><strong>SQL</strong> · 국어국문학과의 휴학 중인 여학생 찾기</p>
<pre><code class="language-sql">SELECT a.STUDENT_NAME
FROM tb_student a
WHERE 1=1
AND a.ABSENCE_YN = 'Y'
AND a.DEPARTMENT_NO = '001'
AND (a.STUDENT_SSN LIKE '______-2%' OR a.STUDENT_SSN LIKE '______-4%');</code></pre>
<ul>
<li><code>WHERE 1=1</code>은 항상 참인 조건을 맨 앞에 하나 깔아두고, 그 뒤로 <code>AND</code> 조건들을 계속 붙이는 습관이다. 나중에 조건을 추가/삭제할 때 매번 첫 줄이 <code>AND</code>냐 아니냐를 신경 쓰지 않아도 돼서 편하다.</li>
<li><code>STUDENT_SSN LIKE '______-2%'</code>: 주민번호 형식(<code>앞6자리-뒷자리</code>)에서, 언더스코어(<code>_</code>) 6개로 앞자리 6글자를 각각 하나씩 표현하고, <code>-</code> 뒤 첫 글자가 <code>2</code>인 것만 찾는다. 한국 주민번호 뒷자리 첫 숫자가 2 또는 4면 여성을 뜻하므로, 이 조건으로 여학생을 걸러낸다.</li>
</ul>
<p>SQL에서는 <code>AND</code>가 <code>OR</code>보다 <strong>우선순위가 높아서</strong>, 이 조건은 사람이 읽은 의도(휴학중 + 국문과 + (2로 시작 또는 4로 시작))가 아니라 <code>(휴학중 AND 국문과 AND 2로 시작) OR (4로 시작)</code>으로 해석된다. 즉 뒤쪽 <code>4로 시작</code> 조건은 휴학·학과 조건과 <strong>전혀 무관하게</strong> 통째로 걸려버려서, 다른 학과의 재학생까지 결과에 섞여 들어갈 수 있는 버그였다. <code>(... OR ...)</code>처럼 <strong>괄호로 묶어야</strong> 의도한 대로 동작한다 — <code>AND</code>/<code>OR</code>을 섞어 쓸 때는 항상 우선순위를 의심하고 괄호를 쳐야 한다는 걸 직접 겪고 배웠다.</p>
<hr />
<h1 id="3-in--여러-값-중-하나와-일치">3. <code>IN</code> — 여러 값 중 하나와 일치</h1>
<pre><code class="language-sql">SELECT a.STUDENT_NAME
FROM tb_student a
WHERE a.STUDENT_NO IN ('A513079', 'A513090', 'A513091', 'A513110', 'A513119')
ORDER BY 1 DESC;</code></pre>
<ul>
<li><code>IN (...)</code>은 <code>학번 = 'A513079' OR 학번 = 'A513090' OR ...</code>를 한 번에 줄여 쓴 것과 같다. 특정 학번 목록(장기 연체자)에 해당하는 사람만 골라낼 때 매번 <code>OR</code>을 여러 번 쓰는 것보다 훨씬 간결하다.</li>
<li><code>ORDER BY 1 DESC</code>: 컬럼 이름 대신 <code>SELECT</code>에 나열한 순서 번호(1번째 컬럼)로 정렬 기준을 지정할 수 있다. <code>DESC</code>는 내림차순.</li>
</ul>
<hr />
<h1 id="4-between--범위-안에-있는지-확인">4. <code>BETWEEN</code> — 범위 안에 있는지 확인</h1>
<pre><code class="language-sql">SELECT a.DEPARTMENT_NAME, a.CATEGORY
FROM tb_department a
WHERE a.CAPACITY BETWEEN 20 AND 30;</code></pre>
<ul>
<li><code>BETWEEN 20 AND 30</code>은 <code>CAPACITY &gt;= 20 AND CAPACITY &lt;= 30</code>과 같다. <strong>양쪽 끝값을 포함</strong>한다는 점(20과 30도 결과에 들어감)이 핵심.</li>
</ul>
<hr />
<h1 id="5-is-null--비어있는-값-찾기">5. <code>IS NULL</code> — 비어있는 값 찾기</h1>
<pre><code class="language-sql">-- 소속 학과가 없는 교수(= 총장) 찾기
SELECT a.PROFESSOR_NAME
FROM TB_PROFESSOR a
WHERE a.DEPARTMENT_NO IS NULL;

-- 학과가 지정 안 된 학생 확인
SELECT a.*
FROM tb_student a
WHERE a.DEPARTMENT_NO IS NULL;</code></pre>
<ul>
<li><code>DEPARTMENT_NO IS NULL</code>: 모든 교수는 소속 학과가 있는데 총장만 예외라면, &quot;학과 번호가 비어있는 사람 = 총장&quot;이라는 논리로 찾아낼 수 있다.</li>
<li>⚠️ <code>= NULL</code>이 아니라 반드시 <code>IS NULL</code>을 써야 한다. NULL은 &quot;알 수 없음&quot;이라는 특수한 상태라서, <code>=</code>(같다) 비교 자체가 성립하지 않는다.</li>
</ul>
<hr />
<h1 id="6-is-not-null--값이-존재하는지-확인">6. <code>IS NOT NULL</code> — 값이 존재하는지 확인</h1>
<pre><code class="language-sql">-- 선수과목이 지정되어 있는 과목 찾기
SELECT a.CLASS_NO
FROM tb_class a
WHERE a.PREATTENDING_CLASS_NO IS NOT NULL;</code></pre>
<ul>
<li><code>IS NOT NULL</code>은 반대로 &quot;값이 채워져 있는 행&quot;만 찾는다. 선수과목이 아예 없는 과목은 이 컬럼이 비어있을 테니, 이 조건으로 &quot;선수과목이 존재하는 과목&quot;만 걸러낸다.</li>
</ul>
<hr />
<h1 id="7-distinct--중복-없이-종류만-보기">7. <code>DISTINCT</code> — 중복 없이 종류만 보기</h1>
<pre><code class="language-sql">SELECT DISTINCT a.CATEGORY
FROM tb_department a
ORDER BY 1;</code></pre>
<ul>
<li>학과는 63개나 있지만, 계열(<code>CATEGORY</code>) 종류는 훨씬 적을 것이다. <code>DISTINCT</code>를 붙이면 같은 값은 한 번만 남기고 나머지는 제거해서, &quot;어떤 계열들이 있는지&quot; 목록만 깔끔하게 볼 수 있다.</li>
</ul>
<hr />
<h1 id="8-여러-조건을-조합--다중-컬럼-정렬">8. 여러 조건을 조합 + 다중 컬럼 정렬</h1>
<pre><code class="language-sql">SELECT  a.STUDENT_NO, a.STUDENT_NAME, a.STUDENT_SSN
FROM tb_student a
WHERE 1=1
AND a.ABSENCE_YN = 'N'
AND a.STUDENT_ADDRESS LIKE '%전주%'
AND a.ENTRANCE_DATE LIKE '2002-%'
ORDER BY 2 ASC, 1, 3 DESC;</code></pre>
<ul>
<li>재학 중(<code>ABSENCE_YN='N'</code>) + 주소에 &quot;전주&quot;가 포함(<code>LIKE '%전주%'</code>) + 2002년 입학(<code>ENTRANCE_DATE LIKE '2002-%'</code>) 세 조건을 <code>AND</code>로 동시에 만족해야 한다.</li>
<li><code>ORDER BY 2 ASC, 1, 3 DESC</code>: 정렬 기준을 <strong>여러 개 동시에</strong> 지정할 수 있다. 2번째 컬럼(이름)을 오름차순으로 먼저 정렬하고, 이름이 같은 사람이 있으면 그다음 1번째 컬럼(학번, 기본은 오름차순)으로, 그래도 같으면 3번째 컬럼(주민번호)을 내림차순으로 정렬한다.</li>
</ul>
<hr />
<h1 id="🤔-헷갈렸던-점">🤔 헷갈렸던 점</h1>
<ul>
<li><strong>오라클과 마리아DB의 차이</strong>: 마리아DB는 ANSI SQL 표준을 따르고, 오라클 전용 문법(예: <code>(+)</code> 외부 조인 표기, <code>ROWNUM</code>, <code>DUAL</code> 테이블)은 지원하지 않는다는 걸 찾아봤다. 반대로 마리아DB에만 있는 내장 함수도 있어서, &quot;표준 SQL&quot;과 &quot;특정 DBMS 전용 문법&quot;을 구분해서 기억해야 할 것 같다.</li>
<li><strong><code>AND</code>/<code>OR</code> 우선순위</strong>: 위 2번 섹션에서 실제로 겪은 버그처럼, 섞어 쓸 때 괄호 없이는 의도와 다르게 동작할 수 있다는 걸 몸으로 배웠다.</li>
</ul>
<hr />
<h1 id="다음에-할-일">다음에 할 일</h1>
<ul>
<li><input disabled="" type="checkbox" /> 오라클 전용 문법(<code>(+)</code>, <code>ROWNUM</code> 등)과 ANSI SQL 표준 문법을 표로 정리해두기</li>
<li><input disabled="" type="checkbox" /> <code>GROUP BY</code>/<code>HAVING</code>은 아직 실습을 못 해봐서 다음에 이어서 연습하기</li>
<li><input disabled="" type="checkbox" /> ERD Cloud로 오늘 다룬 <code>TB_STUDENT</code>, <code>TB_DEPARTMENT</code>, <code>TB_PROFESSOR</code>, <code>TB_CLASS</code> 테이블 관계도 그려보기</li>
</ul>
<hr />
<h1 id="📚-참고-자료">📚 참고 자료</h1>
<ul>
<li><a href="https://www.erdcloud.com/">ERD Cloud</a></li>
</ul>
<hr />
<h1 id="💡-오늘의-회고">💡 오늘의 회고</h1>
<p>오늘은 SQL 문법 하나하나보다, <strong>조건을 정확하게 표현하는 것 자체가 생각보다 어렵다</strong>는 걸 느낀 날이었다. 특히 <code>AND</code>/<code>OR</code>을 괄호 없이 섞어 쓴 조건이 내가 생각한 것과 다르게 해석된다는 걸 직접 겪고 나서, &quot;이 조건문이 정말 내가 원하는 걸 걸러내는지&quot;를 실행 전에 한 번 더 의심해봐야겠다는 생각이 들었다.</p>