<hr />
<h3 id="📌-오늘의-한-줄-요약">📌 오늘의 한 줄 요약</h3>
<p><code>react-router-dom</code>으로 페이지 라우팅을 구성하고, DOM 이벤트와 React 이벤트의 차이를 이해하며 로그인 폼을 직접 구현해본 시간.</p>
<hr />
<h3 id="🧠-오늘-배운-개념-정리">🧠 오늘 배운 개념 정리</h3>
<h4 id="1-javascript-함수의-종류--순수함수pure-vs-비순수함수impure">1) JavaScript 함수의 종류 — 순수함수(pure) vs 비순수함수(impure)</h4>
<table>
<thead>
<tr>
<th>구분</th>
<th>순수함수(pure)</th>
<th>비순수함수(impure)</th>
</tr>
</thead>
<tbody><tr>
<td>외부 상태</td>
<td>의존하지 않음(오직 매개변수만 사용)</td>
<td>외부 상태를 참조하거나 변경함</td>
</tr>
<tr>
<td>결과</td>
<td>같은 입력이면 항상 같은 출력 (예측 가능)</td>
<td>같은 입력이어도 결과가 달라질 수 있음</td>
</tr>
<tr>
<td>부수 효과(Side Effect)</td>
<td>없음</td>
<td>있음 (전역 변수 변경, 콘솔 출력, API 호출 등)</td>
</tr>
</tbody></table>
<p>→ 예: <code>(a, b) =&gt; a + b</code>는 순수함수, <code>count += 1</code>처럼 외부 변수를 바꾸는 함수는 비순수함수.</p>
<h4 id="2-dom의-event-vs-react의-event">2) DOM의 Event vs React의 Event</h4>
<table>
<thead>
<tr>
<th>구분</th>
<th>DOM Event</th>
<th>React Event(SyntheticEvent)</th>
</tr>
</thead>
<tbody><tr>
<td>등록 방식</td>
<td><code>addEventListener('click', ...)</code></td>
<td>JSX 안에서 <code>onClick={handler}</code></td>
</tr>
<tr>
<td>이벤트 이름</td>
<td>소문자 (<code>onclick</code>)</td>
<td>카멜케이스 (<code>onClick</code>)</td>
</tr>
<tr>
<td>핸들러 형태</td>
<td>문자열/함수 모두 가능</td>
<td>반드시 함수로 전달</td>
</tr>
<tr>
<td>등록 위치</td>
<td>각 DOM 요소에 개별 등록</td>
<td>실제로는 최상위 루트에서 한 번에 위임 처리</td>
</tr>
<tr>
<td>기본 동작 취소</td>
<td><code>return false</code>로도 가능</td>
<td>반드시 <code>e.preventDefault()</code> 호출 필요</td>
</tr>
</tbody></table>
<p>→ React는 브라우저마다 다른 이벤트 동작을 통일하기 위해 네이티브 이벤트를 감싼 <strong>합성 이벤트(SyntheticEvent)</strong> 객체를 사용한다.</p>
<h4 id="3-react의-form">3) React의 Form</h4>
<p><code>&lt;input value={상태} onChange={(e) =&gt; set상태(e.target.value)}/&gt;</code>처럼, 입력값을 그때그때 state와 동기화시키는 방식(제어 컴포넌트)으로 폼을 다룬다. 오늘 실습(<code>EventPage</code>)의 이메일/비밀번호 입력창이 그 예시.</p>
<h4 id="4-react-router-dom">4) react-router-dom</h4>
<p>페이지 이동(라우팅)을 관리해주는 라이브러리.</p>
<ul>
<li><code>&lt;BrowserRouter&gt;</code>: 라우팅 기능 전체를 감싸는 최상위 컴포넌트</li>
<li><code>&lt;Routes&gt;</code> / <code>&lt;Route path=&quot;...&quot; element={...}/&gt;</code>: 경로별로 어떤 컴포넌트를 보여줄지 매핑</li>
<li><code>&lt;Link&gt;</code>: <code>&lt;a&gt;</code> 태그처럼 페이지를 이동하되, 새로고침 없이 이동시켜주는 컴포넌트</li>
</ul>
<h4 id="5-react-router-dom의-hook-종류">5) react-router-dom의 Hook 종류</h4>
<table>
<thead>
<tr>
<th>Hook</th>
<th>용도</th>
<th>예시</th>
</tr>
</thead>
<tbody><tr>
<td><code>useParams()</code></td>
<td>경로(path)에 포함된 동적 파라미터를 꺼낼 때 (<code>/read/:id</code>)</td>
<td><code>const { id } = useParams();</code></td>
</tr>
<tr>
<td><code>useLocation()</code></td>
<td>현재 URL 정보(pathname, search 등) 전체를 객체로 가져올 때</td>
<td><code>const location = useLocation();</code></td>
</tr>
<tr>
<td><code>useSearchParams()</code></td>
<td>쿼리 스트링(<code>?key=value</code>)만 편하게 읽고 쓸 때</td>
<td><code>const [params] = useSearchParams(); params.get('category')</code></td>
</tr>
</tbody></table>
<p>→ 오늘 실습에서 로그인 실패 시 이동한 <code>/error?category=react&amp;sort=latest</code> 같은 쿼리 스트링은 <code>useSearchParams()</code>로 꺼내 쓰면 된다.</p>
<h4 id="6-conditional-rendering-조건부-렌더링">6) Conditional Rendering (조건부 렌더링)</h4>
<p>상태 값에 따라 보여줄 화면을 다르게 그리는 방식. 삼항 연산자나 <code>&amp;&amp;</code> 연산자를 JSX 안에서 활용해 조건에 따라 다른 요소를 렌더링한다.</p>
<hr />
<h3 id="🤔-헷갈렸던-점">🤔 헷갈렸던 점</h3>
<ul>
<li><strong>reactive status?</strong>: 화면과 실시간으로 연결되는 &quot;상태(state)&quot;라는 개념 자체가 아직 낯설었음 — 일반 변수(<code>let</code>)와 달리, state가 바뀌면 컴포넌트가 자동으로 다시 렌더링된다는 점이 핵심</li>
<li><strong>useEffect, useState의 필요성</strong>: 왜 굳이 훅을 써야 하는지 헷갈렸음 → 일반 변수는 값이 바뀌어도 화면이 갱신되지 않기 때문에, 화면과 동기화가 필요한 값은 <code>useState</code>로 관리해야 함</li>
<li><strong>화살표 함수의 this</strong>: 화살표 함수는 자신만의 <code>this</code>를 만들지 않고, 함수가 정의된 시점의 바깥 스코프 <code>this</code>를 그대로 사용한다는 점이 헷갈렸음</li>
<li><strong>scope(전역/지역) 차이</strong>: 함수 밖에서 선언한 변수(전역)와 함수 안에서 선언한 변수(지역)의 접근 범위 차이를 실습 코드의 <code>email</code>, <code>pswd</code> state를 보며 다시 정리함</li>
</ul>
<hr />
<h3 id="💻-실습--적용">💻 실습 / 적용</h3>
<h4 id="--testrouterappjsx">--TestRouterApp.jsx</h4>
<pre><code class="language-jsx">import { BrowserRouter, Route, Routes } from &quot;react-router-dom&quot;;

//Routerpage
import EventPage from &quot;./pages/event/EventPage&quot;;
import SuccessPage from &quot;./pages/event/SuccessPage&quot;;
import ErrorPage from &quot;./pages/event/ErrorPage&quot;;
import ViewPage from &quot;./pages/event/ViewPage&quot;;


const TestRouterApp = () =&gt; {
    return(
        &lt;BrowserRouter&gt;

            &lt;Routes&gt;
                &lt;Route path=&quot;/&quot;     element={&lt;EventPage/&gt;} /&gt;
                &lt;Route path=&quot;/success&quot;     element={&lt;SuccessPage/&gt;} /&gt;
                &lt;Route path=&quot;/error&quot;     element={&lt;ErrorPage/&gt;} /&gt;
                &lt;Route path=&quot;/read/:id&quot;     element={&lt;ViewPage/&gt;} /&gt;
            &lt;/Routes&gt;
        &lt;/BrowserRouter&gt;
    )
}

export default TestRouterApp;</code></pre>
<h4 id="--eventpagejsx">--EventPage.jsx</h4>
<pre><code class="language-jsx">import Button from 'react-bootstrap/Button';
import 'bootstrap/dist/css/bootstrap.min.css';
import { useEffect, useState } from 'react';
import api from '../../api/axios';
import { useNavigate } from 'react-router-dom';


const EventPage = () =&gt; {
    /*
    변수?
        - scope : 전역, 지역
        - state 
            ex) const data = {                                                
                    id: 'hsson', password : '1234'                                                                                                                            }  
                }   
                상태관리 x
                let id = data.id;
                let password = data.password; 

                상태관리
                const [id, setId] = useState();
                const [password, setPassword] = useState();
                setId(data.id), setPassword(data.password);
    */
    // 사용자의 입력값과 스크립트의 변수를 동기화를 위한 상태관리
    const [email,setEmail]  = useState('');
    const [pswd,setPswd]     = useState('');

    // const emailHandler = (e) =&gt; {
    //     setEmail(e.target.value);

    // }

    // const pswdHandler = (e) =&gt; {
    //     setPswd(e.target.value);

    // }

    //transition 위한 HOOK
    const moveUrl = useNavigate();

    const signInHandler = async(e,email,pswd) =&gt;{
        e.preventDefault();

        await api.get(`/users?email=${email}&amp;pswd=${pswd}`)
            .then(response =&gt; {
                console.log(`debug &gt;&gt;&gt;&gt;&gt; response:` , response);
                const ary = response.data;
                if(ary.length &gt; 0) {
                    // 인증된 사용자 정보관리
                    // sessionStorage, localStorage
                    // 인증 - 신원확인 , 인가 - 특정 url 접근할 수 있는 권한
                    // Json Web Token(JWT) - token(header)
                    // response.header.get('Authorization');
                    // localStorage.setItem('token', 'token-xxxxx');
                    // react component transition
                    const user = ary[0];
                    localStorage.setItem('userName', user.name);
                    moveUrl('/success', {
                        state : {
                            user,
                            from : '/singIn'
                        }
                    });
                } else{ moveUrl('/error?category=react&amp;sort=latest');

                }

            })
            .catch( err =&gt; {
                console.log(`debug &gt;&gt;&gt;&gt;&gt; error:` , err);
            })

    } 

    useEffect(() =&gt; {
        console.log(`debug &gt;&gt;&gt;&gt;&gt; emailHandler email :` , email);
        console.log(`debug &gt;&gt;&gt;&gt;&gt; pwdHandler pwd :` , pswd);
    })

    return (
        &lt;div className='container'&gt;
            &lt;div className=&quot;mb-3 mt-3&quot;&gt;
                &lt;label htmlFor=&quot;email&quot; className=&quot;form-label&quot;&gt;Email:&lt;/label&gt;
                &lt;input  type=&quot;email&quot;
                        className=&quot;form-control&quot;
                        id=&quot;email&quot;
                        placeholder=&quot;Enter email&quot;
                        name=&quot;email&quot;
                        value={email} 
                        onChange={(e) =&gt; {setEmail(e.target.value)}
                        }/&gt;
            &lt;/div&gt;
            &lt;div className=&quot;mb-3&quot;&gt;
                &lt;label htmlFor=&quot;pwd&quot; className=&quot;form-label&quot;&gt;Password:&lt;/label&gt;
                &lt;input  type=&quot;password&quot;
                        className=&quot;form-control&quot;
                        id=&quot;pwd&quot;
                        placeholder=&quot;Enter password&quot;
                        name=&quot;pswd&quot;
                        value={pswd} 
                        onChange={(e) =&gt; {setPswd(e.target.value)}
                        }/&gt;
            &lt;/div&gt;
            &lt;div className=&quot;form-check mb-3&quot;&gt;
                &lt;label className=&quot;form-check-label&quot;&gt;
                    &lt;input  className=&quot;form-check-input&quot;
                            type=&quot;checkbox&quot;
                            name=&quot;remember&quot; /&gt; Remember me
                &lt;/label&gt;
            &lt;/div&gt;
            &lt;Button variant='primary'
                    onClick={(e)=&gt; signInHandler(e,email,pswd)}&gt;SignIn&lt;/Button&gt;
        &lt;/div&gt;
    );
};

export default EventPage;</code></pre>
<blockquote>
<p>💡 <code>EventPage</code>는 이메일/비밀번호 입력값을 <code>useState</code>로 관리하다가, 로그인 버튼 클릭 시 <code>api.get()</code>으로 사용자 정보를 조회하고 <code>useNavigate()</code>로 <code>/success</code> 또는 <code>/error?...</code>로 이동시키는 구조. <code>useEffect</code>에는 의존성 배열을 넣지 않아서, <code>email</code>/<code>pswd</code>가 바뀔 때마다(=렌더링될 때마다) 콘솔에 값을 찍도록 되어 있다.</p>
</blockquote>
<hr />
<h3 id="📚-참고-자료">📚 참고 자료</h3>
<ul>
<li>(오늘은 별도로 정리한 참고 링크 없음)</li>
</ul>
<hr />
<h3 id="🔜-내일을-위한-예습">🔜 내일을 위한 예습</h3>
<ul>
<li><input disabled="" type="checkbox" /> todolist에 테스트용 데이터 받아서 적용해보기</li>
</ul>