<h2 id="1-오늘의-한-줄-요약-✏️">1. 오늘의 한 줄 요약 ✏️</h2>
<p>프론트엔드의 의미, HTML 문서 구조, SPA 개념, 네이밍 컨벤션, JSON 기본 문법을 학습한 시간</p>
<hr />
<h2 id="2-배운-내용">2. 배운 내용</h2>
<h3 id="html-css-javascript란">HTML, CSS, JavaScript란?</h3>
<p>각각의 역할을 실생활에 비유하면 이렇게 정리할 수 있다.</p>
<table>
<thead>
<tr>
<th>언어</th>
<th>역할</th>
<th>비유</th>
</tr>
</thead>
<tbody><tr>
<td>🦴 <strong>HTML</strong></td>
<td>구조</td>
<td>골조 공사 — 방이 몇 개, 문은 어디, 창문은 어디 (기본 뼈대)</td>
</tr>
<tr>
<td>🎨 <strong>CSS</strong></td>
<td>디자인</td>
<td>인테리어 — 벽지, 가구 배치, 조명 (예쁘게 꾸미기)</td>
</tr>
<tr>
<td>⚡ <strong>JavaScript</strong></td>
<td>행동</td>
<td>전기/스마트홈 시스템 — 스위치 누르면 불 켜지고, 센서가 문 열리고 (작동하는 기능)</td>
</tr>
</tbody></table>
<h3 id="document-문법-구조">Document 문법 구조</h3>
<pre><code class="language-html">&lt;!DOCTYPE html&gt;
&lt;html lang=&quot;en&quot;&gt;
    &lt;head&gt;
        &lt;!-- 문서 정보 (눈에 안 보이는 설정들) --&gt;
    &lt;/head&gt;
    &lt;body&gt;
        &lt;!-- 실제 화면에 보이는 내용 --&gt;
    &lt;/body&gt;
&lt;/html&gt;</code></pre>
<ul>
<li><code>&lt;!DOCTYPE html&gt;</code> : 이 문서가 HTML5임을 브라우저에 알림</li>
<li><code>&lt;html&gt;</code> : 전체 문서를 감싸는 최상위 태그</li>
<li><code>&lt;head&gt;</code> : 화면에 안 보이는 메타 정보</li>
<li><code>&lt;body&gt;</code> : 실제 화면에 보이는 내용</li>
</ul>
<h3 id="head에-자주-쓰이는-태그들"><code>&lt;head&gt;</code>에 자주 쓰이는 태그들</h3>
<pre><code class="language-html">&lt;head&gt;
    &lt;meta charset=&quot;UTF-8&quot;&gt;
    &lt;!-- 문자 인코딩 설정 (한글 깨짐 방지) --&gt;

    &lt;meta name=&quot;viewport&quot; content=&quot;width=device-width, initial-scale=1.0&quot;&gt;
    &lt;!-- 모바일 화면 대응 설정 --&gt;

    &lt;title&gt;Document&lt;/title&gt;
    &lt;!-- 브라우저 탭에 표시되는 제목 --&gt;

    &lt;link href=&quot;...&quot; rel=&quot;stylesheet&quot;&gt;
    &lt;!-- 외부 CSS 파일 연결 --&gt;

    &lt;script src=&quot;...&quot;&gt;&lt;/script&gt;
    &lt;!-- 외부 JavaScript 파일 연결 --&gt;
&lt;/head&gt;</code></pre>
<p><strong>태그의 기본 문법</strong></p>
<pre><code>&lt;meta charset=&quot;UTF-8&quot;&gt;
   ↑      ↑      ↑
태그이름  속성   속성값</code></pre><h3 id="full-browsing-vs-spa">Full Browsing vs SPA</h3>
<table>
<thead>
<tr>
<th>구분</th>
<th>Full Browsing (MPA)</th>
<th>SPA (Single Page Application)</th>
</tr>
</thead>
<tbody><tr>
<td>방식</td>
<td>페이지 이동마다 서버에서 새 HTML을 통째로 받아옴</td>
<td>최초 1회만 HTML을 받고, 이후는 JS가 필요한 부분만 갱신</td>
</tr>
<tr>
<td>화면 전환</td>
<td>새로고침 발생 (하얀 화면이 잠깐 보임)</td>
<td>새로고침 없이 부드럽게 전환</td>
</tr>
<tr>
<td>서버 부담</td>
<td>매번 전체 페이지를 렌더링해서 전달</td>
<td>데이터(JSON)만 주고받아 서버 부담 적음</td>
</tr>
<tr>
<td>예시</td>
<td>전통적인 웹사이트, 게시판</td>
<td>React, Vue로 만든 앱 (Gmail, Twitter 등)</td>
</tr>
</tbody></table>
<ul>
<li><strong>Full Browsing</strong>: 링크 클릭 → 서버에 요청 → 서버가 새 HTML 페이지 전체를 응답 → 브라우저가 다시 그림</li>
<li><strong>SPA</strong>: 최초에 HTML 한 장만 받고, 이후는 자바스크립트가 데이터만 받아서 화면 일부를 동적으로 바꿔치기</li>
</ul>
<h3 id="queryselector-선택자-문법">querySelector 선택자 문법</h3>
<p><code>querySelector()</code>는 CSS 선택자 문법을 그대로 사용해서 HTML 요소를 찾는다.</p>
<table>
<thead>
<tr>
<th>기호</th>
<th>의미</th>
<th>예시</th>
</tr>
</thead>
<tbody><tr>
<td><code>#</code></td>
<td>id를 뜻함</td>
<td><code>#email</code> → id=&quot;email&quot; 요소 찾기</td>
</tr>
<tr>
<td><code>.</code></td>
<td>class를 뜻함</td>
<td><code>.form-control</code> → class=&quot;form-control&quot; 요소 찾기</td>
</tr>
<tr>
<td>(없음)</td>
<td>태그를 뜻함</td>
<td><code>input</code> → input 태그 찾기</td>
</tr>
</tbody></table>
<p><strong>id vs class</strong></p>
<ul>
<li>id: 한 페이지에 <strong>하나만</strong> 존재 (유일한 값)</li>
<li>class: <strong>여러 요소</strong>에 재사용 가능</li>
</ul>
<h3 id="네이밍-컨벤션-naming-convention">네이밍 컨벤션 (Naming Convention)</h3>
<p>코드를 작성할 때 변수명, 함수명, 파일명을 어떻게 지을지에 대한 작명 규칙.</p>
<table>
<thead>
<tr>
<th>케이스</th>
<th>규칙</th>
<th>예시</th>
</tr>
</thead>
<tbody><tr>
<td><strong>Camel Case</strong> 🐫</td>
<td>첫 단어는 소문자, 이후 단어는 첫 글자만 대문자</td>
<td><code>signIn</code></td>
</tr>
<tr>
<td><strong>Pascal Case</strong> 🏛️</td>
<td>모든 단어의 첫 글자를 대문자</td>
<td><code>SignIn</code></td>
</tr>
<tr>
<td><strong>Snake Case</strong> 🐍</td>
<td>모든 단어 소문자, <code>_</code>로 연결</td>
<td><code>sign_in</code></td>
</tr>
<tr>
<td><strong>Kebab Case</strong> 🍢</td>
<td>모든 단어 소문자, <code>-</code>로 연결</td>
<td><code>sign-in</code></td>
</tr>
</tbody></table>
<blockquote>
<p>낙타 등(camel), 뱀(snake), 꼬치(kebab) 모양을 닮아서 붙여진 이름 🐫🐍🍢</p>
</blockquote>
<p><strong>실무에서는 이렇게 사용</strong></p>
<table>
<thead>
<tr>
<th>용도</th>
<th>추천 케이스</th>
<th>예시</th>
</tr>
</thead>
<tbody><tr>
<td>HTML/CSS 파일명</td>
<td>kebab-case</td>
<td><code>sign-in.html</code></td>
</tr>
<tr>
<td>CSS 클래스명</td>
<td>kebab-case</td>
<td><code>.btn-primary</code></td>
</tr>
<tr>
<td>JS 변수/함수명</td>
<td>camelCase</td>
<td><code>getUserInfo()</code></td>
</tr>
<tr>
<td>JS 클래스명</td>
<td>PascalCase</td>
<td><code>class UserForm {}</code></td>
</tr>
<tr>
<td>상수(constant)</td>
<td>SCREAMING_SNAKE_CASE</td>
<td><code>MAX_COUNT</code></td>
</tr>
</tbody></table>
<h3 id="json이란">JSON이란?</h3>
<p><strong>JSON (JavaScript Object Notation)</strong>: 데이터를 저장하고 주고받기 위한 텍스트 형식. JS 객체 문법에서 유래했지만 지금은 언어 상관없이 데이터 교환의 표준으로 쓰인다.</p>
<pre><code class="language-json">{
    &quot;name&quot;: &quot;홍길동&quot;,
    &quot;age&quot;: 25,
    &quot;isStudent&quot;: false,
    &quot;hobbies&quot;: [&quot;독서&quot;, &quot;축구&quot;]
}</code></pre>
<ul>
<li>key는 반드시 큰따옴표(<code>&quot; &quot;</code>) 사용</li>
<li>value는 문자열, 숫자, boolean, 배열, 객체, null 가능</li>
</ul>
<p><strong>JS 객체 vs JSON</strong></p>
<table>
<thead>
<tr>
<th>구분</th>
<th>JS 객체</th>
<th>JSON</th>
</tr>
</thead>
<tbody><tr>
<td>key 따옴표</td>
<td>생략 가능</td>
<td>필수</td>
</tr>
</tbody></table>
<p><strong>JS에서 변환</strong></p>
<pre><code class="language-javascript">JSON.stringify(obj)   // 객체 → JSON 문자열
JSON.parse(jsonStr)   // JSON 문자열 → 객체</code></pre>
<hr />
<h2 id="3-헷갈렸던-점-🤔">3. 헷갈렸던 점 🤔</h2>
<ul>
<li><p><strong><code>onclick</code> vs <code>onClick</code></strong>
JS는 대소문자를 엄격히 구분한다. <code>onClick</code>이라고 쓰면 그냥 존재하지 않는 새 속성이 하나 생길 뿐, 브라우저는 이걸 이벤트로 인식하지 못한다. 반드시 소문자 <code>onclick</code>으로 써야 클릭 이벤트가 정상 동작한다.</p>
</li>
<li><p><strong><code>console.log</code>에 콤마(,)로 넘길 때 vs 템플릿 리터럴(<code>`</code>)로 넘길 때 차이</strong></p>
<pre><code class="language-javascript">console.log(&quot;debug &gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt; &quot;, email);        // 문자열과 변수를 따로따로 출력
console.log(`debug &gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt; ${email}`);        // 하나의 문자열로 합쳐서 출력</code></pre>
<p>결과는 콘솔에서 비슷해 보이지만, 콤마 방식은 값이 여러 개 나열되고 템플릿 리터럴은 완전히 하나의 문자열로 합쳐진다는 차이가 있다.</p>
</li>
<li><p><strong>id와 class를 언제 구분해서 써야 하는지</strong>
처음엔 둘 다 &quot;이름 붙이는 것&quot; 같아서 헷갈렸는데, id는 <strong>하나뿐인 특정 요소</strong>를 콕 집을 때(예: submit 버튼 하나), class는 <strong>같은 스타일/동작을 여러 요소에 반복 적용</strong>할 때 쓴다는 걸 알게 됨.</p>
</li>
</ul>
<hr />
<h2 id="4-실습--적용">4. 실습 / 적용</h2>
<h3 id="직접-해본-것">직접 해본 것</h3>
<p><strong>CSS</strong></p>
<pre><code class="language-css">center {
    display: block;
    margin-top: 80px;
    font-size: 28px;
    font-weight: 700;
    color: blue;
}

.btn {
    cursor: pointer;
    border: none;
    border-radius: 8px;
    color: white;
    background-color: black;
}</code></pre>
<ul>
<li><code>display: block</code> : 요소를 한 줄 전체를 차지하는 블록 형태로 보이게 함</li>
<li><code>border: none</code> : 테두리선 제거</li>
<li><code>border-radius: 8px</code> : 모서리를 둥글게 (숫자가 클수록 더 둥글어짐)</li>
<li><code>cursor: pointer</code> : 마우스를 올렸을 때 손가락 모양 커서로 바뀜 (클릭 가능함을 시각적으로 알려줌)</li>
</ul>
<p><strong>HTML</strong></p>
<pre><code class="language-html">&lt;!DOCTYPE html&gt;
&lt;html lang=&quot;en&quot;&gt;
    &lt;head&gt;
        &lt;meta charset=&quot;UTF-8&quot;&gt;
        &lt;meta name=&quot;viewport&quot; content=&quot;width=device-width, initial-scale=1.0&quot;&gt;
        &lt;title&gt;Document&lt;/title&gt;

        &lt;link href=&quot;https://cdn.jsdelivr.net/npm/bootstrap@5.3.8/dist/css/bootstrap.min.css&quot; rel=&quot;stylesheet&quot;&gt;
        &lt;script src=&quot;https://cdn.jsdelivr.net/npm/bootstrap@5.3.8/dist/js/bootstrap.bundle.min.js&quot;&gt;&lt;/script&gt;
    &lt;/head&gt;

    &lt;body&gt;
        &lt;form action=&quot;/server-destination&quot; method=&quot;get&quot;&gt;
            &lt;div class=&quot;mb-3 mt-3&quot;&gt;
                &lt;label for=&quot;email&quot; class=&quot;form-label&quot;&gt;Email:&lt;/label&gt;
                &lt;input type=&quot;email&quot; class=&quot;form-control&quot; id=&quot;email&quot; placeholder=&quot;Enter email&quot; name=&quot;email&quot;&gt;
            &lt;/div&gt;
            &lt;div class=&quot;mb-3&quot;&gt;
                &lt;label for=&quot;pwd&quot; class=&quot;form-label&quot;&gt;Password:&lt;/label&gt;
                &lt;input type=&quot;password&quot; class=&quot;form-control&quot; id=&quot;pwd&quot; placeholder=&quot;Enter password&quot; name=&quot;pswd&quot;&gt;
            &lt;/div&gt;
            &lt;div class=&quot;form-check mb-3&quot;&gt;
                &lt;label class=&quot;form-check-label&quot;&gt;
                    &lt;input class=&quot;form-check-input&quot; type=&quot;checkbox&quot; name=&quot;remember&quot;&gt; Remember me
                &lt;/label&gt;
            &lt;/div&gt;
            &lt;button type=&quot;button&quot; class=&quot;btn btn-danger&quot; id=&quot;submit&quot;&gt;Submit&lt;/button&gt;
        &lt;/form&gt;

        &lt;script&gt;
            document.querySelector(&quot;#submit&quot;).onclick = function() {
                window.alert(&quot;&gt;&gt;&gt; submit button click &quot;);
                let email = document.querySelector(&quot;#email&quot;).value;
                let pwd = document.querySelector(&quot;#pwd&quot;).value;
                console.log(&quot;debug &gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt; &quot;, email);
                console.log(`debug &gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt; ${email}`);
                console.log(&quot;debug &gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt; &quot;, pwd);
                console.log(`debug &gt;&gt;&gt;&gt;&gt;&gt;&gt;&gt; ${pwd}`);
            }
        &lt;/script&gt;
    &lt;/body&gt;
&lt;/html&gt;</code></pre>
<ul>
<li>Bootstrap CDN을 <code>&lt;link&gt;</code>(CSS)와 <code>&lt;script&gt;</code>(JS)로 각각 연결해서, 직접 CSS를 짜지 않아도 <code>form-control</code>, <code>btn-danger</code> 같은 클래스만으로 스타일을 적용</li>
<li><code>#submit</code> 버튼을 클릭하면 이메일/비밀번호 입력값을 <code>.value</code>로 가져와서 콘솔에 출력하도록 구현</li>
</ul>
<h3 id="결과">결과</h3>
<p><img alt="" src="https://velog.velcdn.com/images/hssondev/post/37e6a93f-b535-411d-a37d-978f6a45261b/image.png" /></p>
<p>Submit 버튼을 클릭하면 alert 창이 뜨고, 콘솔에는 입력한 이메일/비밀번호 값이 출력되는 것을 확인했다.</p>
<hr />
<h2 id="5-참고-자료">5. 참고 자료</h2>
<ul>
<li>HTML/CSS/JS 문법 참고: <a href="https://www.w3schools.com/">w3schools</a></li>
</ul>