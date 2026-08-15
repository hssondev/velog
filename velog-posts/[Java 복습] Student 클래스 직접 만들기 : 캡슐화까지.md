<blockquote>
<p><strong>한 줄 요약</strong></p>
<p><code>Student</code> 클래스를 처음부터 직접 만들어보며 필드 선언 → 생성자 → 메서드(void/리턴형) → 캡슐화(getter/setter)까지 단계별로 짜고, 그 과정에서 반복됐던 실수 패턴(역할 분리, 중괄호 범위, <code>this</code>)을 정리했다.</p>
</blockquote>
<hr />
<h1 id="📚-오늘-학습한-커리큘럼">📚 오늘 학습한 커리큘럼</h1>
<pre><code>클래스 정의 (필드만) — Student
        ↓
객체 생성 후 필드 직접 접근 — StudentApp
        ↓
생성자 추가 — Student(name, age, grade)
        ↓
void 메서드 (매개변수 있음) — changeGrade()
        ↓
값을 리턴하는 메서드 — isAdult()
        ↓
캡슐화 — private 필드 + getter/setter, 생성자 오버로딩</code></pre><h1 id="📌-오늘의-학습-키워드">📌 오늘의 학습 키워드</h1>
<ul>
<li>클래스와 실행 클래스의 역할 분리</li>
<li>생성자, <code>this</code></li>
<li><code>void</code> 메서드 vs 리턴 메서드</li>
<li>캡슐화(getter/setter), 생성자 오버로딩</li>
</ul>
<hr />
<h1 id="0-개념-정리-키워드-사전">0. 개념 정리 (키워드 사전)</h1>
<p>본격적인 코드 정리에 앞서, 오늘 사용한 핵심 용어들을 표로 정리한다.</p>
<table>
<thead>
<tr>
<th>키워드</th>
<th>한 줄 정의</th>
<th>비고 / 예시</th>
</tr>
</thead>
<tbody><tr>
<td><strong>클래스 (Class)</strong></td>
<td>객체를 만들기 위한 <strong>설계도</strong>. 어떤 필드와 메서드를 가질지 정의한 틀</td>
<td>그 자체로는 실체가 없고 <code>new</code>로 찍어내야 객체가 됨</td>
</tr>
<tr>
<td><strong>객체 / 인스턴스 (Object / Instance)</strong></td>
<td>클래스를 바탕으로 <strong>실제 힙 메모리에 만들어진 것</strong></td>
<td><code>new Student()</code> 실행 시 태어남. 이 과정을 &quot;인스턴스화&quot;라 부름</td>
</tr>
<tr>
<td><strong>필드 (Field)</strong></td>
<td>클래스 안에 선언된 <strong>변수</strong>, 객체가 가진 데이터(상태)</td>
<td><code>name</code>, <code>age</code>, <code>grade</code></td>
</tr>
<tr>
<td><strong>참조 변수 (Reference Variable)</strong></td>
<td>객체 자체가 아니라, <strong>객체가 저장된 힙 주소</strong>를 담는 변수</td>
<td><code>Student stu = new Student();</code>에서 <code>stu</code></td>
</tr>
<tr>
<td><strong>생성자 (Constructor)</strong></td>
<td>객체가 생성되는 순간 <strong>딱 한 번</strong> 자동 실행되는 코드 블록</td>
<td>이름이 클래스명과 동일, 리턴 타입 없음</td>
</tr>
<tr>
<td><strong>메서드 (Method)</strong></td>
<td>클래스 안에 정의된 <strong>기능(동작)</strong></td>
<td>이름 자유, 리턴 타입(<code>void</code> 포함) 필수</td>
</tr>
<tr>
<td><strong>매개변수 (Parameter)</strong></td>
<td>메서드/생성자가 실행될 때 <strong>외부에서 값을 전달받는</strong> 변수</td>
<td>호출 시 넘기는 실제 값은 &quot;인자(argument)&quot;로 구분</td>
</tr>
<tr>
<td><strong><code>this</code></strong></td>
<td><strong>&quot;지금 이 객체 자신&quot;</strong>을 가리키는 키워드</td>
<td>매개변수와 필드 이름이 같을 때 <code>this.필드</code>로 구분</td>
</tr>
<tr>
<td><strong>캡슐화 (Encapsulation)</strong></td>
<td>필드를 <code>private</code>으로 감추고, getter/setter로만 접근하게 만드는 것</td>
<td>잘못된 값 유입 방지, 값 검증 로직 추가 가능</td>
</tr>
<tr>
<td><strong>Getter / Setter</strong></td>
<td>캡슐화를 위한 메서드 쌍</td>
<td>Getter: 값을 꺼냄(<code>get+필드명</code>) / Setter: 값을 바꿈(<code>set+필드명</code>)</td>
</tr>
<tr>
<td><strong>오버로딩 (Overloading)</strong></td>
<td><strong>같은 클래스 안</strong>에서 이름은 같고 매개변수가 다른 메서드/생성자를 여러 개 만드는 것</td>
<td><code>Student()</code>와 <code>Student(name, age, grade)</code></td>
</tr>
<tr>
<td><strong><code>void</code></strong></td>
<td>리턴 타입 자리에 쓰며, <strong>&quot;실행만 하고 값을 돌려주지 않는다&quot;</strong>는 뜻</td>
<td>setter류 메서드에서 흔히 사용</td>
</tr>
</tbody></table>
<h3 id="코드로-다시-보기">코드로 다시 보기</h3>
<pre><code class="language-java">public class Student {              // 클래스
    private String name;            // 필드 (private → 캡슐화)
    private int age;
    private int grade;

    public Student() { }            // 생성자 ① (매개변수 없음)

    public Student(String name, int age, int grade) {   // 생성자 ② (오버로딩)
        this.name = name;           // this로 매개변수 name과 필드 name을 구분
        this.age = age;
        this.grade = grade;
    }

    public String getName() { return name; }        // Getter (메서드)
    public void setName(String name) { this.name = name; }   // Setter (void 메서드)
}</code></pre>
<pre><code class="language-java">Student stu = new Student(&quot;손호성&quot;, 20, 3);   // stu = 참조 변수, new Student(...) = 인스턴스화</code></pre>
<hr />
<pre><code class="language-java">package features.practice;

public class Student {
    String name;
    int age;
    int grade;
}</code></pre>
<ul>
<li><code>Student</code>는 데이터를 담는 &quot;부품&quot; 클래스로 두고, <code>main</code>은 별도의 <code>StudentApp</code>에 두기로 역할을 나눴다.</li>
</ul>
<hr />
<h1 id="2-객체-생성-후-필드-직접-접근">2. 객체 생성 후 필드 직접 접근</h1>
<pre><code class="language-java">public class StudentApp {
    public static void main(String[] args) {
        Student stu = new Student();
        stu.name  = &quot;손호성&quot;;
        stu.age   = 20;
        stu.grade = 3;

        System.out.println(stu.name);
        System.out.println(stu.age);
        System.out.println(stu.grade);
    }
}</code></pre>
<ul>
<li>처음엔 필드를 <code>Student</code>가 아니라 <code>StudentApp</code>(실행 클래스) 안에 직접 선언하고 <code>new StudentApp()</code>으로 객체를 만들었었다. <code>Student</code>(데이터를 담는 곳)와 <code>StudentApp</code>(실행하는 곳)의 역할이 뒤섞인 상태였고, 필드는 반드시 <code>Student</code> 쪽에 있어야 한다는 걸 다시 짚었다.</li>
</ul>
<hr />
<h1 id="3-생성자-추가">3. 생성자 추가</h1>
<pre><code class="language-java">public class Student {
    String name;
    int age;
    int grade;

    public Student(String name, int age, int grade) {
        this.name = name;
        this.age = age;
        this.grade = grade;
    }
}</code></pre>
<pre><code class="language-java">Student stu = new Student(&quot;손호성&quot;, 20, 3);</code></pre>
<ul>
<li>필드 선언을 빼먹은 채 생성자만 작성해서 <code>this.name</code>이 가리킬 대상 자체가 없었던 실수, <code>this.name;</code>처럼 <code>=</code> 없이 써서 아무 값도 대입되지 않았던 실수를 겪었다.</li>
<li><code>this</code>가 필요한 이유: 매개변수 <code>name</code>과 필드 <code>name</code>은 이름은 같지만 서로 다른 존재라서, <code>this</code> 없이 <code>name = name;</code>이라고 쓰면 매개변수를 자기 자신에게 대입하는 의미 없는 코드가 되어버린다. <code>this.name = name;</code>이라고 써야 &quot;이 객체의 name 필드에, 매개변수로 받은 값을 저장하라&quot;는 뜻이 된다.</li>
</ul>
<hr />
<h1 id="4-void-메서드-매개변수-있음">4. <code>void</code> 메서드 (매개변수 있음)</h1>
<pre><code class="language-java">public void changeGrade(int newGrade){
    this.grade = newGrade;
    System.out.println(&quot;학년이&quot; + grade + &quot;학년으로 변경되었습니다&quot;);
}</code></pre>
<ul>
<li>메서드 이름을 <code>ChangeGrade</code>(대문자 시작)로 지었다가, 자바 관례(camelCase)에 맞춰 <code>changeGrade</code>로 고쳤다.</li>
<li><code>newGrade</code>를 필드로도 따로 선언했었는데, 매개변수는 메서드 실행 중에만 잠깐 존재하는 값이라 필드로 만들 필요가 없다. <code>this.grade = newGrade;</code>로 진짜 필드(<code>grade</code>)에만 저장하면 충분하다.</li>
</ul>
<hr />
<h1 id="5-값을-리턴하는-메서드">5. 값을 리턴하는 메서드</h1>
<pre><code class="language-java">public boolean isAdult() {
    if (age &gt;= 20) {
        return true;
    } else {
        return false;
    }
}</code></pre>
<pre><code class="language-java">boolean adult = stu.isAdult();
System.out.println(adult);   // true</code></pre>
<ul>
<li><code>changeGrade()</code>의 닫는 중괄호 <code>}</code>를 빼먹고 바로 <code>isAdult()</code>를 이어 써서, 메서드 안에 메서드가 들어가 있는 구조 오류가 났다. 각 메서드는 독립적으로 자기 <code>{ }</code>를 완전히 닫아야 한다는 걸 확인했다.</li>
<li><code>void</code>(실행만 하고 끝)와 리턴형 메서드(결과값을 돌려줌)의 차이: <code>changeGrade</code>는 &quot;바꾸고 출력만&quot; 하면 되니 <code>void</code>, <code>isAdult</code>는 &quot;성인 여부 판단 결과&quot;를 돌려줘야 하니 <code>boolean</code>으로 선언했다.</li>
</ul>
<hr />
<h1 id="6-캡슐화--private-필드--gettersetter">6. 캡슐화 — private 필드 + getter/setter</h1>
<pre><code class="language-java">public class Student {
    private String name;
    private int age;
    private int grade;

    public Student() {
    }

    public Student(String name, int age, int grade) {
        this.name = name;
        this.age = age;
        this.grade = grade;
    }

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }

    public int getAge() { return age; }
    public void setAge(int age) { this.age = age; }

    public int getGrade() { return grade; }
    public void setGrade(int grade) { this.grade = grade; }

    public void changeGrade(int newGrade){
        this.grade = newGrade;
        System.out.println(&quot;학년이&quot; + grade + &quot;학년으로 변경되었습니다&quot;);
    }

    public boolean isAdult() {
        if (age &gt;= 20) {
            return true;
        } else {
            return false;
        }
    }
}</code></pre>
<ul>
<li>getter/setter 6개를 먼저 만들었는데, 필드를 <code>private</code>으로 바꾸는 걸 빼먹어서 캡슐화가 실제로 적용되지 않은 상태였다. <code>public</code> → <code>private</code>으로 바꾸고 나니, 이전에 <code>stu.name</code>처럼 필드에 직접 접근하던 코드들을 <code>stu.setName(...)</code>, <code>stu.getAge()</code> 형태로 바꿔야 했다.</li>
<li>매개변수 없는 생성자(<code>public Student() {}</code>)와 매개변수 3개짜리 생성자를 같이 만들면서, 이름이 같은 생성자를 매개변수 형태만 다르게 여러 개 두는 것도 <strong>생성자 오버로딩</strong>이라는 걸 다시 확인했다.</li>
</ul>
<pre><code class="language-java">Student stu1 = new Student();                        // 빈 객체
Student stu2 = new Student(&quot;손호성&quot;, 20, 3);           // 값 채운 객체</code></pre>
<hr />
<h1 id="✨-새롭게-알게-된-점">✨ 새롭게 알게 된 점</h1>
<ul>
<li>필드는 반드시 데이터를 담을 클래스에 있어야 하고, <code>main</code>이 있는 실행 클래스와는 역할이 다르다.</li>
<li><code>this</code>는 매개변수와 필드 이름이 같을 때, &quot;필드 쪽&quot;이라고 명시적으로 구분해주는 역할을 한다.</li>
<li>메서드는 이름이 클래스명과 다르고 리턴 타입이 있어야 한다는 조건으로 생성자와 구분된다.</li>
<li>getter/setter를 만드는 것과 필드를 <code>private</code>으로 바꾸는 것은 세트로 같이 해야 캡슐화가 실제로 의미를 가진다.</li>
<li>생성자도 매개변수 형태를 다르게 해서 여러 개 만들 수 있다 (생성자 오버로딩).</li>
</ul>
<hr />
<h1 id="🤔-헷갈렸던-점">🤔 헷갈렸던 점</h1>
<ul>
<li><strong>역할 분리</strong>: 데이터를 담는 클래스와 실행하는 클래스를 헷갈려서 필드나 객체 생성 위치를 잘못 넣는 실수를 반복했다.</li>
<li><strong>중괄호 <code>{ }</code> 범위</strong>: 메서드를 이어서 작성하다가 닫는 괄호를 빼먹어, 메서드 안에 메서드가 들어가는 구조 오류가 여러 번 있었다.</li>
<li><strong><code>this</code>의 필요성</strong>: <code>this</code> 없이 <code>name = name;</code>처럼 쓰면 필드에 값이 저장되지 않는다는 걸 여러 번 헷갈렸다.</li>
<li><strong>캡슐화 적용 순서</strong>: getter/setter만 만들고 필드를 <code>private</code>으로 바꾸는 걸 빼먹으면, 여전히 외부에서 필드에 직접 접근할 수 있어 캡슐화가 안 된 상태라는 걸 놓치기 쉬웠다.</li>
</ul>
<hr />
<h1 id="다음에-할-일">다음에 할 일</h1>
<ul>
<li><input disabled="" type="checkbox" /> <code>Student</code> 배열로 여러 명을 관리하는 기능 만들기 (<code>ary</code>, <code>idx</code> 활용)</li>
<li><input disabled="" type="checkbox" /> <code>Student</code>를 부모로 하는 자식 클래스(<code>EBook</code> 예제 참고) 만들어서 상속 적용해보기</li>
<li><input disabled="" type="checkbox" /> 인터페이스를 활용한 예제(<code>Discountable</code> 등)로 다형성 연습 이어가기</li>
</ul>
<hr />
<h1 id="💡-오늘의-회고">💡 오늘의 회고</h1>
<p>개념은 이전에 다 배웠다고 생각했는데, 막상 처음부터 직접 <code>Student</code> 클래스를 짜보니 필드 위치, 중괄호 범위, <code>this</code> 같은 기본적인 부분에서 계속 막혔다. 특히 &quot;데이터를 담는 클래스&quot;와 &quot;실행하는 클래스&quot;의 역할을 명확히 나누는 게 생각보다 헷갈렸는데, 여러 번 고쳐보면서 왜 나눠야 하는지 체감이 됐다. 다음엔 배열로 여러 <code>Student</code>를 관리하고, 상속/인터페이스까지 이어서 연습할 계획이다.</p>