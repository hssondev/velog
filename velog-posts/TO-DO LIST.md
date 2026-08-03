<h1 id="🔐-리액트-todo-list-프로젝트---로그인-화면-만들기">🔐 리액트 Todo List 프로젝트 - 로그인 화면 만들기</h1>
<p>리액트로 Todo List 앱을 만들기 전에, 먼저 순수 HTML/CSS/JS로 <strong>로그인 화면</strong>을 연습해봤다. 단순히 화면만 만든 게 아니라 실제로 동작하는 검증 로직, 이벤트 처리, 데이터 저장까지 붙여보면서 많이 배운 하루였다.</p>
<hr />
<h2 id="📋-오늘-다룬-내용-한눈에-보기">📋 오늘 다룬 내용 한눈에 보기</h2>
<table>
<thead>
<tr>
<th>주제</th>
<th>핵심 키워드</th>
</tr>
</thead>
<tbody><tr>
<td>🏷️ HTML 기초</td>
<td><code>form method</code>, <code>label</code>, <code>placeholder</code>, <code>button type</code></td>
</tr>
<tr>
<td>🐛 JS 디버깅</td>
<td><code>querySelector</code>, 콤마 연산자, 이벤트 리스너 중첩</td>
</tr>
<tr>
<td>📍 코드 위치</td>
<td><code>&lt;script&gt;</code> 태그 위치, 절대/상대 경로</td>
</tr>
<tr>
<td>💾 데이터 저장</td>
<td><code>localStorage</code></td>
</tr>
<tr>
<td>⌨️ 이벤트 처리</td>
<td>Enter 키 로그인, 비밀번호 토글</td>
</tr>
</tbody></table>
<hr />
<h2 id="1️⃣-html-기초-다지기">1️⃣ HTML 기초 다지기</h2>
<h3 id="form의-method-속성">form의 method 속성</h3>
<p><code>&lt;form&gt;</code>의 <code>method</code>에는 3가지가 올 수 있다.</p>
<table>
<thead>
<tr>
<th>방식</th>
<th>데이터 위치</th>
<th>용도</th>
</tr>
</thead>
<tbody><tr>
<td>🔎 GET</td>
<td>URL에 노출</td>
<td>조회, 검색</td>
</tr>
<tr>
<td>🔒 POST</td>
<td>안 보이게 전송</td>
<td>로그인, 저장</td>
</tr>
<tr>
<td>💬 dialog</td>
<td>페이지 이동 없음</td>
<td><code>&lt;dialog&gt;</code> 모달 닫기 전용</td>
</tr>
</tbody></table>
<p>로그인 폼은 민감한 정보(비밀번호)를 다루기 때문에 <strong>POST</strong>를 써야 한다.</p>
<h3 id="label과-placeholder의-차이">label과 placeholder의 차이</h3>
<pre><code class="language-html">&lt;label for=&quot;id&quot;&gt;아이디&lt;/label&gt;
&lt;input type=&quot;text&quot; placeholder=&quot;이메일&quot; id=&quot;id&quot;&gt;</code></pre>
<ul>
<li><code>label</code>: <strong>항상 보이는</strong> 고정 제목. <code>for</code> 값과 input의 <code>id</code> 값을 맞춰주면 라벨 클릭 시 input에 포커스가 간다.</li>
<li><code>placeholder</code>: <strong>입력 전에만</strong> 보이는 흐린 안내 텍스트. 타이핑을 시작하면 사라지고, 폼 제출 시 값으로 전송되지 않는다.</li>
</ul>
<blockquote>
<p>⚠️ <code>&lt;input for=&quot;id&quot;&gt;</code>처럼 input에 <code>for</code>를 쓰는 건 잘못된 사용이다. <code>for</code>는 label 전용!</p>
</blockquote>
<h3 id="button의-type-속성">button의 type 속성</h3>
<table>
<thead>
<tr>
<th>type 값</th>
<th>동작</th>
</tr>
</thead>
<tbody><tr>
<td>✅ submit (기본값)</td>
<td>form 제출</td>
</tr>
<tr>
<td>🔄 reset</td>
<td>입력값 초기화</td>
</tr>
<tr>
<td>🖱️ button</td>
<td>아무 동작 없음 (JS로 직접 제어)</td>
</tr>
</tbody></table>
<p>form 안에서 JS로 직접 클릭 이벤트를 제어하고 싶다면, 기본값(submit)이 의도치 않게 form을 제출시키지 않도록 <strong><code>type=&quot;button&quot;</code>을 명시</strong>해줘야 한다.</p>
<hr />
<h1 id="🔐-로그인-화면-기능-추가하기---요구사항별-정리">🔐 로그인 화면 기능 추가하기 - 요구사항별 정리</h1>
<p>로그인 화면 기본 골격을 만든 뒤, &quot;고객 요청&quot;이라는 컨셉으로 기능을 하나씩 추가해봤다. 요청 사항별로 어떤 로직을 어떻게 짰는지 정리해본다.</p>
<hr />
<h2 id="📋-요청사항-한눈에-보기">📋 요청사항 한눈에 보기</h2>
<table>
<thead>
<tr>
<th>번호</th>
<th>요청 내용</th>
<th>핵심 개념</th>
</tr>
</thead>
<tbody><tr>
<td>1️⃣</td>
<td>빈칸 입력 방지</td>
<td>조건문 순서, 빈 문자열 체크</td>
</tr>
<tr>
<td>2️⃣</td>
<td>Enter 키로 로그인</td>
<td>함수 분리, <code>keydown</code> 이벤트</td>
</tr>
<tr>
<td>3️⃣</td>
<td>비밀번호 보이기/숨기기</td>
<td><code>type</code> 속성 토글</td>
</tr>
<tr>
<td>4️⃣</td>
<td>계정 여러 개 관리</td>
<td>배열 + <code>.find()</code></td>
</tr>
</tbody></table>
<hr />
<h2 id="1️⃣-요청-1호---빈칸-체크">1️⃣ 요청 1호 - 빈칸 체크</h2>
<blockquote>
<p>&quot;아이디나 비밀번호를 안 쓰고 로그인 버튼을 누르면, 안내 문구가 떴으면 좋겠어요.&quot;</p>
</blockquote>
<h3 id="내-로직">내 로직</h3>
<pre><code class="language-js">//요청 1로 인한 account === &quot;&quot; || pwd === &quot;&quot; 로직 추가
if (account === &quot;&quot; || pwd === &quot;&quot;) {
    window.alert(&quot;아이디와 비밀번호를 입력해주세요&quot;);
} else if (account === &quot;shs10025@naver.com&quot; &amp;&amp; pwd === &quot;1234&quot;) {
    localStorage.setItem(&quot;role&quot;, &quot;user&quot;);
    window.alert(`${account} 님 환영합니다`);
    window.location.href = &quot;../html/todo.html&quot;;
} else if (account === &quot;admin&quot; &amp;&amp; pwd === &quot;admin&quot;) {
    localStorage.setItem(&quot;role&quot;, &quot;admin&quot;);
    window.alert(&quot;관리자로 로그인되었습니다&quot;);
    window.location.href = &quot;../html/todo.html&quot;;
} else {
    window.alert(&quot;잘못된 계정입니다&quot;);
}</code></pre>
<h3 id="배운-점">배운 점</h3>
<ul>
<li>처음엔 빈칸 체크를 계정 검증 <strong>다음 순서</strong>에 뒀었는데, 논리적으로는 &quot;입력이 됐는지&quot;를 <strong>가장 먼저</strong> 확인하는 게 자연스럽다는 걸 깨달았다. 그래서 <code>if</code> 맨 앞으로 옮겼다.</li>
<li><code>||</code>(또는)를 써서 아이디, 비밀번호 <strong>둘 중 하나라도</strong> 비어있으면 걸리게 처리했다.</li>
</ul>
<hr />
<h2 id="2️⃣-요청-2호---enter-키로-로그인">2️⃣ 요청 2호 - Enter 키로 로그인</h2>
<blockquote>
<p>&quot;매번 마우스로 버튼 누르기 귀찮아요. 비밀번호 입력하고 Enter만 눌러도 로그인되게 해주세요.&quot;</p>
</blockquote>
<h3 id="내-로직-1">내 로직</h3>
<pre><code class="language-js">function checkLogin() {
    const account = document.querySelector(&quot;#id&quot;).value;
    const pwd = document.querySelector(&quot;#pwd&quot;).value;

    if (account === &quot;&quot; || pwd === &quot;&quot;) {
        window.alert(&quot;아이디와 비밀번호를 입력해주세요&quot;);
    } else if (account === &quot;shs10025@naver.com&quot; &amp;&amp; pwd === &quot;1234&quot;) {
        localStorage.setItem(&quot;role&quot;, &quot;user&quot;);
        window.alert(`${account} 님 환영합니다`);
        window.location.href = &quot;../html/todo.html&quot;;
    } else if (account === &quot;admin&quot; &amp;&amp; pwd === &quot;admin&quot;) {
        localStorage.setItem(&quot;role&quot;, &quot;admin&quot;);
        window.alert(&quot;관리자로 로그인되었습니다&quot;);
        window.location.href = &quot;../html/todo.html&quot;;
    } else {
        window.alert(&quot;잘못된 계정입니다&quot;);
    }
}

// 버튼 클릭 시
document.querySelector(&quot;#btn&quot;).onclick = checkLogin;

// 비밀번호 칸에서 Enter 입력 시
document.querySelector(&quot;#pwd&quot;).addEventListener(&quot;keydown&quot;, (e) =&gt; {
    if (e.key === &quot;Enter&quot;) {
        checkLogin();
    }
});</code></pre>
<h3 id="시행착오">시행착오</h3>
<ol>
<li>처음엔 <code>$(&quot;#btn&quot;).onclick</code>처럼 jQuery 문법(<code>$</code>)을 썼는데, 프로젝트에 jQuery를 불러온 적이 없어서 동작하지 않았다. → <code>document.querySelector</code>로 수정.</li>
<li><code>onclick</code>으로 Enter 키를 감지하려 했는데, <code>onclick</code>은 <strong>클릭 이벤트</strong>라 키보드 입력은 애초에 감지 대상이 아니었다. → <code>keydown</code> + <code>addEventListener</code>로 변경.</li>
<li><code>.keydown = (e) =&gt; {...}</code>처럼 속성에 직접 함수를 대입했더니 브라우저가 이벤트로 인식하지 않았다. → <code>addEventListener(&quot;keydown&quot;, ...)</code> 형태로 등록해야 한다는 걸 배웠다.</li>
<li>검증 로직이 클릭과 Enter 두 군데 중복될 뻔했는데, <code>checkLogin</code>이라는 <strong>함수로 분리</strong>해서 양쪽에서 재사용하도록 정리했다.</li>
</ol>
<h3 id="왜-id가-아니라-pwd에-이벤트를-걸었나">왜 <code>#id</code>가 아니라 <code>#pwd</code>에 이벤트를 걸었나</h3>
<p>사용자가 Enter를 누르는 시점은 보통 비밀번호까지 다 입력한 직후다. 즉 포커스가 <code>#pwd</code>에 가 있을 때이므로, 이벤트도 <code>#pwd</code>에 등록하는 게 자연스럽다.</p>
<hr />
<h2 id="3️⃣-요청-3호---비밀번호-보이기숨기기">3️⃣ 요청 3호 - 비밀번호 보이기/숨기기</h2>
<blockquote>
<p>&quot;비밀번호 칸에 뭘 쳤는지 안 보이니 오타 확인이 안 돼요. 눈 모양 아이콘으로 잠깐 볼 수 있게 해주세요.&quot;</p>
</blockquote>
<h3 id="내-로직-2">내 로직</h3>
<pre><code class="language-html">&lt;div class=&quot;pwd&quot;&gt;
     &lt;label for=&quot;pwd&quot;&gt;비밀번호&lt;/label&gt;
     &lt;input type=&quot;password&quot; placeholder=&quot;패스워드&quot; id=&quot;pwd&quot;&gt;
     &lt;span id=&quot;toggle&quot;&gt;👁&lt;/span&gt;
&lt;/div&gt;</code></pre>
<pre><code class="language-js">document.querySelector(&quot;#toggle&quot;).onclick = () =&gt; {
    const pwdInput = document.querySelector(&quot;#pwd&quot;);

    if (pwdInput.type === &quot;password&quot;) {
        pwdInput.type = &quot;text&quot;;
    } else {
        pwdInput.type = &quot;password&quot;;
    }
};</code></pre>
<h3 id="시행착오-1">시행착오</h3>
<ol>
<li><code>querySelectorl</code>처럼 함수명에 오타(<code>l</code> 추가)가 있어서 처음엔 에러가 났다.</li>
<li><code>document.querySelector(&quot;#pwd&quot;).value</code>처럼 <code>.value</code>를 붙여버려서 <code>pwdInput</code>이 <strong>문자열</strong>이 돼버린 적이 있다. 문자열에는 <code>.type</code>이 없으므로, <code>.value</code>를 빼고 <strong>엘리먼트 자체</strong>를 담아야 했다.</li>
<li>처음엔 <code>if</code>/<code>else</code> 양쪽 다 <code>&quot;text&quot;</code>로 바꿔버려서 한번 보이면 다시 숨겨지지 않는 버그가 있었다. <code>else</code>에서는 반대로 <code>&quot;password&quot;</code>로 되돌려야 진짜 &quot;토글&quot;이 된다는 걸 깨달았다.</li>
</ol>
<h3 id="css로-아이콘을-input-안쪽에-배치">CSS로 아이콘을 input 안쪽에 배치</h3>
<pre><code class="language-css">.pwd {
    position: relative;
}

#pwd {
    width: 100%;
    padding-right: 36px;
}

#toggle {
    position: absolute;
    right: 12px;
    top: 38px;
    cursor: pointer;
}</code></pre>
<p>부모(<code>.pwd</code>)에 <code>position: relative</code>, 아이콘에 <code>position: absolute</code>를 줘서 input 위에 겹쳐 보이게 만들었다.</p>
<h3 id="hover-대신-click을-쓴-이유">hover 대신 click을 쓴 이유</h3>
<p>hover(마우스 올리기)로도 구현할 수 있지만, 모바일(터치 화면)에서는 hover 자체가 작동하지 않는다. 그래서 클릭 방식으로 유지했다.</p>
<hr />
<h2 id="4️⃣-요청-4호---계정-목록을-배열로-관리하기-⏳-내일-진행-예정">4️⃣ 요청 4호 - 계정 목록을 배열로 관리하기 (⏳ 내일 진행 예정)</h2>
<p>&quot;팀원이 늘어나는데 매번 if / else if로 계정을 추가하려니 코드가 너무 길어져요. 배열로 관리해서 처리해줄 수 있나요?&quot;</p>
<p>이 부분은 오늘 미리 감만 잡아두고, 실제 구현은 내일 해볼 예정이다.</p>
<h2 id="🌱-오늘의-정리">🌱 오늘의 정리</h2>
<p>요청 1~3호를 거치면서 <strong>&quot;문제를 하나씩 요구사항으로 받고, 직접 짜보고, 실수하고, 고쳐나가는&quot;</strong> 흐름이 실제 개발 과정이랑 비슷하다는 걸 느꼈다. 특히:</p>
<ul>
<li>조건문 순서를 논리적으로 배치하는 습관</li>
<li>이벤트 리스너를 중복 없이 함수로 분리해서 재사용하는 습관</li>
<li><code>type</code> 속성처럼 엘리먼트의 상태를 직접 제어하는 법</li>
</ul>
<h2 id="📅-내일-할-일">📅 내일 할 일</h2>
<p>요청 4호 완료하기 — 로그인 계정을 배열 + .find()로 관리하도록 리팩터링
로그인 성공 후 다음 화면 만들기 — 로그인된 상태에서 넘어갈 Todo 화면의 사이드바 만들기</p>
<p>오늘 만든 로그인 화면 → 내일은 로그인 이후 진입하는 <strong>Todo 메인 화면의 골격(사이드바)</strong>을 잡아볼 예정이다. 🚀</p>