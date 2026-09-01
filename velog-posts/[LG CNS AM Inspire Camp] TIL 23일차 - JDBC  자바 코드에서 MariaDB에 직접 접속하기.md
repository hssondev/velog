<blockquote>
<p><strong>한 줄 요약</strong></p>
<p>지금까지는 배열에 데이터를 흉내만 냈다면, 오늘은 <code>JDBC</code>로 자바 코드에서 진짜 MariaDB에 접속해서 <code>department</code> 테이블을 조회하고, 그 결과를 DTO 리스트로 변환하는 것까지 처음부터 완성했다. 이어서 Spring Boot 프로젝트를 새로 만들고 프로파일로 개발/운영 설정도 나눠봤다.</p>
</blockquote>
<hr />
<h1 id="📚-오늘-학습한-커리큘럼">📚 오늘 학습한 커리큘럼</h1>
<pre><code>JDBC란? — 자바와 DB를 연결하는 표준 인터페이스
        ↓
드라이버 로딩 + 접속 정보 준비 (URL, USER, PASSWORD)
        ↓
Connection 얻기 (DriverManager.getConnection)
        ↓
PreparedStatement + ResultSet으로 데이터 조회
        ↓
조회 결과를 DTO 리스트로 변환 (Builder 패턴)
        ↓
자원 해제 (close)
        ↓
Spring Boot 프로젝트 시작 + 프로파일(dev/prod) 설정
        ↓
(예고) JPA(자동) vs MyBatis(수동), TDD, SPA vs JSP</code></pre><h1 id="📌-오늘의-학습-키워드">📌 오늘의 학습 키워드</h1>
<ul>
<li>JDBC, 드라이버(Driver)</li>
<li><code>Connection</code>, <code>DriverManager</code></li>
<li><code>PreparedStatement</code>, <code>ResultSet</code></li>
<li>자원 해제(<code>close</code>)</li>
<li><code>@SpringBootApplication</code>, 프로파일(Profile)</li>
<li>(예고) JPA, MyBatis, TDD</li>
</ul>
<hr />
<h1 id="0-개념-정리-키워드-사전">0. 개념 정리 (키워드 사전)</h1>
<blockquote>
<p><strong>한 줄 요약</strong></p>
<p>JDBC는 &quot;자바에서 DB에 말을 거는 표준 방법&quot;이고, 순서는 항상 <strong>드라이버 로딩 → 연결 → SQL 실행 → 결과 처리 → 연결 종료</strong> 다섯 단계로 고정되어 있다.</p>
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
<td><strong>JDBC (Java DataBase Connectivity)</strong></td>
<td>자바 프로그램이 DB에 접속해서 SQL을 주고받을 수 있게 해주는 표준 인터페이스</td>
<td>DB 종류가 바뀌어도 코드 구조는 거의 동일</td>
</tr>
<tr>
<td><strong>드라이버(Driver)</strong></td>
<td>특정 DBMS(MariaDB, Oracle 등)와 실제로 통신하는 방법을 구현한 라이브러리</td>
<td><code>mariadb-java-client-3.5.10.jar</code></td>
</tr>
<tr>
<td><strong><code>Class.forName(...)</code></strong></td>
<td>드라이버 클래스를 메모리에 미리 불러오는 코드</td>
<td><code>&quot;org.mariadb.jdbc.Driver&quot;</code></td>
</tr>
<tr>
<td><strong><code>Connection</code></strong></td>
<td>DB와 맺어진 하나의 접속(세션)을 나타내는 객체</td>
<td><code>DriverManager.getConnection(URL, USER, PASSWORD)</code>로 얻음</td>
</tr>
<tr>
<td><strong><code>PreparedStatement</code></strong></td>
<td>실행할 SQL문을 담아두는 객체</td>
<td><code>conn.prepareStatement(sql)</code></td>
</tr>
<tr>
<td><strong><code>ResultSet</code></strong></td>
<td><code>SELECT</code> 실행 결과(여러 행)를 담은, <strong>커서로 한 행씩 이동하며 읽는</strong> 가상의 테이블</td>
<td><code>rset.next()</code>로 다음 행으로 이동</td>
</tr>
<tr>
<td><strong>자원 해제</strong></td>
<td>다 쓴 <code>Connection</code> 등을 <code>close()</code>로 반드시 반납하는 것</td>
<td>안 하면 연결이 계속 쌓여 리소스 누수가 남</td>
</tr>
</tbody></table>
<h3 id="코드로-다시-보기">코드로 다시 보기</h3>
<pre><code class="language-java">Class.forName(DRIVER);                                       // ① 드라이버 로딩
Connection conn = DriverManager.getConnection(URL, USER, PASSWORD); // ② 연결
PreparedStatement pstmt = conn.prepareStatement(sql);         // ③ SQL 준비
ResultSet rset = pstmt.executeQuery();                        // ④ 실행 + 결과 받기
while (rset.next()) { /* 한 행씩 처리 */ }                     // ⑤ 결과 순회
conn.close();                                                  // ⑥ 연결 종료</code></pre>
<hr />
<h1 id="1-드라이버-로딩과-접속-정보-준비">1. 드라이버 로딩과 접속 정보 준비</h1>
<pre><code class="language-java">private static final String DRIVER   = &quot;?&quot;;
private static final String USER     = &quot;?&quot;;
private static final String PASSWORD = &quot;?&quot;;
private static final String URL      = &quot;?&quot;;

public MariadbDao() {
    try {
        Class.forName(DRIVER);
        System.out.println(&quot;debug &gt;&gt;&gt;&gt; driver loading ok!&quot;);
    } catch (Exception e) {
        e.printStackTrace();
    }
}</code></pre>
<ul>
<li>네 값(<code>DRIVER</code>, <code>USER</code>, <code>PASSWORD</code>, <code>URL</code>)을 <code>static final</code>로 선언했다 — <strong>한 번 정해지면 바뀌지 않아야 하는 값</strong>이라는 뜻으로 <code>final</code>을, 인스턴스마다 새로 만들 필요 없이 클래스 전체가 공유하는 값이라는 뜻으로 <code>static</code>을 붙였다. 코드 주석에 &quot;<code>.env</code> 파일에 관리&quot;라고 적어둔 걸 보면, 실제로는 이런 민감한 접속정보를 소스코드에 직접 적지 않고 환경변수 파일로 분리해서 관리하는 게 정석이라는 것도 함께 배운 것으로 보인다.</li>
<li><code>URL</code>의 형식(<code>jdbc:mariadb://호스트:포트/DB이름</code>)에서 마지막의 <code>lgcns</code>는 접속할 <strong>데이터베이스 이름</strong>이다.</li>
<li><code>Class.forName(DRIVER)</code>: 문자열로 적힌 클래스 이름(<code>org.mariadb.jdbc.Driver</code>)을 실제로 메모리에 로딩한다. 이 드라이버가 로딩돼야, 뒤에서 <code>DriverManager</code>가 &quot;MariaDB에 접속하는 방법&quot;을 알 수 있다.</li>
<li>이 로딩 코드를 <strong>생성자</strong>에 넣어뒀다는 점도 눈여겨볼 만하다 — <code>MariadbDao</code> 객체가 만들어지는 시점에 딱 한 번 드라이버를 준비해두고, 이후 여러 메서드에서 반복해서 연결할 수 있게 하려는 구조다.</li>
</ul>
<hr />
<h1 id="2-connection-얻기--빈-값에서-실제-값으로">2. <code>Connection</code> 얻기 — 빈 값에서 실제 값으로</h1>
<p><strong>Java</strong> · 처음 시도 (13:45)</p>
<pre><code class="language-java">try {
   conn = DriverManager.getConnection(,,);   // ❌ 문법 오류, 인자가 비어있음
} catch(Exception e) {
    e.printStackTrace();
}</code></pre>
<ul>
<li>처음엔 <code>getConnection()</code>에 어떤 값을 넣어야 할지 구조만 잡아두고 인자를 비워뒀다(컴파일도 안 되는 상태). 이런 식으로 <strong>일단 뼈대(구조)부터 잡고, 그 다음 실제 값을 채워 넣는</strong> 순서로 코드를 짜는 걸 오늘 계속 반복했다.</li>
</ul>
<p><strong>Java</strong> · 값 채워 넣기 (14:03)</p>
<pre><code class="language-java">conn = DriverManager.getConnection(URL, USER, PASSWORD);
System.out.println(&quot;debug &gt;&gt;&gt;&gt; conn ok!&quot; + conn);</code></pre>
<ul>
<li><code>DriverManager.getConnection(URL, USER, PASSWORD)</code>: 위에서 준비해둔 세 값으로 실제 DB에 접속을 시도하고, 성공하면 <code>Connection</code> 객체를 돌려준다. 이 객체가 있어야 그 뒤로 SQL을 실행할 수 있다.</li>
</ul>
<hr />
<h1 id="3-preparedstatement--resultset으로-데이터-조회하기">3. <code>PreparedStatement</code> + <code>ResultSet</code>으로 데이터 조회하기</h1>
<pre><code class="language-java">String sql = &quot;select * from department&quot;;
List&lt;DepartmentResponseDTO&gt; list = new ArrayList&lt;&gt;();

pstmt = conn.prepareStatement(sql);        // ①
rset = pstmt.executeQuery();                // ②

while (rset.next()) {                        // ③
    list.add(DepartmentResponseDTO.builder()  // ④
        .dept_id(rset.getString(1))
        .dept_name(rset.getString(2))
        .loc_id(rset.getString(3))
        .build());
}</code></pre>
<ul>
<li>① <code>conn.prepareStatement(sql)</code>: 실행할 SQL 문자열을 <code>PreparedStatement</code> 객체로 감싼다.</li>
<li>② <code>pstmt.executeQuery()</code>: 그 SQL을 실제로 실행하고, <code>SELECT</code>의 결과(여러 행)를 <code>ResultSet</code>으로 돌려받는다.</li>
<li>③ <code>while (rset.next())</code>: <code>ResultSet</code>은 처음엔 &quot;첫 번째 행 앞&quot;을 가리키고 있는 커서 상태다. <code>rset.next()</code>를 호출할 때마다 <strong>커서가 한 행씩 아래로 이동</strong>하고, 더 이상 이동할 행이 없으면 <code>false</code>를 반환해서 반복이 끝난다.</li>
<li>④ <code>rset.getString(1)</code>, <code>rset.getString(2)</code>, <code>rset.getString(3)</code>: 지금 커서가 가리키는 행에서, <strong>몇 번째 컬럼</strong>의 값을 문자열로 꺼내는지 숫자로 지정한다(1부터 시작). <code>department</code> 테이블의 컬럼 순서(<code>dept_id</code>, <code>dept_name</code>, <code>loc_id</code>)에 맞춰 하나씩 꺼내서 <code>DepartmentResponseDTO.builder()</code>로 객체 하나를 만들고, <code>list</code>에 담는다.</li>
<li>이전에 배운 <strong>빌더 패턴</strong>이 여기서 그대로 재사용됐다 — DB에서 꺼낸 값 하나하나를 빌더 체인으로 조립해서 DTO 객체를 완성하는 방식이다.</li>
</ul>
<hr />
<h1 id="4-자원-해제--finally에서-close">4. 자원 해제 — <code>finally</code>에서 <code>close()</code></h1>
<pre><code class="language-java">} finally {
    try {
        if (conn != null) { conn.close(); }
    } catch (Exception e) {
        e.printStackTrace();
    }
}
return list;</code></pre>
<ul>
<li><code>Connection</code>은 DB 서버와 실제로 맺어진 연결이라, 다 쓰고 나면 반드시 반납(<code>close()</code>)해야 한다. 안 그러면 연결이 계속 쌓여서 나중에는 더 이상 새 연결을 못 만드는 상황이 생길 수 있다고 배웠다.</li>
<li><code>close()</code> 자체도 예외를 던질 수 있어서, <code>finally</code> 블록 안에 또 <code>try-catch</code>를 둬야 하는 이중 구조가 됐다 — 지난번 IO Stream에서 겪었던 것과 똑같은 번거로움이다. (<code>try-with-resources</code>로 더 깔끔하게 만들 수 있다는 것도 그때 이미 배웠으니, 다음엔 <code>Connection</code>에도 적용해볼 만하다.)</li>
<li><code>if (conn != null)</code>으로 감싼 이유: 애초에 연결 자체가 실패했다면 <code>conn</code>이 여전히 <code>null</code>일 수 있는데, 그 상태에서 <code>close()</code>를 부르면 또 다른 예외(<code>NullPointerException</code>)가 나기 때문이다.</li>
</ul>
<hr />
<h1 id="5-실행해서-결과-확인하기">5. 실행해서 결과 확인하기</h1>
<pre><code class="language-java">public class JdbcApp {
    public static void main(String[] args) {
        MariadbDao dao = new MariadbDao();
        List&lt;DepartmentResponseDTO&gt; list = dao.departments();
        list.stream()
            .forEach(System.out::println);
    }
}</code></pre>
<ul>
<li><code>dao.departments()</code>로 받아온 리스트를, 저번에 배운 Stream API(<code>stream().forEach(...)</code>)로 한 줄씩 출력했다 — 오늘 새로 배운 JDBC와 예전에 배운 Stream이 자연스럽게 이어 붙은 부분이다.</li>
</ul>
<hr />
<h1 id="6-spring-boot-프로젝트-시작하기">6. Spring Boot 프로젝트 시작하기</h1>
<p>오늘 마지막에는 순수 JDBC 코드에서 한 걸음 나아가, <strong>Spring Boot 프로젝트</strong>를 새로 만들어봤다.</p>
<pre><code class="language-java">package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.jdbc.autoconfigure.DataSourceAutoConfiguration;

@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)   // ①
public class DemoApplication {

    /*
    npm start (리액트 기동)
    build - ./gradlew bootJar
    deploy - start tomcat server(xxx.jar)
    */

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);            // ②
    }

}</code></pre>
<ul>
<li>① <code>@SpringBootApplication</code>: 이 클래스가 Spring Boot 애플리케이션의 <strong>시작점</strong>이라는 걸 알리는 어노테이션이다. 지금까지 우리가 손으로 하나씩 만들었던 <code>BeanFactory</code>, <code>싱글톤</code>, <code>DI</code> 같은 것들을 Spring이 대신 자동으로 설정해준다.</li>
<li><code>exclude = DataSourceAutoConfiguration.class</code>: Spring Boot는 보통 프로젝트를 켜자마자 DB 연결(DataSource)까지 자동으로 준비하려고 하는데, 아직 DB 접속 정보(<code>application.yml</code> 등)를 안 넣어둔 상태라 그대로 두면 시작하자마자 에러가 난다. 그래서 &quot;DataSource 자동 설정은 일단 빼고 켜라&quot;고 명시적으로 제외한 것 — 오늘 배운 JDBC를 직접 손으로 짜본 다음이라, Spring이 이 과정을 얼마나 많이 자동화해주는지 대비되어 보였다.</li>
<li>② <code>SpringApplication.run(...)</code>: 실제로 애플리케이션을 실행시키는 코드. 이 한 줄 안에서 내장 톰캣 서버를 띄우고, 여러 설정을 읽어들이는 등 많은 일이 자동으로 일어난다고 배웠다.</li>
<li>주석에 적힌 배포 흐름(<code>npm start</code> → <code>./gradlew bootJar</code>(빌드) → 톰캣 서버로 실행)은, 프론트(React)와 백엔드(Spring Boot)가 개발할 땐 따로 실행되다가, 배포할 땐 백엔드가 빌드된 결과물(<code>.jar</code>)을 톰캣 서버로 띄우는 방식이라는 걸 미리 정리해둔 메모로 보인다.</li>
</ul>
<h2 id="프로파일profile로-환경별-설정-나누기">프로파일(Profile)로 환경별 설정 나누기</h2>
<pre><code class="language-yaml"># application.yml
spring:
  profiles:
    default: dev</code></pre>
<pre><code class="language-yaml"># application-dev.yml
server:
  port: 8000

spring:
  config:
    activate:
      on-profile: dev</code></pre>
<pre><code class="language-yaml"># application-prod.yml
spring:
  config:
    activate:
      on-profile: prod</code></pre>
<ul>
<li><code>spring.profiles.default: dev</code>: 별도로 어떤 프로파일을 쓸지 지정하지 않으면, <strong>기본으로 <code>dev</code>(개발) 설정을 쓰겠다</strong>는 뜻이다.</li>
<li><code>spring.config.activate.on-profile: dev</code> / <code>on-profile: prod</code>: 각 설정 파일이 <strong>&quot;어떤 프로파일일 때만 적용될지&quot;</strong>를 스스로 표시해둔 것이다. <code>dev</code> 파일에는 <code>server.port: 8000</code>(개발 중엔 8000번 포트로 실행)이 들어있고, <code>prod</code> 파일은 아직 세부 설정은 비어있지만 &quot;운영 환경용&quot;이라는 자리만 잡아둔 상태로 보인다.</li>
<li><strong>왜 나누는가</strong>: 개발할 때 쓰는 설정(포트, DB 주소 등)과 실제 서비스에 배포할 때 쓰는 설정은 다른 경우가 많다. 프로파일로 나눠두면, 코드는 그대로 두고 <strong>어떤 프로파일로 실행하느냐만 바꿔서</strong> 개발용/운영용 설정을 손쉽게 전환할 수 있다.</li>
</ul>
<hr />
<h1 id="7-다음으로-이어질-것들">7. 다음으로 이어질 것들</h1>
<p>Spring Boot 프로젝트 세팅까지는 오늘 실제로 손을 대봤고, 아래는 아직 이름만 스쳐 지나간 것들이다.</p>
<ul>
<li><strong>JPA(자동) vs MyBatis(수동)</strong>: 지금까지 오늘처럼 SQL을 직접 문자열로 써서 실행하는 방식이 MyBatis에 가깝고, JPA는 이런 JDBC 코드 대부분을 자동으로 대신 처리해준다는 것으로 이해했다. 정확한 차이는 다음에 배울 것 같다.</li>
<li><strong>Spring Data JPA, MyBatis, Lombok 등 나머지 의존성</strong>: 이름만 나열됐고, 각각 뭘 하는지는 아직 안 배웠다.</li>
<li><strong>TDD</strong>: 이름만 언급됨.</li>
<li><strong>SPA(react/vue, 비동기, JSON 응답) vs JSP(레거시 방식, 서버가 페이지 자체를 만들어 내려주는 방식)</strong>: 지금 배우는 구조(React + 비동기 + JSON)와, 현장에서 여전히 쓰이는 구조(JSP)가 다르다는 것도 짧게 예고됐다.</li>
</ul>
<hr />
<h1 id="✨-새롭게-알게-된-점">✨ 새롭게 알게 된 점</h1>
<ul>
<li>JDBC 코드는 항상 &quot;드라이버 로딩 → 연결 → SQL 준비 → 실행 → 결과 순회 → 자원 해제&quot;라는 같은 순서를 따른다.</li>
<li><code>ResultSet</code>은 커서를 가진 구조라서, <code>rset.next()</code>로 한 행씩 이동하며 값을 꺼내야 한다.</li>
<li><code>rset.getString(인덱스)</code>의 인덱스는 <strong>테이블 컬럼 순서(1부터 시작)</strong>와 일치해야 한다.</li>
<li>접속정보(URL/USER/PASSWORD)는 실무에서는 소스코드에 직접 적지 않고 <code>.env</code> 같은 별도 파일로 분리해서 관리한다.</li>
<li><code>Connection</code>도 <code>IO Stream</code>처럼 반드시 <code>close()</code>로 반납해야 하는 자원이다.</li>
<li><code>@SpringBootApplication</code>은 지금까지 손으로 만들어온 BeanFactory·싱글톤·DI 같은 것들을 자동화해주는 시작점이고, <code>exclude</code>로 자동 설정의 일부를 끌 수도 있다.</li>
<li>프로파일(<code>on-profile</code>)로 개발용/운영용 설정을 분리해두면, 코드 변경 없이 실행 환경만 바꿔서 설정을 전환할 수 있다.</li>
</ul>
<hr />
<h1 id="🤔-헷갈렸던-점">🤔 헷갈렸던 점</h1>
<ul>
<li><strong><code>Class.forName</code>이 정확히 무슨 일을 하는지</strong>: &quot;드라이버를 로딩한다&quot;는 설명은 들었지만, 왜 굳이 문자열로 클래스 이름을 넘겨서 로딩해야 하는지는 아직 완전히 이해하지 못했다.</li>
<li><strong><code>PreparedStatement</code>와 그냥 <code>Statement</code>의 차이</strong>: 오늘은 <code>PreparedStatement</code>만 써봤는데, 이름에 &quot;Prepared&quot;가 왜 붙는지는 다음에 SQL 인젝션 방지 등과 관련이 있다고 들은 것 같아 더 찾아봐야겠다.</li>
<li><strong>텍스트 블록(<code>&quot;&quot;&quot;</code>) 안에서 따옴표 처리</strong>: 일반 문자열과 달리 텍스트 블록 안에서는 <code>&quot;</code>가 특별한 의미가 없어서, 실수로 남겨둔 <code>&quot;</code>가 그대로 SQL의 일부가 되어버린다는 걸 오늘 코드에서 발견했다.</li>
</ul>
<hr />
<h1 id="다음에-할-일">다음에 할 일</h1>
<ul>
<li><input disabled="" type="checkbox" /> SQL 텍스트 블록에 남은 <code>&quot; ;</code> 잘못된 문자 제거하고 정상 실행 확인하기</li>
<li><input disabled="" type="checkbox" /> <code>Connection</code>/<code>PreparedStatement</code>/<code>ResultSet</code>에 <code>try-with-resources</code> 적용해보기</li>
<li><input disabled="" type="checkbox" /> JPA와 MyBatis의 차이를 다음 수업에서 배우면 오늘 짠 JDBC 코드와 비교해보기</li>
<li><input disabled="" type="checkbox" /> <code>application-prod.yml</code>에 실제 운영용 설정 채워보기</li>
<li><input disabled="" type="checkbox" /> <code>spring.profiles.active</code>를 dev/prod로 바꿔가며 실제로 포트가 달라지는지 확인해보기</li>
</ul>
<hr />
<h1 id="💡-오늘의-회고">💡 오늘의 회고</h1>
<p>오늘은 처음으로 &quot;진짜 데이터베이스&quot;에 자바 코드로 직접 접속해봤다. 그동안 <code>BlogReactDao</code>에서 배열로 데이터를 흉내 냈던 것과 달리, 오늘은 실제로 MariaDB의 <code>department</code> 테이블에서 값을 꺼내와 화면에 찍어보니 느낌이 확실히 달랐다.</p>
<p><code>getConnection(,,)</code>처럼 빈 틀부터 잡고 하나씩 채워나가는 방식으로 코드를 짜는 걸 보면서, 처음부터 완성된 코드를 한 번에 쓰는 게 아니라 &quot;일단 구조를 잡고 → 컴파일 에러를 하나씩 없애며 → 실제 동작까지 채워나가는&quot; 흐름 자체가 오늘 배운 중요한 습관이라는 생각이 들었다. <code>close()</code>를 빼먹으면 안 된다는 것도, IO Stream 때 배운 원리가 DB 연결에도 똑같이 적용된다는 걸 다시 확인한 부분이었다.</p>