<h2 id="1-오늘의-맥락--프론트엔드에서-자바백엔드로-넘어가는-첫날">1. 오늘의 맥락 — 프론트엔드에서 자바(백엔드)로 넘어가는 첫날</h2>
<p>지금까지 9일 동안은 화면(React)만 다뤘다면, 오늘부터는 그 화면 뒤에서 데이터를 처리할 <strong>자바</strong>를 시작했다. 첫날이라 실습보다는 개념 위주였다. 개발 환경(JDK/JRE)을 세팅하는 것부터, 자바에서 &quot;변수&quot;와 &quot;타입&quot;이 자바스크립트와 어떻게 다른지, 그리고 자바의 핵심 단위인 <strong>class</strong>가 뭔지까지 쭉 훑었다.</p>
<p>자바스크립트는 <code>let</code>, <code>const</code> 하나로 뭐든 담을 수 있었는데, 자바는 변수 앞에 <strong>타입을 반드시 명시</strong>해야 한다는 것부터가 오늘의 첫 번째 체감 포인트였다.</p>
<hr />
<h2 id="2-개발-환경--jdk-jre-그리고-빌드-도구">2. 개발 환경 — JDK, JRE, 그리고 빌드 도구</h2>
<p><strong>JVM, JRE, JDK는 서로 포함하는 관계</strong>다. 그림으로 보면 이렇다.</p>
<pre><code>JDK (개발 키트)
 └─ JRE (실행 환경)
     └─ JVM (실행 엔진)</code></pre><p><strong>하나씩 풀어보면</strong></p>
<ul>
<li><strong>JVM (Java Virtual Machine)</strong>: 자바 코드를 실제로 &quot;실행&quot;시키는 엔진. 자바로 짠 코드는 컴퓨터가 바로 못 알아듣고, 일단 &quot;바이트코드&quot;라는 중간 형태로 바뀌는데, 그 바이트코드를 한 줄씩 읽어서 실제로 동작시키는 게 JVM이다.</li>
<li><strong>JRE (Java Runtime Environment)</strong>: JVM + 자바 프로그램이 동작하는 데 필요한 기본 라이브러리 묶음. &quot;실행&quot;만 하고 싶다면 이것만 있으면 된다. 다만 <strong>컴파일러(코드를 바이트코드로 바꿔주는 도구)는 없다.</strong></li>
<li><strong>JDK (Java Development Kit)</strong>: JRE + 컴파일러(<code>javac</code>) + 여러 개발 도구. <strong>&quot;개발&quot;까지 하려면 이게 필요하다.</strong></li>
</ul>
<p>→ 그래서 우리는 오늘 JDK를 설치했다. 코드를 &quot;작성하고 실행&quot;까지 다 해야 하니까, 컴파일러가 없는 JRE만으로는 부족하기 때문이다.</p>
<p><strong>maven / gradle이란?</strong>
자바 프로젝트에서 필요한 외부 라이브러리를 자동으로 받아오고, 소스코드를 컴파일해서 실행 가능한 형태로 만들어주는 <strong>빌드 도구</strong>다. React를 배울 때 썼던 <code>npm</code>이 <code>package.json</code>을 보고 라이브러리를 설치해줬던 것처럼, 자바에서는 maven이나 gradle이 그 역할을 한다고 이해하니 낯설지 않았다.</p>
<hr />
<h2 id="3-변수란--데이터를-담는-그릇">3. 변수란? — &quot;데이터를 담는 그릇&quot;</h2>
<p>강사님이 변수를 <strong>&quot;데이터를 담는 그릇&quot;</strong>이라고 표현해주셨는데, 자바는 이 그릇을 두 가지 종류로 명확히 나눠서 쓴다.</p>
<h3 id="3-1-기본타입primitive-type--그릇-안에-값-자체가-들어있다">3-1) 기본타입(primitive type) — 그릇 안에 값 자체가 들어있다</h3>
<p>숫자, 문자, 참/거짓처럼 아주 단순한 값을 담을 때 쓰는 타입이다. 종류가 정해져 있어서 <strong>암기가 필요</strong>하다고 강조하셨다.</p>
<table>
<thead>
<tr>
<th>분류</th>
<th>타입 이름</th>
<th>담는 값</th>
</tr>
</thead>
<tbody><tr>
<td>정수형</td>
<td><code>byte</code>, <code>short</code>, <code>int</code>, <code>long</code></td>
<td>정수 (크기 순서: byte &lt; short &lt; int &lt; long)</td>
</tr>
<tr>
<td>실수형</td>
<td><code>float</code>, <code>double</code></td>
<td>소수점이 있는 숫자</td>
</tr>
<tr>
<td>문자형</td>
<td><code>char</code></td>
<td>글자 하나 (<code>'A'</code>처럼 작은따옴표 사용)</td>
</tr>
<tr>
<td>논리형</td>
<td><code>boolean</code></td>
<td><code>true</code> 또는 <code>false</code></td>
</tr>
</tbody></table>
<p>여기에 더해 <code>String</code>(문자열)도 자주 쓰이는데, 엄밀히는 아래에서 설명할 <strong>참조타입</strong>에 속한다. (문자 하나만 담는 <code>char</code>와 헷갈리지 않게 구분해서 기억해두면 좋다.)</p>
<h3 id="3-2-참조타입reference-type--그릇-안에-주소가-들어있다">3-2) 참조타입(reference type) — 그릇 안에 &quot;주소&quot;가 들어있다</h3>
<p>기본타입 8가지(byte, short, int, long, float, double, char, boolean)를 뺀 <strong>나머지 전부</strong>가 참조타입이다. class로 직접 만든 타입들이 여기 해당한다.</p>
<p>여기서 오늘 가장 크게 걸렸던 부분이 나온다. <strong>참조타입 변수는 값 자체를 담는 게 아니라, 그 값이 실제로 저장된 &quot;메모리 주소&quot;를 담는다.</strong> 그래서 참조타입을 쓰려면 먼저 그 실체를 메모리 어딘가에 만들어야 하는데, 그게 바로 <code>new</code> 연산자다.</p>
<ul>
<li>자바스크립트: <code>const car = { name: &quot;소나타&quot; };</code> — 그냥 바로 만들어서 씀</li>
<li>자바: <code>Car car = new Car();</code> — &quot;Car라는 타입의 실체를 하나 새로(new) 만들고, 그 주소를 car라는 변수에 담아라&quot;</li>
</ul>
<p>이 차이를 이해하고 나니, 자바 코드에서 왜 이렇게 <code>new</code>가 자주 등장하는지 납득이 됐다.</p>
<h3 id="3-3-변수-선언-문법">3-3) 변수 선언 문법</h3>
<pre><code>접근지정자 변수타입 변수명 = literal value;

예) public int age = 10;</code></pre><p>자바스크립트의 <code>let age = 10;</code>과 비교하면, 자바는 <strong>접근지정자(public/private 등)</strong>와 <strong>타입(int)</strong>이라는 두 가지 정보가 앞에 더 붙는다. &quot;이 변수를 누가 쓸 수 있는지&quot;와 &quot;이 변수에 어떤 종류의 값만 담을 수 있는지&quot;를 미리 못 박아두는 셈이다.</p>
<h3 id="3-4-변수의-scope범위--어디에-썼느냐가-이름을-정한다">3-4) 변수의 Scope(범위) — 어디에 썼느냐가 이름을 정한다</h3>
<table>
<thead>
<tr>
<th>선언 위치</th>
<th>이름</th>
<th>설명</th>
</tr>
</thead>
<tbody><tr>
<td>클래스 블록 바로 안 (메서드 밖)</td>
<td><strong>멤버변수</strong></td>
<td>그 클래스로 만든 인스턴스라면 어디서든 접근 가능</td>
</tr>
<tr>
<td>메서드 블록 안</td>
<td><strong>지역변수</strong></td>
<td>그 메서드가 실행되는 동안에만 존재, 메서드가 끝나면 사라짐</td>
</tr>
</tbody></table>
<p>자바스크립트의 전역/지역 변수 개념과 방향은 비슷하지만, 자바는 &quot;클래스 블록이냐 메서드 블록이냐&quot;라는 아주 명확한 기준으로 나눈다는 점이 오늘 새로 알게 된 부분이다.</p>
<hr />
<h2 id="4-class란-무엇인가--붕어빵-틀에-비유해서-이해하기">4. class란 무엇인가 — 붕어빵 틀에 비유해서 이해하기</h2>
<p>오늘 배운 것 중 가장 핵심이었던 개념이라, 처음부터 차근차근 풀어본다.</p>
<h3 id="4-1-class--설계도틀-instance--그-틀로-찍어낸-실제-결과물">4-1) class = 설계도(틀), instance = 그 틀로 찍어낸 실제 결과물</h3>
<p>강사님 설명을 내 방식대로 정리하면: <strong>class는 붕어빵 틀이고, instance는 그 틀로 찍어낸 붕어빵 하나하나</strong>다.</p>
<ul>
<li>붕어빵 틀(class)은 &quot;어떤 모양으로 만들지&quot;에 대한 설계도일 뿐, 그 자체는 먹을 수 있는 붕어빵이 아니다.</li>
<li>그 틀에 반죽을 붓고(=<code>new</code>) 찍어내야 비로소 진짜 붕어빵(=instance, 인스턴스)이 나온다.</li>
<li>같은 틀로 여러 개의 붕어빵을 찍어낼 수 있듯이, 하나의 class로 <code>new</code>를 여러 번 호출해서 여러 개의 인스턴스를 만들 수 있다.</li>
</ul>
<p>React를 배울 때 컴포넌트 하나를 정의해두고 <code>&lt;Book bookName=&quot;java&quot;/&gt;</code>, <code>&lt;Book bookName=&quot;kor&quot;/&gt;</code>처럼 여러 번 재사용해서 화면에 여러 개를 찍어냈던 것과 원리가 닮아있다고 느꼈다. 컴포넌트가 &quot;틀&quot;, 화면에 실제로 그려진 각각의 결과가 &quot;인스턴스&quot;에 가깝다고 비유하니 이해가 쉬웠다.</p>
<h3 id="4-2-class-안에는-무엇이-들어가는가">4-2) class 안에는 무엇이 들어가는가</h3>
<p>class 안에는 세 가지가 들어간다: <strong>변수, 메서드, 생성자</strong>.</p>
<p>여기서 정말 중요한 포인트를 하나 짚어주셨는데, <strong>class 안에 적힌 변수와 메서드는 &quot;class 자신의 것&quot;이 아니라 &quot;그 class로 만들어진 각 인스턴스 각자의 것&quot;</strong>이라는 점이다.</p>
<p>비유하자면, 붕어빵 틀 자체에는 팥이 안 들어있다. 팥은 그 틀로 찍어낸 붕어빵 하나하나 안에 들어있는 것이다. 마찬가지로, class 자체에는 실제 값이 들어있지 않고, <code>new</code>로 만들어진 인스턴스 하나하나가 각자 자기만의 변수 값을 갖는다. 그래서 <code>new Car()</code>로 자동차를 두 대 만들면, 두 대는 같은 class(설계도)에서 나왔지만 서로 다른 색깔, 다른 이름을 각자 가질 수 있는 것이다.</p>
<h3 id="4-3-최소-형태의-class-예시">4-3) 최소 형태의 class 예시</h3>
<pre><code class="language-java">public class App {
    public static void main(String[] args) throws Exception {
        System.out.println(&quot;Hello, World!&quot;);
    }
}</code></pre>
<p><strong>하나씩 뜯어보면</strong></p>
<ul>
<li><code>public class App { ... }</code> — <code>App</code>이라는 이름의 class를 하나 정의한다는 뜻이다. <code>{ }</code> 안에 이 class의 내용(변수, 메서드, 생성자)이 들어간다.</li>
<li><code>public static void main(String[] args)</code> — 이건 자바 프로그램이 시작할 때 <strong>가장 먼저 실행되는 특별한 메서드</strong>다. &quot;여기서부터 프로그램을 시작하세요&quot;라고 자바에게 알려주는 진입점이라고 이해했다. 이름(<code>main</code>), 형태(<code>public static void</code>, 매개변수 <code>String[] args</code>)가 거의 고정된 약속이라, 오늘은 &quot;이런 모양이구나&quot; 정도로 눈에 익혀두는 걸 목표로 삼았다.</li>
<li><code>System.out.println(&quot;Hello, World!&quot;);</code> — 괄호 안의 내용을 화면(콘솔)에 출력하는 코드. <code>console.log()</code>와 하는 일이 똑같아서 바로 이해가 됐다.</li>
</ul>
<hr />
<h2 id="5-생성자constructor--왜-일반-메서드와-다른가">5. 생성자(Constructor) — 왜 일반 메서드와 다른가</h2>
<h3 id="5-1-생성자가-하는-일">5-1) 생성자가 하는 일</h3>
<p>앞서 &quot;class로 인스턴스를 만들 때 <code>new</code>를 쓴다&quot;고 했는데, 이때 <code>new</code> 뒤에 붙는 <code>클래스이름()</code> 부분이 바로 <strong>생성자를 호출하는 것</strong>이다.</p>
<pre><code class="language-java">new Car();</code></pre>
<p>이 한 줄은 &quot;Car라는 class의 <strong>생성자</strong>를 호출해서, 그 결과로 새 인스턴스 하나를 메모리에 만들어라&quot;는 뜻이다. 생성자는 <strong>인스턴스가 태어나는 순간에 자동으로 실행되는, 초기 세팅을 담당하는 특별한 메서드</strong>라고 이해하면 된다.</p>
<h3 id="5-2-생성자가-일반-메서드와-다른-세-가지-규칙">5-2) 생성자가 일반 메서드와 다른 세 가지 규칙</h3>
<table>
<thead>
<tr>
<th>규칙</th>
<th>설명</th>
</tr>
</thead>
<tbody><tr>
<td>반환타입이 없다</td>
<td>일반 메서드는 <code>void</code>나 <code>int</code> 같은 반환타입을 꼭 적어야 하는데, 생성자는 아예 반환타입 자리 자체가 없다</td>
</tr>
<tr>
<td>이름이 클래스 이름과 완전히 같다</td>
<td>class가 <code>Car</code>면 생성자 이름도 반드시 <code>Car</code>여야 한다</td>
</tr>
<tr>
<td><code>new</code> 뒤에서만 호출 가능</td>
<td><code>car.Car()</code>처럼 일반 메서드 부르듯 직접 호출할 수 없고, 오직 <code>new Car()</code> 형태로만 실행된다</td>
</tr>
</tbody></table>
<p>이 세 가지 규칙을 처음엔 그냥 암기해야 하는 규칙으로만 받아들였는데, &quot;생성자는 인스턴스가 <strong>태어나는 순간에만</strong> 실행되는 특별한 메서드&quot;라는 걸 다시 떠올리니 왜 일반 메서드와 다르게 생겼는지 조금씩 납득이 갔다. 태어날 때 한 번만 실행되어야 하니, 아무 때나 마음대로 부를 수 없게 문법 자체로 제한을 걸어둔 셈이다.</p>
<h3 id="5-3-생성자-오버로딩constructor-overloading">5-3) 생성자 오버로딩(Constructor Overloading)</h3>
<p>하나의 class 안에 <strong>매개변수의 개수나 타입이 다른 생성자를 여러 개</strong> 만들어둘 수 있다는 개념이다. 예를 들면 &quot;이름만 받는 생성자&quot;와 &quot;이름과 나이를 함께 받는 생성자&quot;를 한 class 안에 둘 다 만들어두고, 상황에 따라 다른 생성자로 인스턴스를 만들 수 있게 하는 것이다. 오늘은 이런 개념이 있다는 것만 배웠고, 직접 코드로 만들어보진 못해서 다음에 실습이 필요한 부분이다.</p>
<hr />
<h2 id="6-wrapper-class--boxingunboxing과-형변환casting">6. Wrapper Class — Boxing/Unboxing과 형변환(Casting)</h2>
<h3 id="6-1-boxing과-unboxing이란">6-1) Boxing과 Unboxing이란</h3>
<pre><code class="language-java">Integer ii = 10;
System.out.println(10 + ii);</code></pre>
<p><code>Integer</code>는 기본타입 <code>int</code>에 대응하는 <strong>참조타입 버전</strong>이다(이런 걸 Wrapper Class, 포장 클래스라고 부른다). 원래는 기본타입 값(<code>int</code> 10)과 참조타입 변수(<code>Integer</code>)는 서로 다른 종류라 그냥 대입이 안 될 것 같은데, 자바가 알아서 변환을 해준다.</p>
<ul>
<li><strong>Boxing</strong>: 기본타입 값(<code>10</code>)을 참조타입 상자(<code>Integer</code>)에 자동으로 포장해서 넣어주는 것. <code>Integer ii = 10;</code>이 바로 이 경우다.</li>
<li><strong>Unboxing</strong>: 반대로 참조타입(<code>Integer</code>)에 들어있는 값을 다시 기본타입처럼 꺼내 쓰는 것. <code>10 + ii</code>에서 <code>ii</code>(Integer)를 숫자 계산에 바로 쓸 수 있는 게 이 Unboxing 덕분이다.</li>
</ul>
<p>즉, 우리가 신경 쓰지 않아도 자바가 필요할 때마다 &quot;상자에 넣기(Boxing)&quot;와 &quot;상자에서 꺼내기(Unboxing)&quot;를 자동으로 처리해준다는 걸로 이해했다.</p>
<h3 id="6-2-형변환casting이-필요한-순간">6-2) 형변환(Casting)이 필요한 순간</h3>
<pre><code class="language-java">byte x = 10, y = 10, sum = 0;
sum = (byte)(x + y);
System.out.println(sum);</code></pre>
<p>이 코드가 오늘 가장 헷갈렸던 부분이다. <code>x</code>, <code>y</code>, <code>sum</code>이 셋 다 <code>byte</code> 타입인데, 왜 <code>sum = x + y;</code>라고 바로 쓰면 안 되고 <code>(byte)</code>를 꼭 붙여야 하는지 처음엔 이해가 안 됐다.</p>
<p><strong>이유를 순서대로 풀어보면:</strong></p>
<ol>
<li>자바는 <code>byte</code>나 <code>short</code>처럼 크기가 작은 정수 타입끼리 연산을 할 때, 계산 실수를 막기 위해 <strong>결과를 자동으로 더 큰 타입인 <code>int</code>로 승격(promotion)</strong>시킨다.</li>
<li>그래서 <code>x + y</code>의 실제 결과 타입은 <code>byte</code>가 아니라 <strong><code>int</code></strong>가 된다.</li>
<li>그런데 그 결과를 다시 <code>byte</code> 타입인 <code>sum</code>에 넣으려고 하니, &quot;더 큰 그릇(int)의 내용물을 더 작은 그릇(byte)에 옮겨 담는&quot; 상황이 된다. 이 과정에서 값이 잘려나갈 위험이 있기 때문에, 자바는 이걸 <strong>개발자가 위험을 알고 있다는 걸 명시적으로 표시</strong>하게 만든다. 그 표시가 바로 <code>(byte)</code>라는 캐스팅이다.</li>
</ol>
<p><strong>형변환이 자동으로 되는 방향과 아닌 방향</strong></p>
<pre><code>byte → short → int → long → float → double   (자동 변환 O, 작은 그릇 → 큰 그릇)
                char → int                      (자동 변환 O)

큰 그릇 → 작은 그릇 (예: int → byte)             (자동 변환 X, 명시적 캐스팅 필요)</code></pre><p>화살표 방향(작은 타입 → 큰 타입)으로 담는 건 자바가 알아서 해주지만, 반대 방향(큰 타입 → 작은 타입)으로 담을 때만 <code>(타입)</code>을 직접 앞에 적어서 &quot;이거 값 잘릴 수도 있는데 내가 알고 하는 거야&quot;라고 알려줘야 한다는 규칙으로 정리했다.</p>
<hr />
<h2 id="7-객체-생성-방법--new-말고-또-있다">7. 객체 생성 방법 — <code>new</code> 말고 또 있다?</h2>
<p>객체를 만드는 방법이 <code>new xxx()</code> 하나만 있는 줄 알았는데, 오늘 <strong>Annotation(어노테이션)을 이용한 Builder 방식</strong>이라는 또 다른 패턴이 있다는 걸 소개받았다.</p>
<ul>
<li><code>lombok</code>이라는 라이브러리를 쓰면, <code>@Builder</code>, <code>@NoArgsConstructor</code>(매개변수 없는 생성자를 자동 생성), <code>@AllArgsConstructor</code>(모든 필드를 매개변수로 받는 생성자를 자동 생성), <code>@Getter</code>(각 변수의 값을 꺼내는 메서드를 자동 생성) 같은 어노테이션을 class 위에 붙이기만 하면, 원래 개발자가 직접 손으로 써야 했던 생성자·getter 코드를 자동으로 만들어준다는 개념을 들었다. 오늘은 &quot;이런 게 있다&quot;까지만 알아뒀고, 다음에 실제로 붙여서 코드가 어떻게 줄어드는지 직접 봐야 실감이 날 것 같다.</li>
<li><code>xxxxDTO.java</code>, <code>xxxxVO.java</code>, <code>xxxxEntity.java</code>라는 네이밍 패턴도 언급됐다. 오늘은 이런 이름을 가진 class들이 존재한다는 정도만 짚고 넘어갔다.</li>
<li><code>has a ~</code> 관계라는 표현도 나왔는데, class 안에 다른 class 타입의 변수를 멤버변수로 갖는 구조(포함 관계)를 가리키는 것 같다고 짐작만 해둔 상태다.</li>
</ul>
<hr />
<h2 id="8-헷갈렸던-점-다음-복습-포인트-🤔">8. 헷갈렸던 점 (다음 복습 포인트) 🤔</h2>
<ul>
<li><strong>class 안 변수·메서드의 소유권</strong>: class 자체가 아니라 인스턴스 각자가 값을 갖는다는 개념이, &quot;붕어빵 틀에는 팥이 없다&quot;는 비유를 떠올려야 겨우 헷갈리지 않는다. 코드를 더 봐야 자연스러워질 것 같다.</li>
<li><strong>생성자 규칙</strong>: 세 가지 규칙(반환타입 없음, 이름이 클래스명과 동일, new 뒤에서만 호출)은 외웠지만, 실제 class를 여러 개 만들어보면서 손에 익혀야 완전히 내 것이 될 것 같다.</li>
<li><strong>byte + byte가 int가 되는 자동 승격</strong>: <code>(byte)(x + y)</code> 캐스팅이 필요한 이유를 오늘 처음 이해했는데, 다른 타입 조합(예: short + short, char + int)에서도 같은 규칙이 적용되는지는 다음에 직접 테스트해봐야겠다.</li>
<li><strong>멤버변수 vs 지역변수</strong>: 선언 위치로 구분한다는 규칙은 알겠는데, 실제 코드를 보면서 즉시 구분하는 감각은 아직 없다.</li>
<li><strong>setter/getter</strong>: 이름만 들었고 오늘은 직접 코드로 다루지 않아서, 왜 필요한지(멤버변수를 <code>private</code>으로 숨기는 것과의 관계)는 다음에 실습하면서 채워야 할 부분.</li>
</ul>
<hr />
<h2 id="9-아직-남겨둔-것">9. 아직 남겨둔 것</h2>
<ul>
<li><strong>OOP의 네 가지 특징(은닉화, 상속, 다형성, 추상화)</strong> — 오늘은 이름만 나열됐고, 각각이 코드로 어떻게 구현되는지는 다루지 않았다.</li>
<li><strong><code>public</code> vs <code>private</code>의 실질적 차이</strong> — 접근지정자라는 개념은 나왔지만, 오늘 실습에서 직접 비교해보지 못해서 &quot;왜 굳이 나눠 쓰는지&quot;에 대한 감이 아직 부족하다.</li>
<li><strong>DTO / VO / Entity의 구분</strong> — 세 네이밍 패턴이 언급만 되고 실제 차이는 다루지 않았다.</li>
</ul>
<hr />
<h2 id="10-다음에-할-일">10. 다음에 할 일</h2>
<ul>
<li><input disabled="" type="checkbox" /> 기본타입 종류(byte/short/int/long/float/double/char/boolean) 암기 — 오늘 자료에도 &quot;암기 필수&quot;로 강조되어 있었다</li>
<li><input disabled="" type="checkbox" /> 생성자 오버로딩 예제 직접 짜보기</li>
<li><input disabled="" type="checkbox" /> <code>new</code>와 Builder(lombok) 방식으로 각각 객체를 만들어서 코드 비교해보기</li>
</ul>
<hr />
<h2 id="11-참고-자료">11. 참고 자료</h2>
<ul>
<li><a href="https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/package-summary.html">https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/package-summary.html</a></li>
</ul>
<hr />
<h2 id="오늘의-한-줄-요약">오늘의 한 줄 요약</h2>
<p>자바 개발 환경(JDK/JRE)과 기본/참조 타입, 그리고 class·생성자의 기본 규칙을 배우며, 자바스크립트와는 다르게 &quot;타입을 명시하고 그 규칙을 지켜야 하는&quot; 언어라는 감각을 잡은 첫날이었다.</p>
<hr />
<h2 id="마무리-회고">마무리 회고</h2>
<p>9일 동안 자바스크립트에 익숙해지고 나서 자바를 처음 보니, 제일 먼저 걸린 건 &quot;타입&quot;이었다. <code>let</code>이나 <code>const</code> 하나로 뭐든 담던 습관이 있어서, <code>byte x = 10, y = 10, sum = 0;</code>처럼 변수마다 타입을 정해줘야 한다는 게 처음엔 번거롭게 느껴졌는데, <code>(byte)(x + y)</code> 캐스팅이 필요한 이유를 알고 나니 오히려 &quot;타입이 있어서 실수를 미리 막아주는구나&quot;라는 생각이 들었다.</p>
<p>class 개념은 &quot;붕어빵 틀&quot;에 비유해서 이해하고 나니 훨씬 편해졌다. 생성자 규칙(반환타입 없음, 이름이 클래스명과 동일, new 뒤에서만 호출)은 아직 외운 수준이라, 왜 이렇게 설계됐는지는 다음 실습에서 직접 만들어보면서 채워야 할 것 같다. 오늘은 개념어가 유독 많았던 하루였다 — OOP 네 가지 특징, DTO/VO/Entity, lombok까지 이름만 등장한 것들이 꽤 있어서, 이 목록을 다음 TIL들에서 하나씩 지워나가는 걸 목표로 삼아야겠다.</p>