<blockquote>
<p><strong>한 줄 요약</strong></p>
<p>Filter/Interceptor/AOP 개념을 정리한 뒤, React(axios)와 Spring Boot 양쪽 코드를 하나씩 맞춰가며 댓글 삭제·수정 API를 완성하고, 로그인 실패를 전역으로 처리하는 <code>@RestControllerAdvice</code>를 만들었으며, 회원가입 폼에 <code>@Valid</code> 기반 유효성 검사를 붙여 필드별 에러 메시지가 화면까지 이어지는 흐름을 완성했다.</p>
</blockquote>
<hr />
<h1 id="📚-오늘-학습한-커리큘럼">📚 오늘 학습한 커리큘럼</h1>
<pre><code>가로채는 세 지점 — Filter, Interceptor, AOP
        ↓
프론트-백 연동 ① 댓글 삭제 — axios DELETE ↔ @DeleteMapping
        ↓
프론트-백 연동 ② 댓글 수정 — PathVariable 방식으로 시행착오
        ↓
전역 예외 처리 — HandlerExceptionResolver 체인 + @RestControllerAdvice
        ↓
회원가입 유효성 검사 — @Valid + Bean Validation
        ↓
프론트-백 연동 ③ 검증 실패 메시지를 화면에 표시하기</code></pre><h1 id="📌-오늘의-학습-키워드">📌 오늘의 학습 키워드</h1>
<ul>
<li>Filter, Interceptor, AOP</li>
<li><code>@DeleteMapping</code>, <code>@PatchMapping</code>, 여러 개의 <code>@PathVariable</code></li>
<li><code>HandlerExceptionResolver</code>, <code>@RestControllerAdvice</code>, <code>@ExceptionHandler</code></li>
<li><code>@Valid</code>, <code>@NotBlank</code>, <code>@Email</code>, <code>@Size</code></li>
<li><code>BindingResult</code>, <code>FieldError</code></li>
</ul>
<hr />
<h1 id="0-개념-정리-키워드-사전">0. 개념 정리 (키워드 사전)</h1>
<blockquote>
<p><strong>한 줄 요약</strong></p>
<p>오늘은 프론트가 보낸 요청 하나하나가 백엔드의 어느 코드로 이어지는지 계속 짝을 맞춰봤고, 잘못된 요청(로그인 실패, 유효성 검사 실패)을 한곳에서 정리해서 처리하는 방법을 배웠다.</p>
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
<td><strong><code>@DeleteMapping</code>/<code>@PatchMapping</code></strong></td>
<td><code>DELETE</code>/<code>PATCH</code> 방식 요청을 처리하는 메서드 표시</td>
<td><code>@GetMapping</code>/<code>@PostMapping</code>과 같은 계열</td>
</tr>
<tr>
<td><strong><code>@RestControllerAdvice</code></strong></td>
<td>여러 컨트롤러에서 발생하는 예외를 <strong>한곳에서 모아 처리</strong>하는 클래스 표시</td>
<td>전역 예외 처리기</td>
</tr>
<tr>
<td><strong><code>@ExceptionHandler</code></strong></td>
<td>특정 예외 타입이 발생했을 때 실행할 메서드를 지정</td>
<td><code>@ExceptionHandler(LoginFailException.class)</code></td>
</tr>
<tr>
<td><strong><code>@Valid</code></strong></td>
<td>이 요청 데이터에 대해 <strong>유효성 검사를 실행하라</strong>는 표시</td>
<td>DTO 필드에 붙인 검증 어노테이션들을 실제로 동작시킴</td>
</tr>
<tr>
<td><strong><code>@NotBlank</code>/<code>@Email</code>/<code>@Size</code></strong></td>
<td>각각 &quot;공백 없이 값 필수&quot;, &quot;이메일 형식&quot;, &quot;길이 범위&quot;를 검증</td>
<td>Bean Validation(자바 표준) 어노테이션</td>
</tr>
<tr>
<td><strong><code>BindingResult</code></strong></td>
<td><code>@Valid</code> 검증 결과(에러 유무, 에러 목록)를 담은 객체</td>
<td><code>@Valid</code> 매개변수 바로 뒤에 위치해야 함</td>
</tr>
<tr>
<td><strong><code>FieldError</code></strong></td>
<td><code>BindingResult</code> 안에 담긴, <strong>어느 필드에서 어떤 이유로</strong> 실패했는지의 정보</td>
<td><code>getField()</code>, <code>getDefaultMessage()</code></td>
</tr>
</tbody></table>
<h3 id="코드로-다시-보기">코드로 다시 보기</h3>
<pre><code class="language-java">@PostMapping(&quot;/signUp&quot;)
public ResponseEntity&lt;?&gt; signUp(@Valid @RequestBody UserRequestDTO request,
                                 BindingResult bindingResult) {
    if (bindingResult.hasErrors()) {
        // 필드별 에러 메시지를 모아서 400으로 반환
    }
    ...
}</code></pre>
<hr />
<h1 id="1-가로채는-세-지점--filter-interceptor-aop">1. 가로채는 세 지점 — Filter, Interceptor, AOP</h1>
<p>오늘 오전엔 지금까지 배운 걸 다시 훑으면서, &quot;모든 요청마다 공통으로 해야 하는 일(예: 토큰 확인)을 컨트롤러마다 반복하지 않으려면 어디서 가로채야 하는가&quot;라는 개념이 먼저 나왔다.</p>
<table>
<thead>
<tr>
<th></th>
<th>위치</th>
<th>소속</th>
</tr>
</thead>
<tbody><tr>
<td><strong>Filter</strong></td>
<td>프론트 컨트롤러(DispatcherServlet) <strong>앞</strong></td>
<td>웹(서블릿) 컴포넌트</td>
</tr>
<tr>
<td><strong>Interceptor</strong></td>
<td>프론트 컨트롤러 <strong>뒤</strong>, 컨트롤러 앞</td>
<td>스프링 컴포넌트</td>
</tr>
<tr>
<td><strong>AOP</strong></td>
<td>컨트롤러와 서비스 <strong>사이</strong></td>
<td>스프링 컴포넌트</td>
</tr>
</tbody></table>
<ul>
<li><strong>위치가 소속을 가른다</strong>: 프론트 컨트롤러보다 앞이면 스프링이 아니라 웹 컴포넌트, 뒤면 스프링 컴포넌트다.</li>
<li><strong>Filter</strong>는 요청이 프론트 컨트롤러에 닿기 전에 &quot;토큰 있어?&quot;를 먼저 묻는다. 없으면 거절, 있으면 통과시킨다 — 정수기 필터처럼 걸러낸다는 비유로 정리했다. 서명이 유효한지, 만료되지 않았는지까지 이 단계에서 검증한다.</li>
<li>로그인·회원가입처럼 토큰이 필요 없는 엔드포인트는 <strong>화이트리스트</strong>(배열)로 만들어 검증을 건너뛰게 한다.</li>
<li><strong>AOP</strong>는 컨트롤러와 서비스 사이에 <strong>크로스커팅</strong>(여러 지점에 공통으로 걸치는 방식)으로 들어간다.</li>
<li>오늘은 <strong>Filter와 AOP만 배우고 Interceptor는 다루지 않는다</strong>고 정리됐다 — 실무에서 쓸 일이 적기 때문이라고 한다. Filter는 다음 주 시큐리티에서 실제로 만들어볼 예정이다.</li>
</ul>
<hr />
<h1 id="2-프론트-백-연동-①--댓글-삭제">2. 프론트-백 연동 ① — 댓글 삭제</h1>
<p><strong>JS</strong> · <code>BlogReadPage.jsx</code></p>
<pre><code class="language-js">const commentDeleteHandler = async (e, id) =&gt; {
    // spring boot version
    await api.delete(`/comments/delete/${id}`, {                 // ①
        headers: { Authorization: at ? at : &quot;&quot; }                   // ②
    })
    .then(response =&gt; {
        if (response.status === 204) {                              // ③
            setComments(comments.filter((c) =&gt; c.id !== id));
        }
    })
    .catch(error =&gt; { console.log(error); });
};</code></pre>
<p><strong>Java</strong> · <code>CommentController.java</code></p>
<pre><code class="language-java">@DeleteMapping(&quot;/delete/{id}&quot;)
public ResponseEntity&lt;?&gt; delete(@PathVariable(&quot;id&quot;) Integer id) {   // ④
    int flag = commentService.delete(id);
    return (flag != 0)
        ? ResponseEntity.status(HttpStatus.NO_CONTENT).build()        // ⑤
        : ResponseEntity.status(HttpStatus.NOT_FOUND).build();
}</code></pre>
<p><strong>한 줄씩 정리하면</strong></p>
<ol>
<li><code>api.delete(...)</code> — URL 안에 삭제할 댓글의 <code>id</code>를 담아 <code>DELETE</code> 요청을 보낸다.</li>
<li><code>headers: { Authorization: at }</code> — 요청에 어제 만든 인증 토큰을 실어 보낸다. 로그인한 사용자만 삭제할 수 있어야 하니, 이 헤더로 &quot;나 이 사람이야&quot;라고 증명하는 셈이다.</li>
<li><code>response.status === 204</code> — 백엔드가 성공 시 <code>204 NO_CONTENT</code>를 돌려줄 걸 알고 있으니, 그 값과 정확히 맞춰서 프론트의 성공 처리 분기를 짰다. (처음엔 <code>200</code>으로 확인하다가 <code>204</code>로 고친 흔적이 diff에 남아있었다 — 백엔드 코드와 상태코드를 맞추는 과정이 있었던 것으로 보인다.)</li>
<li><code>@PathVariable(&quot;id&quot;) Integer id</code> — URL 경로에 실린 <code>id</code>를 꺼낸다. 프론트가 보낸 <code>/comments/delete/3</code>의 <code>3</code>이 여기로 들어온다.</li>
<li><code>204 NO_CONTENT</code> / <code>404 NOT_FOUND</code> — 삭제 성공 시 &quot;본문은 없지만 성공&quot;이라는 뜻의 <code>204</code>를, 대상이 없으면 <code>404</code>를 반환한다.</li>
</ol>
<p><strong>전체 왕복 흐름</strong></p>
<pre><code>[React] &quot;삭제&quot; 클릭
   → DELETE /comments/delete/3  (Authorization 헤더 포함)
[Spring] CommentController.delete(3)
   → CommentService.delete(3)
   → CommentMapper.delete(3)  → DB에서 실제 삭제
   ← 204 NO_CONTENT
[React] response.status === 204 확인
   → 화면의 댓글 목록에서 해당 댓글 제거</code></pre><hr />
<h1 id="3-프론트-백-연동-②--댓글-수정-방식을-바꾼-시행착오">3. 프론트-백 연동 ② — 댓글 수정, 방식을 바꾼 시행착오</h1>
<p>댓글 수정 기능은 오늘 <strong>한 번에 완성되지 않고, 방식을 바꿔가며</strong> 만들어졌다.</p>
<p><strong>PATCH를 선택한 이유</strong>: PATCH와 PUT은 둘 다 &quot;수정&quot;이지만, 리소스의 <strong>일부만</strong> 고칠 때는 PATCH를, 리소스 <strong>전체</strong>를 덮어쓸 때는 PUT을 쓰는 게 관례라고 정리했다. 댓글 내용 하나만 바꾸는 이번 케이스는 PATCH가 맞다.</p>
<p>⚠️ <strong>axios 메서드별 인자 순서 주의</strong>: <code>api.get(url, config)</code>은 두 번째 자리가 설정(헤더 등)인데, <code>api.patch(url, data, config)</code>는 두 번째 자리가 <strong>데이터</strong>이고 헤더는 세 번째로 밀린다. GET에 익숙해진 채로 PATCH를 쓰면 이 자리를 헷갈리기 쉽다는 걸 짚어두었다.</p>
<p><strong>JS</strong> · 첫 번째 시도 — body로 수정 내용 전달</p>
<pre><code class="language-js">// await api.patch(`/comments/${id}`, {
//     comment: mention
// }, { headers: { Authorization: at ? at : &quot;&quot; } })</code></pre>
<p><strong>JS</strong> · 최종 방식 — URL 경로에 id와 수정 내용을 함께 담기</p>
<pre><code class="language-js">await api.patch(
    `/comments/update/${id}/${encodeURIComponent(mention)}`,   // ①
    null,                                                          // ②
    { headers: { Authorization: at ? at : &quot;&quot; } }
)
.then(response =&gt; {
    if (response.status === 204) {
        setComments(ary =&gt; ary.map(comment =&gt;
            comment.id === id ? { ...comment, comment: mention } : comment
        ));
    }
});</code></pre>
<p><strong>Java</strong> · <code>CommentController.java</code> — Method 1(주석 처리) vs Method 2(채택)</p>
<pre><code class="language-java">@PatchMapping(&quot;/update/{id}/{comment}&quot;)
//Method 1
// public ResponseEntity&lt;?&gt; update(@PathVariable(&quot;id&quot;) Integer id,
//                                  @RequestBody Map&lt;String,Object&gt; map) { ... }

//Method 2 (채택)
public ResponseEntity&lt;?&gt; update(@PathVariable(&quot;id&quot;) Integer id,        // ③
                                 @PathVariable(&quot;comment&quot;) String comment) {
    Map&lt;String, Object&gt; map = new HashMap&lt;&gt;();                          // ④
    map.put(&quot;id&quot;, id);
    map.put(&quot;comment&quot;, comment);

    int flag = commentService.update(map);
    return (flag != 0)
        ? ResponseEntity.status(HttpStatus.NO_CONTENT).build()
        : ResponseEntity.status(HttpStatus.NOT_FOUND).build();
}</code></pre>
<p><strong>한 줄씩 정리하면</strong></p>
<ol>
<li><code>encodeURIComponent(mention)</code> — 수정할 댓글 내용을 <strong>URL 경로 자체에</strong> 실어 보내기로 했기 때문에, 댓글 안에 공백이나 특수문자, 슬래시(<code>/</code>)가 있으면 URL이 깨질 수 있다. <code>encodeURIComponent</code>로 그런 문자를 URL에 안전한 형태로 미리 인코딩했다.</li>
<li><code>null</code> — <code>patch(url, body, config)</code>에서 두 번째 자리는 원래 body 자리인데, 이번엔 데이터를 전부 URL 경로에 실었으니 body는 안 쓴다는 뜻으로 <code>null</code>을 넣었다.</li>
<li><code>@PathVariable(&quot;comment&quot;) String comment</code> — 프론트에서 인코딩해서 보낸 댓글 내용을, 경로 변수로 그대로 받는다.</li>
<li><code>Map&lt;String, Object&gt; map</code> — <code>CommentMapper.update()</code>가 <code>id</code>와 <code>comment</code> 두 값을 한 번에 받을 수 있도록, DTO 대신 간단히 <code>Map</code>으로 묶어서 넘겼다.</li>
</ol>
<p>⚠️ <strong>왜 Method 1(body로 받기)을 버리고 Method 2(경로로 받기)를 택했을까</strong>: <code>@PathVariable</code>을 여러 개(<code>{id}/{comment}</code>) 쓸 수 있다는 걸 실습해보려는 의도도 있었지만, 더 근본적인 이유는 <strong>URL 경로에 값을 실으면 헤더 없이도 어디서든 요청을 재현하기 쉽다</strong>는 것과, 최근 트렌드상 이런 방식이 자주 쓰인다는 설명이 있었다. 다만 데이터가 길어지거나 민감한 값이면 URL에 그대로 노출되니, 그런 경우엔 다시 Method 1(body)이 더 나을 수 있다.</p>
<p><strong>ORM에는 매개변수를 하나만 넘길 수 있다</strong> — <code>commentService.update()</code>에서 <code>id</code>와 <code>comment</code> <strong>두 값</strong>을 매퍼로 넘겨야 하는데, MyBatis(ORM)는 매개변수를 하나만 받는다. ORM 자체가 <strong>객체 하나를 테이블과 매핑</strong>하는 기술이라, 두 값을 넘기려면 DTO로 묶거나 <code>Map</code>으로 묶는 두 가지 방법 중 하나를 골라야 한다 — 오늘은 다양한 패턴을 보여주려고 <code>Map</code>을 택했다.</p>
<p><strong>XML</strong> · <code>CommentMapper.xml</code> — <code>HashMap</code>을 파라미터로</p>
<pre><code class="language-xml">&lt;update id=&quot;update&quot;
        parameterType=&quot;java.util.HashMap&quot;&gt;
    UPDATE SPRING_COMMENT_TBL
    SET COMMENT = #{comment}
    WHERE ID = #{id}
&lt;/update&gt;</code></pre>
<p>⚠️ <strong><code>parameterType</code>에 <code>Map</code>이 아니라 <code>HashMap</code>을 쓴 이유</strong>: <code>Map</code>은 <strong>인터페이스</strong>라서 프레임워크가 그 자체로는 인스턴스를 만들 수 없다(어느 구현체를 써야 할지 모른다). 그대로 두면 <code>BeanCreationException</code>이 난다. 그래서 <strong>구체 클래스</strong>인 <code>java.util.HashMap</code>을 명시해야 한다 — 예전에 배운 &quot;인터페이스는 <code>new</code>할 수 없고 구현체가 필요하다&quot;는 원칙이 여기서 실제 에러로 나타난 것이다. <code>#{comment}</code>, <code>#{id}</code>는 이제 DTO의 getter가 아니라 <strong><code>Map</code>에 담긴 키</strong>를 그대로 참조한다.</p>
<hr />
<h1 id="4-전역-예외-처리--처리되지-않은-예외는-어디로-가는가">4. 전역 예외 처리 — 처리되지 않은 예외는 어디로 가는가</h1>
<p>로그인에 실패했을 때 Swagger로 확인해보니 처음엔 <code>500 Internal Server Error</code>가 그대로 떨어졌다. 왜 이런 일이 생기는지, 예외가 흘러가는 경로부터 짚었다.</p>
<pre><code>Service에서 예외 발생
      ↓  컨트롤러에 try-catch가 없다 → 그냥 통과(throws)
DispatcherServlet까지 전달
      ↓  여기서도 처리가 없으면
HandlerExceptionResolver  ← 프레임워크의 전역 예외 처리기 체인
      ↓  등록된 핸들러 중에 이 예외를 처리할 것이 있는가?
   있다 → 그 핸들러가 응답을 만든다
   없다 → 500 Internal Server Error</code></pre><ul>
<li>지금까지 <code>500</code>이 떨어진 이유는, 그 예외를 처리할 핸들러가 이 체인에 <strong>등록되어 있지 않았기</strong> 때문이다. 그렇다면 <strong>우리가 만든 핸들러를 이 체인에 등록</strong>하면 되고, 그게 바로 <code>@RestControllerAdvice</code>다.</li>
<li>그래서 <strong>컨트롤러에 <code>try-catch</code>가 필요 없어진다</strong> — 실무에서도 Spring Boot 컨트롤러엔 <code>try-catch</code>가 거의 없고, 서비스에서 예외를 던지기만 하면 전역 핸들러가 대신 처리하는 구조가 일반적이라고 한다.</li>
</ul>
<pre><code class="language-java">public class LoginFailException extends RuntimeException {     // ①
    public LoginFailException(String message) { super(message); }
}</code></pre>
<pre><code class="language-java">@RestControllerAdvice                                            // ②
public class GlobalExceptionHandler {

    @ExceptionHandler(LoginFailException.class)                    // ③
    public ResponseEntity&lt;?&gt; handlerLoginFail(LoginFailException e) {
        ErrorResponse error = ErrorResponse.builder()
                .message(e.getMessage())
                .build();

        return ResponseEntity.status(HttpStatus.UNAUTHORIZED).body(error);   // ④
    }
}</code></pre>
<pre><code class="language-java">// UserService.java
UserResponseDTO response = userMapper.signIn(request)
        .orElseThrow(() -&gt; new LoginFailException(&quot;로그인 실패&quot;));   // ⑤</code></pre>
<p><strong>한 줄씩 정리하면</strong></p>
<ol>
<li><code>LoginFailException extends RuntimeException</code> — &quot;로그인 실패&quot;라는 상황을 표현하는 <strong>커스텀 예외 클래스</strong>를 직접 만들었다. 웹에서 발생하는 예외는 대부분 실행 시점(runtime)의 것이라 컴파일 예외로 만들 필요가 없어서 <code>RuntimeException</code>을 상속했다. 이런 예외 클래스들은 도메인별로(<code>commons/exception/users/</code> 등) 분리해서 관리하면, 회원가입 실패·게시글 조회 실패처럼 계속 늘어나도 정리가 된다.</li>
<li><code>@RestControllerAdvice</code> — 이 클래스를 <code>HandlerExceptionResolver</code> 체인에 <strong>등록</strong>한다. 예전에 배운 <code>catch(Exception e)</code>를 컨트롤러마다 반복해서 쓰는 대신, 예외 종류별로 한 번만 처리 방법을 정의해두면 모든 컨트롤러에 자동으로 적용된다.</li>
<li><code>@ExceptionHandler(LoginFailException.class)</code> — <code>LoginFailException</code>이 애플리케이션 어디서든 발생하면, <strong>우리가 직접 호출한 적 없는 이 메서드가 콜백</strong>으로 실행된다.</li>
<li><code>401 UNAUTHORIZED</code> — 로그인 실패는 &quot;인증되지 않음&quot;이라는 뜻이라 <code>401</code>로 응답한다. <code>ResponseEntity</code>에는 원래 헤더·바디·상태코드만 담기고 오류 메시지를 담을 자리가 없어서, <code>ErrorResponse</code>라는 별도 객체를 만들어 바디에 실었다.</li>
<li><code>orElseThrow(() -&gt; new LoginFailException(&quot;로그인 실패&quot;))</code> — 예전엔 그냥 <code>RuntimeException</code>을 던졌는데, 오늘 만든 <code>LoginFailException</code>으로 바꾸면서 <code>GlobalExceptionHandler</code>가 정확히 이 예외만 골라서 처리할 수 있게 됐다.</li>
</ol>
<p>⚠️ <strong><code>ErrorResponse</code>에 숨어있던 함정</strong>: <code>ErrorResponse</code>에는 <code>@Builder</code>와 <code>@Getter</code>만 있고 <code>@AllArgsConstructor</code>가 없는데도 <code>new ErrorResponse(...)</code> 형태의 생성자 호출이 동작하는 경우가 있다. <code>@Builder</code>가 자동으로 만들어주는 전체 필드 생성자가 <strong>패키지 접근 수준(default)</strong>으로 생성되기 때문에, 같은 패키지 안에서 호출할 때만 우연히 동작하는 것이다. 다른 패키지에서 그대로 호출하면 컴파일 에러가 난다 — 눈에 잘 안 띄는 함정이라 기록해둔다.</p>
<hr />
<h1 id="5-회원가입-유효성-검사--valid로-dto에-규칙-걸기">5. 회원가입 유효성 검사 — <code>@Valid</code>로 DTO에 규칙 걸기</h1>
<p>검증 없이 회원가입 폼을 빈 값으로 제출해보면, 화면은 정상 전환되고 서버 로그엔 필드가 전부 빈 문자열인 DTO가 찍히고 DB에도 그대로 저장됐다. <code>NOT NULL</code> 제약이 걸려 있어도 <strong><code>null</code>이 아니라 빈 문자열</strong>이 들어온 거라 막히지 않았다 — 예전에 정리한 &quot;NULL과 빈 문자열은 다른 상태&quot;라는 원칙이 그대로 재현된 상황이다.</p>
<p>검증은 <strong>두 단계</strong>로 나눠서 한다: 1차는 프론트에서 막아 불필요한 네트워크 요청 자체를 줄이고, 2차는 백엔드에서 한 번 더 검증해서 중간에 데이터가 조작되더라도 서비스까지 타지 않게 막는다.</p>
<pre><code class="language-java">public class UserRequestDTO {
    @Email(message = &quot;이메일 형식을 지켜주세요!&quot;)                    // ①
    private String email;

    @NotBlank(message = &quot;비밀번호는 필수입력 항목입니다.&quot;)             // ②
    @Size(min = 4, max = 20)
    private String password;

    @NotBlank(message = &quot;이름은 필수입력 항목입니다.&quot;)
    private String name;
}</code></pre>
<pre><code class="language-java">@PostMapping(&quot;/signUp&quot;)
public ResponseEntity&lt;?&gt; signUp(@Valid @RequestBody UserRequestDTO request,   // ③
                                 BindingResult bindingResult) {

    if (bindingResult.hasErrors()) {                                            // ④
        Map&lt;String, String&gt; errMap = new HashMap&lt;&gt;();
        bindingResult.getAllErrors().forEach(err -&gt; {
            FieldError field = (FieldError) err;
            errMap.put(field.getField(), err.getDefaultMessage());               // ⑤
        });
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(errMap);       // ⑥
    }

    int signUpFlag = userService.signUp(request);
    ...
}</code></pre>
<p><strong>한 줄씩 정리하면</strong></p>
<ol>
<li><code>@Email(message = &quot;...&quot;)</code> — 값이 이메일 형식이 아니면, 지정한 메시지로 검증 실패를 표시한다. import 경로가 <code>jakarta.validation</code>이라는 점도 눈여겨볼 만하다 — Spring 전용이 아니라 <strong>자바 표준(JSR)</strong>이라, 웹이 아닌 다른 곳에서도 쓸 수 있다.</li>
<li><code>@NotBlank</code> + <code>@Size(min=4, max=20)</code> — &quot;공백만 있으면 안 됨&quot; + &quot;길이가 4~20자여야 함&quot;을 <strong>동시에</strong> 검증한다. 한 필드에 여러 검증 어노테이션을 겹쳐 붙일 수 있다.</li>
<li><code>@Valid @RequestBody UserRequestDTO request</code> — <code>@Valid</code>가 있어야 DTO에 붙여둔 검증 어노테이션들이 <strong>실제로 동작</strong>한다. <code>BindingResult</code>는 반드시 검증 대상 매개변수(<code>request</code>) <strong>바로 뒤</strong>에 와야 한다는 규칙도 함께 배웠다.</li>
<li><code>bindingResult.hasErrors()</code> — 검증에 하나라도 실패한 게 있는지 확인한다. 오류가 있으면 여기서 <strong>바로 반환</strong>해서 서비스를 타지 않는다.</li>
<li><code>FieldError</code> — 각 에러가 &quot;어느 필드(<code>getField()</code>)에서 어떤 메시지(<code>getDefaultMessage()</code>)로 실패했는지&quot;를 담고 있다. <code>getAllErrors()</code>가 반환하는 <code>ObjectError</code> 목록을 <code>FieldError</code>로 캐스팅해야 이 정보를 꺼낼 수 있다.</li>
<li><code>400 BAD_REQUEST</code> + <code>errMap</code> — 검증 실패 시, 어느 필드가 왜 실패했는지 전부 담아서 응답한다.</li>
</ol>
<p>⚠️ <strong>겪은 시행착오 ① — 200으로 갔다가 다시 400으로</strong>: <code>bindingResult.hasErrors()</code>일 때 처음엔 <code>ResponseEntity.status(0)</code>(잘못된 값)으로 응답하다가 → <code>BAD_REQUEST</code>로 고치고 → axios가 <code>4xx</code>를 자동으로 <code>catch</code>로 보내버려서 &quot;에러 데이터를 못 꺼내나?&quot; 싶어 실험 삼아 <code>200 OK</code>로 바꿔봤다가 → 결국 다시 <code>BAD_REQUEST</code>로 되돌렸다. 4xx로 응답해도 <code>error.response.data</code>로 본문을 꺼낼 수 있다는 걸 확인한 뒤에는, 상태코드의 의미(실패는 400)도 지키고 메시지도 프론트에 전달하는 두 마리 토끼를 다 잡을 수 있었다.</p>
<p>⚠️ <strong>겪은 시행착오 ② — 코드는 맞는데 동작을 안 함(워크스페이스 캐시 문제)</strong>: 검증 코드를 다 작성했는데도 <code>bindingResult.hasErrors()</code>가 계속 <code>false</code>로 나오는 상황이 있었다. <code>@Valid</code>와 <code>@Validated</code>를 바꿔보고 어노테이션을 지웠다 다시 붙여봐도 그대로였는데, 원인은 코드가 아니라 <strong>VS Code의 Java 언어 서버 캐시</strong>가 꼬여있던 것이었다. <code>Java: Clean Java Language Server Workspace</code> 명령으로 캐시를 지우고 서버를 다시 띄우니 정상 동작했다. <strong>코드가 분명히 맞는데 동작하지 않고, 같은 코드인데 사람마다 결과가 다르면 캐시부터 의심</strong>해볼 만하다는 교훈으로 남겼다.</p>
<hr />
<h1 id="6-프론트-백-연동-③--검증-실패-메시지를-화면까지-이어붙이기">6. 프론트-백 연동 ③ — 검증 실패 메시지를 화면까지 이어붙이기</h1>
<pre><code class="language-js">// SignUpPage.jsx
.catch(error =&gt; {
    console.log(error.response.data);
    setErrors(error.response.data);   // ①
})</code></pre>
<pre><code class="language-jsx">{errors.name &amp;&amp; &lt;ErrorMessage&gt;{errors.name}&lt;/ErrorMessage&gt;}       // ②
{errors.email &amp;&amp; &lt;ErrorMessage&gt;{errors.email}&lt;/ErrorMessage&gt;}
{errors.password &amp;&amp; &lt;ErrorMessage&gt;{errors.password}&lt;/ErrorMessage&gt;}</code></pre>
<p><strong>한 줄씩 정리하면</strong></p>
<ol>
<li><code>error.response.data</code> — axios는 <code>400</code>처럼 실패 응답을 받으면 <code>.then</code>이 아니라 <code>.catch</code>로 흐름이 넘어간다. 그 <code>error.response.data</code> 안에 5번 섹션에서 백엔드가 만든 <code>errMap</code>(<code>{email: &quot;...&quot;, password: &quot;...&quot;}</code>)이 그대로 담겨 있어서, 그걸 그대로 <code>errors</code> state에 저장했다.</li>
<li><code>{errors.email &amp;&amp; &lt;ErrorMessage&gt;...}</code> — 각 입력창 아래에서, 해당 필드의 에러 메시지가 있을 때만(<code>&amp;&amp;</code>) 빨간 글씨로 보여준다.</li>
</ol>
<p>전체적으로 보면, <strong>5번 섹션에서 백엔드가 만든 <code>Map</code>의 key(<code>email</code>, <code>password</code>, <code>name</code>)와, 6번 섹션에서 프론트가 읽는 <code>errors.email</code>, <code>errors.password</code>, <code>errors.name</code>의 이름이 정확히 일치</strong>해야 이 흐름이 성립한다. 아래는 이 전체 과정을 정리한 것이다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/hssondev/post/3ab3608d-0e6c-413a-8c05-d64c48f95276/image.png" /></p>
<hr />
<h1 id="✨-새롭게-알게-된-점">✨ 새롭게 알게 된 점</h1>
<ul>
<li>Filter(웹 컴포넌트, 컨트롤러 앞)와 AOP(스프링 컴포넌트, 컨트롤러-서비스 사이)는 위치도 소속도 다르다 — &quot;프론트 컨트롤러 기준 앞이냐 뒤냐&quot;가 기준이다.</li>
<li>처리되지 않은 예외는 <code>DispatcherServlet</code>을 거쳐 <code>HandlerExceptionResolver</code> 체인까지 가고, 등록된 핸들러가 없으면 그제서야 <code>500</code>이 된다 — <code>@RestControllerAdvice</code>는 그 체인에 우리 핸들러를 등록하는 것이다.</li>
<li>ORM(MyBatis)은 매개변수를 객체 하나로만 받을 수 있어서, 값 두 개를 넘기려면 DTO나 <code>Map</code>으로 묶어야 한다. <code>Map</code> 인터페이스는 그대로 <code>parameterType</code>에 못 쓰고, 구체 클래스인 <code>HashMap</code>을 명시해야 한다.</li>
<li>URL 경로에 텍스트 데이터를 실을 땐 <code>encodeURIComponent</code>로 인코딩해야 특수문자·공백이 있어도 안전하다.</li>
<li>axios는 메서드마다 인자 순서가 달라서(<code>get(url, config)</code> vs <code>patch(url, data, config)</code>), 헤더 자리를 혼동하기 쉽다.</li>
<li><code>@Valid</code>와 <code>BindingResult</code>를 짝으로 쓰면, DTO에 붙인 검증 어노테이션들이 실제로 동작하고 그 결과를 코드로 다룰 수 있다.</li>
<li>코드가 분명히 맞는데 동작하지 않고 사람마다 결과가 다르면, 코드보다 워크스페이스 캐시 같은 환경 문제를 먼저 의심해볼 수 있다.</li>
<li>백엔드가 응답으로 보내는 key 이름과 프론트가 그 값을 읽는 이름이 정확히 일치해야 전체 흐름이 끊기지 않는다.</li>
</ul>
<hr />
<h1 id="🤔-헷갈렸던-점">🤔 헷갈렸던 점</h1>
<ul>
<li><strong><code>@RestControllerAdvice</code>가 정확히 어떤 범위까지 잡아주는지</strong>: &quot;여러 컨트롤러에 걸쳐&quot;라고는 배웠는데, 정말 프로젝트 안의 모든 컨트롤러를 다 잡아주는 건지, 설정이 더 필요한 부분이 있는 건지는 다음에 확인이 필요하다.</li>
<li><strong>Filter/Interceptor/AOP 세 개의 경계</strong>: 위치로 구분한다는 원칙은 이해했지만, 실제로 Filter를 직접 만들어보기 전까지는 감이 완전히 잡히지 않을 것 같다. 다음 주 시큐리티에서 다시 확인해야겠다.</li>
<li><strong><code>Map</code> 인터페이스 vs <code>HashMap</code> 구체 클래스</strong>: 왜 인터페이스로는 인스턴스를 못 만드는지 원리는 알겠는데, <code>parameterType</code>을 실제로 <code>Map</code>으로 바꿔서 <code>BeanCreationException</code>을 직접 재현해보진 못했다.</li>
</ul>
<hr />
<h1 id="다음에-할-일">다음에 할 일</h1>
<ul>
<li><input disabled="" type="checkbox" /> <code>parameterType</code>을 <code>java.util.Map</code>으로 바꿔 실행해보고 <code>BeanCreationException</code>을 직접 확인한 뒤 <code>HashMap</code>으로 되돌리기</li>
<li><input disabled="" type="checkbox" /> <code>LoginFailException</code> 외에 다른 예외(예: 회원가입 중복 이메일)도 커스텀 예외로 만들어서 <code>GlobalExceptionHandler</code>에 추가해보기</li>
<li><input disabled="" type="checkbox" /> <code>@Pattern</code>(정규표현식) 유효성 검사도 추가로 적용해보기 (주석으로 남겨둔 비밀번호 정책 정규식)</li>
<li><input disabled="" type="checkbox" /> 정규표현식 관련 참고 링크(<a href="https://adjh54.tistory.com/104">https://adjh54.tistory.com/104</a>) 읽어보기</li>
<li><input disabled="" type="checkbox" /> 동작이 이상할 때 <code>Java: Clean Java Language Server Workspace</code>로 캐시부터 지워보는 습관 들이기</li>
</ul>
<hr />
<h1 id="💡-오늘의-회고">💡 오늘의 회고</h1>
<p>오늘은 프론트와 백엔드 코드를 나란히 놓고 하나씩 짝을 맞춰보는 하루였다. 댓글 삭제 하나만 봐도, 프론트의 <code>response.status === 204</code>와 백엔드의 <code>ResponseEntity.status(HttpStatus.NO_CONTENT)</code>가 서로 정확히 맞아떨어져야 동작한다는 걸 눈으로 확인하니, 지금까지 &quot;따로따로 배운&quot; 프론트와 백엔드가 실제로는 숫자 하나, 이름 하나까지 맞춰야 하는 긴밀한 관계라는 게 실감났다.</p>
<p><code>500</code> 에러가 &quot;서버가 고장났다&quot;는 뜻이 아니라 &quot;이 예외를 처리할 핸들러가 체인에 등록되어 있지 않다&quot;는 뜻이라는 걸 알고 나니, 예외 처리라는 게 막연하게 무섭게 느껴지던 것에서 &quot;체인에 담당자를 등록해주기만 하면 되는 문제&quot;로 바뀌었다. <code>@Valid</code>로 유효성 검사를 배우면서, 백엔드가 만든 에러 <code>Map</code>의 key와 프론트가 읽는 <code>errors.필드명</code>이 정확히 같은 이름이어야 한다는 것도 인상 깊었다. 코드가 맞는데도 동작하지 않던 순간엔 워크스페이스 캐시를 의심해야 했던 것처럼, 오늘은 코드 바깥의 것들도 함께 살펴야 한다는 걸 배운 하루였다.</p>