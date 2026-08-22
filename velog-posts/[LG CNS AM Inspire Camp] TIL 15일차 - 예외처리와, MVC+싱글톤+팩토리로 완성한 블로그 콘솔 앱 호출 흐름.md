<blockquote>
<p><strong>한 줄 요약</strong></p>
<p>예외처리(try-catch, checked/unchecked)를 먼저 정리하고, <code>BlogReactView</code>(콘솔) → <code>BlogFrontController</code>(파사드) → <code>BlogBeanFactory</code>(싱글톤+팩토리) → <code>ListController</code> → <code>Service</code> → <code>Dao</code>로 이어지는 실제 요청 흐름을 직접 디버깅하며 완성했다.</p>
</blockquote>
<hr />
<h1 id="📚-오늘-학습한-커리큘럼">📚 오늘 학습한 커리큘럼</h1>
<pre><code>예외처리 (Error vs Exception, try-catch-finally, checked/unchecked)
        ↓
전체 구조 한눈에 보기 (View → Front → Factory → Controller → Service → Dao)
        ↓
BlogReactView — 메뉴 입력받는 콘솔 화면
        ↓
BlogFrontController — 파사드(Facade) 패턴
        ↓
BlogBeanFactory — 싱글톤 + 팩토리 패턴
        ↓
ListController → Service → Dao — 의존성 주입으로 이어지는 계층</code></pre><h1 id="📌-오늘의-학습-키워드">📌 오늘의 학습 키워드</h1>
<ul>
<li>Error vs Exception, checked vs unchecked</li>
<li><code>try-catch-finally</code>, <code>throws</code>, <code>throw</code></li>
<li>파사드(Facade) 패턴</li>
<li>싱글톤 + 팩토리 패턴 실전 적용</li>
<li>의존성 주입(DI)</li>
<li><code>NullPointerException</code>, <code>NoSuchElementException</code></li>
</ul>
<hr />
<h1 id="0-개념-정리-키워드-사전">0. 개념 정리 (키워드 사전)</h1>
<blockquote>
<p><strong>한 줄 요약</strong></p>
<p>예외는 &quot;예기치 못한 상황&quot;을 안전하게 처리하는 자바의 장치이고, 오늘 만든 구조는 콘솔 화면(View)이 직접 로직을 모르게 하고, 파사드가 요청을 받아 팩토리에게 &quot;누가 처리할지&quot; 물어보는 방식이다.</p>
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
<td><strong>Error</strong></td>
<td>시스템 레벨의 심각한 오류. 코드로 복구하기 어려움</td>
<td>메모리 부족(OutOfMemoryError) 등</td>
</tr>
<tr>
<td><strong>Exception</strong></td>
<td>프로그램 실행 중 발생할 수 있는, <strong>처리하면 복구 가능한</strong> 예외 상황</td>
<td>&quot;mild Error&quot;라고 필기했듯, Error보다 가벼운 개념</td>
</tr>
<tr>
<td><strong>Checked Exception</strong></td>
<td><strong>컴파일 시점</strong>에 처리 여부를 강제하는 예외</td>
<td><code>IOException</code> — try-catch나 throws 없으면 컴파일 자체가 안 됨</td>
</tr>
<tr>
<td><strong>Unchecked Exception</strong></td>
<td><strong>실행 시점(runtime)</strong>에만 발생하는 예외, 컴파일러가 강제하지 않음</td>
<td><code>RuntimeException</code>과 그 자식들(<code>NullPointerException</code>, <code>NumberFormatException</code> 등)</td>
</tr>
<tr>
<td><strong>try-catch-finally</strong></td>
<td>예외가 날 수 있는 코드를 감싸고, 예외 발생 시 처리 블록으로 넘기는 문법</td>
<td><code>finally</code>는 예외 발생 여부와 상관없이 항상 실행</td>
</tr>
<tr>
<td><strong>throws</strong></td>
<td>메서드 선언부에 붙여, &quot;이 메서드는 이런 예외를 던질 수 있다&quot;고 알리는 키워드</td>
<td>처리를 호출한 쪽으로 미루는 것</td>
</tr>
<tr>
<td><strong>throw</strong></td>
<td>코드 안에서 <strong>직접</strong> 예외를 발생시키는 키워드</td>
<td><code>throw new XXXException();</code></td>
</tr>
<tr>
<td><strong>사용자 정의 예외</strong></td>
<td><code>Exception</code>을 상속해서 직접 만든 예외 클래스</td>
<td><code>class XXXX extends Exception {}</code></td>
</tr>
<tr>
<td><strong>파사드(Facade) 패턴</strong></td>
<td>여러 하위 시스템을 감춘 뒤, <strong>창구 하나만</strong> 외부에 노출하는 패턴</td>
<td><code>BlogFrontController</code></td>
</tr>
<tr>
<td><strong>Bean</strong></td>
<td>팩토리가 미리 만들어서 들고 있다가 요청 시 꺼내주는 객체</td>
<td><code>BlogBeanFactory</code>의 <code>map</code>에 저장된 <code>ListController</code> 등</td>
</tr>
<tr>
<td><strong>의존성 주입(DI)</strong></td>
<td>객체가 필요한 협력 객체를 <strong>직접 만들지 않고 외부에서 받는 것</strong></td>
<td><code>new BlogReactServiceImpl(dao)</code>처럼 생성자로 <code>dao</code>를 넘겨받음</td>
</tr>
</tbody></table>
<h3 id="코드로-다시-보기">코드로 다시 보기</h3>
<pre><code class="language-java">try {
    // 예외가 날 수 있는 코드
} catch (NumberFormatException e) {
    // 그 예외가 발생했을 때 처리
} finally {
    // 예외 발생 여부와 무관하게 항상 실행
}</code></pre>
<hr />
<h1 id="1-예외처리--기초-개념부터-오늘-실제로-겪은-세-가지-예외까지">1. 예외처리 — 기초 개념부터 오늘 실제로 겪은 세 가지 예외까지</h1>
<h2 id="1-1-error-vs-exception-checked-vs-unchecked">1-1) Error vs Exception, checked vs unchecked</h2>
<p><strong>Java</strong> · 배열 범위를 벗어난 접근 (unchecked)</p>
<pre><code class="language-java">String[] strAry = {&quot;hsson&quot;, &quot;inspire&quot;, &quot;lgcns&quot;};
try {
    for (int idx = 0; idx &lt;= strAry.length; idx++) {   // &lt;= 가 문제
        System.out.println(strAry[idx]);
    }
} catch (ArrayIndexOutOfBoundsException e) {
    e.printStackTrace();
}</code></pre>
<p>배열 <code>strAry</code>는 길이가 3(인덱스 0~2)인데, 조건을 <code>idx &lt;= strAry.length</code>(즉 <code>idx &lt;= 3</code>)로 써서 범위를 벗어난다. <code>ArrayIndexOutOfBoundsException</code>은 <strong>런타임에만 발생하는 unchecked 예외</strong>라, <code>try-catch</code> 없이도 컴파일은 되지만 실행하면 예외가 난다.</p>
<p><strong>Java</strong> · 콘솔 입력을 받는 checked 예외</p>
<pre><code class="language-java">BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
String line = null;
try {
    line = br.readLine();
} catch (IOException e) {
    e.printStackTrace();
}</code></pre>
<p><code>br.readLine()</code>은 <code>IOException</code>을 던질 수 있는데, <code>IOException</code>은 <strong>checked exception</strong>이라 <code>try-catch</code>(또는 <code>throws</code>)로 처리해주지 않으면 <strong>컴파일 자체가 안 된다.</strong> unchecked(<code>NumberFormatException</code>, <code>NullPointerException</code> 등)는 컴파일은 통과하고 실행 중에만 터진다는 게 핵심 차이.</p>
<h2 id="1-2-실전에서-겪은-예외-①--catchexception-e가-진짜-원인을-가림">1-2) 실전에서 겪은 예외 ① — <code>catch(Exception e)</code>가 진짜 원인을 가림</h2>
<pre><code class="language-java">try {
    Integer number = Integer.parseInt(scan.nextLine());
    switch (number) {
        case 1:
            list();
            break;
        ...
    }
} catch (Exception e) {
    System.out.println(&quot;ERR &gt;&gt;&gt; 메뉴에 있는 숫자만 가능합니다. &quot;);
}</code></pre>
<p><code>&quot;1&quot;</code>을 입력하면 <code>Integer.parseInt</code>는 문제없이 통과하고 <code>list()</code>가 실행되는데, 그 안에서 전혀 다른 예외(나중에 확인해보니 <code>NullPointerException</code>)가 나도 <strong><code>catch (Exception e)</code>가 부모 타입이라 전부 다 걸려버려서</strong>, 실제로는 숫자와 무관한 에러인데도 &quot;메뉴에 있는 숫자만 가능합니다&quot;라는 엉뚱한 메시지가 떴다. <code>e.printStackTrace()</code>를 추가하고 나서야 진짜 예외 이름을 확인할 수 있었다.</p>
<h2 id="1-3-실전에서-겪은-예외-②--nullpointerexception과-공백-하나">1-3) 실전에서 겪은 예외 ② — <code>NullPointerException</code>과 공백 하나</h2>
<p><code>e.printStackTrace()</code>로 확인한 진짜 예외는 <code>NullPointerException</code>이었다. 원인을 추적한 순서:</p>
<ol>
<li><code>BlogFrontController.list()</code>의 <code>factory.getBean(endPoint)</code>가 <code>null</code>을 반환하고 있었음</li>
<li><code>BlogBeanFactory</code>의 <code>map.put(&quot;list.inspire &quot;, ...)</code> — 등록할 때 키 끝에 <strong>공백이 하나</strong> 껴 있었음</li>
<li>반면 조회할 때는 <code>&quot;list.inspire&quot;</code>(공백 없음)로 찾고 있어서, 문자열로는 서로 다른 값 → <code>map.get(...)</code>이 못 찾고 <code>null</code> 반환</li>
<li>그 <code>null</code>을 <code>((ListController) controller).list()</code>로 그대로 호출하려다 <code>NullPointerException</code> 발생</li>
</ol>
<p>눈으로는 완전히 똑같아 보이는 두 문자열이 실제로는 다른 값이라는 걸 직접 겪고 나서야, 왜 로그(<code>e.printStackTrace()</code>)를 먼저 찍어보는 습관이 중요한지 체감했다.</p>
<p>했을 때, 실제로는 이런 순서로 객체들이 서로를 호출한다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/hssondev/post/dd6b82be-ef93-4146-a526-48a40c42c6dc/image.png" /></p>
<p>콘솔에 실제로 찍힌 로그와 다이어그램을 맞춰보면 정확히 일치한다:</p>
<pre><code>원하시는 메뉴 번호를 입력하시오 : 1
&gt;&gt;&gt;&gt;&gt; 데이터 출력
debug &gt;&gt;&gt;&gt; front controller endpoint : list.inspire
debug &gt;&gt;&gt;&gt;&gt; list controller list()
debug&gt;&gt;&gt;&gt;&gt;&gt; blog service list
debug &gt;&gt;&gt;&gt;&gt; blog dao findAll()
BlogResponseDTO(status=0, ... title=react, ...)</code></pre><p>이 4줄의 디버그 로그가 다이어그램의 <code>View → Front → Controller → Service → Dao</code> 화살표 하나하나에 정확히 대응한다. 아래 3~6번에서 이 여섯 단계를 코드 한 줄씩 순서대로 따라가 본다.</p>
<hr />
<h1 id="3-blogreactview--콘솔-메뉴와-무한루프">3. <code>BlogReactView</code> — 콘솔 메뉴와 무한루프</h1>
<pre><code class="language-java">public void landingPage(){
    while(true) {                                          // ①
        System.out.println(&quot;1. 전체검색&quot;);                    // ②
        ...
        System.out.println(&quot;99.프로그램 종료&quot;);
        System.out.print(&quot;원하시는 메뉴 번호를 입력하시오 : &quot;);

        try {
            Integer number = Integer.parseInt(scan.nextLine());  // ③
            switch (number) {                                     // ④
                case 1:
                    list();                                       // ⑤
                    break;
                case 99:
                    exit();
                    break;
                default:
                    break;
            }
        } catch (Exception e) {                                  // ⑥
            e.printStackTrace();
            System.out.println(&quot;ERR &gt;&gt;&gt; 메뉴에 있는 숫자만 가능합니다. &quot;);
        }
    }
}

public void list(){
    String endPoint = &quot;list.inspire&quot;;                        // ⑦
    List&lt;BlogResponseDTO&gt; response = front.list(endPoint);    // ⑧
    response.stream().forEach(System.out::println);           // ⑨
}</code></pre>
<p><strong>한 줄씩 뜯어보면</strong></p>
<ul>
<li>① <code>while(true)</code>: 조건이 항상 참이라, 안에 있는 코드를 <strong>무한히 반복</strong>한다. 메뉴를 한 번 보여주고 끝나는 게 아니라, 사용자가 종료(99번)를 선택하기 전까지 계속 메뉴를 다시 보여주기 위한 구조다.</li>
<li>② <code>System.out.println(...)</code> 여러 줄: 화면에 메뉴 목록을 그냥 순서대로 출력하는 것뿐이다. 특별한 로직은 없다.</li>
<li>③ <code>scan.nextLine()</code>으로 사용자가 입력한 한 줄을 문자열로 받고, <code>Integer.parseInt(...)</code>로 그 문자열을 정수로 바꾼다. <code>&quot;1&quot;</code>이라는 문자와 <code>1</code>이라는 숫자는 자바에서 다른 타입이라, 이 변환이 꼭 필요하다.</li>
<li>④ <code>switch (number)</code>: 방금 변환한 숫자 값에 따라 아래 <code>case</code> 중 하나로 분기한다. 지금은 <code>case 1</code>(전체검색)과 <code>case 99</code>(종료)만 실제로 동작이 연결돼 있고, 나머지는 <code>default</code>로 빠져서 아무 일도 안 일어난다.</li>
<li>⑤ <code>case 1</code>에 해당하면 <code>list()</code> 메서드를 호출한다 — 이 메서드가 바로 아래 ⑦~⑨번이다.</li>
<li>⑥ <code>catch (Exception e)</code>: ③~⑤ 사이에서 어떤 예외든 발생하면 여기로 떨어진다. (이게 왜 문제였는지는 1-2번에서 이미 다뤘다.)</li>
<li>⑦ <code>list()</code> 메서드 안에서, <code>&quot;list.inspire&quot;</code>라는 <strong>문자열 하나</strong>를 <code>endPoint</code>라는 변수에 담아둔다. 이건 그냥 &quot;어떤 기능을 요청하는지&quot;를 나타내는 이름표일 뿐이다.</li>
<li>⑧ <code>front.list(endPoint)</code>: 방금 만든 이름표를 <code>front</code>(= <code>BlogFrontController</code> 객체)에게 그대로 넘긴다. <code>BlogReactView</code>는 이 이름표로 실제로 어떤 클래스가 어떻게 동작하는지 <strong>전혀 모른다</strong> — 그냥 &quot;이 이름표에 해당하는 걸 처리해줘&quot;라고 부탁만 하는 것이다. 이 호출이 다이어그램의 첫 번째 화살표다.</li>
<li>⑨ <code>front.list(endPoint)</code>가 결과로 돌려준 목록(<code>response</code>)을 하나씩 꺼내서 화면에 출력한다. <code>.stream().forEach(...)</code>는 리스트 안의 항목을 하나씩 꺼내 괄호 안의 동작(여기선 출력)을 반복 실행하는 문법이다.</li>
</ul>
<hr />
<h1 id="4-blogfrontcontroller--파사드facade-패턴">4. <code>BlogFrontController</code> — 파사드(Facade) 패턴</h1>
<pre><code class="language-java">public class BlogFrontController {
    private BlogBeanFactory factory;                          // ①

    public BlogFrontController() {
        factory = BlogBeanFactory.getInstance();               // ②
    }

    public List&lt;BlogResponseDTO&gt; list(String endPoint) {       // ③
        System.out.println(&quot;debug &gt;&gt;&gt;&gt; front controller endpoint : &quot; + endPoint);
        Object controller = factory.getBean(endPoint);         // ④
        return ((ListController) controller).list();           // ⑤
    }
}</code></pre>
<p><strong>한 줄씩 뜯어보면</strong></p>
<ul>
<li>① <code>factory</code>라는 필드를 선언해둔다. <code>BlogFrontController</code>가 팩토리를 계속 기억해두고 쓰기 위해서다.</li>
<li>② 생성자에서 <code>BlogBeanFactory.getInstance()</code>를 호출해서, 팩토리 객체를 가져와 <code>factory</code>에 저장한다. (이 팩토리가 싱글톤이라는 건 5번에서 다룬다.)</li>
<li>③ <code>list(String endPoint)</code>: <code>BlogReactView</code>가 넘긴 <code>endPoint</code>(예: <code>&quot;list.inspire&quot;</code>)를 매개변수로 받는다. 이 메서드가 3번 섹션 ⑧의 <code>front.list(endPoint)</code> 호출과 정확히 연결되는 지점이다 — 다이어그램의 두 번째 화살표.</li>
<li>④ <code>factory.getBean(endPoint)</code>: 받은 이름표를 그대로 팩토리에게 넘겨서 &quot;이 이름표에 등록된 객체를 찾아줘&quot;라고 요청한다. 이때 반환 타입이 <code>Object</code>인 이유는, 팩토리 안에는 <code>ListController</code>뿐 아니라 나중에 다른 여러 종류의 컨트롤러가 함께 들어갈 수 있기 때문이다 — <code>Object</code>는 자바의 모든 타입을 담을 수 있는 가장 넓은 타입이라, 팩토리 입장에서는 &quot;뭐가 들어있는지 모르지만 일단 꺼내준다&quot;는 뜻으로 쓰인다.</li>
<li>⑤ 그런데 <code>Object</code> 타입인 채로는 <code>.list()</code> 메서드를 바로 호출할 수 없다 — <code>Object</code>는 그런 메서드를 모르기 때문이다. 그래서 <code>(ListController)</code>로 <strong>캐스팅</strong>해서 &quot;이 값은 사실 ListController 타입이야&quot;라고 자바에게 알려준 다음에야 <code>.list()</code>를 호출할 수 있다. 이 결과를 그대로 <code>return</code>해서, <code>BlogReactView</code>의 ⑧번으로 다시 돌려준다.</li>
</ul>
<p><strong>파사드(Facade) 패턴이란</strong>: <code>BlogReactView</code>는 이 안에서 <code>factory</code>가 뭘 하는지, <code>ListController</code>가 어떻게 생겼는지 전혀 몰라도 된다. <code>BlogFrontController</code>가 그 뒤에 있는 여러 객체를 다 가려주고, <code>list(endPoint)</code>라는 <strong>창구 하나</strong>만 밖으로 보여주기 때문이다.</p>
<hr />
<h1 id="5-blogbeanfactory--싱글톤--팩토리">5. <code>BlogBeanFactory</code> — 싱글톤 + 팩토리</h1>
<pre><code class="language-java">public class BlogBeanFactory {
    private Map&lt;String, Object&gt; map;                 // ①
    private static BlogBeanFactory instance;          // ②

    private BlogReactService service;
    private BlogReactDao dao;

    private BlogBeanFactory() {                       // ③
        map = new HashMap&lt;&gt;();
        dao = new BlogReactDao();                      // ④
        service = new BlogReactServiceImpl(dao);        // ⑤
        map.put(&quot;list.inspire&quot;, new ListController(service));  // ⑥
    }

    public static BlogBeanFactory getInstance() {      // ⑦
        if (instance == null) {
            instance = new BlogBeanFactory();
        }
        return instance;
    }

    public Object getBean(String endPoint) {           // ⑧
        return map.get(endPoint);
    }
}</code></pre>
<p><strong>한 줄씩 뜯어보면</strong></p>
<ul>
<li>① <code>map</code>: 문자열(키)과 객체(값)를 짝지어 저장하는 자료구조. <code>&quot;list.inspire&quot;</code>라는 이름표에 <code>ListController</code> 객체를 매달아두는 용도로 쓰인다.</li>
<li>② <code>instance</code>: 이 클래스 자신의 유일한 인스턴스를 담아둘 자리. <code>static</code>이라 클래스 전체에 하나만 존재한다 — 이게 싱글톤의 핵심 재료다.</li>
<li>③ 생성자가 <code>private</code>이다 — 클래스 바깥에서 <code>new BlogBeanFactory()</code>를 직접 실행할 수 없게 막아둔 것. 오직 이 클래스 내부(⑦번)에서만 생성자를 부를 수 있다.</li>
<li>④~⑤ 생성자 안에서 <code>BlogReactDao</code>와 <code>BlogReactServiceImpl</code>을 미리 만들어둔다. 이때 <code>service</code>를 만들 때 방금 만든 <code>dao</code>를 생성자 매개변수로 그대로 넘긴다 — 이게 6번에서 다룰 의존성 주입이다.</li>
<li>⑥ <code>map.put(&quot;list.inspire&quot;, new ListController(service))</code>: <code>ListController</code>를 새로 만들면서 그 안에 <code>service</code>를 넘겨주고, 완성된 <code>ListController</code> 객체를 <code>&quot;list.inspire&quot;</code>라는 이름표를 붙여 <code>map</code>에 저장한다. (이 문자열에 공백이 껴 있었던 게 1-3번에서 다룬 버그의 원인이었다.)</li>
<li>⑦ <code>getInstance()</code>: <code>instance</code>가 아직 <code>null</code>이면(=처음 호출이면) 그제서야 <code>new BlogBeanFactory()</code>로 하나를 만들고, 이미 만들어져 있으면 그 기존 것을 그대로 돌려준다. 그래서 이 메서드를 몇 번을 호출하든 <strong>항상 같은 객체 하나</strong>만 나온다 — 이게 다이어그램에서 팩토리가 한 번만 등장하는 이유다.</li>
<li>⑧ <code>getBean(endPoint)</code>: <code>map</code>에서 넘겨받은 이름표(<code>endPoint</code>)로 저장된 객체를 찾아서 돌려준다. 이게 4번 섹션 ④번의 <code>factory.getBean(endPoint)</code> 호출과 연결되는 지점이다 — 다이어그램의 세 번째 화살표.</li>
</ul>
<hr />
<h1 id="6-listcontroller-→-service-→-dao--의존성-주입으로-이어지는-계층">6. <code>ListController → Service → Dao</code> — 의존성 주입으로 이어지는 계층</h1>
<h2 id="6-1-listcontroller">6-1) <code>ListController</code></h2>
<pre><code class="language-java">public class ListController {
    private BlogReactService service;                    // ①

    public ListController(BlogReactService service) {    // ②
        this.service = service;
    }

    public List&lt;BlogResponseDTO&gt; list() {                 // ③
        System.out.println(&quot;debug &gt;&gt;&gt;&gt;&gt; list controller list()&quot;);
        return service.list();                             // ④
    }
}</code></pre>
<ul>
<li>① <code>service</code>라는 필드를 선언해둔다. <code>ListController</code>가 실제 데이터 처리는 직접 하지 않고, 이 <code>service</code>에게 맡길 것이기 때문이다.</li>
<li>② 생성자가 <code>service</code>를 매개변수로 받는다 — 5번 섹션 ⑥번에서 <code>new ListController(service)</code>로 넘겨준 그 <code>service</code>가 바로 여기로 들어온다. <code>ListController</code> 스스로는 <code>new BlogReactServiceImpl(...)</code>을 직접 실행하지 않는다는 게 핵심이다.</li>
<li>③ <code>list()</code>: 4번 섹션 ⑤번의 <code>((ListController) controller).list()</code> 호출과 연결되는 지점 — 다이어그램의 네 번째 박스.</li>
<li>④ <code>service.list()</code>: 자기가 할 일을 직접 처리하지 않고, 갖고 있는 <code>service</code>에게 그대로 넘긴다.</li>
</ul>
<h2 id="6-2-blogreactserviceimpl">6-2) <code>BlogReactServiceImpl</code></h2>
<pre><code class="language-java">public class BlogReactServiceImpl implements BlogReactService {   // ①
    private BlogReactDao dao;

    public BlogReactServiceImpl(BlogReactDao dao) {                 // ②
        this.dao = dao;
    }

    @Override
    public List&lt;BlogResponseDTO&gt; list() {                           // ③
        System.out.println(&quot;debug&gt;&gt;&gt;&gt;&gt;&gt; blog service list&quot;);
        return dao.findByAll();                                      // ④
    }
}</code></pre>
<ul>
<li>① <code>implements BlogReactService</code>: 이 클래스가 <code>BlogReactService</code>라는 인터페이스의 규격을 따른다는 뜻. (이 인터페이스가 왜 필요한지는 아래에서 설명한다.)</li>
<li>② 생성자가 <code>dao</code>를 매개변수로 받는다 — 5번 섹션 ⑤번에서 넘겨준 <code>dao</code>가 여기로 들어온다. <code>ListController</code>와 똑같은 패턴이다: 필요한 협력 객체를 직접 만들지 않고 <strong>받는다.</strong></li>
<li>③ <code>list()</code>가 6-1번의 ④번(<code>service.list()</code>)과 연결되는 지점 — 다이어그램의 다섯 번째 박스.</li>
<li>④ <code>dao.findByAll()</code>: 여기서도 직접 데이터를 처리하지 않고, <code>dao</code>에게 그대로 넘긴다.</li>
</ul>
<h2 id="6-3-blogreactdao">6-3) <code>BlogReactDao</code></h2>
<pre><code class="language-java">public class BlogReactDao {
    private List&lt;BlogResponseDTO&gt; blogs;              // ①

    public BlogReactDao() {
        blogs = new ArrayList&lt;&gt;(List.of(                // ②
            BlogResponseDTO.builder().blogId(1).title(&quot;react&quot;)/*...*/.build(),
            /* ... 4건 더 ... */
        ));
    }

    public List&lt;BlogResponseDTO&gt; findByAll() {          // ③
        System.out.println(&quot;debug &gt;&gt;&gt;&gt;&gt; blog dao findAll()&quot;);
        return blogs;                                      // ④
    }
}</code></pre>
<ul>
<li>① <code>blogs</code>: 게시글 데이터를 담아둘 리스트.</li>
<li>② 생성자에서 미리 5건의 <code>BlogResponseDTO</code>를 만들어 <code>blogs</code>에 채워넣는다. 지금은 진짜 DB 대신 이렇게 코드 안에 데이터를 미리 넣어두고 흉내를 내는 것이다.</li>
<li>③ <code>findByAll()</code>이 6-2번의 ④번(<code>dao.findByAll()</code>)과 연결되는 지점 — 다이어그램의 마지막 박스.</li>
<li>④ 미리 채워둔 <code>blogs</code> 리스트를 그대로 반환한다. 여기서 만들어진 결과가 <code>Service → Controller → Front → View</code> 순서로 다시 거슬러 올라가면서, 3번 섹션 ⑨번에서 화면에 출력된다.</li>
</ul>
<h2 id="6-4-왜-이렇게-받기만-하는가--의존성-주입di">6-4) 왜 이렇게 &quot;받기만&quot; 하는가 — 의존성 주입(DI)</h2>
<p><code>ListController</code>도, <code>BlogReactServiceImpl</code>도 필요한 객체(<code>service</code>, <code>dao</code>)를 <strong>자기 손으로 <code>new</code>하지 않는다.</strong> 대신 생성자를 통해 <strong>외부(BlogBeanFactory)로부터 완성된 객체를 그대로 받는다.</strong> 이걸 의존성 주입(DI)이라고 부른다.</p>
<p>이렇게 하는 이유는, <code>BlogBeanFactory</code> 한 곳만 보면 &quot;누가 누구를 필요로 하는지&quot;가 전부 드러나고, 각 클래스는 &quot;나는 이런 타입을 하나 받아서 쓴다&quot;는 것만 알면 되기 때문이다. 만약 <code>ListController</code>가 내부에서 직접 <code>new BlogReactServiceImpl(...)</code>을 실행했다면, 나중에 서비스 구현이 바뀔 때마다 <code>ListController</code> 코드까지 같이 고쳐야 했을 것이다.</p>
<p><code>BlogReactServiceImpl</code>이 <code>BlogReactService</code> <strong>인터페이스</strong>를 구현하고 있다는 점도 여기서 힘을 발휘한다. <code>ListController</code>는 <code>BlogReactService</code> 타입으로만 <code>service</code>를 들고 있으므로, 실제 구현체가 <code>BlogReactServiceImpl</code>이든 나중에 다른 구현체로 바뀌든 <code>ListController</code>의 코드는 전혀 바꿀 필요가 없다.</p>
<hr />
<h1 id="7-디버거로-직접-흐름-따라가기">7. 디버거로 직접 흐름 따라가기</h1>
<p>코드만 눈으로 읽어서는 6단계 호출 순서가 잘 안 와닿아서, 직접 디버그 모드로 하나씩 멈춰가며 확인해봤다.</p>
<p><strong>1) 브레이크포인트(중단점) 찍은 위치 — 각 단계의 핵심 줄 하나씩</strong></p>
<ol>
<li><code>BlogReactView.list()</code> — <code>String endPoint = &quot;list.inspire&quot;;</code></li>
<li><code>BlogFrontController.list()</code> — <code>Object controller = factory.getBean(endPoint);</code></li>
<li><code>BlogBeanFactory.getBean()</code> — <code>return map.get(endPoint);</code></li>
<li><code>ListController.list()</code> — <code>return service.list();</code></li>
<li><code>BlogReactServiceImpl.list()</code> — <code>return dao.findByAll();</code></li>
<li><code>BlogReactDao.findByAll()</code> — <code>return blogs;</code></li>
</ol>
<p><strong>2) 진행 방법</strong></p>
<ul>
<li><code>BlogReactApp.java</code>를 Debug 모드(벌레 아이콘, 또는 F5)로 실행</li>
<li>콘솔에서 <code>1</code>을 입력하면 첫 번째 브레이크포인트에서 멈춤</li>
<li><strong>Step Into(F11)</strong>: 지금 줄이 다른 메서드를 호출하면 그 메서드 <strong>안으로</strong> 들어감 (예: <code>front.list(endPoint)</code>에서 누르면 <code>BlogFrontController.list()</code> 내부로 이동)</li>
<li><strong>Step Over(F10)</strong>: 메서드 안으로 들어가지 않고, 지금 줄만 실행하고 다음 줄로 이동</li>
<li>좌측 VARIABLES 패널에서 <code>endPoint</code>, <code>controller</code>, <code>number</code> 같은 변수에 <strong>실시간으로 어떤 값이 들어있는지</strong> 확인</li>
</ul>
<p><strong>3) 실제로 확인한 것</strong></p>
<p><code>Step Into</code>를 계속 눌러가며 ①→②→③→④→⑤→⑥ 순서로 이동해보니, 위 2번 섹션 다이어그램(<code>View → Front → Factory → Controller → Service → Dao</code>)과 정확히 같은 순서로 화면이 옮겨갔다. 특히 <code>factory.getBean(endPoint)</code> 줄에서 <code>endPoint</code> 변수에 마우스를 올려보면 지금 이 변수에 담긴 문자열을 정확한 값 그대로 툴팁으로 보여준다 — 1-3번에서 겪었던 공백 버그도, 이렇게 값을 직접 눈으로 확인했다면 코드를 안 고쳐도 바로 알아챌 수 있었을 것 같다.</p>
<p>코드를 읽을 때는 &quot;이렇게 흘러가겠지&quot;라고 머릿속으로 추측만 했는데, 디버거로 직접 멈춰가며 값을 확인하니 &quot;진짜 이 순서로, 이 값을 들고 넘어가는구나&quot;가 훨씬 확실하게 체감됐다.</p>
<hr />
<ul>
<li>checked exception(<code>IOException</code>)은 컴파일러가 처리를 강제하고, unchecked exception(<code>NullPointerException</code> 등)은 실행해봐야 알 수 있다.</li>
<li><code>catch (Exception e)</code>처럼 최상위 타입으로 예외를 잡으면, 의도하지 않은 다른 예외까지 다 걸려서 진짜 원인을 가릴 수 있다 — <code>e.printStackTrace()</code>로 항상 실제 예외를 먼저 확인하는 습관이 필요하다.</li>
<li><code>factory.getBean()</code>이 <code>Object</code>를 돌려주는 이유는, 팩토리 안에 여러 종류의 객체가 함께 들어있을 수 있기 때문이고, 그래서 꺼낼 때 원래 타입으로 캐스팅이 필요하다.</li>
<li>싱글톤(인스턴스 개수 통제) + 팩토리(무엇을 줄지 통제) + DI(직접 안 만들고 받기)가 합쳐지면, View는 실제 로직을 전혀 몰라도 동작하는 구조를 만들 수 있다.</li>
<li>문자열 키 하나의 공백 차이가 눈으로는 안 보이지만 완전히 다른 값이 된다.</li>
</ul>
<hr />
<h1 id="🤔-헷갈렸던-점">🤔 헷갈렸던 점</h1>
<ul>
<li><strong><code>catch(Exception e)</code>의 위험성</strong>: 부모 타입으로 예외를 잡는 게 다형성처럼 편리해 보였는데, 여기서는 오히려 원인을 숨기는 역효과를 냈다. &quot;잡을 예외는 최대한 구체적으로&quot; 잡아야 한다는 감각이 아직 부족하다.</li>
<li><strong><code>NoSuchElementException: No line found</code></strong>: 언제 나는 예외인지는 확인했지만, 오늘 상황에서 정확히 왜 났는지는 못 밝혔다.</li>
<li><strong>checked vs unchecked를 코드만 보고 구분하기</strong>: 어떤 예외가 checked인지 unchecked인지는 아직 다 외우지 못했다.</li>
</ul>
<hr />
<h1 id="다음에-할-일">다음에 할 일</h1>
<ul>
<li><input disabled="" type="checkbox" /> <code>BlogBeanFactory.getInstnace()</code> 오타(<code>Instnace</code> → <code>Instance</code>) 전체 파일에서 일괄 정리</li>
<li><input disabled="" type="checkbox" /> <code>NoSuchElementException</code>이 재현되면 원인 정확히 파악하기</li>
<li><input disabled="" type="checkbox" /> <code>catch(Exception e)</code>를 실제 예상되는 예외(<code>NumberFormatException</code> 등)로 좁혀서 다시 작성해보기</li>
<li><input disabled="" type="checkbox" /> 2번(상세보기)부터 6번(검색)까지 나머지 메뉴도 같은 구조로 구현하기</li>
</ul>
<hr />
<h1 id="💡-오늘의-회고">💡 오늘의 회고</h1>
<p>오늘은 배운 걸 바로 실전 디버깅으로 확인한 하루였다. <code>catch(Exception e)</code>가 얼마나 위험한지 이론으로 듣기만 했으면 와닿지 않았을 텐데, &quot;메뉴 숫자 에러&quot;라는 엉뚱한 메시지 뒤에 진짜는 <code>NullPointerException</code>이 숨어있던 걸 직접 겪고 나니 왜 예외를 구체적으로 잡아야 하는지 몸으로 이해했다.</p>
<p><code>&quot;list.inspire &quot;</code>와 <code>&quot;list.inspire&quot;</code>의 공백 하나 차이로 전체 흐름이 막혔던 것도 인상 깊었다 — 싱글톤, 팩토리, 파사드, DI까지 큰 개념들을 다 맞게 짰어도, 문자열 하나가 안 맞으면 아무 소용이 없다는 걸 느꼈다. 코드를 한 줄씩 번호 매겨 다시 뜯어보니, View부터 Dao까지 6단계가 사실은 &quot;요청을 한 단계씩 아래로 위임하고, 결과를 한 단계씩 위로 돌려받는&quot; 아주 단순한 왕복 구조라는 것도 눈으로 확인할 수 있었다.</p>