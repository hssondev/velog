<hr />
<h3 id="📌-오늘의-한-줄-요약">📌 오늘의 한 줄 요약</h3>
<p>React Router DOM을 활용해 블로그 서비스의 주요 화면을 설계하고, json-server 연동을 통해 회원가입, 로그인, 게시글 관련 기능 흐름을 실습.</p>
<hr />
<h3 id="🧠-오늘-배운-내용">🧠 오늘 배운 내용</h3>
<ul>
<li>json-server 연동한 회원가입, 로그인 구현</li>
<li>styled component를 이용한 UI 작성</li>
<li>메인 페이지 구성(useEffect)</li>
</ul>
<hr />
<h3 id="🤔-헷갈렸던-점--관련-개념">🤔 헷갈렸던 점 &amp; 관련 개념</h3>
<h4 id="1-useeffect---loaddata--">1) <code>useEffect(() =&gt; { loadData() }, [])</code></h4>
<p>컴포넌트가 처음 화면에 그려진 직후(마운트 시) <strong>한 번만</strong> 서버 데이터를 불러오도록, 의존성 배열을 빈 배열 <code>[]</code>로 둔 패턴. <code>axios request</code> 응답 상태가 200일 때만 <code>setBlogs()</code>로 state를 갱신해 화면에 반영한다.</p>
<h4 id="2-keyhandler의-spread-패턴">2) <code>keyHandler</code>의 spread 패턴</h4>
<pre><code class="language-js">const keyHandler = (e) =&gt; {
  const {name, value} = e.target;      // input의 name, value 속성을 구조분해로 꺼냄
  setForm({...form, [name]:value});    // 기존 form은 그대로 복사(spread)하고, name에 해당하는 필드만 새 값으로 덮어씀
}</code></pre>
<p>input 하나가 바뀔 때마다 <code>form</code> 객체 전체를 다시 만들지 않고, <code>...form</code>(spread)으로 기존 값은 그대로 복사한 뒤 <code>[name]</code>(계산된 속성명)으로 바뀐 필드 하나만 덮어쓰는 방식. <code>name=&quot;email&quot;</code>인 input이 바뀌면 <code>{...form, email: 새값}</code>이 되는 것과 같다.</p>
<h4 id="3-dto란">3) DTO란?</h4>
<p>계층(프론트/백엔드, 또는 DB ↔ API) 사이에서 데이터를 주고받기 위해 정의하는 전달용 객체. DB에 저장된 원본 데이터를 그대로 노출하지 않고, 화면이나 API 응답에 필요한 형태로 가공·정리해서 전달하는 역할을 한다.</p>
<h4 id="4-hook의-규칙">4) Hook의 규칙</h4>
<ul>
<li>Hook은 <strong>컴포넌트 최상위</strong>에서만 호출해야 한다 (반복문·조건문·중첩 함수 안에서 호출 금지)</li>
<li>Hook은 <strong>React 함수(컴포넌트 또는 커스텀 Hook)</strong> 안에서만 호출해야 한다</li>
</ul>
<p>→ React는 Hook이 호출되는 <strong>순서</strong>로 어떤 state가 어떤 <code>useState</code>에 해당하는지 구분하기 때문에, 조건문 안에서 Hook을 호출하면 렌더링마다 순서가 달라져 상태가 꼬일 수 있다.</p>
<h4 id="5-querystring을-axios로-다루는-두-가지-방법">5) QueryString을 axios로 다루는 두 가지 방법</h4>
<pre><code class="language-js">// 방법1: URL에 직접 바인딩
api.get(`/users?email=${form.email}&amp;password=${form.password}`);

// 방법2: params 옵션으로 분리 (axios가 자동으로 쿼리스트링을 만들어줌)
api.get('/users', {
  params: { email: form.email, password: form.password }
});</code></pre>
<h4 id="6-bloglist의-조건부-렌더링">6) <code>BlogList</code>의 조건부 렌더링</h4>
<pre><code class="language-jsx">{ props.ary &amp;&amp; props.ary.length &gt; 0 ?      // 배열이 존재하고, 길이가 0보다 클 때만
    props.ary.map((blog, idx) =&gt; &lt;BlogItem key={idx} blog={blog}/&gt;)  // true면 목록 렌더링
    :
    '등록된 글이 없습니다.'                  // false면 안내 문구 렌더링
}</code></pre>
<p>배열이 존재하고(<code>props.ary &amp;&amp;</code>) 길이가 0보다 클 때만 목록을 렌더링하고, 그렇지 않으면 안내 문구를 보여주는 삼항 연산자 기반 조건부 렌더링.</p>
<hr />
<h3 id="💻-실습--적용--코드-흐름-정리">💻 실습 / 적용 — 코드 흐름 정리</h3>
<p>오늘 실습은 <strong>라우팅 → 회원가입/로그인 → 블로그 목록 → 글쓰기</strong>로 이어지는 하나의 사용자 흐름이었다.</p>
<pre><code>ToyApp (라우팅 설정)
   │
   ├─ &quot;/&quot;              → SignUpPage (회원가입)
   │                          │  api.post('/users', data) 성공 시
   │                          ▼
   ├─ &quot;/users/signIn&quot;  → SignInPage (로그인)
   │                          │  api.get('/users?email=...&amp;password=...') 성공 시
   │                          │  localStorage에 user 저장
   │                          ▼
   ├─ &quot;/blogs/index&quot;   → BlogIndexPage (블로그 목록)
   │                          │  useEffect(마운트 시 1회) → api.get('/blogs') → setBlogs()
   │                          │  카테고리 필터링 → BlogList(ary=blogs)에 전달
   │                          │  &quot;글 작성하기&quot; 클릭 시 이동
   │                          ▼
   ├─ &quot;/blogs/write&quot;   → BlogWritePage (글쓰기)
   │                          │  카테고리 선택 + TextInput(title/content)
   │                          │  &quot;이전&quot; 클릭 시 &quot;/blogs/index&quot;로 복귀
   │
   └─ &quot;/blogs/read/:blogId&quot; → BlogReadPage (상세보기, 오늘은 라우팅만 등록)</code></pre><h4 id="--toyappjsx-전체-라우팅">--ToyApp.jsx (전체 라우팅)</h4>
<pre><code class="language-jsx">import { BrowserRouter, Route, Routes } from &quot;react-router-dom&quot;;

//Routerpage
import SignUpPage from &quot;./user/page/SignUpPage&quot;;
import SignInPage from &quot;./user/page/SignInPage&quot;;
import BlogIndexPage from &quot;./features/blog/page/BlogIndexPage&quot;;
import BlogWritePage from &quot;./features/blog/page/BlogWritePage&quot;;
import BlogReadPage from &quot;./features/blog/page/BlogReadPage&quot;;


const ToyApp = () =&gt; {
    return(
        // BrowserRouter: 앱 전체를 라우팅 기능으로 감싸는 최상위 컴포넌트
        &lt;BrowserRouter&gt;
            {/* Routes: 아래 Route들 중 현재 URL과 맞는 것 하나만 렌더링 */}
            &lt;Routes&gt;
                {/* user */}
                &lt;Route path=&quot;/&quot;                     element={&lt;SignUpPage/&gt;} /&gt;        {/* 최초 진입 = 회원가입 */}
                &lt;Route path=&quot;/users/signIn&quot;         element={&lt;SignInPage/&gt;} /&gt;        {/* 로그인 화면 */}
                {/* blog */}
                &lt;Route path=&quot;/blogs/index&quot;          element={&lt;BlogIndexPage/&gt;} /&gt;     {/* 블로그 목록 */}
                &lt;Route path=&quot;/blogs/write&quot;          element={&lt;BlogWritePage/&gt;} /&gt;     {/* 글쓰기 */}
                &lt;Route path=&quot;/blogs/read/:blogId&quot;   element={&lt;BlogReadPage/&gt;} /&gt;      {/* 상세보기, :blogId는 동적 파라미터(useParams로 추출 예정) */}
                {/* blog-comment */}
            &lt;/Routes&gt;
        &lt;/BrowserRouter&gt;
    )
}

export default ToyApp;</code></pre>
<h4 id="--signuppagejsx-①-회원가입">--SignUpPage.jsx (① 회원가입)</h4>
<pre><code class="language-jsx">const SignUpPage = () =&gt; {
    // 이름/이메일/비밀번호를 하나의 객체 상태로 관리 (개별 useState 3개 대신)
    const [form , setForm] = useState({
        name :'',
        email : '',
        password : ''
    })

    // input의 name 속성 값을 key로 사용해, 어떤 필드가 바뀌든 이 함수 하나로 처리
    // 기존값 유지하면서 현재 입력된 필드에 대한 상태변화(업데이트) 처리
    const keyHandler = (e) =&gt; {
      const {name, value} = e.target;   // e.target에서 name, value만 구조분해로 추출
      setForm({...form, [name]:value}); // 나머지 필드는 그대로, 바뀐 필드만 갱신
    }

    const moveUrl = useNavigate(); // 페이지 이동을 위한 훅 (다른 URL로 전환할 때 사용)

    const signUpHandler = async(e) =&gt; {
      e.preventDefault(); // form의 기본 제출(새로고침) 동작을 막음
      console.log(`debug &gt;&gt;&gt;&gt;&gt; SignUpPage`,signUpHandler);
      const data = {...form}; // form 상태를 그대로 복사해서 서버로 보낼 data 생성

      // json-server에 POST 요청 → 회원 데이터 저장
      await api.post('/users', data)
            .then( response =&gt; {
              console.log(`debug &gt;&gt;&gt;&gt;&gt; axios request response`,response);
              if(response.status === 201){       // 201 = 리소스 생성 성공
                moveUrl('/users/signIn');         // 가입 성공 시 로그인 페이지로 이동
              }
            })
            .catch(error =&gt; {
               console.log(`debug &gt;&gt;&gt;&gt;&gt; axios request error`,error);
            })
    }

    return(
        &lt;Container&gt;
                    &lt;FormWrapper&gt;
                        &lt;Title&gt;회원가입&lt;/Title&gt;
                        {/* form 태그의 onSubmit에 연결 → 버튼(type=submit) 클릭 시 signUpHandler 실행 */}
                        &lt;form onSubmit={signUpHandler}&gt;
                            &lt;Input  type='text' 
                                    name='name'                 // keyHandler에서 name 값을 식별자로 사용
                                    placeholder=&quot;이름 입력하세요&quot;
                                    value={form.name}           // 제어 컴포넌트: state 값을 그대로 표시
                                    onChange={keyHandler}
                                    /&gt;
                            &lt;Input  type='email' 
                                    name='email'
                                    placeholder=&quot;이메일 입력하세요&quot;
                                    value={form.email}
                                    onChange={keyHandler}/&gt;
                            &lt;Input  type='password' 
                                    name='password'
                                    placeholder=&quot;패스워드 입력하세요&quot;
                                    value={form.password}
                                    onChange={keyHandler}/&gt;
                            &lt;Button type='submit'&gt;가입하기&lt;/Button&gt;
                        &lt;/form&gt;
                        &lt;TextLink to='/users/signIn'&gt;이미회원이시면 로그인&lt;/TextLink&gt;
                    &lt;/FormWrapper&gt;
                &lt;/Container&gt;
    )       
}

export default SignUpPage;</code></pre>
<h4 id="--signinpagejsx-②-로그인">--SignInPage.jsx (② 로그인)</h4>
<pre><code class="language-jsx">const SignInPage = () =&gt; {

  // state
  const [form,setForm] = useState({
      email: '', password: ''
  });
  const moveUrl = useNavigate();

  // 기존값 유지하면서 현재 입력된 필드에 대한 상태변화(업데이트) 처리
    const keyHandler = (e) =&gt; {
      const {name, value} = e.target;
      setForm({...form, [name]:value});
    };

    /*
    CRUD
    - axios : get(), post(), put() | patch(), delete();
    QueryString(url 뒤에 직접 바인딩) -&gt; router에서도 사용가능 확인!
    - api.get(`url?email=xxxx&amp;password=xxxx`);
    - api.get('url' ,{
          params : {
            email : form.email,
            password: form.password
          }
    })
    DB : SQL

    */
    const signInHandler = async(e) =&gt; {
      e.preventDefault();
      //json-server version
      // 이메일/비밀번호를 쿼리스트링으로 붙여서 GET 요청 (실제 서비스에서는 POST + 서버 인증이 일반적)
      await api.get(`/users?email=${form.email}&amp;password=${form.password}`)
            .then( response =&gt; {
              console.log(`debug &gt;&gt;&gt;&gt;&gt; axios request response`,response);
              if(response.status===200){
                  // 조건에 맞는 사용자를 찾으면 응답 배열의 첫 번째 값(response.data[0])을 로그인 사용자로 취급
                  localStorage.setItem('user',response.data[0].email);

                  //추후 추가작업
                  //header access token 가져오고 싶을 수 있어야함
                  //인증 , 인가 -&gt; JWT or security

                  moveUrl('/blogs/index'); // 로그인 성공 시 블로그 목록으로 이동
              }
            })
            .catch(error =&gt; {
               console.log(`debug &gt;&gt;&gt;&gt;&gt; axios request error`,error);
            });
    //spring version
    };

   return (
        &lt;Container&gt;
            &lt;FormWrapper&gt;
                &lt;Title&gt;로그인&lt;/Title&gt;
                &lt;form onSubmit={signInHandler}&gt;
                    &lt;Input  type='email' 
                            name='email'
                            placeholder=&quot;이메일 입력하세요&quot;
                            value={form.email}
                            onChange={keyHandler}/&gt;
                    &lt;Input  type='password' 
                            name='password'
                            placeholder=&quot;패스워드 입력하세요&quot;
                            value={form.password}
                            onChange={keyHandler}/&gt;
                    &lt;Button type='submit'&gt;로그인&lt;/Button&gt;
                &lt;/form&gt;
            &lt;/FormWrapper&gt;
        &lt;/Container&gt;
    );
}

export default SignInPage;</code></pre>
<h4 id="--blogindexpagejsx-③-블로그-목록">--BlogIndexPage.jsx (③ 블로그 목록)</h4>
<pre><code class="language-jsx">import styled from &quot;styled-components&quot;;
import Button from &quot;../../../components/styled/Button&quot; ;
import BlogList from &quot;../list/BlogList&quot;;
import { useEffect, useState } from &quot;react&quot;;
import api from &quot;../../../api/axios&quot;;
import '../ui/blog.css';
import { useNavigate } from &quot;react-router-dom&quot;;

// 아래 styled(...)들은 전부 &quot;이 화면 전용 스타일이 적용된 컴포넌트&quot;를 미리 만들어두는 부분
const Wrapper = styled.div`
    padding: 16px;
    width: calc(100% - 32px);
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;

`;

const Container = styled.div`
    width: 100%;
    max-width: 720px;

    &amp; &gt; *:not(:last-child) {
        margin-bottom: 16px;
    }

`;

const WelcomeMessage = styled.div`
    font-size: 18px;
    font-weight: bold;
    margin-bottom: 16px;
    color: #333;
`;

const LogoutButton = styled(Button)`
    background-color: #f44336;
    color: white;

    &amp;:hover {
        background-color: #d32f2f;
    }
`;

// ---------- 버튼 영역 ----------
const ButtonRow = styled.div`
    display: flex;
    align-items: center;
    gap: 12px;
    flex-wrap: wrap;

    &amp; &gt; button {
        box-sizing: border-box;
        height: 40px;
        outline: none;
    }

    &amp; &gt; button:focus {
        outline: none;
        box-shadow: none;
    }
`;

// ---------- 카테고리 필터 UI ----------
const CategoryRow = styled.div`
    display: flex;
    flex-wrap: wrap;
    align-items: center;
    gap: 8px;
    width: 100%;
    margin-top: 24px;
`;

const CategoryChip = styled.button`
    box-sizing: border-box;
    display: inline-flex;
    align-items: center;
    justify-content: center;
    flex-shrink: 0;
    white-space: nowrap;
    height: 36px;
    line-height: 1;

    // $active prop 값에 따라 선택된 카테고리 칩의 색상을 다르게 적용
    border: 1.5px solid ${(props) =&gt; (props.$active ? &quot;#6366f1&quot; : &quot;#e5e7eb&quot;)};
    background: ${(props) =&gt; (props.$active ? &quot;#6366f1&quot; : &quot;#ffffff&quot;)};
    color: ${(props) =&gt; (props.$active ? &quot;#ffffff&quot; : &quot;#4b5563&quot;)};
    font-size: 13px;
    font-weight: 600;
    padding: 0 18px;
    border-radius: 999px;
    cursor: pointer;
    transition: all 0.15s;
    outline: none;

    &amp;:focus {
        outline: none;
        box-shadow: none;
    }

    &amp;:hover {
        border-color: #6366f1;
    }
`;
// blog property - title, content, category, email(pk)
const BlogIndexPage = () =&gt; {

    const CATEGORIES = [&quot;전체&quot;, &quot;개발&quot;, &quot;생활&quot;, &quot;취미&quot;, &quot;일상&quot;]; // 카테고리 필터 목록

    const user = localStorage.getItem('user'); // 로그인 시 저장해둔 이메일을 꺼내 환영 메시지에 사용

    const [blogs , setBlogs] = useState([]); // 서버에서 불러온 블로그 목록을 담을 state

    /*
    Q)
    - axios 통신(get(blogs) , params X)
    - 데이터를 reactive state 관리(setXXXX) 
    - 렌더링시점에 데이터 바인딩이 X, side effect  필요함!!
    */
    // 서버에서 블로그 목록을 불러와 state에 반영하는 함수
    const loadData = async () =&gt; {
        // json-server version
        await api.get(`/blogs`)
                .then( response =&gt; {
                    console.log(`debug &gt;&gt;&gt;&gt; axios request success` , response);  
                    if(response.status === 200) {
                        setBlogs(response.data); // 성공하면 blogs state 갱신 → 자동 리렌더링
                    }
                })
                .catch( error =&gt; {
                    console.log(`debug &gt;&gt;&gt;&gt; axios request error` , error); 
                });
    }
    // 컴포넌트가 처음 마운트될 때 딱 1번 loadData 실행 (의존성 배열이 빈 배열이므로)
    useEffect(() =&gt; {
        loadData() ;
    }, []);


    // 선택된 카테고리에 따라 blogs 필터링
    const [selectedCategory, setSelectedCategory] = useState(&quot;전체&quot;); // 현재 선택된 카테고리 chip
    const filteredBlogs = selectedCategory === &quot;전체&quot;
        ? blogs                                                   // &quot;전체&quot;면 그대로
        : blogs.filter((blog) =&gt; blog.category === selectedCategory); // 아니면 해당 카테고리만 추림


    const moveUrl = useNavigate();
    // handler
    const writeHandler = (e) =&gt; {
        moveUrl('/blogs/write');  // 글쓰기 페이지로 이동
    };

    return (
        &lt;Wrapper&gt;
            &lt;Container&gt;
                {/* user가 존재할 때만(로그인 상태일 때만) 환영 메시지 렌더링 */}
                {user &amp;&amp; &lt;WelcomeMessage&gt;{user}님 환영합니다.&lt;/WelcomeMessage&gt;}
                &lt;ButtonRow&gt;
                    &lt;Button title='글 작성하기'
                            onClick={(e) =&gt; writeHandler(e)}&gt;&lt;/Button&gt;
                    &lt;Button title='로그아웃'&gt;&lt;/Button&gt;
                    &lt;Button title='기상예보'&gt;&lt;/Button&gt;
                &lt;/ButtonRow&gt;

                &lt;CategoryRow&gt;
                    {/* CATEGORIES 배열을 순회하며 카테고리 칩을 하나씩 렌더링 */}
                    {CATEGORIES.map((category) =&gt; (
                        &lt;CategoryChip
                            key={category}
                            $active={category === selectedCategory}   // 현재 선택된 카테고리면 활성 스타일
                            onClick={() =&gt; setSelectedCategory(category)} // 클릭 시 필터 상태 변경
                        &gt;
                            {category}
                        &lt;/CategoryChip&gt;
                    ))}
                &lt;/CategoryRow&gt;

                {/* 참고: 실제로는 filteredBlogs가 아닌 blogs를 그대로 넘기고 있음(오늘 실습 코드 원문 그대로) */}
                &lt;BlogList ary={blogs || []}/&gt;

            &lt;/Container&gt;
        &lt;/Wrapper&gt;
    );
}
export default BlogIndexPage ;</code></pre>
<h4 id="--blogwritepagejsx-④-글쓰기">--BlogWritePage.jsx (④ 글쓰기)</h4>
<pre><code class="language-jsx">const BlogWritePage = () =&gt; {
    const user = localStorage.getItem('user');
    const CATEGORIES = [&quot;개발&quot;, &quot;생활&quot;, &quot;취미&quot;, &quot;일상&quot;];

    const moveUrl = useNavigate();

    return(
        &lt;Wrapper&gt;
            &lt;Container&gt;
                {user &amp;&amp; &lt;WelcomeMessage&gt;{user}님 환영합니다.&lt;/WelcomeMessage&gt;}

                {/* 카데코리 선택 */} 
                &lt;CategoryWrapper&gt;
                    &lt;CategoryLabel&gt;카테고리&lt;/CategoryLabel&gt;
                    &lt;CategoryRow&gt;
                        {/* 카테고리 버튼들을 map으로 렌더링 (아직 선택 상태 관리 로직은 없음) */}
                        {
                            CATEGORIES.map((category, idx) =&gt; {
                                return &lt;CategoryChip   key={idx}
                                                type='button'&gt;
                                    {category}
                                &lt;/CategoryChip&gt;
                            })
                        }
                    &lt;/CategoryRow&gt;
                &lt;/CategoryWrapper&gt;

                {/* title 입력용 (height=20으로 한 줄 정도 높이) */}
                &lt;TextInput height={20} /&gt;

                {/* content 입력용 (height=280으로 넓게) */}
                &lt;TextInput height={280} /&gt;

                {/* button */}
                &lt;Button title='글 작성하기' /&gt;
                &amp;nbsp;&amp;nbsp;&amp;nbsp;
                &lt;Button title='이전' 
                        onClick={() =&gt; {
                            moveUrl('/blogs/index') ; // 목록으로 복귀
                        }}/&gt;
            &lt;/Container&gt;
        &lt;/Wrapper&gt;
    )
}

export default BlogWritePage ;</code></pre>
<h4 id="--textinputjsx-글쓰기에-쓰이는-공용-입력-컴포넌트">--TextInput.jsx (글쓰기에 쓰이는 공용 입력 컴포넌트)</h4>
<pre><code class="language-jsx">import styled from &quot;styled-components&quot;;

// props로 받은 height 값이 있을 때만 height 스타일을 추가하는 동적 스타일링
const StyledTextArea = styled.textarea`
    width: calc(100% - 32px);
    ${(props) =&gt;
        props.height &amp;&amp;
        `
        height: ${props.height}px;
    `}
    padding: 16px;
    font-size: 16px;
    line-height: 20px;
    margin-down : 16px ;
    margin-top  : 16px ;
`;

// 여러 페이지에서 재사용할 수 있도록 만든 공용 텍스트 입력 컴포넌트
const TextInput = ({height, value, handler, disabled}) =&gt; {
    return(
        &lt;StyledTextArea 
                    height={height}
                    value={value}
                    handler={handler}
                    disabled={disabled} /&gt;
    );
}

export default TextInput ; </code></pre>
<h4 id="--bloglistjsx-블로그-목록에-쓰이는-리스트-컴포넌트">--BlogList.jsx (블로그 목록에 쓰이는 리스트 컴포넌트)</h4>
<pre><code class="language-jsx">const BlogList = (props) =&gt; {
    return (
        &lt;Wrapper&gt;
           {   // props.ary가 존재하고 원소가 1개 이상이면 목록 렌더링, 아니면 안내 문구
               props.ary &amp;&amp; props.ary.length &gt; 0 ?
                props.ary.map((blog, idx) =&gt; {
                    return &lt;BlogItem    key={idx}      // 리스트 렌더링 시 key는 필수 (React가 항목을 구분하는 용도)
                                        blog={blog} /&gt;  // 블로그 데이터 하나를 props로 BlogItem에 전달
                })
                :
                '등록된 글이 없습니다.'
            }
        &lt;/Wrapper&gt;
    );
}</code></pre>
<blockquote>
<p>💡 오늘 실습을 한 줄로 요약하면, <code>SignUpPage</code>·<code>SignInPage</code>는 <code>useState</code>로 폼 상태를 관리하다가 <code>axios</code>로 서버(json-server)와 통신해 회원가입/로그인을 처리하고, <code>BlogIndexPage</code>는 <code>useEffect</code>로 마운트 시 목록을 불러와 <code>BlogList</code> → <code>BlogItem</code>으로 내려주는 구조. 페이지 간 이동은 전부 <code>useNavigate()</code>가 담당한다.</p>
</blockquote>
<hr />
<h3 id="📚-참고-자료">📚 참고 자료</h3>
<ul>
<li>(오늘은 별도로 정리한 참고 링크 없음)</li>
</ul>
<hr />
<h3 id="🔜-오늘-복습">🔜 오늘 복습</h3>
<ul>
<li>TODO-LIST에 적용해보기</li>
</ul>