<h2 id="1-오늘의-맥락--문법을-배우고-바로-작은-프로그램으로-확인해본-하루">1. 오늘의 맥락 — 문법을 배우고 바로 작은 프로그램으로 확인해본 하루</h2>
<p>어제는 class와 변수 개념 위주였다면, 오늘은 그 위에서 실제로 움직이는 코드를 많이 짰다. static 키워드, 반복문(for/while/do-while), 제어문(if/switch), 그리고 콘솔에서 숫자를 맞추는 <strong>업다운 게임(GuessGame)</strong>까지 만들어봤다.</p>
<blockquote>
<p>각 코드 아래 &quot;예상 실행 결과&quot;는 로직을 한 줄씩 직접 따라가며 계산한 결과다. (<code>Math.random()</code>이 들어간 부분과 <code>Scanner</code> 입력 부분은 예시 값을 하나 정해서 트레이싱했다.)</p>
</blockquote>
<hr />
<h2 id="2-파일-이름-규칙--app과-demo는-무슨-관계일까">2. 파일 이름 규칙 — <code>App</code>과 <code>Demo</code>는 무슨 관계일까</h2>
<p>오늘 만든 파일들을 보면 두 종류로 나뉜다.</p>
<table>
<thead>
<tr>
<th>이름 패턴</th>
<th>역할</th>
<th>오늘 예시</th>
</tr>
</thead>
<tbody><tr>
<td><code>xxxApp.java</code></td>
<td><strong>실행 진입점</strong>. <code>main()</code> 메서드가 있는 파일</td>
<td><code>OperatorApp</code>, <code>StaticApp</code>, <code>GuessGameApp</code></td>
</tr>
<tr>
<td><code>xxxDemo.java</code> (또는 그냥 기능 이름)</td>
<td><strong>실제 기능이 담긴 클래스</strong>. <code>main()</code>이 없고, <code>App</code>이 이 클래스를 <code>new</code>로 만들어서 메서드를 호출</td>
<td><code>OperatorDemo</code>, <code>StaticDemo</code></td>
</tr>
</tbody></table>
<p>즉 <code>App</code> = &quot;실행 버튼을 누르는 곳&quot;, <code>Demo</code>(또는 기능 클래스) = &quot;실제로 일이 일어나는 곳&quot;으로 역할이 나뉜다.</p>
<p>실제로 오늘 <code>OperatorDemoApp.java</code>를 처음 만들었다가, 이름을 <code>OperatorApp.java</code>로 바꿨다. <code>App</code> 이름은 바뀌었지만 안에서 <code>new OperatorDemo()</code>로 같은 <code>OperatorDemo</code> 클래스를 계속 썼다 — <strong>이름이 뭐든 &quot;App이 Demo를 호출한다&quot;는 관계 자체는 그대로 유지됐다.</strong> <code>GuessGame</code>처럼 이름에 <code>Demo</code>가 안 붙은 경우도, <code>GuessGameApp</code>이 <code>new GuessGame()</code>으로 호출하는 같은 구조다.</p>
<hr />
<h2 id="3-static-키워드--class-소유-vs-instance-소유">3. static 키워드 — class 소유 vs instance 소유</h2>
<p><strong>Java</strong> · <code>StaticDemo.java</code></p>
<pre><code class="language-java">public class StaticDemo {
    public String              message = &quot;인스턴스 소유의 변수&quot;;
    public static String       staticMessage = &quot;클래스 소유의 변수&quot;;
    public static final double PI = 3.14;

    public void instanceMethod() {
        System.out.println(message);
    }

    public static void classMethod() {
        // System.out.println(message); // ← 에러
        System.out.println(new StaticDemo().message); // 인스턴스를 직접 만들어야 접근 가능
        System.out.println(staticMessage);
    }
}</code></pre>
<p><strong>코드 리뷰</strong></p>
<ul>
<li><code>static</code>이 붙으면 인스턴스가 아니라 <strong>class 자체가</strong> 그 변수/메서드를 소유한다. 그래서 <code>new</code> 없이 <code>StaticDemo.staticMessage</code>처럼 바로 접근 가능.</li>
<li><code>static final</code>은 값을 못 바꾸는 <strong>상수</strong>.</li>
<li>⚠️ static 메서드(<code>classMethod</code>) 안에서는 인스턴스 소유 변수(<code>message</code>)에 바로 접근 불가. <code>new StaticDemo().message</code>처럼 인스턴스를 직접 만들어야 함 — &quot;어느 인스턴스 것인지&quot; 정할 방법이 없기 때문.</li>
</ul>
<p><strong>예상 실행 결과</strong></p>
<pre><code>인스턴스 소유의 변수
클래스 소유의 변수
3.14</code></pre><p>(<code>instanceMethod()</code>, <code>classMethod()</code> 둘 다 같은 결과 — 둘 다 결국 <code>message</code>, <code>staticMessage</code>, <code>PI</code> 값을 그대로 출력하기 때문)</p>
<hr />
<h2 id="4-연산자-실습--메서드와-dto">4. 연산자 실습 — 메서드와 DTO</h2>
<p><strong>Java</strong> · <code>OperatorDemo.java</code></p>
<pre><code class="language-java">public BlogResponseDTO register(String title, String content, String email) {
    if (email == &quot;shs10025@naver.com&quot;) {
        return new BlogResponseDTO(201, &quot;OK&quot;);
    } else {
        return new BlogResponseDTO(400, &quot;FAIL&quot;);
    }
}</code></pre>
<p><strong>코드 리뷰</strong></p>
<ul>
<li>반환타입·매개변수가 있는 메서드 연습. 매개변수를 받아 판단하고, 결과를 <code>BlogResponseDTO</code>(응답 객체)에 담아 반환.</li>
<li>매개변수 3개 대신 <code>BlogRequestDTO</code> 객체 하나로 받는 버전도 오버로딩으로 추가함.</li>
<li>⚠️ <code>BlogResposeDTO</code>(n 오타) → <code>BlogResponseDTO</code>로 고침. 자바는 <strong>클래스명 = 파일명</strong>이라, 오타 고칠 때 둘 다 맞춰야 함.</li>
<li>⚠️ <code>email == &quot;...&quot;</code> — 문자열은 <code>==</code> 대신 <code>.equals()</code>를 써야 함 (5번 섹션에서 이유 확인). 아직 안 고쳐진 상태.</li>
</ul>
<p><strong>예상 실행 결과</strong> (<code>register(&quot;오늘도 무사히&quot;, &quot;앗&quot;, &quot;shs10025@naver.com&quot;)</code> 호출 시)</p>
<pre><code>201
OK</code></pre><p>→ 흥미로운 점: 이메일을 문자열 리터럴로 직접 넘겼기 때문에, 자바의 문자열 리터럴 캐싱(String Pool) 덕분에 <code>==</code> 비교가 우연히 <code>true</code>로 맞아떨어진 것. <code>new String(...)</code>이나 Scanner 입력으로 받은 값이었다면 <code>==</code>는 <code>false</code>가 되어 무조건 &quot;FAIL&quot;이 나왔을 것 — &quot;지금은 우연히 동작하지만 위험하다&quot;는 게 바로 이 상황.</p>
<hr />
<h2 id="5-if-→-switch-→-화살표-switch">5. if → switch → 화살표 switch</h2>
<p><strong>Java</strong> · <code>OperatorDemo.java</code></p>
<pre><code class="language-java">public String ifWoodMain(int number) {
    String result = null;
    switch (number) {
        case 1  -&gt; result = &quot;거짓말하는구나&quot;;
        case 2  -&gt; result = &quot;또 거짓말하는구나&quot;;
        case 3  -&gt; result = &quot;정직하구나 너에게 모든 도끼를 주겠다&quot;;
        default -&gt; result = &quot;1 ~ 3 사이의 숫자를 입력해주세요.&quot;;
    }
    return result;
}</code></pre>
<p><strong>코드 리뷰</strong></p>
<ul>
<li>같은 문제를 세 가지 방식으로 다시 짜봄: 중첩 if → 전통 switch → 화살표 switch(<code>-&gt;</code>).</li>
<li>화살표 switch는 <code>:</code>와 <code>break</code>가 필요 없어서 더 간결함.</li>
<li>이전 시도는 지우지 않고 주석으로 남겨서, 왜 이 방식을 골랐는지 흐름이 남음.</li>
</ul>
<p><strong>예상 실행 결과</strong> (입력값별)</p>
<table>
<thead>
<tr>
<th>입력</th>
<th>출력</th>
</tr>
</thead>
<tbody><tr>
<td>1</td>
<td>거짓말하는구나</td>
</tr>
<tr>
<td>2</td>
<td>또 거짓말하는구나</td>
</tr>
<tr>
<td>3</td>
<td>정직하구나 너에게 모든 도끼를 주겠다</td>
</tr>
<tr>
<td>4</td>
<td>1 ~ 3 사이의 숫자를 입력해주세요.</td>
</tr>
</tbody></table>
<hr />
<h2 id="6-sumnumber--값이-거꾸로-들어와도-안전하게">6. <code>sumNumber</code> — 값이 거꾸로 들어와도 안전하게</h2>
<p><strong>Java</strong> · <code>OperatorDemo.java</code></p>
<pre><code class="language-java">public int sumNumber(int start, int end) {
    if (start &gt; end) {
        int temp = start;
        start = end;
        end = temp;
    }
    int result = 0;
    for (int data = start; data &lt;= end; data++) {
        result += data;
    }
    return result;
}</code></pre>
<p><strong>코드 리뷰</strong></p>
<ul>
<li><code>sumNumber(100, 1)</code>처럼 거꾸로 넣으면 <code>for</code> 조건이 처음부터 거짓이라 0이 나오는 문제 발견.</li>
<li><code>temp</code> 변수로 두 값을 맞바꾸는 코드를 추가해서 해결. 임시 변수 없이 바로 바꾸면 원래 값이 먼저 사라져서 안 됨.</li>
</ul>
<p><strong>예상 실행 결과</strong> (<code>sumNumber(100, 1)</code> 호출 시)</p>
<pre><code>5050</code></pre><p>(100↔1이 스왑되어 1부터 100까지의 합을 계산)</p>
<hr />
<h2 id="7-sumrandom--forwhiledo-while-세-가지-버전-비교">7. <code>sumRandom</code> — for/while/do-while 세 가지 버전 비교</h2>
<p>원본 코드에는 for/while 버전이 주석으로 남아있었다. 세 버전 모두 &quot;1부터 난수(<code>nan</code>)까지 더한다&quot;는 같은 로직이고, 반복문 형태만 다르다.</p>
<p><strong>Java</strong> · <code>OperatorDemo.java</code> — for 버전</p>
<pre><code class="language-java">public static int sumRandomFor() {
    int nan = (int)(Math.random() * 100) + 1;
    int result = 0;
    for (int data = 1; data &lt;= nan; data++) {
        result += data;
    }
    return result;
}</code></pre>
<p><strong>코드 리뷰</strong></p>
<ul>
<li>초기식(<code>data=1</code>)·조건식(<code>data&lt;=nan</code>)·증감식(<code>data++</code>)이 한 줄에 다 모여있어서, &quot;1부터 nan까지&quot;라는 범위가 눈에 바로 들어옴.</li>
</ul>
<p><strong>예상 실행 결과</strong> (<code>nan=7</code>인 경우)</p>
<pre><code>28</code></pre><p>(1+2+3+4+5+6+7 = 28)</p>
<hr />
<p><strong>Java</strong> · <code>OperatorDemo.java</code> — while 버전</p>
<pre><code class="language-java">public static int sumRandomWhile() {
    int nan = (int)(Math.random() * 100) + 1;
    int result = 0;
    int data = 1;
    while (data &lt;= nan) {
        result += data;
        data++;
    }
    return result;
}</code></pre>
<p><strong>코드 리뷰</strong></p>
<ul>
<li><code>data</code>를 미리 선언·초기화해두고, 조건식만 앞에 쓰는 구조. <code>data++</code>(증감)을 몸통 안에 직접 챙겨줘야 해서 for보다 깜빡하기 쉬움.</li>
</ul>
<p><strong>예상 실행 결과</strong> (<code>nan=7</code>인 경우)</p>
<pre><code>28</code></pre><p>(for 버전과 로직이 같아서 같은 <code>nan</code>이면 결과도 동일)</p>
<hr />
<p><strong>Java</strong> · <code>OperatorDemo.java</code> — do-while 버전 </p>
<pre><code class="language-java">public static int sumRandom() {
    int nan = (int)(Math.random() * 100) + 1;
    int result = 0, data = 1;
    do {
        result += data;
        data++;
    } while (data &lt;= nan);
    return result;
}</code></pre>
<p><strong>코드 리뷰</strong></p>
<ul>
<li><code>Math.random() * 100 + 1</code>로 1~100 난수 생성 (형변환은 어제 배운 캐스팅 그대로 적용).</li>
<li>for/while 버전도 짜봤지만, <strong>몸통이 최소 1번은 실행돼야 하는</strong> 상황이라 do-while을 최종 선택.</li>
<li>세 버전 다 <code>nan</code> 값이 같으면 결과도 똑같다 — <strong>반복문 종류가 달라져도 &quot;무엇을 계산하는가&quot;는 그대로</strong>라는 걸 다시 확인한 부분.</li>
</ul>
<p><strong>예상 실행 결과</strong> (매번 랜덤이라 실행할 때마다 달라짐 — 예시: <code>nan=7</code>이 나온 경우)</p>
<pre><code>debug &gt;&gt;&gt;&gt;&gt; generate nan=7
28</code></pre><p>(1+2+3+4+5+6+7 = 28)</p>
<hr />
<h2 id="8-구구단--라벨label-붙은-break">8. 구구단 — 라벨(label) 붙은 break</h2>
<p><strong>Java</strong> · <code>OperatorDemo.java</code></p>
<pre><code class="language-java">public void gugudan() {
    outer:
    for (int row = 2; row &lt;= 9; row++) {
        for (int col = 1; col &lt;= 9; col++) {
            if (row == 5) break outer;
            System.out.printf(&quot;%d * %d =%d\t&quot;, row, col, row*col);
        }
        System.out.println();
    }
}</code></pre>
<p><strong>코드 리뷰</strong></p>
<ul>
<li><code>break</code>는 안쪽 반복문만 빠져나가서, 5단에서 멈추려면 바깥 반복문에 <code>outer:</code> 라벨을 붙이고 <code>break outer;</code>로 지정해야 했음.</li>
</ul>
<p><strong>예상 실행 결과</strong></p>
<pre><code>2 * 1 =2    2 * 2 =4    2 * 3 =6    2 * 4 =8    2 * 5 =10    2 * 6 =12    2 * 7 =14    2 * 8 =16    2 * 9 =18    
3 * 1 =3    3 * 2 =6    3 * 3 =9    3 * 4 =12    3 * 5 =15    3 * 6 =18    3 * 7 =21    3 * 8 =24    3 * 9 =27    
4 * 1 =4    4 * 2 =8    4 * 3 =12    4 * 4 =16    4 * 5 =20    4 * 6 =24    4 * 7 =28    4 * 8 =32    4 * 9 =36    </code></pre><p>(5단은 <code>col=1</code>에서 바로 <code>break outer</code>가 실행돼서 한 줄도 안 찍히고 메서드가 끝남 — &quot;5단만 건너뛰기&quot;가 아니라 &quot;5단부터 전체 종료&quot;라는 점 주의)</p>
<hr />
<h2 id="9-scanner--guessgame">9. Scanner + GuessGame</h2>
<p><strong>Java</strong> · <code>GuessGame.java</code> — 최종 버전(do-while)</p>
<pre><code class="language-java">public String gameDoWhile() {
    Scanner scan = new Scanner(System.in);
    boolean isCorrect = false;
    int answer = (int)(Math.random() * 100) + 1;
    int cnt = 1;

    do {
        int guess = scan.nextInt();
        if (guess == answer) {
            isCorrect = true;
            break;
        } else if (guess &gt; answer) {
            System.out.println(&quot;Down&quot;);
        } else {
            System.out.println(&quot;Up&quot;);
        }
        cnt++;
    } while (!isCorrect &amp;&amp; cnt &lt;= 10);

    return isCorrect ? &quot;성공&quot; : &quot;실패&quot;;
}</code></pre>
<p><strong>코드 리뷰</strong></p>
<ul>
<li><code>Scanner</code>로 콘솔 입력을 받는 걸 처음 써봄. <code>scan.nextInt()</code>가 입력을 기다렸다가 정수로 돌려줌.</li>
<li>같은 게임을 <code>gameFor()</code> → <code>gameWhile()</code> → <code>gameDoWhile()</code> 순서로 세 번 다시 짬.</li>
<li>⚠️ <code>gameFor()</code>에서 처음엔 Down/Up 판정을 정답 판정보다 먼저 검사해서, 맞혀도 &quot;Up&quot;이 먼저 뜨는 문제가 있었음 → 정답 검사를 맨 위로 옮겨서 해결.</li>
<li>⚠️ <code>gameWhile()</code>에서 조건식을 <code>isCorrect == true || cnt == 10</code>으로 잘못 써서 로직이 반대로 동작 → <code>!isCorrect &amp;&amp; cnt &lt;= 10</code>(계속할 조건 기준)으로 수정.</li>
<li>두 실수를 겪은 뒤라 <code>gameDoWhile()</code>은 같은 실수 없이 한 번에 완성.</li>
</ul>
<p><strong>예상 실행 결과</strong> (Scanner로 사람이 입력하는 코드라 값이 매번 달라짐 — <code>answer=42</code>, 입력을 <code>50 → 20 → 42</code> 순으로 넣은 예시)</p>
<pre><code>Down
Up
성공</code></pre><p>(50은 42보다 커서 Down, 20은 작아서 Up, 42는 정답이라 <code>isCorrect=true</code>가 되고 반복문 종료 → <code>&quot;성공&quot;</code> 반환)</p>
<hr />
<h2 id="10-string-stringbuffer--와-equals">10. String, StringBuffer — <code>==</code>와 <code>.equals()</code></h2>
<p><strong>Java</strong> · <code>StringApp.java</code></p>
<pre><code class="language-java">String str01 = new String(&quot;:lgcns&quot;);
String str02 = new String(&quot;:lgcns&quot;);

str01 == str02;          // false — 서로 다른 객체(주소값 비교)
str01.equals(str02);     // true  — 내용이 같은지 비교

StringBuffer sb = new StringBuffer(str01);
sb.append(&quot;!!&quot;);</code></pre>
<p><strong>코드 리뷰</strong></p>
<ul>
<li><code>new</code>로 따로 만든 문자열 두 개는 내용이 같아도 <code>==</code>로 비교하면 <code>false</code> (주소 비교이기 때문).</li>
<li><code>.equals()</code>는 내용을 비교해서 <code>true</code>. → 4번 섹션의 <code>email == &quot;...&quot;</code> 문제와 바로 연결됨.</li>
<li><code>StringBuffer</code>는 내용을 직접 바꿀 수 있는(가변) 문자열 클래스. <code>.append()</code>로 이어붙임.</li>
</ul>
<p><strong>예상 실행 결과</strong></p>
<pre><code>str01 != str02
str01 == equals(str02)</code></pre><p>(<code>sb.toString()</code>의 최종 값은 <code>&quot;:lgcns!!&quot;</code>)</p>
<hr />
<h2 id="11-개념-정리--string-3형제-breakcontinue-printf">11. 개념 정리 — String 3형제, break/continue, printf</h2>
<p><strong>String vs StringBuffer vs StringBuilder</strong></p>
<table>
<thead>
<tr>
<th>구분</th>
<th>특징</th>
</tr>
</thead>
<tbody><tr>
<td><code>String</code></td>
<td>불변(내용 변경 불가), 값이 잘 안 바뀔 때</td>
</tr>
<tr>
<td><code>StringBuffer</code></td>
<td>가변 + 동기화 지원(멀티스레드 안전)</td>
</tr>
<tr>
<td><code>StringBuilder</code></td>
<td>가변, 동기화 없음(단일스레드에서 더 빠름)</td>
</tr>
</tbody></table>
<p><strong>break vs continue vs label</strong></p>
<ul>
<li><code>break</code>: 반복문 완전히 종료</li>
<li><code>continue</code>: 이번 회차만 건너뛰고 다음 회차로</li>
<li><code>outer:</code> 같은 라벨을 붙이면, 중첩 반복문에서 어느 반복문을 멈출지 지정 가능 (라벨 없으면 항상 가장 안쪽에만 적용)</li>
</ul>
<p><strong>printf 포맷팅</strong>
<code>%d</code>(정수), <code>%s</code>(문자열), <code>%f</code>(실수) 자리표시자에 값을 순서대로 채워 넣는 방식. <code>+</code>로 문자열 이어붙이는 것보다 구구단 같은 반복 출력에 편함.</p>
<hr />
<h2 id="12-헷갈렸던-점-다음-복습-포인트-🤔">12. 헷갈렸던 점 (다음 복습 포인트) 🤔</h2>
<ul>
<li>String, StringBuffer, StringBuilder — 개념 표는 정리했지만 실전에서 바로 판단하는 감각은 아직 부족</li>
<li>static 메서드에서 인스턴스 변수 접근 안 되는 이유 — class/instance 소유 관계를 다시 떠올려야 이해됨</li>
<li>반복문 조건은 &quot;언제 멈출지&quot;가 아니라 &quot;언제 계속할지&quot; 기준으로 써야 한다는 것 (gameWhile 실수에서 배움)</li>
</ul>
<hr />
<h2 id="오늘의-한-줄-요약">오늘의 한 줄 요약</h2>
<p>static으로 class/instance 소유를 구분하고, 같은 문제(도끼 이야기, 업다운 게임)를 if/switch, for/while/do-while로 반복해서 짜보며 반복문 사이의 차이를 직접 겪은 하루였다.</p>
<hr />
<h2 id="마무리-회고">마무리 회고</h2>
<p>오늘은 같은 문제를 여러 문법으로 반복해서 풀어본 하루였다. 두 번째, 세 번째로 갈수록 앞서 했던 실수(판정 순서, 조건식 방향)를 자연스럽게 피하게 되는 걸 보면서, 반복해서 짜보는 게 문법을 몸에 익히는 데 확실히 도움이 된다는 걸 느꼈다. 내일은 오늘 남겨둔 <code>.equals()</code> 수정부터 정리해야겠다.</p>