<blockquote>
<p><strong>한 줄 요약</strong></p>
<p>쇼핑몰을 회원·상품·주문 세 개의 마이크로서비스로 쪼개서 Eureka와 API Gateway에 연결하고, User 서비스에 BCrypt 비밀번호 저장과 JWT 로그인을 붙인 뒤, Gateway가 토큰을 검사하는 필터까지 이어서 구성했다.</p>
</blockquote>
<hr />
<h1 id="📚-오늘-학습한-커리큘럼">📚 오늘 학습한 커리큘럼</h1>
<pre><code>E-commerce 애플리케이션 구조 — 서비스 3개 + 공통 구성요소
        ↓
User Service 만들기 — Eureka 등록, 랜덤 포트, H2 + JPA
        ↓
회원가입 — 객체 역할 분리, UUID, BCrypt
        ↓
Spring Security 기본 설정
        ↓
Gateway 라우팅 — 경로 맞추기, Catalog / Order 서비스 추가
        ↓
로그인 — AuthenticationFilter로 JWT 발급
        ↓
Gateway의 AuthorizationHeaderFilter로 토큰 검증</code></pre><h1 id="📌-오늘의-학습-키워드">📌 오늘의 학습 키워드</h1>
<ul>
<li>MSA, Eureka, API Gateway(<code>lb://</code>)</li>
<li>H2, <code>CrudRepository</code></li>
<li><code>RequestUser</code> / <code>UserDto</code> / <code>UserEntity</code> / <code>ResponseUser</code></li>
<li><code>BCryptPasswordEncoder</code>, 해시 vs 암호화</li>
<li>Spring Security, <code>SecurityFilterChain</code></li>
<li>JWT, <code>AuthenticationFilter</code>, <code>AuthorizationHeaderFilter</code></li>
</ul>
<hr />
<h1 id="0-개념-정리-키워드-사전">0. 개념 정리 (키워드 사전)</h1>
<blockquote>
<p><strong>한 줄 요약</strong></p>
<p>하나의 큰 앱을 기능별로 쪼개 각자 독립된 서비스로 띄우고, 손님(Client)은 문지기(Gateway) 한 곳만 알면 되게 만드는 구조다. 그리고 그 문지기가 &quot;토큰 있는 사람만 들여보내는&quot; 역할까지 맡는다.</p>
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
<td><strong>MSA (마이크로서비스)</strong></td>
<td>하나의 앱을 기능별로 쪼개 독립적으로 개발·배포하는 구조</td>
<td>회원 / 상품 / 주문 서비스, 서비스마다 자기 DB를 따로 가짐</td>
</tr>
<tr>
<td><strong>Eureka</strong></td>
<td>서비스가 자기 이름과 위치를 등록해두는 &quot;전화번호부&quot;</td>
<td>서비스 이름으로 찾을 수 있게 해줌</td>
</tr>
<tr>
<td><strong>API Gateway</strong></td>
<td>모든 요청이 가장 먼저 들어오는 단일 진입점, 경로를 보고 알맞은 서비스로 전달</td>
<td><code>lb://USER-SERVICE</code> = Eureka에서 찾아서 부하 분산하며 전달</td>
</tr>
<tr>
<td><strong>H2</strong></td>
<td>자바로 만든 가벼운 오픈소스 DB, 메모리로도 동작</td>
<td>실습용으로 서비스마다 하나씩 사용</td>
</tr>
<tr>
<td><strong>DTO 분리</strong></td>
<td>요청/전달/저장/응답 객체를 역할별로 나눠 쓰는 방식</td>
<td><code>RequestUser</code> → <code>UserDto</code> → <code>UserEntity</code> → <code>ResponseUser</code></td>
</tr>
<tr>
<td><strong>UUID</strong></td>
<td>중복될 가능성이 사실상 없는 랜덤 문자열 ID</td>
<td>회원·주문의 외부 식별자로 사용</td>
</tr>
<tr>
<td><strong>BCrypt</strong></td>
<td>비밀번호를 단방향으로 해싱하는 알고리즘, 매번 랜덤 salt 사용</td>
<td>같은 비밀번호도 저장값이 매번 다름</td>
</tr>
<tr>
<td><strong>해시 vs 암호화</strong></td>
<td>해시는 단방향(복구 불가), 암호화는 키가 있으면 복호화 가능</td>
<td>비밀번호는 원본을 알 필요가 없어서 해시를 씀</td>
</tr>
<tr>
<td><strong>Spring Security</strong></td>
<td>인증(누구인지) + 인가(무엇을 할 수 있는지)를 담당하는 프레임워크</td>
<td>요청이 컨트롤러에 가기 전에 필터 체인을 거침</td>
</tr>
<tr>
<td><strong>JWT</strong></td>
<td>사용자 정보와 서명을 담은 토큰, 서버가 세션을 저장하지 않아도 됨</td>
<td><code>Authorization: Bearer 토큰</code></td>
</tr>
</tbody></table>
<h3 id="코드로-다시-보기">코드로 다시 보기</h3>
<pre><code class="language-java">// 비밀번호는 평문이 아니라 BCrypt 결과를 저장한다
userEntity.setEncryptedPwd(passwordEncoder.encode(userDto.getPwd()));</code></pre>
<hr />
<h1 id="1-오늘-만든-e-commerce-애플리케이션의-구조">1. 오늘 만든 E-commerce 애플리케이션의 구조</h1>
<pre><code>Client
  │
  ▼
API Gateway (8000)
  ├─ USER-SERVICE     ─ User DB
  ├─ CATALOG-SERVICE  ─ Catalog DB
  └─ ORDER-SERVICE    ─ Order DB</code></pre><p>세 서비스가 하는 일과 호출 주소를 정리하면 이렇다.</p>
<table>
<thead>
<tr>
<th>서비스</th>
<th>하는 일</th>
<th>주요 API</th>
</tr>
</thead>
<tbody><tr>
<td>USER-SERVICE</td>
<td>회원 등록·조회, 로그인</td>
<td><code>POST /user-service/users</code>, <code>GET /user-service/users/{user_id}</code></td>
</tr>
<tr>
<td>CATALOG-SERVICE</td>
<td>상품 목록 제공</td>
<td><code>GET /catalog-service/catalogs</code></td>
</tr>
<tr>
<td>ORDER-SERVICE</td>
<td>주문 등록·조회</td>
<td><code>POST /order-service/{user_id}/orders</code>, <code>GET /order-service/{user_id}/orders</code></td>
</tr>
</tbody></table>
<ul>
<li>서비스마다 <strong>자기 DB를 따로</strong> 갖는다. 다른 서비스의 DB를 직접 건드리지 않는 게 MSA의 기본 원칙이다.</li>
<li>구성요소 중 Config Server, Eureka Server, API Gateway, Message Broker는 서비스들 바깥에서 도와주는 &quot;Outer&quot; 쪽이고, 회원·주문·상품 서비스가 실제 비즈니스를 맡는 &quot;Inner&quot; 쪽이다. 오늘은 이 중 Eureka와 Gateway까지만 실제로 연결했다.</li>
</ul>
<hr />
<h1 id="2-user-service-만들기--등록-포트-db">2. User Service 만들기 — 등록, 포트, DB</h1>
<pre><code class="language-java">@SpringBootApplication
@EnableDiscoveryClient            // ①
public class UserServiceApplication { ... }</code></pre>
<pre><code class="language-yaml">server:
  port: 0                         # ②

spring:
  application:
    name: user-service

eureka:
  instance:
    instanceId: ${spring.application.name}:${spring.application.instance_id:${random.value}}   # ③
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      defaultZone: http://localhost:8761/eureka</code></pre>
<ol>
<li><code>@EnableDiscoveryClient</code> — 이 앱이 Eureka에 자기 자신을 등록하고, 다른 서비스를 찾을 수 있게 한다.</li>
<li><code>port: 0</code> — 포트를 직접 정하지 않고 <strong>실행할 때마다 빈 포트를 랜덤으로</strong> 받는다. 같은 서비스를 여러 개 띄워도 포트가 겹치지 않는다. 대신 &quot;서비스가 어느 포트에 떴는지&quot;를 우리가 외울 수 없으니, 그걸 대신 알려주는 게 Eureka와 Gateway다.</li>
<li><code>instanceId</code>에 랜덤 값을 섞은 이유도 같다. 포트가 0이라 같은 이름의 인스턴스가 여러 개 뜰 수 있어서, Eureka에서 인스턴스를 서로 구분하려면 고유한 ID가 필요하다.</li>
</ol>
<p><strong>상태 확인 API로 먼저 확인하기</strong></p>
<pre><code class="language-java">@GetMapping(&quot;/health-check&quot;)
public String status() {
    return String.format(&quot;It's Working in User Service, port(local.server.port)=%s, port(server.port)=%s&quot;,
            env.getProperty(&quot;local.server.port&quot;), env.getProperty(&quot;server.port&quot;));
}</code></pre>
<p><code>server.port</code>는 설정값이라 <code>0</code>으로 나오지만 <code>local.server.port</code>는 <strong>실제로 할당된 포트</strong>가 나온다. 둘을 같이 찍어서 &quot;랜덤 포트가 진짜로 적용됐는지&quot; 눈으로 확인할 수 있다.</p>
<p><strong>H2 DB 연결</strong>: 의존성을 추가하고 <code>application.yml</code>에 H2 설정을 넣은 뒤 <code>h2-console</code>로 접속해 확인한다. 참고로 H2는 1.4.198 버전 이후부터 보안 때문에 <strong>데이터베이스를 자동으로 만들어주지 않아서</strong>, 설정에 DB 이름/URL을 직접 지정해줘야 했다.</p>
<hr />
<h1 id="3-회원가입--객체를-왜-이렇게-많이-나누는가">3. 회원가입 — 객체를 왜 이렇게 많이 나누는가</h1>
<p>회원가입 요청이 지나가는 길은 이렇다.</p>
<pre><code>RequestUser (요청 JSON을 받는 객체)
   → UserController
   → UserDto (계층 사이를 오가는 객체)
   → UserService
   → UserEntity (DB 테이블과 매핑되는 객체)
   → UserRepository
   → H2</code></pre><pre><code class="language-java">@Override
public UserDto createUser(UserDto userDto) {
    userDto.setUserId(UUID.randomUUID().toString());         // ①

    ModelMapper mapper = new ModelMapper();
    mapper.getConfiguration().setMatchingStrategy(MatchingStrategies.STRICT);   // ②
    UserEntity userEntity = mapper.map(userDto, UserEntity.class);              // ③

    userEntity.setEncryptedPwd(passwordEncoder.encode(userDto.getPwd()));       // ④
    userRepository.save(userEntity);
    ...
}</code></pre>
<ol>
<li><code>UUID</code> — 회원의 외부 식별자는 DB 자동 증가 번호 대신 UUID로 만든다. 번호가 1, 2, 3으로 노출되면 다음 회원 번호를 추측하기 쉬워서다.</li>
<li><code>STRICT</code> — <code>ModelMapper</code>가 필드를 짝지을 때 <strong>이름이 정확히 같은 것끼리만</strong> 매핑하게 한다. 느슨하게 두면 이름이 비슷한 필드끼리 엉뚱하게 연결될 수 있다.</li>
<li><code>mapper.map(userDto, UserEntity.class)</code> — DTO의 값을 하나하나 꺼내 Entity에 옮기는 반복 코드를 한 줄로 줄여준다.</li>
<li><code>encode(...)</code> — 비밀번호를 BCrypt 결과로 바꿔 저장한다. 처음 실습에서는 이 자리에 <code>&quot;encrypted_password&quot;</code>라는 <strong>임시 문자열</strong>을 넣어두고 흐름만 확인했다가, Security 설정을 마친 뒤 실제 <code>BCryptPasswordEncoder</code>로 바꿨다. 동작 흐름부터 확인하고 세부 구현을 나중에 채우는 순서였다.</li>
</ol>
<p><strong>왜 요청 객체를 그대로 Entity로 쓰지 않나</strong>: 요청 JSON에는 평문 비밀번호가 들어있고, Entity에는 DB 컬럼 정보가 들어있다. 하나의 클래스로 둘을 다 맡으면 화면에 불필요한 값이 노출되거나 DB 구조가 요청 형식에 끌려다닌다. 그래서 <code>RequestUser</code>, <code>UserDto</code>, <code>UserEntity</code>, <code>ResponseUser</code>로 역할을 나눴다.</p>
<p><strong><code>ResponseUser</code>의 <code>@JsonInclude(NON_NULL)</code></strong>: 값이 <code>null</code>인 필드는 응답 JSON에서 아예 빼는 설정이다. 예를 들어 <code>orders</code>를 아직 채우지 않았다면 <code>&quot;orders&quot;: null</code>이 내려가지 않고 그 필드가 통째로 사라진다. (빈 배열 <code>[]</code>은 <code>null</code>이 아니라서 그대로 내려간다.)</p>
<p><strong><code>CrudRepository</code>를 쓴 이유</strong>: 오늘 만든 Repository는 <code>JpaRepository</code>가 아니라 <code>CrudRepository</code>를 상속한다. 저장과 단순 조회만 필요해서, 페이징·<code>flush()</code> 같은 JPA 전용 기능까지 열어둘 이유가 없었다. 동작 여부의 차이가 아니라 &quot;필요한 기능만 쓴다&quot;는 선택이다.</p>
<hr />
<h1 id="4-spring-security-기본-설정">4. Spring Security 기본 설정</h1>
<pre><code class="language-java">@Bean
protected SecurityFilterChain configure(HttpSecurity http) throws Exception {
    http.csrf(csrf -&gt; csrf.disable())                                    // ①
        .authorizeHttpRequests(auth -&gt; auth
            .requestMatchers(&quot;/h2-console/**&quot;).permitAll()               // ②
            .anyRequest().authenticated())
        .httpBasic(Customizer.withDefaults())
        .headers(headers -&gt; headers
            .frameOptions(frame -&gt; frame.sameOrigin()));                 // ③
    return http.build();
}</code></pre>
<ol>
<li><code>csrf.disable()</code> — 세션/쿠키 기반이 아니라 토큰 기반으로 갈 것이라 CSRF 방어를 끈다.</li>
<li><code>permitAll()</code> / <code>authenticated()</code> — <code>h2-console</code> 경로는 인증 없이 허용하고, 나머지 모든 요청은 인증을 요구한다.</li>
<li><code>frameOptions().sameOrigin()</code> — 이 설정이 없으면 <strong>h2-console 화면이 깨져서 보인다.</strong> h2-console이 화면 안에 프레임을 쓰는데, Security가 기본으로 프레임 삽입을 막기 때문이다. 코드 로직과 상관없는 설정 한 줄이 화면 접근 여부를 가른 부분이라 기억해둘 만하다.</li>
</ol>
<p>Security 의존성을 추가하는 순간 <strong>모든 요청이 인증을 요구하는 상태</strong>가 되기 때문에, 기존에 열려 있던 API도 <code>permitAll</code>로 따로 열어주거나 인증 방법을 붙여야 했다.</p>
<hr />
<h1 id="5-gateway-라우팅--경로-맞추기-그리고-catalog--order-추가">5. Gateway 라우팅 — 경로 맞추기, 그리고 Catalog / Order 추가</h1>
<pre><code class="language-yaml">routes:
  - id: user-service
    uri: lb://USER-SERVICE
    predicates:
      - Path=/user-service/**
  - id: catalog-service
    uri: lb://CATALOG-SERVICE
    predicates:
      - Path=/catalog-service/**
  - id: order-service
    uri: lb://ORDER-SERVICE
    predicates:
      - Path=/order-service/**</code></pre>
<p>Client는 서비스의 랜덤 포트를 몰라도 <code>8000</code>번 Gateway 하나로 요청하면 된다. <code>lb://USER-SERVICE</code>는 &quot;Eureka에서 <code>USER-SERVICE</code>를 찾아서(<code>lb</code> = load balancer) 전달하라&quot;는 뜻이다.</p>
<p><strong>겪은 문제 — <code>404 Not Found</code></strong>: Gateway로 <code>/user-service/health-check</code>를 호출하면 <code>404</code>가 났다. Gateway는 <code>/user-service/**</code> 요청을 <strong>경로 그대로</strong> User Service에 넘기는데, 정작 User Service의 컨트롤러는 <code>/health-check</code>만 처리하도록 되어 있어서 경로가 안 맞았기 때문이다.</p>
<pre><code class="language-java">@RestController
@RequestMapping(&quot;/user-service&quot;)      // Gateway가 붙여서 보내는 앞부분과 맞춘다
public class UserController { ... }</code></pre>
<p>컨트롤러의 루트 경로를 <code>/user-service</code>로 바꿔서 해결했다. Gateway의 <code>Path</code> 조건과 컨트롤러의 <code>@RequestMapping</code>이 정확히 이어져야 한다는 걸 확인했다. (다만 이렇게 바꾸면 Gateway를 거치지 않고 서비스를 직접 호출할 때도 <code>/user-service/...</code>로 불러야 한다.)</p>
<p><strong>Catalog / Order 서비스</strong>: 구조는 User Service와 같다 — 프로젝트 생성 → 의존성/설정 → Entity·Repository → Dto·Response → Service → Controller → Gateway 등록 → 테스트.</p>
<pre><code class="language-java">// Catalog: data.sql로 상품 데이터를 미리 넣어둠
insert into catalog(product_id, product_name, stock, unit_price) values('CATALOG-0001', 'Berlin', 100, 1500);

// Order: 주문 ID는 UUID, 총금액은 수량 × 단가로 계산
orderDto.setOrderId(UUID.randomUUID().toString());
orderDto.setTotalPrice(orderDto.getQty() * orderDto.getUnitPrice());

// 사용자별 주문 조회
Iterable&lt;OrderEntity&gt; findByUserId(String userId);</code></pre>
<p>⚠️ <strong>아직 연결되지 않은 부분</strong>: 회원 상세 조회(<code>/users/{user_id}</code>)는 주문 목록을 응답에 넣게 되어 있는데, 지금은 빈 리스트로 채워서 내려준다. 세 서비스가 각자 돌아가고 Gateway에도 등록됐지만, <strong>User Service가 Order Service를 호출해 실제 주문 내역을 가져오는 서비스 간 통신은 아직 구현 전</strong>이다.</p>
<hr />
<h1 id="6-로그인--세션-대신-jwt를-쓰는-이유">6. 로그인 — 세션 대신 JWT를 쓰는 이유</h1>
<p><img alt="" src="https://velog.velcdn.com/images/hssondev/post/91734d39-755a-4043-be50-5d8f3b191c1e/image.png" /></p>
<p><strong>전통적인 세션 방식</strong>은 로그인하면 서버가 세션 ID를 쿠키로 내려주고, 이후 요청마다 그 쿠키를 보내는 식이다. 문제는 두 가지다. 세션과 쿠키는 <strong>모바일 앱에서 쓰기 어렵고</strong>, 서버가 여러 대일 때 세션을 공유해야 한다. 응답도 화면(HTML) 중심이라 앱이 원하는 JSON과 맞지 않는다.</p>
<p><strong>토큰 방식(JWT)</strong>은 로그인에 성공하면 서버가 서명한 토큰을 내려주고, 클라이언트가 이후 요청마다 <code>Authorization: Bearer 토큰</code>으로 보낸다. 서버가 상태를 저장하지 않아도 되고(stateless), 서버가 몇 대든 같은 방식으로 검증할 수 있다.</p>
<h2 id="6-1-로그인-요청-처리--authenticationfilter">6-1) 로그인 요청 처리 — <code>AuthenticationFilter</code></h2>
<pre><code class="language-java">public class AuthenticationFilter extends UsernamePasswordAuthenticationFilter {

    @Override
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response)
            throws AuthenticationException {
        try {
            RequestLogin creds = new ObjectMapper().readValue(request.getInputStream(), RequestLogin.class);  // ①
            return getAuthenticationManager().authenticate(
                    new UsernamePasswordAuthenticationToken(creds.getEmail(), creds.getPassword(), new ArrayList&lt;&gt;()));  // ②
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
    ...
}</code></pre>
<ol>
<li>요청 본문(JSON)을 <code>RequestLogin</code>(이메일, 비밀번호만 담은 객체)으로 읽는다.</li>
<li>읽어낸 이메일·비밀번호를 <code>UsernamePasswordAuthenticationToken</code>에 담아 <code>AuthenticationManager</code>에게 <strong>&quot;이 사람 맞는지 확인해줘&quot;</strong>라고 맡긴다. 실제 확인은 아래 <code>UserDetailsService</code>가 한다.</li>
</ol>
<p><strong>누가 비밀번호를 비교하는가</strong>: 로그인 확인은 우리가 직접 <code>if</code>로 비교하지 않는다. <code>UserService</code>가 <code>UserDetailsService</code>를 구현해서 <code>loadUserByUsername(email)</code>로 DB에서 <code>findByEmail()</code>로 회원을 꺼내주면, Spring Security가 입력한 비밀번호와 저장된 BCrypt 값을 알아서 비교한다. 이때 BCrypt는 복호화하지 않고, 저장된 값 안에 들어있는 salt와 설정으로 입력값을 다시 계산해서 같은지만 확인한다.</p>
<h2 id="6-2-인증에-성공하면-jwt-발급">6-2) 인증에 성공하면 JWT 발급</h2>
<pre><code class="language-java">@Override
protected void successfulAuthentication(HttpServletRequest req, HttpServletResponse res,
                                        FilterChain chain, Authentication authResult)
        throws IOException, ServletException {
    String userName = ((User) authResult.getPrincipal()).getUsername();
    UserDto userDetails = userService.getUserDetailsByEmail(userName);               // ①

    byte[] secretKeyBytes = environment.getProperty(&quot;token.secret&quot;).getBytes(StandardCharsets.UTF_8);
    SecretKey secretKey = Keys.hmacShaKeyFor(secretKeyBytes);                         // ②

    String token = Jwts.builder()
            .subject(userDetails.getUserId())                                         // ③
            .expiration(Date.from(now.plusMillis(Long.parseLong(environment.getProperty(&quot;token.expiration_time&quot;)))))
            .issuedAt(Date.from(now))
            .signWith(secretKey)                                                      // ④
            .compact();

    res.addHeader(&quot;token&quot;, token);                                                    // ⑤
    res.addHeader(&quot;userId&quot;, userDetails.getUserId());
}</code></pre>
<ol>
<li>인증된 사용자의 이메일로 회원 정보를 다시 조회한다.</li>
<li>서명에 쓸 비밀 키를 설정 파일(<code>token.secret</code>)에서 읽어 만든다. 키를 코드에 직접 적지 않고 설정으로 뺐다.</li>
<li>토큰의 주인(<code>subject</code>)으로 회원의 <code>userId</code>를 담는다.</li>
<li><code>signWith(...)</code> — 비밀 키로 서명해야 위조를 막을 수 있다.</li>
<li>만든 토큰을 <strong>응답 헤더</strong>에 담아 돌려준다.</li>
</ol>
<p>토큰 유효 시간은 설정 파일에 <code>86400000</code>(밀리초 단위, 슬라이드 주석으로는 약 10일)처럼 적어두고 읽어 쓴다. JWT 시간은 밀리초 기준이라는 점을 기억해둔다.</p>
<p><code>WebSecurity</code>에서는 모든 요청이 이 <code>AuthenticationFilter</code>를 거치도록 연결하고, 로그인 요청은 Gateway의 IP에서 온 것만 허용하는 식으로 설정을 바꿨다.</p>
<hr />
<h1 id="7-gateway에서-토큰-검사하기--authorizationheaderfilter">7. Gateway에서 토큰 검사하기 — <code>AuthorizationHeaderFilter</code></h1>
<p>로그인 후에는 다른 요청들을 Gateway가 먼저 검사한다. 각 서비스마다 토큰 검사 코드를 따로 두지 않고, <strong>Gateway 한 곳에서 한 번에</strong> 검사하는 구조다.</p>
<pre><code class="language-java">@Override
public GatewayFilter apply(Config config) {
    return (exchange, chain) -&gt; {
        ServerHttpRequest request = exchange.getRequest();

        if (!request.getHeaders().containsKey(HttpHeaders.AUTHORIZATION)) {              // ①
            return onError(exchange, &quot;No authorization header&quot;, HttpStatus.UNAUTHORIZED);
        }

        String authorizationHeader = request.getHeaders().get(HttpHeaders.AUTHORIZATION).get(0);
        String jwt = authorizationHeader.replace(&quot;Bearer&quot;, &quot;&quot;);                          // ②

        if (!isJwtValid(jwt)) {                                                          // ③
            return onError(exchange, &quot;JWT token is not valid&quot;, HttpStatus.UNAUTHORIZED);
        }

        return chain.filter(exchange);                                                   // ④
    };
}</code></pre>
<ol>
<li><code>Authorization</code> 헤더 자체가 없으면 바로 <code>401</code>로 돌려보낸다.</li>
<li>헤더 값에서 <code>&quot;Bearer&quot;</code> 글자를 지우고 진짜 토큰만 꺼낸다.</li>
<li>토큰이 유효한지(서명, 만료) 확인하고, 아니면 <code>401</code>.</li>
<li>모두 통과하면 <code>chain.filter(exchange)</code>로 요청을 다음 단계(서비스)로 넘긴다.</li>
</ol>
<p><strong>어떤 요청에 이 필터를 붙일지는 Gateway의 <code>application.yml</code>에서 라우트별로 정한다.</strong></p>
<table>
<thead>
<tr>
<th>요청</th>
<th>인증 필요</th>
</tr>
</thead>
<tbody><tr>
<td><code>POST /user-service/users</code> (회원가입)</td>
<td>필요 없음</td>
</tr>
<tr>
<td><code>POST /user-service/login</code> (로그인)</td>
<td>필요 없음</td>
</tr>
<tr>
<td><code>GET /user-service/users</code> (회원 정보 확인)</td>
<td>필요함</td>
</tr>
<tr>
<td><code>GET /user-service/health-check</code> (상태 체크)</td>
<td>필요함</td>
</tr>
</tbody></table>
<p>회원가입과 로그인은 토큰을 받기 <em>전</em>에 하는 일이라 당연히 토큰을 요구할 수 없다. 이 두 경로만 필터 없이 통과시키고, 나머지에는 <code>AuthorizationHeaderFilter</code>를 붙였다. 테스트도 Postman에서 같은 순서로 했다 — 회원가입과 로그인은 <code>No Auth</code>로, 이후 조회는 로그인 응답으로 받은 토큰을 <code>Bearer Token</code>에 넣어서.</p>
<p>이전에 Spring Boot 단일 서버에서 만든 <code>JwtFilter</code>의 <strong>화이트리스트</strong>와 같은 발상이다. 다만 거기서는 서버 안에서 경로 목록으로 거르고, 여기서는 Gateway 설정에서 라우트마다 필터를 붙일지 말지를 정한다는 차이가 있다.</p>
<hr />
<h1 id="✨-새롭게-알게-된-점">✨ 새롭게 알게 된 점</h1>
<ul>
<li>포트를 <code>0</code>으로 두면 매번 랜덤 포트로 뜨고, 서비스를 찾는 일은 Eureka와 Gateway가 맡는다.</li>
<li>요청·전달·저장·응답 객체를 나누는 건 번거로워 보여도, 평문 비밀번호 같은 값이 응답이나 DB 구조에 섞여 나가는 걸 막아준다.</li>
<li>BCrypt는 같은 비밀번호도 매번 다른 값을 만들고(랜덤 salt), 로그인 때는 복호화가 아니라 &quot;다시 계산해서 비교&quot;한다.</li>
<li>해시는 원본을 복구할 수 없는 단방향, 암호화는 키가 있으면 복구되는 양방향이라 비밀번호엔 해시가 맞다.</li>
<li>Spring Security 의존성을 추가하면 모든 요청이 기본으로 인증 대상이 되고, <code>h2-console</code>처럼 프레임을 쓰는 화면은 <code>frameOptions</code> 설정이 없으면 깨진다.</li>
<li>Gateway의 <code>Path</code> 조건과 컨트롤러의 <code>@RequestMapping</code>이 이어지지 않으면 <code>404</code>가 난다.</li>
<li>세션은 모바일·다중 서버에 불리하고, JWT는 서버가 상태를 저장하지 않아서 이런 환경에 잘 맞는다.</li>
</ul>
<hr />
<h1 id="🤔-헷갈렸던-점">🤔 헷갈렸던 점</h1>
<ul>
<li><strong><code>CrudRepository</code>와 <code>JpaRepository</code>의 선택 기준</strong>: 오늘 기능만 보면 둘 다 동작하는데, 실무에서는 어느 쪽을 기본으로 두는지 아직 감이 부족하다.</li>
<li><strong>로그인 비교를 우리가 안 하는 것</strong>: <code>attemptAuthentication</code>에서 <code>AuthenticationManager</code>에게 맡기면 누가 어디서 비밀번호를 비교하는지가 코드에 안 보여서 처음엔 흐름이 끊겨 보였다. <code>UserDetailsService</code>가 그 연결고리라는 걸 다이어그램으로 따라가고서야 이해됐다.</li>
<li><strong>Gateway에서 경로를 맞추는 방식</strong>: 컨트롤러 루트 경로를 <code>/user-service</code>로 바꾸면 직접 호출 주소도 같이 바뀌는데, 경로를 잘라서 전달하는 다른 방법이 있는지는 아직 모른다.</li>
<li><strong><code>&quot;Bearer&quot;</code> 처리</strong>: 슬라이드 코드에서 <code>replace(&quot;Bearer&quot;, &quot;&quot;)</code>로 지우는데, 뒤에 공백이 남을 수 있어서 <code>&quot;Bearer &quot;</code>처럼 공백까지 포함해 지우는 쪽이 더 안전한지 확인이 필요하다.</li>
</ul>
<hr />
<h1 id="다음에-할-일">다음에 할 일</h1>
<ul>
<li><input disabled="" type="checkbox" /> User Service가 Order Service를 호출해서 회원 상세 조회에 실제 주문 목록 채우기</li>
<li><input disabled="" type="checkbox" /> 주문이 생기면 Catalog Service의 재고를 줄이는 흐름 연결하기</li>
<li><input disabled="" type="checkbox" /> 서비스마다 DB가 따로일 때 데이터를 일관되게 유지하는 방법 알아보기</li>
<li><input disabled="" type="checkbox" /> <code>JwtFilter</code>(단일 서버)와 <code>AuthorizationHeaderFilter</code>(Gateway)의 검증 방식을 나란히 비교해보기</li>
</ul>
<hr />
<h1 id="💡-오늘의-회고">💡 오늘의 회고</h1>
<p>오늘은 &quot;기능을 쪼갠다&quot;는 말이 실제로 어떤 모양인지 처음 손으로 확인한 날이었다. 서비스가 셋으로 나뉘고 포트가 랜덤이 되니, 이전처럼 <code>localhost:8080</code>으로 바로 호출하는 방식은 더 이상 통하지 않았다. 그 대신 Eureka가 위치를 기억하고 Gateway가 길을 안내한다는 게 왜 필요한 장치인지 자연스럽게 이해됐다.</p>
<p><code>/user-service/health-check</code>가 <code>404</code>가 났던 일은 작은 경로 불일치였지만, Gateway와 컨트롤러가 서로 다른 곳에서 같은 약속(경로)을 지켜야 한다는 점에서 지난번 JWT의 <code>&quot;Bearer &quot;</code> 형식 문제와 닮아 있었다. 로그인과 토큰 검증도 결국 &quot;만드는 쪽과 검사하는 쪽이 같은 형식을 약속한다&quot;는 같은 원리 위에 있다는 게 이어졌다.</p>
<p>아직 서비스끼리 직접 통신하는 부분은 비어 있어서, 회원 상세 조회의 주문 목록이 빈 배열로 나오는 게 마음에 걸린다. 다음에는 그 빈 곳을 채우면서, 서비스가 나뉘면 데이터 정합성은 누가 어떻게 책임지는지 이어서 보고 싶다.</p>