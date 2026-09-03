<blockquote>
<p><strong>한 줄 요약</strong></p>
<p><code>@RequestBody</code>(JSON)와 <code>@RequestParam</code>(쿼리스트링)의 차이를 짚고, Swagger 어노테이션(<code>@Operation</code>, <code>@ApiResponses</code>, <code>@Schema</code>)으로 회원가입/로그인 API 문서를 자동으로 만들었으며, 로그인 성공 시 응답 헤더에 인증 토큰을 담아 돌려주는 것까지 실습했다.</p>
</blockquote>
<hr />
<h1 id="📚-오늘-학습한-커리큘럼">📚 오늘 학습한 커리큘럼</h1>
<pre><code>JSON Body vs QueryString — @RequestBody vs @RequestParam
        ↓
Swagger로 API 문서 자동화 — @Operation, @ApiResponses
        ↓
Swagger에 요청 스키마 명세하기 — @Schema(implementation=...)
        ↓
로그인 API — 응답 헤더에 인증 토큰 담기
        ↓
CORS 예고 (@Configuration)
        ↓
axios에서 같은 요청을 두 가지 방식으로 보내기</code></pre><h1 id="📌-오늘의-학습-키워드">📌 오늘의 학습 키워드</h1>
<ul>
<li><code>@RequestParam</code>, <code>@PathVariable</code>, <code>@RequestBody</code></li>
<li>Swagger(OpenAPI), <code>@Operation</code>, <code>@ApiResponses</code>, <code>@Schema</code></li>
<li><code>ResponseEntity</code>, <code>HttpHeaders</code></li>
<li>CORS, <code>@Configuration</code></li>
<li>axios <code>params</code> 옵션 vs 쿼리스트링 직접 조합</li>
</ul>
<hr />
<h1 id="0-개념-정리-키워드-사전">0. 개념 정리 (키워드 사전)</h1>
<blockquote>
<p><strong>한 줄 요약</strong></p>
<p><code>@RequestParam</code>/<code>@PathVariable</code>은 URL에 실려오는 값을, <code>@RequestBody</code>는 요청 몸통(JSON)에 실려오는 값을 받는다 — 그리고 Swagger는 이 API들이 어떤 값을 주고받는지 문서로 자동 정리해준다.</p>
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
<td><strong><code>@RequestParam</code></strong></td>
<td>URL의 <strong>쿼리스트링</strong>(<code>?key=value</code>)에서 값을 꺼냄</td>
<td><code>GET</code> 요청에서 자주 사용</td>
</tr>
<tr>
<td><strong><code>@PathVariable</code></strong></td>
<td>URL 경로 자체에 포함된 값을 꺼냄</td>
<td><code>/users/{id}</code>의 <code>{id}</code></td>
</tr>
<tr>
<td><strong><code>@RequestBody</code></strong></td>
<td>요청의 <strong>몸통(JSON)</strong>을 자바 객체로 변환해서 받음</td>
<td><code>POST</code>/<code>PUT</code>처럼 데이터를 몸통에 실어 보낼 때</td>
</tr>
<tr>
<td><strong>Swagger(OpenAPI)</strong></td>
<td>작성한 API의 요청/응답 형태를 <strong>자동으로 문서화</strong>해서 웹 화면으로 보여주는 도구</td>
<td><code>/swagger-ui/index.html</code>에서 확인</td>
</tr>
<tr>
<td><strong><code>@Operation</code></strong></td>
<td>이 API가 무슨 일을 하는지 한 줄 설명을 붙임</td>
<td><code>summary</code>, <code>description</code></td>
</tr>
<tr>
<td><strong><code>@ApiResponses</code></strong></td>
<td>이 API가 돌려줄 수 있는 응답 코드들을 나열</td>
<td><code>200</code>, <code>400</code>, <code>500</code> 등</td>
</tr>
<tr>
<td><strong><code>@Schema(implementation=...)</code></strong></td>
<td>이 요청/응답이 어떤 DTO 형태인지 Swagger 문서에 명세</td>
<td>문서 화면에 필드 구조가 그대로 보임</td>
</tr>
<tr>
<td><strong><code>ResponseEntity</code></strong></td>
<td>응답 상태코드, 헤더, 본문(body)을 한 번에 담아 반환하는 객체</td>
<td><code>.status(...).headers(...).body(...)</code></td>
</tr>
<tr>
<td><strong>CORS (Cross-Origin Resource Sharing)</strong></td>
<td>다른 출처(도메인/포트)의 요청을 허용할지 서버가 정하는 정책</td>
<td>React(3000번)와 Spring(8000번)처럼 포트가 다르면 기본적으로 막힘</td>
</tr>
</tbody></table>
<h3 id="코드로-다시-보기">코드로 다시 보기</h3>
<pre><code class="language-java">@PostMapping(&quot;/signUp&quot;)
public ResponseEntity&lt;?&gt; signUp(@RequestBody UserRequestDTO request) { ... }
// vs
@GetMapping(&quot;/signIn&quot;)
public ResponseEntity&lt;?&gt; signIn(@RequestParam String email, @RequestParam String password) { ... }</code></pre>
<hr />
<h1 id="1-requestbody-vs-requestparam--값이-어디에-실려오는가">1. <code>@RequestBody</code> vs <code>@RequestParam</code> — 값이 어디에 실려오는가</h1>
<p>오늘 헷갈렸던 지점이 &quot;JSON 데이터&quot;와 &quot;파라미터 데이터&quot;가 뭐가 다른가였는데, 결국 <strong>값이 요청의 어느 부분에 실려오는지</strong>의 차이였다.</p>
<ul>
<li><code>@RequestParam</code>: URL 뒤에 <code>?email=xxx&amp;password=xxx</code>처럼 붙는 <strong>쿼리스트링</strong>에서 값을 꺼낸다. <code>GET</code> 요청에서 자주 쓴다.</li>
<li><code>@PathVariable</code>: URL 경로 자체(<code>/users/{id}</code>)에 값이 박혀있는 경우.</li>
<li><code>@RequestBody</code>: 요청의 <strong>몸통(body)</strong>에 JSON 형태로 실려온 데이터를 자바 객체로 변환해서 받는다. 회원가입처럼 필드가 여러 개인 데이터를 보낼 때 주로 쓴다.</li>
</ul>
<p>⚠️ <strong>오늘 필기에 남긴 규칙 하나</strong>: <code>@GetMapping</code>일 때는 <code>@RequestBody</code>를 쓰지 않는다. <code>GET</code> 요청은 원래 몸통에 데이터를 싣지 않는 게 관례라서, 데이터는 쿼리스트링(<code>@RequestParam</code>)으로 보내고, 몸통에 데이터를 실어야 하는 <code>POST</code>/<code>PUT</code> 같은 요청에만 <code>@RequestBody</code>를 쓴다는 걸로 정리했다.</p>
<hr />
<h1 id="2-swagger로-api-문서-자동화하기--회원가입-api">2. Swagger로 API 문서 자동화하기 — 회원가입 API</h1>
<pre><code class="language-java">@Operation(summary = &quot;회원가입&quot;, description = &quot;신규가입(email,password,name)&quot;)   // ①
@ApiResponses({                                                                    // ②
    @ApiResponse(responseCode = &quot;201&quot;, description = &quot;가입성공&quot;),
    @ApiResponse(responseCode = &quot;400&quot;, description = &quot;유효성검사실패&quot;),
    @ApiResponse(responseCode = &quot;500&quot;, description = &quot;가입실패&quot;)
})
@PostMapping(&quot;/signUp&quot;)
public ResponseEntity&lt;?&gt; signUp(@RequestBody UserRequestDTO request) {
    int signUpFlag = userService.signUp(request);
    if (signUpFlag != 0) {                                                          // ③
        return ResponseEntity.status(HttpStatus.CREATED).body(null);
    } else {
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(null);
    }
}</code></pre>
<p><strong>한 줄씩 정리하면</strong></p>
<ol>
<li><code>@Operation(summary=..., description=...)</code> — 이 API가 뭘 하는 API인지, Swagger 문서 화면에 표시될 <strong>제목과 설명</strong>을 붙인다.</li>
<li><code>@ApiResponses({...})</code> — 이 API가 상황에 따라 돌려줄 수 있는 <strong>응답 코드 목록</strong>을 미리 문서에 적어둔다. 실제 로직과 별개로, &quot;이 API를 쓸 사람에게 어떤 응답이 올 수 있는지&quot; 미리 알려주는 역할이다.</li>
<li><code>signUpFlag != 0</code> — 예전에 배운 &quot;0/1로 성공 여부를 나타내는 관례&quot;가 여기서도 그대로 쓰여서, <code>1</code>(성공)이면 <code>201 CREATED</code>, <code>0</code>이면 <code>500</code>을 반환한다.</li>
</ol>
<p>이 API를 실제로 Swagger 화면(<code>/swagger-ui/index.html</code>)에서 열어보면, ②에서 나열한 응답 코드들이 아래처럼 그대로 정리되어 보인다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/hssondev/post/da2c904e-71f8-4055-8b65-fa8be02de23f/image.png" /></p>
<hr />
<h1 id="3-요청-몸통request-body에-스키마-명세-붙이기">3. 요청 몸통(Request Body)에 스키마 명세 붙이기</h1>
<pre><code class="language-java">@PostMapping(&quot;/signUp&quot;)
public ResponseEntity&lt;?&gt; signUp(
    @io.swagger.v3.oas.annotations.parameters.RequestBody(          // ①
        description = &quot;사용자 정보를 담는 DTO&quot;,
        required    = true,
        content     = @Content(
            schema  = @Schema(implementation = UserRequestDTO.class)  // ②
        )
    )
    @RequestBody UserRequestDTO request) {                             // ③
    ...
}</code></pre>
<p><strong>한 줄씩 정리하면</strong></p>
<ol>
<li><code>@io.swagger.v3.oas.annotations.parameters.RequestBody(...)</code> — Spring의 <code>@RequestBody</code>와 <strong>이름은 같지만 다른 어노테이션</strong>이다. 이건 Swagger 문서에 &quot;이 요청 몸통이 어떤 설명과 구조를 갖는지&quot; 적어두는 용도라서, 패키지 전체 경로(<code>io.swagger.v3...</code>)를 그대로 적어서 구분했다.</li>
<li><code>@Schema(implementation = UserRequestDTO.class)</code> — &quot;이 요청 몸통은 <code>UserRequestDTO</code> 클래스와 같은 구조다&quot;라고 Swagger에게 알려준다. Swagger는 이걸 보고 <code>UserRequestDTO</code>의 필드(<code>email</code>, <code>password</code>, <code>name</code>)를 자동으로 읽어서 문서에 펼쳐 보여준다.</li>
<li><code>@RequestBody UserRequestDTO request</code> — 실제로 요청 몸통을 자바 객체로 받는, Spring이 쓰는 진짜 어노테이션.</li>
</ol>
<p>이렇게 스키마를 붙여두면, Swagger 문서 화면에서 이 API를 테스트해볼 때 아래처럼 <strong>DTO 구조가 그대로 펼쳐져서</strong> 어떤 필드를 채워야 하는지 바로 보여준다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/hssondev/post/c34dd05e-bb8e-4d03-8981-87246d5e85cf/image.png" /></p>
<hr />
<h1 id="4-로그인-api--응답-헤더에-인증-토큰-담기">4. 로그인 API — 응답 헤더에 인증 토큰 담기</h1>
<pre><code class="language-java">@Operation(summary = &quot;로그인&quot;, description = &quot;사용자 로그인(email, password)&quot;)
@ApiResponses({
    @ApiResponse(responseCode = &quot;200&quot;, description = &quot;로그인 성공&quot;),
    @ApiResponse(responseCode = &quot;400&quot;, description = &quot;로그인 실패&quot;),
})
@PostMapping(&quot;/signIn&quot;)
public ResponseEntity&lt;?&gt; signIn(@RequestBody UserRequestDTO request) {

    Map&lt;String, Object&gt; map = userService.signIn(request);           // ①

    HttpHeaders headers = new HttpHeaders();                          // ②
    headers.add(&quot;Authorization&quot;, (String)(map.get(&quot;at&quot;)));
    headers.add(&quot;Refresh-Token&quot;, (String)(map.get(&quot;rt&quot;)));
    headers.add(&quot;Access-Control-Expose-Headers&quot;, &quot;Authorization,Refresh-Token&quot;);  // ③

    return ResponseEntity
            .status(HttpStatus.OK)
            .headers(headers)                                          // ④
            .body((UserResponseDTO)(map.get(&quot;response&quot;)));
}</code></pre>
<p><strong>한 줄씩 정리하면</strong></p>
<ol>
<li><code>userService.signIn(request)</code> — 로그인 처리 결과를, 단순한 DTO 하나가 아니라 <strong><code>Map</code>에 여러 값을(<code>at</code>=Access Token, <code>rt</code>=Refresh Token, <code>response</code>=사용자 정보) 함께 담아</strong> 받았다.</li>
<li><code>HttpHeaders headers = new HttpHeaders()</code> — 응답에 실어 보낼 <strong>헤더를 따로 준비</strong>한다. 지금까지는 <code>body</code>에만 데이터를 담았는데, 오늘은 처음으로 헤더에도 값을 실었다.</li>
<li><code>Access-Control-Expose-Headers</code> — 브라우저는 기본적으로 CORS 요청에서 서버가 보낸 <strong>일부 헤더만</strong> 자바스크립트 코드가 읽을 수 있게 허용한다. 우리가 새로 추가한 <code>Authorization</code>, <code>Refresh-Token</code>은 기본 허용 목록에 없어서, 이 헤더로 &quot;이 두 헤더는 프론트에서 읽어도 된다&quot;고 명시적으로 노출해야 했다.</li>
<li><code>.headers(headers)</code> — 준비한 헤더를 <code>ResponseEntity</code>에 실어서 함께 반환한다.</li>
</ol>
<p>실제로 이 API를 호출하면, 응답 몸통(사용자 정보)과 함께 <code>authorization</code>, <code>refresh-token</code> 헤더가 같이 내려오는 걸 Swagger 화면에서도 확인할 수 있었다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/hssondev/post/7f8e611c-069e-46d1-8f02-5ff8a6db54b8/image.png" /></p>
<hr />
<h1 id="5-cors--이름만-예고">5. CORS — 이름만 예고</h1>
<p>오늘은 &quot;다른 출처의 요청을 서버가 허용해줘야 한다&quot;는 개념과, <code>@Configuration</code>으로 그 설정을 관리한다는 이름만 짧게 스쳤다. 3번 섹션에서 겪은 <code>Access-Control-Expose-Headers</code>도 결국 CORS 정책의 일부라는 걸 미리 몸으로 겪어본 셈이라, 다음 시간에 개념을 배우면 오늘 코드가 왜 필요했는지 더 잘 이어질 것 같다.</p>
<hr />
<h1 id="6-axios에서-같은-요청을-두-가지-방식으로-보내기">6. axios에서 같은 요청을 두 가지 방식으로 보내기</h1>
<pre><code class="language-js">// 방식 A: params 옵션 사용
await api.get('/users/signIn', {
    params: { email: form.email, password: form.password }
})
.then(response =&gt; { console.log(response); })
.catch(error =&gt; { console.log(error); });</code></pre>
<pre><code class="language-js">// 방식 B: 쿼리스트링을 직접 문자열로 조합
await api.get(`/users/signIn?email=${form.email}&amp;password=${form.password}`)
.then(response =&gt; { console.log(response); })
.catch(error =&gt; { console.log(error); });</code></pre>
<ul>
<li>두 코드는 <strong>완전히 같은 요청</strong>을 만든다. <code>params</code> 옵션에 객체를 넘기면 axios가 알아서 <code>?email=...&amp;password=...</code> 형태로 URL을 조립해주고, 방식 B는 그 조립을 우리가 직접 문자열로 미리 해둔 것뿐이다.</li>
<li>실제로 브라우저 개발자 도구로 두 방식의 요청을 확인해보면, <code>responseURL</code>이 <code>http://localhost:8000/users/signIn?email=shs10025@naver.com&amp;password=1234</code>처럼 <strong>똑같은 형태</strong>로 나가는 걸 볼 수 있었다.</li>
</ul>
<p><img alt="" src="https://velog.velcdn.com/images/hssondev/post/3df127a1-6530-4b4a-a075-3cf388ff434b/image.png" /></p>
<p>이미지에서 확인할 수 있듯, 응답의 <code>headers</code>에 <code>authorization: 'Bearer XXXXX'</code>와 <code>refresh-token</code>이 실제로 담겨서 돌아왔고, <code>data</code>에는 <code>email</code>, <code>name</code>, <code>password</code>가 담긴 사용자 정보가 그대로 들어있었다 — 4번 섹션에서 서버가 만든 응답 구조(헤더 + body)가 프론트에서 정확히 그대로 수신된 걸 눈으로 확인한 부분이다.</p>
<hr />
<h1 id="✨-새롭게-알게-된-점">✨ 새롭게 알게 된 점</h1>
<ul>
<li><code>GET</code> 요청엔 관례상 <code>@RequestBody</code>를 쓰지 않고, 데이터는 <code>@RequestParam</code>(쿼리스트링)으로 보낸다.</li>
<li>Swagger의 <code>@io.swagger.v3...RequestBody</code>와 Spring의 <code>@RequestBody</code>는 이름은 같지만 역할이 다른 별개의 어노테이션이다.</li>
<li><code>@Schema(implementation=...)</code>를 붙이면 Swagger 문서에 DTO 구조가 자동으로 펼쳐져 보인다.</li>
<li>응답 헤더에 값을 실을 수 있고(<code>HttpHeaders</code>), CORS 환경에서는 <code>Access-Control-Expose-Headers</code>로 프론트가 읽을 수 있는 헤더를 명시적으로 허용해줘야 한다.</li>
<li>axios의 <code>params</code> 옵션과 직접 쿼리스트링을 조합하는 것은 결과적으로 완전히 같은 요청을 만든다.</li>
</ul>
<hr />
<h1 id="🤔-헷갈렸던-점">🤔 헷갈렸던 점</h1>
<ul>
<li><strong>JSON body와 쿼리스트링 파라미터의 근본적 차이</strong>: 둘 다 &quot;클라이언트가 서버에 값을 보내는 방법&quot;이라는 점은 같은데, 언제 어느 쪽을 써야 하는지(대체로 GET엔 파라미터, POST엔 body) 감이 아직 완전히 잡히진 않았다.</li>
<li><strong><code>Access-Control-Expose-Headers</code>가 왜 필요한지</strong>: 서버가 헤더를 분명히 보냈는데 왜 프론트에서 못 읽을 수 있는지, CORS 개념을 아직 안 배운 상태라 &quot;일단 이렇게 하면 된다&quot;로만 알아뒀다.</li>
<li><strong>같은 이름, 다른 어노테이션(<code>@RequestBody</code>)</strong>: Spring 것과 Swagger 것을 구분하려고 패키지 전체 경로를 써야 했는데, 왜 굳이 이름이 겹치게 설계됐는지는 의문이 남는다.</li>
</ul>
<hr />
<h1 id="다음에-할-일">다음에 할 일</h1>
<ul>
<li><input disabled="" type="checkbox" /> CORS 개념을 정식으로 배우고 나서, 오늘 겪은 <code>Access-Control-Expose-Headers</code>가 왜 필요했는지 다시 정리하기</li>
<li><input disabled="" type="checkbox" /> Access Token(<code>at</code>)과 Refresh Token(<code>rt</code>)이 각각 어떤 역할인지(로그인 유지 방식) 찾아보기</li>
<li><input disabled="" type="checkbox" /> <code>@PathVariable</code> 예제도 직접 만들어서 <code>@RequestParam</code>과 비교해보기</li>
</ul>
<hr />
<h1 id="💡-오늘의-회고">💡 오늘의 회고</h1>
<p>오늘은 API를 &quot;만드는 것&quot;뿐 아니라 &quot;문서로 남기는 것&quot;까지 처음 해봤다. Swagger 화면에서 내가 만든 API의 요청/응답 구조가 자동으로 정리되어 나오는 걸 보니, 그동안 콘솔 로그로만 확인하던 결과를 처음으로 &quot;문서&quot;라는 형태로 눈으로 확인한 느낌이었다.</p>
<p>로그인 응답에 헤더로 토큰을 실어 보내는 부분은 아직 CORS를 안 배운 상태라 &quot;일단 이렇게 해야 동작한다&quot;는 느낌으로 따라갔는데, <code>Access-Control-Expose-Headers</code>를 빼먹으면 분명 서버는 보냈는데 프론트에서 못 읽는 상황이 생길 수 있다는 걸 짐작만 해뒀다. axios의 두 가지 요청 방식이 실제로 똑같은 결과를 낸다는 걸 devtools로 직접 확인한 것도, &quot;다른 코드처럼 보여도 결국 같은 요청일 수 있다&quot;는 걸 실감한 부분이었다.</p>