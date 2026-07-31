<p><img alt="" src="https://velog.velcdn.com/images/hssondev/post/fd317a6d-07ed-4add-9473-e39932263be9/image.jpg" /></p>
<h2 id="1-오늘의-한-줄-요약-✏️">1. 오늘의 한 줄 요약 ✏️</h2>
<p>CSS3의 의미와 구성 요소, 선택자 종류, HTML5 작성 규칙, 반응형 웹 개념, JS 변수의 기초를 알아보는 시간</p>
<hr />
<h2 id="2-배운-내용">2. 배운 내용</h2>
<h3 id="css3란">CSS3란?</h3>
<p>CSS(Cascading Style Sheets)는 웹 문서의 <strong>디자인</strong>을 담당하는 언어다. 하나의 웹 문서에서 <strong>내용은 HTML이, 디자인은 CSS가</strong> 담당하도록 분리한다.</p>
<p><strong>디자인을 분리했을 때 장점</strong></p>
<ul>
<li>내용과 디자인 수정이 용이</li>
<li>다양한 기능으로 확장 가능</li>
<li>통일된 문서 양식 제공</li>
<li>전송 및 로딩 시간 단축</li>
</ul>
<h3 id="css3의-구성">CSS3의 구성</h3>
<pre><code>h1 { color: blue; font-size: 12px; }
 ↑     ↑                      ↑
선택자 속성                  속성값</code></pre><table>
<thead>
<tr>
<th>구성 요소</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td>선택자(Selector)</td>
<td>스타일을 적용할 대상을 지정</td>
</tr>
<tr>
<td>속성(Property)</td>
<td>어떤 속성을 적용할지 선택</td>
</tr>
<tr>
<td>속성값(Value)</td>
<td>속성에 어떤 값을 반영할지 선택</td>
</tr>
</tbody></table>
<h3 id="css-작성-위치-양식-차이">CSS 작성 위치 (양식 차이)</h3>
<table>
<thead>
<tr>
<th>종류</th>
<th>적용 위치</th>
<th>특징</th>
</tr>
</thead>
<tbody><tr>
<td>인라인 스타일 시트</td>
<td>태그 안에 <code>style=&quot;...&quot;</code> 직접 작성</td>
<td>태그 하나에만 적용</td>
</tr>
<tr>
<td>내부 스타일 시트</td>
<td><code>&lt;head&gt;</code> 안 <code>&lt;style&gt;</code> 태그에 작성</td>
<td>해당 HTML 문서에만 적용</td>
</tr>
<tr>
<td>외부 스타일 시트</td>
<td>별도 <code>.css</code> 파일 → <code>&lt;link&gt;</code>로 연결</td>
<td>여러 HTML 문서에서 재사용 가능</td>
</tr>
</tbody></table>
<pre><code class="language-html">&lt;head&gt;
    &lt;!-- 외부 --&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;style.css&quot;&gt;
    &lt;!-- 내부 --&gt;
    &lt;style&gt;
        p2 { color: blue; }
    &lt;/style&gt;
&lt;/head&gt;
&lt;body&gt;
    &lt;!-- 인라인 --&gt;
    &lt;p style=&quot;color: red;&quot;&gt;인라인 적용&lt;/p&gt;
&lt;/body&gt;</code></pre>
<h3 id="css의-우선순위">CSS의 우선순위</h3>
<pre><code>인라인 스타일 &gt; 내부 스타일 &gt; 외부 스타일 &gt; 브라우저 기본 스타일</code></pre><ul>
<li>하나의 요소에 스타일이 중복 정의되면 <strong>더 우선순위 높은 쪽 + 나중에 작성된 값</strong>이 적용됨</li>
<li>우선순위와 상관없이 강제 적용하고 싶을 땐 <code>!important</code> 사용</li>
</ul>
<h3 id="css-선택자">CSS 선택자</h3>
<table>
<thead>
<tr>
<th>종류</th>
<th>사용 방법</th>
<th>설명</th>
</tr>
</thead>
<tbody><tr>
<td>전체 선택자</td>
<td><code>*</code></td>
<td>모든 태그에 스타일 적용</td>
</tr>
<tr>
<td>타입(태그) 선택자</td>
<td><code>태그명</code></td>
<td>지정한 태그에 적용</td>
</tr>
<tr>
<td>클래스 선택자</td>
<td><code>.클래스명</code></td>
<td>지정한 class에 적용</td>
</tr>
<tr>
<td>아이디 선택자</td>
<td><code>#아이디명</code></td>
<td>지정한 id에 적용 (한 페이지에 유일)</td>
</tr>
<tr>
<td>속성 선택자</td>
<td><code>[속성]</code>, <code>[속성=값]</code></td>
<td>특정 속성을 가진 태그에 적용</td>
</tr>
</tbody></table>
<h3 id="조합-선택자-후손--자식--형제--그룹">조합 선택자 (후손 / 자식 / 형제 / 그룹)</h3>
<p>이번에 새로 배운 <strong>관계 기반 선택자</strong>들이에요. HTML은 태그들이 부모-자식처럼 중첩되는 구조라, 그 관계를 이용해서 선택할 수 있다.</p>
<table>
<thead>
<tr>
<th>구분</th>
<th>조합 방법</th>
<th>설명</th>
</tr>
</thead>
<tbody><tr>
<td>후손 선택자</td>
<td><code>A B</code></td>
<td>A 안에 포함된 모든 B (몇 단계든 상관없음)</td>
</tr>
<tr>
<td>자손(자식) 선택자</td>
<td><code>A &gt; B</code></td>
<td>A의 <strong>바로 아래 자식</strong>인 B만</td>
</tr>
<tr>
<td>인접 형제 선택자</td>
<td><code>A + B</code></td>
<td>A <strong>바로 다음</strong>에 있는 B 하나만</td>
</tr>
<tr>
<td>일반 형제 선택자</td>
<td><code>A ~ B</code></td>
<td>A 뒤에 오는 <strong>모든</strong> 형제 B</td>
</tr>
<tr>
<td>그룹 선택자</td>
<td><code>A, B</code></td>
<td>A와 B를 동시에 선택 (한 번에 스타일 적용)</td>
</tr>
</tbody></table>
<pre><code class="language-css">div ul { color: red; }       /* div 안의 모든 ul (후손) */
div &gt; h3 { color: red; }      /* div의 직계 자식인 h3만 (자손) */
h3 + p { color: purple; }     /* h3 바로 다음 p 하나 (인접 형제) */
h3 ~ p { color: purple; }     /* h3 뒤에 오는 모든 p (일반 형제) */
h3, p, h2 { color: red; }     /* h3, p, h2 모두 (그룹) */</code></pre>
<h3 id="박스-모델---패딩과-마진의-차이">박스 모델 - 패딩과 마진의 차이</h3>
<pre><code>margin(바깥 여백) &gt; border(테두리) &gt; padding(안쪽 여백) &gt; content(내용)</code></pre><table>
<thead>
<tr>
<th>구분</th>
<th>위치</th>
<th>쉬운 비유</th>
</tr>
</thead>
<tbody><tr>
<td><code>padding</code></td>
<td>콘텐츠와 테두리 <strong>사이</strong>의 여백 (안쪽)</td>
<td>액자 속 그림과 액자 틀 사이의 여백</td>
</tr>
<tr>
<td><code>border</code></td>
<td>박스의 테두리 자체</td>
<td>액자 틀</td>
</tr>
<tr>
<td><code>margin</code></td>
<td>테두리 <strong>바깥</strong>, 다른 요소와의 여백 (바깥쪽)</td>
<td>액자와 액자 사이, 벽에 걸린 간격</td>
</tr>
</tbody></table>
<p><strong>한 줄 정리</strong>: <code>padding</code>은 &quot;내용물이 테두리에 붙지 않게&quot; 안쪽으로 밀어내는 여백이고, <code>margin</code>은 &quot;이 박스가 다른 박스와 붙지 않게&quot; 바깥으로 밀어내는 여백이다.</p>
<pre><code class="language-css">div {
    padding: 25px;          /* 안쪽 여백 */
    border: 15px solid navy; /* 테두리 */
    margin: 25px;            /* 바깥 여백 */
}</code></pre>
<p>여백은 <code>top</code>, <code>bottom</code>, <code>left</code>, <code>right</code>로 방향별 지정도 가능하고, 값 개수에 따라 적용 순서가 달라진다.</p>
<pre><code class="language-css">margin: 5px 10px 5px 10px;  /* top right bottom left (시계방향) */
margin: 5px 10px 5px;       /* top, left&amp;right, bottom */
margin: 5px 10px;           /* top&amp;bottom, left&amp;right */
margin: 5px;                 /* 네 방향 모두 동일 */</code></pre>
<hr />
<h2 id="3-html5-기초">3. HTML5 기초</h2>
<h3 id="html-작성-규칙">HTML 작성 규칙</h3>
<ul>
<li>태그는 <strong>소문자</strong>로 작성하는 것이 관례</li>
<li>속성값은 반드시 <strong>큰따옴표(<code>&quot; &quot;</code>)</strong>로 감싸기</li>
<li>태그는 열었으면 반드시 닫기 (<code>&lt;p&gt;...&lt;/p&gt;</code>)</li>
<li>문서는 <code>&lt;!DOCTYPE html&gt;</code>로 시작</li>
</ul>
<h3 id="기본-태그-목록-자주-쓰이는-것">기본 태그 목록 (자주 쓰이는 것)</h3>
<table>
<thead>
<tr>
<th>태그</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td><code>&lt;h1&gt;~&lt;h6&gt;</code></td>
<td>제목 (숫자가 작을수록 크고 중요)</td>
</tr>
<tr>
<td><code>&lt;p&gt;</code></td>
<td>문단</td>
</tr>
<tr>
<td><code>&lt;div&gt;</code></td>
<td>블록 영역을 나누는 컨테이너</td>
</tr>
<tr>
<td><code>&lt;span&gt;</code></td>
<td>인라인 영역을 나누는 컨테이너</td>
</tr>
<tr>
<td><code>&lt;a&gt;</code></td>
<td>링크</td>
</tr>
<tr>
<td><code>&lt;img&gt;</code></td>
<td>이미지</td>
</tr>
<tr>
<td><code>&lt;ul&gt;</code>, <code>&lt;ol&gt;</code>, <code>&lt;li&gt;</code></td>
<td>목록(순서 없음/있음), 목록 항목</td>
</tr>
<tr>
<td><code>&lt;table&gt;</code>, <code>&lt;tr&gt;</code>, <code>&lt;td&gt;</code></td>
<td>표, 행, 셀</td>
</tr>
</tbody></table>
<h3 id="fieldset과-input-타입"><code>&lt;fieldset&gt;</code>과 <code>&lt;input&gt;</code> 타입</h3>
<p><code>&lt;fieldset&gt;</code>은 <strong>폼 안의 관련된 입력 요소들을 하나로 묶는</strong> 태그예요. 보통 테두리가 생기고, <code>&lt;legend&gt;</code>로 그룹 제목을 붙인다.</p>
<pre><code class="language-html">&lt;fieldset&gt;
    &lt;legend&gt;회원 정보&lt;/legend&gt;
    &lt;input type=&quot;text&quot; name=&quot;name&quot;&gt;
    &lt;input type=&quot;button&quot; value=&quot;확인&quot;&gt;
&lt;/fieldset&gt;</code></pre>
<p><strong>자주 쓰는 input 타입</strong></p>
<table>
<thead>
<tr>
<th>타입</th>
<th>설명</th>
</tr>
</thead>
<tbody><tr>
<td><code>text</code></td>
<td>일반 텍스트 입력</td>
</tr>
<tr>
<td><code>button</code></td>
<td>클릭 가능한 버튼</td>
</tr>
<tr>
<td><code>radio</code></td>
<td>여러 개 중 <strong>하나만</strong> 선택 (같은 <code>name</code>으로 그룹 묶음)</td>
</tr>
<tr>
<td><code>checkbox</code></td>
<td>여러 개를 <strong>중복 선택</strong> 가능</td>
</tr>
</tbody></table>
<pre><code class="language-html">&lt;!-- radio: 하나만 선택 --&gt;
&lt;input type=&quot;radio&quot; name=&quot;gender&quot; value=&quot;m&quot;&gt; 남
&lt;input type=&quot;radio&quot; name=&quot;gender&quot; value=&quot;f&quot;&gt; 여

&lt;!-- checkbox: 여러 개 선택 --&gt;
&lt;input type=&quot;checkbox&quot; name=&quot;hobby&quot; value=&quot;read&quot;&gt; 독서
&lt;input type=&quot;checkbox&quot; name=&quot;hobby&quot; value=&quot;movie&quot;&gt; 영화</code></pre>
<blockquote>
<p><code>radio</code>는 같은 <code>name</code> 값을 가진 것들끼리 하나의 그룹으로 묶여서, 그중 하나만 선택 가능해진다. <code>checkbox</code>는 이런 제약이 없어서 여러 개를 동시에 체크할 수 있다.</p>
</blockquote>
<hr />
<h2 id="4-반응형-웹-media-query">4. 반응형 웹 (Media Query)</h2>
<p><strong>반응형 웹</strong>이란, <strong>브라우저(화면) 해상도에 따라 컴포넌트의 배치를 다르게 만드는 것</strong>이다. PC에서는 옆으로 나란히 보이던 메뉴가, 모바일에서는 세로로 쌓이는 것이 대표적인 예다.</p>
<p><strong>텍스트 크기 단위 차이</strong></p>
<table>
<thead>
<tr>
<th>방식</th>
<th>기준</th>
<th>예시</th>
</tr>
</thead>
<tbody><tr>
<td>HTML 기본</td>
<td><code>px</code> (고정 크기)</td>
<td><code>16px</code></td>
</tr>
<tr>
<td>Bootstrap</td>
<td><code>em</code> (상대 크기)</td>
<td><code>1.5em</code> → <code>16px × 1.5 = 24px</code></td>
</tr>
</tbody></table>
<p><code>em</code>은 부모 요소의 크기를 기준으로 <strong>비율로 계산</strong>되기 때문에, 화면 크기가 달라져도 비율에 맞게 조정하기 쉬워서 반응형 웹에서 자주 쓰인다.</p>
<hr />
<h2 id="5-javascript-변수-기초">5. JavaScript 변수 기초</h2>
<h3 id="변수란">변수란?</h3>
<p><strong>데이터를 담는 그릇</strong>. 프로그램이 실행되는 동안 값을 저장하고, 필요할 때 꺼내 쓸 수 있게 해주는 공간이다.</p>
<h3 id="js-변수의-특징">JS 변수의 특징</h3>
<ul>
<li><strong>변수 자체에 타입이 정해져 있지 않음</strong> (다른 언어와 다른 점)</li>
<li>코드가 <strong>실행되는 시점</strong>에, 담긴 값을 보고 타입이 <strong>묵시적으로(암묵적으로)</strong> 결정됨</li>
</ul>
<pre><code class="language-javascript">let a = 10;        // 이 순간 a는 숫자 타입
a = &quot;hello&quot;;        // 이제 a는 문자열 타입 (재할당 가능)</code></pre>
<h3 id="변수-선언-키워드">변수 선언 키워드</h3>
<table>
<thead>
<tr>
<th>키워드</th>
<th>특징</th>
</tr>
</thead>
<tbody><tr>
<td><code>var</code></td>
<td>ES6 이전 방식, 재선언/재할당 모두 가능 (지금은 잘 안 씀)</td>
</tr>
<tr>
<td><code>let</code></td>
<td>재할당은 가능, 재선언은 불가 (요즘 가장 많이 씀)</td>
</tr>
<tr>
<td><code>const</code></td>
<td>재할당도 재선언도 불가 (상수, 값이 안 바뀔 때)</td>
</tr>
</tbody></table>
<blockquote>
<p>TypeScript는 JS와 달리 변수에 <strong>명시적으로 타입을 지정</strong>할 수 있는 언어. JS의 &quot;타입이 실행 시점에 정해진다&quot;는 특징 때문에 생기는 실수를 줄이기 위해 등장했다.</p>
</blockquote>
<h3 id="데이터-타입">데이터 타입</h3>
<table>
<thead>
<tr>
<th>구분</th>
<th>종류</th>
</tr>
</thead>
<tbody><tr>
<td>기본 타입</td>
<td>숫자(Number), 문자열(String), 논리(Boolean), null, undefined</td>
</tr>
<tr>
<td>참조 타입</td>
<td>Object, Array, Function 등 (모두 Object 기반)</td>
</tr>
</tbody></table>
<h3 id="그-외-오늘-언급된-개념">그 외 오늘 언급된 개념</h3>
<ul>
<li><strong>흐름제어구문</strong>: <code>if</code>, <code>switch</code> 등 조건에 따라 코드 실행 흐름을 바꾸는 구문</li>
<li><strong>반복구문</strong>: <code>for</code>, <code>while</code> 등 코드를 반복 실행하는 구문</li>
<li><strong>함수(Function)</strong>: 특정 동작을 묶어서 재사용 가능하게 만든 코드 블록</li>
<li><strong>배열(Array)</strong>: 여러 개의 값을 순서대로 담는 자료구조</li>
</ul>
<hr />
<h2 id="6-헷갈렸던-점-🤔">6. 헷갈렸던 점 🤔</h2>
<ul>
<li><strong>padding과 margin의 차이</strong>: padding은 콘텐츠 <strong>안쪽</strong>(테두리와 내용 사이) 여백, margin은 박스 <strong>바깥쪽</strong>(다른 요소와의 거리) 여백이라는 점이 헷갈렸는데, &quot;안쪽/바깥쪽&quot; 기준으로 구분하니 정리됨</li>
<li><strong>script(JS) 실행 순서</strong>: 변수 선언 → 값 할당 → 이 시점에 타입이 정해진다는 흐름이 처음엔 낯설었음 (다른 언어처럼 타입을 먼저 정하는 게 아니라는 점)</li>
<li><strong>변수들의 타입 구분</strong>: 기본 타입과 참조 타입이 왜 나뉘는지, Object/Array/Function이 왜 다 참조 타입으로 묶이는지 아직 완전히 이해되진 않음 (다음에 더 깊게 알아볼 예정)</li>
</ul>
<h2 id="7-실습-내용">7. 실습 내용</h2>
<h3 id="실습-1-css로-카드형-레이아웃-만들기">실습 1) CSS로 카드형 레이아웃 만들기</h3>
<p><code>card.html</code>을 만들고, <code>&lt;head&gt;</code>의 <code>&lt;link&gt;</code> 태그로 <code>card.css</code>를 연결해서 스타일을 적용해봤다.</p>
<p><strong>card.html</strong></p>
<pre><code class="language-html">&lt;!DOCTYPE html&gt;
&lt;html lang=&quot;en&quot;&gt;
    &lt;head&gt;
        &lt;meta charset=&quot;UTF-8&quot;&gt;
        &lt;meta name=&quot;viewport&quot; content=&quot;width=device-width, initial-scale=1.0&quot;&gt;
        &lt;title&gt;Document&lt;/title&gt;

        &lt;link href=&quot;../css/card.css&quot; rel=&quot;stylesheet&quot;&gt;
    &lt;/head&gt;

    &lt;body&gt;
        &lt;div class=&quot;container&quot;&gt;

            &lt;div class=&quot;tour-intro&quot;&gt;
                &lt;h1&gt;Inspire Camp&lt;/h1&gt;
                &lt;p&gt;방에콕하는 &lt;span&gt;상품&lt;/span&gt;을 소개합니다.&lt;/p&gt;
            &lt;/div&gt;

            &lt;div class=&quot;tour-list&quot;&gt;
                &lt;div class=&quot;tour-prd&quot;&gt;
                    &lt;a href=&quot;#&quot;&gt;
                        &lt;img src=&quot;../img/tr-1.png&quot;&gt;
                        &lt;h3&gt;집캉스&lt;/h3&gt;
                        &lt;button type=&quot;button&quot;&gt;상세페이지&lt;/button&gt;
                    &lt;/a&gt;
                &lt;/div&gt;
                &lt;div class=&quot;tour-prd&quot;&gt;
                    &lt;a href=&quot;#&quot;&gt;
                        &lt;img src=&quot;../img/tr-2.png&quot;&gt;
                        &lt;h3&gt;집캉스&lt;/h3&gt;
                        &lt;button type=&quot;button&quot;&gt;상세페이지&lt;/button&gt;
                    &lt;/a&gt;
                &lt;/div&gt;
                &lt;div class=&quot;tour-prd&quot;&gt;
                    &lt;a href=&quot;#&quot;&gt;
                        &lt;img src=&quot;../img/tr-3.png&quot;&gt;
                        &lt;h3&gt;집캉스&lt;/h3&gt;
                        &lt;button type=&quot;button&quot;&gt;상세페이지&lt;/button&gt;
                    &lt;/a&gt;
                &lt;/div&gt;
                &lt;div class=&quot;tour-prd&quot;&gt;
                    &lt;a href=&quot;#&quot;&gt;
                        &lt;img src=&quot;../img/tr-4.png&quot;&gt;
                        &lt;h3&gt;집캉스&lt;/h3&gt;
                        &lt;button type=&quot;button&quot;&gt;상세페이지&lt;/button&gt;
                    &lt;/a&gt;
                &lt;/div&gt;
            &lt;/div&gt;

        &lt;/div&gt;
    &lt;/body&gt;
&lt;/html&gt;</code></pre>
<p><strong>card.css</strong></p>
<pre><code class="language-css">.container {
    border: 1px solid red;
    width: 1000px;
    margin-left: 0;
}

.tour-intro {
    border: 1px solid blue;
    text-align: center;
    color: antiquewhite;
    background-color: black;
    font-weight: bold;
    margin-top: 10px;
}

.tour-list {
    border: 1px solid pink;
    width: 1000px;
    overflow: hidden;
}

/* 자손 선택자 - .tour-list의 직계 자식 .tour-prd, 그 안의 직계 자식 a, 그 안의 직계 자식 img를 순서대로 타고 들어감 */
.tour-list &gt; .tour-prd &gt; a &gt; img {
    width: 200px;
    border-radius: 50%;
}

.tour-prd &gt; a {
    text-decoration: none;
}

button {
    width: 180px;
    border-radius: 25px;
    padding: 10px 20px;
    margin: 10px 5px 10px 10px;
}

.tour-prd h3, button {
    text-align: center;
}

.tour-prd {
    display: inline-block;
    width: 200px;
    border: 1px solid purple;
    margin: 5px 30px 10px 15px;
}

span {
    color: blue;
}</code></pre>
<p><strong>주요 포인트</strong></p>
<table>
<thead>
<tr>
<th>코드</th>
<th>설명</th>
</tr>
</thead>
<tbody><tr>
<td><code>.tour-list &gt; .tour-prd &gt; a &gt; img</code></td>
<td><strong>자손(자식) 선택자</strong>를 연달아 이어서, <code>.tour-list</code>의 직계 자식인 <code>.tour-prd</code> → 그 직계 자식인 <code>a</code> → 그 직계 자식인 <code>img</code>까지 순서대로 타고 들어가서 선택</td>
</tr>
<tr>
<td><code>border-radius: 50%</code></td>
<td>정사각형에 가까운 이미지에 적용하면 완전한 원형으로 보이게 만드는 효과</td>
</tr>
<tr>
<td><code>overflow: hidden</code></td>
<td><code>.tour-list</code> 안의 카드들이 <code>display: inline-block</code>으로 옆으로 나열되면서 부모 요소를 벗어나는 것을 방지</td>
</tr>
<tr>
<td><code>display: inline-block</code></td>
<td>카드(<code>.tour-prd</code>)들을 한 줄에 옆으로 나란히 배치하면서도, <code>width</code>/<code>margin</code> 같은 박스 속성은 그대로 적용되게 함</td>
</tr>
<tr>
<td><code>text-decoration: none</code></td>
<td><code>&lt;a&gt;</code> 태그에 기본으로 붙는 밑줄 제거</td>
</tr>
</tbody></table>
<blockquote>
<p>참고: HTML에 남아있던 인라인 스타일(<code>&lt;style&gt;</code>)과 <code>&lt;span style=&quot;color: red;&quot;&gt;</code> 부분은 외부 CSS(<code>card.css</code>)의 <code>span { color: blue; }</code>로 대체해서 정리했다. (인라인 스타일이 원래 우선순위가 가장 높지만, 지금은 주석 처리되어 있어서 외부 스타일이 그대로 적용됨)</p>
</blockquote>
<hr />
<h3 id="실습-2-javascript-변수와-타입-실습">실습 2) JavaScript 변수와 타입 실습</h3>
<p><code>var</code>, <code>let</code>, <code>const</code>의 차이와 JS의 <strong>동적 타입</strong> 특징을 콘솔 로그로 직접 확인해봤다.</p>
<pre><code class="language-html">&lt;!DOCTYPE html&gt;
&lt;html lang=&quot;en&quot;&gt;
    &lt;head&gt;
        &lt;meta charset=&quot;UTF-8&quot;&gt;
        &lt;meta name=&quot;viewport&quot; content=&quot;width=device-width, initial-scale=1.0&quot;&gt;
        &lt;title&gt;Document&lt;/title&gt;
    &lt;/head&gt;

    &lt;body&gt;
        &lt;script&gt;
            // var로 선언 - 값을 재할당하면 타입도 같이 바뀜
            var message = &quot;script&quot;;
            console.log(`debug &gt;&gt;&gt;&gt;&gt; message = ${message}`);
            console.log(`debug &gt;&gt;&gt;&gt;&gt; message type = ${typeof message}`);

            message = 10;
            console.log(`debug &gt;&gt;&gt;&gt;&gt; message = ${message}`);
            console.log(`debug &gt;&gt;&gt;&gt;&gt; message type = ${typeof message}`);

            // == vs === 비교
            console.log(`debug &gt;&gt;&gt;&gt;&gt; message == ${message == &quot;10&quot;}`);   // true (값만 비교)
            console.log(`debug &gt;&gt;&gt;&gt;&gt; message === ${message === &quot;10&quot;}`); // false (타입까지 비교)

            // let - 재할당 가능, 재선언 불가
            let msg = &quot;let 키워드를 이용하면 중복 변수 선언을 방지할 수 있습니다.&quot;;

            // const - 재할당 자체가 불가능
            const num = 10;
            // num = 20; // 이 줄의 주석을 풀면 에러 발생

            // 참조 타입 - Object
            let obj = {
                name: &quot;hsson&quot;,
                age: 29,
                major: &quot;cs&quot;
            };
            console.log(&quot;debug &gt;&gt;&gt;&gt;&gt; obj value &quot;, obj);
            console.log(&quot;debug &gt;&gt;&gt;&gt;&gt; obj type &quot;, typeof obj);
            console.log(&quot;debug &gt;&gt;&gt;&gt;&gt; obj property &quot;, typeof obj.major);
            console.log(&quot;debug &gt;&gt;&gt;&gt;&gt; obj property &quot;, typeof obj['major']);

            // 참조 타입 - Array
            let ary = [
                { name: &quot;inspire&quot; },
                { name: &quot;lgcns&quot; },
                { name: &quot;am&quot; },
                { name: &quot;camp6th&quot; }
            ];
            console.log(&quot;debug &gt;&gt;&gt;&gt;&gt; array value &quot;, ary);
            console.log(&quot;debug &gt;&gt;&gt;&gt;&gt; array type &quot;, typeof ary);
            console.log(&quot;debug &gt;&gt;&gt;&gt;&gt; array index &quot;, ary[0].name);
            console.log(`debug &gt;&gt;&gt;&gt;&gt; array index  ${ary[3].name}`);
        &lt;/script&gt;
    &lt;/body&gt;
&lt;/html&gt;</code></pre>
<p><strong>주요 포인트</strong></p>
<table>
<thead>
<tr>
<th>개념</th>
<th>설명</th>
</tr>
</thead>
<tbody><tr>
<td><code>var</code></td>
<td>재할당하면 <strong>값과 타입이 그대로 바뀜</strong> (JS는 변수 자체에 타입이 고정되어 있지 않음)</td>
</tr>
<tr>
<td><code>let</code></td>
<td>같은 이름으로 <strong>재선언은 막아주지만</strong>, 재할당은 자유로움</td>
</tr>
<tr>
<td><code>const</code></td>
<td><strong>재할당 자체가 금지</strong>됨 (상수) — <code>num = 20</code>처럼 다시 값을 넣으려 하면 에러 발생</td>
</tr>
<tr>
<td><code>==</code></td>
<td>값만 비교 (타입이 달라도 값이 같으면 <code>true</code>)</td>
</tr>
<tr>
<td><code>===</code></td>
<td>값과 <strong>타입까지</strong> 비교 (더 엄격하고, 실무에서 권장되는 방식)</td>
</tr>
<tr>
<td><code>typeof</code></td>
<td>변수의 현재 타입을 문자열로 반환해주는 연산자</td>
</tr>
<tr>
<td><code>Object</code></td>
<td><code>{ key: value }</code> 형태 — 이전에 배운 JSON과 구조가 거의 동일</td>
</tr>
<tr>
<td><code>Array</code></td>
<td><code>[ ]</code>로 여러 값을 순서대로 담는 자료구조. <code>ary[0]</code>처럼 인덱스(0부터 시작)로 접근</td>
</tr>
</tbody></table>
<blockquote>
<p><code>==</code>와 <code>===</code>의 차이는 실무에서 자주 실수하는 부분이라, <code>===</code>(일치 연산자)를 기본으로 쓰는 습관을 들이는 게 좋다고 알려져 있다.</p>
</blockquote>
<hr />
<h2 id="8-결과">8. 결과</h2>
<ul>
<li><p>실습1)
카드 4개가 가로로 나열되고, 각 카드의 이미지가 원형으로 표시되는 화면을 확인했다.</p>
<p>로컬 서버(Live Server) 환경에서 확인: <code>http://127.0.0.1:5500/html/card.html</code></p>
</li>
</ul>
<ul>
<li>실습2) 결과
<img alt="" src="https://velog.velcdn.com/images/hssondev/post/34f4f4f5-1d03-4720-b034-001fba99aad3/image.png" /></li>
</ul>