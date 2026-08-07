<h3 id="📝-todo-목록-기능-완성하고-로그인한-사람-이름-표시하기">📝 Todo 목록 기능 완성하고, 로그인한 사람 이름 표시하기</h3>
<blockquote>
<p><strong>TIL 원문</strong>: <a href="https://velog.io/@hssondev/LG-CNS-AM-Inspire-Camp-TIL-7%EC%9D%BC%EC%B0%A8-REACT%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-%EB%B8%94%EB%A1%9C%EA%B7%B8-%EA%B5%AC%ED%98%84">LG CNS AM Inspire Camp TIL 7일차 - REACT를 이용한 블로그 구현</a></p>
</blockquote>
<p>오늘은 TIL에서 배운 <strong>폼 통합 관리(spread 패턴)</strong>, <strong>조건부 렌더링</strong>, <strong>배열 다루기</strong> 개념을 우리 Todo 프로젝트에 본격적으로 적용해봤다. 로그아웃 기능을 만들고, Todo 목록(추가/체크/삭제) 기능을 처음부터 완성했고, 사이드바와 Todo 목록을 하나의 페이지로 합쳤다.</p>
<hr />
<h2 id="🔗-오늘-배운-개념-↔-실제-작업-매칭표">🔗 오늘 배운 개념 ↔ 실제 작업 매칭표</h2>
<table>
<thead>
<tr>
<th>TIL에서 배운 개념</th>
<th>오늘 프로젝트에서 한 작업</th>
<th>적용 여부</th>
</tr>
</thead>
<tbody><tr>
<td><code>keyHandler</code>의 spread 패턴 (<code>{...form, [name]: value}</code>)</td>
<td>로그인 폼의 <code>id</code>, <code>pwd</code> state를 <code>form</code> 객체 하나로 통합</td>
<td>✅ 적용</td>
</tr>
<tr>
<td>Hook의 규칙 (컴포넌트 최상위에서만 호출)</td>
<td><code>useNavigate()</code> 위치를 함수 안에서 컴포넌트 최상위로 재확인</td>
<td>✅ 이미 적용 중이었음, 원리 재확인</td>
</tr>
<tr>
<td><code>BlogList</code>의 조건부 렌더링 (<code>&amp;&amp;</code>, 삼항연산자)</td>
<td><code>.filter()</code>로 체크된 항목만 걸러 삭제하는 로직에 응용</td>
<td>✅ 유사 패턴 적용</td>
</tr>
<tr>
<td>json-server 연동</td>
<td>아직 하드코딩된 배열(<code>loginData.js</code>)로 계정 관리 중</td>
<td>⏳ 다음에 적용 예정</td>
</tr>
<tr>
<td><code>useEffect(() =&gt; { loadData() }, [])</code></td>
<td>Todo 목록은 아직 서버에서 안 불러오고 <code>useState</code> 초기값만 사용</td>
<td>⏳ json-server 연동 후 적용 예정</td>
</tr>
<tr>
<td>DTO 개념</td>
<td>아직 프론트-백엔드 분리가 없어서 미적용</td>
<td>📝 개념 학습만</td>
</tr>
</tbody></table>
<hr />
<h2 id="📋-오늘-한-일-한눈에-보기">📋 오늘 한 일 한눈에 보기</h2>
<table>
<thead>
<tr>
<th>구분</th>
<th>내용</th>
</tr>
</thead>
<tbody><tr>
<td>🔑 폼 리팩터링</td>
<td><code>keyHandler</code> spread 패턴으로 로그인 폼 통합</td>
</tr>
<tr>
<td>🚪 로그아웃</td>
<td>props로 함수 전달 + <code>localStorage</code> 정리</td>
</tr>
<tr>
<td>✅ Todo 기능</td>
<td>추가 / 체크박스 토글 / 선택 삭제</td>
</tr>
<tr>
<td>🧩 페이지 합치기</td>
<td><code>Sidebar</code> + <code>TodoList</code> → <code>TodoPage</code>로 통합</td>
</tr>
<tr>
<td>🎨 스타일링</td>
<td>로그인 화면과 톤을 맞춘 CSS</td>
</tr>
<tr>
<td>👤 사용자 표시</td>
<td><code>localStorage</code>로 로그인한 아이디를 사이드바에 표시</td>
</tr>
</tbody></table>
<hr />
<h2 id="1️⃣-keyhandler-spread-패턴으로-로그인-폼-리팩터링">1️⃣ <code>keyHandler</code> spread 패턴으로 로그인 폼 리팩터링</h2>
<h3 id="왜-바꿨나">왜 바꿨나</h3>
<p>기존엔 <code>id</code>, <code>pwd</code>를 각각 <code>useState</code>로 따로 관리했다. TIL에서 배운 것처럼, 폼 필드가 여러 개일 때는 <strong>객체 하나로 묶어서 관리</strong>하는 게 표준적인 패턴이라는 걸 배웠다.</p>
<pre><code class="language-jsx">// 이전
const [id, setId] = useState(&quot;&quot;);
const [pwd, setPwd] = useState(&quot;&quot;);

// 이후
const [form, setForm] = useState({ id: &quot;&quot;, pwd: &quot;&quot; });</code></pre>
<h3 id="keyhandler-함수"><code>keyHandler</code> 함수</h3>
<pre><code class="language-jsx">const keyHandler = (e) =&gt; {
    const { name, value } = e.target;
    setForm({ ...form, [name]: value });
};</code></pre>
<ul>
<li><code>e.target</code>에서 <code>name</code>, <code>value</code>를 구조 분해로 꺼냄</li>
<li><code>...form</code>으로 기존 값은 그대로 복사</li>
<li><code>[name]: value</code> — <strong>계산된 속성명</strong> 문법으로, <code>name</code>이 <code>&quot;id&quot;</code>면 <code>id</code> 필드만, <code>&quot;pwd&quot;</code>면 <code>pwd</code> 필드만 덮어씀</li>
</ul>
<p>각 <code>&lt;input&gt;</code>에 <code>name</code> 속성을 추가하고, <code>onChange</code>를 전부 <code>keyHandler</code> 하나로 통일했다. input이 몇 개로 늘어나도 함수 하나로 대응할 수 있게 된 게 핵심 이득이다.</p>
<hr />
<h2 id="2️⃣-로그아웃-기능">2️⃣ 로그아웃 기능</h2>
<p>로그인 때 썼던 <strong>props로 함수 전달하는 패턴</strong>을 로그아웃에도 그대로 재사용했다.</p>
<pre><code class="language-jsx">// App.js
const handleLogout = () =&gt; {
    setIsLoggedIn(false);
    localStorage.removeItem(&quot;logInYN&quot;);
};

&lt;Sidebar onLogout={handleLogout} /&gt;</code></pre>
<pre><code class="language-jsx">// Sidebar.jsx
const Sidebar = ({ onLogout }) =&gt; {
    const moveUrl = useNavigate();

    const checkLogOut = () =&gt; {
        localStorage.removeItem(&quot;logInYN&quot;);
        onLogout();
        moveUrl('/');
    };</code></pre>
<h3 id="헷갈렸던-점">헷갈렸던 점</h3>
<ul>
<li><code>useState</code>로 로그인/로그아웃 두 개의 state를 따로 만들려고 했었는데, &quot;로그인 여부&quot;는 <strong>하나의 boolean 값</strong>(<code>isLoggedIn</code>)으로 충분하다는 걸 다시 확인했다. 반대 상태는 그 값을 뒤집으면 되는 거지, 별도 state가 필요한 게 아니었다.</li>
<li><code>moveUrl('/')</code>로 페이지를 이동시켜도, <code>App.js</code>의 <code>isLoggedIn</code>은 <strong>자동으로 안 바뀐다</strong>는 걸 처음 알았다. <code>useState</code>는 컴포넌트가 처음 마운트될 때만 초기값을 계산하기 때문에, 라우터로 이동만 시켜서는 그 값이 재계산되지 않는다. 그래서 로그인 때처럼 <code>onLogout</code> 함수를 명시적으로 호출해서 state를 바꿔줘야 했다.</li>
</ul>
<hr />
<h2 id="3️⃣-todo-목록-기능-todolistjsx">3️⃣ Todo 목록 기능 (<code>TodoList.jsx</code>)</h2>
<h3 id="데이터-추가---spread로-배열에-항목-붙이기">데이터 추가 - spread로 배열에 항목 붙이기</h3>
<pre><code class="language-jsx">const addTodo = () =&gt; {
    const newTodo = { id: todos.length + 1, title: inputText, done: false };
    setTodos([...todos, newTodo]);
    setInputText(&quot;&quot;);
};</code></pre>
<p><code>[...todos, newTodo]</code> — 기존 배열을 펼쳐놓고 뒤에 새 항목을 붙여서, 원본은 건드리지 않고 새 배열을 만드는 방식. 객체에 썼던 spread 문법을 배열에도 똑같이 적용할 수 있다는 걸 확인했다.</p>
<h3 id="완료-토글---map으로-하나만-바꾸기">완료 토글 - <code>.map()</code>으로 하나만 바꾸기</h3>
<pre><code class="language-jsx">const toggleDone = (id) =&gt; {
    const newTodos = todos.map((item) =&gt; {
        if (item.id === id) {
            return { ...item, done: !item.done };
        }
        return item;
    });
    setTodos(newTodos);
};</code></pre>
<p><code>.map()</code>은 원본과 <strong>개수가 같은</strong> 새 배열을 만든다. 클릭한 항목만 <code>done</code> 값을 뒤집고, 나머지는 그대로 반환하는 방식으로 &quot;하나만 바꾸기&quot;를 구현했다.</p>
<h3 id="선택-삭제---filter로-걸러내기">선택 삭제 - <code>.filter()</code>로 걸러내기</h3>
<pre><code class="language-jsx">const deleteChecked = () =&gt; {
    const newTodos = todos.filter((item) =&gt; !item.done);
    setTodos(newTodos);
};</code></pre>
<p><code>.filter()</code>는 조건에 맞는 것만 남기고 <strong>개수가 줄어드는</strong> 새 배열을 만든다. <code>!item.done</code>(체크 안 된 것)만 남기는 방식으로, 체크된 항목들을 한번에 제거했다.</p>
<h3 id="map-vs-filter-정리"><code>.map()</code> vs <code>.filter()</code> 정리</h3>
<table>
<thead>
<tr>
<th></th>
<th><code>.map()</code></th>
<th><code>.filter()</code></th>
</tr>
</thead>
<tbody><tr>
<td>결과 배열 개수</td>
<td>원본과 동일</td>
<td>조건에 따라 줄어듦</td>
</tr>
<tr>
<td>용도</td>
<td>항목의 <strong>내용</strong>을 바꿀 때</td>
<td>조건에 맞는 항목만 <strong>남길/제거할</strong> 때</td>
</tr>
<tr>
<td>오늘 쓴 곳</td>
<td><code>toggleDone</code></td>
<td><code>deleteChecked</code></td>
</tr>
</tbody></table>
<hr />
<h2 id="4️⃣-sidebar--todolist-→-todopage로-합치기">4️⃣ <code>Sidebar</code> + <code>TodoList</code> → <code>TodoPage</code>로 합치기</h2>
<p>두 컴포넌트를 각각 만들어두고, 이걸 한 화면에 보여줄 <strong>부모 페이지</strong>가 필요했다.</p>
<pre><code class="language-jsx">// TodoPage.jsx
const TodoPage = ({ onLogout }) =&gt; {
    return (
        &lt;div className=&quot;todoPage&quot;&gt;
            &lt;Sidebar onLogout={onLogout} /&gt;
            &lt;div className=&quot;todoContent&quot;&gt;
                &lt;TodoList /&gt;
            &lt;/div&gt;
        &lt;/div&gt;
    );
};</code></pre>
<h3 id="props-릴레이중계-개념">props 릴레이(중계) 개념</h3>
<p><code>TodoPage</code>는 <code>onLogout</code>을 직접 쓰지 않지만, <code>App.js</code>로부터 받은 걸 다시 <code>Sidebar</code>에게 그대로 넘겨준다. 부모가 자식에게, 그 자식이 또 자기 자식에게 함수를 전달하는 <strong>&quot;중계&quot;</strong> 구조를 처음 경험했다.</p>
<pre><code>App (handleLogout 생성)
  └─ TodoPage (onLogout으로 받아서, 다시 그대로 넘김)
        └─ Sidebar (onLogout으로 받아서, 실제로 사용)</code></pre><hr />
<h2 id="5️⃣-css로-로그인-화면과-톤-맞추기">5️⃣ CSS로 로그인 화면과 톤 맞추기</h2>
<p><code>Sidebar</code>, <code>TodoList</code>, <code>TodoPage</code>에 공통된 스타일 언어를 적용했다.</p>
<ul>
<li>흰 배경 + <code>border-radius</code> + <code>box-shadow</code>로 카드 느낌 (로그인 폼과 동일한 패턴)</li>
<li>포인트 색상(<code>#6f4e37</code>, 갈색)을 버튼/아이콘/체크박스에 일관되게 사용</li>
<li><code>.todoPage</code>에 <code>display: flex</code>로 사이드바와 콘텐츠 영역을 가로로 배치</li>
<li>완료된 할 일은 <code>text-decoration: line-through</code>로 취소선 처리</li>
</ul>
<hr />
<h2 id="6️⃣-로그인한-사용자-아이디-표시하기">6️⃣ 로그인한 사용자 아이디 표시하기</h2>
<h3 id="흐름">흐름</h3>
<ol>
<li><p><code>LoginPage.jsx</code> — 로그인 성공 시 아이디를 <code>localStorage</code>에 저장</p>
<pre><code class="language-jsx">localStorage.setItem(&quot;account&quot;, found.id);</code></pre>
</li>
<li><p><code>Sidebar.jsx</code> — 컴포넌트가 렌더링될 때 그 값을 꺼내옴</p>
<pre><code class="language-jsx">const account = localStorage.getItem(&quot;account&quot;);</code></pre>
</li>
<li><p>화면에 표시</p>
<pre><code class="language-jsx">&lt;span&gt;{account}&lt;/span&gt;</code></pre>
</li>
</ol>
<h3 id="핵심---localstorage는-props-없이도-공유된다">핵심 - <code>localStorage</code>는 props 없이도 공유된다</h3>
<p><code>LoginPage</code>와 <code>Sidebar</code>는 부모-자식 관계가 아니라서 props로 직접 값을 주고받을 수 없다. 하지만 <code>localStorage</code>는 <strong>브라우저에 저장되는 공용 저장소</strong>라서, 어떤 컴포넌트든 같은 이름표(<code>&quot;account&quot;</code>)로 저장하고 꺼내 쓸 수 있다는 걸 배웠다.</p>
<table>
<thead>
<tr>
<th></th>
<th>props</th>
<th>localStorage</th>
</tr>
</thead>
<tbody><tr>
<td>관계</td>
<td>부모→자식만 가능</td>
<td>어떤 컴포넌트든 접근 가능</td>
</tr>
<tr>
<td>유지 기간</td>
<td>컴포넌트 사라지면 같이 사라짐</td>
<td>브라우저에 계속 남음 (새로고침해도 유지)</td>
</tr>
</tbody></table>
<hr />
<h2 id="🌱-오늘의-정리">🌱 오늘의 정리</h2>
<ul>
<li><code>keyHandler</code> spread 패턴으로 폼 관리 방식을 실무 스타일로 리팩터링</li>
<li>로그인/로그아웃에 반복적으로 쓰이는 <strong>props로 함수 전달</strong> 패턴을 완전히 체화</li>
<li><code>.map()</code>, <code>.filter()</code>, spread(<code>...</code>) — 배열을 다루는 세 가지 핵심 도구를 모두 실전에 적용</li>
<li>컴포넌트를 여러 개로 쪼갠 뒤, 부모 페이지에서 합치고 props를 중계하는 구조 설계</li>
<li><code>localStorage</code>가 props와 다르게 &quot;컴포넌트 관계와 무관하게&quot; 데이터를 공유하는 저장소라는 것 이해</li>
</ul>
<h2 id="📅-다음-할-일">📅 다음 할 일</h2>
<ul>
<li>json-server 연동해서 계정/할 일 데이터를 진짜 서버처럼 관리</li>
<li><code>useEffect</code>로 컴포넌트 마운트 시 서버에서 데이터 불러오기</li>
<li>Todo 목록이 비어있을 때 &quot;등록된 할 일이 없습니다&quot; 같은 조건부 렌더링 추가</li>
<li>메뉴 클릭 시 선택 항목 하이라이트</li>
</ul>
<p>오늘로 로그인부터 Todo 등록/완료/삭제까지 기본적인 한 사이클이 완성됐다. 다음은 이 모든 걸 진짜 서버와 연결하는 단계로 넘어갈 예정이다. 🚀</p>