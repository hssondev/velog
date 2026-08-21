<blockquote>
<p><strong>한 줄 요약</strong></p>
<p>블로그 목록(<code>List&lt;BlogResponseDTO&gt;</code>)을 가지고, filter(거르기)·map(변환)·collect(모으기)를 기본으로, groupingBy(그룹화)·평균·중복제거·정렬·존재판단·Optional(NPE 회피)까지 Stream API 한 바퀴를 실습했다.</p>
</blockquote>
<hr />
<h1 id="📚-오늘-학습한-커리큘럼">📚 오늘 학습한 커리큘럼</h1>
<pre><code>filter — 조건에 맞는 것만 거르기
        ↓
map — 다른 타입으로 변환하기
        ↓
collect / toList — 최종 리스트로 모으기
        ↓
Collectors.groupingBy — 기준별로 묶기
        ↓
mapToInt + average — 숫자만 뽑아 평균내기
        ↓
distinct / sorted — 중복 제거, 정렬
        ↓
anyMatch / allMatch / noneMatch — 존재 여부, 조건 검증
        ↓
Optional — null 안전하게 다루기</code></pre><h1 id="📌-오늘의-학습-키워드">📌 오늘의 학습 키워드</h1>
<ul>
<li>Stream, 중간 연산(filter, map) vs 최종 연산(collect, forEach)</li>
<li><code>Collectors.groupingBy</code>, <code>mapToInt</code>, <code>average</code></li>
<li><code>distinct</code>, <code>sorted</code>, <code>Comparator.reversed</code></li>
<li><code>anyMatch</code>, <code>allMatch</code>, <code>noneMatch</code></li>
<li><code>Optional</code>, <code>ifPresentOrElse</code>, <code>orElseThrow</code></li>
<li>메서드 참조(<code>::</code>)</li>
</ul>
<hr />
<h1 id="0-개념-정리-키워드-사전">0. 개념 정리 (키워드 사전)</h1>
<blockquote>
<p><strong>한 줄 요약</strong></p>
<p>Stream은 리스트를 &quot;가공하는 컨베이어 벨트&quot;이고, filter·map처럼 벨트 중간에서 다듬는 작업(중간 연산)은 collect·forEach 같은 마지막 작업(최종 연산)이 나와야 비로소 실제로 돌아간다.</p>
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
<td><strong>Stream</strong></td>
<td>리스트 같은 데이터 묶음을 한 번에 처리(필터링, 변환, 집계 등)하기 위한 통로</td>
<td><code>blogs.stream()</code>으로 시작</td>
</tr>
<tr>
<td><strong>중간 연산</strong></td>
<td>스트림을 받아서 다시 스트림을 돌려주는 연산, <strong>여러 번 이어붙일 수 있음</strong></td>
<td><code>filter</code>, <code>map</code>, <code>sorted</code>, <code>distinct</code></td>
</tr>
<tr>
<td><strong>최종 연산</strong></td>
<td>스트림을 실제로 실행시켜서 결과(리스트, 값 등)를 내놓는 연산, <strong>한 번만 쓸 수 있음</strong></td>
<td><code>collect</code>, <code>toList</code>, <code>forEach</code>, <code>average</code></td>
</tr>
<tr>
<td><strong>filter</strong></td>
<td>조건(<code>boolean</code>을 반환하는 함수)에 맞는 요소만 통과시킴</td>
<td><code>filter(blog -&gt; blog.getViewCnt() &gt;= 30)</code></td>
</tr>
<tr>
<td><strong>map</strong></td>
<td>각 요소를 다른 값/타입으로 하나씩 바꿈</td>
<td><code>map(blog -&gt; blog.getEmail())</code> — DTO를 이메일 문자열로</td>
</tr>
<tr>
<td><strong>collect / toList</strong></td>
<td>스트림에 남은 요소들을 진짜 <code>List</code>로 모아줌</td>
<td>최종 연산</td>
</tr>
<tr>
<td><strong>Collectors.groupingBy</strong></td>
<td>지정한 기준(키)으로 요소들을 묶어서 <code>Map&lt;키, List&lt;값&gt;&gt;</code>을 만듦</td>
<td>이메일별로 게시글을 묶기</td>
</tr>
<tr>
<td><strong>distinct</strong></td>
<td>중복된 요소를 제거하고 한 번씩만 남김</td>
<td></td>
</tr>
<tr>
<td><strong>sorted / Comparator</strong></td>
<td>기준에 따라 정렬</td>
<td><code>.reversed()</code>를 붙이면 역순</td>
</tr>
<tr>
<td><strong>anyMatch / allMatch / noneMatch</strong></td>
<td>각각 &quot;하나라도 맞나?&quot; / &quot;전부 맞나?&quot; / &quot;하나도 안 맞나?&quot;를 <code>true</code>/<code>false</code>로 반환</td>
<td></td>
</tr>
<tr>
<td><strong>Optional</strong></td>
<td>값이 있을 수도, 없을 수도 있는 상황을 감싸는 상자. <code>null</code>을 직접 다루지 않게 해줌</td>
<td>반환타입으로만 사용</td>
</tr>
<tr>
<td><strong>메서드 참조 (<code>::</code>)</strong></td>
<td><code>blog -&gt; blog.getEmail()</code>처럼 인자 하나를 그대로 어떤 메서드에 넘기기만 할 때, 그 메서드 이름만 적는 축약 문법</td>
<td><code>BlogResponseDTO::getEmail</code></td>
</tr>
</tbody></table>
<h3 id="코드로-다시-보기">코드로 다시 보기</h3>
<pre><code class="language-java">blogs.stream()               // ① 리스트를 스트림으로 바꿈 (컨베이어 벨트 시작)
    .filter(blog -&gt; ...)     // ② 중간 연산 — 조건에 맞는 것만 통과
    .map(blog -&gt; ...)        // ③ 중간 연산 — 형태를 바꿈
    .toList();                // ④ 최종 연산 — 여기서 비로소 ①~③이 실제로 실행되고 List가 만들어짐</code></pre>
<hr />
<h1 id="1-filter--조건에-맞는-것만-걸러내기">1. <code>filter</code> — 조건에 맞는 것만 걸러내기</h1>
<pre><code class="language-java">blogs.stream()
    .filter(blog -&gt; blog.getViewCnt() &gt;= 30)   // ①
    .forEach(blog -&gt; System.out.println(blog));  // ②</code></pre>
<ul>
<li>① <code>filter(blog -&gt; blog.getViewCnt() &gt;= 30)</code>: 스트림에 있는 <code>blog</code> 하나하나에 대해, <code>blog.getViewCnt() &gt;= 30</code>(조회수 30 이상)이 <code>true</code>인 것만 다음 단계로 통과시킨다. <code>blog -&gt; ...</code> 부분은 &quot;이 blog를 받아서 이 조건을 계산해라&quot;는 뜻의 람다식이다.</li>
<li>② <code>forEach(...)</code>: 통과한 요소들을 하나씩 꺼내서 괄호 안의 동작(여기선 출력)을 실행하는 <strong>최종 연산</strong>이다. <code>forEach</code>가 실행되는 순간 비로소 ①의 <code>filter</code>가 진짜로 동작한다.</li>
</ul>
<hr />
<h1 id="2-map--타입을-다른-형태로-바꾸기">2. <code>map</code> — 타입을 다른 형태로 바꾸기</h1>
<pre><code class="language-java">blogs.stream()
    .filter(blog -&gt; blog.getViewCnt() &gt;= 30)
    .map(blog -&gt; blog.getEmail())      // ①
    .forEach(System.out::println);       // ②</code></pre>
<ul>
<li>① <code>map(blog -&gt; blog.getEmail())</code>: <code>filter</code>를 통과한 <code>BlogResponseDTO</code> 객체 하나하나를, 그 안의 <code>email</code> 값(문자열)으로 <strong>바꿔치기</strong>한다. 그래서 이 줄 이후로는 스트림 안에 더 이상 <code>BlogResponseDTO</code>가 아니라 <code>String</code>(이메일)만 남는다.</li>
<li>② <code>System.out::println</code>: <code>blog -&gt; System.out.println(blog)</code>와 완전히 같은 뜻인데, &quot;인자를 받아서 그대로 어떤 메서드에 넘기기만 한다&quot;는 패턴일 때는 이렇게 <strong>메서드 참조</strong>로 줄여 쓸 수 있다.</li>
</ul>
<hr />
<h1 id="3-collect--tolist--최종-리스트로-모으기">3. <code>collect</code> / <code>toList</code> — 최종 리스트로 모으기</h1>
<pre><code class="language-java">List&lt;BlogResponseDTO&gt; result = blogs.stream()
    .filter(blog -&gt; blog.getEmail().equals(&quot;lim&quot;))   // ①
    .toList();                                          // ②

result.forEach(System.out::println);</code></pre>
<ul>
<li>① 이메일이 <code>&quot;lim&quot;</code>인 게시글만 걸러낸다. (문자열 비교라 <code>.equals()</code>를 쓴다 — 예전에 정리했던 <code>==</code> 대신 <code>.equals()</code> 원칙이 여기서도 그대로.)</li>
<li>② <code>.toList()</code>: 지금까지 걸러진 스트림을 진짜 <code>List&lt;BlogResponseDTO&gt;</code>로 만들어서 <code>result</code>에 담는다. 주석 처리된 <code>.collect(Collectors.toList())</code>도 같은 역할을 하는 예전 방식이고, <code>.toList()</code>는 자바 16부터 추가된 더 짧은 표현이다.</li>
</ul>
<hr />
<h1 id="4-collectorsgroupingby--기준별로-묶기">4. <code>Collectors.groupingBy</code> — 기준별로 묶기</h1>
<pre><code class="language-java">Map&lt;String, List&lt;BlogResponseDTO&gt;&gt; map = blogs.stream()
    .collect(Collectors.groupingBy(BlogResponseDTO::getEmail));   // ①

map.get(&quot;park&quot;).stream()
    .forEach(System.out::println);   // ②</code></pre>
<p><img alt="Stream 파이프라인 (filter → map → collect)" src="https://api.velog.io/rss/stream_pipeline.png" /></p>
<ul>
<li>① <code>Collectors.groupingBy(BlogResponseDTO::getEmail)</code>: 스트림의 각 요소에서 <code>getEmail()</code> 값을 꺼내, <strong>같은 이메일끼리 묶어서</strong> <code>Map&lt;String, List&lt;BlogResponseDTO&gt;&gt;</code>를 만든다. 예를 들어 <code>&quot;lim&quot;</code>이라는 키에는 이메일이 <code>lim</code>인 모든 게시글이 리스트로 담긴다.</li>
<li>② <code>map.get(&quot;park&quot;)</code>로 <code>&quot;park&quot;</code> 키에 묶인 리스트만 꺼내서, 그 리스트를 다시 스트림으로 돌려 출력한다.</li>
</ul>
<hr />
<h1 id="5-maptoint--average--숫자만-뽑아서-평균-내기">5. <code>mapToInt</code> + <code>average</code> — 숫자만 뽑아서 평균 내기</h1>
<pre><code class="language-java">double avg = blogs.stream()
    .mapToInt(BlogResponseDTO::getViewCnt)   // ①
    .average()                                  // ②
    .orElse(0);                                  // ③
System.out.println(avg);</code></pre>
<ul>
<li>① <code>mapToInt(BlogResponseDTO::getViewCnt)</code>: 일반 <code>map</code>과 비슷하지만, 결과를 <code>int</code> 전용 스트림(<code>IntStream</code>)으로 바꿔준다. 평균·합계 같은 숫자 계산을 하려면 이렇게 숫자 전용 스트림으로 바꿔야 한다.</li>
<li>② <code>.average()</code>: <code>IntStream</code>에서 제공하는 메서드로, 평균값을 계산한다. 그런데 스트림이 비어있으면 평균을 낼 수 없으므로, 결과 타입이 <code>double</code>이 아니라 <code>OptionalDouble</code>(있을 수도 없을 수도 있는 값)이다.</li>
<li>③ <code>.orElse(0)</code>: <code>OptionalDouble</code> 안에 값이 있으면 그 값을, 없으면 <code>0</code>을 대신 돌려준다 — 아래 9번에서 다룰 <code>Optional</code>과 같은 원리다.</li>
</ul>
<hr />
<h1 id="6-distinct--중복-제거">6. <code>distinct</code> — 중복 제거</h1>
<pre><code class="language-java">blogs.stream()
    .map(BlogResponseDTO::getEmail)   // ①
    .distinct()                        // ②
    .forEach(System.out::println);</code></pre>
<ul>
<li>① 모든 게시글에서 이메일만 뽑는다 — 5건이면 이메일도 5개(중복 포함)가 나온다.</li>
<li>② <code>.distinct()</code>: 그중 <strong>중복된 값을 제거</strong>하고 한 번씩만 남긴다. <code>&quot;lim&quot;</code>이 두 번 나왔다면 한 번만 남는 식.</li>
</ul>
<hr />
<h1 id="7-sorted--comparatorreversed--정렬">7. <code>sorted</code> + <code>Comparator.reversed</code> — 정렬</h1>
<pre><code class="language-java">blogs.stream()
    .sorted(Comparator.comparing(BlogResponseDTO::getViewCnt).reversed())   // ①
    .forEach(System.out::println);</code></pre>
<ul>
<li>① <code>Comparator.comparing(BlogResponseDTO::getViewCnt)</code>: &quot;조회수(<code>getViewCnt</code>) 기준으로 비교해서 정렬하라&quot;는 비교 규칙을 만든다. 기본은 오름차순(작은 값 → 큰 값)인데, <code>.reversed()</code>를 붙이면 <strong>내림차순</strong>(큰 값 → 작은 값)으로 뒤집힌다.</li>
</ul>
<hr />
<h1 id="8-anymatch--allmatch--nonematch--존재-여부와-조건-검증">8. <code>anyMatch</code> / <code>allMatch</code> / <code>noneMatch</code> — 존재 여부와 조건 검증</h1>
<pre><code class="language-java">boolean isEmail = blogs.stream()
    .anyMatch(blog -&gt; blog.getEmail().equals(&quot;jslim&quot;));   // ①

boolean isExists = blogs.stream()
    .allMatch(blog -&gt; blog.getViewCnt() &gt;= 20);              // ②

boolean isUnder = blogs.stream()
    .noneMatch(blog -&gt; blog.getViewCnt() &lt; 10);               // ③</code></pre>
<ul>
<li>① <code>anyMatch(...)</code>: 조건에 맞는 요소가 <strong>하나라도</strong> 있으면 <code>true</code>. 여기선 이메일이 <code>&quot;jslim&quot;</code>인 게시글이 하나라도 있는지 확인 — 데이터에 없으니 <code>false</code>가 나온다.</li>
<li>② <code>allMatch(...)</code>: <strong>전부 다</strong> 조건을 만족해야 <code>true</code>. 모든 게시글의 조회수가 20 이상인지 확인.</li>
<li>③ <code>noneMatch(...)</code>: 조건에 맞는 게 <strong>하나도 없어야</strong> <code>true</code>. 조회수가 10 미만인 게시글이 하나도 없는지 확인.</li>
</ul>
<hr />
<h1 id="9-optional--null을-안전하게-다루기">9. <code>Optional</code> — null을 안전하게 다루기</h1>
<pre><code class="language-java">Optional&lt;String&gt; optional = Optional.of(&quot;lgcns&quot;);   // ①

optional.ifPresentOrElse(                              // ②
    value -&gt; System.out.println(value),
    () -&gt; System.out.println(&quot;값이 없습니다.&quot;)
);

optional = Optional.empty();                            // ③
String err = optional.orElseThrow(                       // ④
    () -&gt; new RuntimeException(&quot;값이 없습니다.&quot;)
);
System.out.println(err);</code></pre>
<ul>
<li>① <code>Optional.of(&quot;lgcns&quot;)</code>: <code>&quot;lgcns&quot;</code>라는 값을 <code>Optional</code>이라는 <strong>상자</strong>에 담는다. <code>Optional</code>은 &quot;이 안에 값이 있을 수도, 없을 수도 있다&quot;는 걸 타입 자체로 표현하는 방법이다.</li>
<li>② <code>ifPresentOrElse(값이 있을 때 할 일, 값이 없을 때 할 일)</code>: 상자 안에 값이 있으면 첫 번째 람다를, 없으면 두 번째 람다를 실행한다. <code>if (optional.isPresent()) {...} else {...}</code>를 한 줄로 줄인 것과 같다.</li>
<li>③ <code>Optional.empty()</code>: 이번엔 값이 <strong>없는</strong> 빈 상자를 만든다.</li>
<li>④ <code>orElseThrow(() -&gt; new RuntimeException(...))</code>: 상자 안에 값이 있으면 그 값을 꺼내주고, 없으면 지정한 예외를 직접 발생시킨다. 이 코드는 상자가 비어있으니 <code>RuntimeException</code>이 실제로 발생하고, <code>System.out.println(err)</code>는 실행되지 않는다.</li>
</ul>
<p><strong><code>Optional</code>을 쓰는 이유</strong>: <code>null</code>을 직접 반환하고 호출하는 쪽에서 매번 <code>if (result != null)</code>으로 검사하는 대신, &quot;값이 없을 수도 있다&quot;는 걸 반환 타입 자체(<code>Optional&lt;String&gt;</code>)로 미리 알려줘서, 실수로 <code>null.method()</code>를 호출해 <code>NullPointerException</code>이 나는 상황을 컴파일 단계에서부터 줄여준다.</p>
<hr />
<h1 id="✨-새롭게-알게-된-점">✨ 새롭게 알게 된 점</h1>
<ul>
<li><code>filter</code>, <code>map</code> 같은 중간 연산은 그 자체로는 아무 일도 안 하고, <code>collect</code>/<code>forEach</code> 같은 최종 연산이 나와야 한 번에 실행된다.</li>
<li>숫자 집계(평균 등)를 하려면 <code>mapToInt</code>처럼 숫자 전용 스트림으로 바꿔야 한다.</li>
<li><code>Comparator.comparing(...).reversed()</code>로 정렬 기준과 방향을 조합할 수 있다.</li>
<li><code>anyMatch</code>/<code>allMatch</code>/<code>noneMatch</code>는 이름 그대로 &quot;하나라도/전부/하나도&quot;로 구분해서 기억하면 헷갈리지 않는다.</li>
<li><code>Optional</code>은 <code>null</code> 대신 &quot;값이 있을 수도 없을 수도 있음&quot;을 타입으로 표현하는 방법이고, 메서드의 반환타입으로만 쓰는 게 정석이다.</li>
</ul>
<hr />
<h1 id="🤔-헷갈렸던-점">🤔 헷갈렸던 점</h1>
<ul>
<li><strong>중간 연산과 최종 연산의 구분</strong>: <code>filter</code>나 <code>map</code>만 써놓고 아무 최종 연산이 없으면 실제로 아무것도 실행이 안 된다는 걸 처음엔 놓치기 쉬웠다.</li>
<li><strong><code>mapToInt</code>가 왜 따로 있는지</strong>: 그냥 <code>map</code>으로는 왜 평균을 못 내는지, 숫자 전용 스트림(<code>IntStream</code>)이 따로 있다는 걸 알고 나서야 이해가 됐다.</li>
<li><strong><code>Optional</code>을 언제 써야 하는지</strong>: &quot;메서드 반환타입으로만 사용, 필드나 매개변수로는 쓰지 말 것&quot;이라는 원칙이 왜 있는지는 아직 감이 완전히 잡히진 않았다.</li>
</ul>
<hr />
<h1 id="다음에-할-일">다음에 할 일</h1>
<ul>
<li><input disabled="" type="checkbox" /> <code>Optional</code>을 필드나 매개변수로 쓰면 안 되는 이유 더 찾아보기</li>
<li><input disabled="" type="checkbox" /> <code>Collectors.groupingBy</code>에 두 번째 인자(다운스트림 collector)를 넣는 형태도 연습해보기</li>
<li><input disabled="" type="checkbox" /> 오늘 배운 것들을 <code>BlogReactDao</code>의 검색/필터 기능에 실제로 적용해보기</li>
</ul>
<hr />
<h1 id="💡-오늘의-회고">💡 오늘의 회고</h1>
<p>Stream API는 하나하나는 짧은 문법인데, 이어붙이면 이어붙일수록 &quot;지금 이 시점에 스트림 안에 뭐가 들어있는지&quot;를 놓치기 쉬웠다. <code>filter</code>를 거치면 원래 타입(<code>BlogResponseDTO</code>) 그대로 개수만 줄고, <code>map</code>을 거치면 아예 타입 자체가 바뀐다는 걸 한 줄씩 짚어보고 나서야 코드가 눈에 들어오기 시작했다.</p>
<p><code>Optional</code>은 처음엔 그냥 <code>if-null</code> 체크를 복잡하게 만든 것처럼 보였는데, &quot;값이 없을 수도 있다는 걸 타입으로 미리 알려준다&quot;는 관점으로 보니 왜 필요한지 이해가 됐다. 내일은 오늘 배운 연산들을 실제 <code>BlogReactDao</code>의 검색 기능에 적용해보면서 손에 익혀야겠다.</p>