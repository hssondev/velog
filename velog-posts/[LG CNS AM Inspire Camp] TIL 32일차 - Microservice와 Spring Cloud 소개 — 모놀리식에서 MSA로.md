<blockquote>
<p><strong>한 줄 요약</strong></p>
<p>모놀리식(Monolithic)과 마이크로서비스(MSA) 아키텍처가 뭐가 다른지, 서비스를 어떻게 쪼갤지 정하는 전략, 그리고 그렇게 쪼갠 서비스들을 운영 가능하게 만들어주는 MSA 표준 구성요소와 Spring Cloud 생태계의 큰 그림을 배운 날.</p>
</blockquote>
<hr />
<h1 id="📚-오늘-학습한-커리큘럼">📚 오늘 학습한 커리큘럼</h1>
<pre><code>Software Architecture란 무엇인가
        ↓
Cloud Native Architecture (확장성 · 탄력성 · 장애 격리)
        ↓
Monolithic vs Microservice 비교
        ↓
MSA 서비스 식별 전략 (어떻게 쪼갤 것인가)
        ↓
MSA 표준 구성요소 (API Gateway, Service Discovery, Service Mesh...)
        ↓
SOA vs MSA 차이
        ↓
RESTful API 설계 원칙
        ↓
Spring Cloud 생태계 소개</code></pre><h1 id="📌-오늘의-학습-키워드">📌 오늘의 학습 키워드</h1>
<ul>
<li>Monolithic vs Microservice</li>
<li>Cloud Native / 12 Factors</li>
<li>서비스 식별 전략 (하향식·상향식·부분분리)</li>
<li>API Gateway / Service Discovery / Service Mesh</li>
<li>SOA vs MSA</li>
<li>RESTful Web Service</li>
<li>Spring Cloud 생태계</li>
</ul>
<hr />
<h1 id="0-개념-정리-키워드-사전">0. 개념 정리 (키워드 사전)</h1>
<blockquote>
<p><strong>한 줄 요약</strong></p>
<p>오늘 배운 걸 한 문장으로 압축하면, &quot;서비스를 독립적으로 쪼개는 게 MSA이고, 쪼갠 서비스들이 서로 찾고·통신하고·배포되게 도와주는 공통 틀이 Spring Cloud다.&quot;</p>
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
<td><strong>Monolithic</strong></td>
<td>모든 업무 로직이 하나의 애플리케이션으로 패키징되어 서비스되는 구조</td>
<td>기능 하나만 고쳐도 전체를 다시 빌드·배포해야 함</td>
</tr>
<tr>
<td><strong>Microservice (MSA)</strong></td>
<td>작은 서비스들이 각자 프로세스로 실행되며 가벼운 통신(주로 HTTP API)으로 협력하는 아키텍처</td>
<td>Sam Newman: &quot;함께 동작하는 작은 자율적 서비스들(Small autonomous services that work together)&quot;</td>
</tr>
<tr>
<td><strong>Cloud Native</strong></td>
<td>수평 확장에 유연하고, 장애가 나도 다른 서비스에 영향 주지 않고, 변화에 빠르게 적응하는 설계 철학</td>
<td>CI/CD + DevOps + Microservices + Container가 네 기둥</td>
</tr>
<tr>
<td><strong>12 Factors</strong></td>
<td>클라우드 환경에서 잘 동작하는 앱을 만들기 위한 설계 원칙 모음 (Heroku가 정리)</td>
<td>예: 구성과 코드 분리, 상태 없는(stateless) 프로세스, 로그는 외부 시스템에서 관리</td>
</tr>
<tr>
<td><strong>API Gateway</strong></td>
<td>클라이언트 요청이 들어오는 단일 진입점. 라우팅·인증 같은 공통 처리를 담당</td>
<td>클라이언트는 내부에 서비스가 몇 개인지 몰라도 됨</td>
</tr>
<tr>
<td><strong>Service Discovery</strong></td>
<td>지금 이 순간 살아있는 서비스 인스턴스가 어디(IP·포트)에 있는지 찾아주는 장치</td>
<td>대표 구현체: Eureka</td>
</tr>
<tr>
<td><strong>Service Mesh</strong></td>
<td>API Gateway, Service Discovery, Load Balancing까지 포함해서 서비스들이 서로 통신하는 경로 전체를 가리키는 네트워크 계층</td>
<td>&quot;서비스 묶음&quot;이 아니라 &quot;서비스들이 통신하는 길&quot; 자체를 뜻하는 용어라는 걸 오늘 그림 보면서 이해함</td>
</tr>
<tr>
<td><strong>SOA</strong></td>
<td>비즈니스 측면의 서비스 재사용성에 집중, ESB라는 공통 채널로 서비스를 공유</td>
<td>MSA보다 상대적으로 &quot;큰&quot; 서비스 단위</td>
</tr>
</tbody></table>
<h3 id="코드로-다시-보기">코드로 다시 보기</h3>
<p>오늘은 실습 코드 대신 RESTful API를 설계할 때의 원칙을 다뤘다. 핵심만 압축하면 이렇다.</p>
<pre><code># 나쁜 예 (Level 0 — 동사 중심, SOAP 스타일을 REST로만 옮긴 것)
GET  /server/getPosts
POST /server/doThis

# 좋은 예 (Level 1~2 — 명사 + 복수형 + 올바른 HTTP 메서드)
GET    /users          # 목록 조회
GET    /users/10       # 10번 사용자 조회
POST   /users          # 사용자 생성
PUT    /users/10       # 10번 사용자 전체 수정
DELETE /users/10       # 10번 사용자 삭제</code></pre><hr />
<h1 id="1-software-architecture란-무엇인가">1. Software Architecture란 무엇인가</h1>
<p>오늘 강의는 바로 MSA로 들어가지 않고, &quot;애키텍처가 왜 필요한가&quot;부터 시작했다. 소프트웨어 아키텍처는 <strong>소프트웨어를 구성하는 요소와 요소 간의 관계</strong>를 정의한 것이다. 코드 자체는 눈에 안 보이는 지식 집합체라서, 이해관계자(기획자·개발자·운영자)마다 시스템을 이해하는 수준이 다 다르다. 그래서 &quot;관점(View)&quot;을 맞추기 위한 공통 언어가 아키텍처라는 설명이 와닿았다.</p>
<p>재미있었던 비유는 &quot;Architecture 스타일&quot;이었다. Peer to Peer 패턴(서비스끼리 직접 메시지를 주고받는 방식)과 Event-Bus 패턴(이벤트를 버스에 던지면 리스너가 받아가는 방식)처럼, 아키텍처 스타일은 &quot;이 문제를 어떻게 풀 것인가&quot;에 대한 접근 방향을 미리 제시해 준다는 점이었다.</p>
<h1 id="2-cloud-native-architecture--깨지지-않는-시스템에서-더-강해지는-시스템으로">2. Cloud Native Architecture — 깨지지 않는 시스템에서 더 강해지는 시스템으로</h1>
<p>IT 시스템의 역사를 Fragile(깨지기 쉬움) → Robust(견고함) → Resilient/Anti-Fragile(회복력/더 강해짐)로 정리한 슬라이드가 있었다. 여기서 Anti-Fragile이라는 개념이 특히 새로웠는데, 단순히 &quot;스트레스를 견디는 것&quot;을 넘어 <strong>스트레스를 받을수록 오히려 더 단단해지는 시스템</strong>을 뜻한다. 이를 구현하는 4가지 요소가 나왔다.</p>
<ul>
<li><strong>Auto scaling</strong>: 트래픽에 따라 서버 수를 자동으로 늘리고 줄인다.</li>
<li><strong>Microservices</strong>: 서비스를 잘게 쪼개서 한쪽이 죽어도 전체가 죽지 않게 한다.</li>
<li><strong>Chaos engineering</strong>: 일부러 장애를 주입해서(VM을 끄거나, 네트워크 지연을 만들거나) 시스템이 버티는지 미리 검증한다.</li>
<li><strong>Continuous deployments</strong>: 배포를 작고 빈번하게 해서, 문제가 생겨도 영향 범위를 최소화한다.</li>
</ul>
<p>이 네 가지가 합쳐진 것이 Cloud Native Application이고, 그 핵심 네 기둥이 <strong>CI/CD, DevOps, Microservices, Containers</strong>라고 정리됐다.</p>
<h1 id="3-monolithic-vs-microservice-비교">3. Monolithic vs Microservice 비교</h1>
<p>모놀리식은 모든 업무 로직이 하나의 애플리케이션으로 패키지되고, 데이터도 한 곳에 모여 참조되는 구조다. 반대로 MSA는 비즈니스 기능 단위로 서비스가 쪼개지고, 각 서비스가 자기 데이터 저장소까지 따로 가진다.</p>
<p>둘을 가장 체감 나게 비교한 자료가 Amazon 창업자 제프 베조스가 2002년에 전 직원에게 보낸 이메일이었다. 요약하면 &quot;모든 팀은 서비스 인터페이스로만 데이터와 기능을 노출하고, 다른 방식의 직접 통신(DB 직접 읽기, 공유 메모리 등)은 전부 금지. 어떤 기술을 쓰든 상관없고, 이걸 안 지키는 사람은 해고&quot;라는 내용인데, 이게 사실상 MSA 원칙 그 자체였다는 게 인상 깊었다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/hssondev/post/f0b06df3-f48b-49df-acbd-fbc3da3c7ee3/image.png" /></p>
<p>모놀리식의 장점(쉬운 디버깅, 쉬운 배포, 성능)과 단점(강한 결합, 확장 어려움)은 그대로 MSA의 단점과 장점으로 뒤집힌다. 즉 어느 한쪽이 항상 옳은 게 아니라 <strong>상황에 맞는 트레이드오프</strong>라는 게 오늘 강의의 결론이었다. &quot;Everything should be a microservice&quot;라는 문장에 취소선이 그어져 있던 게 그 메시지를 잘 보여줬다.</p>
<h1 id="4-msa-서비스-식별-전략--어떻게-쪼갤-것인가">4. MSA 서비스 식별 전략 — 어떻게 쪼갤 것인가</h1>
<p>MSA로 가기로 했다면 &quot;어디서부터 어디까지가 한 서비스인가&quot;를 정해야 한다. 오늘 배운 접근은 크게 세 가지였다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/hssondev/post/76bcb06d-9da4-4ec2-93a0-bf9c65d35a48/image.png" /></p>
<ul>
<li><strong>하향식(Top-down) 접근</strong>: 업무의 중요도·형태(핵심/부가, 온라인/배치)를 기준으로 위에서부터 나눈다. 구조가 깔끔하게 나오지만, 실제 데이터가 서비스 간에 어떻게 얽혀 있는지를 놓치기 쉽다.</li>
<li><strong>상향식(Bottom-up) 접근</strong>: 데이터 간의 연관성(Ownership)부터 보고 아래에서부터 서비스를 묶어 올라간다. 실제 쓰임새에 더 가깝게 경계를 잡을 수 있다.</li>
<li><strong>서비스 일부 분리</strong>: 이미 있는 서비스에서 트래픽이 몰리거나 자주 바뀌는 기능 하나만 떼어내는 방식. 처음부터 완벽하게 쪼개기보다, 필요한 부분부터 점진적으로 분리할 때 현실적으로 많이 쓰인다고 한다.</li>
</ul>
<p>여기서 Eric Evans의 &quot;Domain Driven Design&quot;에 나오는 <strong>Bounded Context</strong>(하나의 서비스에 담을 수 있는 기능들의 경계)와 <strong>Ubiquitous Language</strong>(같은 업무를 하는 사람들이 쓰는 공통 용어) 개념이 같이 소개됐는데, 서비스 경계를 &quot;기술적으로&quot; 나누는 게 아니라 &quot;업무 단위로&quot; 나눠야 한다는 포인트로 이해했다.</p>
<h1 id="5-msa-표준-구성요소--service-mesh는-서비스-모음이-아니라-네트워크-계층">5. MSA 표준 구성요소 — Service Mesh는 &quot;서비스 모음&quot;이 아니라 &quot;네트워크 계층&quot;</h1>
<p>오늘 수업에서 가장 헷갈렸다가 가장 명확해진 부분이다. 강의 자료에는 External Gateway, Service Mesh, Runtime Platform, Backing Services, CI/CD Automation으로 이루어진 MSA 표준 구성도가 나왔는데, 처음엔 &quot;Service Mesh&quot;라는 이름 때문에 이것도 서비스들의 묶음인 줄 알았다. 그런데 설명을 들어보니 Service Mesh는 <strong>서비스들이 서로 통신하는 경로, 즉 네트워크 계층 자체</strong>를 가리키는 말이었다. 그래서 수업 중 자료에 직접 &quot;Service Mesh = Network&quot;라고 메모해 뒀다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/hssondev/post/144484f5-7428-482c-abf2-cb272bd80d95/image.png" /></p>
<p>흐름을 정리하면 이렇다.</p>
<ol>
<li>클라이언트 요청은 <strong>API Gateway</strong>(단일 진입점)로 먼저 들어온다.</li>
<li><strong>Service Discovery</strong>가 &quot;지금 이 서비스의 살아있는 인스턴스가 어디 있는지&quot;를 찾아준다.</li>
<li><strong>Load Balancing</strong>이 찾아낸 여러 인스턴스 중 어디로 보낼지 분산한다.</li>
<li>실제 요청을 처리한 서비스는 <strong>Backing Services</strong>(DB, 메시지 큐 등)에 접근한다.</li>
<li>이 모든 통신 경로에서 발생하는 지표는 <strong>Telemetry</strong>(모니터링·장애 추적)로 모인다.</li>
</ol>
<p>이 1~4번 경로 전체를 감싸는 게 Service Mesh이고, 이 인프라를 자동으로 만들고 운영할 수 있게 해주는 게 CI/CD Automation이라는 구조다.</p>
<h1 id="6-soa-vs-msa--둘-다-서비스인데-뭐가-다른가">6. SOA vs MSA — 둘 다 &quot;서비스&quot;인데 뭐가 다른가</h1>
<p>SOA(Service Oriented Architecture)와 MSA는 둘 다 &quot;서비스&quot;라는 단어를 쓰고 있어서 처음엔 차이가 잘 안 와닿았는데, 오늘 비교 슬라이드를 보고 정리됐다.</p>
<ul>
<li><strong>SOA</strong>: 비즈니스 측면에서의 서비스 <strong>재사용성</strong>에 집중한다. ESB(Enterprise Service Bus)라는 공통 채널에 서비스를 모아두고, 여러 팀이 그걸 공유해서 재사용한다. → 서비스 공유를 최대화하는 방향.</li>
<li><strong>MSA</strong>: 작은 서비스 하나에 집중한다. 서비스끼리 공유하지 않고 독립적으로 실행된다. → 서비스 공유를 최소화해서, 서비스 간 결합도를 낮추고 변화에 능동적으로 대응하는 방향.</li>
</ul>
<p>기술 방식으로 보면 SOA는 WSDL/UDDI/ESB 같은 미들웨어를 통해 Web Service(SOAP)로 서비스를 공유하고, MSA는 각 서비스가 노출한 REST API를 직접 호출한다는 차이도 있었다.</p>
<h1 id="7-restful-web-service-설계-원칙">7. RESTful Web Service 설계 원칙</h1>
<p>Leonard Richardson의 REST 성숙도 모델(Level 0~3)을 기준으로, &quot;그냥 REST처럼 보이는 API&quot;와 &quot;진짜 REST다운 API&quot;의 차이를 짚었다. Level 0은 SOAP 스타일을 URL만 바꾼 것(<code>/server/getPosts</code>), Level 1은 리소스에 맞는 URI를 쓰는 것(<code>/accounts/10</code>), Level 2는 여기에 HTTP 메서드(GET/POST/PUT/DELETE)를 제대로 활용하는 것이다.</p>
<p>실무 팁으로 몇 가지가 더 나왔다. URI에는 리소스를 <strong>명사의 복수형</strong>으로 쓰고(<code>/users</code>가 <code>/user</code>보다 선호됨), URI에 보안 정보를 절대 넣지 말고, 예외 상황에 대한 URL 패턴도 팀 안에서 일관되게 정해둬야 한다는 점이었다.</p>
<h1 id="8-spring-cloud-생태계-소개">8. Spring Cloud 생태계 소개</h1>
<p>분산 시스템을 만들다 보면 설정 관리, 서비스 탐색, 로드 밸런싱, 장애 복원력(Circuit Breaker) 같은 비슷한 문제가 매 프로젝트마다 반복된다. Spring Cloud는 이런 반복되는 패턴(boilerplate pattern)을 미리 구현해 둔 프로젝트 모음이다.</p>
<p>오늘 소개된 핵심 구성요소는 이렇게 매칭됐다.</p>
<ul>
<li><strong>중앙 설정 관리</strong>: Spring Cloud Config Server</li>
<li><strong>위치 투명성(서비스 찾기)</strong>: Naming Server (Eureka)</li>
<li><strong>부하 분산</strong>: Ribbon(클라이언트 사이드), Spring Cloud Gateway</li>
<li><strong>REST 호출 간편화</strong>: FeignClient</li>
<li><strong>모니터링/추적</strong>: Zipkin Distributed Tracing</li>
<li><strong>장애 복원력</strong>: Hystrix</li>
</ul>
<p>이 중 상당수(Ribbon, Hystrix, Zookeeper 기반 구성 등)는 이제 Netflix OSS가 유지보수를 중단하면서 Kubernetes 기반 대안(Service Discovery는 K8s Service, Config는 ConfigMap/Secret 등)으로 옮겨가는 추세라는 설명도 있었다. 그래서 &quot;Spring Cloud vs Kubernetes&quot;를 완전히 대립 관계가 아니라, Kubernetes가 인프라 레벨을 담당하고 Spring Cloud가 애플리케이션 레벨의 설정·로직을 담당하는 <strong>보완 관계</strong>로 이해하는 게 맞다는 정리가 특히 도움이 됐다.</p>
<hr />
<h1 id="✨-새롭게-알게-된-점">✨ 새롭게 알게 된 점</h1>
<ul>
<li>MSA가 무조건 좋은 선택이 아니라는 것. 네트워크 지연, 여러 서비스 간 데이터 일관성, 분산 트랜잭션 관리 같은 비용이 따라오기 때문에, 모놀리식의 장점(쉬운 디버깅·배포)이 더 중요한 상황도 있다.</li>
<li>Service Mesh가 &quot;서비스들의 집합&quot;이 아니라 &quot;서비스들이 통신하는 경로 자체&quot;를 가리키는 용어라는 것. API Gateway, Service Discovery, Load Balancing을 하나로 묶어서 부르는 개념이라는 걸 그림으로 직접 그려보고서야 이해됐다.</li>
<li>2002년 제프 베조스의 사내 이메일이 사실상 오늘날 MSA 원칙의 원형이라는 것. &quot;서비스 인터페이스로만 소통하라&quot;는 규칙이 20년 넘게 이어져 온 아이디어라는 게 흥미로웠다.</li>
</ul>
<hr />
<h1 id="🤔-헷갈렸던-점">🤔 헷갈렸던 점</h1>
<ul>
<li>API Gateway, Service Discovery, Load Balancing, Service Mesh가 서로 뭐가 다른지 처음엔 다 비슷해 보였다. 정리하고 보니 API Gateway는 &quot;입구&quot;, Service Discovery는 &quot;주소록&quot;, Load Balancing은 &quot;교통정리&quot;, Service Mesh는 이 전체를 아우르는 &quot;통신 인프라(네트워크 계층)&quot;라는 역할 구분으로 이해됐다.</li>
<li>SOA와 MSA 둘 다 &quot;서비스 지향&quot;이라는 말이 붙어서 헷갈렸는데, &quot;서비스를 공유해서 재사용할 것이냐(SOA) vs 공유하지 않고 독립적으로 실행할 것이냐(MSA)&quot;라는 기준으로 구분하니 명확해졌다.</li>
</ul>
<hr />
<h1 id="💡-오늘의-회고">💡 오늘의 회고</h1>
<p>오늘은 코드를 거의 치지 않고 개념만 쭉 들었는데도 생각보다 머리를 많이 쓴 하루였다. 그동안 &quot;마이크로서비스&quot;라는 말을 막연하게 &quot;작은 서비스들로 나누는 것&quot; 정도로만 알고 있었는데, 왜 나누는지(독립 배포, 장애 격리), 어떻게 나누는지(하향식/상향식/부분분리), 나눈 다음엔 어떻게 서로 찾고 통신하는지(API Gateway, Service Discovery, Service Mesh)까지 하나의 흐름으로 이어지니까 훨씬 또렷해졌다. 특히 Service Mesh 부분은 용어만 보고 오해하고 있었는데, 직접 그림을 그려보면서 &quot;아, 이게 서비스 묶음이 아니라 통신 경로구나&quot;를 스스로 이해한 게 오늘 가장 큰 수확이었다. 다음 Section부터는 Service Discovery와 API Gateway를 직접 코드로 띄워본다고 하니, 오늘 배운 개념도를 다시 펼쳐두고 실습과 연결해서 복습해야겠다.</p>