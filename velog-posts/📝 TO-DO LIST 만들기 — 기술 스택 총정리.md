<h3 id="🛠️-포트폴리오-정리용-그동안-쓴-기술들-모아보기">🛠️ 포트폴리오 정리용, 그동안 쓴 기술들 모아보기</h3>
<blockquote>
<p><strong>관련 TIL 모음</strong></p>
<ul>
<li><a href="https://velog.io/@hssondev/LG-CNS-AM-Inspire-Camp-TIL-7%EC%9D%BC%EC%B0%A8-REACT%EB%9E%80">TIL 7일차 - REACT란?</a></li>
<li><a href="https://velog.io/@hssondev/LG-CNS-AM-Inspire-Camp-TIL-7%EC%9D%BC%EC%B0%A8-REACT%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-%EB%B8%94%EB%A1%9C%EA%B7%B8-%EA%B5%AC%ED%98%84">TIL 7일차 - REACT를 이용한 블로그 구현</a></li>
<li><a href="https://velog.io/@hssondev/LG-CNS-AM-Inspire-CampTIL-9%EC%9D%BC%EC%B0%A8-%EC%98%A4%ED%94%88-API-%EC%97%B0%EB%8F%99-Weather-Kakao-Map">TIL 9일차 - 오픈 API 연동 Weather/Kakao Map</a></li>
</ul>
</blockquote>
<p>로그인 화면부터 Todo 목록, 회원가입, 서버 연동까지 만들어오면서 그동안 배운 개념들이 실제로 어디에 쓰였는지 한 번에 정리가 필요할 것 같아서 모아봤다. 포트폴리오에 쓸 때도 참고하려고 남겨둔다.</p>
<hr />
<h2 id="🔗-그동안-배운-개념-↔-실제-작업-매칭표">🔗 그동안 배운 개념 ↔ 실제 작업 매칭표</h2>
<table>
<thead>
<tr>
<th>배운 개념</th>
<th>실제 프로젝트에서 한 작업</th>
<th>적용 여부</th>
</tr>
</thead>
<tbody><tr>
<td><code>useState</code></td>
<td>폼 입력값, Todo 목록, 로그인 여부 등 상태 전반 관리</td>
<td>✅ 적용</td>
</tr>
<tr>
<td>제어 컴포넌트 (<code>value</code> + <code>onChange</code>)</td>
<td>로그인/회원가입/Todo 입력창 전체</td>
<td>✅ 적용</td>
</tr>
<tr>
<td><code>useRef</code></td>
<td>할 일 추가 후 입력창 자동 포커스</td>
<td>✅ 적용</td>
</tr>
<tr>
<td>props (부모→자식 함수 전달)</td>
<td>로그인/로그아웃 처리, Todo 토글 (<code>onLogin</code>, <code>onLogout</code>, <code>onToggle</code>)</td>
<td>✅ 적용</td>
</tr>
<tr>
<td><code>.map()</code> / <code>.filter()</code> / <code>.find()</code></td>
<td>목록 렌더링, 선택 삭제, 로그인 계정 검증</td>
<td>✅ 적용</td>
</tr>
<tr>
<td>spread 문법 (<code>...</code>)</td>
<td>폼 통합 관리, 배열/객체 불변성 유지</td>
<td>✅ 적용</td>
</tr>
<tr>
<td>계산된 속성명 (<code>[name]: value</code>)</td>
<td><code>keyHandler</code>로 여러 input 하나의 함수로 처리</td>
<td>✅ 적용</td>
</tr>
<tr>
<td>조건부 렌더링 (<code>? :</code>)</td>
<td>로그인 여부별 화면 분기, 빈 목록 안내 문구</td>
<td>✅ 적용</td>
</tr>
<tr>
<td>react-router-dom</td>
<td><code>/</code>, <code>/todo</code>, <code>/signup</code> 경로 분리, <code>useNavigate</code>로 이동</td>
<td>✅ 적용</td>
</tr>
<tr>
<td>axios + json-server</td>
<td>회원가입/로그인 서버 연동</td>
<td>✅ 적용</td>
</tr>
<tr>
<td>async/await</td>
<td>서버 응답을 기다린 뒤 로직 처리</td>
<td>✅ 적용</td>
</tr>
<tr>
<td>localStorage</td>
<td>로그인 상태 유지, 로그인한 아이디 표시</td>
<td>✅ 적용</td>
</tr>
</tbody></table>
<hr />
<h2 id="📦-프로젝트-구조">📦 프로젝트 구조</h2>
<pre><code>src/
├── pages/           # 화면 단위 컴포넌트
│   ├── LoginPage.jsx
│   ├── SignupPage.jsx
│   ├── Sidebar.jsx
│   ├── TodoList.jsx
│   ├── TodoItem.jsx
│   └── TodoPage.jsx
├── components/
│   └── styled/       # 컴포넌트별 CSS
├── data/             # 정적 데이터 (초기값)
├── App.js            # 라우팅 및 전역 로그인 상태 관리
└── db.json           # json-server용 목 데이터베이스</code></pre><p><code>pages</code>는 화면 하나를 통째로 나타내는 컴포넌트(라우팅 단위), <code>components</code>는 여러 화면에서 재사용 가능한 조각으로 나눴다. 데이터, 로직, 스타일을 파일 단위로 분리해서 관심사를 나눴다.</p>
<hr />
<h2 id="1️⃣-usestate--컴포넌트-상태-관리">1️⃣ <code>useState</code> — 컴포넌트 상태 관리</h2>
<p>값이 바뀌면 화면이 자동으로 다시 그려지는 변수(state)를 만드는 Hook.</p>
<pre><code class="language-jsx">const [form, setForm] = useState({ id: &quot;&quot;, pwd: &quot;&quot; });</code></pre>
<p>로그인/회원가입 폼 입력값, Todo 목록, 비밀번호 표시 여부, 로그인 여부(<code>isLoggedIn</code>)까지 프로젝트 전반의 상태를 이걸로 관리했다.</p>
<hr />
<h2 id="2️⃣-제어-컴포넌트-controlled-component">2️⃣ 제어 컴포넌트 (Controlled Component)</h2>
<p>input의 값을 DOM이 아니라 React state가 직접 소유하고 제어하는 패턴. <code>value</code>와 <code>onChange</code>를 항상 짝지어 썼다.</p>
<pre><code class="language-jsx">&lt;input
    value={form.id}
    onChange={(e) =&gt; setForm({ ...form, id: e.target.value })}
/&gt;</code></pre>
<hr />
<h2 id="3️⃣-useref--dom-요소-직접-제어">3️⃣ <code>useRef</code> — DOM 요소 직접 제어</h2>
<p>값을 저장은 하지만, 값이 바뀌어도 리렌더링을 유발하지 않는 Hook. <code>useState</code>와 다르게 &quot;화면엔 영향 없이 그냥 기억만 해두고 싶을 때&quot; 썼다.</p>
<pre><code class="language-jsx">const inputRef = useRef(null);
// ...
inputRef.current.focus();</code></pre>
<p>할 일을 추가한 직후, 입력창에 커서를 자동으로 다시 놓아주는 데 활용했다.</p>
<hr />
<h2 id="4️⃣-props--컴포넌트-간-데이터함수-전달">4️⃣ props — 컴포넌트 간 데이터/함수 전달</h2>
<p>부모가 자식에게 데이터나 함수를 전달하는 유일한 통로. 함수를 props로 내려주면, 자식이 부모의 상태를 간접적으로 바꿀 수 있다.</p>
<pre><code class="language-jsx">// 부모
&lt;LoginPage onLogin={handleLogin} /&gt;

// 자식
const LoginPage = ({ onLogin }) =&gt; { ... onLogin(); ... }</code></pre>
<p>로그인 성공 시 App의 전역 상태를 바꾸는 데(<code>onLogin</code>), 로그아웃 처리(<code>onLogout</code>)에 썼다. 특히 로그아웃은 <code>App → TodoPage → Sidebar</code>까지 3단계로 props를 릴레이(중계)하는 구조를 처음 경험했다.</p>
<hr />
<h2 id="5️⃣-배열-메서드-3종--불변성을-유지하며-데이터-다루기">5️⃣ 배열 메서드 3종 — 불변성을 유지하며 데이터 다루기</h2>
<p>React에서는 state를 직접 수정하지 않고, 항상 <strong>새로운 배열/객체를 만들어 교체</strong>하는 방식을 쓴다.</p>
<table>
<thead>
<tr>
<th>메서드</th>
<th>역할</th>
<th>결과 배열 크기</th>
<th>적용 사례</th>
</tr>
</thead>
<tbody><tr>
<td><code>.map()</code></td>
<td>각 요소를 원하는 형태로 변환</td>
<td>원본과 동일</td>
<td>할 일 완료 상태 토글, 목록 렌더링</td>
</tr>
<tr>
<td><code>.filter()</code></td>
<td>조건에 맞는 요소만 남김</td>
<td>조건에 따라 감소</td>
<td>체크된 할 일 선택 삭제</td>
</tr>
<tr>
<td><code>.find()</code></td>
<td>조건에 맞는 첫 번째 요소 하나 반환</td>
<td>단일 값</td>
<td>로그인 계정 검증</td>
</tr>
</tbody></table>
<pre><code class="language-jsx">// map: 특정 항목만 값 변경
setTodos(todos.map(item =&gt; item.id === id ? { ...item, done: !item.done } : item));

// filter: 조건에 안 맞는 것 제거
setTodos(todos.filter(item =&gt; !item.done));

// find: 조건에 맞는 것 하나 탐색
const found = users.find(user =&gt; user.account === id &amp;&amp; user.pwd === pwd);</code></pre>
<hr />
<h2 id="6️⃣-spread-문법---불변성-유지">6️⃣ Spread 문법 (<code>...</code>) — 불변성 유지</h2>
<p>객체나 배열의 내용을 복사해서 펼치는 문법. 기존 데이터는 건드리지 않고, 필요한 부분만 바꾼 새 데이터를 만들 때 썼다.</p>
<pre><code class="language-jsx">// 객체: 기존 값 복사 + 특정 필드만 덮어쓰기
setForm({ ...form, [name]: value });

// 배열: 기존 배열 + 새 항목 추가
setTodos([...todos, newTodo]);</code></pre>
<hr />
<h2 id="7️⃣-계산된-속성명-computed-property-name">7️⃣ 계산된 속성명 (Computed Property Name)</h2>
<p>변수에 담긴 문자열 값을 객체의 key로 사용하는 문법. 여러 input을 하나의 핸들러 함수로 처리할 때 활용했다.</p>
<pre><code class="language-jsx">const keyHandler = (e) =&gt; {
    const { name, value } = e.target;
    setForm({ ...form, [name]: value });
};</code></pre>
<p>input 개수가 늘어나도 핸들러 함수 하나로 전부 대응할 수 있게 됐다.</p>
<hr />
<h2 id="8️⃣-조건부-렌더링">8️⃣ 조건부 렌더링</h2>
<p>특정 조건에 따라 다른 UI를 보여주는 패턴. 삼항연산자(<code>? :</code>)를 주로 썼다.</p>
<pre><code class="language-jsx">{isLoggedIn ? &lt;TodoPage /&gt; : &lt;LoginPage /&gt;}
{todos.length === 0 ? &lt;p&gt;등록된 할 일이 없습니다.&lt;/p&gt; : &lt;ul&gt;...&lt;/ul&gt;}</code></pre>
<hr />
<h2 id="9️⃣-react-router-dom--클라이언트-사이드-라우팅">9️⃣ react-router-dom — 클라이언트 사이드 라우팅</h2>
<table>
<thead>
<tr>
<th>API</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td><code>&lt;BrowserRouter&gt;</code></td>
<td>라우팅 기능을 앱 전체에 적용하는 최상위 래퍼</td>
</tr>
<tr>
<td><code>&lt;Routes&gt;</code> / <code>&lt;Route&gt;</code></td>
<td>경로(path)와 컴포넌트(element)를 매핑</td>
</tr>
<tr>
<td><code>useNavigate()</code></td>
<td>코드 실행 중 특정 경로로 프로그래밍 방식 이동</td>
</tr>
</tbody></table>
<pre><code class="language-jsx">&lt;Routes&gt;
    &lt;Route path=&quot;/&quot; element={isLoggedIn ? &lt;TodoPage/&gt; : &lt;LoginPage/&gt;} /&gt;
    &lt;Route path=&quot;/todo&quot; element={&lt;TodoPage/&gt;} /&gt;
    &lt;Route path=&quot;/signup&quot; element={&lt;SignupPage/&gt;} /&gt;
&lt;/Routes&gt;</code></pre>
<pre><code class="language-jsx">const moveUrl = useNavigate();
moveUrl('/todo');</code></pre>
<p>Hook은 반드시 컴포넌트 최상위에서만 호출해야 한다는 규칙을 여러 번 실수하면서 몸에 익혔다.</p>
<hr />
<h2 id="🔟-axios--json-server--서버-연동">🔟 axios + json-server — 서버 연동</h2>
<p><strong>json-server</strong>: JSON 파일(<code>db.json</code>) 하나로 REST API 서버를 흉내 내주는 도구.</p>
<pre><code class="language-bash">npx json-server --watch db.json --port 5000</code></pre>
<p><strong>axios</strong>: HTTP 요청을 보내는 라이브러리.</p>
<pre><code class="language-jsx">// 조회
const response = await axios.get('http://localhost:5000/users');
const users = response.data;

// 생성
await axios.post('http://localhost:5000/users', form);</code></pre>
<p><strong>async/await</strong>: 서버 응답을 기다렸다가 그 다음 코드를 순차적으로 실행하게 해주는 문법.</p>
<pre><code class="language-jsx">const checkLogin = async () =&gt; {
    const response = await axios.get(...);
    const users = response.data;
    const found = users.find(...);
};</code></pre>
<p>회원가입은 서버에 새 계정을 저장하는 데, 로그인은 서버에서 전체 계정을 받아온 뒤 검증하는 데 썼다. 하드코딩된 배열(<code>loginData.js</code>)에서 진짜 서버 연동 구조로 리팩터링한 게 오늘의 가장 큰 변화였다.</p>
<hr />
<h2 id="1️⃣1️⃣-localstorage--클라이언트-측-영속성">1️⃣1️⃣ localStorage — 클라이언트 측 영속성</h2>
<p>브라우저에 데이터를 저장해 새로고침해도 유지되게 하는 저장소. 컴포넌트 간 관계와 무관하게 어디서든 접근 가능하다는 게 props와 가장 다른 점이었다.</p>
<pre><code class="language-jsx">localStorage.setItem(&quot;logInYN&quot;, &quot;true&quot;);
localStorage.getItem(&quot;logInYN&quot;);
localStorage.removeItem(&quot;logInYN&quot;);</code></pre>
<p><code>useState</code>의 초기값과 결합해서 로그인 상태를 새로고침 후에도 유지시켰다.</p>
<pre><code class="language-jsx">const [isLoggedIn, setIsLoggedIn] = useState(
    localStorage.getItem(&quot;logInYN&quot;) === &quot;true&quot;
);</code></pre>
<hr />
<h2 id="✅-구현한-주요-기능-목록">✅ 구현한 주요 기능 목록</h2>
<ul>
<li>로그인 (빈칸 검증, Enter 키 지원, 비밀번호 표시/숨김 토글)</li>
<li>회원가입 (서버에 신규 계정 저장)</li>
<li>로그인 상태 유지 (새로고침 대응)</li>
<li>로그아웃</li>
<li>라우팅 기반 화면 전환 (<code>/</code>, <code>/todo</code>, <code>/signup</code>)</li>
<li>사이드바 (메뉴 목록 렌더링)</li>
<li>Todo 할 일 추가 / 완료 체크 / 선택 삭제</li>
<li>입력 후 자동 포커스 (UX 개선)</li>
<li>할 일 없을 때 안내 문구 (조건부 렌더링)</li>
<li>로그인한 사용자 정보 표시</li>
</ul>
<hr />
<h2 id="🌱-오늘의-정리">🌱 오늘의 정리</h2>
<p>돌아보니 로그인 화면 하나 만드는 것부터 시작해서, 어느새 실제 서버랑 통신하는 것까지 왔다. 특히:</p>
<ul>
<li><code>useState</code> 하나로도 처음엔 헷갈렸는데, 이제는 폼/목록/토글 상태를 자연스럽게 나눠서 관리하게 됐다</li>
<li>props로 함수를 전달하는 패턴(App → TodoPage → Sidebar)을 여러 번 반복하면서 완전히 체화했다</li>
<li>하드코딩된 배열 → json-server 연동까지, &quot;가짜 데이터&quot;에서 &quot;진짜 서버처럼 동작하는 구조&quot;로 넘어가는 과정을 직접 경험했다</li>
</ul>
<h2 id="📝-포트폴리오-작성-시-활용-팁">📝 포트폴리오 작성 시 활용 팁</h2>
<p><strong>한 줄 소개 예시</strong></p>
<blockquote>
<p>React와 react-router-dom을 활용해 SPA 구조의 Todo List 애플리케이션을 구현했으며, json-server와 axios로 실제 클라이언트-서버 통신 흐름을 구성했습니다.</p>
</blockquote>
<p><strong>기술적 도전 포인트로 어필할 수 있는 부분</strong></p>
<ul>
<li>컴포넌트 간 상태 공유를 위한 props 릴레이 패턴 설계 (App → TodoPage → Sidebar)</li>
<li>불변성을 유지하는 상태 업데이트 (spread, map, filter 활용)</li>
<li>하드코딩된 데이터 구조에서 실제 서버 연동 구조로의 리팩터링 경험</li>
<li>제어 컴포넌트 패턴을 활용한 폼 관리 및 재사용 가능한 이벤트 핸들러(<code>keyHandler</code>) 설계</li>
</ul>