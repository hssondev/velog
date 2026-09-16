<blockquote>
<p><strong>한 줄 요약</strong></p>
<p>OpenAI API를 처음엔 OkHttp로 직접 호출해서 &quot;요청이 어떻게 생겼고 응답이 어떻게 오는지&quot; 원리를 손으로 확인한 다음, Spring AI의 <code>ChatClient</code>로 갈아타면서 훨씬 짧은 코드로 같은 결과를 만들고, 그 응답을 DTO로 파싱해서 실제 블로그 글쓰기 화면(키워드 → AI 본문 생성)까지 연결했다.</p>
</blockquote>
<hr />
<h1 id="📚-오늘-학습한-커리큘럼">📚 오늘 학습한 커리큘럼</h1>
<pre><code>프론트 리팩터링 (댓글 CRUD, 로그아웃 연동 정리)
        ↓
.env / build.gradle / application.yml에 OpenAI 설정 추가
        ↓
OkHttp로 OpenAI API 직접 호출해보기 (요청/응답 구조 이해)
        ↓
Spring AI ChatClient로 전환 (코드 단순화)
        ↓
프롬프트 설계 (페르소나 전략 + 출력 형식 지정) → DTO로 파싱
        ↓
BlogWritePage에 &quot;키워드 → AI 추천 본문&quot; 기능 연동
        ↓
JSON 응답 대신 순수 텍스트(.content())로 받도록 수정</code></pre><h1 id="📌-오늘의-학습-키워드">📌 오늘의 학습 키워드</h1>
<ul>
<li>OpenAI Chat Completions API 구조</li>
<li>OkHttp (Request/Response)</li>
<li>Spring AI <code>ChatClient</code></li>
<li>프롬프트 엔지니어링 (페르소나 전략)</li>
<li>ObjectMapper / JSON 파싱</li>
<li>DTO 설계 (<code>@JsonIgnoreProperties</code>, <code>@Builder</code>)</li>
<li>LangChain / RAG / MCP (개념만)</li>
</ul>
<hr />
<h1 id="0-개념-정리-키워드-사전">0. 개념 정리 (키워드 사전)</h1>
<blockquote>
<p><strong>한 줄 요약</strong></p>
<p>AI에게 뭔가 물어보는 것도 결국 &quot;HTTP 요청 보내고 JSON 응답 받기&quot;라는 점에서는 여느 REST API 호출과 다르지 않다. 다만 응답이 사람 말투의 텍스트라서, 그걸 우리 코드가 다루기 좋은 형태(DTO)로 &quot;다시 정리해서 받는 방법&quot;을 알려주는 게 오늘의 핵심이었다.</p>
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
<td><strong>Chat Completions API</strong></td>
<td>&quot;이런 대화를 줄 테니 다음 대답을 만들어줘&quot;라고 요청하는 OpenAI의 기본 API</td>
<td><code>/v1/chat/completions</code></td>
</tr>
<tr>
<td><strong>role (system/user)</strong></td>
<td>대화에서 &quot;누가 말한 것처럼 취급할지&quot; 구분하는 값</td>
<td>system: AI의 성격/규칙 지정, user: 실제 질문</td>
</tr>
<tr>
<td><strong>페르소나 전략</strong></td>
<td>AI에게 &quot;너는 OO 전문가야&quot;라고 역할을 부여해서 답변 품질/형식을 유도하는 프롬프트 기법</td>
<td>&quot;너는 날씨 전문가이자 미식가야&quot;</td>
</tr>
<tr>
<td><strong>ChatClient</strong></td>
<td>Spring AI가 제공하는, OpenAI 호출을 짧은 코드로 감싸주는 도구</td>
<td><code>.prompt().user(...).call().content()</code></td>
</tr>
<tr>
<td><strong>ObjectMapper</strong></td>
<td>JSON 문자열 ↔ 자바 객체를 서로 변환해주는 Jackson 라이브러리의 도구</td>
<td><code>readTree</code>, <code>readValue</code>, <code>writeValueAsString</code></td>
</tr>
<tr>
<td><strong>@JsonIgnoreProperties(ignoreUnknown = true)</strong></td>
<td>JSON에 우리 DTO에 없는 필드가 섞여 있어도 에러 내지 말고 무시하라는 설정</td>
<td>AI 응답에 예상 못 한 필드가 껴 있어도 안전</td>
</tr>
<tr>
<td><strong>RAG (Retrieval-Augmented Generation)</strong></td>
<td>AI가 답하기 전에 관련 자료를 먼저 검색해서 참고하게 만드는 방식</td>
<td>오늘은 개념만, 구현은 안 함</td>
</tr>
<tr>
<td><strong>MCP (Model Context Protocol)</strong></td>
<td>AI가 로컬/원격 도구나 데이터에 접근할 수 있게 해주는 표준 규격</td>
<td>오늘은 개념만 훑음</td>
</tr>
</tbody></table>
<h3 id="코드로-다시-보기">코드로 다시 보기</h3>
<pre><code class="language-java">// 오늘 배운 &quot;AI에게 물어보고 정리된 답 받기&quot; 흐름을 압축하면:
RecommandResponseDTO result = chatClient
    .prompt()
    .system(&quot;전처리된 json 형태로만 반환해줘.&quot;)   // 1. AI에게 규칙 부여 (페르소나+출력형식)
    .user(&quot;너는 날씨 전문가야... 날씨: %s, 위치: %s&quot;.formatted(weather, location)) // 2. 실제 질문
    .call()                                        // 3. 호출
    .entity(RecommandResponseDTO.class);            // 4. 응답을 곧바로 DTO로 변환</code></pre>
<p><img alt="" src="https://velog.velcdn.com/images/hssondev/post/02db5b17-8638-4b2c-9e4a-7aa631fd7fc9/image.png" /></p>
<p>오늘 만든 기능 전체를 그림으로 보면, 요청 하나가 브라우저 → 컨트롤러 → 서비스 → ChatClient → OpenAI를 거쳐 다시 브라우저로 돌아오는 왕복 구조라는 게 한눈에 보인다. 이 그림을 기준으로 아래 섹션들을 하나씩 따라가 본다.</p>
<hr />
<h1 id="1-프론트-리팩터링--댓글-crud와-로그아웃-정리">1. 프론트 리팩터링 — 댓글 CRUD와 로그아웃 정리</h1>
<p>본격적으로 AI 기능을 붙이기 전에, 먼저 기존 댓글 기능의 자잘한 불일치를 정리했다. <code>BlogCommentItem</code>에서 삭제/수정 버튼이 <code>comment.id</code>를 참조하고 있었는데, 실제 백엔드 응답 DTO의 필드명은 <code>commentId</code>였다. 지금까지는 우연히 동작했더라도, 이런 이름 불일치는 데이터가 조금만 바뀌어도 조용히 깨지는 버그의 원인이 되기 쉬워서 <code>comment.commentId</code>로 전부 통일했다.</p>
<pre><code class="language-jsx">&lt;Button title='삭제'
        onClick={(e) =&gt; handler(e, comment.commentId)} /&gt;
&lt;Button title={ isEdit ? '수정완료' : '수정' }
        onClick={(e) =&gt; updateMentionHandler(e)} /&gt;</code></pre>
<p>로그아웃 버튼도 지금까지는 그냥 메인 페이지로 이동만 시키는 가짜 버튼이었는데, 실제로 서버에 로그아웃 요청(<code>/users/signOut</code>)을 보내고 204 응답을 받으면 이동하도록 연동했다.</p>
<pre><code class="language-jsx">const logoutHandler = async (e) =&gt; {
  await api.post(`/users/signOut`, null, {
    headers: { Authorization: at ? at : &quot;&quot; }
  })
  .then(response =&gt; {
    if (response.status === 204) {
      moveUrl('/');
    }
  });
};</code></pre>
<p><code>BlogReadPage</code>도 댓글 등록/삭제/수정 API 호출부에 있던 json-server 시절 코드(주석 처리된 옛날 버전)들을 정리하고, 스프링부트 버전 엔드포인트(<code>/comments/insert</code>, <code>/comments/delete/{id}</code>, <code>/comments/update/{id}/{comment}</code>)로 통일했다.</p>
<hr />
<h1 id="2-openai-연동-준비--설정-파일부터">2. OpenAI 연동 준비 — 설정 파일부터</h1>
<p>AI 기능을 쓰려면 먼저 API 키와 사용할 모델을 환경변수로 관리해야 한다. <code>.env</code>에 키를 추가하고, <code>application-dev.yml</code>에서 그 값을 읽어오도록 연결했다.</p>
<pre><code class="language-yaml"># application-dev.yml
spring:
  ai:
    openai:
      api-key: ${OPEN_AI_KEY}
      chat:
        options:
          model: ${OPEN_AI_MODEL}</code></pre>
<p><code>build.gradle</code>에는 Spring AI 관련 의존성 3개(<code>spring-ai-starter-model-openai</code>, MCP server/client)와, 직접 HTTP 요청을 보내기 위한 <code>okhttp3</code>, 그리고 그 결과를 비동기로 다루기 위한 <code>webflux</code>를 추가했다. Spring Security를 이미 쓰고 있는 상태에서 Swagger 문서 라이브러리가 충돌을 일으켜서, <code>configurations.configureEach { exclude ... }</code>로 특정 모듈을 빌드에서 제외하는 것도 함께 처리했다.</p>
<p><code>SecurityConfig</code>의 화이트리스트에도 <code>/openai/**</code> 경로를 추가해서, AI 관련 API는 별도 인증 없이도 테스트할 수 있게 열어뒀다.</p>
<hr />
<h1 id="3-okhttp로-openai-api-직접-호출해보기">3. OkHttp로 OpenAI API 직접 호출해보기</h1>
<p>Spring AI의 편한 도구를 바로 쓰기 전에, 먼저 &quot;OpenAI API가 실제로 어떤 형태의 요청과 응답을 주고받는지&quot; 원리를 이해하기 위해 OkHttp로 직접 REST 호출을 만들어봤다.</p>
<p>OpenAI의 Chat Completions API는 결국 이런 형태의 JSON을 요구한다.</p>
<pre><code class="language-json">{
  &quot;model&quot;: &quot;gpt-4o-mini&quot;,
  &quot;messages&quot;: [
    {&quot;role&quot;: &quot;user&quot;, &quot;content&quot;: &quot;...&quot;},
    {&quot;role&quot;: &quot;system&quot;, &quot;content&quot;: &quot;전처리된 json 형태로만 반환해줘&quot;}
  ]
}</code></pre>
<p>이걸 자바 코드로 만들면 이렇게 된다.</p>
<pre><code class="language-java">Map&lt;String, Object&gt; messages = new HashMap&lt;&gt;();
messages.put(&quot;model&quot;, model);

Map&lt;String, Object&gt; user = new HashMap&lt;&gt;();
user.put(&quot;role&quot;, &quot;user&quot;);
user.put(&quot;content&quot;, prompt);

Map&lt;String, Object&gt; system = new HashMap&lt;&gt;();
system.put(&quot;role&quot;, &quot;system&quot;);
system.put(&quot;content&quot;, &quot;전처리된 json 형태로만 반환해줘&quot;);

messages.put(&quot;messages&quot;, List.of(user, system));

String requestJson = objectMapper.writeValueAsString(messages); // Map → JSON 문자열</code></pre>
<ul>
<li><code>Map</code>으로 구조를 먼저 만들고, <code>objectMapper.writeValueAsString(...)</code>으로 그 Map을 실제 JSON 문자열로 바꾼다. 이게 필요한 이유는, HTTP 요청 본문(body)에는 결국 &quot;문자열&quot;을 실어보내야 하기 때문이다.</li>
</ul>
<p>이 JSON 문자열을 실제 요청에 담아 보내는 부분은 이렇다.</p>
<pre><code class="language-java">Request request = new Request.Builder()
        .url(endPoint)
        .header(&quot;Authorization&quot;, &quot;Bearer &quot; + key)
        .header(&quot;Content-Type&quot;, &quot;application/json&quot;)
        .post(RequestBody.create(requestJson, MediaType.parse(&quot;application/json&quot;)))
        .build();

Response response = okHttpClient.newCall(request).execute();</code></pre>
<p>응답이 오면, 그 응답 몸체(body) 자체도 문자열이라서 다시 <code>ObjectMapper</code>로 필요한 부분만 뽑아냈다.</p>
<pre><code class="language-java">String responseJson = response.body().string();
JsonNode node = objectMapper.readTree(responseJson);
String content = node.at(&quot;/choices/0/message/content&quot;).asText();</code></pre>
<p><code>node.at(&quot;/choices/0/message/content&quot;)</code>처럼 경로를 지정해서 값을 꺼내는 방식(JSON Pointer)을 처음 써봤는데, OpenAI 응답 구조가 <code>choices</code> 배열 안에 <code>message.content</code>가 들어있는 형태라서 이렇게 깊이 파고들어가야 실제 답변 텍스트를 얻을 수 있었다.</p>
<p>여기까지 만들어서 실제로 호출해보니 응답이 정상적으로 왔다.</p>
<pre><code>debug &gt;&gt;&gt;&gt; response
Response{protocol=h2, code=200, ...}
{
    &quot;weather&quot;: &quot;청명한 날씨&quot;,
    &quot;location&quot;: &quot;강남&quot;,
    &quot;restaurants&quot;: [
        {&quot;name&quot;: &quot;한남동 국수집&quot;, &quot;category&quot;: &quot;국수&quot;, &quot;reason&quot;: &quot;...&quot;},
        ...
    ]
}</code></pre><p>이 시점에 중요한 걸 하나 확인했는데, 이 응답은 눈으로 보기엔 JSON처럼 생겼지만 실제로는 그냥 &quot;문자열&quot;이라는 점이다. AI는 JSON 형식을 흉내 낸 텍스트를 뱉을 뿐이지, 진짜 JSON 객체를 반환하는 게 아니다. 그래서 이 문자열을 다시 우리가 원하는 자바 객체로 변환하는 과정(파싱)이 필요하다.</p>
<pre><code class="language-java">RecommandResponseDTO result = objectMapper.readValue(exr, RecommandResponseDTO.class);</code></pre>
<p><img alt="" src="https://velog.velcdn.com/images/hssondev/post/67aafba9-eb5b-47de-b50f-9e9c395515ca/image.png" /></p>
<p>요청은 <code>system</code>/<code>user</code> 두 역할로 구성해서 보내고, 응답은 <code>choices[0].message.content</code>까지 몇 단계 더 들어가야 실제 답변 텍스트가 나온다는 걸 그림으로 정리하면 이렇다.</p>
<hr />
<h1 id="4-spring-ai의-chatclient로-갈아타기">4. Spring AI의 ChatClient로 갈아타기</h1>
<p>OkHttp로 직접 호출하는 방식은 원리를 이해하기엔 좋았지만, 요청 Map 만들기 → JSON 문자열 변환 → HTTP 요청 조립 → 응답 파싱까지 코드가 꽤 길었다. Spring AI가 제공하는 <code>ChatClient</code>를 쓰면 이 과정을 훨씬 짧게 줄일 수 있다는 걸 알게 되어 전환했다.</p>
<pre><code class="language-java">@Configuration
public class OpenAiConfig {
    @Bean
    public ChatClient chatClient(ChatClient.Builder builder) {
        return builder.build();
    }
}</code></pre>
<ul>
<li><code>ChatClient.Builder</code>는 Spring AI가 <code>application-dev.yml</code>에 설정해둔 API 키/모델 정보를 이용해서 자동으로 만들어주는 빌더다. 우리는 그냥 <code>.build()</code>만 호출해서 빈으로 등록하면 된다.</li>
</ul>
<pre><code class="language-java">public RecommandResponseDTO recommand(String weather, String location) {
    RecommandResponseDTO result = chatClient
        .prompt()
        .system(&quot;전처리된 json 형태로만 반환해줘.&quot;)
        .user(&quot;&quot;&quot;
            너는 날씨 전문가이고 맛있는 음식을 즐겨하는 인공지능전문가야.
            &lt;조건&gt;
                - 날씨 : &quot;%s&quot;
                - 위치 : &quot;%s&quot;
            &lt;/조건&gt;
            ...
            &quot;&quot;&quot;.formatted(weather, location))
        .call()
        .entity(RecommandResponseDTO.class);
    return result;
}</code></pre>
<p>OkHttp 버전과 비교하면 확 줄어든 걸 볼 수 있다. <code>.prompt()</code>로 대화를 시작하고, <code>.system(...)</code>으로 AI의 규칙(페르소나+출력 형식)을 정하고, <code>.user(...)</code>로 실제 질문을 넣은 다음, <code>.call().entity(DTO.class)</code> 한 줄이면 &quot;요청 보내기 → 응답 파싱하기&quot;까지 알아서 처리해준다. Map 만들기, JSON 직렬화, HTTP 요청 조립, JsonNode 파싱까지 전부 Spring AI 내부에서 대신 해주는 셈이다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/hssondev/post/d2788166-4c21-43ba-9080-0406b969bf1b/image.png" /></p>
<p>왼쪽(OkHttp)의 여섯 단계가 오른쪽(ChatClient)에서는 메서드 체인 한 번으로 압축된다는 걸 그림으로 나란히 놓고 보면 왜 전환했는지가 명확해진다.</p>
<hr />
<h1 id="5-프롬프트-설계--페르소나-전략과-출력-형식-강제하기">5. 프롬프트 설계 — 페르소나 전략과 출력 형식 강제하기</h1>
<p>AI에게 원하는 형태의 답을 받으려면, 그냥 질문만 던지는 게 아니라 &quot;역할&quot;과 &quot;출력 형식&quot;을 같이 알려줘야 한다는 걸 오늘 프롬프트를 몇 번 고쳐보면서 체감했다.</p>
<pre><code class="language-java">.user(&quot;&quot;&quot;
    너는 날씨 전문가이고 맛있는 음식을 즐겨하는 인공지능전문가야.
    현재날씨에 따른 지역맛집을 추천해줘
    &lt;조건&gt;
        - 날씨 : &quot;%s&quot;
        - 위치 : &quot;%s&quot;
    &lt;/조건&gt;
    &lt;출력예시&gt;
    {
        &quot;weather&quot;  : &quot;날씨&quot;,
        &quot;location&quot; : &quot;위치&quot;,
        &quot;restaurants&quot; : [
             {&quot;name&quot; : &quot;음식점명&quot;, &quot;category&quot; : &quot;분류&quot;, &quot;reason&quot; : &quot;추천이유&quot;}
        ]
    }
    &lt;/출력예시&gt;
    &quot;&quot;&quot;.formatted(weather, location))</code></pre>
<ul>
<li>&quot;너는 OO 전문가야&quot;라고 역할을 부여하는 걸 <strong>페르소나 전략</strong>이라고 부른다. 이렇게 하면 AI가 그 역할에 맞는 말투와 전문성으로 답을 생성하는 경향이 커진다.</li>
<li><code>&lt;조건&gt;</code>, <code>&lt;출력예시&gt;</code> 같은 태그로 감싸서 &quot;이 형식 그대로 답해라&quot;라고 강제하는 것도 중요한 포인트였다. 실제로 처음엔 이 출력 형식 지정 없이 질문만 던졌더니 답변이 자유 서술형으로 왔었는데, 출력 예시를 JSON 구조로 명시하고 나서야 우리가 파싱하기 좋은 형태로 답이 왔다.</li>
</ul>
<p>이 원칙을 그대로 재사용해서 <code>QuizResponseDTO</code>용 퀴즈 생성 프롬프트도 같은 패턴으로 만들었다. <code>system</code>에는 &quot;무조건 json으로만 반환해줘&quot; 같은 전역 규칙을, <code>user</code>에는 &quot;10문제를 만들어줘, 이 형식으로&quot; 같은 구체적 요구사항을 나눠서 넣는 방식이다.</p>
<p>DTO를 설계할 때는 AI 응답에 우리가 예상 못 한 필드가 섞여 들어올 가능성을 대비해서 <code>@JsonIgnoreProperties(ignoreUnknown = true)</code>를 꼭 붙였다.</p>
<pre><code class="language-java">@Builder
@Getter
@NoArgsConstructor
@AllArgsConstructor
@JsonIgnoreProperties(ignoreUnknown = true)
public class RecommandResponseDTO {
    private String weather;
    private String location;
    private List&lt;RestaurantDTO&gt; restaurants;

    @Builder
    @Getter
    @NoArgsConstructor
    @AllArgsConstructor
    @JsonIgnoreProperties(ignoreUnknown = true)
    public static class RestaurantDTO {
        private String name, category, reason;
    }
}</code></pre>
<p>여기서 한 가지 실수를 겪었는데, 처음엔 <code>restaurant</code>(단수)로 필드명을 지었다가, AI가 실제로 보내주는 JSON 키는 <code>restaurants</code>(복수)라서 매핑이 안 됐다. AI가 응답을 만들 때 참고하는 건 결국 우리가 프롬프트의 <code>&lt;출력예시&gt;</code>에 적어준 그 키 이름이기 때문에, DTO 필드명도 프롬프트에 적은 키 이름과 정확히 맞춰야 한다는 걸 알게 됐다. <code>Lombok</code>의 <code>@NoArgsConstructor</code>/<code>@AllArgsConstructor</code>도 빼먹으면 Jackson이 JSON을 객체로 변환할 때 생성자를 못 찾아 실패하기 때문에 반드시 같이 붙여야 했다.</p>
<hr />
<h1 id="6-블로그-작성-화면에-ai-키워드-기능-연동하기">6. 블로그 작성 화면에 AI 키워드 기능 연동하기</h1>
<p>이제 실제 화면에 이 기능을 붙였다. <code>BlogWritePage</code>에 카테고리 칩(선택 버튼), 키워드 입력창, &quot;키워드 전송&quot; 버튼을 추가했다.</p>
<pre><code class="language-jsx">const CATEGORIES = [&quot;개발&quot;, &quot;생활&quot;, &quot;취미&quot;, &quot;일상&quot;];

const keywordHandler = async () =&gt; {
    await api.post('/blogs/ai/agent', { keyword, category }, {
        headers: { Authorization: at ? at : &quot;&quot; }
    })
    .then(response =&gt; {
        if (response.status === 201) {
            setContent(response.data);
        }
    });
};</code></pre>
<p>여기서 처음엔 AI 응답을 받은 뒤 <code>moveUrl('/blogs/index')</code>로 목록 페이지로 바로 이동시켰는데, 생각해보니 사용자가 AI가 만들어준 초안을 확인·수정한 다음 직접 &quot;글 작성하기&quot; 버튼을 눌러야 자연스러운 흐름이다. 그래서 이동시키는 대신 <code>setContent(response.data)</code>로 본문(content) textarea에 AI가 생성한 글을 바로 채워넣도록 수정했다.</p>
<p>카테고리 칩 UI는 <code>styled-components</code>로 만들었는데, 현재 선택된 카테고리인지 여부를 prop으로 넘겨서 스타일을 다르게 주는 방식을 썼다.</p>
<pre><code class="language-jsx">const CategoryChip = styled.button`
    border: 1.5px solid ${(props) =&gt; (props.$active ? &quot;#6366f1&quot; : &quot;#e5e7eb&quot;)};
    background: ${(props) =&gt; (props.$active ? &quot;#6366f1&quot; : &quot;#ffffff&quot;)};
    color: ${(props) =&gt; (props.$active ? &quot;#ffffff&quot; : &quot;#4b5563&quot;)};
`;

{CATEGORIES.map((cat, idx) =&gt; (
    &lt;CategoryChip key={idx}
                  $active={category === cat}
                  onClick={() =&gt; setCategory(cat)}&gt;
        {cat}
    &lt;/CategoryChip&gt;
))}</code></pre>
<p><code>$active</code>처럼 이름 앞에 <code>$</code>를 붙인 prop은 <code>styled-components</code>가 실제 HTML 태그에는 넘기지 않고 스타일 계산용으로만 쓰는 규칙이다. 이렇게 하면 &quot;지금 선택된 칩만 보라색으로 강조&quot;하는 걸 별도의 클래스 토글 없이 상태값 하나로 처리할 수 있었다.</p>
<hr />
<h1 id="7-블로그-본문-자동-생성--json이-아니라-순수-텍스트로">7. 블로그 본문 자동 생성 — JSON이 아니라 순수 텍스트로</h1>
<p><code>BlogController</code>에 <code>/ai/agent</code> 엔드포인트를 추가하고, <code>BlogService.contentGenerate()</code>에서 카테고리와 키워드를 받아 AI에게 블로그 본문을 요청하도록 만들었다.</p>
<pre><code class="language-java">@PostMapping(&quot;/ai/agent&quot;)
public ResponseEntity&lt;?&gt; agent(@RequestBody Map&lt;String, Object&gt; map) {
    return ResponseEntity
            .status(HttpStatus.CREATED)
            .body(blogService.contentGenerate(map));
}</code></pre>
<pre><code class="language-java">public String contentGenerate(Map&lt;String, Object&gt; map) {
    String content = chatClient
        .prompt()
        .system(&quot;JSON이나 특수기호(중괄호, 따옴표) 없이, 순수한 블로그 본문 텍스트로만 답해줘.&quot;)
        .user(&quot;&quot;&quot;
                넌 국문학과박사 수료한 블로그작성 전문가야.
                주어진 카테고리와 키워드로 차분한 톤의 블로그를 작성해줘.
                글자수는 200자 이내로 작성해줘.
                &lt;조건&gt;
                    - 카테고리   : &quot;%s&quot;
                    - 키워드     : &quot;%s&quot;
                &lt;/조건&gt;
            &quot;&quot;&quot;.formatted((String) map.get(&quot;category&quot;), (String) map.get(&quot;keyword&quot;)))
        .call()
        .content();

    return content;
}</code></pre>
<p>여기서 흥미로운 시행착오가 있었다. 앞서 5번 섹션에서는 &quot;JSON 형태로만 반환해줘&quot;라고 요청해서 구조화된 DTO로 파싱하는 방식이 잘 맞았는데, 블로그 본문은 애초에 &quot;사람이 읽을 문장&quot;이 결과물이지 구조화된 데이터가 아니다. 그래서 처음엔 습관적으로 <code>.entity(String.class)</code>(JSON을 String 타입으로 파싱 시도)를 썼다가, 이건 애초에 JSON이 아닌 순수 텍스트를 억지로 JSON 파서에 넣으려는 것이라 맞지 않는다는 걸 깨닫고 <code>.content()</code>로 바꿨다. <code>.content()</code>는 파싱 없이 AI가 준 텍스트 응답을 그대로 문자열로 돌려주는 메서드다. 그리고 system 프롬프트도 &quot;json 형태로 반환해줘&quot;에서 &quot;JSON이나 특수기호 없이 순수 텍스트로만 답해줘&quot;로 정반대로 바꿔서, 목적에 맞는 응답 형식을 AI에게 명확히 지정해줬다.</p>
<p><strong>정리하면:</strong> 구조화된 데이터(추천 목록, 퀴즈 등)가 필요할 땐 &quot;JSON으로 달라&quot;고 요청하고 <code>.entity(DTO.class)</code>로 파싱하지만, 사람이 읽을 자연어 글이 필요할 땐 &quot;순수 텍스트로 달라&quot;고 요청하고 <code>.content()</code>로 그대로 받는다 — 결과물의 성격에 따라 요청 방식과 받는 방식을 다르게 가져가야 한다는 게 오늘의 핵심 교훈이었다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/hssondev/post/695766e5-c50a-4e92-8a9f-b22b9e81c824/image.png" /></p>
<p>같은 AI 응답이라도 &quot;구조화된 데이터가 필요한가, 자연어 문장이 필요한가&quot;에 따라 <code>entity()</code>와 <code>content()</code> 중 어느 쪽으로 받을지가 갈린다는 걸 그림으로 정리하면 이렇다.</p>
<hr />
<h1 id="✨-새롭게-알게-된-점">✨ 새롭게 알게 된 점</h1>
<ul>
<li>AI의 JSON 형태 응답은 진짜 JSON 객체가 아니라 &quot;JSON처럼 생긴 문자열&quot;일 뿐이라서, 반드시 <code>ObjectMapper</code>로 한 번 더 파싱하는 과정이 필요하다는 것.</li>
<li>DTO 필드명은 프롬프트의 <code>&lt;출력예시&gt;</code>에 적은 JSON 키 이름과 정확히 일치해야 파싱이 성공한다는 것 (<code>restaurant</code> vs <code>restaurants</code> 오타로 실패를 겪어봄).</li>
<li>OkHttp로 직접 호출하는 방식과 Spring AI <code>ChatClient</code>는 결국 같은 일(요청 조립 → 전송 → 응답 파싱)을 하지만, <code>ChatClient</code>는 그 반복 작업을 라이브러리가 대신 처리해준다는 것.</li>
<li>원하는 결과물이 &quot;구조화된 데이터&quot;냐 &quot;자연어 문장&quot;이냐에 따라 프롬프트의 요청 형식(JSON vs 순수 텍스트)과 받는 방식(<code>.entity()</code> vs <code>.content()</code>)을 다르게 써야 한다는 것.</li>
</ul>
<h1 id="🤔-헷갈렸던-점">🤔 헷갈렸던 점</h1>
<ul>
<li><code>.entity(String.class)</code>와 <code>.content()</code>의 차이 — 전자는 &quot;JSON 문자열을 String 타입으로 파싱&quot;하려는 시도이고, 후자는 애초에 파싱 없이 텍스트를 그대로 받는 것이라는 차이를 실제로 써보고서야 구분이 됐다.</li>
<li><code>node.at(&quot;/choices/0/message/content&quot;)</code>처럼 JSON 경로를 지정해서 값을 꺼내는 방식(JSON Pointer) — OpenAI 응답 구조가 몇 단계 중첩되어 있어서 처음엔 어디를 봐야 할지 헷갈렸다.</li>
<li>카테고리 칩의 <code>$active</code> prop처럼, <code>styled-components</code>에서 <code>$</code>로 시작하는 prop은 실제 DOM에는 전달되지 않는 스타일 전용 prop이라는 규칙.</li>
</ul>
<h1 id="다음에-할-일">다음에 할 일</h1>
<ul>
<li><input disabled="" type="checkbox" /> MCP(Model Context Protocol)가 실제로 어떻게 로컬 도구를 AI에 연결하는지 간단한 예제로 직접 실습해보기</li>
<li><input disabled="" type="checkbox" /> RAG 개념을 적용해서, 우리 블로그 데이터를 참고자료로 AI에게 넘겨주는 기능도 만들어보기</li>
<li><input disabled="" type="checkbox" /> <code>QuizResponseDTO</code> 기반 퀴즈 생성 기능도 프론트와 연동해서 실제로 문제 풀이 화면까지 완성하기</li>
<li><input disabled="" type="checkbox" /> AI 응답 실패(네트워크 오류, 파싱 실패) 시 사용자에게 보여줄 예외 처리 추가하기</li>
</ul>
<hr />
<h1 id="💡-오늘의-회고">💡 오늘의 회고</h1>
<p>오늘은 &quot;AI를 서버에 붙인다&quot;는 게 막연하게 느껴졌는데, 직접 OkHttp로 로우레벨 호출을 만들어보고 나서야 실체가 보였다. 결국 AI 호출도 평범한 REST API 호출과 다르지 않고, 다만 응답이 사람이 쓴 것 같은 텍스트라서 그걸 다시 우리 코드가 쓰기 좋은 구조(JSON → DTO)로 변환하는 한 단계가 더 필요할 뿐이었다. Spring AI의 <code>ChatClient</code>를 나중에 써보면서 &quot;아, 이게 방금 내가 손으로 짠 그 과정을 대신해주는 거구나&quot;라고 바로 연결이 돼서 이해가 훨씬 잘 됐다. 프롬프트에 역할(페르소나)과 출력 형식을 명확히 적어줄수록 원하는 답을 받기 쉬워진다는 것도 직접 실패해보고 나서 체감한 부분이다. 다음엔 오늘 건드려본 MCP와 RAG 개념까지 실습으로 이어서, &quot;AI가 우리 데이터를 참고해서 답하는&quot; 흐름까지 만들어봐야겠다.</p>