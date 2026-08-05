<h1 id="⚛️-로그인-화면-리액트로-옮기기">⚛️ 로그인 화면, 리액트로 옮기기</h1>
<p>어제 순수 HTML/CSS/JS로 만든 로그인 화면을 오늘은 리액트(Vite → CRA로 교육 환경 맞춰 재설치)로 옮겼다. 그냥 복사-붙여넣기가 안 되고, 리액트 방식(<code>useState</code>, <code>props</code>)으로 새로 짜야 했는데, 그 과정에서 배운 것들을 정리해본다.</p>
<hr />
<h2 id="📋-오늘-다룬-요청사항-한눈에-보기">📋 오늘 다룬 요청사항 한눈에 보기</h2>
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
<td>빈칸 입력 방지 (React 버전)</td>
<td><code>useState</code>, 제어 컴포넌트</td>
</tr>
<tr>
<td>2️⃣</td>
<td>Enter 키로 로그인 (React 버전)</td>
<td><code>onKeyDown</code></td>
</tr>
<tr>
<td>3️⃣</td>
<td>비밀번호 보이기/숨기기 (React 버전)</td>
<td>삼항연산자로 <code>type</code> 전환</td>
</tr>
<tr>
<td>4️⃣</td>
<td>계정 배열 관리 (React 버전)</td>
<td><code>.find()</code> + <code>data</code> 폴더 분리</td>
</tr>
<tr>
<td>5️⃣</td>
<td>로그인 성공 시 화면 전환</td>
<td><strong>props로 함수 전달</strong></td>
</tr>
<tr>
<td>6️⃣</td>
<td>새로고침해도 로그인 유지</td>
<td><code>localStorage</code> + <code>useState</code> 초기값</td>
</tr>
</tbody></table>
<hr />
<h2 id="0️⃣-시작-전에---프로젝트-구조-정리">0️⃣ 시작 전에 - 프로젝트 구조 정리</h2>
<p>기존 HTML/CSS/JS 코드를 리액트로 옮기기 전에, 실무에서 자주 쓰는 폴더 구조로 먼저 정리했다.</p>
<pre><code>src/
├── components/    # 재사용 가능한 UI 부품 (Sidebar 등)
├── pages/         # 화면 단위 컴포넌트 (LoginPage 등)
├── data/          # 정적 데이터 (계정 목록, 메뉴 목록 등)
└── App.js</code></pre><ul>
<li><strong>pages</strong>: 화면 하나를 통째로 나타내는 것</li>
<li><strong>components</strong>: 여러 화면에서 재사용될 수 있는 작은 조각</li>
</ul>
<hr />
<h2 id="1️⃣-요청-1호---빈칸-체크-react-버전">1️⃣ 요청 1호 - 빈칸 체크 (React 버전)</h2>
<blockquote>
<p>&quot;React로 옮겼으니 input 값을 <code>useState</code>로 관리하고, 빈칸이면 예전처럼 안내 문구가 뜨게 해주세요.&quot;</p>
</blockquote>
<h3 id="핵심-개념---제어-컴포넌트controlled-component">핵심 개념 - 제어 컴포넌트(Controlled Component)</h3>
<p>순수 JS 때는 <code>document.querySelector(...).value</code>로 값을 직접 읽어왔는데, 리액트에서는 <strong>input의 값 자체를 state가 관리</strong>하도록 만든다.</p>
<pre><code class="language-jsx">const [id, setId] = useState(&quot;&quot;);

&lt;input
  value={id}                                // 현재 state 값을 보여줌
  onChange={(e) =&gt; setId(e.target.value)}   // 타이핑할 때마다 state 업데이트
/&gt;</code></pre>
<ul>
<li><code>value={id}</code> — 지금 상자 안에 든 값을 화면에 보여줌</li>
<li><code>onChange</code> — 사용자가 타이핑할 때마다 그 값을 상자에 새로 담음</li>
</ul>
<h3 id="내-로직">내 로직</h3>
<pre><code class="language-jsx">const checkLogin = () =&gt; {
  if (id === &quot;&quot; || pwd === &quot;&quot;) {
    window.alert(&quot;아이디와 비밀번호를 입력해주세요.&quot;);
    return;
  }
  // ...
};</code></pre>
<h3 id="헷갈렸던-점">헷갈렸던 점</h3>
<ul>
<li><code>useState</code>로 선언한 변수 이름(<code>id</code>, <code>setId</code>)과 실제 JSX에서 쓰는 이름이 달라서(<code>account</code>, <code>setAccount</code> 등) 에러가 났었다. → <strong>선언한 이름을 그대로, 정확히 똑같이 써야 한다</strong>는 걸 확인했다.</li>
</ul>
<hr />
<h2 id="2️⃣-요청-2호---enter-키로-로그인-react-버전">2️⃣ 요청 2호 - Enter 키로 로그인 (React 버전)</h2>
<blockquote>
<p>&quot;비밀번호 입력하고 Enter만 눌러도 로그인되게 해주세요.&quot;</p>
</blockquote>
<p>순수 JS 때는 <code>addEventListener(&quot;keydown&quot;, ...)</code>를 따로 등록했는데, 리액트에서는 JSX 안에 바로 이벤트를 붙일 수 있다.</p>
<pre><code class="language-jsx">&lt;input
  onKeyDown={(e) =&gt; {
    if (e.key === &quot;Enter&quot;) {
      checkLogin();
    }
  }}
/&gt;</code></pre>
<p>버튼 클릭과 Enter 키 둘 다 <strong>같은 <code>checkLogin</code> 함수</strong>를 호출하게 만들어서, 검증 로직이 중복되지 않게 했다.</p>
<hr />
<h2 id="3️⃣-요청-3호---비밀번호-보이기숨기기-react-버전">3️⃣ 요청 3호 - 비밀번호 보이기/숨기기 (React 버전)</h2>
<blockquote>
<p>&quot;눈 모양 아이콘 눌러서 비밀번호를 보이게/숨기게 해주세요.&quot;</p>
</blockquote>
<p>원리는 순수 JS 때와 같다 — <code>input</code>의 <code>type</code>을 <code>&quot;password&quot;</code> ↔ <code>&quot;text&quot;</code>로 바꾸는 것. 다만 리액트에서는 <code>document.querySelector</code>로 DOM을 직접 건드리지 않고, <strong>state로 &quot;지금 보이는 상태인지&quot;를 관리</strong>한다.</p>
<pre><code class="language-jsx">const [showPwd, setShowPwd] = useState(false);

&lt;input type={showPwd ? &quot;text&quot; : &quot;password&quot;} ... /&gt;

&lt;span onClick={() =&gt; setShowPwd(!showPwd)}&gt;👁&lt;/span&gt;</code></pre>
<ul>
<li><code>showPwd ? &quot;text&quot; : &quot;password&quot;</code> — 삼항연산자로 상태에 따라 <code>type</code> 값을 다르게</li>
<li><code>!showPwd</code> — 지금 값의 반대로 뒤집기 (토글)</li>
</ul>
<h3 id="실수했던-부분">실수했던 부분</h3>
<ul>
<li><code>&lt;span&gt;</code>을 실수로 <code>&lt;input&gt;</code> 태그 <strong>안에</strong> 넣으려고 했는데, <code>input</code>은 속성만 들어가는 자리라 다른 태그를 담을 수 없다는 걸 알게 됐다. <code>span</code>은 input과 <strong>형제 태그</strong>로 나란히 빼야 했다.</li>
<li><code>&lt;span onClick={...} /&gt;</code>처럼 self-closing으로 닫으면 내용(이모지)이 없어서 화면에 아무것도 안 보인다 — 클릭 이벤트는 동작하지만 사용자 입장에선 &quot;숨겨진 버튼&quot;이 되어버린다는 것도 배웠다.</li>
</ul>
<hr />
<h2 id="4️⃣-요청-4호---계정을-배열로-관리-react-버전">4️⃣ 요청 4호 - 계정을 배열로 관리 (React 버전)</h2>
<blockquote>
<p>&quot;계정이 늘어나도 코드가 안 늘어나게, 배열로 관리해주세요.&quot;</p>
</blockquote>
<h3 id="데이터-분리">데이터 분리</h3>
<p>계정 목록은 <code>data/loginData.js</code>로 따로 뺐다.</p>
<pre><code class="language-js">// data/loginData.js
export const users = [
  { id: &quot;shs10025@naver.com&quot;, pwd: &quot;1234&quot;, role: &quot;user&quot; },
  { id: &quot;admin&quot;, pwd: &quot;admin&quot;, role: &quot;admin&quot; },
  // ...
];</code></pre>
<pre><code class="language-jsx">// LoginPage.jsx
import { users } from &quot;../data/loginData&quot;;</code></pre>
<h3 id="export-default-vs-export-const">export default vs export const</h3>
<p>여기서 <code>export</code>와 <code>import</code> 문법이 헷갈렸는데, 기준을 정리하면:</p>
<table>
<thead>
<tr>
<th></th>
<th><code>export default</code></th>
<th><code>export const</code></th>
</tr>
</thead>
<tbody><tr>
<td>파일당 개수</td>
<td>1개만</td>
<td>여러 개 가능</td>
</tr>
<tr>
<td>가져올 때</td>
<td><code>{}</code> 없이, 이름 자유</td>
<td><code>{}</code>로 감싸고, 이름 그대로</td>
</tr>
<tr>
<td>주로 쓰는 곳</td>
<td>컴포넌트 파일</td>
<td>데이터, 상수</td>
</tr>
</tbody></table>
<blockquote>
<p>💡 <code>{}</code>를 쓰는 이유는 &quot;데이터가 여러 개라서&quot;가 아니라, <strong>named export를 가져올 때 항상 필요한 문법</strong>이기 때문이다.</p>
</blockquote>
<h3 id="find로-계정-찾기"><code>.find()</code>로 계정 찾기</h3>
<pre><code class="language-jsx">const checkLogin = () =&gt; {
  if (id === &quot;&quot; || pwd === &quot;&quot;) {
    window.alert(&quot;아이디와 비밀번호를 입력해주세요.&quot;);
    return;
  }

  const found = users.find((user) =&gt; user.id === id &amp;&amp; user.pwd === pwd);

  if (found) {
    window.alert(`${found.id}님 환영합니다.`);
  } else {
    window.alert(&quot;잘못된 계정입니다.&quot;);
  }
};</code></pre>
<p><code>.find()</code>는 배열을 하나씩 돌면서 조건에 맞는 <strong>첫 번째 객체</strong>를 통째로 반환해준다. 찾으면 그 객체가, 못 찾으면 <code>undefined</code>가 담긴다.</p>
<hr />
<h2 id="5️⃣-요청-5호---로그인-성공하면-화면-전환">5️⃣ 요청 5호 - 로그인 성공하면 화면 전환</h2>
<blockquote>
<p>&quot;알림창만 뜨고 끝나는 게 아니라, 진짜 Todo 화면으로 넘어갔으면 좋겠어요.&quot;</p>
</blockquote>
<p>오늘 가장 헷갈렸던 부분. <strong>부모(App) ↔ 자식(LoginPage) 컴포넌트 간 소통</strong>이 필요했다.</p>
<h3 id="왜-필요한가">왜 필요한가</h3>
<p><code>isLoggedIn</code>이라는 state는 <code>App.js</code>에 있는데, 로그인 성공 여부를 판단하는 <code>checkLogin</code>은 <code>LoginPage.jsx</code> 안에 있다. 서로 다른 컴포넌트라 자식이 부모의 state를 직접 못 건드린다.</p>
<h3 id="해결---props로-함수-전달하기">해결 - props로 함수 전달하기</h3>
<p><strong>1. 부모(App)가 함수를 만들어서 자식에게 건네줌</strong></p>
<pre><code class="language-jsx">// App.js
const handleLogin = () =&gt; {
  setIsLoggedIn(true);
};

&lt;LoginPage onLogin={handleLogin} /&gt;</code></pre>
<p><strong>2. 자식(LoginPage)이 props로 그 함수를 받음</strong></p>
<pre><code class="language-jsx">// LoginPage.jsx
const LoginPage = ({ onLogin }) =&gt; {
  // ...</code></pre>
<p><strong>3. 필요한 순간에 자식이 그 함수를 호출</strong></p>
<pre><code class="language-jsx">if (found) {
  window.alert(`${found.id}님 환영합니다.`);
  onLogin(); // 부모의 handleLogin이 실행되면서 isLoggedIn이 true로 바뀜
}</code></pre>
<h3 id="비유로-정리">비유로 정리</h3>
<blockquote>
<p>부모(사장님)가 함수(도장)를 만들어서 <code>onLogin</code>이라는 이름으로 자식(직원)에게 건네준다. 자식은 도장 안에 뭐가 들었는지 몰라도, &quot;로그인 성공&quot;이라는 조건이 오면 그냥 도장을 찍는다(<code>onLogin()</code>). 도장이 찍히는 순간 부모의 장부(state)가 바뀐다.</p>
</blockquote>
<p>이름은 반드시 일치해야 한다: <code>&lt;LoginPage onLogin={...} /&gt;</code>로 넘겼으면, 받는 쪽도 <code>({ onLogin })</code>으로 똑같이 받아야 한다.</p>
<hr />
<h2 id="6️⃣-요청-6호---새로고침해도-로그인-유지">6️⃣ 요청 6호 - 새로고침해도 로그인 유지</h2>
<blockquote>
<p>&quot;새로고침하면 다시 로그인 화면으로 돌아가버려요. 로그인 상태가 유지됐으면 좋겠어요.&quot;</p>
</blockquote>
<p><code>useState</code>는 새로고침하면 메모리가 초기화되기 때문에 사라진다. 어제 배운 <code>localStorage</code>를 다시 활용했다.</p>
<h3 id="1-로그인-성공-시-localstorage에도-저장">1. 로그인 성공 시 localStorage에도 저장</h3>
<pre><code class="language-jsx">const handleLogin = () =&gt; {
  setIsLoggedIn(true);
  localStorage.setItem(&quot;logInYN&quot;, &quot;true&quot;);
};</code></pre>
<h3 id="2-app이-처음-켜질-때-localstorage-값을-초기값으로-사용">2. App이 처음 켜질 때, localStorage 값을 초기값으로 사용</h3>
<pre><code class="language-jsx">const [isLoggedIn, setIsLoggedIn] = useState(
  localStorage.getItem(&quot;logInYN&quot;) === &quot;true&quot;
);</code></pre>
<ul>
<li><code>localStorage.getItem(&quot;logInYN&quot;)</code> → 저장된 값을 꺼냄 (<code>&quot;true&quot;</code> 또는 <code>null</code>)</li>
<li><code>=== &quot;true&quot;</code> → 정확히 <code>&quot;true&quot;</code> 문자열인지 비교 → 결과가 <code>true</code>/<code>false</code>(boolean)로 나옴</li>
<li>이 결과 전체가 <code>useState</code>의 시작값이 됨</li>
</ul>
<p>즉 &quot;App이 켜질 때, localStorage에 로그인 기록이 있으면 바로 로그인된 상태로 시작한다&quot;는 뜻. 새로고침해도 Sidebar 화면이 유지되는 걸 확인했다.</p>
<hr />
<h2 id="🌱-오늘의-정리">🌱 오늘의 정리</h2>
<p>오늘 가장 크게 배운 건 <strong>&quot;컴포넌트는 서로 독립적이라, state를 공유하려면 props로 함수를 넘겨야 한다&quot;</strong>는 부분이었다. 순수 JS에서는 그냥 전역처럼 아무 데서나 값을 건드릴 수 있었는데, 리액트는 데이터 흐름이 명확하게 &quot;부모 → 자식&quot;으로만 흐른다는 걸 몸으로 익힌 느낌이다.</p>
<ul>
<li><code>useState</code>로 input 값을 실시간 관리하는 제어 컴포넌트 패턴</li>
<li>이벤트를 JSX 안에 직접 붙이는 법 (<code>onClick</code>, <code>onKeyDown</code>, <code>onChange</code>)</li>
<li><code>export default</code> / <code>export const</code>의 차이와 각각의 import 방식</li>
<li>배열 데이터 + <code>.find()</code>로 하드코딩된 조건문을 데이터 기반으로 전환</li>
<li><strong>props로 함수를 전달</strong>해서 자식이 부모의 상태를 바꾸게 하는 패턴</li>
<li><code>localStorage</code> + <code>useState</code> 초기값 조합으로 새로고침에도 상태 유지하기</li>
</ul>
<h2 id="📅-내일-할-일">📅 내일 할 일</h2>
<ul>
<li>로그아웃 기능 만들기 (<code>localStorage.removeItem</code> + <code>setIsLoggedIn(false)</code>)</li>
<li>Sidebar 쪽 마무리 (클릭 시 선택 표시 등)</li>
<li>Todo 항목 추가/체크 기능</li>
</ul>
<p>로그인 → 화면 전환 → 새로고침 유지까지 오늘 한 사이클을 완성해서 뿌듯하다. 내일은 로그아웃부터 이어서! 🚀</p>