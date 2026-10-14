<blockquote>
<p><strong>한 줄 요약</strong></p>
<p>서버 재기동도, 서비스마다 일일이 refresh 호출도 없이 Spring Cloud Bus로 전체 서비스 설정을 한 번에 갱신하는 법과, Git에 비밀번호가 평문으로 올라가지 않도록 대칭키/비대칭키로 설정값을 암호화하는 법을 익혔다.</p>
</blockquote>
<hr />
<h1 id="📚-오늘-학습한-커리큘럼">📚 오늘 학습한 커리큘럼</h1>
<pre><code>RabbitMQ 설치 및 실행
        ↓
Config Server에 Actuator + Spring Cloud Bus 의존성 추가
        ↓
POST /actuator/busrefresh 로 전체 서비스 설정 동시 갱신
        ↓
설정값 암호화가 필요한 이유 (Git에 평문 노출 문제)
        ↓
Symmetric Encryption (대칭키) — encrypt.key, /encrypt, /decrypt
        ↓
Asymmetric Encryption (비대칭키) — keytool로 RSA 키 쌍 생성</code></pre><h1 id="📌-오늘의-학습-키워드">📌 오늘의 학습 키워드</h1>
<ul>
<li>Spring Cloud Bus</li>
<li>RabbitMQ / AMQP</li>
<li>Actuator busrefresh</li>
<li>Symmetric Encryption (대칭키 암호화)</li>
<li>Asymmetric Encryption (비대칭키 암호화)</li>
<li>JCE (Java Cryptography Extension)</li>
<li>{cipher} 접두어</li>
</ul>
<hr />
<h1 id="0-개념-정리-키워드-사전">0. 개념 정리 (키워드 사전)</h1>
<blockquote>
<p><strong>한 줄 요약</strong></p>
<p>Spring Cloud Bus는 &quot;방송(broadcast)&quot;, 암호화는 &quot;Git에는 자물쇠 채운 채로 보관&quot;이라고 생각하면 된다.</p>
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
<td><strong>Spring Cloud Bus</strong></td>
<td>분산된 노드들을 경량 메시지 브로커(RabbitMQ 등)로 연결해, 설정 변경을 모든 노드에 한 번에 전달(Broadcast)하는 기능</td>
<td>Config Server + Spring Cloud Bus 조합으로 사용</td>
</tr>
<tr>
<td><strong>RabbitMQ</strong></td>
<td>AMQP(메시지 지향 미들웨어 표준 프로토콜)를 구현한 메시지 브로커. Spring Cloud Bus가 이벤트를 발행/수신하는 통로 역할</td>
<td><code>brew install rabbitmq</code> 또는 Docker로 설치, 관리 UI는 <code>localhost:15672</code></td>
</tr>
<tr>
<td><strong>Actuator bus-refresh</strong></td>
<td><code>/actuator/busrefresh</code> 엔드포인트. 호출 한 번으로 Config Server가 Git의 최신 설정을 읽어 Bus를 통해 연결된 모든 서비스에 전파</td>
<td>서비스별로 <code>/actuator/refresh</code>를 각각 호출하지 않아도 됨</td>
</tr>
<tr>
<td><strong>Symmetric Encryption (대칭키)</strong></td>
<td>암호화와 복호화에 같은 키(encrypt.key)를 사용하는 방식</td>
<td>설정이 간단하지만, 키가 유출되면 바로 복호화 가능해 상대적으로 덜 안전</td>
</tr>
<tr>
<td><strong>Asymmetric Encryption (비대칭키)</strong></td>
<td>Public Key로 암호화하고 Private Key로만 복호화하는 방식(RSA). Java keytool로 키 쌍 생성</td>
<td><code>keytool -genkeypair</code> 로 <code>.jks</code> 키스토어 생성 후 Config Server에 등록</td>
</tr>
<tr>
<td><strong>JCE</strong></td>
<td>Java Cryptography Extension. 강력한 암호화 알고리즘을 쓰려면 정책 파일 교체가 필요할 수 있음</td>
<td>&quot;Illegal key size&quot; 에러 발생 시 무제한 암호화 정책 파일로 교체</td>
</tr>
<tr>
<td><strong>{cipher} 접두어</strong></td>
<td>암호화된 값임을 Config Server가 식별하는 표시. 이 접두어가 붙은 값은 클라이언트에 전달되기 직전 자동으로 복호화됨</td>
<td><code>password: '{cipher}a93997a5da...'</code></td>
</tr>
</tbody></table>
<h3 id="코드로-다시-보기">코드로 다시 보기</h3>
<pre><code class="language-bash"># 1) 평문 값을 암호화 (Config Server에 요청)
curl -X POST http://127.0.0.1:8012/encrypt -d &quot;sa&quot;
# → a93997a5da8b87c123fee0c6e1627f678b6c60db8a45ab31514253cc4a4b01fd

# 2) Git에 저장하는 설정 파일 (user-service.yml)
# spring:
#   datasource:
#     password: '{cipher}a93997a5da8b87c123fee0c6e1627f678b6c60db8a45ab31514253cc4a4b01fd'

# 3) 설정이 변경된 뒤, 전체 서비스에 한 번에 알리기
curl -X POST http://127.0.0.1:8012/actuator/busrefresh</code></pre>
<hr />
<h1 id="1-서버-재기동-없이-설정-갱신하기--그런데-서비스가-여러-개라면">1. 서버 재기동 없이 설정 갱신하기 — 그런데 서비스가 여러 개라면?</h1>
<p>지난 시간엔 <code>spring-boot-starter-actuator</code>를 추가해서 <code>/actuator/refresh</code>만 호출하면 서버를 재기동하지 않고도 바뀐 설정을 반영할 수 있다는 걸 배웠다. 문제는 이 방식이 서비스 하나당 한 번씩만 적용된다는 점이다. User, Order, Catalog 서비스가 각각 떠 있다면 설정을 바꿀 때마다 세 번 호출해야 한다는 뜻이다. 서비스가 10개, 100개로 늘어나면 이 방식은 현실적으로 쓸 수 없다. 이 문제를 해결하는 게 오늘 배운 <strong>Spring Cloud Bus</strong>다.</p>
<pre><code class="language-xml">&lt;dependency&gt;
    &lt;groupId&gt;org.springframework.boot&lt;/groupId&gt;
    &lt;artifactId&gt;spring-boot-starter-actuator&lt;/artifactId&gt;
&lt;/dependency&gt;</code></pre>
<p>위 Actuator 의존성은 그대로 두고, Config Server에 Spring Cloud Bus 의존성을 추가하는 것부터 시작한다.</p>
<h1 id="2-rabbitmq-설치-및-실행">2. RabbitMQ 설치 및 실행</h1>
<p>Spring Cloud Bus는 실제로 노드 간 이벤트를 주고받을 메시지 브로커가 필요하다. 오늘은 RabbitMQ를 사용했다.</p>
<pre><code class="language-bash"># macOS (Homebrew)
$ brew update
$ brew install rabbitmq
$ brew services start rabbitmq
# 또는 백그라운드 서비스 없이 바로 실행
$ rabbitmq-server

# Docker로 띄우는 경우
$ docker run -it --rm --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:4-management</code></pre>
<p>설치 후 <code>http://127.0.0.1:15672</code>에 접속하면 RabbitMQ 관리 콘솔이 뜨고, 기본 계정(guest/guest)으로 로그인해서 Connections, Queues 같은 상태를 확인할 수 있다. 처음엔 포트가 왜 두 개(5672, 15672)인지 헷갈렸는데, 5672는 실제 메시지를 주고받는 AMQP 포트이고 15672는 사람이 보는 웹 관리 UI 포트라는 걸 알고 나니 이해가 됐다.</p>
<h1 id="3-spring-cloud-bus-적용--한-번의-호출로-전체-서비스-갱신">3. Spring Cloud Bus 적용 — 한 번의 호출로 전체 서비스 갱신</h1>
<p>RabbitMQ가 떠 있는 상태에서 Config Server에 Spring Cloud Bus 의존성을 추가하면, Config Server가 RabbitMQ를 통해 &quot;설정이 바뀌었다&quot;는 이벤트를 발행할 수 있게 된다. 이제 <code>/actuator/refresh</code>가 아니라 <code>/actuator/busrefresh</code>를 호출하면 된다.</p>
<pre><code>POST http://127.0.0.1:8012/actuator/busrefresh</code></pre><p><img alt="" src="https://velog.velcdn.com/images/hssondev/post/aa765e52-728d-420e-93e9-0defe399d343/image.png" /></p>
<p>흐름을 보면, 클라이언트는 Config Server에 딱 한 번만 요청을 보낸다. Config Server는 Git 저장소에서 최신 설정을 읽어온 뒤, 그 변경 사항을 RabbitMQ(Message Broker)에 이벤트로 발행(발행)한다. 그러면 RabbitMQ에 연결되어 있던 User, Order, Catalog 세 서비스가 동시에 이 이벤트를 받아서 각자 최신 설정을 다시 가져온다. 즉 &quot;한 곳에 알리면 알아서 퍼진다(Broadcast)&quot;는 구조다. 처음엔 각 서비스가 Config Server를 직접 다시 호출하는 건지, 아니면 RabbitMQ가 설정 값 자체를 들고 있다가 뿌려주는 건지 헷갈렸는데, 실제로는 RabbitMQ는 &quot;설정이 바뀌었다&quot;는 신호(이벤트)만 전달하고, 실제 최신 설정 값은 각 서비스가 Config Server에 다시 요청해서 가져오는 방식이라는 걸 알게 됐다.</p>
<h1 id="4-설정값-암호화가-필요한-이유">4. 설정값 암호화가 필요한 이유</h1>
<p>Config Server는 데이터베이스 접속 정보 같은 설정을 Git 저장소(Private Git Repository)에서 읽어온다. 그런데 이 설정 파일에 비밀번호를 평문 그대로 적어두면, Git 저장소 접근 권한이 있는 사람 누구나 비밀번호를 볼 수 있다는 문제가 생긴다. 사내 Private Repository라고 해도 이력(히스토리)에 평문 비밀번호가 영구히 남는 건 좋은 방식이 아니다. 그래서 Config Server는 설정값을 암호화된 상태로 저장하고, 서비스에 내려줄 때만 복호화하는 기능을 제공한다.</p>
<h1 id="5-symmetric-encryption-대칭키-암호화">5. Symmetric Encryption (대칭키 암호화)</h1>
<p>가장 먼저 해본 건 대칭키 방식이다. Config Server의 <code>bootstrap.yml</code>에 암호화/복호화에 쓸 키 하나만 등록하면 된다.</p>
<pre><code class="language-yaml"># bootstrap.yml
encrypt:
  key: abcdefghijklmnopqrstuvexyz1234567890</code></pre>
<p>이 키가 등록된 Config Server는 <code>/encrypt</code>, <code>/decrypt</code> 엔드포인트를 제공한다. 평문을 <code>/encrypt</code>에 POST하면 암호문이 돌아오고, 이 암호문을 다시 <code>/decrypt</code>에 POST하면 원래 평문으로 돌아온다.</p>
<pre><code class="language-bash"># 암호화
curl -X POST http://127.0.0.1:8888/encrypt -d &quot;kenneth&quot;
# → 342af5d280abf8ac7537579292acbeb52f50093aefe69489c71b248085c02616

# 복호화 (역방향으로도 확인 가능)
curl -X POST http://127.0.0.1:8888/decrypt -d &quot;342af5d280abf8ac...&quot;
# → kenneth</code></pre>
<p>이렇게 얻은 암호문을 Git에 올리는 설정 파일(user-service.yml)의 비밀번호 자리에 <code>{cipher}</code> 접두어와 함께 넣어두면 끝이다.</p>
<pre><code class="language-yaml"># user-service.yml (Git 저장소에 실제로 올라가는 파일)
spring:
  datasource:
    driver-class-name: org.h2.Driver
    url: jdbc:h2:mem:testdb
    username: sa
    password: '{cipher}a93997a5da8b87c123fee0c6e1627f678b6c60db8a45ab31514253cc4a4b01fd'</code></pre>
<p>여기서 신기했던 부분은, Config Server가 이 설정을 클라이언트(서비스)에 내려줄 때 <code>{cipher}</code> 접두어를 보고 자동으로 복호화해서 평문으로 바꿔준다는 점이다. 즉 Git에는 암호문만 남고, 서비스가 받는 순간에만 평문이 된다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/hssondev/post/823001c2-8c8b-49fa-b619-a4b02caa4504/image.png" /></p>
<p>실습 중 한 가지 실수도 겪었다. Config Server의 <code>user-service.yml</code>에서 암호문 뒤에 글자를 잘못 추가해 <code>{cipher}...d_wrong</code> 처럼 암호문을 깨뜨린 적이 있었는데, 이때 Config Server가 복호화에 실패하면서 <code>invalid.spring.datasource.password: &quot;&lt;n/a&gt;&quot;</code> 라는 값으로 응답했다. 처음엔 왜 설정값 자체가 사라졌는지 당황했는데, 암호문이 조금이라도 손상되면 복호화가 아예 실패해서 원래 키가 <code>invalid.</code> 접두어로 바뀌어 내려온다는 걸 알고 나서야 원인을 파악할 수 있었다.</p>
<h1 id="6-asymmetric-encryption-비대칭키-암호화">6. Asymmetric Encryption (비대칭키 암호화)</h1>
<p>대칭키 방식은 설정은 간단하지만, <code>encrypt.key</code> 값 하나만 유출되면 모든 암호문을 바로 풀 수 있다는 약점이 있다. 그래서 실무에서는 Public Key로 암호화하고 Private Key로만 복호화할 수 있는 비대칭키(RSA) 방식을 더 선호한다고 배웠다. Private Key만 안전하게 보관하면, Public Key가 외부에 알려져도 암호문을 풀 수는 없기 때문이다.</p>
<p>Java의 <code>keytool</code>로 키 쌍을 생성한다.</p>
<pre><code class="language-bash">$ mkdir ${user.home}/Desktop/Work/keystore
$ keytool -genkeypair -alias apiEncryptionKey -keyalg RSA \
  -dname &quot;CN=Kenneth Lee, OU=API Development, O=joneconsulting.co.kr, L=Seoul, C=KR&quot; \
  -keypass &quot;1q2w3e4r&quot; -keystore apiEncryptionKey.jks -storepass &quot;1q2w3e4r&quot;</code></pre>
<p>각 옵션이 뭘 의미하는지 처음엔 헷갈렸는데, 정리하면 <code>-alias</code>는 키스토어 안에서 이 키 쌍을 구분하는 이름, <code>-dname</code>은 인증서에 들어갈 소유자 정보, <code>-keypass</code>는 개별 키에 대한 비밀번호, <code>-storepass</code>는 키스토어 파일 자체를 여는 비밀번호다. 이렇게 생성된 <code>.jks</code> 파일을 Config Server의 <code>bootstrap.yml</code>에 등록하면 대칭키 대신 비대칭키로 암호화/복호화를 처리한다.</p>
<pre><code class="language-yaml"># bootstrap.yml
encrypt:
  key-store:
    location: file://${user.home}/Desktop/Work/keystore/apiEncryptionKey.jks
    password: 1q2w3e4r
    alias: apiEncryptionKey</code></pre>
<p>이후 <code>/encrypt</code>를 호출하면 대칭키 때보다 훨씬 긴 암호문이 돌아오는데, RSA 암호화 특성상 결과값 길이가 길어진다는 걸 직접 확인할 수 있었다. JCE 정책 때문에 &quot;Illegal key size&quot; 같은 에러가 날 수 있다는 것도 알게 됐는데, 이 경우 Oracle에서 제공하는 무제한 강도 정책 파일(jce_policy)을 JDK의 <code>jre/lib/security</code> 경로에 교체해서 넣어주면 해결된다고 한다.</p>
<hr />
<h1 id="✨-새롭게-알게-된-점">✨ 새롭게 알게 된 점</h1>
<ul>
<li>Spring Cloud Bus는 메시지 브로커를 통해 &quot;설정이 바뀌었다&quot;는 이벤트만 전파하고, 실제 최신 설정 값은 각 서비스가 Config Server에 다시 요청해서 받아온다는 것 — 메시지 브로커가 설정값 자체를 들고 있는 게 아니라는 점이 새로웠다.</li>
<li><code>{cipher}</code> 접두어가 붙은 설정값은 Config Server를 거치는 순간 자동으로 복호화된다는 것, 그리고 암호문이 조금이라도 손상되면 <code>invalid.</code> 접두어로 응답이 바뀌면서 바로 실패를 알려준다는 것.</li>
<li>대칭키보다 비대칭키(RSA) 방식이 실무에서 더 선호되는 이유 — 키 하나가 유출되는 것과 Private Key가 안전히 분리되어 있는 것의 보안 수준 차이.</li>
</ul>
<hr />
<h1 id="🤔-헷갈렸던-점">🤔 헷갈렸던 점</h1>
<ul>
<li><code>encrypt.key</code>를 <code>bootstrap.yml</code>에 평문으로 적어두는 게 괜찮은 건지 처음엔 의아했다. 결국 이 키 자체도 외부 보안 저장소(Vault 등)로 분리해서 관리해야 한다는 걸 알고 나서 이해가 됐다 — 암호화 체계의 &quot;가장 바깥쪽 열쇠&quot;는 결국 어딘가에는 안전하게 보관해야 한다는 것.</li>
<li><code>keytool</code> 명령어의 <code>-alias</code>, <code>-dname</code>, <code>-keypass</code>, <code>-storepass</code> 옵션이 뭐가 다른지 헷갈렸는데, alias는 키 식별자, dname은 인증서 소유자 정보, keypass/storepass는 각각 키와 키스토어 파일을 보호하는 별개의 비밀번호라는 걸 구분하고 나니 정리됐다.</li>
</ul>
<hr />
<h1 id="다음에-할-일">다음에 할 일</h1>
<ul>
<li><input disabled="" type="checkbox" /> Section 11 마이크로서비스 간 통신 방식 학습</li>
<li><input disabled="" type="checkbox" /> 실제 프로젝트에 비대칭키 방식 적용해보기</li>
<li><input disabled="" type="checkbox" /> JCE 무제한 정책 파일 적용이 필요한 상황 직접 재현해보기</li>
</ul>
<hr />
<h1 id="💡-오늘의-회고">💡 오늘의 회고</h1>
<p>오늘은 &quot;설정을 어떻게 안전하고 효율적으로 다룰 것인가&quot;에 집중한 하루였다. Spring Cloud Bus는 서비스가 많아질수록 진가를 발휘하는 기능이라, 당장 눈에 보이는 서비스 세 개로는 체감이 크지 않았지만 수백 개 서비스를 운영한다고 상상하니 왜 필요한 기술인지 바로 이해가 됐다. 암호화 부분은 처음엔 단순히 &quot;비밀번호를 가려주는 기능&quot;이라고만 생각했는데, 실습을 하면서 암호문이 손상됐을 때 어떻게 실패하는지, 왜 비대칭키가 더 안전한지를 직접 겪어보니 훨씬 와닿았다. 특히 Git에는 암호문만 남고 서비스가 받는 순간에만 평문이 된다는 구조를 그림으로 직접 그려보면서, &quot;저장&quot;과 &quot;사용 시점&quot;을 분리하는 게 보안 설계의 핵심이라는 걸 체감했다.</p>