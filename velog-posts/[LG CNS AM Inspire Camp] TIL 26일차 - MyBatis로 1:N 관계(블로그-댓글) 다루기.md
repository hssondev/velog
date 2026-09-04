<blockquote>
<p><strong>한 줄 요약</strong></p>
<p>블로그 목록/등록/상세조회 API를 만들고, 어제 로그인 API에서 만든 <code>Authorization</code> 헤더를 오늘 <code>@RequestHeader</code>로 직접 읽어봤다. 블로그 하나에 댓글 여러 개(1:N)를 담는 걸 두 가지 방식으로 짜보고, <code>${}</code>와 <code>#{}</code>을 헷갈려서 생긴 보안 버그도 하나 발견했다.</p>
</blockquote>
<hr />
<h1 id="📚-오늘-학습한-커리큘럼">📚 오늘 학습한 커리큘럼</h1>
<pre><code>블로그 목록/등록 API — 상태코드로 분기하기 (204/200/201/400)
        ↓
상세 조회 API — @PathVariable + @RequestHeader
        ↓
1:N 관계 표현하기 — 블로그(1) : 댓글(N)
        ↓
불변 DTO에 값 추가하기 — toBuilder()
        ↓
@Transactional(readOnly = true)
        ↓
댓글 등록 + &lt;selectKey&gt;로 생성된 ID 즉시 받기</code></pre><h1 id="📌-오늘의-학습-키워드">📌 오늘의 학습 키워드</h1>
<ul>
<li>HTTP 상태코드(204/200/201/400/404)</li>
<li><code>@PathVariable</code>, <code>@RequestHeader</code></li>
<li>1:N 관계, <code>@Builder(toBuilder = true)</code></li>
<li><code>@Transactional(readOnly = true)</code></li>
<li><code>&lt;selectKey&gt;</code></li>
</ul>
<hr />
<h1 id="0-개념-정리-키워드-사전">0. 개념 정리 (키워드 사전)</h1>
<blockquote>
<p><strong>한 줄 요약</strong></p>
<p>MyBatis는 관계형 DB를 그대로 다루기 때문에, &quot;블로그 하나에 댓글 여러 개&quot;를 자바 객체로 표현하려면 우리가 직접 두 번 조회해서 합쳐줘야 한다 — JPA처럼 자동으로 묶어주지 않는다는 걸 오늘 몸으로 겪었다.</p>
</blockquote>
<table>
<thead>
<tr>
<th>키워드</th>
<th>한 줄 정의</th>
<th>비고 / 예시</th>
</tr>
</thead>
<tbody><tr>
<td><strong><code>@PathVariable</code></strong></td>
<td>URL 경로 자체에 박힌 값을 꺼냄</td>
<td><code>/blogs/read/{id}</code>의 <code>{id}</code></td>
</tr>
<tr>
<td><strong><code>@RequestHeader</code></strong></td>
<td>요청의 <strong>헤더</strong>에서 값을 꺼냄</td>
<td><code>@RequestHeader(&quot;Authorization&quot;) String at</code></td>
</tr>
<tr>
<td><strong>1:N 관계</strong></td>
<td>부모 하나에 자식 여러 개가 딸린 관계</td>
<td>블로그 하나 : 댓글 여러 개</td>
</tr>
<tr>
<td><strong><code>@Builder(toBuilder = true)</code></strong></td>
<td>이미 만들어진 객체를 바탕으로, 일부 필드만 바꾼 <strong>새 객체</strong>를 만들 수 있게 해주는 옵션</td>
<td><code>기존객체.toBuilder().필드(새값).build()</code></td>
</tr>
<tr>
<td><strong><code>@Transactional(readOnly = true)</code></strong></td>
<td>이 메서드는 <strong>조회만</strong> 하고 데이터를 바꾸지 않는다고 표시</td>
<td>DB가 내부적으로 최적화할 수 있음</td>
</tr>
<tr>
<td><strong><code>&lt;selectKey&gt;</code></strong></td>
<td><code>INSERT</code> 실행 <strong>직전(또는 직후)</strong>에 값을 미리 조회해서, 그 값을 파라미터에 채워 넣는 MyBatis 문법</td>
<td>시퀀스로 미리 발급받은 id를 응답에 바로 쓸 때</td>
</tr>
</tbody></table>
<hr />
<h1 id="1-블로그-목록등록-api--상태코드로-상황을-표현하기">1. 블로그 목록/등록 API — 상태코드로 상황을 표현하기</h1>
<pre><code class="language-java">@GetMapping(&quot;/index&quot;)
public ResponseEntity&lt;?&gt; index() {
    List&lt;BlogResponseDTO&gt; list = blogService.list();

    // status code : NO_CONTENT(204) , OK(200)
    return list.isEmpty()                                              // ①
            ? ResponseEntity.status(HttpStatus.NO_CONTENT).build()
            : ResponseEntity.status(HttpStatus.OK).body(list);          // ②
}</code></pre>
<p><strong>한 줄씩 정리하면</strong></p>
<ol>
<li><code>list.isEmpty()</code> — 조회된 목록이 <strong>비어있는지</strong> 먼저 확인한다.</li>
<li>비어있으면 <code>204 NO_CONTENT</code>(성공은 했지만 보여줄 내용이 없음), 데이터가 있으면 <code>200 OK</code>와 함께 실제 목록(<code>body(list)</code>)을 반환한다.</li>
</ol>
<ul>
<li>처음엔 <code>.build()</code>만 쓰고 <code>body(list)</code>를 빼먹은 적이 있었는데(파트2 초반), 그러면 상태코드는 맞게 나가도 <strong>정작 데이터는 하나도 안 실려서</strong> 프론트에서 빈 화면만 보게 됐을 것이다. 상태코드와 실제 데이터를 <strong>둘 다</strong> 챙겨야 한다는 걸 이 시행착오로 확인했다.</li>
</ul>
<pre><code class="language-java">@PostMapping(&quot;/insert&quot;)
public ResponseEntity&lt;?&gt; insert(@RequestBody BlogRequestDTO request) {
    int flag = blogService.insert(request);

    return (flag != 0)
        ? ResponseEntity.status(HttpStatus.CREATED).build()    // ③
        : ResponseEntity.status(HttpStatus.BAD_REQUEST).build();
}</code></pre>
<ol start="3">
<li>저장 성공 시 <code>201 CREATED</code>, 실패 시 <code>400 BAD_REQUEST</code>로 상태코드를 구분했다 — 어제 배운 회원가입 API(<code>CREATED</code>/<code>INTERNAL_SERVER_ERROR</code>)와 같은 패턴이 여기서도 반복됐다.</li>
</ol>
<hr />
<h1 id="2-상세-조회-api--pathvariable과-어제-만든-authorization-헤더를-오늘-꺼내-쓰기">2. 상세 조회 API — <code>@PathVariable</code>과, 어제 만든 <code>Authorization</code> 헤더를 오늘 꺼내 쓰기</h1>
<pre><code class="language-java">@GetMapping(&quot;/read/{id}&quot;)
public ResponseEntity&lt;?&gt; read(@PathVariable(&quot;id&quot;) Integer id,              // ①
                               @RequestHeader(&quot;Authorization&quot;) String at) { // ②
    BlogResponseDTO response = blogService.read(id);

    return (response != null)                                              // ③
        ? ResponseEntity.status(HttpStatus.OK).body(response)
        : ResponseEntity.status(HttpStatus.NOT_FOUND).build();
}</code></pre>
<p><strong>한 줄씩 정리하면</strong></p>
<ol>
<li><code>@PathVariable(&quot;id&quot;) Integer id</code> — URL 경로(<code>/read/3</code>)에 박힌 값(<code>3</code>)을 꺼낸다.</li>
<li><code>@RequestHeader(&quot;Authorization&quot;) String at</code> — 요청 헤더에서 <code>Authorization</code> 값을 꺼낸다. <strong>이게 바로 어제 로그인 API에서 <code>HttpHeaders</code>에 담아 보냈던 그 <code>Authorization</code> 헤더</strong>다 — 어제는 &quot;만드는 쪽&quot;이었다면, 오늘은 &quot;받는 쪽&quot;에서 그 값을 실제로 읽어보는 흐름이 이어졌다.</li>
<li>조회 결과가 있으면 <code>200 OK</code>, 없으면 <code>404 NOT_FOUND</code>로 응답한다.</li>
</ol>
<hr />
<h1 id="3-1n-관계-표현하기--블로그-하나에-댓글-여러-개">3. 1:N 관계 표현하기 — 블로그 하나에 댓글 여러 개</h1>
<pre><code class="language-java">public class BlogResponseDTO {
    private Integer id;
    private String title, content, category, email;

    ///////////////////blog(1) : comments (N)
    private List&lt;CommentResoponseDTO&gt; comments;   // ①
}</code></pre>
<pre><code class="language-java">@Transactional(readOnly = true)
public BlogResponseDTO read(Integer id) {
    BlogResponseDTO blog = blogMapper
            .findById(id)
            .orElseThrow(() -&gt; new RuntimeException(id + &quot;BLOG NOT FOUND&quot;));

    blog.setComments(commentMapper.findByBlogId(blog.getId()));   // ②
    return blog;
}</code></pre>
<p><strong>한 줄씩 정리하면</strong></p>
<ol>
<li><code>BlogResponseDTO</code>에 <code>List&lt;CommentResoponseDTO&gt; comments</code> 필드를 추가해서, &quot;블로그 하나가 댓글 여러 개를 가진다&quot;는 1:N 관계를 자바 객체 구조로 표현했다.</li>
<li><code>read(id)</code>에서 <strong>블로그를 먼저 조회</strong>하고, 그 블로그의 <code>id</code>로 <strong>댓글을 따로 한 번 더 조회</strong>해서 <code>setComments(...)</code>로 끼워 넣었다. MyBatis는 JPA처럼 관계를 자동으로 엮어주지 않기 때문에, &quot;부모 조회 → 자식 조회 → 합치기&quot;를 우리가 직접 코드로 짜야 한다는 걸 확인했다.</li>
</ol>
<hr />
<h1 id="4-불변-dto에-값을-추가하는-두-가지-방법">4. 불변 DTO에 값을 &quot;추가&quot;하는 두 가지 방법</h1>
<p>2번 코드(<code>blog.setComments(...)</code>)에 대해, 코드 주석에 스스로 &quot;bad case&quot;라고 적어두고 개선한 부분이 있었다.</p>
<p><strong>Java</strong> · 방식 A — setter로 직접 변경(bad case로 표시됨)</p>
<pre><code class="language-java">blog.setComments(commentMapper.findByBlogId(blog.getId()));
return blog;</code></pre>
<p><strong>Java</strong> · 방식 B — <code>toBuilder()</code>로 새 객체를 만들어서 반환(최종 채택)</p>
<pre><code class="language-java">@Builder(toBuilder = true)   // ①
public class BlogResponseDTO { ... }</code></pre>
<pre><code class="language-java">return blog.toBuilder()                                          // ②
            .comments(commentMapper.findByBlogId(blog.getId()))
            .build();</code></pre>
<p><strong>한 줄씩 정리하면</strong></p>
<ol>
<li><code>@Builder(toBuilder = true)</code> — 클래스에 이 옵션을 추가하면, 이미 만들어진 객체를 바탕으로 <strong>일부 필드만 바꾼 새 객체</strong>를 만드는 <code>toBuilder()</code> 메서드가 생긴다.</li>
<li><code>blog.toBuilder().comments(...).build()</code> — 기존 <code>blog</code> 객체의 다른 필드(<code>title</code>, <code>content</code> 등)는 그대로 유지한 채, <code>comments</code> 필드만 채운 <strong>완전히 새로운 객체</strong>를 만들어서 반환한다.</li>
</ol>
<p>⚠️ <strong>왜 방식 A를 &quot;bad case&quot;라고 적어뒀을까</strong>: <code>setComments(...)</code>는 이미 만들어진 <code>blog</code> 객체를 <strong>그 자리에서 직접 바꿔버린다.</strong> 이렇게 하면 이 객체를 참조하고 있는 다른 코드가 있을 때, 내가 모르는 사이에 그쪽 데이터까지 같이 바뀔 위험이 있다. <code>toBuilder()</code>로 새 객체를 만들어 반환하면, 원본은 그대로 두고 &quot;댓글이 채워진 새 버전&quot;만 돌려주는 셈이라 더 안전하다 — DTO를 <strong>불변(immutable)</strong>하게 다루려는 의도로 보인다.</p>
<hr />
<h1 id="5-transactionalreadonly--true">5. <code>@Transactional(readOnly = true)</code></h1>
<pre><code class="language-java">@Transactional(readOnly = true)
public List&lt;BlogResponseDTO&gt; list() { ... }</code></pre>
<ul>
<li>이 메서드는 데이터를 <strong>조회만</strong> 하고 절대 바꾸지 않는다는 걸 명시적으로 표시했다. 이렇게 표시해두면 DB나 프레임워크가 &quot;이 작업은 변경이 없으니 좀 더 가볍게 처리해도 된다&quot;는 식으로 내부적으로 최적화할 여지가 생긴다고 배웠다. 필기에 &quot;Proxy Transactional: 논리적 작업의 단위&quot;라고 적혀 있던 것도 이 맥락으로, 트랜잭션이 &quot;여러 SQL을 하나의 논리적 작업 단위로 묶어서, 전부 성공하거나 전부 실패하게 만드는&quot; 개념이라는 걸 짚어둔 것으로 보인다.</li>
</ul>
<hr />
<h1 id="6-댓글-등록--selectkey로-방금-만든-id-바로-받기">6. 댓글 등록 — <code>&lt;selectKey&gt;</code>로 방금 만든 ID 바로 받기</h1>
<p>댓글을 저장할 때, <code>id</code>를 시퀀스로 미리 발급받아서 <strong>저장과 동시에 그 id를 응답에 바로 담아 돌려주고 싶은</strong> 상황이었다.</p>
<pre><code class="language-xml">&lt;insert id=&quot;save&quot;
        parameterType=&quot;...CommentRequestDTO&quot;&gt;
    &lt;selectKey keyProperty=&quot;id&quot;    &lt;!-- ① --&gt;
               resultType=&quot;int&quot;
               order=&quot;BEFORE&quot;&gt;      &lt;!-- ② --&gt;
        SELECT NEXTVAL(COMMENT_SEQ)
    &lt;/selectKey&gt;
    INSERT INTO SPRING_COMMENT_TBL
    VALUES(#{id}, #{comment}, #{email}, #{blogId})   &lt;!-- ③ --&gt;
&lt;/insert&gt;</code></pre>
<pre><code class="language-java">public int insert(CommentRequestDTO request) {
    commentMapper.save(request);
    return request.getId();   // ④
}</code></pre>
<p><strong>한 줄씩 정리하면</strong></p>
<ol>
<li><code>keyProperty=&quot;id&quot;</code> — 조회한 값을 파라미터 객체(<code>CommentRequestDTO</code>)의 <code>id</code> 필드에 <strong>자동으로 채워 넣으라</strong>는 뜻이다.</li>
<li><code>order=&quot;BEFORE&quot;</code> — 이 <code>&lt;selectKey&gt;</code>를 <code>INSERT</code> 문 <strong>실행 전</strong>에 먼저 실행하라는 뜻이다. 그래야 <code>INSERT</code>에 쓸 <code>#{id}</code> 값이 미리 준비된다.</li>
<li><code>VALUES(#{id}, ...)</code> — ①에서 채워진 <code>id</code>가 이 자리에 그대로 들어간다.</li>
<li><code>request.getId()</code> — <code>save()</code>가 실행되고 나면, 파라미터로 넘겼던 <code>request</code> 객체 안에 <strong>이미 <code>id</code> 값이 채워져 있다.</strong> 그래서 별도로 다시 조회하지 않고 그 값을 그대로 꺼내 컨트롤러에 돌려줬다.</li>
</ol>
<p>이 흐름이 정착되기 전에는, <code>save()</code>가 그냥 영향받은 행 수(<code>int</code>)만 반환하는 방식으로 짜여 있었다. 그 방식으로는 &quot;방금 저장한 댓글의 id가 몇 번인지&quot; 알 방법이 없어서, 프론트에 새로 등록된 댓글 정보를 바로 돌려주기 어려웠다. <code>&lt;selectKey&gt;</code>로 바꾸고 나서야 저장과 동시에 새 id를 응답에 실어 보낼 수 있게 됐다.</p>
<hr />
<h1 id="✨-새롭게-알게-된-점">✨ 새롭게 알게 된 점</h1>
<ul>
<li>MyBatis에서 1:N 관계는 자동으로 안 엮이고, 부모를 조회한 뒤 자식을 따로 조회해서 직접 합쳐야 한다.</li>
<li><code>#{}</code>는 안전한 바인딩, <code>${}</code>는 텍스트 그대로 끼워 넣는 방식이라 SQL Injection 위험이 있다 — 값이 들어가는 자리는 기본적으로 <code>#{}</code>를 써야 한다.</li>
<li><code>@Builder(toBuilder = true)</code>로 기존 객체를 건드리지 않고 &quot;일부만 바꾼 새 객체&quot;를 만들 수 있다.</li>
<li><code>&lt;selectKey&gt;</code>로 <code>INSERT</code> 전에 값을 미리 채번해서, 저장과 동시에 그 id를 활용할 수 있다.</li>
<li><code>@RequestHeader</code>로 어제 만든 인증 헤더를 오늘 실제로 꺼내 쓰면서, 로그인(헤더를 만드는 쪽)과 이후 API(헤더를 읽는 쪽)가 어떻게 이어지는지 감이 잡혔다.</li>
</ul>
<hr />
<h1 id="🤔-헷갈렸던-점">🤔 헷갈렸던 점</h1>
<ul>
<li><strong><code>${}</code>와 <code>#{}</code>을 실수로 바꿔 쓰는 것</strong>: 겉보기엔 비슷해 보이는데 동작 방식이 완전히 다르다는 걸 오늘에서야 명확히 짚었다. 앞으로 MyBatis XML을 쓸 때마다 이 부분을 계속 확인해야 할 것 같다.</li>
<li><strong>setter로 값 변경 vs <code>toBuilder()</code>로 새 객체 생성</strong>: 둘 다 결과적으로 &quot;댓글이 채워진 blog&quot;를 얻지만, 왜 하나는 &quot;bad case&quot;이고 하나는 아닌지 원리(불변성)는 이해했는데 언제나 <code>toBuilder()</code>가 정답인지는 아직 확신이 없다.</li>
<li><strong><code>@Transactional(readOnly = true)</code>가 정확히 뭘 최적화해주는지</strong>: &quot;조회만 한다고 표시해두면 좋다&quot;는 정도로만 알아뒀고, 실제로 내부에서 어떤 최적화가 일어나는지는 다음에 더 알아봐야겠다.</li>
</ul>
<hr />
<h1 id="다음에-할-일">다음에 할 일</h1>
<ul>
<li><input disabled="" type="checkbox" /> <code>CommentResoponseDTO</code> 오타(<code>Resoponse</code> → <code>Response</code>) 정리하기</li>
<li><input disabled="" type="checkbox" /> <code>@Transactional(readOnly = true)</code>가 실제로 어떤 이점을 주는지 더 찾아보기</li>
<li><input disabled="" type="checkbox" /> 블로그 목록에서도 각 블로그의 댓글 개수를 함께 보여줄 수 있을지 고민해보기</li>
</ul>
<hr />
<h1 id="💡-오늘의-회고">💡 오늘의 회고</h1>
<p><code>blog.setComments(...)</code>를 스스로 &quot;bad case&quot;라고 적어두고 <code>toBuilder()</code>로 바꾼 과정도 인상 깊었다. 당장 동작만 시키려면 setter로 충분했을 텐데, 굳이 &quot;이 객체를 직접 바꾸는 게 맞나?&quot;를 다시 고민하고 더 안전한 방식으로 바꾼 걸 보면서, 동작하는 코드와 좋은 코드는 다를 수 있다는 걸 다시 느꼈다. 어제 만든 <code>Authorization</code> 헤더를 오늘 <code>@RequestHeader</code>로 직접 꺼내 써본 것도, 하루 전 작업이 오늘로 그대로 이어진다는 게 실감 나서 좋았다.</p>