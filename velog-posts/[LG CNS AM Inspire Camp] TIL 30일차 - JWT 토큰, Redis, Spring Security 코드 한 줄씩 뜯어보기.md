<blockquote>
<p><strong>한 줄 요약</strong></p>
<p>코드를 &quot;왜 이렇게 썼는지&quot;만 알면 반은 이해한 거다. 토큰을 열어보는 것도, Redis에 값을 넣는 것도, Security가 로그인 정보를 기억해두는 것도 결국 &quot;누가 요청했는지를 어떻게 안전하게 확인하고 전달할 것인가&quot;라는 한 가지 문제를 풀기 위한 도구들이다.</p>
</blockquote>
<hr />
<h1 id="📚-오늘-학습한-커리큘럼">📚 오늘 학습한 커리큘럼</h1>
<pre><code>요청이 서버에 도착 (Authorization 헤더에 토큰이 들어있음)
        ↓
Filter(검문소)가 토큰을 열어서 &quot;누구인지&quot; 확인
        ↓
확인된 사용자 정보를 SecurityContextHolder(메모장)에 적어둠
        ↓
Controller/Service는 그 메모장만 보고 &quot;누가 요청했는지&quot; 앎
        ↓
로그인 시 발급한 Refresh Token은 Redis(물품보관함)에 유통기한과 함께 저장
        ↓
로그아웃하면 그 물품보관함에서 토큰을 꺼내서 버림</code></pre><h1 id="📌-오늘의-학습-키워드">📌 오늘의 학습 키워드</h1>
<ul>
<li>JWT 토큰 구조 (header.payload.signature)</li>
<li>Filter / OncePerRequestFilter</li>
<li>Claims (토큰 안에 든 정보)</li>
<li>SecurityContextHolder</li>
<li>RedisTemplate / TTL</li>
<li>BCrypt (비밀번호 해싱)</li>
</ul>
<hr />
<h1 id="0-개념-정리-키워드-사전">0. 개념 정리 (키워드 사전)</h1>
<blockquote>
<p><strong>한 줄 요약</strong></p>
<p>토큰은 &quot;신분증&quot;, Filter는 &quot;출입구 검문소&quot;, SecurityContextHolder는 &quot;검문소를 통과한 사람 이름을 적어두는 방문자 명단&quot;, Redis는 &quot;물품 보관함&quot;이라고 생각하면 오늘 배운 코드가 전부 이 4개 비유 안에서 움직인다.</p>
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
<td><strong>JWT (JSON Web Token)</strong></td>
<td>&quot;이 사람은 누구고 언제까지 유효한지&quot;를 암호화해서 담아둔 문자열 신분증</td>
<td><code>eyJhbGciOi...</code> 처럼 생긴 긴 문자열</td>
</tr>
<tr>
<td><strong>Claims</strong></td>
<td>JWT 신분증 안에 적힌 내용(이메일, 권한, 만료시간 등)</td>
<td><code>claims.getSubject()</code>로 이메일 꺼냄</td>
</tr>
<tr>
<td><strong>Filter</strong></td>
<td>요청이 컨트롤러에 도착하기 전에 먼저 거쳐가는 검문소</td>
<td>여기서 신분증(토큰) 검사</td>
</tr>
<tr>
<td><strong>OncePerRequestFilter</strong></td>
<td>검문소를 요청 1번당 딱 1번만 통과시키게 보장하는 스프링 기본 클래스</td>
<td>중복 검사 방지</td>
</tr>
<tr>
<td><strong>SecurityContextHolder</strong></td>
<td>검문소를 통과한 사람의 이름을 적어두는 메모장(방문자 명단)</td>
<td>이후 어디서든 &quot;지금 누구야?&quot; 조회 가능</td>
</tr>
<tr>
<td><strong>RedisTemplate</strong></td>
<td>Redis 물품보관함에 물건을 넣고 빼는 도구</td>
<td><code>opsForValue().set(...)</code></td>
</tr>
<tr>
<td><strong>TTL</strong></td>
<td>보관함에 넣은 물건의 유통기한</td>
<td>7일 지나면 자동으로 사라짐</td>
</tr>
<tr>
<td><strong>BCrypt</strong></td>
<td>비밀번호를 원래대로 되돌릴 수 없게 섞어버리는 암호화 방식</td>
<td><code>encode()</code>로 섞고 <code>matches()</code>로 비교</td>
</tr>
</tbody></table>
<h3 id="코드로-다시-보기">코드로 다시 보기</h3>
<pre><code class="language-java">// 방문자가 왔을 때(=요청이 들어왔을 때) 검문소가 하는 일을 한 줄씩 압축하면:
String token = header.substring(7);                 // 1. 신분증(토큰) 꺼내기
Claims claims = ...parseClaimsJws(token).getBody();  // 2. 신분증 내용 확인 (위조 검사 포함)
String email = claims.getSubject();                  // 3. 이름표(이메일) 확인
SecurityContextHolder.getContext()
        .setAuthentication(인증객체);                 // 4. 방문자 명단에 이름 적기</code></pre>
<hr />
<h1 id="1-요청-하나가-서버에-들어오면-무슨-일이-일어날까">1. 요청 하나가 서버에 들어오면 무슨 일이 일어날까</h1>
<p>먼저 큰 그림부터 그려보면 이렇다.</p>
<pre><code>[브라우저]                    [우리 서버]
   │  Authorization: Bearer eyJhbG...
   ▼
┌─────────────────────────────────────────┐
│ 1) JwtAuthentificationFilter (검문소)     │
│    - 헤더에서 토큰 꺼내기                  │
│    - 토큰이 진짜인지 검증                  │
│    - &quot;이 사람은 OO다&quot; 명단에 기록          │
└─────────────────────────────────────────┘
                │ 통과
                ▼
┌─────────────────────────────────────────┐
│ 2) Controller / Service (실제 업무)       │
│    - 명단만 보고 &quot;지금 누구 요청이지?&quot; 확인 │
└─────────────────────────────────────────┘</code></pre><p>브라우저가 요청을 보낼 때마다 헤더(요청에 딸려오는 부가정보 칸)에 <code>Authorization: Bearer eyJhbG...</code> 같은 값을 함께 보낸다. 여기서 <code>eyJhbG...</code>이 JWT 토큰, 즉 &quot;이 사람의 신분증&quot;이다. 이 신분증이 컨트롤러(실제 회원가입/블로그 작성 같은 로직)에 도착하기 전에, Filter라는 검문소를 먼저 통과해야 한다.</p>
<pre><code class="language-java">@Component
public class JwtAuthentificationFilter extends OncePerRequestFilter {</code></pre>
<ul>
<li><code>@Component</code>: &quot;이 클래스를 스프링이 관리하는 부품으로 등록해줘&quot;라는 표시다. 스프링은 이렇게 등록된 부품들을 알아서 만들고 필요한 곳에 꽂아준다.</li>
<li><code>extends OncePerRequestFilter</code>: 이 클래스가 &quot;검문소&quot; 역할을 하겠다는 선언이다. <code>OncePerRequestFilter</code>를 상속받으면, 스프링이 &quot;요청 하나당 이 검문소를 딱 1번만 통과시켜줄게&quot;라고 보장해준다. (상속을 안 받고 직접 만들면 같은 요청이 검문소를 여러 번 통과해버리는 실수가 생길 수 있다.)</li>
</ul>
<pre><code class="language-java">String header = request.getHeader(&quot;Authorization&quot;);
if (header == null || !header.startsWith(&quot;Bearer &quot;)) {
    filterChain.doFilter(request, response);
    return;
}</code></pre>
<ul>
<li><code>request.getHeader(&quot;Authorization&quot;)</code>: 요청에 딸려온 여러 정보 중에서 &quot;Authorization&quot;이라는 이름표가 붙은 값을 꺼낸다. 편지봉투에서 &quot;받는 사람&quot; 칸만 골라 보는 것과 비슷하다.</li>
<li><code>header == null</code>: 애초에 그 칸이 비어있는 경우 (신분증을 아예 안 가져온 사람).</li>
<li><code>!header.startsWith(&quot;Bearer &quot;)</code>: 값이 있어도 <code>&quot;Bearer &quot;</code>로 시작하지 않으면 우리가 기대하는 형식이 아니라는 뜻이다.</li>
<li>둘 중 하나라도 해당하면 <code>filterChain.doFilter(request, response)</code>로 &quot;일단 다음 검문소로 넘겨&quot;라고 통과시키고, <code>return</code>으로 이 함수를 끝낸다. <strong>여기서 막지 않는 이유</strong>: 신분증이 없다고 무조건 차단하면, 로그인/회원가입처럼 애초에 토큰이 필요 없는 요청까지 막혀버린다. 그래서 &quot;진짜로 막을지 말지&quot;는 뒤에 나올 <code>SecurityConfig</code>가 최종 결정한다.</li>
</ul>
<pre><code class="language-java">String token = header.substring(7);</code></pre>
<p><code>&quot;Bearer &quot;</code>는 <code>B-e-a-r-e-r-공백</code>으로 정확히 7글자다. <code>substring(7)</code>은 &quot;앞의 7글자는 잘라내고 그 뒤부터 끝까지&quot;를 의미하므로, 결과적으로 순수한 토큰 문자열만 남는다.</p>
<pre><code>&quot;Bearer eyJhbGciOiJIUzI1NiJ9...&quot;
        └────────7글자────────┘
                 자르고 나면 → &quot;eyJhbGciOiJIUzI1NiJ9...&quot; 만 남음</code></pre><pre><code class="language-java">Claims claims = Jwts.parserBuilder()
                     .setSigningKey(key)
                     .build()
                     .parseClaimsJws(token)
                     .getBody();
String email = claims.getSubject();
String role = claims.get(&quot;role&quot;, String.class);</code></pre>
<p>JWT 토큰은 사실 점(<code>.</code>)으로 구분된 3부분으로 이루어져 있다.</p>
<pre><code>eyJhbGciOi...  .  eyJzdWIiOi...  .  SflKxwRJ...
   (header)        (payload)         (signature)
   &quot;어떤 방식으로     &quot;누구고, 언제까지     &quot;위조 안 됐다는
    암호화했나&quot;        유효한지&quot; 내용        증명 서명&quot;</code></pre><ul>
<li><code>Jwts.parserBuilder().setSigningKey(key)...</code>: &quot;이 서명(signature)이 우리 서버가 발급할 때 쓴 열쇠(key)로 만든 게 맞는지&quot; 확인하는 과정이다. 만약 누군가 토큰 내용을 몰래 바꿨다면 서명이 안 맞아서 여기서 에러가 난다.</li>
<li><code>.parseClaimsJws(token).getBody()</code>: 검증을 통과하면 payload(내용물)를 꺼내주는데, 이걸 <code>Claims</code>라고 부른다.</li>
<li><code>claims.getSubject()</code>: 토큰을 처음 만들 때 &quot;이 토큰의 주인은 이 이메일이야&quot;라고 박아둔 값을 꺼낸다.</li>
<li><code>claims.get(&quot;role&quot;, String.class)</code>: 마찬가지로 토큰 안에 같이 넣어둔 &quot;권한(role)&quot; 값을 꺼낸다.</li>
</ul>
<pre><code class="language-java">UsernamePasswordAuthenticationToken authenticationToken =
    new UsernamePasswordAuthenticationToken(
        email,
        null,
        role != null
            ? List.of(new SimpleGrantedAuthority(&quot;ROLE_&quot; + role))
            : List.of()
    );
authenticationToken.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
SecurityContextHolder.getContext().setAuthentication(authenticationToken);
filterChain.doFilter(request, response);</code></pre>
<p>여기가 이 필터의 핵심이다. Spring Security는 &quot;누가 인증됐다&quot;는 사실을 아무 방식으로나 저장하면 못 알아듣고, 자기가 정해놓은 전용 상자(<code>UsernamePasswordAuthenticationToken</code>)에 담아야만 이해한다.</p>
<ul>
<li><code>new UsernamePasswordAuthenticationToken(email, null, 권한목록)</code>: &quot;이 사람은 email이고, 비밀번호는 이미 확인 끝났으니 필요 없고(null), 권한은 이거다&quot;라는 상자를 만드는 것. 이름 때문에 헷갈릴 수 있는데, 로그인 폼(아이디/비밀번호)이 아니어도 인증 정보를 담는 표준 상자로 흔히 재사용된다.</li>
<li><code>SimpleGrantedAuthority(&quot;ROLE_&quot; + role)</code>: Spring Security는 권한 이름 앞에 관례적으로 <code>&quot;ROLE_&quot;</code>을 붙인다. role이 <code>&quot;ADMIN&quot;</code>이면 <code>&quot;ROLE_ADMIN&quot;</code>이 되는 식이다.</li>
<li><code>authenticationToken.setDetails(...)</code>: 이 요청이 어디서(IP 등) 왔는지 같은 부가정보를 상자에 덧붙이는 것. 필수는 아니지만 관례적으로 붙여준다.</li>
<li><code>SecurityContextHolder.getContext().setAuthentication(authenticationToken)</code>: 드디어 방문자 명단에 이름을 적는 순간이다. 이 한 줄 덕분에, 이 요청을 처리하는 동안에는 어디서든(컨트롤러든 서비스든) &quot;지금 요청 보낸 사람이 누구야?&quot;라고 물어볼 수 있게 된다.</li>
<li><code>filterChain.doFilter(request, response)</code>: &quot;확인 끝났으니 다음 단계(컨트롤러)로 넘어가&quot;라는 뜻.</li>
</ul>
<pre><code class="language-java">} catch (Exception e) {
    response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
}</code></pre>
<p>토큰이 위조됐거나 만료됐으면 위의 검증 과정에서 예외(에러)가 터진다. 그러면 그냥 &quot;401 인증 실패&quot; 상태코드만 응답에 박아두고 끝낸다. (여기서 <code>filterChain.doFilter</code>를 호출하지 않으므로 컨트롤러까지 요청이 도달하지 못하고 여기서 막힌다.)</p>
<hr />
<h1 id="2-securityconfig--어떤-문은-검문-없이-통과시킬지-정하는-설계도">2. SecurityConfig — &quot;어떤 문은 검문 없이 통과시킬지&quot; 정하는 설계도</h1>
<pre><code class="language-java">@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http
        .cors(Customizer.withDefaults())
        .csrf(csrf -&gt; csrf.disable())
        .authorizeHttpRequests(auth -&gt; auth
            .requestMatchers(&quot;/users/**&quot;, &quot;/swagger-ui/**&quot;, &quot;/v3/api-docs/**&quot;).permitAll()
            .requestMatchers(HttpMethod.OPTIONS, &quot;/**&quot;).permitAll()
            .requestMatchers(&quot;/admin/**&quot;).hasRole(&quot;ADMIN&quot;)
            .anyRequest().authenticated())
        .sessionManagement(session -&gt; session.sessionCreationPolicy(SessionCreationPolicy.STATELESS));

    http.addFilterBefore(jwtAuthenticationFilter, UsernamePasswordAuthenticationFilter.class);

    return http.build();
}</code></pre>
<p>한 줄씩 풀면:</p>
<ul>
<li><code>.cors(Customizer.withDefaults())</code>: &quot;다른 주소(우리 프론트는 <code>localhost:3000</code>)에서 온 요청도 허용해줄게&quot;라는 CORS 설정을 켠다. CORS는 브라우저가 &quot;이 요청, 다른 출처에서 왔는데 서버가 허락한 거 맞아?&quot;를 확인하는 보안 장치다.</li>
<li><code>.csrf(csrf -&gt; csrf.disable())</code>: CSRF는 원래 &quot;로그인 세션을 훔쳐서 사용자 몰래 요청을 대신 보내는 공격&quot;을 막는 기능인데, 이건 브라우저가 쿠키/세션을 자동으로 실어 보내는 방식에서 문제가 된다. 우리는 세션이 아니라 매번 토큰을 직접 헤더에 실어 보내는 방식(JWT)이라 이 공격 자체가 성립하지 않으므로 꺼둔다.</li>
<li><code>.authorizeHttpRequests(...)</code>: &quot;어떤 주소는 검문 없이 통과, 어떤 주소는 로그인해야 통과&quot;를 정하는 규칙표다.<ul>
<li><code>requestMatchers(&quot;/users/**&quot;, ...).permitAll()</code>: 회원가입/로그인 같은 <code>/users/</code>로 시작하는 주소, 그리고 API 문서 주소는 로그인 안 해도 통과.</li>
<li><code>requestMatchers(HttpMethod.OPTIONS, &quot;/**&quot;).permitAll()</code>: 브라우저가 본 요청 전에 미리 물어보는 사전 요청(OPTIONS)은 전부 통과.</li>
<li><code>requestMatchers(&quot;/admin/**&quot;).hasRole(&quot;ADMIN&quot;)</code>: <code>/admin/</code>으로 시작하는 주소는 role이 ADMIN인 사람만 통과.</li>
<li><code>.anyRequest().authenticated()</code>: 위에서 정하지 않은 나머지 모든 주소는 &quot;로그인(인증)된 사람만&quot; 통과.</li>
</ul>
</li>
<li><code>.sessionManagement(... STATELESS)</code>: &quot;서버는 세션(로그인 상태를 서버 메모리에 저장하는 방식)을 아예 안 만든다&quot;는 뜻. 매 요청마다 토큰만으로 판단하겠다는 선언이다.</li>
<li><code>http.addFilterBefore(jwtAuthenticationFilter, UsernamePasswordAuthenticationFilter.class)</code>: &quot;내가 만든 검문소(jwtAuthenticationFilter)를, Security가 원래 갖고 있는 기본 로그인 검문소보다 앞자리에 세워줘&quot;라는 뜻이다. 검문소들도 순서가 있는데, 우리 토큰 검문소가 먼저 실행되어야 한다.</li>
</ul>
<hr />
<h1 id="3-redis로-로그인-상태refresh-token-관리하기">3. Redis로 로그인 상태(Refresh Token) 관리하기</h1>
<pre><code>saveToken(&quot;cns@gmail.com&quot;, &quot;eyJhbG...&quot;)

┌───────────────── Redis (물품보관함) ─────────────────┐
│  이름표: &quot;RT:cns@gmail.com&quot;                          │
│  내용물: &quot;eyJhbG...&quot;                                  │
│  유통기한: 7일 뒤 자동 폐기                             │
└──────────────────────────────────────────────────────┘</code></pre><pre><code class="language-java">public void saveToken(String email, String rt) {
    redisTemplate.opsForValue().set(&quot;RT:&quot; + email, rt, RT_TTL, TimeUnit.SECONDS);
}</code></pre>
<ul>
<li><code>opsForValue()</code>: Redis는 여러 종류의 저장 방식(단순 값, 리스트, 집합 등)을 지원하는데, 그중 &quot;이름표 하나에 값 하나&quot;짜리 가장 단순한 방식을 쓰겠다는 뜻이다.</li>
<li><code>.set(키, 값, 시간, 단위)</code>: &quot;이 이름표로 이 값을 넣되, 이만큼 시간이 지나면 자동으로 삭제해줘&quot;라는 명령이다.</li>
<li>왜 이메일 앞에 <code>&quot;RT:&quot;</code>를 붙일까? Redis 창고 안에 수많은 물건(다른 기능에서도 Redis를 쓸 수 있으니까)이 섞여 있을 때, &quot;이건 Refresh Token 종류다&quot;라고 구분하기 위한 이름 규칙(prefix)이다.</li>
</ul>
<pre><code class="language-java">public void deleteToken(String email) {
    redisTemplate.delete(&quot;RT:&quot; + email);
}</code></pre>
<p>로그아웃할 때 이 함수를 호출한다. 이름표를 보고 그 물건을 통째로 창고에서 꺼내 버리는 것과 같다. 이렇게 하면 이 사용자의 RT가 사라지므로, 나중에 Access Token이 만료돼도 더 이상 새 토큰을 재발급받을 수 없게 된다 — 이게 바로 &quot;로그아웃&quot;의 실제 의미다.</p>
<hr />
<h1 id="4-bcrypt--비밀번호를-안전하게-저장하기">4. BCrypt — 비밀번호를 안전하게 저장하기</h1>
<pre><code class="language-java">@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}</code></pre>
<p>비밀번호를 그대로(평문으로) 저장하면, 만약 DB가 해킹당했을 때 모든 사용자의 비밀번호가 그대로 노출된다. BCrypt는 비밀번호를 &quot;원래대로 되돌릴 수 없는 방식&quot;으로 섞어버리는 암호화 방법이다.</p>
<pre><code>&quot;1234&quot;  --encode()--&gt;  &quot;$2a$10$nAK0CgeYDF...&quot;   (저장은 이걸로)</code></pre><p>한 번 섞이면 다시 <code>&quot;1234&quot;</code>로 되돌릴 수 없다. 그럼 로그인할 때는 어떻게 비교할까?</p>
<pre><code class="language-java">if (!passwordEncoder.matches(request.getPassword(), entity.getPassword())) {
    throw new RuntimeException(&quot;Password Not Matches!&quot;);
}</code></pre>
<p><code>matches(사용자가입력한평문, DB에저장된해시)</code>는 &quot;이 평문을 같은 방식으로 섞었을 때 이 해시가 나오는가?&quot;를 확인해주는 함수다. 즉 되돌리는 게 아니라, &quot;같은 규칙으로 다시 섞어서 결과가 똑같은지&quot;만 비교하는 방식이다.</p>
<pre><code class="language-java">UserRequestDTO hashingDTO = request.toBuilder()
        .password(passwordEncoder.encode(request.getPassword()))
        .build();</code></pre>
<ul>
<li><code>request.toBuilder()</code>: 기존 <code>request</code> 객체의 모든 값을 그대로 복사해서 &quot;수정 가능한 설계도(Builder)&quot;를 새로 하나 만드는 것.</li>
<li><code>.password(passwordEncoder.encode(...))</code>: 그 설계도에서 password 값만 &quot;암호화된 값&quot;으로 바꿔치기한다.</li>
<li><code>.build()</code>: 바뀐 설계도로 새 객체를 최종적으로 완성한다.</li>
</ul>
<p>이렇게 하는 이유는, <code>UserRequestDTO</code>가 &quot;한 번 만들어지면 내용을 바꿀 수 없는&quot; 불변 객체이기 때문이다. 그래서 직접 값을 수정하는 대신, &quot;이 값만 바꾼 새 복사본&quot;을 만드는 방식을 쓴다.</p>
<hr />
<h1 id="5-방문자-명단을-재활용하기--securitycontextholder">5. 방문자 명단을 재활용하기 — SecurityContextHolder</h1>
<pre><code class="language-java">Authentication auth = SecurityContextHolder.getContext().getAuthentication();
String email = auth.getName();</code></pre>
<p>이 두 줄이 오늘 배운 것 중 가장 &quot;이득&quot;을 크게 본 부분이다. 필터에서 이미 방문자 명단에 이름을 적어뒀기 때문에(2번 섹션 참고), 블로그 작성 로직이나 로그아웃 로직에서 &quot;지금 누가 요청했는지&quot;를 다시 파라미터로 넘겨받을 필요 없이 이 두 줄만 쓰면 바로 알 수 있다.</p>
<pre><code class="language-java">public void signOut() {
    Authentication auth = SecurityContextHolder.getContext().getAuthentication();
    String email = auth.getName();
    redisService.deleteToken(email);
}</code></pre>
<p>로그아웃 함수가 파라미터를 하나도 안 받는 이유가 여기 있다. &quot;누구를 로그아웃시킬지&quot;를 프론트가 알려줄 필요가 없다. 어차피 그 사람의 토큰이 헤더에 실려서 왔고, 필터가 이미 그걸 확인해서 명단에 적어뒀기 때문이다.</p>
<hr />
<h1 id="✨-새롭게-알게-된-점">✨ 새롭게 알게 된 점</h1>
<ul>
<li>JWT 토큰은 &quot;암호화된 하나의 덩어리&quot;가 아니라, <code>header.payload.signature</code> 3부분으로 나뉘어 있고, 우리가 실제로 꺼내 쓰는 이메일·권한 같은 정보는 payload(Claims) 부분에 들어있다는 것.</li>
<li><code>SecurityContextHolder</code>에 한 번 로그인 정보를 적어두면, 이후 코드에서는 &quot;누가 요청했는지&quot;를 매번 파라미터로 넘길 필요가 없어진다는 것 — 이게 Filter를 거치는 진짜 이유였다.</li>
<li><code>toBuilder()</code>는 &quot;불변 객체의 일부 값만 바꾼 새 복사본&quot;을 만드는 패턴이라는 것.</li>
<li>BCrypt는 &quot;원래대로 되돌리는&quot; 암호화가 아니라, &quot;같은 규칙으로 다시 섞었을 때 똑같이 나오는지&quot;만 비교하는 방식이라는 것.</li>
</ul>
<h1 id="🤔-헷갈렸던-점">🤔 헷갈렸던 점</h1>
<ul>
<li><code>UsernamePasswordAuthenticationToken</code>이라는 이름 때문에 &quot;아이디/비밀번호 로그인 전용&quot;인 줄 알았는데, 실제로는 &quot;인증된 사용자 정보를 담는 표준 상자&quot;로 JWT 인증에서도 그대로 재활용된다는 점.</li>
<li>Filter 안에서 요청을 막을지 말지를 직접 결정하는 부분과, <code>SecurityConfig</code>에서 경로별로 통과 규칙을 정하는 부분이 서로 다른 파일에 나뉘어 있어서 처음엔 전체 흐름을 한 번에 그리기 어려웠다.</li>
</ul>
<h1 id="다음에-할-일">다음에 할 일</h1>
<ul>
<li><input disabled="" type="checkbox" /> JWT의 3부분(header.payload.signature) 구조를 실제 토큰 값으로 직접 디코딩해보면서 눈으로 확인해보기</li>
<li><input disabled="" type="checkbox" /> Access Token 만료 → Refresh Token으로 재발급하는 전체 흐름도 그려보기</li>
<li><input disabled="" type="checkbox" /> <code>SecurityContextHolder</code>를 활용하는 다른 예시(예: 내가 쓴 글만 삭제 가능하게 만들기)도 직접 만들어보기</li>
</ul>
<hr />
<h1 id="💡-오늘의-회고">💡 오늘의 회고</h1>
<p>전에는 Redis에 값 넣기, Filter 만들기, Security 설정하기를 각각 따로따로 외우려고 했는데, 오늘 코드를 한 줄씩 뜯어보면서 &quot;결국 다 같은 이야기&quot;라는 걸 알게 됐다. 요청이 들어오면 검문소(Filter)가 신분증(토큰)을 확인하고, 확인된 사람 이름을 명단(SecurityContextHolder)에 적어두고, 그 이름을 나중에 다른 코드들이 재활용하는 것. 그리고 그 사람의 로그인 상태(Refresh Token)를 물품보관함(Redis)에 유통기한과 함께 넣어두는 것. 비유로 정리하고 나니 코드가 왜 그 순서로 쓰였는지가 훨씬 잘 보였다. 다음엔 실제 토큰 값을 직접 디코딩해보면서 이 구조를 눈으로 한 번 더 확인해봐야겠다.</p>