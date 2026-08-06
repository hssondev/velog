<h3 id="🔀-라우팅-붙이고-사이드바-처음부터-쌓아올리기">🔀 라우팅 붙이고, 사이드바 처음부터 쌓아올리기</h3>
<blockquote>
<p><strong>TIL 원문</strong>: <a href="https://velog.io/@hssondev/LG-CNS-AM-Inspire-Camp-TIL-7%EC%9D%BC%EC%B0%A8-REACT%EB%9E%80">LG CNS AM Inspire Camp TIL 7일차 - REACT란?</a></p>
</blockquote>
<hr />
<p>오늘은 어제 만든 로그인 화면에 <strong>react-router-dom</strong>을 적용해서 진짜 페이지 이동처럼 만들고, Todo 화면의 <strong>사이드바</strong>를 처음부터 구성해봤다. 헷갈렸던 <code>props</code> 개념도 이번에 확실히 짚고 넘어갔다.</p>
<hr />
<h2 id="🔗-오늘-배운-개념-↔-실제-작업-매칭표">🔗 오늘 배운 개념 ↔ 실제 작업 매칭표</h2>
<p>강의(TIL)에서 배운 개념이 프로젝트 어디에 실제로 쓰였는지 정리해봤다.</p>
<table>
<thead>
<tr>
<th>TIL에서 배운 개념</th>
<th>오늘 프로젝트에서 한 작업</th>
<th>적용 여부</th>
</tr>
</thead>
<tbody><tr>
<td><code>react-router-dom</code><br />(<code>BrowserRouter</code>, <code>Routes</code>, <code>Route</code>)</td>
<td><code>App.js</code>에 라우팅 도입 — <code>/</code>(로그인 or 조건부 Sidebar), <code>/todo</code>(Sidebar) 경로 분리</td>
<td>✅ 적용</td>
</tr>
<tr>
<td><code>useNavigate()</code></td>
<td>로그인 성공 시 <code>LoginPage.jsx</code>에서 <code>moveUrl('/todo')</code>로 강제 이동</td>
<td>✅ 적용</td>
</tr>
<tr>
<td>DOM Event vs React Event<br />(카멜케이스 <code>onClick</code>, 핸들러는 반드시 함수)</td>
<td>로그인 폼의 <code>onClick</code>, <code>onChange</code>, <code>onKeyDown</code> 전부 이 규칙대로 작성</td>
<td>✅ 어제부터 적용, 오늘 원리를 명확히 이해</td>
</tr>
<tr>
<td>React의 Form<br />(<code>value</code> + <code>onChange</code>로 상태 동기화, 제어 컴포넌트)</td>
<td>아이디/비밀번호 input을 <code>id</code>, <code>pwd</code> state와 동기화</td>
<td>✅ 어제부터 적용, 오늘 개념 재확인</td>
</tr>
<tr>
<td>Conditional Rendering<br />(삼항연산자, <code>&amp;&amp;</code>)</td>
<td><code>&lt;Route path=&quot;/&quot;&gt;</code>의 <code>element</code>에 삼항연산자로 로그인 여부에 따라 다른 컴포넌트 렌더링</td>
<td>✅ 적용</td>
</tr>
<tr>
<td><code>useParams()</code> / <code>useLocation()</code> / <code>useSearchParams()</code></td>
<td>아직 동적 경로(<code>/read/:id</code>)나 쿼리스트링을 쓸 일이 없어서 미적용</td>
<td>⏳ 다음에 필요할 때 적용 예정</td>
</tr>
<tr>
<td>순수함수 vs 비순수함수</td>
<td>개념만 이해, 아직 프로젝트에 명시적으로 구분해서 적용하진 않음</td>
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
<td>🔀 라우팅</td>
<td><code>react-router-dom</code>으로 <code>/</code>, <code>/todo</code> 경로 분리</td>
</tr>
<tr>
<td>🧭 페이지 이동</td>
<td><code>useNavigate()</code>로 코드에서 강제 이동</td>
</tr>
<tr>
<td>🔗 컴포넌트 통신</td>
<td>props로 함수 전달하기 (<code>onLogin</code>)</td>
</tr>
<tr>
<td>🧱 사이드바</td>
<td>데이터 → <code>.map()</code> 렌더링 → CSS 스타일링</td>
</tr>
</tbody></table>
<hr />
<h2 id="1️⃣-react-router-dom-적용하기">1️⃣ react-router-dom 적용하기</h2>
<h3 id="왜-필요했나">왜 필요했나</h3>
<p>기존에는 <code>App.js</code>에서 <code>isLoggedIn</code> state 하나로 <code>LoginPage</code>/<code>Sidebar</code>를 삼항연산자로 전환했다. 동작은 했지만, <strong>주소창 URL이 안 바뀐다</strong>는 한계가 있었다. 실제 서비스처럼 <code>/login</code>, <code>/todo</code>가 진짜 페이지 이동처럼 동작하게 만들고 싶어서 라우터를 도입했다.</p>
<h3 id="설치">설치</h3>
<pre><code class="language-bash">npm install react-router-dom</code></pre>
<h3 id="appjs-구조">App.js 구조</h3>
<pre><code class="language-jsx">import { BrowserRouter, Route, Routes } from &quot;react-router-dom&quot;;

function App() {
  const [isLoggedIn, setIsLoggedIn] = useState(
    localStorage.getItem(&quot;logInYN&quot;) === &quot;true&quot;
  );

  const handleLogin = () =&gt; {
    setIsLoggedIn(true);
    localStorage.setItem(&quot;logInYN&quot;, &quot;true&quot;);
  };

  return (
    &lt;BrowserRouter&gt;
      &lt;Routes&gt;
        &lt;Route
          path=&quot;/&quot;
          element={isLoggedIn ? &lt;Sidebar /&gt; : &lt;LoginPage onLogin={handleLogin} /&gt;}
        /&gt;
        &lt;Route path=&quot;/todo&quot; element={&lt;Sidebar /&gt;} /&gt;
      &lt;/Routes&gt;
    &lt;/BrowserRouter&gt;
  );
}</code></pre>
<p>핵심 구성요소:</p>
<table>
<thead>
<tr>
<th>구성요소</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td><code>&lt;BrowserRouter&gt;</code></td>
<td>라우팅 기능 전체를 감싸는 최상위 포장지</td>
</tr>
<tr>
<td><code>&lt;Routes&gt;</code></td>
<td>경로 규칙들을 모아두는 곳</td>
</tr>
<tr>
<td><code>&lt;Route path=&quot;...&quot; element={...}/&gt;</code></td>
<td>&quot;이 주소면 이 컴포넌트&quot; 매핑</td>
</tr>
</tbody></table>
<p><code>path=&quot;/&quot;</code>의 <code>element</code>에는 삼항연산자를 그대로 써서, <strong>이미 로그인된 사람이 <code>/</code>로 들어와도 바로 <code>Sidebar</code>가 보이게</strong> 처리했다. <code>element</code> 자리도 결국 JSX라서 조건부 렌더링이 그대로 적용된다는 걸 알게 됐다.</p>
<h3 id="usenavigate로-로그인-성공-시-이동">useNavigate()로 로그인 성공 시 이동</h3>
<pre><code class="language-jsx">// LoginPage.jsx
import { useNavigate } from &quot;react-router-dom&quot;;

const LoginPage = ({ onLogin }) =&gt; {
  const moveUrl = useNavigate();

  const checkLogin = () =&gt; {
    // ...
    if (found) {
      window.alert(`${found.id}님 환영합니다.`);
      onLogin();
      moveUrl('/todo');
    }
  };</code></pre>
<h3 id="실수했던-부분">실수했던 부분</h3>
<ul>
<li><code>useNavigate()</code>를 <code>checkLogin</code> 함수 <strong>안에서</strong> 호출했다가 에러가 났다. 리액트 hook(<code>use</code>로 시작하는 것들)은 <strong>반드시 컴포넌트 최상위에서만</strong> 호출해야 한다는 규칙을 몰랐어서다. <code>useState</code>와 같은 위치(컴포넌트 함수 바로 아래)로 옮겨야 했다.</li>
<li><code>moveUrl('/todo')</code>를 <code>if(found)</code> 밖에 둬서, 로그인 성공/실패 상관없이 무조건 페이지가 이동해버린 적도 있었다. → 성공 분기 <strong>안으로</strong> 위치를 옮겨서 해결.</li>
</ul>
<hr />
<h2 id="2️⃣-props로-함수-전달하기---헷갈렸던-개념-정리">2️⃣ props로 함수 전달하기 - 헷갈렸던 개념 정리</h2>
<p>오늘 가장 오래 붙잡았던 부분. <strong>부모(<code>App</code>)와 자식(<code>LoginPage</code>)은 서로 완전히 분리된 컴포넌트</strong>라서, 자식이 부모의 state를 직접 바꿀 수 없다는 게 출발점이었다.</p>
<h3 id="해결-흐름">해결 흐름</h3>
<p><strong>1. 부모가 함수를 만들어서, 자식에게 이름표를 붙여 건네줌</strong></p>
<pre><code class="language-jsx">// App.js
const handleLogin = () =&gt; {
  setIsLoggedIn(true);
};

&lt;LoginPage onLogin={handleLogin} /&gt;</code></pre>
<p><strong>2. 자식이 그 이름표(<code>onLogin</code>)로 함수를 받음</strong></p>
<pre><code class="language-jsx">// LoginPage.jsx
const LoginPage = ({ onLogin }) =&gt; {</code></pre>
<p><strong>3. 필요한 순간에 자식이 그 함수를 실행</strong></p>
<pre><code class="language-jsx">if (found) {
  onLogin(); // 사실 App의 handleLogin이 실행되는 것
}</code></pre>
<h3 id="이해에-도움-됐던-비유">이해에 도움 됐던 비유</h3>
<blockquote>
<p>부모(사장님)가 함수(도장)를 만들어서 <code>onLogin</code>이라는 이름으로 자식(직원)에게 건네준다. 직원은 도장 안에 뭐가 들었는지 몰라도, &quot;로그인 성공&quot;이라는 조건이 오면 그냥 도장을 찍는다(<code>onLogin()</code>). 도장이 찍히는 순간 부모의 장부(state)가 바뀐다.</p>
</blockquote>
<p>핵심은 <strong>이름은 그냥 라벨일 뿐이고, 실제로는 똑같은 함수 하나를 서로 다른 이름으로 부르고 있다</strong>는 것. <code>App.js</code>의 <code>handleLogin</code>과 <code>LoginPage.jsx</code>의 <code>onLogin</code>은 실행되는 진짜 코드가 완전히 동일하다. 다만 이름을 넘길 때(<code>&lt;LoginPage onLogin={handleLogin} /&gt;</code>)와 받을 때(<code>({ onLogin })</code>)는 <strong>반드시 똑같은 이름</strong>을 써야 한다.</p>
<hr />
<h2 id="3️⃣-사이드바-만들기-처음부터">3️⃣ 사이드바 만들기 (처음부터)</h2>
<h3 id="목업-참고-구조">목업 참고 구조</h3>
<pre><code>D  Denise
[작업추가 버튼]
검색
관리함
오늘
다음</code></pre><p>프로젝트/팀 그룹은 나중에 &quot;할 일 등록 시 프로젝트 선택&quot; 기능과 함께 연결하기로 하고, 오늘은 프로필 + 메뉴까지만 구성했다.</p>
<h3 id="데이터부터-datasidebardatajs">데이터부터 (<code>data/sidebarData.js</code>)</h3>
<pre><code class="language-js">export const Menu = [
    { id: 1, label: &quot;검색&quot; },
    { id: 2, label: &quot;관리함&quot; },
    { id: 3, label: &quot;오늘&quot; },
    { id: 4, label: &quot;다음&quot; }
];</code></pre>
<h3 id="map으로-렌더링-sidebarjsx"><code>.map()</code>으로 렌더링 (<code>Sidebar.jsx</code>)</h3>
<pre><code class="language-jsx">import { Menu } from &quot;../data/sidebarData&quot;;
import &quot;../components/styled/Sidebar.css&quot;;

const Sidebar = () =&gt; {
  return (
    &lt;div className=&quot;sidebar&quot;&gt;
      &lt;div className=&quot;profile&quot;&gt;
        &lt;span className=&quot;icon&quot;&gt;D&lt;/span&gt;
        &lt;span&gt; Denise&lt;/span&gt;
      &lt;/div&gt;

      &lt;button className=&quot;work&quot;&gt;작업추가&lt;/button&gt;

      &lt;ul className=&quot;menuList&quot;&gt;
        {Menu.map((item) =&gt; (
          &lt;li key={item.id}&gt;{item.label}&lt;/li&gt;
        ))}
      &lt;/ul&gt;
    &lt;/div&gt;
  );
};</code></pre>
<h3 id="실수했던-부분-1">실수했던 부분</h3>
<ul>
<li><code>.map()</code>을 JSX 안에서 그냥 텍스트처럼 썼다가 화면에 아무것도 안 나왔다 → JSX 안에서 JS 코드를 쓰려면 <strong><code>{ }</code>로 감싸야</strong> 한다는 걸 다시 확인.</li>
<li>화살표 함수에서 <code>{ }</code>(중괄호)를 쓰면 <code>return</code>이 필요한데, <code>return</code> 없이 JSX만 써서 아무것도 반환되지 않았던 적도 있었다. → <code>( )</code>(소괄호)를 쓰면 자동으로 그 값이 반환된다는 것도 배웠다.</li>
<li><code>class=&quot;work&quot;</code>처럼 HTML 문법을 그대로 썼다가 에러가 났다. JSX에서는 <code>className</code>을 써야 한다 (<code>for</code> → <code>htmlFor</code>와 같은 이유, JS 예약어와 겹쳐서).</li>
<li><code>&lt;li&gt;</code>를 <code>&lt;ul&gt;</code> 밖에 두거나, <code>&lt;ul&gt;</code> 안에 <code>&lt;button&gt;</code>을 넣는 등 태그 중첩 구조가 꼬였던 걸 하나씩 정리했다. <code>&lt;ul&gt;</code>은 <code>&lt;li&gt;</code>만 자식으로 가져야 한다는 원칙을 다시 확인.</li>
</ul>
<h3 id="css-스타일링">CSS 스타일링</h3>
<pre><code class="language-css">.sidebar {
    width: 200px;
    background-color: ghostwhite;
    padding: 16px;
}

.profile {
    display: flex;
    gap: 20px;
}

.icon {
    background-color: #ddd;
    width: 28px;
    height: 28px;
    border-radius: 50%;
    display: flex;
    justify-content: center;
    align-items: center;
}

.work {
    width: 100%;
    text-align: left;
    border: none;
    padding: 8px;
    cursor: pointer;
}

.menuList {
    list-style: none;
    padding: 0;
    margin: 0;
}

.menuList li {
    padding: 6px 8px;
    cursor: pointer;
}</code></pre>
<h3 id="배운-점">배운 점</h3>
<ul>
<li><code>border-radius: 50%</code> + <code>width</code>/<code>height</code>를 똑같이 주면 정원(正圓)이 된다.</li>
<li><code>display: flex</code> + <code>align-items: center</code> + <code>justify-content: center</code>로 텍스트를 요소 정중앙에 놓을 수 있다.</li>
<li><code>padding</code>에는 <code>auto</code>를 쓸 수 없다 (숫자 값이 필요, <code>auto</code>는 주로 <code>margin</code>에서 씀) — 이것 때문에 한 번 에러가 났었다.</li>
<li><code>list-style: none</code>으로 <code>&lt;ul&gt;</code>의 점(•)을 없앨 수 있다.</li>
</ul>
<hr />
<h2 id="🌱-오늘의-정리">🌱 오늘의 정리</h2>
<ul>
<li><strong>react-router-dom</strong>으로 진짜 페이지 이동처럼 라우팅 구성 (<code>BrowserRouter</code>, <code>Routes</code>, <code>Route</code>, <code>useNavigate</code>)</li>
<li>Hook은 반드시 컴포넌트 최상위에서 호출해야 한다는 규칙</li>
<li><strong>props로 함수를 전달</strong>해서 부모-자식 컴포넌트가 소통하는 패턴을 확실히 이해</li>
<li>사이드바를 데이터 → <code>.map()</code> 렌더링 → CSS 스타일링까지 처음부터 완성</li>
</ul>
<h2 id="📅-다음-할-일">📅 다음 할 일</h2>
<ul>
<li>메뉴 클릭 시 선택된 항목 하이라이트 표시 (<code>useState</code> 활용)</li>
<li>로그아웃 기능</li>
<li>프로젝트/팀 그룹 추가 (할 일 등록 시 프로젝트 선택 기능과 함께)</li>
<li>Todo 항목(할 일 목록) 실제 추가/체크 기능</li>
</ul>
<p>오늘은 특히 컴포넌트 간 데이터 흐름(props)을 확실히 잡고 넘어간 게 큰 수확이었다. 다음엔 이 사이드바에 실제 인터랙션(클릭 시 상태 변화)을 붙여볼 예정이다. 🚀</p>