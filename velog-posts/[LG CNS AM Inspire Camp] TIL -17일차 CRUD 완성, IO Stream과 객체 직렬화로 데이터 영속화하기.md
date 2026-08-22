<blockquote>
<p><strong>한 줄 요약</strong></p>
<p>어제 만든 아키텍처에 수정(update)·삭제(delete)를 마저 붙여 CRUD를 완성하고, <code>System.in.read()</code>부터 시작해 IO Stream을 발전시키며 <code>Serializable</code>로 블로그 목록을 파일에 저장·복원하는 것까지 완성했다.</p>
</blockquote>
<hr />
<h1 id="📚-오늘-학습한-커리큘럼">📚 오늘 학습한 커리큘럼</h1>
<pre><code>Stream 복습 — 컬렉션(DTO)을 대상으로 filter·map·collect·Optional 재확인
        ↓
CRUD 완성 — DeleteController / UpdateController 추가
        ↓
IO Stream의 진화 — System.in.read() → BufferedReader → try-with-resources → 파일 IO
        ↓
객체 직렬화(Serializable) — ObjectOutputStream / ObjectInputStream
        ↓
프로그램 시작·종료 시 자동 저장/불러오기 연결
        ↓
ResponseEntity&lt;T&gt; — 응답을 상태코드+메시지+데이터로 감싸기</code></pre><h1 id="📌-오늘의-학습-키워드">📌 오늘의 학습 키워드</h1>
<ul>
<li><code>removeIf</code>, Stream의 <code>filter().findAny().map().orElse()</code> 조합</li>
<li><code>try-with-resources</code>, <code>AutoCloseable</code></li>
<li><code>BufferedReader</code>/<code>BufferedWriter</code>, <code>FileReader</code>/<code>FileWriter</code></li>
<li><code>Serializable</code>, <code>serialVersionUID</code></li>
<li><code>ObjectOutputStream</code>/<code>ObjectInputStream</code> (직렬화/역직렬화)</li>
<li>제네릭 래퍼 클래스(<code>ResponseEntity&lt;T&gt;</code>)</li>
</ul>
<hr />
<h1 id="0-개념-정리-키워드-사전">0. 개념 정리 (키워드 사전)</h1>
<blockquote>
<p><strong>한 줄 요약</strong></p>
<p>IO Stream은 &quot;데이터가 흐르는 통로&quot;이고, 그 통로로 객체를 통째로 파일에 흘려보내는 것이 직렬화(Serializable)다.</p>
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
<td><strong><code>removeIf</code></strong></td>
<td>리스트에서 조건에 맞는 요소를 찾아 <strong>바로 제거</strong>하는 메서드</td>
<td><code>blogs.removeIf(blog -&gt; blog.getBlogId() == blogId)</code></td>
</tr>
<tr>
<td><strong><code>try-with-resources</code></strong></td>
<td><code>try(...)</code> 괄호 안에서 자원을 열면, 블록이 끝날 때 <strong>자동으로 close()</strong> 해주는 문법</td>
<td><code>AutoCloseable</code>을 구현한 클래스만 가능</td>
</tr>
<tr>
<td><strong><code>BufferedReader</code>/<code>BufferedWriter</code></strong></td>
<td>문자 단위 입출력을 한 번에 여러 글자씩 묶어(버퍼링) 처리해서 더 효율적으로 읽고 쓰는 클래스</td>
<td><code>System.in</code>, 파일 등과 함께 사용</td>
</tr>
<tr>
<td><strong><code>Serializable</code></strong></td>
<td>이 인터페이스를 구현한 클래스는 &quot;객체를 파일이나 네트워크로 통째로 보낼 수 있다&quot;는 표시만 하는 <strong>마커 인터페이스</strong>(메서드가 없음)</td>
<td><code>class BlogResponseDTO implements Serializable</code></td>
</tr>
<tr>
<td><strong>직렬화 / 역직렬화</strong></td>
<td>객체를 바이트로 바꿔 저장(직렬화)하고, 그 바이트를 다시 객체로 복원(역직렬화)하는 과정</td>
<td><code>ObjectOutputStream.writeObject()</code> / <code>ObjectInputStream.readObject()</code></td>
</tr>
<tr>
<td><strong><code>serialVersionUID</code></strong></td>
<td>직렬화된 데이터와 클래스가 같은 버전인지 확인하는 데 쓰이는 고유 번호</td>
<td>명시하지 않으면 자바가 자동 생성(위험할 수 있어 직접 지정 권장)</td>
</tr>
<tr>
<td><strong>제네릭 래퍼 클래스</strong></td>
<td><code>&lt;T&gt;</code>처럼 타입을 나중에 정하도록 열어두고, 그 안에 여러 종류의 데이터를 공통 형식으로 감싸는 클래스</td>
<td><code>ResponseEntity&lt;List&lt;BlogResponseDTO&gt;&gt;</code></td>
</tr>
</tbody></table>
<h3 id="코드로-다시-보기">코드로 다시 보기</h3>
<pre><code class="language-java">try (BufferedReader br = new BufferedReader(new InputStreamReader(System.in))) {
    System.out.println(br.readLine());
} catch (Exception e) {
    e.printStackTrace();
}
// br.close()를 따로 안 써도, try 블록이 끝나면 자동으로 닫힘</code></pre>
<hr />
<h1 id="1-stream-복습--이번엔-dto를-대상으로">1. Stream 복습 — 이번엔 DTO를 대상으로</h1>
<p>어제 문자열 위주로 연습했던 Stream을, 오늘은 실제 <code>BlogRequestDTO</code>/<code>BlogResponseDTO</code> 객체를 다루는 데 바로 활용했다. (filter/map/collect/Optional의 기본 개념은 이미 정리해뒀으니, 오늘은 그게 실전 CRUD 코드에 어떻게 쓰였는지에 집중한다.)</p>
<hr />
<h1 id="2-crud-완성--삭제delete">2. CRUD 완성 — 삭제(delete)</h1>
<pre><code class="language-java">public int delete(int blogId) {                                    // ①
    System.out.println(&quot;debug &gt;&gt;&gt;&gt; blog dao delete params :  &quot;+ blogId);
    boolean isFlag = blogs.removeIf(blog -&gt; blog.getBlogId() == blogId);  // ②
    return (isFlag) ? 1 : 0;                                          // ③
}</code></pre>
<ul>
<li>① <code>BlogReactDao.delete(int blogId)</code>: 삭제할 게시글의 id를 매개변수로 받는다.</li>
<li>② <code>blogs.removeIf(조건)</code>: 리스트를 순회하면서 조건(<code>blog.getBlogId() == blogId</code>)에 맞는 요소를 <strong>찾아서 바로 제거</strong>하고, 하나라도 지웠으면 <code>true</code>를 반환한다. 예전처럼 <code>for</code>문을 직접 돌면서 지울 위치를 찾고 <code>remove()</code>를 호출하는 것보다 훨씬 짧다.</li>
<li>③ 지워졌으면 <code>1</code>(성공), 못 지웠으면(그런 id가 없었으면) <code>0</code>(실패)을 반환한다 — 16일차에 정리했던 &quot;0/1로 성공 여부를 나타내는 관례&quot;가 여기서도 이어진다.</li>
</ul>
<p>호출 흐름은 어제 정리한 것과 동일한 패턴을 그대로 반복한다: <code>BlogReactView.delete()</code> → <code>BlogFrontController.delete()</code> → <code>BlogBeanFactory</code>에 등록해둔 <code>DeleteController</code> → <code>Service.delete()</code> → <code>Dao.delete()</code>. 새 기능을 붙일 때마다 이 네 단계(FrontController에 메서드 추가 → Controller 클래스 생성 → Factory에 등록 → Service/Dao 구현)를 반복한다는 게 이 아키텍처의 패턴이라는 걸 다시 확인했다.</p>
<hr />
<h1 id="3-crud-완성--수정update">3. CRUD 완성 — 수정(update)</h1>
<pre><code class="language-java">public int update(BlogRequestDTO request) {
    System.out.println(&quot;debug &gt;&gt;&gt;&gt; blog dao update params :  &quot;+ request);
    return blogs.stream()
        .filter(blog -&gt; blog.getBlogId() == request.getBlogId())   // ①
        .findAny()                                                     // ②
        .map(blog -&gt; {                                                  // ③
            blog.setTitle(request.getTitle());
            blog.setContent(request.getContent());
            return 1;
        })
        .orElse(0);                                                     // ④
}</code></pre>
<p>이 코드는 처음엔 평범한 <code>for</code>문으로 짰다가(리스트를 순회하면서 id가 같은 걸 찾으면 값을 바꾸고 <code>return 1</code>, 못 찾으면 반복문 끝나고 <code>return 0</code>), 나중에 지금처럼 스트림 체이닝으로 다시 정리했다.</p>
<ul>
<li>① <code>filter(...)</code>: 수정하려는 <code>blogId</code>와 일치하는 게시글만 걸러낸다.</li>
<li>② <code>findAny()</code>: 그중 하나를 <code>Optional&lt;BlogResponseDTO&gt;</code>로 꺼낸다 — 있으면 값이 담긴 <code>Optional</code>, 없으면 빈 <code>Optional</code>.</li>
<li>③ <code>.map(blog -&gt; {...})</code>: <code>Optional</code> 안에 값이 있을 때만 실행된다. 여기서 흥미로운 점은, <code>map</code>이 원래 &quot;다른 값으로 변환&quot;하는 용도인데, 여기서는 <strong><code>blog</code>의 필드를 직접 바꿔치기(부수효과)하면서 동시에 <code>1</code>이라는 값을 반환</strong>하는 데 쓰였다는 것이다. <code>blog</code>가 참조타입이라 이 안에서 값을 바꾸면 원본 리스트 안의 객체도 그대로 바뀐다.</li>
<li>④ <code>.orElse(0)</code>: <code>Optional</code>이 비어있었으면(=해당 id가 없었으면) <code>0</code>을 대신 반환한다.</li>
</ul>
<p>즉 &quot;찾아서 있으면 수정하고 1, 없으면 0&quot;이라는 로직을 <code>for</code>문 없이 스트림 하나로 표현한 것이다.</p>
<hr />
<h1 id="4-io-stream의-진화--한-글자-읽기부터-파일-입출력까지">4. IO Stream의 진화 — 한 글자 읽기부터 파일 입출력까지</h1>
<p>오늘 IO Stream을 단계적으로 발전시켜가며 배웠다.</p>
<p><strong>1단계: 가장 기본, 한 글자씩</strong></p>
<pre><code class="language-java">int input = System.in.read();
System.out.println((char) input);</code></pre>
<p><code>System.in.read()</code>는 콘솔에서 딱 <strong>한 글자(정수 코드값)</strong>만 읽는다. 문자열 전체를 받으려면 부족하다.</p>
<p><strong>2단계: <code>BufferedReader</code>로 한 줄 읽기 — 하지만 자원 정리가 번거로움</strong></p>
<pre><code class="language-java">BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
try {
    String input = br.readLine();
} catch (IOException e) {
    e.printStackTrace();
} finally {
    try {
        if (br != null) { br.close(); }
    } catch (Exception e) {
        e.printStackTrace();
    }
}</code></pre>
<p><code>BufferedReader</code>로 한 줄 전체를 받을 수 있게 됐지만, 다 쓴 뒤에는 <strong>직접 <code>close()</code>를 호출</strong>해서 자원을 정리해야 한다. 그런데 <code>close()</code> 자체도 예외를 던질 수 있어서, <code>finally</code> 안에 또 <code>try-catch</code>가 필요한 — 이중 예외처리라는 번거로운 상황이 생긴다.</p>
<p><strong>3단계: <code>try-with-resources</code>로 자동 정리</strong></p>
<pre><code class="language-java">try (BufferedReader br = new BufferedReader(new InputStreamReader(System.in))) {
    System.out.println(br.readLine());
} catch (Exception e) {
    e.printStackTrace();
}</code></pre>
<p><code>try(...)</code> 괄호 안에서 자원을 선언하면, <code>try</code> 블록이 끝나는 순간(정상 종료든 예외든) <strong>자동으로 <code>close()</code>가 호출된다.</strong> 2단계의 <code>finally</code> 블록이 통째로 사라졌다 — <code>AutoCloseable</code>을 구현한 클래스(<code>BufferedReader</code> 포함)라면 이 문법을 쓸 수 있다.</p>
<p><strong>4단계: 콘솔이 아니라 파일을 대상으로</strong></p>
<pre><code class="language-java">String path = &quot;./test.txt&quot;;
try (BufferedWriter bw = new BufferedWriter(new FileWriter(new File(path), StandardCharsets.UTF_8))) {
    bw.write(&quot;메시지\n&quot;);
} catch (Exception e) {
    e.printStackTrace();
}</code></pre>
<p><code>System.in</code>(콘솔) 대신 <code>FileWriter(new File(path), ...)</code>로 바꾸면, 같은 방식으로 <strong>파일에 쓰기</strong>가 된다. <code>StandardCharsets.UTF_8</code>을 명시한 건, 한글이 인코딩 문제로 깨지는 걸 방지하기 위해서다.</p>
<hr />
<h1 id="5-객체-직렬화serializable--리스트-전체를-파일-하나로">5. 객체 직렬화(Serializable) — 리스트 전체를 파일 하나로</h1>
<p><strong>Java</strong> · <code>BlogResponseDTO.java</code> — 직렬화 가능하다고 표시하기</p>
<pre><code class="language-java">public class BlogResponseDTO implements Serializable {   // ①
    private static final long serialVersionUID = 1L;      // ②
    ...
}</code></pre>
<ul>
<li>① <code>implements Serializable</code>: 이 클래스의 객체는 &quot;파일이나 네트워크로 통째로 보낼 수 있다&quot;고 표시한다. <code>Serializable</code>은 구현해야 할 메서드가 하나도 없는 <strong>마커 인터페이스</strong>라, <code>implements</code>만 붙이면 끝이다.</li>
<li>② <code>serialVersionUID</code>: 나중에 이 파일을 다시 읽을 때, &quot;지금 이 클래스&quot;와 &quot;파일에 저장됐을 때의 클래스&quot;가 같은 버전인지 확인하는 값. 명시하지 않으면 자바가 알아서 만들어주지만, 클래스 구조가 조금만 바뀌어도 값이 달라져서 예전 파일을 못 읽게 될 수 있어 직접 고정값을 주는 걸 권장한다고 배웠다.</li>
</ul>
<p><strong>Java</strong> · 리스트 전체를 파일에 저장(직렬화)</p>
<pre><code class="language-java">String path = &quot;./object.txt&quot;;
try (ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(new File(path)))) {
    oos.writeObject(blogs);                         // ①
    System.out.println(&quot;debug &gt;&gt;&gt;&gt; object.txt save ok &quot;);
} catch (Exception e) {
    e.printStackTrace();
}</code></pre>
<ul>
<li>① <code>oos.writeObject(blogs)</code>: <code>List&lt;BlogResponseDTO&gt;</code> 전체를 <strong>한 번에</strong> 파일에 써넣는다. 항목 하나하나를 텍스트로 변환해서 쓰는 게 아니라, 객체 구조 그대로를 바이트로 바꿔서 저장하는 것이다.</li>
</ul>
<p><strong>Java</strong> · 파일에서 다시 읽어오기(역직렬화)</p>
<pre><code class="language-java">try (ObjectInputStream ois = new ObjectInputStream(new FileInputStream(new File(path)))) {
    List&lt;BlogResponseDTO&gt; list = (List&lt;BlogResponseDTO&gt;) ois.readObject();   // ①
    list.forEach(System.out::println);
} catch (Exception e) {
    e.printStackTrace();
}</code></pre>
<ul>
<li>① <code>ois.readObject()</code>는 반환 타입이 <code>Object</code>라서, 원래 타입(<code>List&lt;BlogResponseDTO&gt;</code>)으로 다시 캐스팅해줘야 한다. 저장할 때 리스트를 통째로 넣었기 때문에, 꺼낼 때도 리스트 하나로 통째로 나온다.</li>
</ul>
<hr />
<h1 id="6-프로그램-시작·종료에-저장불러오기-연결하기">6. 프로그램 시작·종료에 저장/불러오기 연결하기</h1>
<p>지금까지 만든 직렬화 코드를, 실제 블로그 앱의 시작/종료 시점에 연결했다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/hssondev/post/29bf155d-afb8-4fb3-ba43-23a2e36aaa4b/image.png" /></p>
<p><strong>Java</strong> · <code>BlogReactView.java</code> — 시작할 때 불러오기</p>
<pre><code class="language-java">public void landingPage() {
    boolean isLoad = front.file(&quot;file.inspire&quot;, &quot;load&quot;);
    System.out.println(isLoad ? &quot;데이터 로딩완료&quot; : &quot;데이터 로딩실패&quot;);
    while (true) {
        ...
    }
}</code></pre>
<p><strong>Java</strong> · <code>BlogReactView.java</code> — 종료할 때 저장 여부 확인 후 저장</p>
<pre><code class="language-java">public void exit() {
    System.out.println(&quot;&gt;&gt;&gt;&gt; 작업된 내용을 저장하고 시스템을 종료하겠습니까?(y/n).&quot;);
    String yesOrNo = scan.nextLine();
    if (yesOrNo.equalsIgnoreCase(&quot;y&quot;)) {
        String endPoint = &quot;file.inspire&quot;;
        System.out.println(front.file(endPoint, &quot;save&quot;) ? &quot;데이터 저장완료&quot; : &quot;데이터 저장실패&quot;);
    }
    System.exit(1);
}</code></pre>
<p><code>file.inspire</code>라는 엔드포인트 하나를 <code>FileController</code>에 등록해두고, <code>&quot;load&quot;</code>/<code>&quot;save&quot;</code>라는 문자열로 동작을 구분한다 — 지금까지 써온 파사드+팩토리 패턴에 새 기능(파일 저장)을 끼워 넣을 때도 똑같은 방식(엔드포인트 등록)을 그대로 재사용한 것이다. <code>BlogReactDao</code>에는 <code>setBlogs(List&lt;BlogResponseDTO&gt; blogs)</code>를 새로 추가해서, 파일에서 읽어온 리스트를 <code>dao</code> 내부의 <code>blogs</code>와 통째로 교체할 수 있게 했다.</p>
<hr />
<h1 id="7-responseentityt--응답을-상태코드메시지데이터로-감싸기">7. <code>ResponseEntity&lt;T&gt;</code> — 응답을 상태코드+메시지+데이터로 감싸기</h1>
<pre><code class="language-java">public class ResponseEntity&lt;T&gt; {          // ①
    private int code;
    private String message;
    private T data;                        // ②

    public ResponseEntity(int code, String message, T data) {
        this.code = code;
        this.message = message;
        this.data = data;
    }
    // getter/setter 생략
}</code></pre>
<ul>
<li>① <code>&lt;T&gt;</code>: 이 클래스는 어떤 타입의 데이터든 담을 수 있도록 <strong>타입을 나중에 정하게(제네릭)</strong> 열어뒀다. <code>ResponseEntity&lt;List&lt;BlogResponseDTO&gt;&gt;</code>처럼 실제로 쓸 때 타입을 채워 넣는다.</li>
<li>② <code>data</code> 필드가 바로 그 <code>T</code> 타입 — 진짜 응답 데이터가 여기 담긴다.</li>
</ul>
<pre><code class="language-java">// ListController.java
public ResponseEntity&lt;List&lt;BlogResponseDTO&gt;&gt; list() {
    return new ResponseEntity&lt;&gt;(200, &quot;ok&quot;, service.list());
}</code></pre>
<pre><code class="language-java">// BlogReactView.java
ResponseEntity&lt;List&lt;BlogResponseDTO&gt;&gt; response = front.list(endPoint);
if (response.getCode() == 200) {
    response.getData().stream().forEach(System.out::println);
}</code></pre>
<p><strong>왜 데이터를 그냥 반환하지 않고 감싸는가</strong>: 지금까지는 <code>list()</code>가 <code>List&lt;BlogResponseDTO&gt;</code>를 바로 반환했는데, 이렇게 되면 &quot;성공했는지 실패했는지&quot;, &quot;왜 실패했는지&quot; 같은 부가 정보를 같이 전달할 방법이 없다. <code>code</code>(상태코드)와 <code>message</code>를 데이터와 함께 묶어서 반환하면, 받는 쪽(<code>View</code>)이 <code>response.getCode()</code>만 보고도 성공 여부를 먼저 판단할 수 있다 — 실제 웹 API 응답이 상태코드(200, 404 등)와 body를 함께 갖는 것과 같은 개념이라고 이해했다.</p>
<hr />
<h1 id="✨-새롭게-알게-된-점">✨ 새롭게 알게 된 점</h1>
<ul>
<li><code>blogs.stream().filter().findAny().map(blog -&gt; {...}).orElse(0)</code>처럼, <code>map</code>을 값 변환이 아니라 &quot;값이 있을 때만 실행할 부수효과 + 반환값&quot;으로 쓸 수도 있다.</li>
<li><code>try-with-resources</code>는 <code>AutoCloseable</code>을 구현한 자원이라면 <code>finally</code>에서 직접 <code>close()</code>를 호출하지 않아도 자동으로 정리해준다.</li>
<li><code>Serializable</code>은 메서드가 없는 마커 인터페이스이고, 붙이기만 하면 그 클래스의 객체를 통째로 파일에 저장하고 복원할 수 있다.</li>
<li><code>ResponseEntity&lt;T&gt;</code>처럼 제네릭으로 감싸면, 데이터가 뭐든 상태코드·메시지라는 공통 형식으로 응답을 통일할 수 있다.</li>
</ul>
<hr />
<h1 id="🤔-헷갈렸던-점">🤔 헷갈렸던 점</h1>
<ul>
<li><strong><code>map</code>을 부수효과 용도로 쓰는 것</strong>: <code>map</code>은 &quot;타입 변환&quot;으로만 배웠는데, <code>update()</code>에서 필드를 직접 바꾸고 <code>1</code>을 반환하는 용도로 쓰인 걸 보고 처음엔 낯설었다. <code>blog</code>가 참조타입이라 안에서 값을 바꾸면 원본에도 반영된다는 걸 다시 떠올리고서야 이해가 됐다.</li>
<li><strong><code>serialVersionUID</code>를 왜 직접 지정해야 하는지</strong>: 자동 생성에 맡기면 왜 위험한지는 알겠는데, 실제로 클래스가 어떻게 바뀌었을 때 문제가 되는지는 아직 직접 겪어보지 못했다.</li>
<li><strong><code>ObjectInputStream.readObject()</code>가 <code>Object</code>를 반환하는 이유</strong>: 파일 안에 어떤 타입이 저장되어 있는지 읽기 전엔 알 수 없어서 일단 가장 넓은 타입으로 주고, 쓰는 쪽에서 캐스팅으로 원래 타입을 알려줘야 한다는 점이 <code>BlogBeanFactory.getBean()</code>의 <code>Object</code> 반환과 같은 패턴이라는 걸 뒤늦게 연결지었다.</li>
</ul>
<hr />
<h1 id="다음에-할-일">다음에 할 일</h1>
<ul>
<li><input disabled="" type="checkbox" /> <code>FileController.java</code>에 남아있는 문법 오류(불필요한 <code>}</code> 하나 더 있음) 확인하고 정리</li>
<li><input disabled="" type="checkbox" /> <code>serialVersionUID</code>가 실제로 버전 불일치를 막아주는 상황을 직접 재현해보기</li>
<li><input disabled="" type="checkbox" /> ORM 개념(오늘 커리큘럼에 있었지만 다루지 못함) 다음에 이어서 정리하기</li>
<li><input disabled="" type="checkbox" /> 정규표현식(오늘 커리큘럼에 있었지만 다루지 못함) 다음에 이어서 정리하기</li>
</ul>
<hr />
<h1 id="💡-오늘의-회고">💡 오늘의 회고</h1>
<p>CRUD의 마지막 조각(update, delete)을 붙이면서, 새 기능을 추가하는 네 단계(FrontController → Controller → Factory 등록 → Service/Dao)가 이제는 거의 자동으로 손이 움직일 정도로 익숙해졌다는 걸 느꼈다.</p>
<p>IO Stream을 <code>System.in.read()</code>부터 <code>try-with-resources</code>, 파일 입출력까지 단계적으로 발전시켜본 것도 좋았다 — 왜 매번 더 편한 방법이 나오는지, 이전 단계에서 뭐가 불편했는지를 순서대로 겪고 나니 각 문법이 왜 존재하는지 이해가 됐다. 특히 <code>Serializable</code> 하나만 붙이면 리스트 전체를 파일로 저장하고 복원할 수 있다는 게 신기했다 — 지금까지는 프로그램을 끄면 입력한 데이터가 다 사라졌는데, 오늘부터는 <code>blogs.txt</code>에 남아서 다음에 켜도 이어진다.</p>
<p><code>ResponseEntity&lt;T&gt;</code>를 붙이면서, 지금까지 그냥 데이터만 주고받던 흐름에 &quot;성공/실패&quot;라는 의미가 하나 더 얹어졌다는 것도 인상 깊었다. 다음에 배울 ORM과 정규표현식은 오늘 시간이 부족해서 못 다뤘는데, 이어서 정리해야겠다.</p>