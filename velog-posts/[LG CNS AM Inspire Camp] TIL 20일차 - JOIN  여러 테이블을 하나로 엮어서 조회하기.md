<blockquote>
<p><strong>한 줄 요약</strong></p>
<p>ERD로 테이블 간 관계(1:N, N:M)를 먼저 파악하고, 암묵적 조인(<code>WHERE</code>)과 ANSI 표준 조인(<code>JOIN ~ ON</code>/<code>USING</code>), <code>LEFT JOIN</code>, <code>SELF JOIN</code>, <code>EXISTS</code>까지 &quot;춘 기술대학교&quot; DB로 실습했다.</p>
</blockquote>
<hr />
<h1 id="📚-오늘-학습한-커리큘럼">📚 오늘 학습한 커리큘럼</h1>
<pre><code>ERD로 테이블 관계 파악하기 (1:N, N:M, FK)
        ↓
암묵적 조인(WHERE) vs 명시적 조인(JOIN ~ ON / USING)
        ↓
여러 테이블 동시에 연결하기 (N:M 관계, 중간 테이블)
        ↓
LEFT JOIN — 관련 데이터가 없어도 포함하기
        ↓
SELF JOIN — 같은 테이블을 자기 자신과 연결하기
        ↓
EXISTS — 서브쿼리로 존재 여부만 확인하기</code></pre><h1 id="📌-오늘의-학습-키워드">📌 오늘의 학습 키워드</h1>
<ul>
<li>ERD, 기본키(PK)/외래키(FK), 1:N, N:M, 중간(연결) 테이블</li>
<li>암묵적 조인, ANSI 조인(<code>JOIN ~ ON</code>, <code>JOIN ~ USING</code>)</li>
<li><code>LEFT JOIN</code>(<code>OUTER JOIN</code>)</li>
<li><code>SELF JOIN</code></li>
<li><code>EXISTS</code></li>
</ul>
<hr />
<h1 id="0-개념-정리-키워드-사전">0. 개념 정리 (키워드 사전)</h1>
<blockquote>
<p><strong>한 줄 요약</strong></p>
<p>JOIN은 &quot;여러 테이블을 임시로 이어붙여 하나의 결과표처럼 보이게 만드는 것&quot;이고, 어떻게 이어붙일지는 ERD에 그려진 FK(외래키) 관계를 그대로 따라가면 된다.</p>
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
<td><strong>ERD (개체-관계 다이어그램)</strong></td>
<td>테이블들이 서로 어떻게 연결되는지 그림으로 표현한 것</td>
<td>오늘 다룬 첨부 ERD</td>
</tr>
<tr>
<td><strong>기본키(PK)</strong></td>
<td>테이블에서 각 행을 유일하게 구분하는 컬럼</td>
<td><code>STUDENT_NO</code>, <code>PROFESSOR_NO</code> 등</td>
</tr>
<tr>
<td><strong>외래키(FK)</strong></td>
<td>다른 테이블의 기본키를 참조하는 컬럼</td>
<td><code>TB_STUDENT.DEPARTMENT_NO</code> → <code>TB_DEPARTMENT.DEPARTMENT_NO</code></td>
</tr>
<tr>
<td><strong>1:N 관계</strong></td>
<td>한쪽 테이블의 행 하나가, 다른 쪽 테이블의 여러 행과 연결됨</td>
<td>학과 1개 : 학생 여러 명</td>
</tr>
<tr>
<td><strong>N:M 관계</strong></td>
<td>양쪽 테이블의 행 여러 개가 서로 여러 개씩 연결됨</td>
<td>과목 여러 개 ↔ 교수 여러 명</td>
</tr>
<tr>
<td><strong>중간(연결) 테이블</strong></td>
<td>N:M 관계를 표현하기 위해 두 테이블의 PK를 각각 FK로 가져와 만든 테이블</td>
<td><code>TB_CLASS_PROFESSOR</code>, <code>TB_GRADE</code></td>
</tr>
<tr>
<td><strong>암묵적 조인</strong></td>
<td><code>FROM</code>에 테이블을 쉼표로 나열하고, <code>WHERE</code>에서 연결 조건을 거는 옛날 방식</td>
<td><code>FROM a, b WHERE a.no = b.no</code></td>
</tr>
<tr>
<td><strong>ANSI 조인</strong></td>
<td><code>JOIN ~ ON</code>으로 연결 조건을 <code>FROM</code> 절 안에 명시하는 표준 문법</td>
<td><code>FROM a JOIN b ON a.no = b.no</code></td>
</tr>
<tr>
<td><strong><code>USING</code></strong></td>
<td>조인하는 두 테이블의 <strong>컬럼명이 완전히 같을 때</strong>, <code>ON</code> 대신 컬럼명만 적는 축약 문법</td>
<td><code>JOIN b USING(no)</code></td>
</tr>
<tr>
<td><strong><code>LEFT JOIN</code></strong></td>
<td>왼쪽 테이블은 전부 남기고, 오른쪽에 대응하는 행이 없으면 <code>NULL</code>로 채움</td>
<td>지도교수가 없는 학생도 보고 싶을 때</td>
</tr>
<tr>
<td><strong><code>SELF JOIN</code></strong></td>
<td>같은 테이블을 별칭을 다르게 줘서 두 번(또는 그 이상) 등장시켜 자기 자신과 연결</td>
<td>사원-사수-사수의 사수</td>
</tr>
<tr>
<td><strong><code>EXISTS</code></strong></td>
<td>서브쿼리 결과가 &quot;한 건이라도 있는지&quot;만 <code>true</code>/<code>false</code>로 확인</td>
<td>구매 이력이 있는 회원만</td>
</tr>
</tbody></table>
<h3 id="코드로-다시-보기">코드로 다시 보기</h3>
<pre><code class="language-sql">-- 암묵적 조인
SELECT E.EMP_NAME, D.DEPT_NAME
FROM employee E, department D
WHERE E.DEPT_ID = D.DEPT_ID;

-- 같은 결과를 ANSI 조인으로
SELECT E.EMP_NAME, D.DEPT_NAME
FROM employee E
JOIN department D ON (E.DEPT_ID = D.DEPT_ID);</code></pre>
<hr />
<h1 id="1-erd로-오늘-실습-db의-관계-파악하기">1. ERD로 오늘 실습 DB의 관계 파악하기</h1>
<p><img alt="" src="https://velog.velcdn.com/images/hssondev/post/141f3e4e-124e-4deb-8fc7-48385d75a5c8/image.png" /></p>
<p>오늘 실습한  DB의 테이블 관계를 ERD에서 읽어보면:</p>
<table>
<thead>
<tr>
<th>관계</th>
<th>설명</th>
</tr>
</thead>
<tbody><tr>
<td><code>TB_DEPARTMENT</code> 1 : N <code>TB_STUDENT</code></td>
<td>학과 하나에 학생 여러 명 (<code>TB_STUDENT.DEPARTMENT_NO</code>가 FK)</td>
</tr>
<tr>
<td><code>TB_DEPARTMENT</code> 1 : N <code>TB_PROFESSOR</code></td>
<td>학과 하나에 교수 여러 명</td>
</tr>
<tr>
<td><code>TB_DEPARTMENT</code> 1 : N <code>TB_CLASS</code></td>
<td>학과 하나에 과목 여러 개</td>
</tr>
<tr>
<td><code>TB_PROFESSOR</code> 1 : N <code>TB_STUDENT</code></td>
<td>교수 한 명이 여러 학생의 <strong>지도교수</strong>(<code>COACH_PROFESSOR_NO</code>)가 됨</td>
</tr>
<tr>
<td><code>TB_CLASS</code> N : M <code>TB_PROFESSOR</code></td>
<td>과목 여러 개 ↔ 교수 여러 명, 중간에 <code>TB_CLASS_PROFESSOR</code> 테이블이 둘을 이어줌</td>
</tr>
<tr>
<td><code>TB_STUDENT</code> N : M <code>TB_CLASS</code></td>
<td>학생 여러 명 ↔ 과목 여러 개(수강신청), 중간에 <code>TB_GRADE</code> 테이블이 이어주며 <code>POINT</code>(성적)도 함께 저장</td>
</tr>
<tr>
<td><code>TB_CLASS</code> 자기 자신 참조</td>
<td><code>PREATTENDING_CLASS_NO</code>(선수과목)가 <code>TB_CLASS.CLASS_NO</code>를 다시 가리킴</td>
</tr>
</tbody></table>
<p><strong>왜 N:M은 직접 연결하지 못하고 중간 테이블이 필요한가</strong>: 과목 하나에 교수가 여러 명 있을 수도 있고, 교수 한 명이 여러 과목을 맡을 수도 있다. 이렇게 <strong>양쪽 다 &quot;여러 개&quot;</strong>인 관계는 어느 한쪽 테이블에 FK 하나만 추가하는 걸로는 표현이 안 된다. 그래서 <code>CLASS_NO</code>와 <code>PROFESSOR_NO</code>를 둘 다 FK로 가진 <code>TB_CLASS_PROFESSOR</code>라는 <strong>중간 테이블</strong>을 따로 두고, &quot;이 과목과 이 교수가 짝지어져 있다&quot;는 조합 하나하나를 행으로 저장한다. <code>TB_GRADE</code>(학생-과목-성적)도 같은 이유로 존재한다.</p>
<hr />
<h1 id="2-암묵적-조인-vs-ansi-조인">2. 암묵적 조인 vs ANSI 조인</h1>
<p><strong>SQL</strong> · 옛날 방식 — <code>WHERE</code>로 연결</p>
<pre><code class="language-sql">SELECT E.EMP_NAME, D.DEPT_NAME
FROM employee E, department D
WHERE E.DEPT_ID = D.DEPT_ID;</code></pre>
<ul>
<li><code>FROM</code>에 테이블 두 개를 쉼표로 나열하면, 일단 두 테이블의 <strong>모든 조합</strong>(카티션 곱)이 만들어진다. 그중 <code>WHERE</code>의 <code>E.DEPT_ID = D.DEPT_ID</code> 조건에 맞는 것만 남긴다. 조건과 &quot;필터링&quot;이 분리되지 않아서, 테이블이 늘어날수록 어떤 게 조인 조건이고 어떤 게 진짜 필터인지 헷갈리기 쉽다.</li>
</ul>
<p><strong>SQL</strong> · ANSI 표준 방식 — <code>JOIN ~ ON</code></p>
<pre><code class="language-sql">SELECT E.EMP_NAME, D.DEPT_NAME
FROM employee E
JOIN department D ON (E.DEPT_ID = D.DEPT_ID);</code></pre>
<ul>
<li>연결 조건(<code>ON</code>)이 <code>FROM</code> 절 안에 명확히 드러나서, &quot;이 두 테이블은 이 조건으로 이어진다&quot;는 게 한눈에 보인다. 실무에서는 이 방식이 표준으로 쓰인다고 배웠다.</li>
</ul>
<p><strong>SQL</strong> · <code>USING</code> — 컬럼명이 같을 때 축약</p>
<pre><code class="language-sql">SELECT E.EMP_NAME, D.DEPT_NAME
FROM employee E
JOIN department D USING (DEPT_ID);</code></pre>
<ul>
<li>양쪽 테이블의 연결 컬럼 이름이 <strong>똑같이 <code>DEPT_ID</code></strong>라면, <code>ON (E.DEPT_ID = D.DEPT_ID)</code> 대신 <code>USING(DEPT_ID)</code>로 더 짧게 쓸 수 있다.</li>
</ul>
<p>⚠️ <strong><code>CROSS JOIN</code>(연결 조건 없이 그냥 JOIN)</strong>: 조건 없이 그냥 <code>JOIN</code>만 쓰면 두 테이블의 모든 조합(카티션 곱)이 그대로 나온다 — 사원 수 × 부서 수만큼의 행이 나오는데, 대부분은 의미 없는 조합이라 실무에서는 거의 쓰지 않는다고 배웠다.</p>
<hr />
<h1 id="3-여러-테이블을-한-번에-연결하기-nm-관계">3. 여러 테이블을 한 번에 연결하기 (N:M 관계)</h1>
<p><strong>SQL</strong> · 과목 이름 + 담당 교수 이름 (중간 테이블 경유)</p>
<pre><code class="language-sql">SELECT a.CLASS_NAME, b.PROFESSOR_NAME
FROM tb_class a               -- 과목
    , tb_professor b           -- 교수
    , tb_class_professor c     -- 과목-교수 중간 테이블
WHERE 1=1
AND a.CLASS_NO = c.CLASS_NO
AND b.PROFESSOR_NO = c.PROFESSOR_NO;</code></pre>
<ul>
<li><code>tb_class</code>와 <code>tb_professor</code>는 서로 직접 연결된 컬럼이 없다(ERD를 봐도 화살표가 안 이어져 있다). 대신 <code>tb_class_professor</code>(중간 테이블)가 <strong>양쪽과 각각 연결</strong>되어 있어서, <code>과목 → 중간테이블 → 교수</code> 두 단계를 거쳐야 원하는 결과가 나온다.</li>
</ul>
<p><strong>SQL</strong> · 여기에 학과 조건까지 추가 (테이블 4개 연결)</p>
<pre><code class="language-sql">SELECT a.CLASS_NAME, c.PROFESSOR_NAME
FROM tb_class a
    , tb_class_professor b
    , tb_professor c
    , tb_department d
WHERE 1=1
AND a.CLASS_NO = b.CLASS_NO
AND b.PROFESSOR_NO = c.PROFESSOR_NO
AND a.DEPARTMENT_NO = d.DEPARTMENT_NO
AND d.CATEGORY = '인문사회';</code></pre>
<ul>
<li>테이블이 4개로 늘었지만 원리는 같다: <strong>각 테이블을 어떤 컬럼으로 연결할지 하나씩 <code>AND</code> 조건으로 추가</strong>해나가면 된다. <code>tb_class</code>가 <code>tb_class_professor</code>(과목-교수), <code>tb_department</code>(학과) 양쪽과 동시에 연결되는 <strong>중심 테이블</strong> 역할을 하고 있다는 걸 조건식 순서에서 확인할 수 있다.</li>
</ul>
<hr />
<h1 id="4-left-join--관련-데이터가-없는-경우도-포함하기">4. <code>LEFT JOIN</code> — 관련 데이터가 없는 경우도 포함하기</h1>
<p><strong>SQL</strong> · 담당 교수가 배정 안 된 예체능 과목 찾기</p>
<pre><code class="language-sql">SELECT a.CLASS_NAME AS 과목이름, d.DEPARTMENT_NAME AS 학과이름
FROM tb_class a
LEFT JOIN tb_class_professor b ON a.CLASS_NO = b.CLASS_NO   -- ①
JOIN tb_department d ON a.DEPARTMENT_NO = d.DEPARTMENT_NO
WHERE 1=1
AND d.CATEGORY = '예체능'
AND b.CLASS_NO IS NULL;                                       -- ②</code></pre>
<ul>
<li>① <code>tb_class</code>를 기준으로 <code>tb_class_professor</code>를 <code>LEFT JOIN</code>한다 — 담당 교수가 배정 안 된 과목도 <code>tb_class_professor</code> 쪽 값이 전부 <code>NULL</code>인 채로 결과에 남는다. 만약 그냥 <code>JOIN</code>(내부 조인)이었다면, 교수가 아예 없는 과목은 애초에 결과에서 사라져버려서 이 문제를 풀 수 없다.</li>
<li>② <code>b.CLASS_NO IS NULL</code>: <code>LEFT JOIN</code>으로 채워진 자리가 <code>NULL</code>인 행만 걸러내면, 그게 바로 <strong>&quot;교수가 한 명도 배정 안 된 과목&quot;</strong>이다.</li>
</ul>
<p><strong>SQL</strong> · 지도교수가 없으면 &quot;미지정&quot;으로 표시</p>
<pre><code class="language-sql">SELECT a.STUDENT_NAME AS 학생이름,
       NVL(b.PROFESSOR_NAME, '지도교수 미지정') AS 지도교수
FROM tb_student a
LEFT JOIN tb_professor b ON b.PROFESSOR_NO = a.COACH_PROFESSOR_NO
JOIN tb_department c ON c.DEPARTMENT_NO = a.DEPARTMENT_NO
WHERE c.DEPARTMENT_NAME = '서반아어학과'
ORDER BY a.STUDENT_NO DESC;</code></pre>
<ul>
<li>위 3번 예제(<code>IS NULL</code>로 걸러내기)와 달리, 이번엔 지도교수가 없는 학생도 <strong>결과에 남기되</strong> <code>NVL()</code>로 화면에는 &quot;지도교수 미지정&quot;이라는 문구를 대신 보여준다. <code>LEFT JOIN</code> 자체는 같지만, 그 뒤에 <strong>거를지(<code>IS NULL</code>) 대체할지(<code>IFNULL</code>)</strong>로 활용 방식이 갈린다는 걸 두 문제를 비교하며 배웠다.</li>
</ul>
<hr />
<h1 id="5-self-join--같은-테이블을-자기-자신과-연결하기">5. <code>SELF JOIN</code> — 같은 테이블을 자기 자신과 연결하기</h1>
<pre><code class="language-sql">SELECT E.EMP_NAME, M.EMP_NAME, S.EMP_NAME
FROM employee E
LEFT JOIN employee M ON (E.MGR_ID = M.EMP_ID)   -- ①
LEFT JOIN employee S ON (M.MGR_ID = S.EMP_ID);  -- ②</code></pre>
<ul>
<li><code>employee</code> 테이블 안에는 &quot;사수(<code>MGR_ID</code>)&quot;라는 컬럼이 <strong>같은 테이블의 다른 행</strong>을 가리킨다. 이럴 때 테이블을 두 번(또는 세 번) 등장시키되, <strong>매번 다른 별칭</strong>(<code>E</code>, <code>M</code>, <code>S</code>)을 붙여서 서로 다른 테이블인 것처럼 취급한다.</li>
<li>① 사원(<code>E</code>)의 <code>MGR_ID</code>로 <code>employee</code>를 다시 조회(<code>M</code>)해서 <strong>사수</strong>를 찾는다.</li>
<li>② 사수(<code>M</code>)의 <code>MGR_ID</code>로 또 한 번 <code>employee</code>를 조회(<code>S</code>)해서 <strong>사수의 사수</strong>까지 찾는다.</li>
<li>사수가 없는 사원(최고 직급)도 결과에서 빠지지 않도록 <code>LEFT JOIN</code>을 썼다 — 여기서도 4번 섹션의 <code>LEFT JOIN</code> 원리가 그대로 이어진다.</li>
</ul>
<hr />
<h1 id="6-exists--있는지-없는지만-확인하기">6. <code>EXISTS</code> — &quot;있는지 없는지&quot;만 확인하기</h1>
<pre><code class="language-sql">SELECT U.USERID, U.NAME, U.ADDR, CONCAT(U.mobile1, '-', U.MOBILE2)
FROM USERTBL U
WHERE EXISTS (
    SELECT *
    FROM BUYTBL B
    WHERE U.USERID = B.USERID
);</code></pre>
<ul>
<li><code>EXISTS (서브쿼리)</code>는 그 서브쿼리가 <strong>한 건이라도 결과를 반환하면</strong> <code>true</code>, 아무것도 없으면 <code>false</code>가 된다. 실제로 서브쿼리 결과값 자체를 쓰는 게 아니라, &quot;있냐 없냐&quot;만 판단에 쓰인다.</li>
<li>이 쿼리는 &quot;구매 기록이 하나라도 있는 회원만&quot; 조회한다. 같은 걸 <code>LEFT JOIN</code> + <code>WHERE ... IS NOT NULL</code> 방식으로도 만들 수 있지만(오늘 실습에 그 반대 버전 — <code>LEFT JOIN</code> 후 <code>PRODNAME IS NULL</code>로 <strong>구매 이력이 없는</strong> 회원을 찾는 예제도 있었다), <code>EXISTS</code>는 &quot;존재 여부&quot;라는 의도를 코드로 더 직접적으로 드러낸다는 차이가 있다.</li>
</ul>
<hr />
<h1 id="7-실전-문제--erd를-따라가며-join-조합하기">7. 실전 문제 — ERD를 따라가며 JOIN 조합하기</h1>
<table>
<thead>
<tr>
<th>번호</th>
<th>문제 요지</th>
<th>관련된 ERD 관계</th>
</tr>
</thead>
<tbody><tr>
<td>3</td>
<td>특정 지역+연대 학생 조회</td>
<td>(JOIN 없음, <code>WHERE</code>+<code>LIKE</code> 복합조건)</td>
</tr>
<tr>
<td>4</td>
<td>법학과 교수를 나이순 정렬</td>
<td><code>TB_DEPARTMENT</code> → <code>TB_PROFESSOR</code> (학과코드 조회 후 필터)</td>
</tr>
<tr>
<td>6</td>
<td>학번·이름·학과이름</td>
<td><code>TB_STUDENT</code> ↔ <code>TB_DEPARTMENT</code> (1:N)</td>
</tr>
<tr>
<td>7</td>
<td>과목이름 + 과목의 학과이름</td>
<td><code>TB_CLASS</code> ↔ <code>TB_DEPARTMENT</code> (1:N)</td>
</tr>
<tr>
<td>8</td>
<td>과목별 담당교수</td>
<td><code>TB_CLASS</code> ↔ <code>TB_CLASS_PROFESSOR</code> ↔ <code>TB_PROFESSOR</code> (N:M)</td>
</tr>
<tr>
<td>9</td>
<td>인문사회 계열 과목의 교수</td>
<td>8번 + <code>TB_DEPARTMENT</code> 조건 추가</td>
</tr>
<tr>
<td>10</td>
<td>음악학과 학생들의 전체 평점</td>
<td><code>TB_STUDENT</code> ↔ <code>TB_GRADE</code> (1:N) + <code>GROUP BY</code></td>
</tr>
<tr>
<td>11</td>
<td>특정 학생의 학과·지도교수 정보</td>
<td><code>TB_DEPARTMENT</code> ↔ <code>TB_STUDENT</code> ↔ <code>TB_PROFESSOR</code></td>
</tr>
<tr>
<td>12</td>
<td>특정 과목 수강 학생·학기</td>
<td><code>TB_STUDENT</code> ↔ <code>TB_GRADE</code> ↔ <code>TB_CLASS</code></td>
</tr>
<tr>
<td>13</td>
<td>담당교수 미배정 예체능 과목</td>
<td><code>TB_CLASS</code> ↔(LEFT)↔ <code>TB_CLASS_PROFESSOR</code>, <code>IS NULL</code></td>
</tr>
<tr>
<td>14</td>
<td>서반아어학과 학생의 지도교수(미지정 포함)</td>
<td><code>TB_STUDENT</code> ↔(LEFT)↔ <code>TB_PROFESSOR</code>, <code>IFNULL</code></td>
</tr>
</tbody></table>
<hr />
<h1 id="✨-새롭게-알게-된-점">✨ 새롭게 알게 된 점</h1>
<ul>
<li>ERD의 화살표(FK 관계)를 따라가면, 어떤 컬럼끼리 <code>JOIN</code> 조건을 걸어야 하는지 거의 그대로 알 수 있다.</li>
<li>N:M 관계는 테이블 두 개만으로는 표현이 안 되고, 반드시 <strong>중간(연결) 테이블</strong>이 필요하다.</li>
<li><code>LEFT JOIN</code> 뒤에 <code>IS NULL</code>을 붙이면 &quot;관련 데이터가 없는 것만&quot; 찾을 수 있고, <code>IFNULL</code>을 붙이면 &quot;없어도 포함하되 대체 문구로 보여주는&quot; 것으로 활용 방향이 갈린다.</li>
<li><code>SELF JOIN</code>은 같은 테이블에 서로 다른 별칭을 붙이는 것만으로, 계층 관계(사원-사수-사수의 사수)를 표현할 수 있다.</li>
<li><code>EXISTS</code>는 서브쿼리의 결과값이 아니라 &quot;존재 여부&quot;만 판단에 쓴다는 점에서 일반 서브쿼리와 다르다.</li>
</ul>
<hr />
<h1 id="🤔-헷갈렸던-점">🤔 헷갈렸던 점</h1>
<ul>
<li><strong>암묵적 조인과 ANSI 조인 중 언제 뭘 봐야 하는지</strong>: 오늘 실습 문제 중 옛날 방식(<code>FROM a, b WHERE</code>)과 새 방식(<code>JOIN ~ ON</code>)이 섞여 있어서, 같은 결과를 내는 두 문법을 눈으로 구분하는 데 시간이 걸렸다.</li>
<li><strong><code>LEFT JOIN</code>에서 <code>ON</code>과 <code>WHERE</code>에 조건을 나눠 쓰는 이유</strong>: 13번 문제처럼 <code>LEFT JOIN</code>의 <code>ON</code> 절 안에 조건을 넣는 경우와, <code>WHERE</code>에 조건을 넣는 경우가 결과가 다를 수 있다는 걸 어렴풋이 느꼈는데 아직 명확히 정리는 못 했다.</li>
<li><strong><code>SELF JOIN</code>에서 별칭을 헷갈리지 않고 따라가기</strong>: <code>E</code>, <code>M</code>, <code>S</code>처럼 같은 테이블이 여러 번 등장하니, 지금 보는 컬럼이 어느 별칭 소속인지 계속 확인해야 했다.</li>
</ul>
<hr />
<h1 id="다음에-할-일">다음에 할 일</h1>
<ul>
<li><input disabled="" type="checkbox" /> <code>LEFT JOIN</code>의 <code>ON</code> 절 조건과 <code>WHERE</code> 절 조건의 결과 차이를 직접 실험해서 비교해보기</li>
<li><input disabled="" type="checkbox" /> <code>NATURAL JOIN</code>(오늘 문법만 보고 직접 실습은 못함)을 실제로 써보기</li>
<li><input disabled="" type="checkbox" /> <code>EXISTS</code>와 <code>LEFT JOIN + IS NULL/IS NOT NULL</code> 두 방식을 같은 문제에 적용해서 결과 비교해보기</li>
</ul>
<hr />
<h1 id="📚-참고-자료">📚 참고 자료</h1>
<ul>
<li>오늘 실습에 사용한 ERD (첨부 이미지)</li>
</ul>
<hr />
<h1 id="💡-오늘의-회고">💡 오늘의 회고</h1>
<p>오늘은 ERD를 먼저 눈에 익히고 문제를 푸니, &quot;이 데이터를 가져오려면 어느 테이블들을 거쳐야 하는지&quot;가 훨씬 명확하게 보였다. 특히 <code>TB_CLASS</code>와 <code>TB_PROFESSOR</code>처럼 직접 연결되지 않은 두 테이블을, 중간 테이블(<code>TB_CLASS_PROFESSOR</code>)을 거쳐 이어야 한다는 걸 그림으로 먼저 확인하고 SQL을 짜니 헤매는 시간이 줄었다.</p>