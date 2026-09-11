<blockquote>
<p><strong>한 줄 요약</strong></p>
<p>Git으로 버전 관리하고, Docker로 환경을 이식 가능하게 포장하고, AWS EC2 위에 GitHub Actions/Jenkins로 빌드-테스트-배포를 자동으로 이어붙이는 CI/CD 파이프라인의 전체 그림을 실습으로 익혔다.</p>
</blockquote>
<hr />
<h1 id="📚-오늘-학습한-커리큘럼">📚 오늘 학습한 커리큘럼</h1>
<pre><code>DevOps 개념 (CI/CD란 무엇인가)
        ↓
Git 기본 명령어 (버전 관리)
        ↓
Docker 기본 명령어 (이미지 → 컨테이너)
        ↓
AWS 설정 (IAM 권한 → EC2 서버 생성 → 보안 그룹)
        ↓
GitHub Actions (개인 프로젝트용 → 설정 파일 분리 → 일반 프로젝트용 → Docker 기반)
        ↓
Jenkins 설치 및 Pipeline Job 구성</code></pre><h1 id="📌-오늘의-학습-키워드">📌 오늘의 학습 키워드</h1>
<ul>
<li>CI/CD (지속적 통합/배포)</li>
<li>Git (init / add / commit / push / pull)</li>
<li>Docker (이미지, 컨테이너, Dockerfile)</li>
<li>AWS (IAM, EC2, 보안 그룹)</li>
<li>GitHub Actions (Workflow, Job, Step, Secrets)</li>
<li>Jenkins (Freestyle, Pipeline, Webhook)</li>
</ul>
<hr />
<h1 id="0-개념-정리-키워드-사전">0. 개념 정리 (키워드 사전)</h1>
<blockquote>
<p><strong>한 줄 요약</strong></p>
<p>DevOps는 &quot;사람이 손으로 하던 빌드-테스트-배포를 도구가 자동으로 이어받게 만드는 문화&quot;이고, 오늘 배운 도구들은 각각 그 릴레이의 한 구간(코드 관리 → 포장 → 서버 → 자동화)을 맡는다.</p>
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
<td><strong>CI (Continuous Integration)</strong></td>
<td>여러 개발자의 코드를 자동으로 합치고 빌드·테스트하는 것</td>
<td>push하면 자동으로 빌드+테스트 실행</td>
</tr>
<tr>
<td><strong>CD (Continuous Delivery/Deployment)</strong></td>
<td>CI를 통과한 코드를 자동으로 서버에 배포하는 것</td>
<td>테스트 통과 → 자동 배포</td>
</tr>
<tr>
<td><strong>Docker 이미지</strong></td>
<td>컨테이너를 만들기 위한 읽기 전용 설계도</td>
<td>클래스(이미지) → 객체(컨테이너)</td>
</tr>
<tr>
<td><strong>컨테이너</strong></td>
<td>호스트 안에서 독립적으로 격리된 실행 환경</td>
<td>서로 충돌 없이 여러 개 동시 실행 가능</td>
</tr>
<tr>
<td><strong>IAM</strong></td>
<td>AWS 계정 안에서 &quot;누가 무엇을 할 수 있는지&quot; 정의하는 권한 관리 시스템</td>
<td>개발자A는 EC2만, 개발자B는 S3만</td>
</tr>
<tr>
<td><strong>EC2</strong></td>
<td>인터넷으로 빌려 쓰는 가상 서버</td>
<td>켜놓은 시간만큼만 과금(종량제)</td>
</tr>
<tr>
<td><strong>보안 그룹</strong></td>
<td>EC2 앞을 지키는 방화벽 규칙(인바운드/아웃바운드)</td>
<td>22(SSH), 80(HTTP), 8080/8088(앱) 포트 개방</td>
</tr>
<tr>
<td><strong>GitHub Actions Workflow</strong></td>
<td>push 등 이벤트가 발생하면 실행되는 자동화 스크립트(.yml)</td>
<td>Job → Step 순서로 구성</td>
</tr>
<tr>
<td><strong>Jenkins</strong></td>
<td>CI/CD를 실현해주는 오픈소스 자동화 서버</td>
<td>직접 서버에 설치해서 운영</td>
</tr>
</tbody></table>
<h3 id="코드로-다시-보기">코드로 다시 보기</h3>
<pre><code class="language-bash"># 오늘 배운 흐름을 한 번에 압축하면:
git add . &amp;&amp; git commit -m &quot;feat: 기능 추가&quot; &amp;&amp; git push   # 1. 코드 push
# → GitHub Actions/Jenkins가 감지 → 빌드 &amp; 테스트
docker build -t my-app:v1 .                                 # 2. 이미지로 포장
docker run -d -p 8080:80 my-app:v1                          # 3. 컨테이너로 실행
# → EC2(또는 ECR 경유)로 배포되어 서비스 재기동</code></pre>
<hr />
<h1 id="1-devops와-cicd-개념">1. DevOps와 CI/CD 개념</h1>
<p>기존 방식(고객 요구사항 → PM → 개발자 → 테스터 → 배포 담당자 → PM 검토)은 단계마다 담당자가 나뉘어 있어서, 문제가 생겼을 때 누구 책임인지 불분명하고 그룹마다 업무 관점이 달라 실질적인 협업이 잘 안 된다는 문제가 있었다. DevOps는 이 문제를 해결하기 위한 문화로, 단순히 개발(Dev)과 운영(Ops)을 합친 게 아니라 개발자가 운영까지 고려하고 운영자가 개발 흐름을 이해하며, 그 사이를 도구로 자동화하는 것이 핵심이다.</p>
<p>그 자동화의 핵심이 CI/CD다. CI(지속적 통합)는 개발자들이 작성한 코드를 자동으로 합치고 검증하는 것이고, CD(지속적 배포)는 CI를 통과한 코드를 자동으로 서버에 배포하는 것이다. 흐름으로 보면 &quot;commit &amp; push → 변경 감지 → 빌드 → 테스트(있다면) → 배포&quot;로 이어지며, 이 과정이 계획부터 운영·모니터링까지 끊기지 않고 유기적으로 연결되어 있는 것이 DevOps가 지향하는 형태다.</p>
<hr />
<h1 id="2-git-기본-명령어">2. Git 기본 명령어</h1>
<p>Git은 분산 버전 관리 시스템으로, 오늘 실습에서는 로컬 저장소 생성부터 원격 저장소 동기화까지의 기본 흐름을 다뤘다.</p>
<pre><code class="language-bash"># 초기화 및 스테이징
mkdir my-devops-project &amp;&amp; cd my-devops-project
git init
git add .

# 커밋 (상태를 사진 찍듯 스냅샷으로 기록)
git commit -m &quot;Initial commit: 프로젝트 초기 구조 생성&quot;

# 원격 저장소 연결 및 업로드
git remote add origin https://github.com/username/repo-name.git
git push -u origin main

# 동기화
git fetch origin   # 가져오기만 함 (병합 X)
git pull origin main   # fetch + merge를 한 번에</code></pre>
<p><code>git add</code>는 변경 사항을 &quot;커밋할 후보&quot;로 스테이징 영역에 올리는 단계이고, <code>git commit</code>은 그 스테이징 영역의 상태를 로컬 저장소에 영구 기록하는 단계다. <code>git fetch</code>와 <code>git pull</code>의 차이가 헷갈렸는데, fetch는 원격의 변경 내용을 가져오기만 하고 내 브랜치에 반영하지 않는 반면, pull은 fetch + merge를 함께 수행해서 바로 내 작업 브랜치에 병합까지 해준다는 점이 다르다. 협업 시 충돌을 줄이려면 작업 전에 pull을 습관화하는 게 좋다고 한다.</p>
<p>보안이 중요한 파일(<code>.env</code>, <code>application.yaml</code>, API Key 등)이나 빌드 산출물은 <code>.gitignore</code>에 등록해서 애초에 버전 관리 대상에서 제외해야 한다는 점도 배웠다.</p>
<p>커밋 메시지는 &quot;제목 → 공란 → 본문(무엇을, 왜) → 꼬리말(이슈 번호)&quot; 구조로 작성하며, 나중에 주간 보고에 그대로 옮겨 써도 될 정도로 구체적으로 쓰는 게 실무 팁이라고 한다.</p>
<hr />
<h1 id="3-docker-기본-명령어">3. Docker 기본 명령어</h1>
<p>Docker를 쓰는 가장 큰 이유는 이식성이다. 특정 프로그램을 다른 환경으로 쉽게 옮겨서 설치·실행할 수 있고, 설치 과정을 반복하지 않아도 되며, 버전이나 환경 설정을 항상 일관되게 유지할 수 있고, 각 프로그램이 독립된 환경에서 실행되어 서로 충돌하지 않는다.</p>
<p>핵심 개념은 이미지와 컨테이너의 관계다. 이미지는 컨테이너를 만들기 위한 읽기 전용 설계도(템플릿)이고, 컨테이너는 그 이미지를 바탕으로 실제 실행되는 격리된 미니 컴퓨터다. &quot;클래스 → 객체&quot; 비유처럼, 하나의 이미지로 동일한 컨테이너를 여러 개 찍어낼 수 있다.</p>
<pre><code class="language-bash"># 이미지 관리
docker pull nginx              # 이미지 다운로드
docker images                  # 로컬 이미지 목록
docker rmi [이미지ID/이미지명]   # 이미지 삭제

# 컨테이너 관리
docker run -d -p 8080:80 --name my-web-server nginx   # 실행 (create+start)
docker ps                      # 실행 중인 컨테이너 확인
docker ps -a                   # 중지된 것 포함 전체 확인
docker stop my-web-server      # 정상 종료
docker start my-web-server     # 재시작
docker rm 컨테이너명            # 삭제 (실행 중이면 -f 필요)</code></pre>
<p>이미지를 직접 만들 때는 <code>Dockerfile</code>을 작성한다.</p>
<pre><code class="language-docker">FROM node:14
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
CMD [&quot;node&quot;, &quot;app.js&quot;]</code></pre>
<p>여기서 헷갈렸던 부분이 <code>RUN</code>과 <code>CMD</code>의 역할 차이였는데, <code>RUN</code>은 이미지를 빌드하는 과정(빌드 타임)에 실행되는 명령이고, <code>CMD</code>(또는 <code>ENTRYPOINT</code>)는 그 이미지로 컨테이너를 시작할 때(런타임) 실행되는 명령이라는 점이 구분 포인트였다. 그리고 <code>COPY package*.json ./</code>을 먼저 하고 <code>RUN npm install</code>을 실행한 다음에 나머지 소스를 <code>COPY</code>하는 순서도, package.json이 바뀌지 않았다면 npm install 레이어를 캐시로 재사용해서 빌드 속도를 높이기 위한 의도적인 배치였다.</p>
<pre><code class="language-bash">docker build -t my-app:v1 .</code></pre>
<p>태그를 생략하면 자동으로 <code>latest</code>가 붙는다는 점도 알게 됐다.</p>
<hr />
<h1 id="4-aws-설정-iam-및-ec2">4. AWS 설정 (IAM 및 EC2)</h1>
<p>AWS는 아마존 데이터센터의 컴퓨터를 빌려서, 서버·저장소·네트워크를 인터넷으로 필요한 만큼만 쓰는 클라우드 서비스다. 예전 온프레미스 방식은 서버를 직접 사서 운영해야 해서 초기 비용이 크고 확장이 어려웠는데, AWS는 필요할 때 켜고 필요 없을 때 끄면 되니 초기 비용 없이 즉시 시작할 수 있다는 차이가 있다.</p>
<p>IAM은 &quot;AWS 계정 안에서 누가 무엇을 할 수 있는지&quot;를 정의하는 권한 관리 시스템이고, EC2는 그 위에서 실제로 빌려 쓰는 가상 서버다. EC2 인스턴스를 생성할 때 거친 주요 단계는 다음과 같다.</p>
<ul>
<li><strong>AMI 선택</strong>: OS는 Ubuntu Server LTS를 선택. Windows/macOS는 편의 기능이 많은 대신 용량과 성능을 많이 잡아먹어 서버 성능에 영향을 주기 때문에 실습에서는 제외</li>
<li><strong>인스턴스 유형</strong>: 실습용으로 <code>t3.medium</code>(최소 <code>t3a.small</code>) 권장 — 인스턴스는 &quot;빌린 컴퓨터 1대&quot;, 인스턴스 유형은 &quot;그 컴퓨터의 사양&quot;을 의미</li>
<li><strong>키 페어</strong>: EC2 접속용 비밀번호 역할. <code>.pem</code> 파일로 다운로드하며 유출되면 위험하므로 주의</li>
</ul>
<p>보안 그룹은 EC2 인스턴스를 감싸는 방화벽 개념으로, &quot;집(EC2)&quot;에 접근하기 전에 &quot;울타리의 대문(보안 그룹)&quot;에서 요청을 검사한다고 비유할 수 있다. 인바운드는 외부 → EC2로 들어오는 트래픽, 아웃바운드는 EC2 → 외부로 나가는 트래픽을 의미한다. 오늘 실습에서 연 포트는 다음과 같다.</p>
<table>
<thead>
<tr>
<th>유형</th>
<th>포트</th>
<th>소스</th>
<th>용도</th>
</tr>
</thead>
<tbody><tr>
<td>SSH</td>
<td>22</td>
<td>Anywhere</td>
<td>원격 접속용</td>
</tr>
<tr>
<td>HTTP</td>
<td>80</td>
<td>Anywhere</td>
<td>웹 서비스용</td>
</tr>
<tr>
<td>사용자 지정 TCP</td>
<td>8080</td>
<td>Anywhere</td>
<td>backend 접속용</td>
</tr>
<tr>
<td>사용자 지정 TCP</td>
<td>8088</td>
<td>Anywhere</td>
<td>backend 접속용</td>
</tr>
</tbody></table>
<p>EC2 접속은 macOS 기준 <code>chmod 400 &quot;키파일.pem&quot;</code> 후 <code>ssh -i &quot;키파일.pem&quot; ubuntu@[퍼블릭IP]</code>로 진행했고, Windows는 git bash 또는 WSL2(Ubuntu)에서 동일한 방식으로 접속했다.</p>
<hr />
<h1 id="5-github-actions로-cicd-구축">5. GitHub Actions로 CI/CD 구축</h1>
<p>GitHub Actions는 CI/CD 로직(빌드, 테스트, 배포)을 실행시켜주는, GitHub에 내장된 자동화 서버다. 별도 서버 구축 비용이 없고 시간도 아낄 수 있다는 게 장점이며, Jenkins는 GitHub 외 다른 VCS를 쓰거나 폐쇄망 환경, 복잡한 파이프라인, 대규모 빌드가 많은 경우에 비용 구조상 더 유리할 수 있다고 한다.</p>
<p>기본 구조는 <code>.github/workflows/</code> 아래에 <code>.yml</code> 파일을 두는 것이며, 하나의 yaml 파일이 하나의 Workflow, Workflow는 1개 이상의 Job(기본적으로 병렬 실행)으로, Job은 순차 실행되는 여러 Step으로 구성된다.</p>
<pre><code class="language-yaml">name: Github Actions 기본 사용법 실습

on:
  push:
    branches:
      - main

jobs:
  My-Deploy-Job:
    runs-on: ubuntu-latest
    steps:
      - name: Hello World 출력
        run: echo &quot;Hello World&quot;</code></pre>
<p>여기서 노출되면 안 되는 값(EC2 접속키, 계정 정보 등)은 GitHub Repository의 Settings → Secrets and variables → Actions에 등록해서 <code>${{ secrets.이름 }}</code>으로 참조한다.</p>
<p>배포 방식은 실습하면서 세 단계로 발전시켰다.</p>
<p><strong>① 개인 프로젝트용(가장 단순한 방식)</strong>: GitHub Actions가 EC2에 SSH로 직접 접속해서, EC2 안에서 <code>git pull</code> → 빌드 → 재실행까지 전부 수행한다.</p>
<pre><code class="language-yaml">jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: SSH(원격 접속)로 EC2에 접속하기
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USERNAME }}
          key: ${{ secrets.EC2_PRIVATE_KEY }}
          script_stop: true
          script: |
            cd /home/ubuntu/devops-service
            git pull origin main
            ./gradlew clean build
            sudo fuser -k -n tcp 8080 || true
            nohup java -jar build/libs/*SNAPSHOT.jar &gt; ./output.log 2&gt;&amp;1 &amp;</code></pre>
<p>처음엔 이 방식으로 실습했는데, 이 구조는 빌드 작업을 EC2 안에서 직접 돌리기 때문에 운영 중인 서버 성능에 영향을 주고, GitHub 계정 정보(또는 private repo면 credential)가 EC2에 저장되어야 한다는 단점이 있어서 개인/토이 프로젝트에만 적합하다는 걸 알게 됐다. <code>application.yaml</code>처럼 <code>.gitignore</code>로 뺀 설정 파일은 <code>secrets.APPLICATION_PROPERTIES</code>로 등록해두고 스크립트 안에서 <code>echo</code>로 파일을 다시 만들어주는 방식으로 자동화했다.</p>
<p><strong>② 일반 프로젝트용</strong>: 빌드·테스트를 EC2가 아니라 GitHub Actions 컴퓨터 안에서 전부 수행하고, 완성된 산출물(jar 파일)만 SCP로 EC2에 전달하는 방식으로 바꿨다. 이렇게 하면 EC2에 Github 계정 정보를 남길 필요가 없고 운영 서버 자원도 아낄 수 있다.</p>
<pre><code class="language-yaml">jobs:
  Deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: 21
      - run: ./gradlew clean build
      - run: mv ./build/libs/*SNAPSHOT.jar ./project.jar
      - name: SCP로 EC2에 빌드된 파일 전송하기
        uses: appleboy/scp-action@v0.1.7
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USERNAME }}
          key: ${{ secrets.EC2_PRIVATE_KEY }}
          source: project.jar
          target: /home/ubuntu/devops-server/output
      - name: SSH로 재실행만 담당
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USERNAME }}
          key: ${{ secrets.EC2_PRIVATE_KEY }}
          script: |
            mv /home/ubuntu/devops-server/output/project.jar /home/ubuntu/devops-server/current/project.jar
            cd /home/ubuntu/devops-server/current
            sudo fuser -k -n tcp 8080 || true
            nohup java -jar project.jar &gt; ./output.log 2&gt;&amp;1 &amp;</code></pre>
<p><strong>③ Docker 기반</strong>: 여기서 한 단계 더 나아가, 빌드된 산출물을 jar 파일 그대로 옮기는 대신 Docker 이미지로 포장해서 AWS ECR(이미지 저장소)에 push하고, EC2는 ECR에서 이미지를 pull 받아 컨테이너로 실행하는 구조로 바꿨다.</p>
<pre><code class="language-yaml">- name: AWS credentials 설정
  uses: aws-actions/configure-aws-credentials@v4
  with:
    aws-region: ap-northeast-2
    aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY }}
    aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
- name: ECR에 로그인하기
  id: login-ecr
  uses: aws-actions/amazon-ecr-login@v2
- run: docker build -t devops-server .
- run: docker tag devops-server ${{ steps.login-ecr.outputs.registry }}/devops-server:latest
- run: docker push ${{ steps.login-ecr.outputs.registry }}/devops-server:latest
- name: SSH로 EC2에서 pull &amp; 재실행
  uses: appleboy/ssh-action@v1.0.3
  with:
    host: ${{ secrets.EC2_HOST }}
    username: ${{ secrets.EC2_USERNAME }}
    key: ${{ secrets.EC2_PRIVATE_KEY }}
    script: |
      docker stop devops-server || true
      docker rm devops-server || true
      docker pull ${{ steps.login-ecr.outputs.registry }}/devops-server:latest
      docker run -d --name devops-server -p 8080:8080 ${{ steps.login-ecr.outputs.registry }}/devops-server:latest</code></pre>
<p>이 방식이 되려면 GitHub Actions의 IAM 사용자에 <code>AmazonEC2ContainerRegistryFullAccess</code> 권한을 추가해서 ECR에 이미지를 push할 수 있어야 하고, 반대로 EC2가 private ECR에서 이미지를 pull 받으려면 EC2에 연결된 IAM Role에도 같은 권한을 추가하고, EC2 안에 Amazon ECR Credential Helper를 설치해 <code>~/.docker/config.json</code>에 <code>&quot;credsStore&quot;: &quot;ecr-login&quot;</code>을 설정해줘야 한다는 점이 처음엔 헷갈렸다. GitHub Actions 쪽 권한(이미지를 올리는 쪽)과 EC2 쪽 권한(이미지를 받는 쪽)이 별개로 필요하다는 걸 실습하면서 이해했다.</p>
<hr />
<h1 id="✨-새롭게-알게-된-점">✨ 새롭게 알게 된 점</h1>
<ul>
<li>CI/CD는 도구 하나가 아니라 &quot;commit → 감지 → 빌드 → 테스트 → 배포&quot;라는 하나의 릴레이 흐름이며, GitHub Actions와 Jenkins는 이 릴레이를 대신 뛰어주는 러너라는 관점으로 이해하니 각 실습 단계가 왜 필요한지 납득이 됐다.</li>
<li>같은 배포 목표라도 &quot;EC2에서 직접 빌드(개인용) → 빌드는 Actions에서, 산출물만 전달(일반용) → 이미지로 포장해서 ECR 경유(Docker용)&quot;처럼 구조가 단계적으로 발전할 수 있고, 뒤로 갈수록 운영 서버 부담과 credential 노출 위험이 줄어든다는 트레이드오프를 실습으로 체감했다.</li>
<li>GitHub Actions 권한(이미지를 올리는 쪽)과 EC2/IAM Role 권한(이미지를 받는 쪽)이 서로 다른 곳에 별도로 설정되어야 한다는 점.</li>
<li>Jenkins를 설치할 때 <code>usermod -aG docker jenkins</code>가 되려면 Docker가 먼저 설치되어 docker 그룹이 존재해야 한다는, 설치 순서 자체가 갖는 의존관계.</li>
</ul>
<h1 id="🤔-헷갈렸던-점">🤔 헷갈렸던 점</h1>
<ul>
<li><code>git fetch</code>와 <code>git pull</code>의 차이 — fetch는 가져오기만 하고 내 브랜치엔 반영되지 않는다는 점.</li>
<li>Dockerfile의 <code>RUN</code>(빌드 타임 실행)과 <code>CMD</code>(런타임 실행)의 역할 구분.</li>
<li>GitHub Actions ECR 연동 시 필요한 권한이 &quot;GitHub Actions IAM 사용자용&quot;과 &quot;EC2 IAM Role용&quot; 두 곳에 나뉘어 있다는 점.</li>
</ul>
<h1 id="다음에-할-일">다음에 할 일</h1>
<ul>
<li><input disabled="" type="checkbox" /> devops-backend 프로젝트로 Jenkinsfile 기반 CI/CD 파이프라인 직접 구축해보기 (실습 과제)</li>
<li><input disabled="" type="checkbox" /> Github Webhook + Jenkins 자동 트리거 연동까지 끝까지 테스트해보기</li>
<li><input disabled="" type="checkbox" /> Docker 기반 배포(ECR 경유) 구조를 개인 프로젝트에도 적용해보기</li>
</ul>
<hr />
<h1 id="💡-오늘의-회고">💡 오늘의 회고</h1>
<p>오늘은 Git, Docker, AWS, GitHub Actions, Jenkins를 하루에 훑는 특강이라 처음엔 각 도구가 따로 노는 느낌이었는데, 실습을 세 단계(EC2 직접 빌드 → 산출물만 전달 → Docker 이미지로 포장)로 밟아가면서 결국 모든 도구가 &quot;코드를 안전하고 반복 가능하게 서버까지 옮기는&quot; 하나의 목표를 위해 각자 역할을 나눠 맡고 있다는 걸 체감할 수 있었다. 특히 왜 굳이 EC2에서 직접 빌드하지 않고 GitHub Actions에서 빌드해서 산출물만 넘기는지, 왜 굳이 jar 파일 대신 Docker 이미지로 포장해서 ECR을 거치는지 각 방식의 단점을 먼저 겪어보고 다음 단계로 넘어간 게 이해에 큰 도움이 됐다. 다음엔 이 흐름을 내 개인 프로젝트에도 직접 적용해보면서 완전히 내 것으로 만들어야겠다.</p>