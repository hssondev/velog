<blockquote>
<p><strong>한 줄 요약</strong></p>
<p>캡슐화는 &quot;숨기기&quot;가 아니라 &quot;검증을 강제하는 장치&quot;이고, 상속은 &quot;코드 재사용&quot;을 넘어 &quot;한 곳만 고쳐도 자식들이 알아서 따라오게 만드는 장치&quot;라는 걸 반대 상황(검증 없는 public 필드, 상속 없이 처음부터 다시 만들기)과 직접 비교하며 체감했다.</p>
</blockquote>
<hr />
<h1 id="📚-오늘-학습한-커리큘럼">📚 오늘 학습한 커리큘럼</h1>
<pre><code class="language-null">캡슐화(은닉화) 심화
        ↓
검증 없는 setter의 문제 확인 — setAge()에 -999를 넣어보기
        ↓
setter에 검증 로직 추가 — 조기 반환(early return) 패턴
        ↓
public 필드 vs private+setter 비교 — UnsafeStudent

상속 심화
        ↓
상속 없이 만들면? — NoInheritGraduateStudent로 코드 중복 체감
        ↓
다단계 상속 — DoctoralStudent → GraduateStudent → Student
        ↓
protected 접근제어자 실험 — 자식 클래스 vs 상속 관계 아닌 클래스</code></pre>
<h1 id="📌-오늘의-학습-키워드">📌 오늘의 학습 키워드</h1>
<ul>
<li>캡슐화(은닉화), 조기 반환(early return)</li>
<li>상속, <code>super()</code> 연쇄 호출, 다단계 상속</li>
<li>접근제어자(<code>private</code> / <code>protected</code> / <code>public</code>)</li>
<li>코드 중복, 유지보수 비용</li>
</ul>
<hr />
<h1 id="0-개념-정리-키워드-사전">0. 개념 정리 (키워드 사전)</h1>
<blockquote>
<p>캡슐화는 값을 숨기는 것보다 &quot;정해진 통로로만 바꾸게 강제하는 것&quot;이 핵심이고, <code>protected</code>는 &quot;가족(자식 클래스)에게는 열어주되 남에게는 닫는&quot; 중간 지점이다.</p>
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
<td><strong>캡슐화 (Encapsulation)</strong></td>
<td>필드를 <code>private</code>으로 숨기고, 정해진 메서드로만 값을 바꾸게 만드는 것</td>
<td>목적은 &quot;숨기기&quot;가 아니라 &quot;검증을 반드시 거치게 강제하는 것&quot;</td>
</tr>
<tr>
<td><strong>조기 반환 (Early Return)</strong></td>
<td>조건에 안 맞으면 <code>return</code>으로 메서드를 먼저 끝내는 패턴</td>
<td><code>if-else</code>로 감싸는 것보다 들여쓰기가 얕아져 가독성이 좋아짐</td>
</tr>
<tr>
<td><strong>상속 (Inheritance)</strong></td>
<td>자식 클래스가 부모의 필드·메서드를 물려받는 것</td>
<td>목적은 코드 중복 제거뿐 아니라 유지보수 비용 절감</td>
</tr>
<tr>
<td><strong>다단계 상속</strong></td>
<td>상속 관계가 두 단계 이상 이어지는 것</td>
<td><code>DoctoralStudent extends GraduateStudent extends Student</code></td>
</tr>
<tr>
<td><strong><code>super()</code></strong></td>
<td>자식 생성자에서 부모 생성자를 호출하는 키워드</td>
<td>다단계 상속에서는 이 호출이 연쇄적으로 일어남</td>
</tr>
<tr>
<td><strong><code>private</code></strong></td>
<td>선언된 클래스 내부에서만 접근 가능</td>
<td>외부·자식 모두 접근 불가</td>
</tr>
<tr>
<td><strong><code>protected</code></strong></td>
<td>같은 패키지 + 다른 패키지의 자식 클래스까지 접근 가능</td>
<td>상속 관계가 아닌 클래스에는 여전히 막힘</td>
</tr>
</tbody></table>
<h3 id="코드로-다시-보기">코드로 다시 보기</h3>
<pre><code class="language-java">// 캡슐화: 조기 반환으로 검증을 강제
public void setAge(int age) {
    if (age &lt;= 0 || age &gt;= 100) {
        System.out.println(&quot;잘못된 나이 입력입니다.&quot;);
        return;                 // 조건에 안 맞으면 여기서 즉시 종료
    }
    this.age = age;
}</code></pre>
<pre><code class="language-java">// 다단계 상속: super() 호출이 연쇄적으로 일어남
public class DoctoralStudent extends GraduateStudent {
    private String advisor;

    public DoctoralStudent(String name, int age, int grade, String major, String advisor) {
        super(name, age, grade, major);   // GraduateStudent 생성자 호출
                                           // → 그 안에서 다시 Student 생성자 호출
        this.advisor = advisor;
    }
}</code></pre>
<hr />
<h1 id="1-캡슐화은닉화-깊이-파기">1. 캡슐화(은닉화) 깊이 파기</h1>
<p><code>private</code> 필드 + <code>setter</code>가 왜 &quot;안전장치&quot;인지, <code>public</code> 필드와 직접 비교해서 확인했다.</p>
<pre><code class="language-java">public void setAge(int age) {
    if (age &lt;= 0 || age &gt;= 100) {
        System.out.println(&quot;잘못된 나이 입력입니다.&quot;);
        return;
    }
    this.age = age;
}</code></pre>
<p>처음엔 <code>if</code> 블록 안에서 <code>return;</code>을 먼저 쓰고 그 아래에 출력문을 두는 실수를 했는데, <code>return</code> 이후의 코드는 절대 실행되지 않는 &quot;도달 불가능한 코드(unreachable code)&quot;라 컴파일 에러가 났다. 출력 → <code>return</code> 순서로 바꿔서 해결했다. 또 처음엔 <code>if-else</code>로 감쌌었는데, <code>if</code> 블록 안에 이미 <code>return</code>이 있으면 그 아래 코드는 조건이 거짓일 때만 실행되는 게 보장되므로 <code>else</code> 없이 바로 이어써도 의미가 같다는 것도 정리했다(조기 반환 패턴).</p>
<p>이어서 필드를 전부 <code>public</code>으로만 열어둔 <code>UnsafeStudent</code> 클래스를 따로 만들어 비교했다.</p>
<pre><code class="language-java">public class UnsafeStudent {
    public int age;
}</code></pre>
<pre><code class="language-java">UnsafeStudent u = new UnsafeStudent();
u.age = -999;
System.out.println(u.age);   // -999가 검증 없이 그대로 출력됨</code></pre>
<p>처음엔 &quot;<code>public</code> 필드에도 검증 메서드를 따로 만들면 <code>private</code>+<code>setter</code>와 똑같지 않나&quot;라는 의문이 들었다. 하지만 핵심은 검증 로직의 존재 여부가 아니라 <strong>그 검증을 우회할 수 있는 다른 통로가 있는지</strong>였다. <code>public</code> 필드는 검증 메서드를 만들어도 필드에 직접 접근하는 &quot;뒷문&quot;이 항상 열려 있어 검증을 그냥 지나칠 수 있는 반면, <code>private</code> + <code>setter</code>는 값을 바꿀 수 있는 통로가 <code>setter</code> 하나뿐이라 검증이 강제된다.</p>
<hr />
<h1 id="2-상속-깊이-파기">2. 상속 깊이 파기</h1>
<h2 id="상속-없이-만들면">상속 없이 만들면?</h2>
<p><code>GraduateStudent</code>를 상속 없이 처음부터 새로 만든 <code>NoInheritGraduateStudent</code>를 작성해보며 코드 중복을 직접 체감했다.</p>
<pre><code class="language-java">// 상속 버전
public class GraduateStudent extends Student {
    private String major;

    public GraduateStudent(String name, int age, int grade, String major) {
        super(name, age, grade);
        this.major = major;
    }

    @Override
    public String checkGrade() {
        return major + &quot;전공 대학원생입니다&quot;;
    }
}</code></pre>
<pre><code class="language-java">// 비상속 버전 — getName/setName/getAge/setAge/getGrade/setGrade/checkGrade/evaluate까지
// Student와 완전히 똑같은 코드 8개가 그대로 중복됨</code></pre>
<p>두 버전을 나란히 놓고 비교하니, 상속을 안 쓰면 getter/setter 6개와 <code>checkGrade()</code>, <code>evaluate()</code>가 <code>Student</code>와 토씨 하나 안 다르게 중복된다는 걸 확인했다. 더 중요했던 건 유지보수 관점이었는데, 나중에 <code>Student</code>의 <code>checkGrade()</code> 로직을 고쳐야 할 때 상속 버전은 부모 한 곳만 고치면 자식들이 자동으로 따라오지만, 비상속 버전은 복붙된 클래스 수만큼 똑같은 수정을 반복해야 하고 하나라도 빠뜨리면 그 클래스만 옛날 로직으로 남는 버그가 생긴다는 걸 정리했다.</p>
<h2 id="다단계-상속">다단계 상속</h2>
<p><code>GraduateStudent</code>를 다시 부모로 하는 <code>DoctoralStudent</code>를 만들어 3단계 상속 체인을 구성했다.</p>
<pre><code class="language-java">public class DoctoralStudent extends GraduateStudent {
    private String advisor;

    public DoctoralStudent(String name, int age, int grade, String major, String advisor) {
        super(name, age, grade, major);
        this.advisor = advisor;
    }

    @Override
    public String checkGrade() {
        return &quot;지도교수 &quot; + advisor + &quot;님과 교육중인 훈련과정입니다.&quot;;
    }
}</code></pre>
<p>처음엔 생성자 매개변수 목록에 <code>advisor</code>를 빼먹고 <code>this.advisor = advisor;</code>만 써서, 존재하지 않는 매개변수를 참조하는 실수를 했다. 매개변수에 <code>String advisor</code>를 추가해서 해결했다.</p>
<pre><code class="language-java">Student stu = doc;                  // 최상위 부모 타입 변수에 담기
System.out.println(stu.checkGrade());   // 그래도 DoctoralStudent 버전이 실행됨</code></pre>
<p><code>Student</code> 타입 변수에 담아 호출해도 <code>DoctoralStudent</code>가 오버라이드한 버전이 그대로 실행되는 걸 확인했다. 상속이 여러 단계로 이어져도 &quot;실제 객체 타입 기준으로 메서드가 결정된다&quot;는 동적 바인딩 원리는 그대로 적용된다는 걸 알 수 있었다.</p>
<h2 id="protected-접근제어자-실험"><code>protected</code> 접근제어자 실험</h2>
<p><code>Student</code>의 필드를 <code>private</code>에서 <code>protected</code>로 바꾸고, 접근 가능 범위를 직접 테스트했다.</p>
<pre><code class="language-java">// GraduateStudent(자식 클래스) 안에서
this.age = age;      // protected라서 접근 가능 → 정상 컴파일

// StudentApp(상속 관계 아닌 클래스) 안에서
stu2.age = 100;       // 빨간 줄(컴파일 에러) 발생</code></pre>
<p><code>GraduateStudent</code>에서는 부모의 <code>protected</code> 필드에 직접 접근이 됐지만, 상속 관계도 아니고 패키지도 다른 <code>StudentApp</code>에서는 여전히 에러가 났다. <code>protected</code>가 <code>public</code>과 다르게 &quot;가족(자식 클래스)에게는 열어주고, 완전한 남에게는 여전히 닫혀 있다&quot;는 걸 코드로 직접 확인한 부분이었다.</p>
<hr />
<h1 id="✨-새롭게-알게-된-점">✨ 새롭게 알게 된 점</h1>
<ul>
<li>캡슐화의 핵심은 &quot;값을 숨기는 것&quot; 자체가 아니라 &quot;검증을 반드시 거치게 강제하는 것&quot;이다.</li>
<li><code>if</code> 블록 안에 <code>return</code>이 있으면 그 아래는 <code>else</code> 없이 이어써도 의미가 같다 (조기 반환 패턴).</li>
<li>상속의 진짜 이득은 코드 줄 수 감소보다 &quot;한 곳만 고치면 자식들이 자동으로 따라온다&quot;는 유지보수 측면에 있다.</li>
<li><code>super(...)</code>는 부모의 생성자를 그대로 호출하는 것이며, 다단계 상속에서는 이 호출이 연쇄적으로 일어난다.</li>
<li><code>protected</code>는 패키지가 달라도 자식 클래스 내부 코드에서는 접근을 허용하지만, 상속 관계가 아닌 클래스에는 여전히 접근을 막는다.</li>
</ul>
<hr />
<h1 id="🤔-헷갈렸던-점">🤔 헷갈렸던 점</h1>
<ul>
<li><strong>캡슐화 우회 가능성</strong>: <code>public</code> 필드에 별도 검증 메서드를 추가하면 <code>private</code>+<code>setter</code>와 같은 효과 아닌가 헷갈렸는데, &quot;검증 로직의 존재 여부&quot;가 아니라 &quot;그 검증을 우회할 수 있는 다른 통로(필드 직접 접근)가 있는지 없는지&quot;가 진짜 차이였다.</li>
<li><strong><code>protected</code>의 허용 범위</strong>: 상속 관계라면 무조건 접근 가능한 줄 알았는데, 정확히는 &quot;다른 패키지여도 자식 클래스 코드 내부에서만&quot; 허용되는 것이었고, 상속 관계가 아닌 클래스에는 여전히 막힌다는 걸 직접 에러를 보고 나서야 확실히 구분했다.</li>
</ul>
<hr />
<h1 id="다음에-할-일">다음에 할 일</h1>
<ul>
<li><input disabled="" type="checkbox" /> 다형성 — 변수 타입의 다형성 (<code>instanceof</code> + 안전한 다운캐스팅)</li>
<li><input disabled="" type="checkbox" /> 다형성 — 매개변수의 다형성 (부모 타입 매개변수로 여러 자식 타입 한 번에 받기)</li>
<li><input disabled="" type="checkbox" /> 다형성 — 메서드의 다형성 (오버라이딩, 타입 분기 없이 배열 순회하며 다형성 체감)</li>
</ul>
<hr />
<h1 id="💡-오늘의-회고">💡 오늘의 회고</h1>
<p>그동안 &quot;이렇게 하면 된다&quot;고만 알고 있던 캡슐화와 상속을, 반대 상황(검증 없는 <code>public</code> 필드, 상속 없이 처음부터 다시 만들기)과 나란히 비교해보면서 &quot;왜 이렇게 하는지&quot;를 훨씬 선명하게 이해할 수 있었다. 특히 상속 없이 <code>NoInheritGraduateStudent</code>를 직접 타이핑해보면서 느낀 중복의 답답함이, 상속이 왜 필요한지를 이론보다 훨씬 강하게 체감하게 해줬다. <code>protected</code> 실험에서 예상대로 자식 클래스는 되고 상속 관계가 아닌 클래스는 막히는 걸 눈으로 확인한 것도 좋았다. 다음엔 다형성 세 가지(변수 타입·매개변수·메서드)를 이어서 정리하며, 오늘 다진 캡슐화·상속 개념과 자연스럽게 연결해볼 예정이다.</p>