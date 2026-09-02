<p>#AMINSPIRECAMP #K-DT #LGCNS #LGCNS6기</p>
<blockquote>
<p><strong>한 줄 요약</strong></p>
<p>지금까지 손으로 직접 만들었던 <code>BeanFactory</code>·<code>DI</code>를 Spring이 어떻게 자동화해주는지 개념으로 잡고, <code>@Mapper</code>/<code>MyBatis XML</code>로 회원 저장·조회·로그인 기능을 만들면서 TDD(given-when-then)로 테스트했고, 마지막엔 같은 인터페이스의 구현체 두 개 중 하나를 골라 주입하는 것까지 해봤다.</p>
</blockquote>
<hr />
<h1 id="📚-오늘-학습한-커리큘럼">📚 오늘 학습한 커리큘럼</h1>
<pre><code>Spring이 대신해주는 것 — Bean과 IoC
        ↓
@Component 계열 어노테이션 (Controller/Service/Repository/Mapper)
        ↓
MyBatis Mapper 구조 — interface + XML
        ↓
TDD(given-when-then)로 Mapper 테스트하기
        ↓
Optional을 반환하는 조회 메서드 (login)
        ↓
같은 인터페이스, 다른 구현체 두 개 — @Service 이름 + @Resource로 선택
        ↓
Controller에서 요청 파라미터를 DTO로 받기</code></pre><h1 id="📌-오늘의-학습-키워드">📌 오늘의 학습 키워드</h1>
<ul>
<li>IoC(제어의 역전), DI(의존성 주입), Bean</li>
<li><code>@Controller</code>, <code>@RestController</code>, <code>@Service</code>, <code>@Repository</code>, <code>@Mapper</code></li>
<li><code>@Autowired</code>, <code>@Inject</code>, <code>@Resource</code>, <code>@RequiredArgsConstructor</code>, <code>@Qualifier</code></li>
<li>MyBatis <code>#{}</code> 파라미터 바인딩, <code>resultType</code>/<code>parameterType</code></li>
<li><code>@SpringBootTest</code>, given-when-then</li>
</ul>
<hr />
<h1 id="0-개념-정리-키워드-사전">0. 개념 정리 (키워드 사전)</h1>
<blockquote>
<p><strong>한 줄 요약</strong></p>
<p>예전엔 우리가 직접 <code>BeanFactory</code>로 객체를 만들고 주입했는데, Spring에서는 그 일을 프레임워크가 대신 해준다 — 이걸 제어의 역전(IoC)이라 부르고, 그 결과로 필드에 객체가 자동으로 채워지는 걸 의존성 주입(DI)이라 부른다.</p>
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
<td><strong>Bean</strong></td>
<td>Spring 프레임워크가 대신 생성하고 관리해주는 객체</td>
<td><code>new</code>를 직접 안 해도 Spring이 만들어서 필요한 곳에 넣어줌</td>
</tr>
<tr>
<td><strong>IoC (Inversion of Control, 제어의 역전)</strong></td>
<td>객체 생성·관리의 주도권을 개발자가 아니라 프레임워크가 갖는 것</td>
<td>예전엔 우리가 만든 <code>BeanFactory</code>가 하던 일</td>
</tr>
<tr>
<td><strong>DI (Dependency Injection, 의존성 주입)</strong></td>
<td>필요한 객체(Bean)를 필드가 직접 만들지 않고 외부(Spring)에서 넣어주는 것</td>
<td><code>@Autowired</code> 등으로 표시</td>
</tr>
<tr>
<td><strong><code>@Component</code>와 그 계열</strong></td>
<td>&quot;이 클래스는 Spring이 관리하는 Bean으로 등록해라&quot;는 표시</td>
<td><code>@Controller</code>, <code>@Service</code>, <code>@Repository</code>, <code>@Mapper</code> 모두 이 계열</td>
</tr>
<tr>
<td><strong><code>@RestController</code></strong></td>
<td>액션을 수행하고 결과를 <strong>데이터(JSON)</strong>로 반환</td>
<td>화면(뷰)이 아니라 데이터를 돌려주는 API용</td>
</tr>
<tr>
<td><strong><code>@Controller</code></strong></td>
<td>액션을 수행하고 <strong>페이지(뷰)</strong>를 반환</td>
<td>전통적인 웹 페이지 방식(JSP 등)</td>
</tr>
<tr>
<td><strong>Dependency Lookup</strong></td>
<td><code>getBean()</code>처럼 필요할 때 직접 Bean을 찾아오는 방식</td>
<td>DI(자동 주입)와 대비되는 개념</td>
</tr>
<tr>
<td><strong>MyBatis <code>#{}</code></strong></td>
<td>SQL 안에서 자바 객체의 필드 값을 안전하게 바인딩하는 문법</td>
<td><code>?</code> 대신 써서 값이 어떤 필드에서 왔는지 명확히 표시됨</td>
</tr>
</tbody></table>
<h3 id="코드로-다시-보기">코드로 다시 보기</h3>
<pre><code class="language-java">@Mapper                    // MyBatis 인터페이스임을 표시
public interface UserMapper {
    public int save(UserRequestDTO request);
}</code></pre>
<pre><code class="language-xml">&lt;insert id=&quot;save&quot; parameterType=&quot;...UserRequestDTO&quot;&gt;
    INSERT INTO SPRING_USER_TBL(EMAIL, PASSWORD, NAME)
    VALUES(#{email}, #{password}, #{name})
&lt;/insert&gt;</code></pre>
<hr />
<h1 id="1-spring이-대신해주는-것--bean과-ioc">1. Spring이 대신해주는 것 — Bean과 IoC</h1>
<p>이전에 블로그 콘솔 앱에서 <code>BlogBeanFactory</code>를 직접 만들어서, 생성자 안에서 <code>new</code>로 객체를 하나씩 만들고 <code>map</code>에 등록해뒀었다. 오늘 배운 개념은, <strong>Spring이 그 <code>BeanFactory</code> 역할을 프레임워크 차원에서 대신 해준다</strong>는 것이다.</p>
<pre><code>new XXXX()  ← 우리가 직접 만들던 방식
     │
     ▼
Spring에게 객체 생성을 &quot;위임&quot;
     │
     ├─ .xml(schema)로 설정하는 방식(예전)
     └─ 어노테이션으로 표시하는 방식(요즘)</code></pre><p><strong>한 줄씩 정리하면</strong></p>
<ol>
<li><code>@Component</code>(또는 그 계열인 <code>@Controller</code>, <code>@Service</code>, <code>@Repository</code>, <code>@Mapper</code>)를 클래스 위에 붙인다.</li>
<li>Spring이 애플리케이션 실행 시 이 클래스를 찾아서 <strong>자동으로 인스턴스(Bean)를 만들어 등록</strong>한다.</li>
<li>그 Bean이 필요한 다른 클래스에서는 <code>@Autowired</code>만 붙이면, Spring이 알아서 해당 Bean을 찾아 필드에 넣어준다(DI).</li>
<li>예전엔 <code>factory.getBean(&quot;list.inspire&quot;)</code>로 직접 꺼내왔다면, 이제는 &quot;이 타입의 Bean을 원한다&quot;고 <strong>표시만 해두면 자동으로 채워진다</strong>는 게 핵심 차이다.</li>
</ol>
<hr />
<h1 id="2-component-계열--역할별로-이름이-다른-이유">2. <code>@Component</code> 계열 — 역할별로 이름이 다른 이유</h1>
<table>
<thead>
<tr>
<th>어노테이션</th>
<th>어느 계층에서 쓰나</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td><code>@RestController</code></td>
<td>View와 맞닿는 계층</td>
<td>요청을 받고, 결과를 JSON 데이터로 반환</td>
</tr>
<tr>
<td><code>@Controller</code></td>
<td>View와 맞닿는 계층</td>
<td>요청을 받고, 페이지(뷰)를 반환</td>
</tr>
<tr>
<td><code>@Service</code></td>
<td>비즈니스 로직 계층</td>
<td>실제 처리 로직 담당</td>
</tr>
<tr>
<td><code>@Repository</code></td>
<td>데이터 접근 계층</td>
<td>DB와 직접 소통</td>
</tr>
<tr>
<td><code>@Mapper</code></td>
<td>데이터 접근 계층(MyBatis 전용)</td>
<td>MyBatis가 SQL을 실행하는 인터페이스</td>
</tr>
</tbody></table>
<ul>
<li>다섯 개 다 결국 &quot;Spring이 관리하는 Bean으로 등록해라&quot;라는 뜻은 같지만(<code>@Component</code>의 사촌들), 이름을 다르게 붙여서 <strong>이 클래스가 어느 계층에서 무슨 역할을 하는지</strong> 코드만 봐도 알 수 있게 해준다. 예전에 정리했던 PL(Controller)-BL(Service)-PL(Repository/Mapper) 계층 구조가, 오늘은 계층마다 전용 어노테이션이 있다는 걸로 한 단계 더 구체화됐다.</li>
</ul>
<pre><code class="language-java">@RestController
@RequestMapping(&quot;/health&quot;)   // ①
public class HealthController {
    @GetMapping(&quot;path&quot;)       // ②
    public String getMethodName() {
        return &quot;OK&quot;;
    }
}</code></pre>
<p><strong>한 줄씩 정리하면</strong></p>
<ol>
<li><code>@RequestMapping(&quot;/health&quot;)</code> — 이 컨트롤러가 처리할 URL의 <strong>공통 앞부분</strong>을 지정한다.</li>
<li><code>@GetMapping(&quot;path&quot;)</code> — <code>GET</code> 방식 요청 중, 그 아래 이어지는 <strong>세부 경로</strong>를 지정한다.</li>
<li>그 결과 실제 요청 주소는 <code>http://localhost:8000/health/path</code>가 된다.</li>
</ol>
<hr />
<h1 id="3-mybatis-mapper-구조--interface와-xml이-짝을-이룬다">3. MyBatis Mapper 구조 — interface와 XML이 짝을 이룬다</h1>
<pre><code class="language-java">@Mapper                                        // ①
public interface UserMapper {
    public int save(UserRequestDTO request);   // ②
}</code></pre>
<pre><code class="language-xml">&lt;mapper namespace=&quot;com.example.testcase.features.users.repository.UserMapper&quot;&gt;  &lt;!-- ③ --&gt;
    &lt;insert id=&quot;save&quot;                                                             &lt;!-- ④ --&gt;
            parameterType=&quot;com.example.testcase.features.users.domain.dto.UserRequestDTO&quot;&gt;  &lt;!-- ⑤ --&gt;
        INSERT INTO SPRING_USER_TBL(EMAIL, PASSWORD, NAME)
        VALUES(#{email}, #{password}, #{name})   &lt;!-- ⑥ --&gt;
    &lt;/insert&gt;
&lt;/mapper&gt;</code></pre>
<p><strong>한 줄씩 정리하면</strong></p>
<ol>
<li><code>@Mapper</code> — 이 인터페이스가 MyBatis가 다루는 대상임을 표시한다.</li>
<li><code>public int save(...)</code> — <strong>메서드 선언만</strong> 있고 몸통(구현부)이 없다. 실제 SQL은 여기 없다.</li>
<li><code>&lt;mapper namespace=&quot;...&quot;&gt;</code> — 이 XML이 <strong>어느 인터페이스와 짝인지</strong> 패키지 경로까지 정확히 지정한다.</li>
<li><code>&lt;insert id=&quot;save&quot;&gt;</code> — 이 <code>id</code>가 인터페이스의 <strong>메서드 이름과 정확히 일치</strong>해야 서로 연결된다.</li>
<li><code>parameterType=&quot;...UserRequestDTO&quot;</code> — 이 SQL이 어떤 타입의 객체를 매개변수로 받는지 알려준다.</li>
<li><code>VALUES(#{email}, #{password}, #{name})</code> — <code>#{필드명}</code>이 매개변수로 받은 객체의 <strong>해당 필드 값</strong>을 그 자리에 채워 넣는다.</li>
</ol>
<p>⚠️ 처음엔 <code>VALUES(?,?,?)</code>로도 써봤는데, <code>?</code>는 &quot;몇 번째 값인지&quot;만 표시할 뿐 어떤 필드에서 온 값인지 코드만 봐서는 알 수 없어서 <code>#{email}</code>처럼 이름이 드러나는 방식으로 바꿨다. 참고로 <code>application-dev.yml</code>에는 이 XML 파일들의 위치도 <code>mybatis.mapper-locations: classpath:/mappers/**/*Mapper.xml</code>처럼 알려줘야 한다.</p>
<hr />
<h1 id="4-tdd로-mapper-테스트하기--given-when-then">4. TDD로 Mapper 테스트하기 — given, when, then</h1>
<pre><code class="language-java">@SpringBootTest                              // ①
public class UserApplicationTests {

    @Autowired                                 // ②
    private UserMapper userMapper;

    @Test                                       // ③
    public void signUp() {
        // given(데이터 준비)                    // ④
        UserRequestDTO request = UserRequestDTO.builder()
                .email(&quot;jslim9413@gmail.com&quot;)
                .password(&quot;1234&quot;)
                .name(&quot;임정섭&quot;)
                .build();

        // when(실행)                            // ⑤
        int flag = userMapper.save(request);

        // then(검증)                            // ⑥
        Assertions.assertEquals(1, flag);
    }
}</code></pre>
<p><strong>한 줄씩 정리하면</strong></p>
<ol>
<li><code>@SpringBootTest</code> — 이 클래스가 Spring Boot 애플리케이션 <strong>전체를 띄운 상태</strong>로 실행되는 통합 테스트임을 표시한다.</li>
<li><code>@Autowired</code> — 그래서 <code>UserMapper</code> Bean을 실제로 주입받아 테스트 안에서 쓸 수 있다.</li>
<li><code>@Test</code> — JUnit이 이 메서드를 테스트 대상으로 실행한다.</li>
<li><strong>given</strong> — 테스트에 필요한 데이터를 미리 준비하는 구간.</li>
<li><strong>when</strong> — 실제로 검증하고 싶은 동작을 실행하는 구간.</li>
<li><strong>then</strong> — 그 결과가 기대한 대로인지 확인하는 구간. <code>Assertions.assertEquals(1, flag)</code>로, <code>save()</code> 성공 시 반환값이 정말 <code>1</code>인지 코드가 자동으로 검증한다.</li>
</ol>
<p><code>System.out.println</code>으로 눈으로 확인하는 것과 달리, 이렇게 코드로 검증하면 값이 달라지는 순간 테스트가 바로 실패로 표시된다.</p>
<hr />
<h1 id="5-전체-목록-조회와-로그인--list와-optional-반환">5. 전체 목록 조회와 로그인 — <code>List</code>와 <code>Optional</code> 반환</h1>
<pre><code class="language-java">public interface UserMapper {
    public int save(UserRequestDTO request);
    public List&lt;UserResponseDTO&gt; findByAll();          // ①
    public Optional&lt;UserResponseDTO&gt; login(UserRequestDTO request);  // ②
}</code></pre>
<pre><code class="language-xml">&lt;select id=&quot;login&quot;
        parameterType=&quot;...UserRequestDTO&quot;
        resultType=&quot;...UserResponseDTO&quot;&gt;
    SELECT EMAIL, PASSWORD, NAME
    FROM SPRING_USER_TBL
    WHERE EMAIL = #{email} AND PASSWORD = #{password}
&lt;/select&gt;</code></pre>
<p><strong>한 줄씩 정리하면</strong></p>
<ol>
<li><code>findByAll()</code> — 결과가 <strong>여러 건</strong>일 수 있으니 <code>List&lt;UserResponseDTO&gt;</code>.</li>
<li><code>login()</code> — 일치하는 사용자가 <strong>있을 수도, 없을 수도</strong> 있으니 <code>Optional&lt;UserResponseDTO&gt;</code>. 결과가 없을 때 <code>null</code>을 그대로 반환하는 대신, 호출하는 쪽이 &quot;값이 없을 수 있다&quot;는 걸 타입만 보고도 알 수 있게 해준다.</li>
</ol>
<pre><code class="language-java">@Test
public void signIn() {
    UserRequestDTO request = UserRequestDTO.builder()
            .email(&quot;jslim9413@gmail.com&quot;).password(&quot;1234&quot;).build();

    Optional&lt;UserResponseDTO&gt; response = userMapper.login(request);   // ③

    Assertions.assertNotNull(response.get());                          // ④
    Assertions.assertEquals(&quot;jslim9413@gmail.com&quot;, response.get().getEmail());
}</code></pre>
<ol start="3">
<li><code>login(request)</code> — 반환된 <code>Optional</code>에 값이 있는지부터 확인해야 한다.</li>
<li><code>response.get()</code> — <code>Optional</code> 안의 실제 값을 꺼내서 검증한다. 주석으로 <code>response.orElseThrow(() -&gt; new RuntimeException(&quot;이메일 또는 패스워드가 불일치&quot;))</code>도 시도해본 흔적이 있는데, &quot;값이 없으면 예외를 던진다&quot;는 <code>Optional</code>의 또 다른 활용법을 미리 살펴본 것으로 보인다.</li>
</ol>
<hr />
<h1 id="6-같은-인터페이스-다른-구현체-두-개--어떤-걸-주입할지-정하기">6. 같은 인터페이스, 다른 구현체 두 개 — 어떤 걸 주입할지 정하기</h1>
<pre><code class="language-java">public interface UserService {
    public UserResponseDTO signIn(UserRequestDTO request);
}

@Service(value = &quot;plain&quot;)                                    // ①
public class UserPlainServiceImpl implements UserService {
    @Override
    public UserResponseDTO signIn(UserRequestDTO request) {
        System.out.println(&quot;debug &gt;&gt;&gt;&gt; User plain service signIn&quot;);
        return null;
    }
}

@Service(value = &quot;encryption&quot;)                                 // ②
public class UserEncrytionServiceImpl implements UserService {
    @Override
    public UserResponseDTO signIn(UserRequestDTO request) {
        System.out.println(&quot;debug &gt;&gt;&gt;&gt; User Encryption Service signIn&quot;);
        return null;
    }
}</code></pre>
<pre><code class="language-java">@Resource(name = &quot;plain&quot;)     // ③
private UserService userService;</code></pre>
<p><strong>한 줄씩 정리하면</strong></p>
<ol>
<li><code>@Service(value = &quot;plain&quot;)</code> — 평문 처리용 구현체에 <code>&quot;plain&quot;</code>이라는 이름표를 붙인다.</li>
<li><code>@Service(value = &quot;encryption&quot;)</code> — 암호화 처리용 구현체에는 <code>&quot;encryption&quot;</code>이라는 이름표를 붙인다.</li>
<li><code>@Resource(name = &quot;plain&quot;)</code> — 주입받는 쪽에서 <strong>이름으로 정확히 어떤 Bean인지</strong> 지정한다. <code>&quot;plain&quot;</code>이라는 이름표가 붙은 <code>UserPlainServiceImpl</code>이 확실하게 주입된다.</li>
</ol>
<p>⚠️ 같은 <code>UserService</code> 인터페이스의 구현체가 두 개 있는 상태에서 <code>@Autowired</code>(타입으로만 찾음)를 쓰면, Spring이 &quot;둘 중 어느 걸 넣어줘야 할지 몰라서&quot; 에러를 낸다(코드 주석의 &quot;container 모호성&quot;). 그래서 ①②로 이름표를 붙이고 ③으로 이름을 지정해 문제를 해결했다.</p>
<p><strong>오늘 배운 DI 방법들</strong></p>
<table>
<thead>
<tr>
<th>방법</th>
<th>기준</th>
</tr>
</thead>
<tbody><tr>
<td><code>@Autowired</code></td>
<td>타입 기준</td>
</tr>
<tr>
<td><code>@Inject</code></td>
<td>자바 표준, <code>@Autowired</code>와 유사</td>
</tr>
<tr>
<td><code>@Resource</code></td>
<td>이름 기준</td>
</tr>
<tr>
<td><code>@Qualifier</code></td>
<td>타입이 여러 개일 때 이름을 추가로 지정</td>
</tr>
<tr>
<td><code>@RequiredArgsConstructor</code></td>
<td>생성자 주입, 롬복이 생성자를 자동 생성</td>
</tr>
</tbody></table>
<hr />
<h1 id="7-controller에서-요청-파라미터를-dto로-받기">7. <code>Controller</code>에서 요청 파라미터를 DTO로 받기</h1>
<pre><code class="language-java">@RestController
@RequestMapping(&quot;/users&quot;)
public class UserController {

    @Resource(name = &quot;plain&quot;)
    private UserService userService;

    @GetMapping(&quot;/signIn&quot;)
    public ResponseEntity&lt;?&gt; signIn(UserRequestDTO request) {   // ①
        System.out.println(&quot;debug &gt;&gt;&gt;&gt;&gt; params &quot; + request);
        return null;                                              // ②
    }
}</code></pre>
<p><strong>한 줄씩 정리하면</strong></p>
<ol>
<li><code>signIn(UserRequestDTO request)</code> — 매개변수 타입을 DTO로 지정하면, Spring이 요청으로 들어온 여러 파라미터(<code>email</code>, <code>password</code> 등)를 <strong>자동으로 이 DTO 객체 하나로 묶어서</strong> 넣어준다. <code>@RequestParam</code>으로 하나하나 따로 안 받아도 된다.</li>
<li><code>return null</code> — 아직 <code>userService.signIn(request)</code>를 실제로 호출하지 않은 상태로 끝나 있다.</li>
</ol>
<p>⚠️ <strong>오늘 아직 안 끝난 부분</strong>: <code>userService</code> 필드까지는 주입해서 출력으로 확인했지만, 컨트롤러 → 서비스로 이어지는 <strong>진짜 호출</strong>은 다음 시간으로 남겨졌다.</p>
<p>참고로 <code>UserRequestDTO</code>에는 <code>@Setter</code>, <code>@NoArgsConstructor</code>, <code>@AllArgsConstructor</code>도 이 시점에 추가됐다 — Spring이 파라미터를 객체로 자동 매핑하려면, <strong>기본 생성자로 빈 객체를 만든 뒤 setter로 값을 하나씩 채워 넣는</strong> 방식이 필요하기 때문으로 보인다.</p>
<hr />
<h1 id="✨-새롭게-알게-된-점">✨ 새롭게 알게 된 점</h1>
<ul>
<li>Spring의 <code>@Component</code> 계열 어노테이션은 결국 우리가 손으로 만들었던 <code>BeanFactory</code>의 역할을 프레임워크가 대신 해주는 것이다.</li>
<li>MyBatis는 인터페이스(메서드 선언)와 XML(실제 SQL)을 <code>namespace</code>와 <code>id</code>로 짝지어 연결한다.</li>
<li>같은 인터페이스를 구현한 Bean이 여러 개면 <code>@Autowired</code>만으로는 모호해지고, <code>@Service(value=&quot;이름&quot;)</code> + <code>@Resource(name=&quot;이름&quot;)</code> 조합으로 어떤 걸 주입할지 정확히 지정해야 한다.</li>
<li>given-when-then으로 테스트를 구조화하면, 나중에 다시 봐도 테스트의 의도가 명확하게 읽힌다.</li>
<li>Controller의 메서드 매개변수를 DTO 타입으로 선언하면, 요청 파라미터가 자동으로 그 객체에 채워진다.</li>
</ul>
<hr />
<h1 id="🤔-헷갈렸던-점">🤔 헷갈렸던 점</h1>
<ul>
<li><strong><code>@Autowired</code>, <code>@Inject</code>, <code>@Resource</code>, <code>@Qualifier</code>의 차이</strong>: 오늘 이름만 쭉 나열되어서, 정확히 언제 뭘 골라 써야 하는지는 아직 감이 부족하다. 오늘 실습에서는 &quot;이름으로 지정해야 하는 상황&quot;에 <code>@Resource</code>를 쓴다는 것만 확실히 체감했다.</li>
<li><strong>Dependency Injection vs Dependency Lookup</strong>: DI는 자동으로 주입받는 거고, Lookup은 <code>getBean()</code>처럼 직접 찾아오는 거라는 개념 차이는 필기했는데, 실제 코드로 Lookup 방식을 써보진 못해서 아직 완전히 와닿지는 않는다.</li>
<li><strong>MyBatis <code>namespace</code>가 인터페이스 경로와 정확히 일치해야 하는 이유</strong>: 오타 하나만 나도 연결이 안 될 것 같은데, 왜 문자열로 이렇게 연결하는 방식을 쓰는지(타입으로 안전하게 연결하는 방법은 없는지)는 다음에 더 알아봐야겠다.</li>
</ul>
<hr />
<h1 id="다음에-할-일">다음에 할 일</h1>
<ul>
<li><input disabled="" type="checkbox" /> <code>UserController.signIn()</code>이 실제로 <code>userService.signIn(request)</code>를 호출하고 결과를 반환하도록 완성하기</li>
<li><input disabled="" type="checkbox" /> <code>userservice</code>(소문자 s로 시작하지만 관례에 안 맞는 이름) 필드명을 <code>userService</code>로 다시 정리하기</li>
<li><input disabled="" type="checkbox" /> <code>@Autowired</code>/<code>@Inject</code>/<code>@Resource</code>/<code>@Qualifier</code>/<code>@RequiredArgsConstructor</code> 각각을 실제 코드로 하나씩 다 써보고 차이 비교해보기</li>
<li><input disabled="" type="checkbox" /> 회원가입(signUp) API도 Controller까지 연결해서 실제 HTTP 요청으로 테스트해보기</li>
</ul>
<hr />
<h1 id="💡-오늘의-회고">💡 오늘의 회고</h1>
<p>오늘 가장 크게 다가온 건, 저번에 블로그 콘솔 앱에서 직접 만들었던 <code>BeanFactory</code>가 사실 Spring이 하는 일을 미리 손으로 흉내 내본 것이었다는 걸 깨달은 순간이었다. <code>@Service</code>, <code>@Mapper</code> 같은 어노테이션 하나만 붙이면 되는 걸 보면서, 그동안 짜본 싱글톤·팩토리·DI 코드가 왜 필요했는지 이해하고 있었던 게 오늘 개념을 훨씬 빨리 받아들이게 해준 것 같다.</p>
<p><code>@Service</code> 구현체가 두 개라서 주입이 모호해지는 상황을 직접 만들어보고, 이름으로 구분해서 해결하는 과정도 인상 깊었다. 지금까지 인터페이스는 &quot;구현체가 하나뿐이어도 유연성을 위해 미리 분리해두는 것&quot; 정도로만 생각했는데, 오늘처럼 진짜 구현체가 여러 개 존재하는 상황을 보니 왜 인터페이스로 규격을 잡아두는지가 훨씬 실감났다. 다음엔 오늘 미완성으로 남긴 Controller-Service 연결부터 마무리해야겠다.</p>