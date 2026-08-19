<blockquote>
<p><strong>한 줄 요약</strong></p>
<p><code>List</code>로 다형성을 확인하고, Generics로 타입 안정성을 확보한 뒤, 함수형 인터페이스(Supplier/Consumer/Function/Predicate)와 람다식을 익혀 명령형 반복문을 Stream API의 선언적 처리(filter-map-collect)로 바꿔보는 하루였다.</p>
</blockquote>
<hr />
<h1 id="📚-오늘-학습한-커리큘럼">📚 오늘 학습한 커리큘럼</h1>
<pre><code>Collection API — List (ArrayList), 다형성으로 여러 DTO 담기
        ↓
Boxing / Unboxing — Wrapper Class
        ↓
Generics — &lt;T&gt;, 타입 파라미터, wildcard(extends/super)
        ↓
명령형 처리 (for + if) → 선언적 처리로 전환 필요성 체감
        ↓
함수형 인터페이스 — Supplier, Consumer, Function, Predicate
        ↓
Stream API — filter, map, collect, forEach</code></pre><h1 id="📌-오늘의-학습-키워드">📌 오늘의 학습 키워드</h1>
<ul>
<li>Collection API (List, Set, Map)</li>
<li>Boxing / Unboxing, Wrapper Class</li>
<li>Generics (<code>&lt;T&gt;</code>), wildcard(<code>? extends</code>, <code>? super</code>)</li>
<li>함수형 인터페이스, 람다식</li>
<li>Stream API (filter, map, collect, forEach)</li>
</ul>
<hr />
<h1 id="0-개념-정리-키워드-사전">0. 개념 정리 (키워드 사전)</h1>
<table>
<thead>
<tr>
<th>키워드</th>
<th>한 줄 정의</th>
<th>비고</th>
</tr>
</thead>
<tbody><tr>
<td><strong>Collection API</strong></td>
<td><code>java.util</code> 패키지의 자료구조 모음. <code>List</code>/<code>Set</code>/<code>Map</code>으로 구성</td>
<td>배열과 달리 크기가 가변적</td>
</tr>
<tr>
<td><strong>List</strong></td>
<td>가변길이, <strong>중복 허용</strong>, <strong>순서 존재</strong>, 요소는 객체 타입만 담김</td>
<td><code>ArrayList</code>, <code>Vector</code> 등이 구현체</td>
</tr>
<tr>
<td><strong>Set</strong></td>
<td>가변길이, <strong>중복 비허용</strong>, <strong>순서 없음</strong>, 요소는 객체 타입만 담김</td>
<td></td>
</tr>
<tr>
<td><strong>Map</strong></td>
<td><code>{key : value}</code> 쌍으로 데이터를 저장</td>
<td></td>
</tr>
<tr>
<td><strong>Boxing / Unboxing</strong></td>
<td>기본 타입(primitive) ↔ 참조 타입(reference) 자동 변환</td>
<td><code>int → Integer</code>(Boxing), <code>Integer → int</code>(Unboxing)</td>
</tr>
<tr>
<td><strong>Wrapper Class</strong></td>
<td>기본 타입을 객체(참조 타입)로 감싼 클래스</td>
<td><code>int</code>→<code>Integer</code>, <code>char</code>→<code>Character</code></td>
</tr>
<tr>
<td><strong>Generics (<code>&lt;T&gt;</code>)</strong></td>
<td>클래스/메서드가 다루는 데이터 타입을 <strong>사용하는 시점에 지정</strong>하는 문법</td>
<td><code>T</code>(type), <code>E</code>(element), <code>K</code>(key), <code>V</code>(value), <code>N</code>(number) 관례</td>
</tr>
<tr>
<td><strong>Wildcard <code>&lt;? extends T&gt;</code></strong></td>
<td><strong>읽기 전용</strong>. <code>T</code>의 하위 타입을 담을 수 있음을 표현</td>
<td><code>add()</code> 불가 (안전하지 않아서 컴파일러가 막음)</td>
</tr>
<tr>
<td><strong>Wildcard <code>&lt;? super T&gt;</code></strong></td>
<td><strong>쓰기 전용</strong>. <code>T</code>의 상위 타입을 담을 수 있음을 표현</td>
<td><code>add()</code> 가능</td>
</tr>
<tr>
<td><strong>함수형 인터페이스</strong></td>
<td>추상 메서드가 <strong>딱 1개</strong>뿐인 인터페이스</td>
<td><code>@FunctionalInterface</code>로 명시, 람다식 대입 가능</td>
</tr>
<tr>
<td><strong>람다식 (Lambda)</strong></td>
<td>함수형 인터페이스의 구현체를 짧게 표현하는 문법 <code>(매개변수) -&gt; 실행내용</code></td>
<td>익명 클래스 구현을 대체</td>
</tr>
<tr>
<td><strong>Stream</strong></td>
<td>Collection 데이터를 가공하기 위한 <strong>일회성</strong> 파이프라인</td>
<td>중간연산(0~n개) + 최종연산(1개)으로 구성, 원본 데이터는 손상되지 않음</td>
</tr>
<tr>
<td><strong>명령형 처리</strong></td>
<td><code>for</code>+<code>if</code>처럼 &quot;어떻게(how) 할지&quot;를 직접 코드로 나열하는 방식</td>
<td></td>
</tr>
<tr>
<td><strong>선언적 처리</strong></td>
<td>Stream처럼 &quot;무엇을(what) 원하는지&quot;만 표현하는 방식</td>
<td>가독성, 병렬처리, 유지보수에 유리</td>
</tr>
</tbody></table>
<hr />
<h1 id="1-collection-api--list와-다형성">1. Collection API — <code>List</code>와 다형성</h1>
<pre><code class="language-java">import java.util.List;
import java.util.ArrayList;

List&lt;String&gt; list = new ArrayList&lt;String&gt;();
list.add(&quot;String&quot;);

System.out.println(&quot;debug &gt;&gt;&gt;&gt; list &quot; + list.size());
System.out.println(&quot;debug &gt;&gt;&gt;&gt; list &quot; + list);

for (int idx = 0; idx &lt; list.size(); idx++) {
    String data = list.get(idx);
    System.out.println(data);
}</code></pre>
<ul>
<li><code>List</code>, <code>Set</code>, <code>Map</code>의 차이를 표로 정리하고, 배열(<code>int[]</code>)과 달리 Collection은 <strong>가변 길이</strong>라는 점을 확인했다.</li>
<li>Boxing 덕분에 <code>list.add(10)</code>처럼 기본 타입도 <code>List</code>에 담을 수 있다는 걸 실습했는데, Generics를 <code>List&lt;String&gt;</code>으로 명시한 뒤에는 <code>list.add(10)</code>이 컴파일 에러가 난다는 것도 함께 확인했다 — Generics가 컴파일 시점에 타입 안정성을 보장해주는 부분이다.</li>
</ul>
<h2 id="dto를-담을-때의-다형성">DTO를 담을 때의 다형성</h2>
<pre><code class="language-java">List&lt;PersonDTO&gt; personList = new ArrayList&lt;PersonDTO&gt;();

StudentDTO student = StudentDTO.builder().name(&quot;inspire&quot;).build();
TeacherDTO teacher = TeacherDTO.builder().name(&quot;sonhs&quot;).build();
ManagerDTO manager = ManagerDTO.builder().name(&quot;lgcns&quot;).build();

personList.add(student);
personList.add(teacher);
personList.add(manager);

for (int idx = 0; idx &lt; personList.size(); idx++) {
    PersonDTO person = personList.get(idx);
    System.out.println(person.personInfo());
}</code></pre>
<ul>
<li><code>List&lt;PersonDTO&gt;</code>인데 <code>StudentDTO</code>, <code>TeacherDTO</code>, <code>ManagerDTO</code>가 그대로 담기는 이유는 <strong>다형성</strong>이다. 세 DTO 모두 <code>PersonDTO</code>를 상속받고 있어서 &quot;PersonDTO의 한 종류&quot;로 취급되어 자동으로 업캐스팅된다. (기존에 <code>setAry(PersonDTO per)</code> 메서드에서 봤던 매개변수의 다형성과 동일한 원리가 <code>List</code>의 제네릭 타입에도 그대로 적용된다.)</li>
</ul>
<hr />
<h1 id="2-boxingunboxing-wrapper-class">2. Boxing/Unboxing, Wrapper Class</h1>
<pre><code class="language-java">/*
Boxing, UnBoxing
Wrapper Class ( primitive type -&gt; reference type, reference type &lt;- primitive type)
- int -&gt; Integer
- char -&gt; Character
*/</code></pre>
<ul>
<li>기본 타입은 <code>Collection</code>에 직접 담을 수 없는데, 자바가 자동으로 Wrapper Class로 변환(Boxing)해줘서 <code>list.add(10)</code>처럼 쓸 수 있었던 것이다.</li>
<li>Generics로 타입을 <code>&lt;String&gt;</code>처럼 명시하고 나면, 이 자동 변환도 타입이 안 맞으면 막힌다 (<code>list.add(10)</code> → 컴파일 에러).</li>
</ul>
<hr />
<h1 id="3-generics--t와-wildcard">3. Generics — <code>&lt;T&gt;</code>와 Wildcard</h1>
<pre><code class="language-java">package features.generics;

public class ResponseTemplate&lt;T&gt; {
    private int code;
    private String message;
    private T data;

    public ResponseTemplate() { }

    public ResponseTemplate(int code, String message, T data) {
        this.code = code;
        this.message = message;
        this.data = data;
    }

    public int getCode() { return code; }
    public void setCode(int code) { this.code = code; }
    public String getMessage() { return message; }
    public void setMessage(String message) { this.message = message; }
    public T getData() { return data; }
    public void setData(T data) { this.data = data; }
}</code></pre>
<pre><code class="language-java">ResponseTemplate&lt;List&lt;PersonDTO&gt;&gt; response =
    new ResponseTemplate&lt;List&lt;PersonDTO&gt;&gt;(200, &quot;OK&quot;, personList);

List&lt;PersonDTO&gt; lst = response.getData();
for (int idx = 0; idx &lt; lst.size(); idx++) {
    PersonDTO person = lst.get(idx);
    System.out.println(person.personInfo());
}</code></pre>
<ul>
<li><code>ResponseTemplate&lt;T&gt;</code>처럼 클래스 자체를 제네릭으로 만들면, <code>code</code>/<code>message</code>는 고정 타입으로 두고 <code>data</code>만 <strong>사용하는 쪽에서 원하는 타입</strong>(<code>Integer</code>, <code>String</code>, <code>List&lt;PersonDTO&gt;</code> 등)으로 유연하게 바꿔 쓸 수 있다.</li>
<li><code>response.getData()</code>의 결과를 <code>List&lt;PersonDTO&gt;</code>로 받으면 타입이 그대로 유지되지만, 타입을 지정하지 않고 그냥 <code>Object</code>로 받으면 다시 캐스팅이 필요해진다 — 제네릭이 캐스팅을 줄여주는 이유가 여기 있다.</li>
</ul>
<h2 id="wildcard-extends-vs-super">Wildcard: <code>extends</code> vs <code>super</code></h2>
<pre><code class="language-java">// extends : 읽기전용(T 하위타입)
// super   : 쓰기전용(T 상위타입)

// List&lt;? extends PersonDTO&gt; personList = new ArrayList&lt;PersonDTO&gt;();
// extends -&gt; 읽기전용이기에 add() 불가

List&lt;? super PersonDTO&gt; personList = new ArrayList&lt;PersonDTO&gt;();
// super -&gt; 쓰기전용이므로 add() 가능</code></pre>
<ul>
<li><code>&lt;? extends T&gt;</code>는 &quot;T 또는 T의 자식 타입&quot;이 들어있다는 것만 보장하므로, <strong>꺼내서 읽는 건 안전</strong>하지만 <strong>뭘 넣을지는 컴파일러가 확신할 수 없어서 <code>add()</code>가 막힌다.</strong></li>
<li><code>&lt;? super T&gt;</code>는 &quot;T 또는 T의 부모 타입&quot;이 들어있다는 뜻이라, <strong>T 타입은 안전하게 넣을 수 있지만</strong>, 꺼낼 때는 정확히 뭐가 나올지 몰라서 <code>Object</code>로만 받을 수 있다.</li>
<li>이 둘을 합쳐서 흔히 <strong>PECS 원칙</strong>(&quot;Producer-Extends, Consumer-Super&quot;: 데이터를 꺼내 쓸 거면 extends, 데이터를 넣을 거면 super)이라고 부른다.</li>
</ul>
<hr />
<h1 id="4-명령형-처리-→-선언적-처리stream로-전환">4. 명령형 처리 → 선언적 처리(Stream)로 전환</h1>
<h2 id="명령형-처리-for--if">명령형 처리 (for + if)</h2>
<pre><code class="language-java">List&lt;String&gt; filteringList = new ArrayList&lt;String&gt;();
for (int idx = 0; idx &lt; personList.size(); idx++) {
    PersonDTO person = personList.get(idx);
    // s로 시작하는 이름 필터링
    if (person.getName().startsWith(&quot;s&quot;)) {
        // 필터링된 데이터를 대문자 변환 후 리스트에 담아 반환
        filteringList.add(person.getName().toUpperCase());
    }
}</code></pre>
<h2 id="선언적-처리-stream-api">선언적 처리 (Stream API)</h2>
<pre><code class="language-java">List&lt;String&gt; filteringList = personList.stream()
                                .filter(s -&gt; s.getName().startsWith(&quot;s&quot;))
                                .map(s -&gt; s.getName().toUpperCase())
                                .collect(Collectors.toList());
System.out.println(filteringList);</code></pre>
<ul>
<li>같은 결과를 만드는 코드인데, <code>for</code>+<code>if</code>로 &quot;어떻게&quot; 순회하고 조건을 검사할지 일일이 나열하던 걸(명령형), <code>filter → map → collect</code>로 &quot;무엇을&quot; 원하는지만 표현하는 방식(선언적)으로 바꿨다.</li>
<li><strong><code>.collect(Collectors.toList())</code>의 의미</strong>: Stream은 중간연산(<code>filter</code>, <code>map</code>)을 거치는 동안은 아직 <code>List</code>나 <code>Set</code> 같은 실제 자료구조가 아니라 &quot;가공 중인 흐름&quot; 상태다. <code>collect(...)</code>는 이 흐름을 최종적으로 원하는 자료구조(여기선 <code>List</code>)로 <strong>모아서(수집해서) 반환</strong>하는 최종연산이다. <code>Collectors.toList()</code>는 &quot;리스트 형태로 모아달라&quot;는 수집 방식을 지정하는 것이다.</li>
</ul>
<h2 id="foreach로-마무리"><code>forEach</code>로 마무리</h2>
<pre><code class="language-java">personList.stream()
    .filter(person -&gt; person.getName().length() &gt; 5)
    .forEach(person -&gt; System.out.println(person.personInfo()));</code></pre>
<ul>
<li><code>map</code> 없이 <code>filter</code>로 걸러낸 결과를 바로 각 요소마다 처리(<code>forEach</code>)하는 방식도 확인했다. <code>forEach</code>는 리턴값이 없는 최종연산으로, <code>Consumer</code> 함수형 인터페이스를 인자로 받는다.</li>
</ul>
<hr />
<h1 id="5-함수형-인터페이스와-람다식">5. 함수형 인터페이스와 람다식</h1>
<pre><code class="language-java">/*
함수형 인터페이스
- 인터페이스가 가질 수 있는 메서드가 딱 하나인 것을 의미
- 람다식을 활용하기 위해서

Supplier : 매개변수 x, 반환타입 o
Consumer : 매개변수 o, 반환타입 x
Function : 매개변수 o, 반환타입 o
Predicate : 매개변수 o, 반환타입 Boolean
*/</code></pre>
<h2 id="직접-만든-함수형-인터페이스">직접 만든 함수형 인터페이스</h2>
<pre><code class="language-java">package features.lambda;

@FunctionalInterface
public interface InspireFunction {
    public int max(int x, int y);
}</code></pre>
<pre><code class="language-java">InspireFunction fun01 = (x, y) -&gt; x &gt; y ? x : y;
System.out.println(fun01.max(100, 200));   // 200

InspireFunction fun02 = (x, y) -&gt; x + y;
System.out.println(fun02.max(100, 200));   // 300</code></pre>
<ul>
<li>같은 인터페이스(<code>InspireFunction</code>)에 서로 다른 람다식을 대입하면, 메서드 이름(<code>max</code>)은 똑같아도 <strong>실제 실행되는 로직은 대입한 람다식에 따라 달라진다.</strong> 함수 자체를 값처럼 변수에 담아 전달할 수 있다는 걸 체감했다.</li>
</ul>
<h2 id="자바-표준-함수형-인터페이스-4종">자바 표준 함수형 인터페이스 4종</h2>
<pre><code class="language-java">// Supplier: 매개변수 없이 값을 &quot;공급&quot;
Supplier&lt;String&gt; supplier = () -&gt; &quot;inspire&quot;;
System.out.println(supplier.get());   // inspire

// Consumer: 값을 받아서 &quot;소비&quot;(처리)만 하고 리턴 없음
Consumer&lt;String&gt; consumer = (str) -&gt; System.out.println(str.split(&quot; &quot;)[1]);
consumer.andThen(System.out::println)
        .accept(&quot;lgcns inspire&quot;);
// 실행순서: consumer 원래 로직 먼저 → andThen으로 이어붙인 동작 나중

// Function: 값을 받아서 다른 값으로 &quot;변환&quot;
Function&lt;String, Integer&gt; function = (str) -&gt; {
    return str.length();
};
int len = function.apply(&quot;lgcns inspire 6th camp(feat.jslim)&quot;);
System.out.println(len);

// Predicate: 값을 받아서 true/false 판단
Predicate&lt;String&gt; predicate = (str) -&gt; str.equals(&quot;lgcns&quot;);
Boolean isFlag = predicate.test(&quot;lgcns&quot;);
System.out.println(isFlag);</code></pre>
<ul>
<li><code>Consumer.andThen(...)</code>은 &quot;원래 하려던 일 + 추가로 할 일&quot;을 이어붙이는 메서드다. <code>consumer</code> 혼자서는 <code>str.split(&quot; &quot;)[1]</code>만 출력하지만, <code>andThen(System.out::println)</code>을 붙이면 그 뒤에 원본 문자열 전체를 한 번 더 출력한다. 단, <code>accept(...)</code>를 호출해야 실제로 실행된다는 걸 직접 겪으며 확인했다 — <code>accept()</code> 없이 정의만 해두면 아무 것도 출력되지 않는다.</li>
<li><code>System.out::println</code>처럼 <code>클래스::메서드</code> 형태로 쓰는 것은 <strong>메서드 참조(method reference)</strong>다. <code>(x) -&gt; System.out.println(x)</code>를 더 짧게 쓴 것과 같다.</li>
</ul>
<hr />
<h1 id="6-stream-api-정리">6. Stream API 정리</h1>
<pre><code class="language-java">/*
Stream API
- 코드의 가독성, 병렬처리, 유지보수 향상
- 작업을 내부적으로 처리(람다식 이용)
- 원본데이터(Collection) 손상을 가하지 않음
- 일회성
*/</code></pre>
<pre><code class="language-java">List&lt;String&gt; brands = Arrays.asList(&quot;samsung&quot;, &quot;lg&quot;, &quot;lgcns&quot;, &quot;inspire&quot;, &quot;camp&quot;);

brands.forEach((brand) -&gt; System.out.println(brand));

Stream&lt;String&gt; stream = brands.stream();
stream.forEach((brand) -&gt; System.out.println(brand));

// stream은 일회성이라 한 번 최종연산을 거치면 다시 못 씀 → 다시 stream()으로 연결해야 함
stream = brands.stream();
stream.forEach(System.out::println);</code></pre>
<ul>
<li>Stream이 <strong>일회성(한 번 최종연산을 거치면 재사용 불가)</strong>이라는 점을 직접 에러를 겪으며 확인했다. 같은 Stream을 다시 쓰려면 <code>brands.stream()</code>을 다시 호출해서 새로 만들어야 한다.</li>
<li>Stream이 내부적으로 thread를 활용해 처리하기 때문에 병렬처리에 유리하다는 점도 짚었다.</li>
</ul>
<hr />
<h1 id="✨-새롭게-알게-된-점">✨ 새롭게 알게 된 점</h1>
<ul>
<li><code>List&lt;PersonDTO&gt;</code>에 자식 DTO(<code>StudentDTO</code>, <code>TeacherDTO</code>, <code>ManagerDTO</code>)가 그대로 담기는 것도 결국 다형성(업캐스팅) 원리라는 걸 Collection에서도 재확인했다.</li>
<li>Generics의 wildcard(<code>? extends</code> / <code>? super</code>)는 &quot;읽기 전용이냐 쓰기 전용이냐&quot;로 구분하면 헷갈리지 않는다 (PECS 원칙).</li>
<li><code>.collect(Collectors.toList())</code>는 Stream의 중간 가공 결과를 실제 <code>List</code>로 &quot;수집&quot;해서 반환하는 최종연산이다.</li>
<li>함수형 인터페이스(<code>Supplier</code>/<code>Consumer</code>/<code>Function</code>/<code>Predicate</code>)는 매개변수·반환타입 유무로 역할이 딱 나뉜다.</li>
<li>Stream은 원본 Collection을 건드리지 않고, 한 번 쓰면 다시 쓸 수 없는 일회성 파이프라인이다.</li>
</ul>
<hr />
<h1 id="🤔-헷갈렸던-점">🤔 헷갈렸던 점</h1>
<ul>
<li><strong><code>Consumer.accept()</code>를 빼먹으면 아무 일도 안 일어남</strong>: 람다식을 변수에 대입만 해두고 <code>accept(...)</code>를 호출하지 않으면 실행 자체가 안 된다는 걸 직접 겪었다 — 정의(람다식)와 실행(<code>accept</code>)은 별개의 단계.</li>
<li><strong><code>@ToString(callSuper = true)</code></strong>: Lombok의 <code>@ToString</code>은 기본적으로 부모 클래스(<code>PersonDTO</code>)의 필드는 출력하지 않는데, <code>callSuper = true</code>를 붙이면 부모의 필드까지 포함해서 문자열로 만들어준다. 상속 구조에서 <code>toString()</code>을 쓸 때 이 옵션이 없으면 부모 필드(<code>name</code>, <code>age</code> 등)가 누락될 수 있다는 걸 확인했다.</li>
<li><strong>wildcard <code>extends</code>에서 <code>add()</code>가 막히는 이유</strong>: <code>List&lt;? extends PersonDTO&gt;</code>는 실제로 <code>StudentDTO</code>의 리스트일 수도, <code>TeacherDTO</code>의 리스트일 수도 있어서, 컴파일러 입장에서는 &quot;정확히 어떤 자식 타입을 넣어야 안전한지&quot; 확신할 수 없어 아예 <code>add()</code> 자체를 막아버린다.</li>
</ul>
<hr />
<h1 id="💡-오늘의-회고">💡 오늘의 회고</h1>
<p><code>for</code>+<code>if</code>로 짰던 필터링 코드를 Stream의 <code>filter().map().collect()</code>로 바꿔보니, 줄 수는 비슷한데 &quot;무엇을 원하는지&quot;가 코드만 봐도 훨씬 잘 읽힌다는 걸 체감했다. 특히 <code>Consumer.accept()</code>를 빼먹어서 아무 것도 출력이 안 됐던 경험이, &quot;람다식은 정의일 뿐이고 실행은 별도로 호출해야 한다&quot;는 걸 몸으로 이해하게 해준 계기가 됐다. Generics의 wildcard는 아직 완전히 손에 익지는 않았지만, &quot;읽기냐 쓰기냐&quot;로 구분하는 기준을 잡은 것만으로도 다음에 다시 봤을 때 헷갈릴 확률이 줄어들 것 같다.</p>