<blockquote>
<p><strong>한 줄 요약</strong></p>
<p>실습 문제·수업 자료·&quot;제일 많이 쓰는 명령어&quot; 노트에 나온 kubectl 명령어들을 전부 모아, 주제별로 목차를 달아 정리했다.</p>
</blockquote>
<hr />
<h1 id="📋-목차">📋 목차</h1>
<ol>
<li><a href="https://api.velog.io/rss/@hssondev#1-%EA%B8%B0%EB%B3%B8-%EC%A1%B0%ED%9A%8C-pods--nodes--%EC%A0%84%EC%B2%B4-%EB%A6%AC%EC%86%8C%EC%8A%A4">기본 조회 (pods / nodes / 전체 리소스)</a></li>
<li><a href="https://api.velog.io/rss/@hssondev#2-poddeployment-%EC%83%9D%EC%84%B1-%EB%B0%A9%EB%B2%95-run--create--apply">Pod/Deployment 생성 방법 (run / create / apply)</a></li>
<li><a href="https://api.velog.io/rss/@hssondev#3-deployment--%EC%8A%A4%EC%BC%80%EC%9D%BC%EB%A7%81--yaml-%EC%B6%94%EC%B6%9C">Deployment — 스케일링 &amp; YAML 추출</a></li>
<li><a href="https://api.velog.io/rss/@hssondev#4-replicationcontroller--replicaset">ReplicationController / ReplicaSet</a></li>
<li><a href="https://api.velog.io/rss/@hssondev#5-daemonset">DaemonSet</a></li>
<li><a href="https://api.velog.io/rss/@hssondev#6-service--endpoints">Service &amp; Endpoints</a></li>
<li><a href="https://api.velog.io/rss/@hssondev#7-label--selector--nodeselector">Label / Selector / nodeSelector</a></li>
<li><a href="https://api.velog.io/rss/@hssondev#8-namespace">Namespace</a></li>
<li><a href="https://api.velog.io/rss/@hssondev#9-%EB%A1%9C%EA%B7%B8--exec--%EB%94%94%EB%B2%84%EA%B9%85">로그 / exec / 디버깅</a></li>
<li><a href="https://api.velog.io/rss/@hssondev#10-%EB%85%B8%EB%93%9C-%EA%B4%80%EB%A6%AC-cordon--drain--uncordon">노드 관리 (cordon / drain / uncordon)</a></li>
<li><a href="https://api.velog.io/rss/@hssondev#11-kubeadm-%ED%86%A0%ED%81%B0--%EC%B4%88%EA%B8%B0%ED%99%94">kubeadm (토큰 / 초기화)</a></li>
<li><a href="https://api.velog.io/rss/@hssondev#12-pv--pvc-%EC%8A%A4%ED%86%A0%EB%A6%AC%EC%A7%80">PV / PVC (스토리지)</a></li>
<li><a href="https://api.velog.io/rss/@hssondev#13-%EA%B8%B0%ED%83%80-port-forward--events--top--docker">기타 (port-forward / events / top / docker)</a></li>
</ol>
<hr />
<h1 id="1-기본-조회-pods--nodes--전체-리소스">1. 기본 조회 (pods / nodes / 전체 리소스)</h1>
<pre><code class="language-bash">kubectl get pods                          # 현재 네임스페이스 pod 조회 (= kubectl get po)
kubectl get pods -o wide                  # pod 상세정보(노드, IP 등) 조회
kubectl get pods --all-namespaces         # 모든 네임스페이스의 pod 리스트 (= -A)
kubectl get pods --show-labels            # pod 의 label 정보까지 조회
kubectl get pods -L app,type              # 특정 label(app, type)의 값을 컬럼으로 보여줌
kubectl get pods --sort-by=.metadata.creationTimestamp   # 생성 시간순 정렬

kubectl get nodes                         # node 상태 확인
kubectl get nodes -o wide                 # node 상세정보 확인
kubectl get nodes --show-labels           # node 의 label 정보 확인
kubectl get nodes -L disk,name            # node 의 특정 label 값 컬럼 조회
kubectl get nodes --selector ssd=true     # label 조건(ssd=true)에 맞는 node만 조회
kubectl describe nodes k8s-worker1        # worker1 노드의 상세(full) 정보 확인

kubectl get all                           # pod/service/deployment/replicaset 등 한 번에 조회
kubectl get pods,deploy,replicaset,service   # 원하는 리소스 종류만 골라서 한 번에 조회
kubectl get rc                            # replicationcontroller 조회
kubectl get rs                            # replicaset 조회
kubectl get deploy                        # deployment 조회 (= kubectl get deployment)

watch kubectl get pods -o wide            # 2초 간격으로 pod 상태 자동 갱신하며 모니터링
kubectl get pods -o wide --watch          # 변화가 생길 때마다 결과를 이어서(append) 보여줌 (watch 명령과 차이점)
kubectl get pods --watch                  # pod 상태 변화(Pending→Running 등)를 단계별로 확인

kubectl get events                        # 클러스터 이벤트 로그 확인
kubectl get pods | wc -l                  # pod 개수 세기 (namespace/node 등도 동일 패턴)
kubectl get nodes | wc -l                 # node 개수 세기</code></pre>
<hr />
<h1 id="2-poddeployment-생성-방법-run--create--apply">2. Pod/Deployment 생성 방법 (run / create / apply)</h1>
<p>Pod를 만드는 방법은 크게 세 가지다 — ① <code>kubectl run</code>(단독 Pod), ② <code>kubectl create deploy</code>(Deployment로 생성, 복구 보장), ③ 기존 YAML로 <code>kubectl apply -f</code>.</p>
<pre><code class="language-bash">kubectl run web2 --image=nginx                         # 단독 Pod 즉시 생성
kubectl run web2 --image=nginx --dry-run=client         # 실제 생성 없이 문법만 검증
kubectl run web2 --image=nginx -o yaml --dry-run=client &gt; web2.yaml   # YAML만 추출(생성 X)

kubectl create deployment demo-deployment --image=nginx:1.14                       # Deployment 생성
kubectl create deployment demo-deployment --image=nginx:1.14 --replicas=5          # 복제본 5개로 생성
kubectl create deploy nginx-web --image=nginx:1.14 --replicas=3 --dry-run=client -o yaml &gt; nginx-web.yaml
# ↑ 실제로 만들지 않고 YAML 파일만 뽑아내는 CKA 실전 패턴 (문제 조건 그대로 옵션에 반영)

kubectl apply -f obj.yaml                 # yaml 파일로 pod/deployment/service 등 생성·갱신
kubectl delete -f obj.yaml                # 같은 yaml 파일로 만든 리소스 삭제
kubectl create -f pod.yaml                # yaml로 처음 생성 (이미 있으면 에러)</code></pre>
<p><strong>단독 Pod vs Deployment Pod 차이</strong>: 단독 Pod는 삭제되면 기본적으로 복구가 안 되지만, Deployment가 관리하는 Pod는 삭제돼도 Desired 개수를 맞추기 위해 자동으로 다시 생성된다.</p>
<p>컨테이너 설정을 바꿔야 할 때는 두 가지 방법이 있다: ① YAML을 고치고 <code>kubectl apply -f</code>로 반영(기존 Deployment 삭제가 필요할 수 있음), ② <code>kubectl edit deploy &lt;이름&gt;</code>으로 실행 중인 설정을 바로 수정(<code>:wq</code>로 저장하면 즉시 반영).</p>
<pre><code class="language-bash">kubectl edit deploy webserver              # 실행 중인 deployment를 에디터로 즉시 수정
kubectl edit replicationcontroller nginx-rc  # RC도 동일하게 바로 수정 가능</code></pre>
<hr />
<h1 id="3-deployment--스케일링--yaml-추출">3. Deployment — 스케일링 &amp; YAML 추출</h1>
<pre><code class="language-bash">kubectl scale deployment demo-deployment --replicas=10   # replica 개수를 10개로 변경
kubectl scale deployment nginx-web2 --replicas=5          # 복제본 5개로 수정

kubectl get deployment.apps demo-deployment -o yaml                      # 현재 상태를 yaml로 확인
kubectl get deployment.apps demo-deployment -o yaml &gt; demo-deployment.yaml   # 파일로 저장(백업/복제)

kubectl delete deployments.apps demo-deployment           # deployment 삭제 (= kubectl delete deploy)
kubectl delete --all all                                   # 현재 네임스페이스의 모든 리소스 삭제</code></pre>
<blockquote>
<p>운영 중인 Pod/Deployment에서 YAML을 뽑아낼 수는 있지만, 이때 필요 없는 옵션이 잔뜩 자동으로 붙는다. 그래서 가능하면 처음부터 <code>--dry-run=client -o yaml</code>로 깨끗한 YAML을 만들어 쓰는 게 좋다.</p>
</blockquote>
<pre><code class="language-bash">kubectl get pods web-pod -o yaml &gt; web-pod.yaml
kubectl run web-pod --image=nginx:1.14 -o yaml --dry-run=client &gt; web-pod.yaml   # 깨끗한 버전
kubectl create deploy web-deploy -o yaml --dry-run=client &gt; web-deploy.yaml      # 깨끗한 버전

kubectl get pods web3 -o json | grep -i podip   # json 출력에서 특정 필드(Pod IP)만 grep으로 뽑기</code></pre>
<hr />
<h1 id="4-replicationcontroller--replicaset">4. ReplicationController / ReplicaSet</h1>
<pre><code class="language-bash">kubectl get rc                                 # replicationcontroller 조회
kubectl describe rc myapp                       # myapp RC의 상세(full) 정보
kubectl scale rc myapp-rc --replicas=5          # RC의 replica 개수 변경
kubectl edit rc myapp-rc                        # RC 설정을 바로 수정
kubectl delete replictioncontroller nginx-rc     # RC 삭제

kubectl get rs                                   # replicaset 조회
kubectl describe pod rs-nginx-XXXX                # RS가 만든 특정 Pod의 상세정보
kubectl delete rs demo-deployment-6d8dbd56d7       # 특정 replicaset 삭제
kubectl delete rs myapp-rs                         # replicaset 삭제</code></pre>
<p>Label을 바꿔서 RC/RS의 관리 대상에서 빠지는 실습도 자주 나온다 — Pod의 label을 바꾸면 기존 Pod는 관리 대상에서 빠지고, 컨트롤러는 &quot;개수가 모자르다&quot;고 판단해 새 Pod를 추가로 만든다.</p>
<pre><code class="language-bash">kubectl label pod myapp-rc-XXX app=nginx --overwrite   # Pod label 변경 → RC 관리 대상에서 분리</code></pre>
<hr />
<h1 id="5-daemonset">5. DaemonSet</h1>
<pre><code class="language-bash">kubectl get daemonsets.apps -n kube-system                 # kube-system의 daemonset 조회
kubectl get daemonsets.apps                                 # daemonset 전체 조회
kubectl edit daemonsets.apps fluentd                         # fluentd(로그수집기) daemonset 수정
kubectl rollout history daemonset                             # daemonset 변경 이력 확인
kubectl rollout status daemonset fluentd                      # fluentd daemonset의 롤아웃 진행 상태
kubectl rollout undo daemonset fluentd                         # 바로 직전 버전으로 롤백
kubectl delete daemonsets.apps fluentd                          # daemonset 삭제

kubectl get pods -n kube-system -o wide | grep -e proxy -e weave
# kube-system 네임스페이스에서 이름에 proxy 또는 weave가 들어간 pod만 필터링해서 보기</code></pre>
<hr />
<h1 id="6-service--endpoints">6. Service &amp; Endpoints</h1>
<pre><code class="language-bash">kubectl expose deploy nginx --type=ClusterIP --port 80
kubectl expose deploy nginx --type=NodePort --port 80
kubectl expose deploy nginx --type=LoadBalancer --port 80 --target-port=80
# nginx deployment를 80포트로 외부에 노출 (타입에 따라 접근 범위가 달라짐)

kubectl expose pod web-pod --name pod-svc --port=80 --target-port=80 --type=LoadBalancer
kubectl expose deploy web-deploy --name deploy-svc --port=80 --target-port=80 --type=LoadBalancer
# 운영 중인 pod/deployment를 그대로 이용해 Service를 생성 (CKA 시험 자주 나오는 패턴)

kubectl get svc                              # service 목록 확인
kubectl describe svc appjs-service            # 특정 service의 full 정보 확인
kubectl get endpoints                          # service가 실제로 연결된 pod IP(엔드포인트) 확인
kubectl edit service ev237                      # service 설정 바로 수정
kubectl delete svc my-first-deploy               # service 삭제

iptables -t nat -S | grep 80                 # NAT 테이블에서 80포트 관련 규칙 확인 (kube-proxy가 만든 규칙 확인용)</code></pre>
<hr />
<h1 id="7-label--selector--nodeselector">7. Label / Selector / nodeSelector</h1>
<pre><code class="language-bash">kubectl label pod redis-pod app=db                       # pod에 label 추가
kubectl label pod nginx-pod app=web-services type=frontend # 여러 label 동시 추가
kubectl label pod label-pod app=web-services --overwrite  # 기존 label 값 덮어쓰기
kubectl label pod testpod2 env=debug --overwrite

kubectl get pod --show-labels                 # label 정보 전체 확인
kubectl get pod -L app,type                    # 특정 label 값만 컬럼으로 확인
kubectl get po -l app=hpe                        # label이 app=hpe 인 pod만 조회
kubectl get po -l '!env'                           # env label이 &quot;없는&quot; pod만 조회 (! 연산자)
kubectl delete po -l app=hpe                         # label 조건에 맞는 pod 전체 삭제
kubectl delete pod -l release=canary                  # release=canary label을 가진 pod 삭제
kubectl delete po --all                                 # 현재 네임스페이스의 모든 pod 삭제

kubectl label nodes k8s-worker1 ssd=true        # 노드에 label 부여 (nodeSelector용)
kubectl label node k8s-worker1 disk=ssd          # 노드에 다른 키의 label 부여
kubectl label node k8s-worker1 ssd-                # 노드 label 삭제 (키 뒤에 '-')</code></pre>
<p>nodeSelector로 특정 조건의 노드에만 Pod를 배치:</p>
<pre><code class="language-yaml">spec:
  nodeSelector:
    ssd: &quot;true&quot;</code></pre>
<pre><code class="language-bash">kubectl create -f pod-nodeselector.yaml       # nodeSelector가 적용된 pod 생성
kubectl get pods nodeselector-pod -o wide      # 어느 노드에 배치됐는지 확인</code></pre>
<hr />
<h1 id="8-namespace">8. Namespace</h1>
<pre><code class="language-bash">kubectl get namespaces                             # namespace 목록 (= kubectl get ns)
kubectl create namespace prod                        # namespace 생성
kubectl create namespace custom-namespace
kubectl get pods -n green                              # green 네임스페이스의 pod 조회
kubectl get pod --namespace kube-system                 # --namespace 풀네임도 동일하게 동작
kubectl create -f testpod.yaml -n custom-namespace        # 특정 namespace에 리소스 생성
kubectl delete namespace custom                              # namespace 삭제
kubectl get namespace | wc -l                                  # namespace 개수 세기</code></pre>
<hr />
<h1 id="9-로그--exec--디버깅">9. 로그 / exec / 디버깅</h1>
<pre><code class="language-bash">kubectl describe pod nginx-pod               # 상세 스펙 + 이벤트 로그 (문제 원인 파악 1순위)
kubectl logs testpod                          # pod 로그 확인
kubectl logs testpod -c appjs-container        # 멀티 컨테이너 pod에서 특정 컨테이너 로그만
kubectl logs multipod -c nginx-container

kubectl exec -it centos-pod -- /bin/bash       # pod 컨테이너 안으로 접속
kubectl exec -it webserver -- /bin/bash
kubectl exec dynamic-web -c web-server -- ls /usr/share/nginx/html         # 접속 없이 명령 하나만 실행
kubectl exec dynamic-web -c web-server -- touch /usr/share/nginx/html/test.html
kubectl exec -n kube-system -it fluentd-elasticsearch-2dqfp -- bash         # 다른 namespace의 pod 접속

kubectl get pod testpod -o yaml                 # pod 상세정보를 yaml로
kubectl get pod testpod -o json                  # pod 상세정보를 json으로</code></pre>
<hr />
<h1 id="10-노드-관리-cordon--drain--uncordon">10. 노드 관리 (cordon / drain / uncordon)</h1>
<p>점검 전/후 흐름: <strong>cordon(신규 배치 차단) → drain(기존 Pod 이동) → (점검) → uncordon(복구)</strong>.</p>
<pre><code class="language-bash">kubectl cordon k8s-worker2                 # 해당 노드에 새 Pod가 더이상 스케줄링되지 않도록 막기
kubectl drain k8s-worker1                   # 노드의 Pod들을 다른 노드로 이동
kubectl drain k8s-worker1 --ignore-daemonsets   # DaemonSet이 만든 Pod는 제외하고 나머지만 이동
kubectl drain k8s-worker1 --dry-run=client       # 실제로 옮기지 않고 미리보기만
kubectl drain k8s-worker1 --force                 # 단독 Pod처럼 drain이 막히는 경우 강제로 삭제
kubectl uncordon k8s-worker2                        # 점검 끝난 뒤 다시 스케줄링 가능하게 복구
kubectl delete node k8s-worker2                       # 노드 자체를 클러스터에서 제거 (= delete nodes)</code></pre>
<hr />
<h1 id="11-kubeadm-토큰--초기화">11. kubeadm (토큰 / 초기화)</h1>
<pre><code class="language-bash">kubeadm token list          # 현재 유효한 join 토큰 목록 조회
kubeadm token create         # 새 join 토큰 발급 (기존 토큰 만료 시)
kubeadm reset                  # (Worker 노드에서) 클러스터 설정 초기화 — join 실패 반복 시 사용</code></pre>
<hr />
<h1 id="12-pv--pvc-스토리지">12. PV / PVC (스토리지)</h1>
<pre><code class="language-bash">kubectl create -f pv1.yaml -f pv2.yaml -f pv3.yaml -f pv4.yaml   # PersistentVolume 여러 개 한 번에 생성
kubectl get pv                                                     # PersistentVolume 목록 확인
kubectl create -f pvc.yaml                                          # PersistentVolumeClaim 생성
kubectl get pvc                                                       # PVC 목록 확인
kubectl get pv,pvc                                                      # 둘 다 한 번에 확인

kubectl create -f pod-mongodb.yaml           # PVC를 사용하는 Pod(MongoDB) 생성
kubectl exec -it mongodb-pod -- mongo         # mongo 셸로 접속
kubectl delete pod mongodb-pod                 # Pod 삭제
kubectl delete pvc mongo-pvc                     # PVC 삭제</code></pre>
<hr />
<h1 id="13-기타-port-forward--events--top--docker">13. 기타 (port-forward / events / top / docker)</h1>
<pre><code class="language-bash">kubectl port-forward testpod 8888:8080        # 로컬 8888 포트를 pod의 8080 포트로 연결
kubectl port-forward webserver 80:80
curl localhost:8888                             # 포트포워딩이 잘 됐는지 확인

kubectl get events                               # 클러스터 이벤트 로그

kubectl top nodes                                  # 노드별 CPU/메모리 사용량 (metrics-server 필요)
kubectl top pods                                     # pod별 CPU/메모리 사용량

docker version                                         # 로컬 docker 버전 확인
docker build -t powermvp/nginx .                         # 이미지 빌드 + 태그 지정
docker push powermvp/nginx                                 # 빌드한 이미지를 레지스트리(도커허브 등)로 push

systemctl start nginx &amp;&amp; systemctl enable nginx              # (VM에서) nginx 실행 + 부팅시 자동시작 등록
systemctl status nginx                                          # nginx 상태 확인
systemctl restart nginx                                           # nginx 재시작</code></pre>