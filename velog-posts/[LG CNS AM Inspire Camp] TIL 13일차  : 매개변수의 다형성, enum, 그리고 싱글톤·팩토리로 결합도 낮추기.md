<blockquote>
<p><strong>한 줄 요약</strong></p>
<p><code>OopService</code>에서 매개변수의 다형성으로 배열을 관리하고, <code>enum Flag</code>로 어떤 자식 객체를 만들지 정한 뒤, TV 예제로 싱글톤과 <code>BeanFactory</code>(팩토리)가 결합도를 낮추는 원리를 정리했다.</p>
</blockquote>
<hr />
<h1 id="📚-오늘-학습한-커리큘럼">📚 오늘 학습한 커리큘럼</h1>
<pre><code>OopService — 매개변수의 다형성 (setAry, getAry)
        ↓
enum Flag — makePerson()에서 어떤 자식을 만들지 결정
        ↓
findPerson() — 이름으로 검색
        ↓
캡슐화 — private 필드는 getter/setter로만 접근
        ↓
추상화 — 추상클래스(Animal) vs 인터페이스(Flyer)
        ↓
약결합(loose coupling) — TV 인터페이스로 브랜드 의존성 제거
        ↓
싱글톤 — SamsungTV/LgTV를 각각 하나씩만 생성
        ↓
팩토리(BeanFactory) — &quot;어떤 브랜드를 쓸지&quot;까지 숨기기</code></pre><h1 id="📌-오늘의-학습-키워드">📌 오늘의 학습 키워드</h1>
<ul>
<li>매개변수의 다형성, 묵시적 업캐스팅</li>
<li><code>enum</code></li>
<li>캡슐화(은닉화)</li>
<li>추상클래스, 인터페이스</li>
<li>강결합 vs 약결합(loose coupling)</li>
<li>싱글톤 패턴</li>
<li>팩토리 패턴, <code>BeanFactory</code>, 제어의 역전(IoC)</li>
</ul>
<hr />
<h1 id="1-oopservice--매개변수의-다형성">1. <code>OopService</code> — 매개변수의 다형성</h1>
<pre><code class="language-java">public class OopService {
    private PersonDTO[] ary;
    private int idx;

    public OopService() {
        ary = new PersonDTO[10];
        idx = 0;
    }

    public void setAry(PersonDTO per) {
        ary[idx++] = per;
    }

    public PersonDTO[] getAry() {
        return ary;
    }
}</code></pre>
<pre><code class="language-java">OopService service = new OopService();
service.setAry(stu);       // StudentDTO
service.setAry(tea);       // TeacherDTO
service.setAry(manager);   // ManagerDTO</code></pre>
<ul>
<li><code>setAry</code>의 매개변수 타입은 <code>PersonDTO</code> 하나뿐인데, 자식 타입(<code>StudentDTO</code>, <code>TeacherDTO</code>, <code>ManagerDTO</code>)을 그대로 넘길 수 있다. 자식을 부모 타입 자리에 넣는 걸 <strong>묵시적 업캐스팅</strong>이라고 하고, 이게 실제로 동작하게 해주는 원리가 <strong>매개변수의 다형성</strong>이다.</li>
<li><code>getAry()</code>가 <code>return ary;</code>로 배열을 그대로 돌려준다는 점도 짚을 만하다 — 이건 배열을 복사하는 게 아니라 <strong>배열의 주소 자체</strong>를 넘기는 것이다. 그래서 <code>getAry()</code>로 받은 배열의 칸을 바깥에서 바꾸면, <code>OopService</code> 내부의 <code>ary</code>도 함께 바뀐다.</li>
</ul>
<hr />
<h1 id="2-enum-flag와-makeperson">2. <code>enum Flag</code>와 <code>makePerson()</code></h1>
<p><code>OopService</code>에는 &quot;flag: 1→Student, 2→Teacher, 3→Manager&quot;라고 설계 의도가 적힌 채로 <code>makePerson()</code>, <code>findPerson()</code> 자리가 마련돼 있었다. 이 flag를 정수 1/2/3이 아니라 <strong>enum</strong>으로 다뤘다.</p>
<pre><code class="language-java">public enum Flag {
    STUDENT(1), TEACHER(2), MANAGER(3);

    private final int flag;
    private Flag(int flag) { this.flag = flag; }
    public int getFlag() { return this.flag; }
}</code></pre>
<ul>
<li><code>enum</code>은 <strong>정해진 상수만 모아둔 특수 클래스</strong>다. <code>Flag.STUDENT</code>, <code>Flag.TEACHER</code>, <code>Flag.MANAGER</code> 세 개만 존재하고, <code>new Flag(1)</code>처럼 마음대로 새로 만들 수 없다.</li>
<li>숫자만 쓰면 <code>1</code>이 학생인지 선생인지 코드만 봐서는 알 수 없고, 실수로 <code>4</code>처럼 범위 밖 숫자가 들어와도 컴파일러가 못 잡아준다. <code>enum</code>은 <strong>존재하는 값 자체가 의미를 담고 있어서</strong>, <code>Flag.STUDENT</code>라고 쓰는 순간 &quot;이건 학생&quot;이라는 게 코드에서 바로 읽힌다.</li>
</ul>
<pre><code class="language-java">public void makePerson(Flag flag, String name, int age, String address, String comm) {
    PersonDTO per = (flag.getFlag() == 1)
        ? StudentDTO.builder().name(name).age(age).address(address).ssn(comm).build()
        : (flag.getFlag() == 2)
            ? TeacherDTO.builder().name(name).age(age).address(address).subject(comm).build()
            : ManagerDTO.builder().name(name).age(age).address(address).dept(comm).build();

    setAry(per);
}</code></pre>
<pre><code class="language-java">service.makePerson(Flag.STUDENT, &quot;손호성&quot;, 29, &quot;서울&quot;, &quot;2026&quot;);</code></pre>
<ul>
<li><code>comm</code>이라는 매개변수 하나로 <code>ssn</code>/<code>subject</code>/<code>dept</code>를 전부 받는다 — 자식마다 마지막 필드 이름은 다르지만 &quot;자식 고유 값 하나&quot;라는 역할은 같다.</li>
<li><code>flag.getFlag() == 1</code>처럼 숫자로 비교한다 — <code>enum</code>은 겉보기엔 <code>Flag.STUDENT</code>라는 이름이지만, 비교할 땐 내부에 저장된 값(<code>getFlag()</code>)을 꺼내 써야 한다. <code>flag.getClass()</code>(클래스 정보)와는 다른 값이라 헷갈리지 않게 구분해야 한다.</li>
<li>마지막 줄에서 방금 만든 <code>per</code>를 1번의 <code>setAry(PersonDTO per)</code>에 그대로 넘긴다 — 매개변수 다형성이 여기서 그대로 재사용된다.</li>
</ul>
<pre><code class="language-java">public PersonDTO findPerson(String name) {
    for (PersonDTO person : ary) {
        if (person == null) {
            break;
        }
        if (person.getName().equals(name)) {
            return person;
        }
    }
    return null;
}</code></pre>
<ul>
<li><code>person == null</code>이면 반복을 멈춘다 — 배열은 크기 10으로 미리 만들어졌지만 실제 데이터는 그보다 적을 수 있어서, 빈 칸을 만나면 더 찾을 필요가 없다.</li>
<li>이름 비교에 <code>==</code>가 아니라 <code>.equals()</code>를 썼다 — 11일차에 정리했던 &quot;문자열 비교는 <code>.equals()</code>로&quot; 원칙 그대로.</li>
</ul>
<hr />
<h1 id="3-캡슐화·추상화--animal과-flyer">3. 캡슐화·추상화 — <code>Animal</code>과 <code>Flyer</code></h1>
<p><strong>캡슐화</strong>는 필드를 <code>private</code>으로 숨기고 getter/setter로만 접근하게 만드는 것 — <code>OopService</code>의 <code>ary</code>도 <code>private</code>이라, 외부에서는 <code>setAry()</code>/<code>getAry()</code>라는 문을 통해서만 접근한다.</p>
<pre><code class="language-java">public abstract class Animal {
    public void eating(String food) {
        System.out.println(food + &quot;를 먹고 살아갑니다.&quot;);
    }
}

public interface Flyer {
    void fly();
    void takeoff();
    void landing();
}

public class SuperMan extends Animal implements Flyer {
    public void fly() { ... }
    public void takeoff() { ... }
    public void landing() { ... }
}</code></pre>
<ul>
<li><code>Animal</code>은 <strong>모든 동물이 공통으로 갖는 기능</strong>(<code>eating</code>)을 담은 추상클래스 — 몸통이 있는 일반 메서드라서 자식이 따로 구현할 필요 없이 그대로 물려받는다.</li>
<li><code>Flyer</code>는 <strong>날 수 있는 존재만</strong> 갖는 능력을 모은 인터페이스 — 모든 동물이 날 수 있는 건 아니므로, &quot;날 수 있음&quot;이라는 성질만 따로 떼어 인터페이스로 만들고 필요한 클래스에만 붙인다.</li>
<li><code>SuperMan extends Animal implements Flyer</code>처럼, 부모(Animal)는 하나만 상속하면서 인터페이스(Flyer)는 필요한 만큼 붙일 수 있다.</li>
</ul>
<pre><code class="language-java">Flyer superman = new SuperMan();
superman.fly();                    // Flyer 타입으로 바로 호출 가능
((SuperMan) superman).eating(&quot;빵&quot;);  // Animal 메서드는 캐스팅 필요</code></pre>
<ul>
<li>변수를 <code>Flyer</code> 타입으로 선언하면, <code>Flyer</code>가 갖고 있는 메서드(<code>fly</code>, <code>takeoff</code>, <code>landing</code>)만 캐스팅 없이 바로 부를 수 있다.</li>
<li><code>Animal</code>이 갖고 있는 <code>eating()</code>을 쓰려면, <code>(SuperMan)</code>으로 캐스팅해서 &quot;이건 사실 SuperMan이야&quot;라고 다시 알려줘야 한다 — 상속관계의 캐스팅이 여기서도 그대로 적용된다.</li>
</ul>
<hr />
<h1 id="4-tv-예제로-보는-강결합-→-약결합-→-싱글톤-→-팩토리">4. TV 예제로 보는 강결합 → 약결합 → 싱글톤 → 팩토리</h1>
<h2 id="4-1-문제-상황--강결합tight-coupling">4-1) 문제 상황 — 강결합(tight coupling)</h2>
<p>브랜드마다 &quot;전원을 켠다&quot;는 동작의 메서드 이름이 다르면(<code>SamsungTV.powerOn()</code>, <code>LgTV.turnOn()</code>), 사용하는 쪽 코드는 브랜드가 바뀔 때마다 <strong>타입과 메서드 이름을 둘 다</strong> 고쳐야 한다. 사용하는 쪽이 구현의 세부사항까지 알아야 하는 이 상태가 <strong>강결합</strong>이다.</p>
<h2 id="4-2-해결-1--인터페이스로-약결합loose-coupling-만들기">4-2) 해결 1 — 인터페이스로 약결합(loose coupling) 만들기</h2>
<pre><code class="language-java">public interface TV {
    void turnOn();
}</code></pre>
<p>삼성 TV든 LG TV든 똑같이 <code>TV</code> 인터페이스를 구현하게 만들면, 사용하는 쪽은 <code>TV tv = ...</code>라는 <strong>규격 하나만</strong> 알면 된다. 브랜드가 바뀌어도 <code>tv.turnOn()</code>이라는 호출 코드는 그대로 — 이게 <strong>약결합</strong>이다.</p>
<h2 id="4-3-해결-2--싱글톤으로-몇-대인지-숨기기">4-3) 해결 2 — 싱글톤으로 &quot;몇 대인지&quot; 숨기기</h2>
<pre><code class="language-java">public class SamsungTV implements TV {
    private static SamsungTV instance;
    private SamsungTV() {}

    public static SamsungTV getInstance() {
        if (instance == null) {
            instance = new SamsungTV();
        }
        return instance;
    }

    public void turnOn() { System.out.println(&quot;삼성 TV 켜짐&quot;); }
}</code></pre>
<ul>
<li>생성자를 <code>private</code>으로 캡슐화해서 <code>new SamsungTV()</code>를 막고, <code>getInstance()</code>라는 통로 하나만 열어둔다.</li>
<li>처음 호출할 때만 실제로 <code>new</code>가 실행되고, 이후로는 항상 같은 객체를 돌려준다.</li>
</ul>
<h2 id="4-4-해결-3--팩토리로-어떤-브랜드인지-숨기기">4-4) 해결 3 — 팩토리로 &quot;어떤 브랜드인지&quot; 숨기기</h2>
<pre><code class="language-java">public class BeanFactory {
    private TV[] ary;

    private BeanFactory() {
        ary = new TV[2];
        ary[0] = SamsungTV.getInstance();
        ary[1] = LgTV.getInstance();
    }

    public TV getBrand(String brandName) {
        return brandName.equalsIgnoreCase(&quot;samsung&quot;) ? ary[0] : ary[1];
    }
}</code></pre>
<pre><code class="language-java">BeanFactory factory = BeanFactory.getInstance();
TV tv = factory.getBrand(&quot;lg&quot;);
tv.turnOn();</code></pre>
<ul>
<li>싱글톤이 &quot;인스턴스가 몇 개인지&quot;를 숨기는 패턴이라면, 팩토리는 한 걸음 더 나아가 <strong>&quot;구체적으로 어떤 클래스의 인스턴스인지&quot;까지 숨긴다.</strong> 사용하는 쪽(<code>TvClientApp</code>)은 <code>factory.getBrand(&quot;lg&quot;)</code>만 부르면 되고, 그 결과가 진짜로는 <code>LgTV</code>라는 사실조차 몰라도 된다.</li>
<li><strong>객체를 생성하는 책임을 사용하는 쪽이 아니라 팩토리가 대신 지는 것</strong>을 제어의 역전(IoC, Inversion of Control)이라고 부르고, 팩토리가 들고 있는 객체를 <strong>빈(Bean)</strong>이라고 부른다 — &quot;BeanFactory → framework&quot;라는 메모가 가리키던 게 바로 이 개념이다. (스프링 프레임워크의 핵심 개념 중 하나이기도 하다.)</li>
</ul>
<p><strong>한 줄로 정리하면</strong>: 강결합(구체 클래스에 직접 의존) → 인터페이스로 약결합 → 싱글톤으로 인스턴스 개수 통제 → 팩토리로 어떤 구현체를 쓸지까지 통제, 라는 4단계 흐름이다.</p>
<hr />
<h1 id="✨-새롭게-알게-된-점">✨ 새롭게 알게 된 점</h1>
<ul>
<li>매개변수를 부모 타입으로 선언하면 자식을 그대로 넘길 수 있다(묵시적 업캐스팅) — 오버로딩보다 확장에 유리하다.</li>
<li><code>enum</code>은 정해진 상수 집합이라, 숫자 플래그보다 의미가 명확하고 잘못된 값이 들어올 위험이 없다.</li>
<li>추상클래스는 &quot;공통 기능을 가진 자식들의 틀&quot;, 인터페이스는 &quot;일부만 가진 능력에 대한 규격&quot;으로 구분해서 쓴다.</li>
<li>강결합 → 약결합(인터페이스) → 싱글톤(개수 통제) → 팩토리(구현체 선택 통제)는 한 흐름으로 이어지는 설계 개선 단계다.</li>
<li>팩토리가 객체 생성을 대신 맡는 것을 제어의 역전(IoC)이라 하고, 그 객체를 빈(Bean)이라 부른다.</li>
</ul>
<hr />
<h1 id="🤔-헷갈렸던-점">🤔 헷갈렸던 점</h1>
<ul>
<li><strong><code>getAry()</code>가 배열 주소를 그대로 넘긴다는 것</strong>: <code>return ary;</code>가 복사가 아니라는 걸 놓치기 쉽다. shallow copy조차 안 하고 원본 참조를 그대로 공유하는 상태.</li>
<li><strong><code>enum</code> 값 비교</strong>: <code>flag.getFlag() == 1</code>처럼 내부 숫자값으로 비교해야 하는데, <code>flag.getClass()</code>(클래스 정보) 같은 것과 헷갈릴 수 있는 지점.</li>
<li><strong>오버라이딩 vs 오버로딩</strong>: 지난번 정리와 동일 — <code>setAry(PersonDTO)</code> 하나로 통일한 것도 오버로딩 대신 다형성을 택한 선택.</li>
<li><strong>강결합 vs 약결합</strong>: TV 예제로 보니, &quot;구체 클래스 이름이 코드에 직접 등장하는가&quot;가 판단 기준이라는 게 명확해졌다.</li>
</ul>
<hr />
<h1 id="💡-오늘의-회고">💡 오늘의 회고</h1>
<p><code>OopService.setAry(PersonDTO per)</code>를 직접 짜보면서 매개변수의 다형성을 확인한 뒤, <code>makePerson()</code>이 왜 필요했는지(플래그에 따라 다른 자식을 만드는 생성 로직을 한 곳에 모으는 것)가 자연스럽게 이어졌다. <code>enum</code>도 처음엔 &quot;숫자를 왜 굳이 이렇게 감싸나&quot; 싶었는데, <code>flag: 1→Student, 2→Teacher, 3→Manager</code>라고 주석으로 적어놨던 걸 실제 타입(<code>Flag.STUDENT</code>)으로 바꿔보니 코드가 훨씬 읽기 좋아진다는 걸 느꼈다.</p>
<p>TV 예제(강결합 → 약결합 → 싱글톤 → 팩토리)는 오늘 배운 개념들을 하나로 꿰어주는 이야기였다. 인터페이스 하나로 브랜드 의존성을 없애고, 싱글톤으로 &quot;몇 개인지&quot;를, 팩토리로 &quot;어떤 건지&quot;를 차례로 숨기는 과정을 보면서, 지금까지 따로따로 배운 캡슐화·추상화·싱글톤이 결국 &quot;사용하는 쪽이 알아야 할 것을 최소화한다&quot;는 하나의 목표로 이어진다는 걸 알게 됐다.</p>