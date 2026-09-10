<blockquote>
<p><strong>한 줄 요약</strong></p>
<p>&quot;Bearer XXXXX&quot; 가짜 토큰을 진짜 JWT로 바꾸고, <code>JwtFilter</code>로 검증하고, 프론트에서도 모든 요청에 그 토큰을 실어 보내도록 연결했다. 그 과정에서 <code>BlogService</code>/<code>CommentService</code>를 MyBatis에서 JPA로 완전히 갈아끼웠다.</p>
</blockquote>
<hr />
<h1 id="📚-오늘-학습한-커리큘럼">📚 오늘 학습한 커리큘럼</h1>
<pre><code>JwtProvide — 가짜 토큰에서 진짜 JWT 발급으로
        ↓
JwtFilter — 요청을 가로채서 토큰 검증
        ↓
Authorization 헤더 형식 통일 — 백엔드(발급) ↔ 프론트(전송) ↔ 필터(검증)
        ↓
BlogService/BlogRepository — MyBatis Mapper를 JPA로 완전히 교체
        ↓
CommentService — Dirty Checking으로 update 완성</code></pre><h1 id="📌-오늘의-학습-키워드">📌 오늘의 학습 키워드</h1>
<ul>
<li>JWT(Access Token/Refresh Token), <code>Jwts.builder()</code></li>
<li><code>Filter</code>, CORS preflight(<code>OPTIONS</code>), 화이트리스트</li>
<li>JPQL, <code>LEFT JOIN FETCH</code></li>
<li>Dirty Checking, <code>@Transactional</code></li>
</ul>
<hr />
<h1 id="0-개념-정리-키워드-사전">0. 개념 정리 (키워드 사전)</h1>
<blockquote>
<p><strong>한 줄 요약</strong></p>
<p>오늘은 &quot;인증(토큰)&quot;이라는 하나의 기능이 프론트와 백엔드 세 군데(발급/전송/검증)에 걸쳐 완성되는 걸 직접 봤고, 백엔드 데이터 계층은 MyBatis에서 JPA로 옮겨갔다.</p>
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
<td><strong>AT/RT (Access/Refresh Token)</strong></td>
<td>AT는 짧은 수명의 인증 토큰, RT는 AT를 재발급받기 위한 긴 수명의 토큰</td>
<td>오늘 각각 30분/일주일로 설정</td>
</tr>
<tr>
<td><strong><code>signWith(key)</code></strong></td>
<td>발급하는 토큰에 시크릿 키로 <strong>서명</strong>을 넣는 것</td>
<td>없으면 위조 방지가 안 됨</td>
</tr>
<tr>
<td><strong>preflight(<code>OPTIONS</code>) 요청</strong></td>
<td>브라우저가 실제 요청 전에 &quot;이 요청 보내도 되는지&quot; 먼저 물어보는 사전 요청</td>
<td>CORS 환경에서 자동 발생</td>
</tr>
<tr>
<td><strong>화이트리스트</strong></td>
<td>인증 없이 통과시킬 경로 목록</td>
<td>로그인·회원가입·Swagger 문서 등</td>
</tr>
<tr>
<td><strong>JPQL</strong></td>
<td>테이블이 아니라 <strong>엔티티</strong>를 대상으로 쓰는 JPA 전용 쿼리 언어</td>
<td>SQL과 문법은 비슷하지만 대상이 다름</td>
</tr>
<tr>
<td><strong><code>LEFT JOIN FETCH</code></strong></td>
<td>연관된 엔티티를 LAZY로 남기지 않고 <strong>한 번에 진짜로 같이 가져옴</strong></td>
<td><code>LazyInitializationException</code> 예방</td>
</tr>
<tr>
<td><strong>Dirty Checking(변경 감지)</strong></td>
<td>조회 시점 스냅샷과 현재 상태를 비교해서 다르면 자동으로 UPDATE</td>
<td><code>save()</code>를 직접 안 불러도 됨</td>
</tr>
</tbody></table>
<h3 id="코드로-다시-보기">코드로 다시 보기</h3>
<pre><code class="language-java">@Query(&quot;&quot;&quot;
    SELECT      b
    FROM        BlogEntity b
    LEFT JOIN   FETCH   b.comments
    WHERE       b.blogId = :blogId
&quot;&quot;&quot;)
public Optional&lt;BlogEntity&gt; findByComments(@Param(&quot;blogId&quot;) Integer blogId);</code></pre>
<hr />
<h1 id="1-jwtprovide--가짜-토큰에서-진짜-jwt로">1. <code>JwtProvide</code> — 가짜 토큰에서 진짜 JWT로</h1>
<p>먼저 로그인할 때 토큰이 어떻게 만들어지는지부터 보자.</p>
<p><img alt="" src="https://velog.velcdn.com/images/hssondev/post/b612fce9-6b6e-4a4b-baa7-a588e8e1b28a/image.png" /></p>
<ol>
<li><strong>React 로그인 폼</strong>에서 이메일·비밀번호를 제출하면 백엔드에 요청이 간다.</li>
<li><strong><code>UserController.signIn()</code></strong>이 이메일·비밀번호가 DB 값과 일치하는지 확인한다.</li>
<li>일치하면 <strong><code>JwtProvide.createAT()</code>/<code>createRT()</code></strong>가 서명이 들어간 진짜 토큰 두 개를 만든다.</li>
<li>이 토큰을 응답 헤더에 담아 돌려주면, 프론트는 <code>localStorage</code>에 저장해둔다.</li>
</ol>
<p>처음엔 <code>createAT()</code>가 <code>&quot;Bearer XXXXX&quot;</code>라는 고정 문자열만 돌려주는 자리 표시자였다. 여기에 실제 발급 로직을 채워넣었다.</p>
<pre><code class="language-java">private final long ACCESS_TOKEN_EXPIRY  = 1000L * 60 * 30;        // 30분
private final long REFRESH_TOKEN_EXPIRY = 1000L * 60 * 60 * 24 * 7; // 일주일

private Key getSecretKey() {
    return Keys.hmacShaKeyFor(secret.getBytes(StandardCharsets.UTF_8));
}

public String createAT(String email) {
    return Jwts.builder()
                .setSubject(email)
                .setIssuedAt(new Date())
                .setExpiration(new Date(System.currentTimeMillis() + ACCESS_TOKEN_EXPIRY))
                .signWith(getSecretKey())
                .compact();
}</code></pre>
<ul>
<li><code>setSubject(email)</code> — 이 토큰이 &quot;누구의&quot; 것인지 담는다.</li>
<li><code>setExpiration(...)</code> — 만료 시각. AT는 30분, RT는 일주일로 유효기간을 다르게 뒀다 — AT는 짧고 자주 갱신, RT는 길게 유지하는 일반적인 전략이다.</li>
<li><code>signWith(getSecretKey())</code> — 처음엔 이 줄이 빠져있어서 서명 없는 토큰이 나갔었는데, 나중에 추가됐다. 서명이 없으면 누구나 비슷한 형태의 토큰을 위조할 수 있어서, <code>.env</code>에 넣어둔 <code>JWT_SECRET_KEY</code>로 서명하는 이 한 줄이 핵심이다.</li>
<li><code>getUserEmailFromAT(String at)</code>이라는 메서드 자리도 만들어뒀는데, 아직 <code>return null</code>인 미완성 상태로 &quot;추후 개발&quot;이라는 주석과 함께 남아있다.</li>
</ul>
<hr />
<h1 id="2-jwtfilter--요청을-가로채서-토큰을-검증하기">2. <code>JwtFilter</code> — 요청을 가로채서 토큰을 검증하기</h1>
<p>로그인 이후, 모든 요청은 컨트롤러로 바로 가지 않고 이 필터를 먼저 거친다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/hssondev/post/1d2924b7-3796-48ca-a680-7342c70b022c/image.png" /></p>
<ol>
<li><strong>React 요청</strong> — <code>localStorage</code>에서 토큰을 꺼내 헤더에 담아 보낸다.</li>
<li><strong><code>JwtFilter</code>가 먼저 가로챔</strong> — 컨트롤러보다 앞자리에 있는 &quot;문지기&quot; 역할.</li>
<li><strong>화이트리스트 확인</strong> — 로그인·Swagger 같은 경로는 검사를 건너뛴다.</li>
<li><strong>토큰 서명·만료 검증</strong> — 화이트리스트가 아니면 헤더 존재 여부와 <code>&quot;Bearer &quot;</code> 형식을 먼저 확인하고, 그다음 서명·만료를 검사한다.</li>
<li><strong>갈림길</strong> — 통과하면 <code>chain.doFilter()</code>로 컨트롤러에 전달되고, 실패하면 401만 응답하고 요청이 끝난다(컨트롤러는 이 요청이 왔다는 것 자체를 모른다).</li>
</ol>
<p>필기에 &quot;filter?&quot;라고만 적혀있던 게, 오늘 이렇게 완성됐다.</p>
<pre><code class="language-java">@Component
public class JwtFilter implements Filter {

    @Value(&quot;${jwt.secret}&quot;)
    private String secret;
    private Key key;

    private static final List&lt;String&gt; WHITE_LIST = List.of(
        &quot;/users/**&quot;, &quot;/swagger-ui/**&quot;, &quot;/v3/api-docs/**&quot;
    );

    @PostConstruct
    private void init() {
        this.key = Keys.hmacShaKeyFor(secret.getBytes(StandardCharsets.UTF_8));
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {

        HttpServletRequest req = (HttpServletRequest) request;
        HttpServletResponse res = (HttpServletResponse) response;
        String endPoint = req.getRequestURI();

        if (&quot;OPTIONS&quot;.equalsIgnoreCase(req.getMethod())) {
            res.setStatus(HttpServletResponse.SC_OK);
            res.setHeader(&quot;Access-Control-Allow-Origin&quot;, &quot;http://localhost:3000&quot;);
            res.setHeader(&quot;Access-Control-Allow-Methods&quot;, &quot;GET, POST, PUT,PATCH, DELETE, OPTIONS&quot;);
            res.setHeader(&quot;Access-Control-Allow-Headers&quot;, &quot;Authorization, Content-Type, Refresh-token&quot;);
            res.setHeader(&quot;Access-Control-Allow-Credentials&quot;, &quot;true&quot;);
            chain.doFilter(request, response);
            return;
        }

        if (isPath(endPoint)) {
            chain.doFilter(request, response);
            return;
        }

        String header = req.getHeader(&quot;Authorization&quot;);
        if (header == null || !header.startsWith(&quot;Bearer &quot;)) {
            res.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
            return;
        }

        String token = header.substring(7);
        try {
            Jwts.parserBuilder().setSigningKey(key).build().parseClaimsJws(token);
            chain.doFilter(request, response);
        } catch (Exception e) {
            e.printStackTrace();
            return;
        }
    }
}</code></pre>
<ul>
<li><strong>① preflight 먼저 처리</strong> → <strong>② 화이트리스트 확인</strong> → <strong>③ 토큰 존재 확인</strong> → <strong>④ 토큰 검증</strong> 순서로 4단계가 이어진다.</li>
<li><code>@Component</code>가 빠지면 이 필터는 클래스로만 존재할 뿐 Spring이 인식하지 못해서, 아무리 로직을 채워도 실제로 요청을 가로채지 못한다. 로직을 다 채우고 나서야 이 어노테이션이 붙었다.</li>
<li><code>Jwts.parserBuilder()...parseClaimsJws(token)</code> 한 줄이 서명 위조 여부와 만료 여부를 전부 검증한다. 실패하면 예외가 나서 <code>chain.doFilter()</code>를 안 부르니, 요청이 컨트롤러 근처에도 못 간다.</li>
</ul>
<hr />
<h1 id="3-authorization-헤더-형식-통일--발급-전송-검증이-만나는-지점">3. Authorization 헤더 형식 통일 — 발급, 전송, 검증이 만나는 지점</h1>
<p>여기가 오늘 가장 흥미로웠던 부분이다. 인증이라는 기능 하나가 <strong>세 군데</strong>에 걸쳐 있고, 셋이 정확히 같은 약속을 지켜야 동작한다.</p>
<p><strong>백엔드(발급) — <code>UserController</code></strong></p>
<pre><code class="language-java">headers.add(&quot;Authorization&quot;, &quot;Bearer &quot; + (String)(map.get(&quot;at&quot;)));</code></pre>
<p>로그인 응답에 토큰을 실을 때, 처음엔 <code>&quot;Bearer &quot;</code>를 안 붙이고 토큰 값만 보냈다가, <code>JwtFilter</code>가 기대하는 형식에 맞춰 <code>&quot;Bearer &quot;</code> 접두사를 붙이도록 고쳤다.</p>
<p><strong>프론트(전송) — <code>BlogIndexPage.jsx</code>, <code>BlogWritePage.jsx</code></strong></p>
<pre><code class="language-js">const at = localStorage.getItem('at');

await api.get(`/blogs/index`, {
    headers: { Authorization: at ? at : &quot;&quot; }
})</code></pre>
<pre><code class="language-js">await api.post(`/blogs/insert`, {
    title, content, category, email: user
}, {
    headers: { Authorization: at ? at : &quot;&quot; }
});</code></pre>
<p>로그인 시 저장해둔 토큰(<code>localStorage</code>의 <code>at</code>)을, 블로그 목록 조회와 글쓰기 요청 <strong>양쪽 모두</strong>에 헤더로 실어 보내도록 추가했다. <code>at ? at : &quot;&quot;</code>처럼 값이 없을 때는 빈 문자열을 보내서, 로그인 전에도 에러 없이 요청이 나가게 방어해뒀다.</p>
<p><strong>백엔드(검증) — <code>JwtFilter</code></strong></p>
<pre><code class="language-java">if (header == null || !header.startsWith(&quot;Bearer &quot;)) { ... }
String token = header.substring(7);</code></pre>
<p>⚠️ <strong>왜 세 군데가 다 맞아야 하는가</strong>: <code>UserController</code>가 <code>&quot;Bearer &quot;</code>를 안 붙였다면, <code>JwtFilter</code>의 <code>startsWith(&quot;Bearer &quot;)</code> 검사에서 <strong>모든 로그인 사용자의 요청이 401로 튕겨나갔을 것</strong>이다. 반대로 프론트가 헤더 자체를 안 보냈다면 <code>header == null</code>에 걸려 역시 401이 났을 것이다. 토큰을 &quot;만드는 곳&quot;, &quot;옮기는 곳&quot;, &quot;확인하는 곳&quot;이 서로 다른 파일, 심지어 다른 프로젝트(백엔드/프론트)에 있는데도 문자열 형식 하나로 묶여 있다는 걸 오늘 직접 확인했다.</p>
<hr />
<h1 id="4-blogserviceblogrepository--mybatis에서-jpa로-완전히-갈아끼우기">4. <code>BlogService</code>/<code>BlogRepository</code> — MyBatis에서 JPA로 완전히 갈아끼우기</h1>
<h2 id="4-1-필드명부터-통일-id-→-blogid-→-blogid">4-1) 필드명부터 통일: <code>id</code> → <code>blogid</code> → <code>blogId</code></h2>
<p><code>BlogResponseDTO</code>의 필드 이름이 MyBatis 시절 이름(<code>id</code>)에서 시작해, 최종적으로 엔티티의 실제 PK 필드명인 <code>blogId</code>(카멜케이스)로 정리됐다. 이 변경은 <strong>프론트까지 그대로 이어졌다</strong>:</p>
<pre><code class="language-diff">- moveUrl(`/blogs/read/${blog.id}`);
+ moveUrl(`/blogs/read/${blog.blogId}`);</code></pre>
<pre><code class="language-diff">- {!blog.id &amp;&amp; &lt;Spinner/&gt; }
- {blog.id &amp;&amp;
+ {!blog.blogId &amp;&amp; &lt;Spinner/&gt; }
+ {blog.blogId &amp;&amp;</code></pre>
<p>백엔드 응답 객체의 필드 이름이 바뀌면, 그 값을 그대로 쓰던 프론트(<code>BlogItem.jsx</code>, <code>BlogReadPage.jsx</code>)도 함께 고쳐야 한다는 걸 눈으로 확인할 수 있는 부분이다.</p>
<h2 id="4-2-blogrequestdto--작성자를-어떻게-채울-것인가">4-2) <code>BlogRequestDTO</code> — 작성자를 어떻게 채울 것인가</h2>
<p>처음엔 <code>author(null)</code>로 자리만 잡아뒀다가, 실제로 이메일로 조회한 <code>UserEntity</code>를 받아 채우도록 바뀌었다.</p>
<pre><code class="language-java">public BlogEntity toEntity(UserEntity user) {
    return BlogEntity.builder()
        .title(this.title)
        .content(this.content)
        .category(this.category)
        .author(user)   // author는 UserEntity 타입이라 email 문자열이 아니라 진짜 객체가 필요
        .build();
}</code></pre>
<h2 id="4-3-blogservice--여러-번의-수정-끝에-완성">4-3) <code>BlogService</code> — 여러 번의 수정 끝에 완성</h2>
<p><code>BlogService</code>는 한 번에 완성되지 않고, 고쳐나가는 과정에서 몇 가지 실수를 거쳤다.</p>
<ul>
<li><code>new Runtime)</code>처럼 괄호도 안 닫힌 미완성 문장으로 시작 → <code>RuntimeException</code>을 쓰려다 존재하지 않는 <code>RuntimeEnvironment</code>라는 이름으로 잘못 적음 → 마지막에 <code>RuntimeException</code>으로 바로잡음.</li>
<li><code>BlogRepository.findByComments(id)</code>처럼 <strong>클래스 이름</strong>(대문자로 시작)을 인스턴스 필드(<code>blogRepository</code>) 대신 그대로 쓴 실수도 있었다 — 나중에 소문자 <code>blogRepository</code>로 정리됐다.</li>
</ul>
<p><strong>최종 완성된 형태</strong></p>
<pre><code class="language-java">@Service
@RequiredArgsConstructor
public class BlogService {

    private final UserRepository userRepository;
    private final BlogRepository blogRepository;

    @Transactional(readOnly = true)
    public List&lt;BlogResponseDTO&gt; list() {
        return blogRepository.findAll()
                    .stream()
                    .map(BlogResponseDTO::fromEntity)
                    .toList();
    }

    public BlogResponseDTO insert(BlogRequestDTO request) {
        return userRepository.findById(request.getEmail())
                    .map(user -&gt; {
                        BlogEntity blog = blogRepository.save(request.toEntity(user));
                        return BlogResponseDTO.fromEntity(blog);
                    })
                    .orElseThrow(() -&gt; new RuntimeException(&quot;Blog Insert Fail!!&quot;));
    }

    @Transactional(readOnly = true)
    public BlogResponseDTO read(Integer id) {
        return blogRepository.findByComments(id)
                    .map(BlogResponseDTO::fromEntity)
                    .orElseThrow(() -&gt; new RuntimeException(id + &quot; BLOG NOT FOUND&quot;));
    }
}</code></pre>
<p><code>list()</code>, <code>insert()</code>, <code>read()</code> 전부 MyBatis Mapper(<code>blogMapper</code>) 대신 JPA <code>Repository</code>를 쓰는 걸로 완전히 바뀌었다.</p>
<h2 id="4-4-blogrepository의-jpql--파라미터와-필드명-두-가지를-고쳐서-완성">4-4) <code>BlogRepository</code>의 JPQL — 파라미터와 필드명, 두 가지를 고쳐서 완성</h2>
<pre><code class="language-java">@Query(&quot;&quot;&quot;
    SELECT      b
    FROM        BlogEntity b
    LEFT JOIN   FETCH   b.comments
    WHERE       b.blogId = :blogId
&quot;&quot;&quot;)
public Optional&lt;BlogEntity&gt; findByComments(@Param(&quot;blogId&quot;) Integer blogId);</code></pre>
<p>이 최종 형태가 나오기까지 두 가지 문제를 거쳤다:</p>
<ol>
<li><strong>파라미터 방식</strong>: 처음엔 <code>@Param(&quot;blogId&quot;)</code>(이름 기반)를 선언해놓고 쿼리 안에서는 <code>?</code>(위치 기반)를 써서 서로 안 맞았다 → <code>:blogId</code>로 통일.</li>
<li><strong>필드명 표기</strong>: <code>b.blog_id</code>처럼 DB 컬럼명 스타일(스네이크케이스)을 썼다가, JPQL은 엔티티의 <strong>자바 필드명</strong>을 써야 해서 <code>b.blogId</code>(4-1에서 정리한 그 이름)로 고쳤다.</li>
</ol>
<p><code>LEFT JOIN FETCH b.comments</code>가 핵심인데, 이걸 빼먹으면 댓글은 LAZY 상태로 남아있다가 세션이 닫힌 뒤 접근할 때 <code>LazyInitializationException</code>이 났을 것이다 — 파트5 필기에 남겨둔 Hibernate SQL 로그가 바로 이 쿼리가 실행되면서 찍힌 결과였다.</p>
<hr />
<h1 id="5-commentservice--mybatis-잔재를-걷어내고-dirty-checking으로-완성">5. <code>CommentService</code> — MyBatis 잔재를 걷어내고 Dirty Checking으로 완성</h1>
<p><code>CommentService</code>는 처음엔 예전 MyBatis 버전의 <code>commentMapper</code> 호출을 그대로 복사해온 상태였다(<code>commentMapper</code>라는 필드 자체가 없어서 컴파일도 안 됐다). 이걸 실제 <code>CommentRepository</code>/<code>BlogRepository</code>를 쓰는 코드로 하나씩 교체했다.</p>
<pre><code class="language-java">@Service
@RequiredArgsConstructor
@Transactional
public class CommentService {

    private final CommentRepository commentRepository;
    private final BlogRepository blogRepository;

    public CommentResoponseDTO insert(CommentRequestDTO request) {
        BlogEntity blog = blogRepository.findById(request.getBlogId())
                        .orElseThrow(() -&gt; new RuntimeException(request.getBlogId() + &quot; Blog Not Found!&quot;));
        CommentEntity entity = request.toEntity(blog);
        CommentEntity result = commentRepository.save(entity);
        return CommentResoponseDTO.fromEntity(result);
    }

    public int delete(Integer commentId) {
        commentRepository.deleteById(commentId);
        return 1;
    }

    /*
    jpa update 주의사항
    - DML : transaction(commit, rollback)
    - Dirty Checking(변경감지)
    findById() - 영속성관리로 진입
    setxxxxx() - 필드 수정(Dirty Checking) Not commit
    현재상태 vs 스냅샷 상태 비교하고 다름을 확인하면 update실행 -commit
    */
    @Transactional
    public int update(Map&lt;String, Object&gt; map) {
        CommentEntity entity = commentRepository
            .findById((Integer)(map.get(&quot;id&quot;)))
            .orElseThrow(() -&gt; new RuntimeException((Integer)(map.get(&quot;id&quot;)) + &quot; Comment Not Found!&quot;));
        entity.updateComment((String) map.get(&quot;comment&quot;));
        return 1;
    }
}</code></pre>
<p><strong>필기에 남긴 질문(&quot;dirty checking?? @Transactional을 붙여야 하는 이유?&quot;)에 대한 답이 이 <code>update()</code> 메서드 자체다.</strong></p>
<ol>
<li><code>findById(...)</code>로 엔티티를 조회하는 순간, 그 엔티티는 영속성 컨텍스트에 들어가면서 <strong>현재 상태의 스냅샷</strong>을 함께 저장해둔다.</li>
<li><code>entity.updateComment(...)</code>로 필드 값을 바꾼다 — <code>save()</code>도, <code>UPDATE</code> SQL도 어디에도 없다.</li>
<li><code>@Transactional</code> 메서드가 끝나는 시점에, JPA가 <strong>&quot;지금 상태&quot;와 &quot;스냅샷&quot;을 비교</strong>해서 다른 점이 있으면 자동으로 <code>UPDATE</code>를 실행한다. <code>@Transactional</code>이 없으면 이 비교·커밋 과정 자체가 없어서 수정이 반영되지 않는다.</li>
<li><code>CommentEntity</code>에 <code>updateComment(String comment)</code>라는 의미 있는 이름의 메서드를 새로 만든 것도, setter를 직접 노출하지 않고 &quot;댓글을 수정한다&quot;는 의도를 코드에 담기 위해서다.</li>
</ol>
<hr />
<h1 id="✨-새롭게-알게-된-점">✨ 새롭게 알게 된 점</h1>
<ul>
<li>토큰을 만드는 쪽(백엔드 발급), 옮기는 쪽(프론트 전송), 확인하는 쪽(백엔드 검증) 세 군데가 문자열 형식 하나(&quot;Bearer &quot; 접두사)로 묶여 있어서, 어느 한 곳만 달라도 전체가 깨진다.</li>
<li><code>@Component</code>가 빠지면 필터 로직을 다 채워도 Spring이 인식하지 못해 동작하지 않는다.</li>
<li>백엔드 응답 객체의 필드 이름이 바뀌면 프론트의 해당 값을 읽는 코드도 함께 고쳐야 한다 (<code>blog.id</code> → <code>blog.blogId</code>).</li>
<li>JPQL은 파라미터 방식(이름/위치)을 섞으면 안 되고, 필드는 DB 컬럼명이 아니라 엔티티의 자바 필드명을 써야 한다.</li>
<li><code>updateComment()</code>처럼 의미 있는 메서드로 필드를 바꾸고 <code>@Transactional</code> 안에 있으면, <code>save()</code> 없이도 Dirty Checking으로 자동 반영된다.</li>
</ul>
<hr />
<h1 id="🤔-헷갈렸던-점">🤔 헷갈렸던 점</h1>
<ul>
<li><strong><code>getUserEmailFromAT(String at)</code>을 왜 아직 구현 안 했는지</strong>: 토큰 안에 이메일이 들어있는데(<code>setSubject(email)</code>), 굳이 이 메서드가 왜 필요한지는 다음에 실제로 쓰이는 곳을 보면 이해가 될 것 같다.</li>
<li><strong><code>LEFT JOIN FETCH</code>를 안 쓰면 정확히 언제 문제가 되는지</strong>: 이번엔 처음부터 <code>FETCH</code>를 붙여서 문제를 안 겪었는데, 안 붙였을 때 실제로 <code>LazyInitializationException</code>이 나는 상황을 아직 직접 재현은 못 해봤다.</li>
<li><strong>Dirty Checking이 여러 필드를 동시에 바꿀 때도 잘 동작하는지</strong>: 원리는 이해했지만 아직 실험은 안 해봤다.</li>
</ul>
<hr />
<h1 id="다음에-할-일">다음에 할 일</h1>
<ul>
<li><input disabled="" type="checkbox" /> <code>getUserEmailFromAT</code>을 실제로 구현해서 토큰에서 이메일을 꺼내 쓰는 곳까지 연결해보기</li>
<li><input disabled="" type="checkbox" /> <code>LEFT JOIN FETCH</code>를 빼고 실제로 <code>LazyInitializationException</code>을 재현해본 뒤 다시 붙여서 비교해보기</li>
<li><input disabled="" type="checkbox" /> <code>BlogResponseDTO.fromEntitywithComments()</code>(만들어뒀지만 아직 서비스에서 호출 안 됨)를 실제로 연결해보기</li>
</ul>
<hr />
<h1 id="💡-오늘의-회고">💡 오늘의 회고</h1>
<p>오늘은 &quot;기능 하나가 여러 파일, 심지어 프론트/백엔드 양쪽에 걸쳐 완성된다&quot;는 걸 제일 크게 느낀 날이었다. <code>Authorization</code> 헤더의 <code>&quot;Bearer &quot;</code> 형식 하나가 세 군데(발급/전송/검증)에서 다 맞아야 동작한다는 걸 직접 코드로 확인하면서, 인증이라는 기능이 왜 &quot;한 곳만 잘 짜면 끝&quot;이 아닌지 이해했다.</p>
<p><code>BlogService</code>를 완성해가는 과정에서 만난 실수들(미완성 문장, 존재하지 않는 클래스 이름, 클래스명과 인스턴스명 혼동)도 결과 코드만 봤으면 몰랐을 것들이었다. Dirty Checking으로 <code>save()</code> 없이 필드만 바꿔도 DB가 갱신되는 걸 보면서, JPA가 편해지는 만큼 &quot;지금 이 엔티티가 영속성 컨텍스트 안에 있는가&quot;를 계속 의식해야 한다는 것도 다시 느꼈다.</p>