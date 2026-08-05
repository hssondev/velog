<hr />
<h3 id="📌-오늘의-한-줄-요약">📌 오늘의 한 줄 요약</h3>
<p>React 개발 환경을 구성하고, 컴포넌트 개념과 Hooks(useState/useEffect)를 이용해 상태를 다루는 법을 익힌 시간.</p>
<hr />
<h3 id="🧠-오늘-배운-개념-정리">🧠 오늘 배운 개념 정리</h3>
<h4 id="1-개발-환경--프로젝트-구조">1) 개발 환경 &amp; 프로젝트 구조</h4>
<ul>
<li><strong>프로젝트 구조 이해</strong>: React 프로젝트의 폴더(components, pages, api, styles 등) 구성과 각 폴더의 역할</li>
<li><strong>CRA vs Vite(TypeScript) 빌드 비교</strong>: CRA(Create React App)는 webpack 기반으로 프로젝트를 빌드해 개발 서버 구동이 상대적으로 느리고, Vite는 esbuild 기반 개발 서버와 번들링을 사용해 빌드·HMR(핫 리로드) 속도가 더 빠르다는 점이 핵심 차이</li>
<li><strong>.env 의 의미</strong>: API 주소, 키 값처럼 코드에 직접 적으면 안 되거나 개발/운영 환경마다 달라지는 값을 분리해서 관리하는 환경변수 파일</li>
</ul>
<h4 id="2-컴포넌트-개념--props">2) 컴포넌트 개념 &amp; Props</h4>
<ul>
<li><strong>컴포넌트의 개념</strong>: 어제(5일차)에 다룬 대로, UI를 재사용 가능한 단위로 쪼갠 조각</li>
<li><strong>선택적 Property(Optional Prop)</strong>: 컴포넌트를 호출할 때 값을 넘기지 않아도 되도록 만든 prop. 값이 없을 때 쓸 기본값을 따로 지정해둘 수 있음</li>
<li><strong>Props를 활용한 데이터 전달</strong>: 부모 컴포넌트 → 자식 컴포넌트로 단방향으로 값을 전달하는 방식 (오늘 실습의 <code>&lt;Comment data={comment}/&gt;</code>처럼)</li>
<li><strong>컴포넌트 합성(Composition)</strong>: 작은 컴포넌트 여러 개를 조합해서 더 큰 화면을 구성하는 방법. 오늘 실습에서 <code>Button</code>을 <code>CapacityPage</code>에, <code>Comment</code>를 <code>CommentPage</code>에 조합해서 쓴 것이 그 예시</li>
</ul>
<h4 id="3-클래스-컴포넌트-생명주기-vs-hooks">3) 클래스 컴포넌트 생명주기 vs Hooks</h4>
<ul>
<li><strong>클래스 컴포넌트 생명주기</strong>: mount(생성) → update(갱신) → unmount(제거) 단계마다 정해진 메서드가 실행되는 방식</li>
<li><strong>Hooks</strong>: 함수형 컴포넌트에서도 상태·생명주기 같은 기능을 쓸 수 있게 해주는 함수들</li>
<li><strong>useState()</strong>: 컴포넌트 안에서 값이 바뀌면 화면도 같이 갱신되는 &quot;상태&quot;를 만드는 훅. <code>[cnt, setCnt] = useState(0)</code>처럼 현재 값과 값을 바꾸는 함수를 한 쌍으로 반환</li>
<li><strong>useEffect()</strong>: 렌더링이 끝난 뒤에 실행할 부수 작업(사이드 이펙트)을 등록하는 훅. 두 번째 인자인 <strong>의존성 배열</strong>에 따라 실행 시점이 달라짐</li>
</ul>
<table>
<thead>
<tr>
<th>의존성 배열</th>
<th>실행 시점</th>
</tr>
</thead>
<tbody><tr>
<td>넣지 않음</td>
<td>렌더링될 때마다 실행</td>
</tr>
<tr>
<td>빈 배열 <code>[]</code></td>
<td>최초 마운트 시 한 번만 실행</td>
</tr>
<tr>
<td>특정 값 <code>[cnt]</code></td>
<td>마운트 시 + <code>cnt</code> 값이 바뀔 때마다 실행</td>
</tr>
</tbody></table>
<p>→ 오늘 실습(<code>CapacityPage.jsx</code>)은 <code>[cnt]</code>를 의존성 배열에 넣어서, 입장/퇴장 버튼으로 <code>cnt</code>가 바뀔 때마다 콘솔 로그를 다시 찍도록 만든 것.</p>
<h4 id="4-이벤트-처리-방법">4) 이벤트 처리 방법</h4>
<p>JSX에서는 <code>onClick={handler}</code>처럼 카멜케이스 이벤트 속성에 함수를 연결해서 이벤트를 처리한다. (예: <code>&lt;Button onClick={(e) =&gt; upCntHandler()}/&gt;</code>)</p>
<h4 id="5-스타일링">5) 스타일링</h4>
<ul>
<li><strong>내부 스타일 적용</strong>: <code>style={{ color: 'red' }}</code>처럼 객체 형태로 인라인 스타일을 지정하는 방식</li>
<li><strong>Styled Components</strong>: 템플릿 리터럴(백틱) 문법 안에 CSS를 그대로 작성해서, CSS 자체를 하나의 컴포넌트로 만드는 CSS-in-JS 라이브러리 (오늘 실습의 <code>StyledButton</code>)</li>
</ul>
<hr />
<h3 id="🤔-헷갈렸던-점--클래스-컴포넌트-생명주기--hook">🤔 헷갈렸던 점 — 클래스 컴포넌트 생명주기 &amp; Hook</h3>
<h4 id="🧩-클래스-컴포넌트-생명주기life-cycle란">🧩 클래스 컴포넌트 생명주기(Life Cycle)란?</h4>
<p>클래스 컴포넌트는 화면에 나타나서 사라질 때까지 정해진 단계를 거치는데, 각 단계마다 자동으로 호출되는 메서드가 정해져 있다. 이 전체 흐름을 <strong>생명주기</strong>라고 부른다.</p>
<pre><code>Mount(생성)
   │  constructor → render → componentDidMount
   ▼
Update(갱신)  ← props/state가 바뀔 때마다 반복
   │  render → componentDidUpdate
   ▼
Unmount(제거)
   │  componentWillUnmount</code></pre><table>
<thead>
<tr>
<th>단계</th>
<th>대표 메서드</th>
<th>언제 호출되나</th>
<th>주로 하는 일</th>
</tr>
</thead>
<tbody><tr>
<td><strong>Mount</strong></td>
<td><code>componentDidMount</code></td>
<td>컴포넌트가 화면에 처음 그려진 직후, 딱 1번</td>
<td>서버에서 데이터 가져오기(ajax), 타이머 설정</td>
</tr>
<tr>
<td><strong>Update</strong></td>
<td><code>componentDidUpdate</code></td>
<td>props나 state가 바뀌어 다시 렌더링된 직후</td>
<td>바뀐 값에 따라 추가 작업 수행</td>
</tr>
<tr>
<td><strong>Unmount</strong></td>
<td><code>componentWillUnmount</code></td>
<td>컴포넌트가 화면에서 사라지기 직전</td>
<td>타이머 제거, 이벤트 리스너 해제 등 정리 작업</td>
</tr>
</tbody></table>
<hr />
<h4 id="🪝-hook이란">🪝 Hook이란?</h4>
<p>원래 함수형 컴포넌트는 단순히 화면만 그리는 역할이라 상태(state)를 가질 수도 없고, 위와 같은 생명주기 메서드도 쓸 수 없었다. <strong>Hook은 함수형 컴포넌트에서도 상태와 생명주기 같은 클래스 컴포넌트의 기능을 쓸 수 있게 해주는 함수들</strong>이다. <code>use</code>로 시작하는 이름(<code>useState</code>, <code>useEffect</code>...)이 특징이고, 컴포넌트 함수 맨 위쪽에서 호출해서 쓴다.</p>
<hr />
<h4 id="🔗-둘을-잇는-연결고리-useeffect가-생명주기-메서드를-대신한다">🔗 둘을 잇는 연결고리: <code>useEffect</code>가 생명주기 메서드를 대신한다</h4>
<p><code>useEffect</code>는 두 번째 인자로 넘기는 <strong>의존성 배열</strong>을 어떻게 쓰느냐에 따라, 클래스 컴포넌트의 생명주기 메서드 세 개를 한 함수 안에서 전부 대신할 수 있다.</p>
<table>
<thead>
<tr>
<th>클래스 생명주기 메서드</th>
<th>함수 컴포넌트(Hook) 대응</th>
</tr>
</thead>
<tbody><tr>
<td><code>componentDidMount</code></td>
<td><code>useEffect(() =&gt; { ... }, [])</code> — 빈 배열 → 최초 1번만 실행</td>
</tr>
<tr>
<td><code>componentDidUpdate</code></td>
<td><code>useEffect(() =&gt; { ... }, [값])</code> — 배열에 넣은 값이 바뀔 때마다 실행</td>
</tr>
<tr>
<td><code>componentWillUnmount</code></td>
<td><code>useEffect(() =&gt; { return () =&gt; { ... } }, [])</code> — return 안의 함수(클린업)가 사라지기 직전 실행</td>
</tr>
</tbody></table>
<blockquote>
<p>💡 클래스 컴포넌트는 세 가지 상황마다 별도의 메서드를 각각 만들어야 했지만, 함수 컴포넌트에서는 <code>useEffect</code> 하나로 의존성 배열만 다르게 줘서 세 가지 상황을 다 표현할 수 있다. Hook(<code>useState</code>, <code>useEffect</code>)의 등장으로 클래스 없이도 함수 컴포넌트에서 상태 관리와 생명주기 제어가 모두 가능해졌다.</p>
</blockquote>
<hr />
<h3 id="💻-실습--적용">💻 실습 / 적용</h3>
<h4 id="--capacitypagejsx">--CapacityPage.jsx</h4>
<pre><code class="language-jsx">import Button from &quot;../../components/styled/Button&quot;;
import { useState, useEffect } from 'react';

const CapacityPage = () =&gt; {
    console.log(`debug &gt;&gt;&gt;&gt;&gt; CapacityPage lifecycle update `);

    // 고민?
    // 해당변수는 양방향 실시간 소통이 이루어져야한다...어떻게?
    // let cnt = 0;
    // solution =&gt; hook state : useState()
    let [cnt, setCnt] = useState(0);

    const upCntHandler = (e) =&gt; {
        setCnt(cnt =&gt; cnt + 1);
        //console.log(`debug &gt;&gt;&gt;&gt;&gt; upCntHandler ${cnt} `);
    }

    const downCntHandler = (e) =&gt; {
        setCnt(cnt =&gt; cnt - 1);
        //console.log(`debug &gt;&gt;&gt;&gt;&gt; downCntHandler ${cnt} `);
    }

    // side effect 로 렌더링 이후 작업이 필요한 경우?
    // useEffect() 함수를 이용해서 작업할 수 있다.
    // useEffect() 함수를 이용해서 생명주기를 관리할수 있다.
    useEffect(() =&gt; {
        console.log(`debug &gt;&gt;&gt;&gt;&gt; CapacityPage lifecycle mount,unmount `);
        console.log(`debug &gt;&gt;&gt;&gt;&gt; side effect render cnt :  ${cnt} `);
    }, [cnt]);

    ///////////////////////////////////////////////////////////////////////////
    /*
    - 입장인원(10명)
    - 입장 ,퇴장 버튼을 만들어서
    - 입장버튼을 클릭하면 인원수가 증가되고 입장인원이 꽉차면 버튼 비활성화
    - 퇴장버튼을 클릭하면 인원수가 감소되고 인원이 0이 되면 버튼 비활성화
    */
    return(
        &lt;div&gt;
            &lt;p&gt;입장인원 : {cnt} &lt;/p&gt;
            &lt;Button     title=&quot;입장&quot;
                        onClick={(e) =&gt; upCntHandler()}/&gt;

            &lt;Button     title=&quot;퇴장&quot;
                        onClick={(e) =&gt; downCntHandler()}/&gt;
        &lt;/div&gt;
    );
}

export default CapacityPage;</code></pre>
<h4 id="--buttonjsx">--Button.jsx</h4>
<pre><code class="language-jsx">//npm install styled-components
import styled from &quot;styled-components&quot;;

const StyledButton = styled.button`
    padding : 8px 16px;
    font-size : 16px;
    border-radius : 8px;
    cursor : pointer
`;

const Button = (props) =&gt; {
    return(
        &lt;StyledButton onClick={props.onClick}&gt;{props.title}&lt;/StyledButton&gt;
    );
}

export default Button;</code></pre>
<h4 id="--commentpagejsx">--CommentPage.jsx</h4>
<pre><code class="language-jsx">import api from '../../api/axios'
import Comment from '../../components/sample/Comment'

const CommentPage = () =&gt; {
    // script
    console.log(`debug &gt;&gt;&gt;&gt;&gt; CommentPage load event`)
    let comments = [{
            &quot;writer&quot; : &quot;abc&quot;,
            &quot;comment&quot; : &quot;강사님과 함께하는 즐거운 react.&quot;
        },
        {
            &quot;writer&quot; : &quot;def&quot;,
            &quot;comment&quot; : &quot;강사님과 함께하는 재미없는 react.&quot;
        },
        {
            &quot;writer&quot; : &quot;hij&quot;,
            &quot;comment&quot; : &quot;강사님과 함께하는 즐겁지않은 react.&quot;
        }];

    // const loadData = async() =&gt; {
    //     await api.get('/comment')
    //                 .then(response =&gt; {
    //                     console.log(`debug &gt;&gt;&gt;&gt;&gt; response`,response.data);
    //                     comments = response.data;
    //                 })
    //                 .catch(err =&gt; {
    //                     console.log(`debug &gt;&gt;&gt;&gt;&gt; err`,err);
    //                 });
    // }
    // loadData();

    // UI
    return(
        &lt;div&gt;
            {
                comments.map((comment, idx) =&gt; {
                    return &lt;Comment key={idx}
                                    data={comment} /&gt;
                })
            }
        &lt;/div&gt;
    );
}

export default CommentPage;</code></pre>
<h4 id="--commentjsx">--Comment.jsx</h4>
<pre><code class="language-jsx">import '../../styles/book.css';
import placeholder from '../../img/placeholder.png';

const Comment = ({data}) =&gt; {
    return(
        &lt;div className=&quot;wrapper&quot;&gt;
            &lt;div&gt;
                &lt;img    src={placeholder}
                        className='image'&gt;&lt;/img&gt;
            &lt;/div&gt;

            &lt;div&gt;
                &lt;span&gt;{data.writer} &lt;/span&gt;&lt;p/&gt;
                &lt;span&gt;{data.comment} &lt;/span&gt;
            &lt;/div&gt;
        &lt;/div&gt;
    );
}

export default Comment;</code></pre>
<blockquote>
<p>💡 오늘 실습은 크게 두 갈래였다. <code>CapacityPage</code>는 <code>useState</code>+<code>useEffect</code>로 인원수 상태를 관리하고, <code>Button</code> 컴포넌트를 합성해서 입장/퇴장 버튼을 재사용했다. <code>CommentPage</code>는 배열 데이터를 <code>map</code>으로 돌려 <code>Comment</code> 컴포넌트에 <code>props</code>(구조분해로 <code>{data}</code>)를 넘겨 렌더링하는, 5일차 <code>Librarypage</code>/<code>Book</code> 구조가 그대로 확장된 형태였다.</p>
</blockquote>
<hr />
<h3 id="📚-참고-자료">📚 참고 자료</h3>
<ul>
<li>(오늘은 별도로 정리한 참고 링크 없음)</li>
</ul>
<hr />
<h3 id="🔜-내일을-위한-예습">🔜 내일을 위한 예습</h3>
<ul>
<li><input disabled="" type="checkbox" /> 오늘 배운 내용으로 TODO 앱에 적용해보기</li>
</ul>