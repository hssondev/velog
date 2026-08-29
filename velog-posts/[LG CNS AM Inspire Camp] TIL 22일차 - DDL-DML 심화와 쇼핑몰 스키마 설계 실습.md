<blockquote>
<p><strong>한 줄 요약</strong></p>
<p><code>FOREIGN KEY</code>의 참조무결성을 에러로 직접 겪어보고, <code>ALTER TABLE</code>로 나중에 제약조건 추가하기, <code>VIEW</code>, <code>SEQUENCE</code>를 배운 뒤, 쇼핑몰 ERD를 기준으로 <code>customers</code>/<code>orders</code>/<code>products</code>/<code>orderdetail</code> 4개 테이블을 직접 설계하고 주문 데이터를 입력·조회하는 실전 과제를 풀었다.</p>
</blockquote>
<hr />
<h1 id="📚-오늘-학습한-커리큘럼">📚 오늘 학습한 커리큘럼</h1>
<pre><code>FOREIGN KEY와 참조무결성 (TB_JOB → TB_DEPT → TB_EMP)
        ↓
ALTER TABLE — 나중에 제약조건 추가/변경하기
        ↓
제약조건 확인하는 방법
        ↓
VIEW — 가상의 논리적 테이블
        ↓
DML 심화 — UPDATE의 서브쿼리, DEFAULT, 참조무결성
        ↓
SEQUENCE vs AUTO_INCREMENT
        ↓
DDL 실습 15문제 (테이블/뷰 생성)
        ↓
쇼핑몰 ERD 실전 과제 — 스키마 설계 + 주문 데이터 입력/조회</code></pre><h1 id="📌-오늘의-학습-키워드">📌 오늘의 학습 키워드</h1>
<ul>
<li><code>FOREIGN KEY</code>, <code>ON DELETE/UPDATE CASCADE</code>, <code>NO ACTION</code></li>
<li><code>ALTER TABLE ADD/DROP/MODIFY/CHANGE COLUMN</code></li>
<li><code>VIEW</code>, <code>WITH CHECK OPTION</code></li>
<li><code>UPDATE ~ SET ~ WHERE</code>(서브쿼리 포함), <code>DEFAULT</code> 되돌리기</li>
<li><code>SEQUENCE</code>, <code>NEXTVAL</code>, <code>AUTO_INCREMENT</code></li>
</ul>
<hr />
<h1 id="0-개념-정리-키워드-사전">0. 개념 정리 (키워드 사전)</h1>
<blockquote>
<p><strong>한 줄 요약</strong></p>
<p><code>FOREIGN KEY</code>는 &quot;부모가 없으면 자식도 존재할 수 없다&quot;는 규칙이고, <code>VIEW</code>는 &quot;자주 쓰는 쿼리를 테이블처럼 저장해두는 것&quot;이며, <code>SEQUENCE</code>는 &quot;중복 없는 번호를 자동으로 채번해주는 장치&quot;다.</p>
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
<td><strong><code>FOREIGN KEY</code></strong></td>
<td>다른 테이블의 PK를 참조하는 컬럼, 참조무결성을 강제</td>
<td>부모 테이블에 없는 값은 자식 테이블에 넣을 수 없음</td>
</tr>
<tr>
<td><strong><code>ON DELETE/UPDATE CASCADE</code></strong></td>
<td>부모 행이 삭제/수정되면, 자식 행도 함께 삭제/수정됨</td>
<td></td>
</tr>
<tr>
<td><strong><code>ON DELETE/UPDATE NO ACTION</code></strong></td>
<td>부모 행을 삭제/수정하려 해도, 참조하는 자식이 있으면 <strong>막아버림</strong>(기본 동작)</td>
<td></td>
</tr>
<tr>
<td><strong><code>ALTER TABLE</code></strong></td>
<td>이미 만들어진 테이블의 구조(컬럼, 제약조건)를 나중에 바꾸는 명령</td>
<td><code>ADD</code>, <code>DROP</code>, <code>MODIFY</code>, <code>CHANGE</code>, <code>RENAME</code></td>
</tr>
<tr>
<td><strong><code>MODIFY COLUMN</code></strong></td>
<td>컬럼의 <strong>타입</strong>만 바꿈(이름은 그대로)</td>
<td></td>
</tr>
<tr>
<td><strong><code>CHANGE COLUMN</code></strong></td>
<td>컬럼의 <strong>이름과 타입</strong>을 함께 바꿈</td>
<td></td>
</tr>
<tr>
<td><strong><code>VIEW</code></strong></td>
<td>서브쿼리를 이름 붙여 저장해두고, 테이블처럼 조회할 수 있게 만든 <strong>가상의 논리적 테이블</strong></td>
<td>실제 데이터를 저장하진 않음(읽기 전용에 가까움)</td>
</tr>
<tr>
<td><strong><code>WITH CHECK OPTION</code></strong></td>
<td><code>VIEW</code>를 통해 데이터를 수정할 때, <code>VIEW</code>의 조건(<code>WHERE</code>)에 안 맞는 값으로는 못 바꾸게 막음</td>
<td></td>
</tr>
<tr>
<td><strong><code>SEQUENCE</code></strong></td>
<td>호출할 때마다 중복 없는 다음 번호를 내어주는 객체</td>
<td><code>NEXTVAL()</code>로 다음 값을 꺼냄</td>
</tr>
<tr>
<td><strong><code>AUTO_INCREMENT</code></strong></td>
<td>컬럼에 값을 안 넣으면 자동으로 1씩 증가하는 번호가 채워짐</td>
<td><code>SEQUENCE</code>보다 간단하지만 유연성은 덜함</td>
</tr>
</tbody></table>
<hr />
<h1 id="1-foreign-key와-참조무결성--에러로-직접-확인하기">1. <code>FOREIGN KEY</code>와 참조무결성 — 에러로 직접 확인하기</h1>
<p><strong>SQL</strong> · 부모 테이블 먼저 만들기</p>
<pre><code class="language-sql">CREATE TABLE TB_JOB (JOB_ID VARCHAR(10), JOB_TITLE VARCHAR(50), PRIMARY KEY(JOB_ID));
CREATE TABLE TB_DEPT (DEPT_ID VARCHAR(10), DEPT_TITLE VARCHAR(50), PRIMARY KEY(DEPT_ID));

CREATE TABLE TB_EMP (
    EMP_ID    VARCHAR(10) PRIMARY KEY,
    EMP_NAME  VARCHAR(50) NOT NULL,
    HIRE_DATE DATE DEFAULT SYSDATE(),
    DEPT_ID   VARCHAR(10) NOT NULL REFERENCES TB_DEPT (DEPT_ID),
    JOB_ID    VARCHAR(10) NOT NULL
);</code></pre>
<ul>
<li><code>DEPT_ID VARCHAR(10) NOT NULL REFERENCES TB_DEPT (DEPT_ID)</code>: 컬럼 선언 뒤에 바로 <code>REFERENCES 부모테이블(부모컬럼)</code>을 붙이면, <code>FOREIGN KEY</code>로 지정한 것과 같은 효과를 낸다 — 어제 배운 <code>PRIMARY KEY</code>처럼, 컬럼 옆에 바로 붙이는 방식(컬럼 제약조건)의 <code>FOREIGN KEY</code> 버전이다.</li>
</ul>
<p><strong>SQL</strong> · 실제로 겪은 에러 두 가지</p>
<pre><code class="language-sql">-- ❌ ERROR: DEPT_ID, JOB_ID가 NOT NULL인데 NULL을 넣으려 함
INSERT INTO TB_EMP(EMP_ID, EMP_NAME, DEPT_ID, JOB_ID)
VALUES('100', 'JSLIM', NULL, NULL);

-- ❌ ERROR: DEPT_ID '30'이 TB_DEPT에 없음 (참조무결성 위반)
INSERT INTO TB_EMP(EMP_ID, EMP_NAME, DEPT_ID, JOB_ID)
VALUES('100', 'JSLIM', '30', 'J3');

-- ✅ 정상: TB_DEPT에 실제로 존재하는 '10'을 넣음
INSERT INTO TB_EMP(EMP_ID, EMP_NAME, DEPT_ID, JOB_ID)
VALUES('100', 'JSLIM', '10', 'J1');</code></pre>
<ul>
<li>두 번째 에러가 참조무결성의 핵심이다: <code>TB_DEPT</code>에 <code>DEPT_ID='30'</code>인 부서가 <strong>존재하지 않는데</strong>, <code>TB_EMP</code>에 그 값을 가진 사원을 넣으려 하면 DB가 막는다. <strong>&quot;자식은 반드시 존재하는 부모를 가리켜야 한다&quot;</strong>는 규칙을 DB가 강제로 지켜주는 것이다.</li>
</ul>
<p><strong>SQL</strong> · 부모를 지우려고 하면?</p>
<pre><code class="language-sql">DELETE FROM TB_DEPT WHERE DEPT_ID = '10';</code></pre>
<ul>
<li><code>TB_EMP</code>에 <code>DEPT_ID='10'</code>을 참조하는 사원이 이미 있는 상태에서 이 삭제를 실행하면, 기본 동작(<code>NO ACTION</code>)에서는 <strong>삭제가 거부된다.</strong> 주석 처리돼 있던 <code>ON DELETE CASCADE</code> 옵션을 걸어뒀다면, 부모(부서)가 삭제될 때 그 부서에 속한 사원들도 <strong>함께 삭제</strong>됐을 것이다 — 어떤 옵션을 쓰느냐에 따라 &quot;부모를 못 지우게 막을지&quot; &quot;부모를 지우면 자식도 같이 지울지&quot;가 갈린다.</li>
</ul>
<hr />
<h1 id="2-alter-table--나중에-제약조건-추가변경하기">2. <code>ALTER TABLE</code> — 나중에 제약조건 추가/변경하기</h1>
<p><strong>SQL</strong> · 제약조건 없이 만들고, 나중에 하나씩 추가</p>
<pre><code class="language-sql">CREATE TABLE TB_EMPLOYEE (
    EMP_ID    VARCHAR(10),
    EMP_NAME  VARCHAR(50) NOT NULL,
    SALARY    INT CHECK(SALARY &gt; 0),
    GENDER    CHAR(1) CHECK(GENDER IN ('F','M')),
    HIRE_DATE DATE DEFAULT SYSDATE(),
    JOB_ID    VARCHAR(10),
    DEPT_ID   VARCHAR(10)
);

-- 나중에 PRIMARY KEY 추가
ALTER TABLE TB_EMPLOYEE ADD CONSTRAINT PRIMARY KEY(EMP_ID);

-- DEPT_ID를 NOT NULL로 바꾸고 FOREIGN KEY 추가
ALTER TABLE TB_EMPLOYEE MODIFY COLUMN DEPT_ID VARCHAR(10) NOT NULL;
ALTER TABLE TB_EMPLOYEE ADD CONSTRAINT FOREIGN KEY (DEPT_ID) REFERENCES tb_dept(DEPT_ID);</code></pre>
<ul>
<li><code>CREATE TABLE</code> 시점에 제약조건을 다 못 넣었더라도, <code>ALTER TABLE</code>로 언제든 나중에 추가할 수 있다는 걸 확인했다. <code>ADD CONSTRAINT</code>로 제약을 붙이고, <code>MODIFY COLUMN</code>으로 타입/제약(예: <code>NOT NULL</code>)을 바꾼다.</li>
</ul>
<hr />
<h1 id="3-제약조건이-뭐가-걸려있는지-확인하는-법">3. 제약조건이 뭐가 걸려있는지 확인하는 법</h1>
<pre><code class="language-sql">-- 방법 1: 테이블 생성 구문 그대로 보기
SHOW CREATE TABLE TB_EMPLOYEE;

-- 방법 2: 시스템 테이블에서 제약조건 목록만 조회
SELECT CONSTRAINT_NAME, CONSTRAINT_TYPE, TABLE_NAME
FROM information_schema.table_constraints
WHERE TABLE_SCHEMA = 'TABLEDB'
AND TABLE_NAME = 'TB_EMPLOYEE';</code></pre>
<ul>
<li><code>SHOW CREATE TABLE</code>은 테이블을 만들 때 썼던 <code>CREATE TABLE</code> 구문을 통째로 다시 보여준다.</li>
<li><code>information_schema.table_constraints</code>는 MySQL/MariaDB가 내부적으로 관리하는 <strong>시스템 테이블</strong>로, 실무에서는 이렇게 쿼리로 제약조건 목록을 뽑아 확인한다고 배웠다.</li>
</ul>
<hr />
<h1 id="4-view--자주-쓰는-쿼리를-테이블처럼-저장하기">4. <code>VIEW</code> — 자주 쓰는 쿼리를 테이블처럼 저장하기</h1>
<pre><code class="language-sql">CREATE OR REPLACE VIEW EMP_VIEW(NAME, DEPT)
AS
SELECT E.EMP_NAME, D.DEPT_NAME
FROM DEPARTMENT D
JOIN EMPLOYEE E ON (D.DEPT_ID = E.DEPT_ID)
WHERE E.DEPT_ID = '90';

SELECT * FROM EMP_VIEW;</code></pre>
<ul>
<li><code>VIEW</code>는 실제로 데이터를 따로 저장하지 않고, <strong><code>SELECT</code> 할 때마다 뒤에 숨겨둔 서브쿼리를 다시 실행</strong>해서 결과를 보여주는 &quot;가상의 테이블&quot;이다. <code>EMP_VIEW(NAME, DEPT)</code>처럼 컬럼 별칭을 아예 뷰 이름 옆에 지정할 수도 있다.</li>
<li><strong>왜 쓰는가</strong>: 복잡한 조인 쿼리를 매번 새로 작성하지 않고 <code>SELECT * FROM EMP_VIEW</code>처럼 짧게 재사용할 수 있고, 원본 테이블의 특정 컬럼만 보여주게 만들어서 <strong>보안(민감한 컬럼 숨기기)</strong>에도 활용할 수 있다고 배웠다.</li>
</ul>
<p><strong>SQL·개념</strong> · <code>WITH CHECK OPTION</code></p>
<pre><code class="language-sql">CREATE OR REPLACE VIEW VW_학생일반정보(학번, 학생이름, 주소)
AS SELECT S.STUDENT_NO, S.STUDENT_NAME, S.STUDENT_ADDRESS
FROM tb_student S
WHERE 학과조건
WITH CHECK OPTION;</code></pre>
<ul>
<li><code>VIEW</code>도 <code>UPDATE</code>로 데이터를 수정할 수 있는데, <code>WITH CHECK OPTION</code>을 붙이면 <strong>그 뷰의 <code>WHERE</code> 조건에서 벗어나는 값으로는 수정할 수 없게</strong> 막아준다. 예를 들어 &quot;국문과 학생만&quot; 보여주는 뷰에서, 어떤 학생의 학과를 다른 학과로 바꾸는 수정을 시도하면 거부된다 — 뷰가 보여주는 범위와, 뷰로 수정 가능한 범위를 <strong>일치</strong>시켜주는 옵션이라고 이해했다.</li>
</ul>
<hr />
<h1 id="5-dml-심화--update의-다양한-활용">5. DML 심화 — <code>UPDATE</code>의 다양한 활용</h1>
<p><strong>SQL</strong> · 서브쿼리로 다른 사람과 같은 값으로 맞추기</p>
<pre><code class="language-sql">UPDATE employee
SET JOB_ID  = (SELECT JOB_ID  FROM employee WHERE EMP_NAME = '성해교'),
    DEPT_ID = (SELECT DEPT_ID FROM employee WHERE EMP_NAME = '성해교')
WHERE EMP_NAME = '심하균';</code></pre>
<ul>
<li><code>SET</code> 절에도 서브쿼리를 쓸 수 있다. &quot;성해교의 직급/부서 값을 그대로 가져와서 심하균에게 넣어라&quot;는 걸, 값을 미리 조회해서 손으로 옮겨 적지 않고 SQL 한 번으로 처리했다.</li>
</ul>
<p><strong>SQL</strong> · <code>DEFAULT</code>로 원래 기본값으로 되돌리기</p>
<pre><code class="language-sql">UPDATE employee SET MARRIAGE = DEFAULT WHERE EMP_NAME = '한선기';</code></pre>
<ul>
<li><code>SET 컬럼 = DEFAULT</code>: 그 컬럼에 미리 정의해둔 기본값으로 되돌린다. &quot;결혼 여부를 초기 상태로 리셋&quot;하는 의도로 쓰였다.</li>
</ul>
<p>⚠️ <strong>참조무결성 때문에 막힌 <code>UPDATE</code></strong></p>
<pre><code class="language-sql">-- ❌ ERROR
UPDATE employee SET DEPT_ID = NULL WHERE EMP_ID = '100';</code></pre>
<ul>
<li><code>DEPT_ID</code>가 <code>NOT NULL</code> 제약이 걸린 <code>FOREIGN KEY</code> 컬럼이라, <code>NULL</code>로 바꾸려는 시도 자체가 막혔다. 1번 섹션에서 본 참조무결성이 <code>INSERT</code>뿐 아니라 <code>UPDATE</code>에도 똑같이 적용된다는 걸 확인했다.</li>
</ul>
<hr />
<h1 id="6-sequence-vs-auto_increment--번호-자동-채번">6. <code>SEQUENCE</code> vs <code>AUTO_INCREMENT</code> — 번호 자동 채번</h1>
<pre><code class="language-sql">CREATE SEQUENCE SEQ_TEST START WITH 1000 INCREMENT BY 2 MAXVALUE 1005;
SELECT NEXTVAL(SEQ_TEST);   -- 1000
SELECT NEXTVAL(SEQ_TEST);   -- 1002 (2씩 증가)

CREATE TABLE TB_TEMP(
    ORDER_ID INT DEFAULT (NEXTVAL(SEQ_TEST)) PRIMARY KEY,
    CUSTOMER_NAME VARCHAR(50)
);</code></pre>
<ul>
<li><code>SEQUENCE</code>는 <strong>시작값(<code>START WITH</code>), 증가폭(<code>INCREMENT BY</code>), 최댓값(<code>MAXVALUE</code>)</strong>을 세밀하게 설정할 수 있는 채번 전용 객체다. <code>DEFAULT (NEXTVAL(...))</code>로 컬럼 기본값에 연결해두면, <code>INSERT</code> 시 값을 안 넣어도 자동으로 다음 번호가 채워진다.</li>
</ul>
<pre><code class="language-sql">CREATE TABLE TB_AUTO_TEMP(
    ORDER_ID INT AUTO_INCREMENT PRIMARY KEY,
    CUSTOMER_NAME VARCHAR(50)
);</code></pre>
<ul>
<li><code>AUTO_INCREMENT</code>는 <code>SEQUENCE</code>보다 훨씬 간단하게, 컬럼 선언에 키워드 하나만 붙이면 자동으로 1씩 증가하는 번호가 채워진다. 대신 시작값·증가폭 같은 세부 설정은 <code>SEQUENCE</code>만큼 유연하지 않다는 차이가 있다.</li>
</ul>
<hr />
<h1 id="7-ddl-실습-문제--tb_categorytb_class_type-다듬어가기">7. DDL 실습 문제 — <code>TB_CATEGORY</code>/<code>TB_CLASS_TYPE</code> 다듬어가기</h1>
<p>오늘 실습에서는 테이블을 한 번에 완벽하게 만들지 않고, <strong>문제를 풀 때마다 <code>ALTER TABLE</code>로 조금씩 개선</strong>해나가는 흐름이었다.</p>
<pre><code class="language-sql">-- 처음엔 제약조건 없이
CREATE TABLE TB_CATEGORY(NAME VARCHAR(10), USE_YN CHAR(1) DEFAULT 'Y');
CREATE TABLE TB_CLASS_TYPE(NO VARCHAR(5) PRIMARY KEY, NAME VARCHAR(10));

-- PRIMARY KEY 추가, NOT NULL 지정, 컬럼명·길이 변경, 제약조건 이름까지 바꾸기
ALTER TABLE tb_category ADD PRIMARY KEY (NAME);
ALTER TABLE tb_category MODIFY NAME VARCHAR(20);
ALTER TABLE tb_category CHANGE NAME CATEGORY_NAME VARCHAR(20);
ALTER TABLE tb_category
    DROP CONSTRAINT PRIMARY KEY,
    ADD CONSTRAINT PK_CATEGORY_NAME PRIMARY KEY (CATEGORY_NAME);</code></pre>
<ul>
<li><code>MODIFY</code>(타입만), <code>CHANGE</code>(이름+타입), <code>DROP CONSTRAINT</code> + <code>ADD CONSTRAINT</code>(제약조건 이름 지정)까지, <code>ALTER TABLE</code>의 여러 형태를 한 테이블에 순서대로 적용해보며 익혔다.</li>
<li>이후 <code>TB_DEPARTMENT</code>에 <code>TB_CATEGORY</code>를 참조하는 <code>FOREIGN KEY</code>를 추가하고, 학생 정보 뷰(<code>VW_학생일반정보</code>)·지도교수 면담 뷰(<code>VW_지도면담</code>)·학과별 학생수 뷰(<code>VW_학과별학생수</code>) 세 개를 만들어 뷰 실습도 함께 진행했다.</li>
</ul>
<pre><code class="language-sql">-- Top-3 조회 (LIMIT 활용, 19일차에 배운 것의 응용)
SELECT C.CLASS_NO AS 과목번호, C.CLASS_NAME AS 과목이름, COUNT(*) AS `누적수강생수(명)`
FROM TB_CLASS C
JOIN TB_GRADE G ON (C.CLASS_NO = G.CLASS_NO)
WHERE SUBSTRING(G.TERM_NO, 1, 4) BETWEEN '2007' AND '2009'
GROUP BY C.CLASS_NO, C.CLASS_NAME
ORDER BY COUNT(*) DESC, C.CLASS_NO
LIMIT 3;</code></pre>
<hr />
<h1 id="8-실전-과제--쇼핑몰-erd로-스키마-설계하기">8. 실전 과제 — 쇼핑몰 ERD로 스키마 설계하기</h1>
<p>첨부된 과제 스펙(쇼핑몰 ERD)을 보고 <code>customers</code>/<code>orders</code>/<code>products</code>/<code>orderdetail</code> 4개 테이블을 직접 설계했다. 관계는 <code>customers</code> 1 : N <code>orders</code>, <code>products</code> 1 : N <code>orderdetail</code>, <code>orders</code>와 <code>products</code>가 <code>orderdetail</code>(중간 테이블)로 N:M 연결되는 구조 — 18~19일차에 배운 &quot;중간 테이블로 N:M 표현하기&quot;가 그대로 적용된다.</p>
<pre><code class="language-sql">CREATE TABLE customers (
    cno     int(5)  PRIMARY KEY,
    cname   varchar(10) not null,
    address varchar(50) not null,
    email   varchar(20) not null,
    phone   varchar(10) not null
);</code></pre>
<pre><code class="language-sql">CREATE TABLE products (
    pno   int(5)  PRIMARY KEY,
    pname varchar(20),
    cost  int(8) DEFAULT 0 not null,
    stock int(5) DEFAULT 0 not null
);</code></pre>
<pre><code class="language-sql">CREATE TABLE orders (
    orderno            int(10)    PRIMARY KEY
    ,orderdate        date  DEFAULT sysdate() not null
    ,address        varchar(50) not null
    ,phone            varchar(20) not null
    ,status            VARCHAR(20) CHECK( status IN ('결제완료','배송중','배송완료'))
    ,cno            int    (5) not null    
    ,FOREIGN KEY(cno) REFERENCES customers (cno) 
);</code></pre>
<pre><code class="language-sql">CREATE TABLE orderdetail (
    orderno            int(10)    
    ,pno            int(5)  
    ,qty            int(5) DEFAULT 0 
    ,cost            int(8) DEFAULT 0
    ,PRIMARY KEY (orderno, pno),
    FOREIGN KEY (orderno) REFERENCES orders (orderno),
    FOREIGN KEY (pno) REFERENCES products (pno)
);</code></pre>
<p><code>orders.cno</code>를 <code>customers.cno</code>를 참조하는 <code>FOREIGN KEY</code>로, <code>orderdetail</code>은 <code>(orderno, pno)</code>를 <strong>복합 기본키</strong>로 두고 각각 <code>orders</code>, <code>products</code>를 참조하는 <code>FOREIGN KEY</code> 두 개를 걸었다.</p>
<hr />
<h1 id="9-실전-과제--주문-데이터-입력하고-결과-검증하기">9. 실전 과제 — 주문 데이터 입력하고 결과 검증하기</h1>
<p><strong>SQL</strong> · 김철수 주문 (문제 4~5)</p>
<pre><code class="language-sql">INSERT INTO orders (orderno, orderdate, address, phone, status, cno)
VALUES (1, CURRENT_DATE - INTERVAL 3 DAY, '서울 강남구', '899-6666', '결제완료', 101);

INSERT INTO orderdetail (orderno, pno, qty, cost)
VALUES (1, 1001, 50, 1000);

UPDATE products SET stock = stock - 50 WHERE pno = 1001;</code></pre>
<ul>
<li><code>CURRENT_DATE - INTERVAL 3 DAY</code>: &quot;3일 전 날짜&quot;를 만드는 표현. 과제에서 요구한 &quot;오늘날짜-3&quot;과 정확히 일치한다.</li>
<li><code>stock = stock - 50</code>: 기존 재고에서 50을 <strong>빼는</strong> 방식으로 갱신했다. 결과적으로 <code>200 - 50 = 150</code>이 되어, 과제에서 요구한 &quot;150개로 변경&quot;과 맞다.</li>
</ul>
<p><strong>전체 주문 목록(문제 10) 검증</strong></p>
<pre><code class="language-sql">SELECT o.orderdate, c.cname, o.address, o.phone, o.status,
       p.pname, od.cost, od.qty, od.cost * od.qty AS amount
FROM orders o
JOIN customers c ON o.cno = c.cno
JOIN orderdetail od ON o.orderno = od.orderno
JOIN products p ON od.pno = p.pno
ORDER BY o.orderno;</code></pre>
<table>
<thead>
<tr>
<th>계산</th>
<th>값</th>
<th>과제 스펙과 비교</th>
</tr>
</thead>
<tbody><tr>
<td>삼양라면 1000 × 50</td>
<td>50,000</td>
<td>✅ 일치</td>
</tr>
<tr>
<td>새우깡 1500 × 100</td>
<td>150,000</td>
<td>✅ 일치</td>
</tr>
<tr>
<td>월드콘 2000 × 150</td>
<td>300,000</td>
<td>✅ 일치</td>
</tr>
<tr>
<td>빼빼로 2000 × 100</td>
<td>200,000</td>
<td>✅ 일치</td>
</tr>
<tr>
<td>코카콜라 1800 × 50</td>
<td>90,000</td>
<td>✅ 일치</td>
</tr>
</tbody></table>
<p><strong>일별 매출(문제 11) 검증</strong></p>
<pre><code class="language-sql">SELECT o.orderdate, SUM(od.cost * od.qty) AS daily_sales
FROM orders o
JOIN orderdetail od ON o.orderno = od.orderno
GROUP BY o.orderdate
ORDER BY o.orderdate;</code></pre>
<table>
<thead>
<tr>
<th>날짜</th>
<th>계산</th>
<th>값</th>
<th>과제 스펙과 비교</th>
</tr>
</thead>
<tbody><tr>
<td>오늘-3</td>
<td>50,000</td>
<td>50,000</td>
<td>✅ 일치</td>
</tr>
<tr>
<td>오늘-2</td>
<td>150,000 + 300,000</td>
<td>450,000</td>
<td>✅ 일치</td>
</tr>
<tr>
<td>오늘-1</td>
<td>200,000 + 90,000</td>
<td>290,000</td>
<td>✅ 일치</td>
</tr>
</tbody></table>
<p><code>GROUP BY o.orderdate</code>로 같은 날짜의 주문(이영희·오민수는 각각 한 주문에 상품 2개씩)을 하나로 묶고, <code>SUM(cost*qty)</code>로 그날의 매출 총합을 구했다 — 이틀 전(주문번호 2)의 두 상품 금액(150,000 + 300,000)이 정확히 더해져서 450,000이 나온 것까지 확인했다.</p>
<p><strong>문제 13(최진국 목캔디 주문) 검증</strong>: <code>orderno=4</code>, <code>pno=1007</code>, <code>qty=200</code>, <code>cost=3000</code> → 금액 <code>600,000</code>, 재고 <code>500 - 200 = 300</code>으로 갱신 — 과제 스펙과 정확히 일치한다.</p>
<hr />
<h1 id="✨-새롭게-알게-된-점">✨ 새롭게 알게 된 점</h1>
<ul>
<li><code>FOREIGN KEY</code>의 참조무결성은 <code>INSERT</code>뿐 아니라 <code>UPDATE</code>(부모 없는 값으로 바꾸려 할 때)와 <code>DELETE</code>(자식이 참조 중인 부모를 지우려 할 때) 모두에 적용된다.</li>
<li>테이블을 처음부터 완벽하게 설계하지 않아도, <code>ALTER TABLE</code>로 제약조건을 나중에 하나씩 추가/변경할 수 있다.</li>
<li><code>VIEW</code>는 데이터를 저장하지 않고 조회할 때마다 서브쿼리를 다시 실행하는 &quot;가상 테이블&quot;이라, 복잡한 쿼리를 재사용하거나 특정 컬럼만 보여주는 용도로 쓰인다.</li>
<li><code>SEQUENCE</code>는 시작값·증가폭 등을 세밀하게 설정 가능하고, <code>AUTO_INCREMENT</code>는 간단하지만 기본 동작(1씩 증가)만 가능하다.</li>
<li>ERD 스펙표와 실제 <code>CREATE TABLE</code> 구문을 나란히 대조하면, 길이나 <code>NOT NULL</code> 같은 세부사항을 놓치기 쉽다는 걸 직접 확인했다.</li>
</ul>
<hr />
<h1 id="🤔-헷갈렸던-점">🤔 헷갈렸던 점</h1>
<ul>
<li><strong><code>ON DELETE CASCADE</code>를 걸지 않았을 때의 기본 동작</strong>: &quot;부모를 못 지우게 막는다&quot;는 걸 에러로 직접 봤는데, 이게 <code>NO ACTION</code>이라는 이름의 명시적 옵션이라는 걸 나중에 알았다 — 아무것도 안 걸면 자동으로 이 동작이라는 점이 헷갈렸다.</li>
<li><strong><code>MODIFY</code>와 <code>CHANGE</code>의 차이</strong>: 둘 다 컬럼을 바꾸는 명령인데, 이름까지 바꾸려면 <code>CHANGE</code>를 써야 한다는 걸 여러 번 확인하고 나서야 구분이 됐다.</li>
<li><strong>스펙표를 꼼꼼히 대조하는 습관</strong>: <code>phone</code> 길이, <code>pname</code>의 <code>NOT NULL</code> 누락처럼, 테이블을 만들 때는 문제없어 보여도 스펙과 다르게 짠 부분이 있었다 — SQL이 &quot;실행되는지&quot;와 &quot;요구사항에 정확히 맞는지&quot;는 다른 문제라는 걸 느꼈다.</li>
</ul>
<hr />
<h1 id="다음에-할-일">다음에 할 일</h1>
<ul>
<li><input disabled="" type="checkbox" /> <code>ON DELETE CASCADE</code>를 실제로 걸어보고, <code>NO ACTION</code>과 동작 차이를 직접 비교해보기</li>
<li><input disabled="" type="checkbox" /> <code>VIEW</code>에 <code>WITH CHECK OPTION</code>을 걸고, 조건을 벗어난 <code>UPDATE</code>가 실제로 거부되는지 재현해보기</li>
</ul>
<hr />
<h1 id="💡-오늘의-회고">💡 오늘의 회고</h1>
<p>오늘은 개념 실습과 실전 과제가 같은 흐름으로 이어졌다. <code>FOREIGN KEY</code> 에러를 직접 겪고, <code>ALTER TABLE</code>로 제약을 나중에 붙여보는 연습을 오전에 하고 나니, 오후의 쇼핑몰 과제에서 <code>orders</code>가 <code>customers</code>를 참조하고 <code>orderdetail</code>이 <code>orders</code>·<code>products</code>를 동시에 참조하는 구조를 설계할 때 훨씬 수월했다.</p>
<p>과제 결과를 스펙표와 하나하나 대조하면서, <code>phone</code> 컬럼 길이나 <code>pname</code>의 <code>NOT NULL</code>처럼 작은 부분을 놓쳤다는 걸 발견한 것도 의미가 있었다. 코드가 에러 없이 실행된다고 해서 요구사항을 다 지킨 건 아니라는 걸 다시 확인한 하루였다. 일별 매출 쿼리에서 여러 상품 주문이 하나의 날짜로 정확히 합산되는 걸 숫자로 검증하고 나니, <code>GROUP BY</code>가 실제 업무 데이터에서 어떻게 쓰이는지 감이 좀 더 잡혔다.</p>