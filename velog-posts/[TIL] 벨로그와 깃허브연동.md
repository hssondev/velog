<h2 id="개발자를-향한-첫-걸음">개발자를 향한 첫 걸음</h2>
<ul>
<li>velog 회원가입과 github 연동<ul>
<li>필요한 이유 : 오늘 무엇을 배웠는지? 앞으로 무엇을 배울지? 기록을 위해</li>
</ul>
</li>
</ul>
<h3 id="1-깃허브에-레포지토리-설정">1. 깃허브에 레포지토리 설정</h3>
<p>   <img alt="" src="https://velog.velcdn.com/images/hssondev/post/7db95ee7-2dff-4a16-88bd-f8d2ef577f79/image.png" /></p>
<h3 id="2-update_blogyml-파일-생성">2. update_blog.yml 파일 생성</h3>
<ul>
<li>파일경로 : .github/workflows/update_blog.yml</li>
<li>update_blog.yml 파일 작성</li>
</ul>
<pre><code class="language-yml">name: Update Blog Posts


on:
  push:
      branches:
        - [branch] # 또는 워크플로우를 트리거하고 싶은 브랜치 이름
  workflow_dispatch:    # 수동 실행 트리거 추가
  schedule:
    - cron: '0 0 * * *' # 매일 자정에 실행

jobs:
  update_blog:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout
      uses: actions/checkout@v2

    - name: Set time zone
      run: |
        sudo ln -sf /usr/share/zoneinfo/Asia/Seoul /etc/localtime
        echo &quot;Asia/Seoul&quot; | sudo tee /etc/timezone
        date

    - name: Push changes
      run: |
        git config --global user.name 'github-actions[bot]'
        git config --global user.email 'github-actions[bot]@users.noreply.github.com'
        git push https://${{ secrets.GH_PAT }}@github.com/[깃허브아이디]/velog.git

    - name: Set up Python
      uses: actions/setup-python@v2
      with:
        python-version: '3.x'

    - name: Install dependencies
      run: |
        pip install feedparser gitpython

    - name: Run script
      run: python scripts/update_blog.py
</code></pre>
<h3 id="3-update_blogpy-파일-생성">3. update_blog.py 파일 생성</h3>
<p>1) velog 레포지토리 -&gt; Add file -&gt; Create new file</p>
<ul>
<li>파일 경로 : <code>scripts/update_blog.py</code></li>
<li><code>update_blog.py</code> 파일 작성</li>
</ul>
<pre><code class="language-py">import feedparser
import git
import os

# 벨로그 RSS 피드 URL
# example : rss_url = 'https://api.velog.io/rss/@rimgosu'
rss_url = 'https://api.velog.io/rss/@[velog 아이디]'

# 깃허브 레포지토리 경로
repo_path = '.'

# 'velog-posts' 폴더 경로
posts_dir = os.path.join(repo_path, 'velog-posts')

# 'velog-posts' 폴더가 없다면 생성
if not os.path.exists(posts_dir):
    os.makedirs(posts_dir)

# 레포지토리 로드
repo = git.Repo(repo_path)

# RSS 피드 파싱
feed = feedparser.parse(rss_url)

# 각 글을 파일로 저장하고 커밋
for entry in feed.entries:
    # 파일 이름에서 유효하지 않은 문자 제거 또는 대체
    file_name = entry.title
    file_name = file_name.replace('/', '-')  # 슬래시를 대시로 대체
    file_name = file_name.replace('\\', '-')  # 백슬래시를 대시로 대체
    # 필요에 따라 추가 문자 대체
    file_name += '.md'
    file_path = os.path.join(posts_dir, file_name)

    # 파일이 이미 존재하지 않으면 생성
    if not os.path.exists(file_path):
        with open(file_path, 'w', encoding='utf-8') as file:
            file.write(entry.description)  # 글 내용을 파일에 작성

        # 깃허브 커밋
        repo.git.add(file_path)
        repo.git.commit('-m', f'Add post: {entry.title}')

# 변경 사항을 깃허브에 푸시
repo.git.push()</code></pre>
<h3 id="4-pat-권한-받기">4. PAT 권한 받기</h3>
<ol>
<li><p>github 계정 - Settings - Developer Settings - Personal access tokens (classic) - Generate New Token - 이름 쓰고 repo, workflow 클릭 - Generate new token</p>
</li>
<li><p>받은 토큰 복사 (<strong>이 창을 벗어난 이후에는 확인이 불가하니 꼭 복사하기</strong>!)</p>
</li>
<li><p>위에서 생성한 레포지토리 - Settings - Actions secrets and variables - Actions - New Repository Secret</p>
</li>
<li><p>Name : GH_PAT
Secret : [2번에서 발급받은 토큰]
<img alt="" src="https://velog.velcdn.com/images/hssondev/post/399755d1-ff96-4fea-9ccc-e589d1a2c92e/image.png" /></p>
</li>
</ol>
<h3 id="5-레포지토리-외부-권한-부여">5. 레포지토리 외부 권한 부여</h3>
<pre><code>1) velog 레포지토리 -&gt; Setting 클릭
2) Actions -&gt; General 클릭
3) 4개 체크 및 save(각 항목 변경시 save 하기)</code></pre><p><img alt="" src="https://velog.velcdn.com/images/hssondev/post/3a4310f2-27fd-4609-804d-18a93abd88f3/image.png" /></p>
<h3 id="6-연동-확인">6. 연동 확인</h3>
<p><img alt="" src="https://velog.velcdn.com/images/hssondev/post/66366ec6-c0de-4a25-8676-95cff7162186/image.png" /></p>
<pre><code>참고 블로그
- https://velog.io/@dev_ssj/velog%EC%99%80-gitHub-%EC%97%B0%EB%8F%99%ED%95%98%EA%B8%B0</code></pre>