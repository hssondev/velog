<blockquote>
<p><strong>한 줄 요약</strong></p>
<p>여러 시스템이 서로 얽혀 연결되는 문제를 Kafka가 어떻게 중간 허브로 풀어주는지 개념부터 잡고, Docker로 Kafka 브로커 컨테이너를 띄운 뒤 그 안에서 Topic을 만들고 Producer/Consumer로 메시지를 주고받아봤다.</p>
</blockquote>
<hr />
<h1 id="📚-오늘-학습한-커리큘럼">📚 오늘 학습한 커리큘럼</h1>
<pre><code>Kafka가 왜 필요한가 (End-to-End 연결의 한계)
        ↓
Kafka 데이터 처리 흐름 (Producer/Consumer 분리)
        ↓
Kafka Broker, Cluster, Zookeeper 구조
        ↓
Docker로 Kafka 브로커 컨테이너 띄우기
        ↓
컨테이너 안에서 Topic 생성 &amp; 확인
        ↓
Producer/Consumer로 메시지 주고받기 테스트
        ↓
Kafka Client, Kafka Connect 생태계</code></pre><h1 id="📌-오늘의-학습-키워드">📌 오늘의 학습 키워드</h1>
<ul>
<li>Apache Kafka</li>
<li>Message Broker</li>
<li>Producer / Consumer</li>
<li>Topic / Partition</li>
<li>Broker / Kafka Cluster</li>
<li>Zookeeper / Controller</li>
<li>Kafka Connect</li>
</ul>
<hr />
<h1 id="0-개념-정리-키워드-사전">0. 개념 정리 (키워드 사전)</h1>
<blockquote>
<p><strong>한 줄 요약</strong></p>
<p>Kafka는 여러 시스템이 서로 직접 연결되는 대신 한 곳(Kafka)에만 데이터를 보내고 받아가게 만들어주는 &quot;중간 우체국&quot;이다.</p>
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
<td><strong>Apache Kafka</strong></td>
<td>링크드인에서 개발해 2011년 오픈소스화된, 실시간 데이터를 높은 처리량·낮은 지연으로 주고받는 메시지 브로커 플랫폼</td>
<td>Apple, Netflix, Kakao, Uber 등 대규모 트래픽을 다루는 기업에서 사용</td>
</tr>
<tr>
<td><strong>Producer</strong></td>
<td>Kafka로 메시지(데이터)를 보내는 쪽</td>
<td><code>kafka-console-producer</code></td>
</tr>
<tr>
<td><strong>Consumer</strong></td>
<td>Kafka에 쌓인 메시지를 가져가는 쪽</td>
<td><code>kafka-console-consumer</code></td>
</tr>
<tr>
<td><strong>Topic</strong></td>
<td>메시지가 쌓이는 논리적인 채널(우편함) 이름</td>
<td><code>--topic quickstart-events</code></td>
</tr>
<tr>
<td><strong>Broker</strong></td>
<td>실제로 실행된 Kafka 애플리케이션 서버 한 대</td>
<td>3대 이상을 묶어 Cluster로 구성</td>
</tr>
<tr>
<td><strong>Kafka Cluster</strong></td>
<td>여러 Broker를 묶은 집합. 장애가 나도 다른 Broker가 대신 처리</td>
<td><code>Broker #0, #1, #2</code></td>
</tr>
<tr>
<td><strong>Zookeeper</strong></td>
<td>Broker ID, Controller ID 같은 메타데이터를 저장·관리하는 역할</td>
<td>Kafka 서버보다 먼저 띄워야 함</td>
</tr>
<tr>
<td><strong>Controller</strong></td>
<td>Cluster 내 Broker 중 한 대가 맡는 역할. 각 Broker에 파티션을 할당하고 정상 동작을 모니터링</td>
<td>Zookeeper가 Controller 정보를 저장</td>
</tr>
<tr>
<td><strong>Kafka Connect</strong></td>
<td>코드 없이 Configuration만으로 다른 시스템과 데이터를 Import/Export하는 Kafka 생태계 도구</td>
<td>Source/Sink Connector로 Hive, S3, MySQL 등과 연동</td>
</tr>
</tbody></table>
<h3 id="코드로-다시-보기">코드로 다시 보기</h3>
<pre><code class="language-bash"># 0) Kafka 브로커 컨테이너 띄우기
docker run -d --name broker apache/kafka:latest

# 1) 컨테이너 안에서 Topic 생성
docker exec -it broker /opt/kafka/bin/kafka-topics.sh --create --topic quickstart-events --bootstrap-server localhost:9092 --partitions 1

# 2) Topic 목록/정보 확인
docker exec -it broker /opt/kafka/bin/kafka-topics.sh --bootstrap-server localhost:9092 --list
docker exec -it broker /opt/kafka/bin/kafka-topics.sh --describe --topic quickstart-events --bootstrap-server localhost:9092

# 3) 메시지 생산 (Producer)
docker exec -it broker /opt/kafka/bin/kafka-console-producer.sh --bootstrap-server localhost:9092 --topic quickstart-events
&gt;Hello
&gt;Hi, there

# 4) 메시지 소비 (Consumer)
docker exec -it broker /opt/kafka/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic quickstart-events --from-beginning
Hello
Hi, there</code></pre>
<hr />
<h1 id="1-kafka가-왜-필요한가--end-to-end-연결의-한계">1. Kafka가 왜 필요한가 — End-to-End 연결의 한계</h1>
<p>Kafka를 배우기 전에 먼저 &quot;왜 이런 게 필요한가&quot;부터 이해하고 넘어갔다. MySQL, Oracle, MongoDB 같은 여러 데이터 소스가 있고, 그 데이터를 Hadoop, 검색엔진, 모니터링, 이메일 시스템 등 여러 곳으로 보내야 한다고 하면, 가장 단순하게는 각 시스템끼리 전부 직접 연결하는 방식을 떠올릴 수 있다. 문제는 시스템이 하나씩 늘어날 때마다 연결선이 기하급수적으로 늘어난다는 점이다. 거기에 각 시스템마다 하드웨어, 운영체제, 장애 대응 방식이 다 달라서 연동 자체의 복잡성도 함께 커진다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/hssondev/post/77d87c5e-6a3d-4bc4-88c9-367ce9dca04c/image.png" /></p>
<p>Kafka는 이 문제를 &quot;모든 시스템이 Kafka 하나만 바라보게&quot; 만들어서 해결한다. 데이터를 보내는 쪽(MySQL, Oracle, MongoDB)은 Kafka에만 데이터를 보내면 되고, 데이터를 받는 쪽(Hadoop, 검색엔진, 모니터링)도 Kafka에서만 가져오면 된다. 새로운 시스템이 추가되더라도 기존 연결을 전혀 건드리지 않고 Kafka에 연결 하나만 추가하면 되기 때문에 확장이 훨씬 쉬워진다.</p>
<h1 id="2-kafka-데이터-처리-흐름--producer와-consumer의-분리">2. Kafka 데이터 처리 흐름 — Producer와 Consumer의 분리</h1>
<p>Kafka의 핵심 구조는 데이터를 &quot;보내는 역할(Producer)&quot;과 &quot;가져가는 역할(Consumer)&quot;을 완전히 분리해둔 것이다. Producer는 Kafka의 특정 Topic으로 메시지를 보내기만 하면 되고, 그 메시지를 누가 언제 가져가는지는 전혀 신경 쓸 필요가 없다. 반대로 Consumer는 자신이 구독하는 Topic에서 메시지를 꺼내 가기만 하면 된다. 이렇게 분리해두면 같은 메시지를 여러 Consumer가 동시에 가져가는 것도 가능하고(Pub/Sub 방식), 처리량이 늘어나면 Consumer나 Broker를 늘리는 식으로 수평 확장(Scale-out)도 가능해진다.</p>
<h1 id="3-kafka-broker-cluster-zookeeper-구조">3. Kafka Broker, Cluster, Zookeeper 구조</h1>
<p>실제로 실행되는 Kafka 애플리케이션 서버 한 대를 <strong>Broker</strong>라고 부르고, 안정적인 운영을 위해 보통 3대 이상의 Broker를 묶어 <strong>Cluster</strong>로 구성한다. 이때 Broker들끼리 &quot;누가 어떤 파티션을 담당하는지&quot; 같은 정보를 공유해야 하는데, 이 메타데이터(Broker ID, Controller ID 등)를 관리해주는 게 <strong>Zookeeper</strong>다. Zookeeper는 Cluster 안의 Broker 중 한 대를 <strong>Controller</strong>로 지정하는데, 이 Controller가 각 Broker에게 담당 파티션을 할당하고 Broker들이 정상적으로 동작하는지 모니터링하는 역할을 맡는다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/hssondev/post/f9cc1167-7984-4b47-8711-8aa19e68687b/image.png" /></p>
<p>처음엔 Zookeeper와 Kafka Broker가 뭐가 다른 역할인지 헷갈렸는데, 비유하자면 Broker들은 실제로 일하는 &quot;직원들&quot;이고 Zookeeper는 &quot;누가 어느 자리에 앉아 있는지 기록해두는 관리 대장&quot;에 가깝다는 느낌으로 이해하고 나니 구조가 명확해졌다.</p>
<h1 id="4-docker로-kafka-브로커-컨테이너-띄우기">4. Docker로 Kafka 브로커 컨테이너 띄우기</h1>
<p>우리는 Kafka를 로컬에 직접 설치하는 대신 Docker 이미지로 바로 띄우는 방식으로 실습했다. Docker Hub의 <code>apache/kafka</code> 공식 이미지를 받아서 컨테이너 하나만 실행하면 된다.</p>
<pre><code class="language-bash">docker run -d --name broker apache/kafka:latest</code></pre>
<p>여기서 처음엔 의아했던 부분이 있었는데, 분명 앞에서 Zookeeper가 Broker의 메타데이터를 관리한다고 배웠는데 정작 컨테이너는 <code>broker</code> 하나만 띄우고 Zookeeper 컨테이너는 따로 실행하지 않는다는 점이었다. 찾아보니 최신 버전의 Kafka(2.8+)는 <strong>KRaft(Kafka Raft) 모드</strong>를 지원해서, Zookeeper 없이도 Broker 스스로 메타데이터와 Controller 역할을 관리할 수 있다. <code>apache/kafka</code> 공식 이미지는 기본적으로 이 KRaft 모드로 동작하기 때문에 컨테이너 하나만으로도 정상적으로 동작하는 것이었다. 즉 Zookeeper가 하던 역할이 사라진 게 아니라, Broker 내부로 흡수됐다고 이해하면 맞았다.</p>
<h1 id="5-컨테이너-안에서-topic-생성--확인">5. 컨테이너 안에서 Topic 생성 &amp; 확인</h1>
<p>브로커 컨테이너가 떠 있는 상태에서, Kafka의 CLI 도구들은 컨테이너 내부(<code>/opt/kafka/bin/</code>)에 들어 있기 때문에 <code>docker exec</code>로 컨테이너 안에 들어가서 명령을 실행해야 한다.</p>
<pre><code class="language-bash"># Topic 생성
docker exec -it broker /opt/kafka/bin/kafka-topics.sh --create --topic quickstart-events --bootstrap-server localhost:9092 --partitions 1

# Topic 목록 확인
docker exec -it broker /opt/kafka/bin/kafka-topics.sh --bootstrap-server localhost:9092 --list

# Topic 상세 정보 확인
docker exec -it broker /opt/kafka/bin/kafka-topics.sh --describe --topic quickstart-events --bootstrap-server localhost:9092</code></pre>
<p>로컬에 직접 설치했다면 <code>$KAFKA_HOME/bin/</code>에 있는 스크립트를 바로 실행했겠지만, Docker로 띄운 경우엔 그 스크립트가 컨테이너 안에 있으니 <code>docker exec</code>를 앞에 붙여서 &quot;컨테이너 안에 들어가 이 명령을 실행해줘&quot;라고 요청하는 셈이라는 걸 알게 됐다.</p>
<h1 id="6-producerconsumer로-메시지-주고받기-테스트">6. Producer/Consumer로 메시지 주고받기 테스트</h1>
<p>Topic이 준비된 뒤, 역시 <code>docker exec</code>로 컨테이너 안의 Producer/Consumer 콘솔 도구를 실행해서 메시지를 주고받아봤다. 먼저 Producer 콘솔을 띄워 메시지를 입력하면,</p>
<pre><code class="language-bash">docker exec -it broker /opt/kafka/bin/kafka-console-producer.sh --bootstrap-server localhost:9092 --topic quickstart-events
&gt;Hello
&gt;Hi, there
&gt;It's working on Kafka cluster</code></pre>
<p>다른 터미널에서 Consumer 콘솔을 띄우고 <code>--from-beginning</code> 옵션을 주면, Topic에 쌓여 있던 메시지를 처음부터 모두 받아볼 수 있다.</p>
<pre><code class="language-bash">docker exec -it broker /opt/kafka/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic quickstart-events --from-beginning
Hello, World!
Hi, there.</code></pre>
<p>실습해보면서 와닿았던 부분은, Consumer를 나중에 띄워도 <code>--from-beginning</code>을 주면 그전에 보냈던 메시지까지 전부 받아올 수 있다는 점이었다. 즉 Kafka는 메시지를 Consumer가 가져가는 순간 바로 지워버리는 게 아니라, Topic에 일정 기간 그대로 보관해둔다는 걸 직접 확인할 수 있었다.</p>
<h1 id="7-kafka-client와-kafka-connect">7. Kafka Client와 Kafka Connect</h1>
<p>Kafka와 실제로 데이터를 주고받는 애플리케이션을 만들 때는 <code>kafka-clients</code>라는 Java 라이브러리를 사용한다. 이 라이브러리가 Producer, Consumer, Admin, Stream 같은 Kafka 관련 API를 제공하고, Java뿐 아니라 C/C++, Node.js, Python, .NET 같은 다양한 언어의 3rd party 라이브러리도 존재한다고 한다.</p>
<p>또 하나 인상 깊었던 건 <strong>Kafka Connect</strong>라는 생태계 도구였다. 코드를 한 줄도 작성하지 않고, Configuration 설정만으로 Source 시스템(Hive, JDBC 등)의 데이터를 Kafka를 거쳐 Target 시스템(S3 등)으로 옮길 수 있다는 개념이었다. Source Connector가 데이터를 Kafka Cluster로 가져오고, Sink Connector가 Kafka Cluster의 데이터를 Target 시스템으로 내보내는 구조인데, Standalone 모드와 Distribution 모드를 지원하고 RESTful API로 관리할 수 있다고 한다. 오늘은 개념만 훑었지만, 다음에 서비스 간 데이터 동기화를 실습할 때 이 Kafka Connect를 직접 써볼 예정이다.</p>
<hr />
<h1 id="✨-새롭게-알게-된-점">✨ 새롭게 알게 된 점</h1>
<ul>
<li>여러 시스템을 직접 연결하면 연결선이 기하급수적으로 늘어나는 문제를, Kafka가 &quot;모든 시스템이 Kafka 하나만 보게&quot; 만들어서 해결한다는 것.</li>
<li>Zookeeper와 Broker의 역할 차이 — Broker는 실제 메시지를 처리하는 서버, Zookeeper는 Broker들의 메타데이터를 관리하고 그중 한 대를 Controller로 지정하는 &quot;관리자&quot; 역할.</li>
<li>최신 Kafka는 KRaft 모드 덕분에 Zookeeper 없이 Broker 컨테이너 하나만으로도 동작한다는 것 — Zookeeper의 역할이 사라진 게 아니라 Broker 내부 기능으로 흡수된 것.</li>
<li>Kafka는 Consumer가 메시지를 가져가도 바로 삭제하지 않고 Topic에 보관해둔다는 점 — <code>--from-beginning</code>으로 직접 확인.</li>
<li>Kafka Connect를 쓰면 코드 없이 설정만으로 시스템 간 데이터 이동(Import/Export)이 가능하다는 것.</li>
</ul>
<hr />
<h1 id="🤔-헷갈렸던-점">🤔 헷갈렸던 점</h1>
<ul>
<li>분명 Zookeeper가 Broker의 메타데이터를 관리한다고 배웠는데, Docker로는 <code>broker</code> 컨테이너 하나만 띄우고 Zookeeper 컨테이너는 따로 실행하지 않아서 처음엔 당황했다. KRaft 모드에서는 Broker가 스스로 그 역할을 겸한다는 걸 알고 나서야 이해가 됐다.</li>
<li>Kafka CLI 명령어 앞에 매번 <code>docker exec -it broker</code>를 붙여야 하는 이유가 처음엔 헷갈렸는데, <code>kafka-topics.sh</code> 같은 스크립트 자체가 컨테이너 안에만 설치돼 있어서 &quot;컨테이너 안에 들어가 그 명령을 대신 실행해달라&quot;는 의미라는 걸 알고 나서 자연스럽게 이해됐다.</li>
</ul>
<hr />
<h1 id="다음에-할-일">다음에 할 일</h1>
<ul>
<li><input disabled="" type="checkbox" /> Kafka Producer/Consumer를 Spring Boot 코드로 직접 구현해보기</li>
<li><input disabled="" type="checkbox" /> Orders Service → Catalogs Service 데이터 동기화 실습 (Kafka Topic 활용)</li>
<li><input disabled="" type="checkbox" /> Kafka Connect의 Source/Sink Connector 직접 구성해보기</li>
<li><input disabled="" type="checkbox" /> Docker Compose로 Kafka + 여러 서비스 컨테이너를 한 번에 띄우는 방법도 알아보기</li>
</ul>
<hr />
<h1 id="💡-오늘의-회고">💡 오늘의 회고</h1>
<p>오늘은 Kafka라는 이름만 듣던 기술을 처음으로 Docker 컨테이너로 직접 띄우고 콘솔에서 메시지를 주고받아본 날이었다. 개념적으로는 &quot;메시지 브로커&quot;라는 말이 추상적으로 느껴졌는데, 여러 시스템이 서로 얽혀 연결되는 그림을 직접 그려보면서 왜 Kafka 같은 중간 허브가 필요한지 체감이 됐다. 특히 Producer로 보낸 메시지가 Consumer를 나중에 띄워도 처음부터 다 받아와지는 걸 직접 눈으로 보니, &quot;메시지가 큐에 쌓여 있다가 꺼내진다&quot;는 설명이 그제야 실감 났다. Zookeeper와 Broker, Controller 역할을 배워놓고도 막상 컨테이너는 하나만 뜬 걸 보고 당황했었는데, KRaft 모드라는 개념을 알고 나서 &quot;개념과 실제 구현이 항상 1:1로 대응하는 건 아니구나&quot;라는 걸 새삼 느꼈다. 다음에는 콘솔이 아니라 실제 Spring Boot 코드로 Producer/Consumer를 만들어보는 게 기대된다.</p>