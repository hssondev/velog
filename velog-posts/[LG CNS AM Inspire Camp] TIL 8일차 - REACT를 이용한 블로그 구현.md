<h2 id="1-오늘의-맥락--목록·글쓰기에-이어-상세보기와-댓글까지-crud-흐름을-완성하다">1. 오늘의 맥락 — 목록·글쓰기에 이어 상세보기와 댓글까지, CRUD 흐름을 완성하다</h2>
<p>어제까지 블로그 목록(<code>BlogIndexPage</code>)과 글쓰기(<code>BlogWritePage</code>)를 만들어 C(생성)와 R(목록 조회)까지 갖췄다면, 오늘은 상세 페이지(<code>BlogReadPage</code>)와 댓글 기능을 붙여서 게시글 하나를 열람하고, 그 안에서 댓글을 쓰고 지우는 흐름까지 이어 붙였다.</p>
<hr />
<h2 id="2-목록-페이지--usememo로-필터링-캐싱하기">2. 목록 페이지 — <code>useMemo</code>로 필터링 캐싱하기</h2>
<p><strong>JSX</strong> · <code>BlogIndexPage.jsx</code> — 카테고리 필터링 로직 (case 01 vs case 02)</p>
<pre><code class="language-jsx">// 선택된 카테고리에 따라 blogs 필터링
const [selectedCategory, setSelectedCategory] = useState(&quot;전체&quot;);

//case 01
const filteredBlogs = selectedCategory === &quot;전체&quot;
    ? blogs
    : blogs.filter((blog) =&gt; blog.category === selectedCategory);

//case 02
// const filteredBlogs = useMemo(() =&gt;{
//     return  selectedCategory === &quot;전체&quot;
//             ? blogs
//             : blogs.filter((blog) =&gt; blog.category === selectedCategory);
// },[blogs, selectedCategory]);</code></pre>
<p><strong>코드 리뷰</strong></p>
<ul>
<li>case 01은 컴포넌트가 리렌더링될 때마다 <code>filter</code>를 매번 새로 실행한다. <code>selectedCategory</code>나 <code>blogs</code>와 무관한 다른 state(예: 댓글 입력값 같은 것)가 바뀌어 리렌더링돼도 똑같이 다시 계산된다.</li>
<li>case 02(<code>useMemo</code>)는 의존성 배열 <code>[blogs, selectedCategory]</code>에 들어있는 값이 바뀔 때만 다시 계산하고, 그렇지 않으면 이전에 계산해둔 결과를 그대로 재사용한다.</li>
<li><code>BlogList</code>에는 <code>ary={filteredBlogs || []}</code>로 넘겨서, 어떤 케이스를 쓰든 실제로 목록에 반영되도록 했다.</li>
<li>⚠️ 데이터 개수가 아직 적어서 두 방식의 성능 차이를 체감하긴 어려웠지만, 목록이 커지고 필터링 외의 상태(댓글 입력 등)가 같은 페이지에서 자주 바뀌는 상황이 되면 <code>useMemo</code> 쪽이 유리해진다는 걸 이해하는 게 오늘의 목적이었다.</li>
</ul>
<table>
<thead>
<tr>
<th>구분</th>
<th>일반 계산(case 01)</th>
<th><code>useMemo</code>(case 02)</th>
</tr>
</thead>
<tbody><tr>
<td>실행 시점</td>
<td>컴포넌트가 리렌더링될 때마다</td>
<td>의존성 배열 값이 바뀔 때만</td>
</tr>
<tr>
<td>장점</td>
<td>코드가 단순함</td>
<td>불필요한 재계산 방지</td>
</tr>
<tr>
<td>주의할 점</td>
<td>계산이 무거우면 렌더링마다 비용 발생</td>
<td>의존성 배열을 빠뜨리면 최신 값을 못 씀</td>
</tr>
</tbody></table>
<hr />
<h2 id="3-작성-페이지--카테고리-선택과-폼-상태-관리">3. 작성 페이지 — 카테고리 선택과 폼 상태 관리</h2>
<p><strong>JSX</strong> · <code>BlogWritePage.jsx</code> — 제목/내용/카테고리 상태와 등록 핸들러</p>
<pre><code class="language-jsx">//state
const [title,setTitle] = useState('');
const [content,setContent] = useState('');
const [category,setCategory] = useState('');

//handler
const writeHandler = async() =&gt; {
        await api.post(`/blogs`,{
            title,
            content,
            category,
            email:user
    })
        .then(response =&gt; {
        console.log(`debug &gt;&gt;&gt;&gt; axios request success` , response);
        if(response === 201) {
        moveUrl('/blogs/index')}
    })
    .catch();
};</code></pre>
<p><strong>코드 리뷰</strong></p>
<ul>
<li><code>title</code>, <code>content</code>, <code>category</code>를 각각 별도의 <code>useState</code>로 관리하고, <code>writeHandler</code>에서 <code>{title, content, category, email:user}</code> 형태로 묶어 서버에 <code>POST</code>한다. 축약 문법(<code>title</code> = <code>title: title</code>)이 여기서도 쓰였다.</li>
<li>카테고리 칩은 <code>CATEGORIES.map</code>으로 렌더링하면서, 클릭한 칩을 <code>$active={category === cat}</code>로 표시하고 <code>onClick</code>에서 <code>setCategory(cat)</code>으로 선택 상태를 갱신한다.</li>
<li>⚠️ <code>if(response === 201)</code>은 버그다. <code>response</code>는 axios 응답 객체 전체이기 때문에 절대 숫자 <code>201</code>과 같아질 수 없다. 어제(<code>SignUpPage</code>)처럼 <code>response.status === 201</code>로 써야 정상 동작한다. 이 상태로는 글 작성 후 <code>/blogs/index</code>로 이동하는 로직이 실행되지 않는다.</li>
<li>⚠️ <code>.catch()</code>를 인자 없이 비워뒀다. 실패해도 아무 처리가 없어서, 에러가 나도 콘솔조차 남지 않는다. 최소한 <code>.catch(error =&gt; console.log(error))</code> 정도는 필요해 보인다.</li>
<li>⚠️ <code>Container</code> 스타일에 중괄호가 한 겹 더 남아있다(<code>&amp; &gt; *:not(:last-child) {...}</code> 뒤에 닫는 <code>}</code>가 하나 더). styled-components가 관대하게 무시해서 화면은 큰 문제 없이 나오지만, 문법상으로는 정리가 필요하다.</li>
</ul>
<hr />
<h2 id="4-상세-페이지--useparams로-게시글-id-읽고-댓글까지-한-번에-불러오기">4. 상세 페이지 — <code>useParams</code>로 게시글 ID 읽고, 댓글까지 한 번에 불러오기</h2>
<p><strong>JSX</strong> · <code>BlogReadPage.jsx</code> — 라우트 파라미터로 게시글+댓글 조회</p>
<pre><code class="language-jsx">const {blogId} = useParams();
const user = localStorage.getItem('user');

//블로그정보
const [blog , setBlog]           = useState({});
//댓글배열정보
const [comments , setComments]   = useState({});
//댓글입력
const [comment,setComment] = useState('');

const loadData = async() =&gt; {
    const id = blogId;
    await api.get(`/blogs/${id}?_embed=comments`)
        .then(response =&gt; {
            console.log(`debug &gt;&gt;&gt;&gt; axios request success` , response);  
                if(response.status === 200) {
                    setBlog(response.data);
                    // 댓글정보 배열에 정보를 담아본다면? - reactive state
                    setComments(response.data.comments);
                }
        })
        .catch( error =&gt; {
            console.log(`debug &gt;&gt;&gt;&gt; axios request error` , error); 
        });
}
useEffect(() =&gt; {
    loadData();
},[]);</code></pre>
<p><strong>코드 리뷰</strong></p>
<ul>
<li><code>useParams()</code>로 <code>/blogs/read/:blogId</code> 경로에 담긴 <code>blogId</code> 값을 꺼낸다. 어제 라우팅만 등록해두고 실제로 못 써봤던 훅을 오늘 처음 써봤다.</li>
<li><code>?_embed=comments</code>는 json-server가 지원하는 기능으로, 게시글을 조회할 때 그 게시글에 달린 댓글까지 <code>response.data.comments</code>에 함께 담아 내려준다. 게시글 요청 따로, 댓글 요청 따로 두 번 보내지 않아도 되는 게 장점이다.</li>
<li>원래 있던 이전 버전(주석 처리된 코드)은 게시글만 단독으로 조회했는데, 오늘은 <code>_embed</code>를 추가해서 한 번의 통신으로 게시글+댓글을 같이 받아오는 방식으로 바뀌었다.</li>
<li>로딩 처리도 눈에 띄었다. <code>{!blog.id &amp;&amp; &lt;Spinner/&gt;}</code>로 아직 데이터가 없을 때는 스피너를, <code>{blog.id &amp;&amp; &lt;Container&gt;...}</code>로 데이터가 도착한 뒤에는 실제 내용을 보여준다. <code>blog.id</code>가 있는지 없는지로 로딩 여부를 판단하는 방식이다.</li>
<li>⚠️ <code>comments</code> state의 초기값이 <code>useState({})</code>(빈 객체)인데, 실제로는 <code>setComments(response.data.comments)</code>로 배열을 담고, <code>commentDeleteHandler</code>에서도 <code>comments.filter(...)</code>처럼 배열 메서드를 쓰고 있다. 초기값을 <code>useState([])</code>(빈 배열)로 맞춰두는 게 더 안전해 보인다.</li>
</ul>
<hr />
<h2 id="5-댓글-crud--함수형-업데이트로-배열-상태에-항목-추가삭제하기">5. 댓글 CRUD — 함수형 업데이트로 배열 상태에 항목 추가/삭제하기</h2>
<p><strong>JSX</strong> · <code>BlogReadPage.jsx</code> — 댓글 작성/삭제 핸들러</p>
<pre><code class="language-jsx">//comment handler
const commentHandler = async(e) =&gt; {
    let email = user;
    await api.post(`/comments`, {blogId : Number(blogId),comment,email})
            .then(response =&gt; {
                console.log(`debug &gt;&gt;&gt;&gt; axios request success` , response);
                    if(response.status===201) {
                        setComments( ary =&gt; {
                           return  [...ary,response.data]
                        });
                        setComments('');
                    }
            })
            .catch(error =&gt; {
                console.log(`debug &gt;&gt;&gt;&gt; axios request error` , error); 
            })
};

//comment delete  handler
const commentDeleteHandler = async(e ,id) =&gt; {
    console.log(`debug &gt;&gt;&gt;&gt;&gt;  commentDeleteHandler event`);
    await api.delete(`/comments/${id}`)
        .then(response =&gt; {
            console.log(`debug &gt;&gt;&gt;&gt; axios request success` , response);
                if(response.status === 200) {
                    setComments(comments.filter( (c) =&gt;{
                        return c.id !== id
                    }));
                }
             })
            .catch(error =&gt; {
                console.log(`debug &gt;&gt;&gt;&gt; axios request error` , error); 
            })
}</code></pre>
<p><strong>코드 리뷰</strong></p>
<ul>
<li><code>commentHandler</code>에서 <code>setComments(ary =&gt; [...ary, response.data])</code>는 함수형 업데이트다. 현재 배열(<code>ary</code>)을 그대로 펼치고 서버가 돌려준 새 댓글(<code>response.data</code>)을 뒤에 추가한 새 배열을 만들어 교체한다. 원본 배열을 직접 <code>push</code>하지 않고 항상 새 배열을 만든다는 점이 포인트.</li>
<li><code>commentDeleteHandler</code>는 반대로 <code>comments.filter(c =&gt; c.id !== id)</code>로, 삭제할 댓글의 <code>id</code>만 제외한 새 배열을 만들어 교체한다. 배열에서 항목을 지울 때도 <code>splice</code>처럼 원본을 직접 건드리지 않고 <code>filter</code>로 &quot;남길 것만 골라 새로 만든다&quot;는 흐름이 axios 응답 처리 패턴과 잘 맞물렸다.</li>
<li>댓글 등록/삭제 각각 서버에 먼저 요청(<code>post</code>/<code>delete</code>)을 보내고, 성공(<code>201</code>/<code>200</code>) 응답을 받은 뒤에야 화면 state를 갱신한다. 화면을 먼저 바꾸고 서버에 맞추는 방식이 아니라, 서버 확인 후 화면을 갱신하는 순서다.</li>
<li>⚠️ <code>commentHandler</code> 마지막의 <code>setComments('')</code>는 버그로 보인다. 문맥상 댓글 입력창을 비우려던 의도라면 <code>setComment('')</code>(단수, 입력값 state)를 호출해야 하는데, 실제로는 <code>setComments('')</code>(복수, 댓글 배열 state)를 호출하고 있다. 이대로면 방금 쌓아올린 댓글 배열이 문자열 <code>''</code>로 통째로 덮어써져서, 화면의 댓글 목록이 사라지는 문제가 생길 수 있다.</li>
</ul>
<p><strong>JSX</strong> · <code>BlogCommentList.jsx</code> — 댓글 배열을 하나씩 <code>BlogCommentItem</code>으로</p>
<pre><code class="language-jsx">import styled from &quot;styled-components&quot;;
import BlogCommentItem from &quot;../item/BlogCommentItem&quot;;

const Wrapper = styled.div`
    display: flex;
    flex-direction: column;
    align-items: flex-start;
    justify-content: center;
    margin-top: 16px;
    &amp; &gt; * {
        :not(:last-child) {
            margin-bottom: 16px;
        }
    }
    gap: 16px;
`;

const BlogCommentList = ({comments,handler}) =&gt; {

    return(
        &lt;Wrapper&gt;
            {
                comments.map((comment, idx) =&gt; {
                    return &lt;BlogCommentItem
                                    key = {idx}
                                    comment={comment} 
                                    handler ={handler}/&gt;
                })
            }
        &lt;/Wrapper&gt;
    );

}

export default BlogCommentList;</code></pre>
<p><strong>JSX</strong> · <code>BlogCommentItem.jsx</code> — 댓글 하나 + 작성자 본인일 때만 삭제 버튼</p>
<pre><code class="language-jsx">import styled from &quot;styled-components&quot;;
import Button from &quot;../../../components/styled/Button&quot;;


const Wrapper = styled.div`
    width: calc(100% - 32px);
    padding: 16px;
    display: flex;
    flex-direction: row;
    align-items: center;
    justify-content: space-between;
    border: 1px solid grey;
    border-radius: 8px;
    cursor: pointer;
    background: white;
    :hover {
        background: lightgrey;
    }
`;

const CommentText = styled.p`
    font-size: 20px;
    white-space : pre-wrap ;
`;

const BlogCommentItem = ({comment,handler}) =&gt; {
    //로그인사용자 이메일
    const user = localStorage.getItem('user');
    return (
        &lt;Wrapper&gt;
            &lt;CommentText&gt;{comment.comment}&lt;/CommentText&gt;
            {
                user === comment.email &amp;&amp;
                    &lt;div&gt;
                        &lt;Button title ='삭제'
                                onClick={(e) =&gt; handler(e, comment.id)}/&gt;
                    &lt;/div&gt;
            }
        &lt;/Wrapper&gt;
    )
}

export default BlogCommentItem;</code></pre>
<p><strong>코드 리뷰</strong></p>
<ul>
<li><code>BlogCommentList</code>는 <code>comments</code> 배열을 받아 <code>map</code>으로 <code>BlogCommentItem</code>을 하나씩 찍어내고, 삭제 함수(<code>handler</code>)를 그대로 자식에게 다시 내려준다(props drilling).</li>
<li><code>BlogCommentItem</code>은 <code>user === comment.email</code>일 때만 삭제 버튼을 보여준다. 로그인한 사용자와 댓글 작성자 이메일이 같을 때만 자기 댓글을 지울 수 있게 만든 조건부 렌더링이다.</li>
<li>클릭 시 <code>handler(e, comment.id)</code>로 이벤트와 댓글 id를 함께 부모(<code>BlogReadPage</code>)의 <code>commentDeleteHandler</code>로 전달한다.</li>
</ul>
<hr />
<h2 id="6-memoization-hook--usememo란">6. Memoization Hook — <code>useMemo</code>란?</h2>
<p>렌더링마다 새로 계산하기엔 비용이 큰 값을, 의존성 배열의 값이 바뀔 때만 다시 계산하고 그렇지 않으면 이전 계산 결과를 그대로 재사용하게 해주는 Hook이다.</p>
<pre><code>렌더링 발생
   │
   ▼
의존성 배열 값이 이전과 같은가?
   │
   ├─ 같다 → 캐시된 이전 계산 결과를 그대로 반환
   └─ 다르다 → 콜백 함수를 다시 실행해서 새 값을 계산 + 캐시 갱신</code></pre><p><code>useEffect</code>가 &quot;부수 효과를 언제 실행할지&quot;를 제어하는 훅이라면, <code>useMemo</code>는 &quot;값 계산을 언제 다시 할지&quot;를 제어하는 훅이라는 점에서 역할이 다르다.</p>
<hr />
<h2 id="7-헷갈렸던-점-다음-복습-포인트-🤔">7. 헷갈렸던 점 (다음 복습 포인트) 🤔</h2>
<ul>
<li><strong><code>useParams</code> vs <code>useSearchParams</code></strong>: 둘 다 URL에서 값을 꺼낸다는 점은 같지만, <code>useParams</code>는 <code>/blogs/read/:blogId</code>처럼 경로 자체에 박혀있는 값을 꺼내고, <code>useSearchParams</code>는 <code>?key=value</code> 형태의 쿼리 스트링을 꺼낸다는 차이를 오늘 실습(<code>useParams</code>)으로 다시 확인했다.</li>
<li><strong><code>setComments(ary =&gt; [...ary, response.data])</code> 구조</strong>: 왜 <code>setComments([...comments, response.data])</code>처럼 바로 쓰지 않고 함수 형태로 감쌌는지 헷갈렸다. 짧은 시간에 상태 변경이 겹칠 수 있는 상황(연속 클릭 등)에서, 클로저에 갇힌 오래된 <code>comments</code> 값이 아니라 React가 그 시점에 실제로 들고 있는 최신 배열을 기준으로 추가하기 위한 패턴이라고 정리했다.</li>
<li><strong><code>commentHandler</code>/<code>commentDeleteHandler</code>의 흐름</strong>: 두 핸들러 모두 &quot;서버 요청 → 성공 응답 확인 → 그 다음에 화면 state 갱신&quot; 순서로 짜여 있다는 공통 흐름을 이해하는 데 시간이 걸렸다. 화면을 먼저 바꾸고 실패하면 되돌리는 방식(낙관적 업데이트)이 아니라는 점이 헷갈렸던 포인트.</li>
</ul>
<hr />
<h2 id="8-다음에-할-일">8. 다음에 할 일</h2>
<ul>
<li><input disabled="" type="checkbox" /> api key 발급 참고 사이트 정리해두기<ul>
<li>OpenWeather: <a href="https://icedhotchoco.tistory.com/entry/OpenWeatherMap-%EB%82%A0%EC%94%A8-API#google_vignette">https://icedhotchoco.tistory.com/entry/OpenWeatherMap-%EB%82%A0%EC%94%A8-API#google_vignette</a></li>
<li>Kakao: <a href="https://fromnowwon.tistory.com/entry/kakao-api">https://fromnowwon.tistory.com/entry/kakao-api</a></li>
</ul>
</li>
</ul>
<hr />
<h2 id="마무리-회고">마무리 회고</h2>
<p>어제까지는 화면 하나하나를 따로 떼어서 봤다면, 오늘은 그 화면들이 실제로 이어지는 걸 처음 느꼈다. 목록에서 글을 클릭하면 상세 페이지로 넘어가고, 거기서 댓글을 달고 지우는 흐름을 한 번에 확인하니 &quot;토이프로젝트&quot;라는 말이 왜 붙었는지 알 것 같았다.</p>
<p><code>useMemo</code>는 처음엔 <code>useEffect</code>랑 뭐가 다른지 헷갈렸는데, 하나는 &quot;언제 실행할지&quot;를 정하는 거고 다른 하나는 &quot;언제 다시 계산할지&quot;를 정하는 거라고 구분하고 나니 정리가 됐다.</p>
<p>코드를 다시 읽으면서 <code>response === 201</code>이나 <code>setComments('')</code>처럼 눈에 잘 안 띄는 실수도 몇 개 찾았다. 당장 화면이 크게 안 깨져서 넘어가기 쉬운 부분들인데, 내일은 이런 것부터 하나씩 고쳐보면서 다시 점검해야겠다.</p>