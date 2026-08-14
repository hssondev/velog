<h2 id="1-오늘의-맥락--배열-oop-4대-특징을-직접-코드로-확인한-하루">1. 오늘의 맥락 — 배열, OOP 4대 특징을 직접 코드로 확인한 하루</h2>
<p>오전엔 배열(array)과 Lombok 빌더 패턴으로 가볍게 시작했고, 오후엔 어제 정리해둔 <strong>this/super/오버라이딩</strong> 개념이 실제로 왜 필요한지 <code>PersonDTO → StudentDTO/TeacherDTO/ManagerDTO</code> 상속 구조로 직접 확인했다. OOP의 4대 특징(캡슐화, 상속, 다형성, 추상화)이 오늘 처음 이름과 코드로 함께 등장했다.</p>
<hr />
<h2 id="2-배열array이란">2. 배열(Array)이란</h2>
<p><strong>Java</strong> · <code>AryApp.java</code></p>
<pre><code class="language-java">String [] ary = new String [10];
ary[0] = &quot;A&quot;;

for (int idx = 0; idx &lt; ary.length; idx++) {
    System.out.print(ary[idx] + &quot;\t&quot;);
}

// 향상된 for문(enhanced for)
for (String data : ary) {
    System.out.print(data + &quot;\t&quot;);
}</code></pre>
<p><strong>코드 리뷰</strong></p>
<ul>
<li>배열은 <strong>같은 타입의 값 여러 개를 하나의 그릇에 담는 참조타입</strong>. 한 번 크기를 정하면(<code>new String[10]</code>) 실행 중에 늘리거나 줄일 수 없는 <strong>고정 길이</strong>라는 게 핵심.</li>
<li>인덱스는 0부터 시작. <code>ary.length</code>로 크기를 알 수 있음.</li>
<li>일반 <code>for</code>와 <code>for(타입 변수 : 배열)</code>(향상된 for) 둘 다 순회 가능 — 향상된 for는 인덱스가 필요 없을 때 더 간결함.</li>
</ul>
<p><strong>예상 실행 결과</strong></p>
<pre><code>A    null    null    null    null    null    null    null    null    null    
A    null    null    null    null    null    null    null    null    null    </code></pre><p>(0번만 값을 넣었고, 나머지는 참조타입 배열의 기본값인 <code>null</code>)</p>
<hr />
<h2 id="3-repository-→-service-계층--어제-정리한-구조가-실제-코드로">3. Repository → Service 계층 — 어제 정리한 구조가 실제 코드로</h2>
<p><strong>Java</strong> · <code>BlogRepository.java</code> / <code>BlogService.java</code></p>
<pre><code class="language-java">@Builder @Getter @Setter
public class BlogRepository {
    private BlogResponseDTO[] list = new BlogResponseDTO[10];
    public BlogResponseDTO[] blogs() { return list; }
}

@Builder @Getter @Setter
public class BlogService {
    public BlogResponseDTO[] blogs() {
        return BlogRepository.builder().build().blogs();
    }
}</code></pre>
<p><strong>코드 리뷰</strong></p>
<ul>
<li>어제 메모리에 정리해둔 <strong>react → controller → service → data(repository)</strong> 계층 구조가, 오늘 <code>BlogService</code>가 <code>BlogRepository</code>를 호출하는 코드로 실제 등장함 — service는 로직만, repository는 데이터 접근만 담당한다는 역할 분리 그대로.</li>
<li><code>@Builder</code>(lombok)로 <code>BlogResponseDTO.builder().status(200).message(&quot;good&quot;).build()</code>처럼 생성자 없이도 객체를 만들 수 있었음.</li>
</ul>
<hr />
<h2 id="4-oop-4대-특징">4. OOP 4대 특징</h2>
<table>
<thead>
<tr>
<th>특징</th>
<th>한 줄 정리</th>
<th>오늘 코드에서</th>
</tr>
</thead>
<tbody><tr>
<td><strong>캡슐화</strong></td>
<td>변수를 <code>private</code>으로 숨기고 getter/setter로만 접근</td>
<td><code>PersonDTO</code>의 <code>name</code>, <code>age</code>, <code>address</code></td>
</tr>
<tr>
<td><strong>상속</strong></td>
<td>부모의 것을 자식이 물려받음 (<code>extends</code>)</td>
<td><code>StudentDTO extends PersonDTO</code></td>
</tr>
<tr>
<td><strong>다형성</strong></td>
<td>같은 타입/메서드 호출인데 실제로는 다르게 동작</td>
<td>아래 5, 6번에서 자세히</td>
</tr>
<tr>
<td><strong>추상화</strong></td>
<td>공통되는 특징만 뽑아서 하나의 틀(부모)로 정리</td>
<td><code>PersonDTO</code>가 name/age/address라는 공통 속성만 추림</td>
</tr>
</tbody></table>
<p>→ <strong>왜 이 넷이 묶여서 다니냐면</strong>, 사실 서로 맞물려 있다. 공통점을 뽑아(추상화) 부모를 만들고, 자식이 그걸 물려받고(상속), 내부는 숨기고 필요한 통로만 열어두고(캡슐화), 그 결과 같은 호출도 객체마다 다르게 동작(다형성)하게 되는 흐름이다.</p>
<hr />
<h2 id="5-상속-실습--persondto와-자식들">5. 상속 실습 — <code>PersonDTO</code>와 자식들</h2>
<p><strong>Java</strong> · <code>PersonDTO.java</code> / <code>StudentDTO.java</code></p>
<pre><code class="language-java">public class PersonDTO {
    private String name;
    private int age;
    private String address;

    public PersonDTO() {}
    public PersonDTO(String name, int age, String address) {
        this.name = name; this.age = age; this.address = address;
    }
    // getter/setter 생략

    public String personInfo() {
        return &quot;name=&quot; + name + &quot;, age=&quot; + age + &quot;, address=&quot; + address;
    }
}

public class StudentDTO extends PersonDTO {
    private String ssn;

    public StudentDTO(String name, int age, String address, String ssn) {
        super(name, age, address);
        this.ssn = ssn;
    }

    @Override
    public String personInfo() {
        return super.personInfo() + &quot;, ssn= &quot; + ssn;
    }
}</code></pre>
<p><strong>코드 리뷰</strong></p>
<ul>
<li><code>super(name, age, address)</code>로 부모 생성자를 먼저 호출 — 어제 정리한 대로 &quot;부모 몫부터 완성하고 자식을 마저 채운다&quot;는 순서 그대로.</li>
<li><code>personInfo()</code>도 어제 정리한 대로, <code>super.personInfo()</code>(부모 결과)에 <code>ssn</code>만 덧붙이는 오버라이딩. <code>TeacherDTO</code>(subject), <code>ManagerDTO</code>(dept)도 같은 패턴.</li>
</ul>
<p><strong>예상 실행 결과</strong> (<code>new StudentDTO(&quot;손호성&quot;,29,&quot;서울&quot;,&quot;2026&quot;)</code>)</p>
<pre><code>name=손호성, age=29, address=서울, ssn= 2026</code></pre><hr />
<h2 id="6-다형성--변수타입은-부모-실제-객체는-자식">6. 다형성 — 변수타입은 부모, 실제 객체는 자식</h2>
<p><strong>Java</strong> · <code>OopApp.java</code></p>
<pre><code class="language-java">PersonDTO manager = new ManagerDTO(&quot;손호성&quot;, 29, &quot;서울&quot;, &quot;교육사무국&quot;);
System.out.println(((ManagerDTO) manager).getDept());

PersonDTO[] ary = new PersonDTO[3];
ary[0] = new TeacherDTO(&quot;임정섭&quot;, 20, &quot;서울&quot;, &quot;react&quot;);
ary[1] = new ManagerDTO(&quot;김혜림&quot;, 20, &quot;서울&quot;, &quot;교육팀&quot;);
ary[2] = new StudentDTO(&quot;손호성&quot;, 20, &quot;서울&quot;, &quot;2026&quot;);

for (int idx = 0; idx &lt; ary.length; idx++) {
    System.out.println(ary[idx].personInfo());
}</code></pre>
<p><strong>코드 리뷰</strong></p>
<ul>
<li><code>PersonDTO manager = new ManagerDTO(...)</code> — <strong>선언한 타입(PersonDTO)과 실제 만들어진 객체(ManagerDTO)가 다르다.</strong> 이게 오늘 배운 &quot;변수타입의 다형성&quot;의 핵심.</li>
<li><code>ManagerDTO</code>만 가진 <code>getDept()</code>를 쓰려면, <code>manager</code> 변수 자체는 <code>PersonDTO</code> 타입이라 바로 못 부르고 <code>((ManagerDTO) manager).getDept()</code>처럼 <strong>캐스팅</strong>해서 &quot;사실 이건 ManagerDTO야&quot;라고 자바에게 알려줘야 했음.</li>
<li><code>ary[idx].personInfo()</code>는 캐스팅 없이 바로 호출 가능. 배열 타입은 <code>PersonDTO[]</code>지만, 각 요소가 실제로 어떤 자식인지에 따라 <strong>각자 오버라이딩한 personInfo()가 자동으로 실행</strong>됨 — 이게 다형성이 실무에서 쓰이는 방식.</li>
</ul>
<p><strong>예상 실행 결과</strong></p>
<pre><code>name=임정섭, age=20, address=서울, subject= react
name=김혜림, age=20, address=서울, dept= 교육팀
name=손호성, age=20, address=서울, ssn= 2026</code></pre><hr />
<h3 id="7🤔-헷갈렸던-점">7.🤔 헷갈렸던 점</h3>
<p><strong>1) 캐스팅(형변환) — 왜 &quot;묵시적&quot;과 &quot;명시적&quot;로 나뉘나</strong></p>
<p>핵심 질문 하나로 정리하면: <strong>&quot;바꿔도 값이 안전한가, 위험할 수 있는가&quot;</strong>다.</p>
<ul>
<li><strong>묵시적(자동)</strong>: <code>int → long</code>처럼, 작은 그릇의 값을 더 큰 그릇에 옮기는 거라 <strong>절대 값이 잘릴 일이 없다.</strong> 그래서 자바가 알아서 바꿔준다. (어제 배운 byte/short/int 캐스팅과 같은 원리)</li>
<li><strong>명시적(직접 표시)</strong>: 반대로 값이 잘리거나 잘못될 위험이 있을 때는, 개발자가 <code>(타입)</code>을 직접 써서 &quot;위험한 거 알고 하는 거야&quot;라고 표시해야 한다.</li>
</ul>
<p><strong>오늘 새로 나온 &quot;상속관계의 캐스팅&quot;도 같은 원리인데, 대상이 숫자가 아니라 객체라는 점만 다르다.</strong></p>
<pre><code class="language-java">PersonDTO manager = new ManagerDTO(...);   // 선언은 PersonDTO, 실제 객체는 ManagerDTO
manager.getDept();                          // ❌ 에러! PersonDTO엔 getDept()가 없음
((ManagerDTO) manager).getDept();           // ✅ &quot;얘 사실 ManagerDTO야&quot;라고 알려줘야 함</code></pre>
<ul>
<li><code>manager</code>라는 변수는 <strong>타입만 PersonDTO</strong>로 선언됐을 뿐, 실제로 메모리에 만들어진 건 <code>ManagerDTO</code>다.</li>
<li><code>PersonDTO</code>에는 <code>getDept()</code>라는 메서드가 없으니, 그냥은 못 부른다.</li>
<li><code>(ManagerDTO)</code>를 앞에 붙여서 &quot;이 변수, 겉보기엔 PersonDTO지만 사실 ManagerDTO야&quot;라고 자바에게 알려줘야 <code>getDept()</code>를 쓸 수 있다.</li>
<li>⚠️ 만약 진짜로는 <code>TeacherDTO</code>인데 <code>(ManagerDTO)</code>로 잘못 캐스팅하면? <strong>실행 도중 에러(ClassCastException)</strong>가 난다. 숫자 캐스팅은 최악의 경우 값이 좀 틀어지는 정도지만, 객체 캐스팅은 아예 프로그램이 멈출 수 있다는 게 다른 점이다.</li>
</ul>
<hr />
<p><strong>2) 오버라이딩 vs 오버로딩 — 이름이 비슷해서 헷갈리지만 완전히 다른 개념</strong></p>
<p>가장 쉬운 구분법: <strong>&quot;누가 다시 정의하는가&quot;</strong>로 기억하면 된다.</p>
<table>
<thead>
<tr>
<th></th>
<th>오버라이딩 (Overriding)</th>
<th>오버로딩 (Overloading)</th>
</tr>
</thead>
<tbody><tr>
<td>한 줄 정의</td>
<td>자식이 부모 걸 <strong>덮어쓰기</strong></td>
<td>한 클래스 안에 <strong>같은 이름 여러 벌</strong> 만들기</td>
</tr>
<tr>
<td>누구 사이에서?</td>
<td>부모-자식 (상속 관계)</td>
<td>같은 클래스 안</td>
</tr>
<tr>
<td>조건</td>
<td>이름·매개변수·반환타입까지 <strong>완전히 동일</strong></td>
<td>이름은 같고, 매개변수만 <strong>다름</strong></td>
</tr>
<tr>
<td>예시</td>
<td><code>StudentDTO</code>가 <code>PersonDTO</code>의 <code>personInfo()</code>를 재정의</td>
<td><code>PersonDTO()</code> / <code>PersonDTO(name)</code> / <code>PersonDTO(name,age,address)</code></td>
</tr>
</tbody></table>
<ul>
<li>오버라이딩은 &quot;이미 있는 메서드를 자식이 자기 방식대로 바꾸는 것&quot; → 그래서 부모-자식 관계가 반드시 필요하다.</li>
<li>오버로딩은 &quot;이름은 같지만 받는 값이 다른, 여러 버전을 준비해두는 것&quot; → 상속과는 무관하고, 한 클래스 안에서도 얼마든지 가능하다.</li>
</ul>
<hr />
<p><strong>3) <code>this</code> vs <code>super</code> — 한 문장으로 요약</strong></p>
<blockquote>
<p><strong><code>this</code>는 &quot;나(지금 이 객체)&quot;, <code>super</code>는 &quot;내 바로 위 부모&quot;</strong></p>
</blockquote>
<pre><code class="language-java">public StudentDTO(String name, int age, String address, String ssn) {
    super(name, age, address);   // 부모(PersonDTO) 몫부터 먼저 채운다
    this.ssn = ssn;              // 그다음 내(Student) 몫을 채운다
}</code></pre>
<ul>
<li><code>super(...)</code>를 생성자 맨 앞에서 부르면 &quot;부모 초기화부터 먼저 끝내라&quot;는 뜻.</li>
<li><code>this.ssn</code>은 &quot;지금 만들어지는 이 인스턴스의 ssn 필드&quot;를 가리키는 것.</li>
</ul>
<pre><code class="language-java">public String personInfo() {
    return super.personInfo() + &quot;, ssn= &quot; + ssn;   // 부모가 만든 결과에 내 것만 덧붙임
}</code></pre>
<ul>
<li>여기서 <code>super.personInfo()</code>는 &quot;부모 버전의 personInfo()를 실행해서 그 결과를 가져와라&quot;는 뜻 — 자식이 처음부터 다시 안 만들고, 부모 결과를 재사용하는 것.</li>
</ul>
<hr />
<h2 id="8-아직-남겨둔-것">8. 아직 남겨둔 것</h2>
<ul>
<li>매개변수의 다형성(overloading과 연결되는 개념으로 보임) — 내일 예정</li>
<li><code>protected</code> 접근지정자 — 오늘 이름만 나왔고 <code>public</code>/<code>private</code>과 실질적 차이는 안 다룸</li>
<li><code>instanceof</code> — 코드에 주석으로만 등장(<code>if(per instanceof TeacherDTO)</code>), 직접 실행은 못 해봄</li>
<li>has-a vs is-a 차이 — 오늘 한 줄 요약에만 등장, 본문에서 다루지 않음</li>
</ul>
<hr />
<h2 id="오늘의-한-줄-요약">오늘의 한 줄 요약</h2>
<p>배열의 기본 구조를 익히고, PersonDTO 상속 구조로 캡슐화·상속·다형성·추상화를 직접 코드로 확인했으며, 어제 정리한 this/super/오버라이딩이 실제로 어떻게 쓰이는지 연결해서 이해한 하루였다.</p>
<hr />
<h2 id="마무리-회고">마무리 회고</h2>
<p>어제는 this/super/오버라이딩을 개념으로만 정리했는데, 오늘 <code>super(name, age, address)</code>와 <code>super.personInfo()</code>를 직접 코드에서 쓰면서 왜 그 개념이 필요했는지 체감했다. 특히 <code>PersonDTO manager = new ManagerDTO(...)</code>처럼 <strong>선언 타입과 실제 객체가 다를 수 있다</strong>는 걸 처음 보고 신기했는데, 배열 하나(<code>PersonDTO[]</code>)에 서로 다른 자식들을 섞어 담고도 <code>personInfo()</code> 한 줄로 각자 다른 결과가 나오는 걸 보니 다형성이 왜 유용한지 감이 왔다.</p>