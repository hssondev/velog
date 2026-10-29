<blockquote>
<p><strong>한 줄 요약</strong></p>
<p>VM 4대로 직접 쿠버네티스 클러스터를 구축하는 것부터 시작해서, 쿠버네티스가 왜 필요한지, Pod가 실제로 어떤 과정을 거쳐 떠오르는지, Controller와 Service 종류가 각각 어떤 문제를 푸는지, 그리고 MetalLB로 LoadBalancer를 실제로 동작시키는 법까지 커리큘럼 전체를 한 편으로 정리했다.</p>
</blockquote>
<hr />
<h1 id="📚-오늘-학습한-커리큘럼">📚 오늘 학습한 커리큘럼</h1>
<pre><code>VM 4대 환경 구성 (VMware, Ubuntu, Docker)
        ↓
kubeadm으로 싱글 쿠버네티스 클러스터 구축
        ↓
쿠버네티스란 무엇인가 (왜 필요한가, 아키텍처)
        ↓
kubectl 기본 명령어
        ↓
Pod 생성 절차 (API server → etcd → scheduler → kubelet)
        ↓
Controller (ReplicationController → ReplicaSet → Deployment → DaemonSet → StatefulSet)
        ↓
Service (ClusterIP → NodePort → LoadBalancer → ExternalName)
        ↓
Label과 Selector
        ↓
MetalLB로 LoadBalancer 실제 동작시키기</code></pre><h1 id="📌-오늘의-학습-키워드">📌 오늘의 학습 키워드</h1>
<ul>
<li>kubeadm / kubelet / kube-proxy</li>
<li>Control Plane (etcd, API server, scheduler, controller manager)</li>
<li>Pod / Deployment / ReplicaSet / DaemonSet / StatefulSet</li>
<li>Service (ClusterIP, NodePort, LoadBalancer, ExternalName)</li>
<li>Label / Selector / nodeSelector</li>
<li>MetalLB</li>
</ul>
<hr />
<h1 id="0-개념-정리-키워드-사전">0. 개념 정리 (키워드 사전)</h1>
<blockquote>
<p><strong>한 줄 요약</strong></p>
<p>쿠버네티스는 &quot;지휘소(Master)&quot;가 결정을 내리고 &quot;일꾼(Worker)&quot;이 실행하는 구조이고, Controller는 &quot;몇 개를 어떻게 유지할지&quot;, Service는 &quot;어떻게 접근하게 할지&quot;를 각각 책임진다.</p>
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
<td><strong>kubeadm</strong></td>
<td>쿠버네티스 클러스터를 초기화·구성해주는 설치 도구</td>
<td><code>kubeadm init</code>(마스터), <code>kubeadm join</code>(워커)</td>
</tr>
<tr>
<td><strong>etcd</strong></td>
<td>클러스터의 모든 상태를 저장하는 key-value 저장소</td>
<td>고가용성을 위해 여러 마스터에 분산, 주기적 백업 권장</td>
</tr>
<tr>
<td><strong>kube-apiserver</strong></td>
<td>모든 요청을 받아 검증하고 각 컴포넌트와 중계하는 창구</td>
<td>클러스터의 유일한 &quot;정문&quot;</td>
</tr>
<tr>
<td><strong>kube-scheduler</strong></td>
<td>새 Pod를 어느 노드에 배치할지 결정</td>
<td>리소스 여유, 조건 등을 고려</td>
</tr>
<tr>
<td><strong>kube-controller-manager</strong></td>
<td>Pod 개수를 원하는 상태로 유지, 장애 시 재배치</td>
<td>ReplicaSet/Deployment 동작의 근간</td>
</tr>
<tr>
<td><strong>kubelet</strong></td>
<td>각 워커 노드에서 동작하는 에이전트. 컨테이너 생성을 실제로 지시</td>
<td>노드마다 1개씩 상주</td>
</tr>
<tr>
<td><strong>kube-proxy</strong></td>
<td>워커 노드에서 네트워크 규칙(iptables)을 구성해 Service 요청을 실제 Pod로 전달</td>
<td>Service의 로드밸런싱을 구현하는 실무 담당자</td>
</tr>
<tr>
<td><strong>Pod</strong></td>
<td>쿠버네티스에서 배포하는 가장 작은 단위. 컨테이너를 감싼 껍데기</td>
<td>단독 Pod는 삭제되면 복구되지 않음</td>
</tr>
<tr>
<td><strong>Deployment</strong></td>
<td>ReplicaSet을 관리하며 롤링 업데이트·롤백까지 지원하는 컨트롤러</td>
<td>실무에서 가장 많이 쓰는 컨트롤러</td>
</tr>
<tr>
<td><strong>Service</strong></td>
<td>Pod의 IP가 바뀌어도 고정된 접근 지점을 제공</td>
<td>ClusterIP가 기본값</td>
</tr>
<tr>
<td><strong>Label / Selector</strong></td>
<td>리소스에 붙이는 key-value 표식(Label)과 그걸로 리소스를 골라내는 조건(Selector)</td>
<td>Deployment, Service가 모두 Selector로 Pod를 찾음</td>
</tr>
<tr>
<td><strong>MetalLB</strong></td>
<td>온프레미스(자체 서버) 환경에서 <code>LoadBalancer</code> 타입 Service에 외부 IP를 할당해주는 도구</td>
<td>클라우드가 아니라서 자체 로드밸런서가 필요할 때 사용</td>
</tr>
</tbody></table>
<h3 id="코드로-다시-보기">코드로 다시 보기</h3>
<pre><code class="language-bash"># 클러스터가 잘 구축됐는지 한 번에 확인하는 핵심 명령 모음
k get nodes                       # 노드 목록과 상태
k get pods -A                     # 전체 네임스페이스 Pod 확인
k get deploy,pods,svc             # Deployment/Pod/Service 한 번에 조회
k describe pod &lt;이름&gt;              # 상세 정보와 이벤트 로그
k apply -f &lt;파일&gt;.yaml             # YAML로 생성/업데이트
k delete -f &lt;파일&gt;.yaml            # YAML 기준으로 삭제
k label pod &lt;이름&gt; rel=prd         # Label 추가
k get pods -l rel=prd              # Selector로 조회</code></pre>
<hr />
<h1 id="1-vm-4대로-쿠버네티스-환경-직접-구축하기">1. VM 4대로 쿠버네티스 환경 직접 구축하기</h1>
<p>쿠버네티스를 배우기 전에, 먼저 그걸 올릴 서버 4대(Master 1 + Worker 3)부터 직접 만들었다. VMware에서 가상 네트워크(VMnet8, NAT, 서브넷 <code>10.0.2.0</code>)를 먼저 설정하고, Ubuntu 24.04를 설치한 뒤 Docker까지 깐 가상머신 하나를 원본(Master)으로 만들었다. 그다음 이 원본을 세 번 복제(Full Clone)해서 Worker1~3을 만들고, 각각 IP만 <code>10.0.2.11</code>, <code>10.0.2.12</code>, <code>10.0.2.13</code>으로 바꿔줬다. 서버 4대를 하나씩 새로 설치하는 대신 복제로 시간을 아끼는 방식이었다.</p>
<pre><code class="language-bash"># 모든 VM 공통 - Docker 설치
sudo -i
mv /home/ubuntu/install-docker.sh .
chmod +x install-docker.sh
./install-docker.sh</code></pre>
<p>복제 후에는 <code>/etc/hostname</code>과 <code>/etc/hosts</code>를 각 서버에 맞게 고쳐줘야 했는데, 처음엔 복제한 VM들이 전부 같은 호스트명을 가지고 있어서 서로 구분이 안 됐다. <code>/etc/hosts</code>에 4대의 IP와 호스트명을 전부 등록해두고 나서야 <code>ping k8s-worker1</code>처럼 이름으로 서로를 찾을 수 있게 됐다.</p>
<h1 id="2-kubeadm으로-쿠버네티스-클러스터-구축하기">2. kubeadm으로 쿠버네티스 클러스터 구축하기</h1>
<p>서버 4대가 준비된 뒤, 쿠버네티스 전용 설정을 모든 노드에 공통으로 적용했다. 핵심은 swap을 꺼야 한다는 점이었다(쿠버네티스는 메모리 리소스를 정확히 관리해야 해서 swap이 켜져 있으면 설치 자체가 막힌다), 방화벽을 끄고, 브리지 네트워크 커널 모듈을 올리고, kubelet/kubeadm/kubectl을 설치했다.</p>
<pre><code class="language-bash"># swap 해제 (모든 노드 공통)
swapoff -a &amp;&amp; sed -i '/swap/s/^/#/' /etc/fstab

# 방화벽 해제
ufw disable

# kubelet, kubeadm, kubectl 설치
sudo apt-get install -y kubelet kubeadm kubectl kubernetes-cni
sudo apt-mark hold kubelet kubeadm kubectl</code></pre>
<p>그다음 <strong>Master에서만</strong> <code>kubeadm init</code>으로 클러스터를 초기화했다. 이 명령이 끝나면 Worker가 합류할 때 쓸 <code>kubeadm join</code> 명령(토큰 포함)이 출력되는데, 이걸 파일로 저장해뒀다가 Worker 3대에서 그대로 실행했다.</p>
<pre><code class="language-bash"># Master에서
kubeadm init
# 출력된 kubeadm join 명령을 저장

# Worker 1~3에서 (Master가 출력한 값 사용)
kubeadm join 10.0.2.10:6443 --token b24253.tjjw8hp0rg7ux7m9 \
  --discovery-token-ca-cert-hash sha256:426ae3e113ce39df1e2a1587a09bb22e85cd0c4d69fbf4a95b204035bdb224c0</code></pre>
<p>처음엔 Worker의 <code>kubeadm join</code>이 실패하는 경우가 있었는데, 포트 충돌이나 이전 설정이 남아있는 게 원인이었다. 이럴 땐 <code>kubeadm reset</code>으로 해당 노드를 초기화하고 containerd를 재시작한 뒤 재부팅하고 다시 join하면 해결됐다. 그래도 반복되면 Master에서 토큰 자체를 새로 발급(<code>kubeadm token create</code>)해야 했는데, 토큰에는 유효시간이 있어서 시간이 지나면 만료된다는 걸 이때 알게 됐다.</p>
<p>마지막으로 Pod끼리 통신할 네트워크(CNI, Calico)와 <code>kubectl top</code> 같은 리소스 확인 명령에 필요한 metrics-server를 추가로 설치하고 나서야 클러스터가 완성됐다.</p>
<pre><code class="language-bash"># Master에서 - 네트워크(CNI)와 metrics-server 설치
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.31.7/manifests/calico.yaml
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# 확인
kubectl get nodes
kubectl get pods --all-namespaces</code></pre>
<h1 id="2-1-쿠버네티스는-왜-필요한가">2-1. 쿠버네티스는 왜 필요한가</h1>
<p>클러스터를 직접 만들어보고 나서 &quot;이게 왜 필요한지&quot;를 다시 짚었다. 마이크로서비스가 수십, 수백 개가 되면 사람이 일일이 &quot;이 컨테이너는 어느 서버에 띄우고, 장애가 나면 누가 재시작하고, 트래픽이 몰리면 몇 개를 더 띄울지&quot;를 판단하기 어려워진다. 쿠버네티스는 이걸 자동화해준다 — 컨테이너의 CPU/메모리 요구에 맞춰 노드에 알아서 배치(빈 패킹)하고, 컨테이너가 죽으면 자동으로 재시작하거나 교체(self-healing)하고, &quot;원하는 상태&quot;를 선언만 하면 지금 상태를 거기에 맞춰준다(선언적 관리).</p>
<pre><code>이미지 빌드 → 레지스트리 push → 서비스 정의(이미지, 복제본 수, 포트 등)
→ control plane이 배치할 노드 결정 → kubelet이 런타임에 실행 지시
→ 런타임이 이미지 pull 후 실행 → 선언된 상태와 지속적으로 비교/유지</code></pre><h1 id="3-쿠버네티스-아키텍처--지휘소와-일꾼">3. 쿠버네티스 아키텍처 — 지휘소와 일꾼</h1>
<p><img alt="" src="https://velog.velcdn.com/images/hssondev/post/44198f35-7aca-4cbe-9934-1ec43d602c4b/image.png" /></p>
<p>클러스터는 크게 두 종류의 노드로 나뉜다. <strong>Master 노드(Control Plane)</strong>는 클러스터의 &quot;지휘소&quot;로, etcd(상태 저장), kube-apiserver(요청 창구), kube-scheduler(배치 결정), kube-controller-manager(개수 유지)로 구성된다. <strong>Worker 노드</strong>는 &quot;일꾼&quot;으로, kubelet(지시받아 실행), kube-proxy(네트워크 규칙), 컨테이너 런타임(실제 컨테이너 실행)으로 구성된다. 처음엔 이 컴포넌트들의 이름이 비슷비슷해서 헷갈렸는데, &quot;결정은 Master가, 실행은 Worker가&quot;라는 한 문장으로 나누고 나니 역할이 명확해졌다.</p>
<h1 id="4-kubectl-기본-명령어">4. kubectl 기본 명령어</h1>
<pre><code class="language-bash">kubectl run &lt;이름&gt; --image=&lt;이미지&gt;     # Pod를 즉시 생성/실행
kubectl create -f &lt;파일&gt;.yaml          # YAML로 자원을 처음 생성
kubectl apply -f &lt;파일&gt;.yaml           # YAML로 생성하거나, 있으면 업데이트
kubectl get &lt;자원&gt;                     # 목록과 상태 조회
kubectl describe &lt;자원&gt; &lt;이름&gt;          # 상세 정보와 이벤트 로그
kubectl edit &lt;자원&gt; &lt;이름&gt;              # 실행 중인 설정을 에디터로 수정
kubectl delete &lt;자원&gt; &lt;이름&gt;            # 삭제</code></pre>
<p>실습 중엔 <code>~/.bashrc</code>에 <code>alias k=kubectl</code>과 자동완성을 등록해서 매번 <code>kubectl</code>을 다 치지 않고 <code>k</code>만으로 명령을 쓸 수 있게 해뒀다. <code>kg</code>(노드 조회), <code>kd</code>(describe), <code>ka</code>(apply -f), <code>kdel</code>(delete) 같은 단축 별칭도 함께 등록했는데, 교육 자료에는 익숙해지기 전까지는 원래 명령어로 먼저 연습하라는 주의도 함께 적혀 있었다 — 별칭에만 의존하면 실제 CKA 시험 환경에서 헷갈릴 수 있기 때문이다.</p>
<h1 id="5-pod-생성-절차--요청-하나가-거치는-긴-여정">5. Pod 생성 절차 — 요청 하나가 거치는 긴 여정</h1>
<p><code>kubectl run</code>이나 <code>kubectl apply</code> 한 줄을 입력했을 때 내부적으로 무슨 일이 일어나는지 단계별로 짚었다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/hssondev/post/d567796f-7619-404a-aed4-35b69466168d/image.png" /></p>
<ol>
<li>사용자가 kubectl로 Pod 생성을 요청한다</li>
<li>kube-apiserver가 요청의 유효성과 권한을 검사한다</li>
<li>검증된 요청이 etcd에 저장된다</li>
<li>apiserver가 kube-scheduler를 호출해 실행할 노드를 결정한다
5<del>6. 선택된 노드 정보가 다시 etcd에 기록된다
7</del>8. apiserver가 선택된 노드의 kubelet을 호출한다</li>
<li>kubelet이 컨테이너 런타임을 통해 실제 컨테이너를 생성한다
10~11. 생성 결과가 다시 kubelet → apiserver → etcd로 보고되어 최신 상태로 유지된다</li>
</ol>
<p>실제로는 Pod 상태가 <code>Pending → Creating → Running</code> 순서로 바뀌는데, <code>kubectl get pod --watch</code>로 이 과정을 실시간으로 지켜볼 수 있다. 이 절차를 직접 보고 나니 &quot;왜 etcd가 두 번이나 등장하는지&quot; 이해가 됐다 — 처음엔 &quot;요청이 들어왔다&quot;는 사실 자체를 저장하고, 나중엔 &quot;어느 노드에 배치하기로 했다&quot;는 결정까지 저장해서, Master가 재시작돼도 클러스터 상태를 잃어버리지 않게 하는 구조였다.</p>
<h1 id="6-controller--몇-개를-어떻게-유지할-것인가">6. Controller — 몇 개를 어떻게 유지할 것인가</h1>
<p><img alt="" src="https://velog.velcdn.com/images/hssondev/post/cabe44fd-03eb-45a1-afb4-3b3dad2c322c/image.png" /></p>
<p>단독 Pod는 지우면 끝이지만, Controller가 관리하는 Pod는 지워도 Controller가 다시 만들어낸다. 교육 과정에서는 Controller를 옛날 것부터 최신 것까지 순서대로 배웠다.</p>
<p><strong>ReplicationController(RC)</strong> 는 가장 단순한 형태로, Selector의 Label과 Pod Template의 Label이 일치해야만 그 Pod를 관리한다. 실습에서 Pod의 Label을 바꿔서 RC의 관리 대상에서 빠지면 어떻게 되는지 확인했는데, 처음엔 &quot;Pod 개수가 줄었다가 금방 다시 늘어나는&quot; 현상이 이상했다. 알고 보니 RC가 &quot;관리하던 Pod가 관리 대상에서 빠졌다&quot;고 판단해서 새 Pod를 하나 더 만들어낸 거였다 — 기존 Pod는 그대로 살아있지만 더 이상 RC 소속이 아니게 된 것이다.</p>
<p><strong>ReplicaSet(RS)</strong> 은 RC와 똑같은 역할을 하지만 Selector가 더 강력하다(<code>matchExpressions</code>로 In/NotIn/Exists 같은 조건을 쓸 수 있다). <code>--cascade=false</code> 옵션으로 RS만 삭제하면 Pod는 그대로 남아 &quot;단독 Pod&quot; 상태가 된다는 것도 실습으로 확인했다.</p>
<p><strong>Deployment</strong> 는 ReplicaSet을 한 단계 위에서 관리하며, 가장 중요한 건 <strong>롤링 업데이트</strong>다. 배포 전략을 세 가지로 비교했다.</p>
<table>
<thead>
<tr>
<th>전략</th>
<th>방식</th>
<th>특징</th>
</tr>
</thead>
<tbody><tr>
<td><strong>카나리아 배포</strong></td>
<td>일부 트래픽만 새 버전으로</td>
<td>오류를 미리 감지, 시간이 오래 걸림</td>
</tr>
<tr>
<td><strong>블루/그린 배포</strong></td>
<td>구/신버전을 동시 구축 후 순간 전환</td>
<td>빠른 롤백, 자원 2배 필요</td>
</tr>
<tr>
<td><strong>롤링 업데이트</strong></td>
<td>새 Pod 하나 생성→기존 Pod 하나 삭제 반복</td>
<td>자원 제한적일 때 적합, Deployment 기본값</td>
</tr>
</tbody></table>
<pre><code class="language-bash">kubectl set image deployment/&lt;이름&gt; &lt;컨테이너명&gt;=&lt;새 이미지&gt;
kubectl rollout history deployment/&lt;이름&gt;
kubectl rollout undo deployment/&lt;이름&gt;
kubectl rollout status deployment/&lt;이름&gt;</code></pre>
<p><strong>DaemonSet</strong> 은 Deployment와 전혀 다른 목적이다 — &quot;개수&quot;가 아니라 &quot;모든 노드마다 정확히 하나씩&quot;을 보장한다. 로그 수집기나 모니터링 에이전트처럼 노드마다 반드시 떠 있어야 하는 프로그램에 쓰인다. 워커가 새로 추가되면 자동으로 Pod가 생기고, 노드가 꺼지면 그 Pod도 함께 사라진다.</p>
<p><strong>StatefulSet</strong> 은 Pod의 <strong>이름과 스토리지를 고정</strong>한다. 보통 Pod는 재생성되면 이름이 바뀌는데(<code>nginx-7d8f9</code> → <code>nginx-4a2b1</code>), StatefulSet이 관리하는 Pod는 <code>sf-nginx-0</code>, <code>sf-nginx-1</code>처럼 이름이 고정되고 순서대로 생성·삭제된다. 데이터베이스처럼 &quot;이 Pod가 어떤 데이터를 가진 그 Pod인지&quot; 구분이 중요한 애플리케이션에 쓴다.</p>
<h1 id="7-service--어떻게-접근하게-할-것인가">7. Service — 어떻게 접근하게 할 것인가</h1>
<p>Pod는 재생성될 때마다 IP가 바뀐다. 그래서 Pod를 직접 IP로 호출하면 금방 깨진다 — 이 문제를 푸는 게 Service다. Service는 고정된 접근 지점을 만들고, kube-proxy가 그 요청을 실제 Pod로 전달(로드밸런싱 포함)해준다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/hssondev/post/c68f5ee0-da6b-4f40-bf6f-0f5d60720177/image.png" /></p>
<p>네 가지 타입을 배웠는데, 처음엔 &quot;넷 중에 뭘 써야 하나&quot; 헷갈렸지만 알고 보니 서로 대체 관계가 아니라 <strong>포함 관계</strong>였다. ClusterIP가 가장 안쪽이고, NodePort는 ClusterIP를 기반으로 외부에 포트를 더 열어주는 것이고, LoadBalancer는 다시 NodePort를 기반으로 외부 로드밸런서와 고정 IP를 얹는 것이다.</p>
<table>
<thead>
<tr>
<th>타입</th>
<th>접근 범위</th>
<th>예시</th>
</tr>
</thead>
<tbody><tr>
<td><strong>ClusterIP</strong> (기본값)</td>
<td>클러스터 내부 전용</td>
<td>WAS ↔ DB 내부 통신</td>
</tr>
<tr>
<td><strong>NodePort</strong></td>
<td><code>&lt;NodeIP&gt;:&lt;NodePort&gt;</code>로 외부 접근</td>
<td>임시로 외부에서 접속해 테스트할 때</td>
</tr>
<tr>
<td><strong>LoadBalancer</strong></td>
<td>외부 로드밸런서가 고정 External-IP 제공</td>
<td>운영 환경의 실제 서비스 노출 (MetalLB)</td>
</tr>
<tr>
<td><strong>ExternalName</strong></td>
<td>서비스 이름을 외부 도메인에 매핑</td>
<td>현업에서는 잘 쓰지 않는다고 함</td>
</tr>
</tbody></table>
<pre><code class="language-bash"># ClusterIP 테스트 (클러스터 내부 전용이라 port-forward로 확인)
kubectl apply -f nginx-clusterIp.yaml
kubectl port-forward svc/nginx-app-clusterip 8080:80
curl http://127.0.0.1:8080

# NodePort 테스트 (워커 노드 포트로 바로 접근 가능)
kubectl apply -f nginx-nodePort.yaml
curl http://127.0.0.1:30080</code></pre>
<p>추가로 <strong>헤드리스 서비스</strong>(<code>clusterIP: None</code>)라는 변형도 배웠다. 일반 Service는 가상 IP 하나로 로드밸런싱을 해주지만, 헤드리스 서비스는 가상 IP가 없고 DNS 조회 시 Pod들의 실제 IP를 그대로 돌려준다. StatefulSet처럼 &quot;어느 Pod인지 구분해서 직접 접속해야 하는&quot; 경우에 쓰는 이유를 이때 이해했다.</p>
<h1 id="8-label과-selector--리소스를-분류하고-골라내는-방법">8. Label과 Selector — 리소스를 분류하고 골라내는 방법</h1>
<p>Label은 리소스에 붙이는 key-value 표식이고, Selector는 그 Label을 조건으로 리소스를 골라내는 기능이다. Deployment도, Service도 내부적으로는 전부 Selector로 &quot;어떤 Pod를 관리할지/어디로 트래픽을 보낼지&quot;를 결정한다.</p>
<pre><code class="language-bash">kubectl label pod cmdpod rel=prd name=product   # Label 추가
kubectl label pod cmdpod rel=stg --overwrite    # 덮어쓰기
kubectl label pod cmdpod rel-                   # 키 삭제(끝에 '-')
kubectl get pods -l rel=dev                     # Selector로 조회
kubectl label nodes k8s-worker2 gpu=true         # 노드에도 Label 가능</code></pre>
<p>노드에 Label을 붙이고 <code>nodeSelector</code>로 특정 조건의 노드에만 Pod를 배치하는 실습도 했다.</p>
<pre><code class="language-yaml">spec:
  nodeSelector:
    gpu: &quot;true&quot;</code></pre>
<p>특히 흥미로웠던 건 <strong>Label을 이용한 카나리아 배포</strong>였다. <code>deploy-stable</code>(label: <code>version: stable</code>)과 <code>deploy-canary</code>(label: <code>version: canary</code>)를 각각 만들고, Service는 <code>app: mainui</code>라는 더 넓은 Label만 보고 두 Deployment의 Pod를 모두 연결한다. 그러면 각 Deployment의 Replica 수 비율이 곧 트래픽 비율이 된다 — 별도의 트래픽 분기 설정 없이 &quot;Pod 개수&quot;만으로 카나리아 배포를 구현하는 방식이라는 게 신선했다.</p>
<p>처음엔 Label과 Annotation이 뭐가 다른지 헷갈렸는데, Label은 Selector로 &quot;선택&quot;하는 데 쓰고 Annotation은 선택에는 전혀 쓰이지 않고 빌드 정보 같은 부가 메모를 남기는 용도라는 걸 구분하고 나서 정리가 됐다.</p>
<h1 id="9-metallb--온프레미스에서-loadbalancer-실제로-써보기">9. MetalLB — 온프레미스에서 LoadBalancer 실제로 써보기</h1>
<p>LoadBalancer 타입 Service는 보통 AWS, GCP 같은 클라우드의 로드밸런서를 자동으로 만들어주는데, 우리 실습 환경은 VMware 위의 자체 서버(온프레미스)라서 그런 클라우드 연동이 없다. LoadBalancer Service를 만들어도 외부 로드밸런서가 없으면 <code>EXTERNAL-IP</code>가 계속 <code>&lt;pending&gt;</code> 상태로 멈춰 있는데, 이 문제를 <strong>MetalLB</strong>가 해결해준다 — 쿠버네티스 클러스터 안에서 직접 로드밸런서 역할을 대신 해주는 도구다.</p>
<pre><code class="language-bash"># MetalLB 설치
kubectl apply -f metallb-native.yaml

# IP 대역(Pool)과 광고 방식 설정 적용
kubectl apply -f my-config.yaml

# LoadBalancer 타입 Service로 노출
kubectl run nginx --image=nginx --labels=app=nginx
kubectl expose pod nginx --name nginx-svc --port=80 --type=LoadBalancer
kubectl get svc</code></pre>
<p>여기서 중요했던 건 MetalLB에 할당해주는 IP 대역이 실제로 쓰이고 있는 네트워크 대역과 겹치면 안 된다는 점이었다. 우리는 VMware의 Host-Only 네트워크(<code>192.168.118.0/24</code>) 중 쓰지 않는 구간(<code>192.168.118.100~120</code>)을 골라서 <code>IPAddressPool</code>로 등록했다. 설정을 처음 적용했을 때 기존에 남아있던 webhook 설정과 충돌이 나서 적용이 안 됐는데, <code>kubectl delete validatingwebhookconfigurations metallb-webhook-configuration</code>으로 기존 webhook을 지우고 나서야 새 설정이 정상적으로 반영됐다. <code>kubectl get svc</code>로 확인했을 때 <code>EXTERNAL-IP</code>가 더 이상 <code>&lt;pending&gt;</code>이 아니라 실제 IP로 바뀌는 걸 보고서야 &quot;아, 이게 되는구나&quot; 싶었다.</p>
<hr />
<h1 id="✨-새롭게-알게-된-점">✨ 새롭게 알게 된 점</h1>
<ul>
<li>쿠버네티스 컴포넌트는 &quot;결정은 Master가, 실행은 Worker가&quot;라는 한 문장으로 구분하면 역할이 명확해진다는 것.</li>
<li>Pod 하나가 생성되는 과정에서 etcd가 두 번 등장하는 이유 — 요청 저장과 배치 결정 저장을 나눠서, Master가 재시작돼도 상태를 잃지 않게 하기 위함.</li>
<li>ClusterIP/NodePort/LoadBalancer가 서로 다른 선택지가 아니라, 바깥으로 갈수록 범위가 넓어지는 포함 관계라는 것.</li>
<li>Label만으로 카나리아 배포를 구현할 수 있다는 것 — Replica 수 비율이 곧 트래픽 비율이 되는 방식.</li>
<li>온프레미스 환경에서는 LoadBalancer Service가 자동으로 동작하지 않고, MetalLB 같은 도구가 그 역할을 대신 해줘야 한다는 것.</li>
</ul>
<hr />
<h1 id="🤔-헷갈렸던-점">🤔 헷갈렸던 점</h1>
<ul>
<li>ReplicationController가 관리하는 Pod의 Label을 바꿨더니 Pod 개수가 늘어나는 현상이 처음엔 이상했는데, 기존 Pod가 사라진 게 아니라 &quot;관리 대상에서 빠지면서 RC가 새 Pod를 추가로 만든 것&quot;이라는 걸 알고 나서 이해됐다.</li>
<li><code>kubeadm join</code>이 반복해서 실패했을 때, 단순히 명령을 다시 실행하는 게 아니라 <code>kubeadm reset</code>으로 노드를 초기화하고 containerd를 재시작한 뒤 재부팅까지 해야 한다는 순서를 몰라서 처음엔 헤맸다.</li>
<li>MetalLB의 IP 대역을 아무 값이나 넣으면 안 되고, VMware 네트워크 중 실제로 쓰이지 않는 대역을 직접 확인해서 골라야 한다는 점 — 네트워크 설정값이 왜 그렇게 정해졌는지 처음엔 바로 와닿지 않았다.</li>
</ul>
<hr />
<h1 id="다음에-할-일">다음에 할 일</h1>
<ul>
<li><input disabled="" type="checkbox" /> Deployment 롤링 업데이트 중 <code>maxSurge</code>/<code>maxUnavailable</code> 값을 직접 바꿔가며 동작 차이 확인</li>
<li><input disabled="" type="checkbox" /> StatefulSet + PV/PVC로 상태 있는 애플리케이션(MongoDB 등) 직접 구성해보기</li>
<li><input disabled="" type="checkbox" /> CKA 연습문제(taint, 노드 스케줄링 등) 추가로 풀어보기</li>
</ul>
<hr />
<h1 id="💡-오늘의-회고">💡 오늘의 회고</h1>
<p>이번 과정은 서버를 직접 설치하는 것부터 시작해서 그 위에 쿠버네티스를 올리고, Pod 하나가 뜨는 원리와 Controller·Service의 역할까지 쭉 이어지는 흐름이라 전체 그림을 한 번에 정리하는 게 의미가 컸다. 특히 Pod 생성 절차를 단계별로 그려보면서 &quot;kubectl 명령 한 줄&quot;이 실제로는 API server, etcd, scheduler, kubelet을 거치는 꽤 긴 여정이라는 걸 체감했고, Controller들을 옛날 것부터 순서대로 배우면서 왜 ReplicationController에서 ReplicaSet으로, 다시 Deployment로 발전해왔는지 그 맥락이 자연스럽게 이해됐다. Service 네 가지 타입이 서로 대체재가 아니라 포함 관계라는 걸 그림으로 그려보고 나서야 &quot;언제 뭘 써야 하는지&quot; 헷갈리지 않게 됐다. MetalLB까지 실제로 돌려서 LoadBalancer의 <code>&lt;pending&gt;</code>이 실제 IP로 바뀌는 걸 직접 본 게 이번 과정에서 가장 기억에 남는 순간이었다.</p>