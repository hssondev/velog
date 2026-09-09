<blockquote>
<p><strong>한 줄 요약</strong></p>
<p>SQL을 직접 쓰던 MyBatis에서 벗어나, <code>JpaRepository</code>만 상속하면 CRUD가 자동으로 생기는 JPA로 넘어갔다. Entity-DTO 변환, <code>cascade</code>/<code>orphanRemoval</code>/<code>fetch</code> 연관관계 옵션을 배우고, 실제로 <code>LazyInitializationException</code>을 겪으며 &quot;영속성&quot;이라는 개념이 왜 필요한지 체감했다.</p>
</blockquote>
<hr />
<h1 id="📚-오늘-학습한-커리큘럼">📚 오늘 학습한 커리큘럼</h1>
<pre><code>MyBatis vs JPA — 큰 그림 비교
        ↓
JpaRepository&lt;Entity, ID&gt; — CRUD가 자동으로 생기는 이유
        ↓
Entity ↔ DTO 변환 — 정적 팩토리 메서드 패턴
        ↓
연관관계 매핑 — @ManyToOne(fetch = LAZY/EAGER)
        ↓
cascade, orphanRemoval — 부모-자식 생명주기
        ↓
⚠️ LazyInitializationException — 영속성 컨텍스트란?
        ↓
(예고) 필터로 토큰 만들기 — 인증/인가</code></pre><h1 id="📌-오늘의-학습-키워드">📌 오늘의 학습 키워드</h1>
<ul>
<li><code>JpaRepository&lt;Entity, ID&gt;</code></li>
<li>Entity-DTO 변환(정적 팩토리 메서드)</li>
<li><code>@ManyToOne</code>, <code>@OneToMany</code>, <code>fetch = FetchType.LAZY/EAGER</code></li>
<li><code>cascade</code>, <code>orphanRemoval</code></li>
<li>영속성 컨텍스트, <code>LazyInitializationException</code></li>
<li>인증(Authentication) / 인가(Authorization)</li>
</ul>
<hr />
<h1 id="0-개념-정리-키워드-사전">0. 개념 정리 (키워드 사전)</h1>
<blockquote>
<p><strong>한 줄 요약</strong></p>
<p>JPA는 &quot;테이블을 다루는 코드(SQL)를 직접 쓰지 않고, 객체(Entity)를 다루면 알아서 테이블에 반영해주는&quot; 기술이다 — 그 대신 &quot;언제 실제로 DB에서 값을 가져올지&quot;를 우리가 이해하고 있어야 오늘 겪은 에러 같은 걸 피할 수 있다.</p>
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
<td><strong><code>JpaRepository&lt;T, ID&gt;</code></strong></td>
<td>기본적인 CRUD 메서드(<code>save</code>, <code>findById</code>, <code>delete</code> 등)를 자동으로 만들어주는 인터페이스</td>
<td><code>T</code>=엔티티 타입, <code>ID</code>=그 엔티티의 기본키 타입</td>
</tr>
<tr>
<td><strong>Entity</strong></td>
<td>DB 테이블과 실제로 매핑되는 객체</td>
<td><code>@Entity</code> 어노테이션이 붙음</td>
</tr>
<tr>
<td><strong>정적 팩토리 메서드</strong></td>
<td><code>new</code> 대신, 이름 있는 static 메서드로 객체를 만들어내는 패턴</td>
<td><code>UserResponseDTO.from(entity)</code></td>
</tr>
<tr>
<td><strong><code>@ManyToOne</code>/<code>@OneToMany</code></strong></td>
<td>엔티티 사이의 N:1, 1:N 관계를 코드로 표현하는 어노테이션</td>
<td><code>@JoinColumn</code>으로 어떤 컬럼을 기준으로 연결할지 지정</td>
</tr>
<tr>
<td><strong><code>fetch</code> (LAZY/EAGER)</strong></td>
<td>연관된 엔티티를 <strong>언제</strong> 가져올지 결정</td>
<td>LAZY=필요할 때 나중에, EAGER=지금 즉시</td>
</tr>
<tr>
<td><strong><code>cascade</code></strong></td>
<td>부모 엔티티에 가해진 변화(저장/삭제 등)를 자식 엔티티에도 전파할지 결정</td>
<td><code>CascadeType.PERSIST</code>, <code>.REMOVE</code>, <code>.ALL</code> 등</td>
</tr>
<tr>
<td><strong><code>orphanRemoval</code></strong></td>
<td>부모와의 연관관계가 끊어진 자식(고아 객체)을 자동으로 삭제할지 결정</td>
<td><code>true</code>면 리스트에서 빼기만 해도 DB에서 삭제됨</td>
</tr>
<tr>
<td><strong>영속성 컨텍스트(Persistence Context)</strong></td>
<td>엔티티를 관리하는 &quot;1차 캐시&quot; 같은 공간, DB 세션이 열려있는 동안만 유지됨</td>
<td>이 공간이 닫히면 지연 로딩이 더 이상 안 됨</td>
</tr>
<tr>
<td><strong><code>LazyInitializationException</code></strong></td>
<td>영속성 컨텍스트(세션)가 이미 닫힌 뒤에, LAZY로 설정된 연관 데이터를 꺼내려다 나는 예외</td>
<td>&quot;no Session&quot; 메시지가 특징</td>
</tr>
</tbody></table>
<h3 id="코드로-다시-보기">코드로 다시 보기</h3>
<pre><code class="language-java">@Repository
public interface UserRepository extends JpaRepository&lt;UserEntity, String&gt; {
    // 메서드를 하나도 안 적어도 save(), findById(), findAll() 등이 기본 제공됨
}</code></pre>
<hr />
<h1 id="1-mybatis-vs-jpa--뭐가-달라지는가">1. MyBatis vs JPA — 뭐가 달라지는가</h1>
<p>지금까지는 인터페이스(<code>@Mapper</code>)를 만들고, 그 옆에 <strong>XML로 직접 SQL을 적어야</strong> CRUD가 동작했다. 오늘부터는 <code>JpaRepository</code>를 상속하는 인터페이스 하나만 만들면, <code>save()</code>, <code>findById()</code>, <code>findAll()</code>, <code>delete()</code> 같은 기본 CRUD 메서드가 <strong>아무 코드도 안 적었는데 자동으로 생긴다.</strong></p>
<pre><code class="language-java">@Repository
public interface UserRepository extends JpaRepository&lt;UserEntity, String&gt; {

}</code></pre>
<ul>
<li><code>JpaRepository&lt;UserEntity, String&gt;</code>에서 첫 번째 타입(<code>UserEntity</code>)은 다룰 엔티티, 두 번째 타입(<code>String</code>)은 <strong>그 엔티티의 기본키(PK) 타입</strong>이다. <code>UserEntity</code>의 기본키가 <code>email</code>(문자열)이라 <code>String</code>을 넣었다 — 예전에 <code>PersonDTO[]</code> 배열을 만들 때 &quot;이 배열이 다룰 타입&quot;을 제네릭으로 지정했던 것과 같은 원리가, 여기서는 &quot;이 리포지토리가 다룰 엔티티와 그 키 타입&quot;을 지정하는 데 쓰였다.</li>
<li>MyBatis는 &quot;SQL을 우리가 쓰고, 그 결과를 객체로 받는&quot; 방식이었다면, JPA는 &quot;객체(Entity)를 다루면, 그 객체 상태 변화를 JPA가 알아서 SQL로 바꿔서 실행해주는&quot; 방식이라는 게 가장 큰 차이다.</li>
</ul>
<hr />
<h1 id="2-entity-↔-dto-변환--정적-팩토리-메서드-패턴">2. Entity ↔ DTO 변환 — 정적 팩토리 메서드 패턴</h1>
<p>JPA를 쓰더라도 <strong>DB와 직접 맞닿는 Entity</strong>와 <strong>화면/API와 데이터를 주고받는 DTO</strong>는 여전히 구분해서 쓴다 (예전에 정리했던 &quot;DTO는 계층 간 전달용, Entity는 DB 매핑용&quot;이라는 역할 구분이 그대로 이어진다). 그래서 둘 사이를 변환하는 코드가 필요한데, 오늘은 이걸 <strong>정적 팩토리 메서드</strong>로 만드는 패턴을 배웠다.</p>
<pre><code class="language-java">public class UserResponseDTO {
    private String email;
    private String name;

    // 정적 팩토리 메서드
    public static UserResponseDTO from(UserEntity entity) {
        return UserResponseDTO.builder()
                .email(entity.getEmail())
                .name(entity.getName())
                .build();
    }
}</code></pre>
<ul>
<li><code>new UserResponseDTO(...)</code> 대신 <code>UserResponseDTO.from(entity)</code>처럼, <strong>이름이 있는 static 메서드</strong>로 객체를 만든다. &quot;이 메서드는 Entity로부터 변환해서 만든다&quot;는 의도가 메서드 이름(<code>from</code>)에 그대로 드러나서, 생성자를 직접 호출하는 것보다 코드를 읽는 사람이 의도를 파악하기 쉽다.</li>
<li>내부적으로는 예전에 배운 <strong>빌더 패턴</strong>을 그대로 활용해서, Entity의 각 필드 값을 꺼내 DTO 필드에 옮겨 담는다.</li>
</ul>
<hr />
<h1 id="3-연관관계-매핑--fetch는-언제-가져올지의-문제">3. 연관관계 매핑 — <code>fetch</code>는 &quot;언제&quot; 가져올지의 문제</h1>
<pre><code class="language-java">@Entity
public class BlogEntity {
    @Id
    private Integer id;
    private String title;

    @ManyToOne(fetch = FetchType.LAZY, optional = false)   // ①
    @JoinColumn(name = &quot;email&quot;)                              // ②
    private UserEntity author;
}</code></pre>
<p><strong>한 줄씩 정리하면</strong></p>
<ol>
<li><code>@ManyToOne(fetch = FetchType.LAZY)</code> — 블로그(N) : 작성자(1) 관계에서, 블로그 쪽이 &quot;여러 개의 블로그가 한 명의 작성자를 가리킨다(N:1)&quot;는 걸 표현한다. <code>fetch = LAZY</code>는 &quot;블로그를 조회할 때, 작성자 정보는 <strong>당장은 안 가져오고 실제로 필요할 때 가져오라</strong>&quot;는 뜻이다.</li>
<li><code>@JoinColumn(name = &quot;email&quot;)</code> — 어떤 컬럼을 기준으로 연결할지 지정한다. <code>BlogEntity</code> 테이블의 <code>email</code> 컬럼이 <code>UserEntity</code>를 가리키는 외래키 역할을 한다.</li>
</ol>
<p><strong>LAZY vs EAGER 비교</strong></p>
<table>
<thead>
<tr>
<th></th>
<th><code>LAZY</code></th>
<th><code>EAGER</code></th>
</tr>
</thead>
<tbody><tr>
<td>가져오는 시점</td>
<td>실제로 그 값을 사용할 때(지연 로딩)</td>
<td>부모를 조회하는 즉시(즉시 로딩)</td>
</tr>
<tr>
<td>장점</td>
<td>필요 없는 데이터까지 미리 안 가져와서 가벼움</td>
<td>나중에 따로 조회할 필요가 없어 간단함</td>
</tr>
<tr>
<td>주의할 점</td>
<td>세션(영속성 컨텍스트)이 닫힌 뒤에 접근하면 에러(아래 6번)</td>
<td>관계가 많으면 불필요한 조회가 늘어나 성능 저하 가능</td>
</tr>
</tbody></table>
<ul>
<li>오늘 필기에 <code>@ManyToOne(fetch = FetchType.LAZY, ...)</code>와 <code>@ManyToOne(fetch = FetchType.EAGER, ...)</code> 둘 다 예시로 나왔는데, 일반적으로는 <strong>연관관계 대부분에 LAZY를 기본으로 쓰고, 정말 필요한 곳에만 EAGER로 최적화</strong>하는 걸 권장한다고 배웠다. <code>@OneToMany</code>(1:N)는 기본값이 이미 LAZY이고, <code>@ManyToOne</code>(N:1)은 기본값이 EAGER라서, N:1 관계에도 의도적으로 LAZY를 붙여주는 게 안전하다는 것도 함께 짚었다.</li>
</ul>
<hr />
<h1 id="4-cascade와-orphanremoval--부모-자식-생명주기-관리">4. <code>cascade</code>와 <code>orphanRemoval</code> — 부모-자식 생명주기 관리</h1>
<pre><code class="language-java">@Entity
public class UserEntity {
    @Id
    private String email;

    @OneToMany(mappedBy = &quot;author&quot;,
               cascade = CascadeType.ALL,      // ①
               orphanRemoval = true)             // ②
    private List&lt;BlogEntity&gt; blogs = new ArrayList&lt;&gt;();
}</code></pre>
<p><strong>한 줄씩 정리하면</strong></p>
<ol>
<li><code>cascade = CascadeType.ALL</code> — 부모(<code>UserEntity</code>)에 가해지는 변화(저장, 삭제 등)를 자식(<code>BlogEntity</code>)에도 <strong>그대로 전파</strong>한다. 예를 들어 사용자를 저장할 때 그 사용자의 <code>blogs</code> 리스트에 새 블로그가 들어있으면, 별도로 블로그를 저장하지 않아도 사용자와 함께 자동으로 저장된다.</li>
<li><code>orphanRemoval = true</code> — 부모와의 연관관계가 끊어진 자식(고아 객체)을 자동으로 삭제한다. <code>user.getBlogs().remove(blog)</code>처럼 리스트에서 빼기만 해도, 그 블로그가 DB에서 삭제된다 — <code>blogService.delete()</code>를 따로 호출하지 않아도 된다.</li>
</ol>
<p>⚠️ <strong><code>cascade</code>와 <code>orphanRemoval</code>을 헷갈리지 않으려면</strong>: <code>cascade</code>는 &quot;부모에 실행한 작업(저장/삭제)을 자식에도 그대로 실행해라&quot;는 것이고, <code>orphanRemoval</code>은 그와 별개로 &quot;자식이 부모 리스트에서 <strong>빠지기만 해도</strong> 삭제해라&quot;는 것이다. <code>CascadeType.REMOVE</code>는 부모를 명시적으로 지울 때만 자식이 같이 지워지지만, <code>orphanRemoval=true</code>는 부모를 안 지워도 리스트에서만 빼면 자식이 지워진다는 차이가 있다고 정리했다.</p>
<hr />
<h1 id="5-⚠️-lazyinitializationexception--영속성-컨텍스트가-뭐길래">5. ⚠️ <code>LazyInitializationException</code> — 영속성 컨텍스트가 뭐길래</h1>
<pre><code>org.hibernate.LazyInitializationException: failed to lazily initialize a collection of role:
com.example.inspire_jpa.features.users.domain.entity.UserEntity.blogs:
could not initialize proxy - no Session</code></pre><p>오늘 실습 중 실제로 만난 에러다. <code>UserEntity</code>의 <code>blogs</code>(3번에서 LAZY로 설정한 연관 컬렉션)를 꺼내 쓰려다가 났다.</p>
<p><strong>왜 나는가 — 영속성 컨텍스트와 세션</strong></p>
<ul>
<li>JPA가 Entity를 조회하면, 그 Entity는 <strong>영속성 컨텍스트</strong>라는 &quot;관리 공간&quot; 안에 들어간다. 이 공간은 DB와 연결된 세션(Session)이 열려있는 동안만 유지된다.</li>
<li><code>fetch = LAZY</code>로 설정된 연관 데이터(<code>blogs</code>)는 진짜 값 대신 <strong>프록시(가짜 객체)</strong>로 채워져 있다가, 실제로 그 값을 꺼내 쓰는 순간(<code>user.getBlogs().get(0)</code> 등) 그제서야 DB에 다시 요청해서 진짜 값을 채운다.</li>
<li>그런데 이 &quot;실제로 꺼내 쓰는 순간&quot;이 <strong>이미 세션이 닫힌 뒤</strong>(예: 서비스 메서드가 끝나고 컨트롤러까지 넘어온 뒤)라면, 프록시는 DB에 다시 요청할 방법이 없어서 <code>no Session</code>이라는 메시지와 함께 예외를 던진다.</li>
</ul>
<p><strong>흐름으로 보면</strong></p>
<pre><code>서비스 메서드 안 (세션 열려있음)
   user = userRepository.findById(email)   ← blogs는 아직 프록시 상태
   return user                              ← 여기서 세션이 닫힘

컨트롤러 (세션 닫힌 뒤)
   user.getBlogs()  ← 프록시를 실제로 열어보려는 순간
   → LazyInitializationException! (DB에 다시 물어볼 세션이 없음)</code></pre><ul>
<li>이 문제를 해결하는 방법으로는(오늘은 이름만 스쳤지만) 세션이 열려있는 <strong>서비스 메서드 안에서 미리 필요한 데이터를 꺼내두거나(예: <code>.size()</code>를 한 번 불러서 강제로 초기화)</strong>, 애초에 <code>fetch = EAGER</code>로 바꾸거나, <code>fetch join</code>으로 한 번에 같이 가져오는 방법들이 있다고 한다.</li>
</ul>
<hr />
<h1 id="6-예고--필터로-토큰-만들기-인증인가">6. 예고 — 필터로 토큰 만들기, 인증/인가</h1>
<p>내일은 <strong>필터(Filter)</strong>를 실제로 만들어서 토큰을 검증하는 실습으로 이어진다고 예고됐다. 오늘은 이름만 정리해뒀다.</p>
<ul>
<li><strong>인증(Authentication)</strong>: &quot;너 누구야?&quot;를 확인하는 것 — 로그인해서 본인이 맞는지 증명하는 과정.</li>
<li><strong>인가(Authorization)</strong>: &quot;너 이거 할 수 있어?&quot;를 확인하는 것 — 인증된 사용자가 특정 자원에 접근할 권한이 있는지 확인하는 과정.</li>
</ul>
<p>앞서 로그인 응답 헤더에 실어 보냈던 <code>Authorization</code> 토큰이, 내일은 필터 단에서 &quot;요청마다 이 토큰이 유효한지&quot; 검증하는 데 실제로 쓰이게 될 것으로 보인다.</p>
<hr />
<h1 id="✨-새롭게-알게-된-점">✨ 새롭게 알게 된 점</h1>
<ul>
<li><code>JpaRepository&lt;Entity, ID&gt;</code>만 상속하면 기본 CRUD 메서드가 자동으로 생긴다 — MyBatis처럼 XML에 SQL을 직접 쓰지 않아도 된다.</li>
<li>Entity ↔ DTO 변환은 <code>new</code> 대신 <code>정적 팩토리 메서드</code>(<code>from(entity)</code> 등)로 만들면 의도가 더 잘 드러난다.</li>
<li><code>fetch</code>는 &quot;언제&quot; 가져올지(LAZY=필요할 때, EAGER=즉시), <code>cascade</code>는 &quot;부모의 작업을 자식에도 전파할지&quot;, <code>orphanRemoval</code>은 &quot;리스트에서만 빠져도 삭제할지&quot;로 각각 역할이 다르다.</li>
<li>영속성 컨텍스트(세션)가 닫힌 뒤에 LAZY 데이터를 꺼내려 하면 <code>LazyInitializationException</code>이 난다 — &quot;언제 세션이 열려있고 언제 닫히는지&quot;를 의식해야 한다.</li>
</ul>
<hr />
<h1 id="🤔-헷갈렸던-점">🤔 헷갈렸던 점</h1>
<ul>
<li><strong><code>cascade</code>와 <code>orphanRemoval</code>의 경계</strong>: 둘 다 &quot;부모-자식 생명주기&quot;와 관련 있어서 처음엔 같은 걸로 헷갈렸는데, &quot;부모에게 가해진 작업의 전파(cascade)&quot;와 &quot;리스트에서 빠지기만 해도 삭제(orphanRemoval)&quot;로 구분하고 나서야 명확해졌다.</li>
<li><strong><code>LazyInitializationException</code>을 실제로 어떻게 고치는지</strong>: 왜 나는지는 이해했지만, 오늘은 실제 해결 코드(fetch join, <code>@Transactional</code> 범위 조정 등)까지는 다루지 못해서 다음에 직접 고쳐봐야 한다.</li>
<li><strong><code>@OneToMany</code>/<code>@ManyToOne</code>의 기본 fetch 값이 서로 다르다는 것</strong>: <code>@OneToMany</code>는 기본이 LAZY, <code>@ManyToOne</code>은 기본이 EAGER라는 걸 외워야 하는데 아직 헷갈린다.</li>
</ul>
<hr />
<h1 id="다음에-할-일">다음에 할 일</h1>
<ul>
<li><input disabled="" type="checkbox" /> <code>LazyInitializationException</code>을 실제로 fetch join이나 <code>@Transactional</code> 범위 조정으로 고쳐보기</li>
<li><input disabled="" type="checkbox" /> <code>cascade = CascadeType.ALL</code> 대신 <code>PERSIST</code>, <code>MERGE</code> 등 세분화된 옵션도 하나씩 테스트해보기</li>
<li><input disabled="" type="checkbox" /> 내일 배울 필터(토큰 검증)를 위해 오늘 만든 <code>Authorization</code> 헤더 흐름 다시 복습해두기</li>
<li><input disabled="" type="checkbox" /> JPA 기반 CRUD 테스트를 JUnit으로 직접 작성해보기(오늘 커리큘럼에 있었지만 다루지 못함)</li>
</ul>
<hr />
<h1 id="💡-오늘의-회고">💡 오늘의 회고</h1>
<p>오늘 가장 크게 느낀 건 &quot;편해진 만큼 몰라도 되는 게 아니라, 다른 걸 알아야 한다&quot;는 것이었다. <code>JpaRepository</code>를 상속하기만 하면 CRUD가 다 되니 편했지만, 그 대신 <code>fetch</code>가 &quot;언제&quot; 데이터를 가져오는지, 그리고 그 &quot;언제&quot;가 세션이 열려있을 때인지 아닌지를 모르면 바로 <code>LazyInitializationException</code>을 만난다는 걸 실제로 겪었다.</p>
<p>MyBatis 때는 우리가 SQL을 직접 써서 &quot;언제 무슨 데이터를 가져올지&quot;가 코드에 명시적으로 드러났는데, JPA는 그 부분이 <code>fetch</code> 옵션 뒤에 숨어있다가 예상치 못한 순간에 드러난다는 느낌을 받았다. 편리함 뒤에 숨은 &quot;영속성 컨텍스트&quot;라는 개념을 이해하지 못하면 오히려 더 헷갈릴 수 있겠다는 생각이 들었다. 내일 필터로 토큰을 검증하는 실습을 하면, 지금까지 헤더로만 주고받던 <code>Authorization</code> 값이 실제로 어떻게 쓰이는지 이어질 것 같아 기대된다.</p>