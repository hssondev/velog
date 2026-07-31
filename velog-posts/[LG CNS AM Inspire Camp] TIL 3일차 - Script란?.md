<h2 id="1-오늘의-한-줄-요약-✏️">1. 오늘의 한 줄 요약 ✏️</h2>
<p>Script의 의미와 구성 요소, DOM/BOM 개념, 화살표 함수, 이벤트 처리 방식을 알아보는 시간</p>
<hr />
<h2 id="2-배운-내용">2. 배운 내용</h2>
<h3 id="dom-vs-bom">DOM vs BOM</h3>
<p>브라우저가 다루는 두 가지 객체 모델을 배웠다.</p>
<table>
<thead>
<tr>
<th>구분</th>
<th>정의</th>
<th>내장 객체</th>
</tr>
</thead>
<tbody><tr>
<td><strong>DOM</strong> (Document Object Model)</td>
<td>HTML 문서를 <strong>트리 구조의 객체</strong>로 만든 것. 문서의 <strong>내용을 제어</strong>할 때 사용</td>
<td><code>document</code> — 요소(Node) 탐색/조작, 이벤트 처리</td>
</tr>
<tr>
<td><strong>BOM</strong> (Browser Object Model)</td>
<td><strong>브라우저 자체</strong>를 제어하기 위한 객체 모델</td>
<td><code>window</code>, <code>location</code>, <code>navigator</code>, <code>history</code>, <code>screen</code></td>
</tr>
</tbody></table>
<p><strong>핵심 차이</strong>: DOM은 &quot;화면에 그려진 HTML 내용&quot;을 다루고, BOM은 &quot;브라우저 창 자체&quot;(주소, 뒤로가기, 화면 크기 등)를 다룬다. 실습 코드에서 쓴 <code>window.alert()</code>, <code>window.prompt()</code>, <code>location.href</code>가 전부 BOM에 속하는 기능이다.</p>
<h3 id="화살표-함수-arrow-function-es6">화살표 함수 (Arrow Function, ES6)</h3>
<p>기존 <code>function</code> 키워드 대신 <code>=&gt;</code>를 이용해 함수를 간결하게 쓰는 문법.</p>
<pre><code class="language-javascript">// 기본형
let arrowFunc = () =&gt; {
    console.log(`화살표 함수 호출.`);
};
arrowFunc();

// 함수 본문이 한 줄이고 값을 바로 반환할 때는 {}와 return 생략 가능
let add = (a, b) =&gt; a + b;
console.log(`debug &gt;&gt;&gt;&gt;&gt; add function ${add(10, 30)}`);</code></pre>
<p><strong><code>{}</code> 유무에 따른 차이</strong></p>
<table>
<thead>
<tr>
<th>형태</th>
<th>의미</th>
</tr>
</thead>
<tbody><tr>
<td><code>(a, b) =&gt; a + b</code></td>
<td><code>{}</code>, <code>return</code> 생략 — 계산 결과를 <strong>바로 반환</strong></td>
</tr>
<tr>
<td><code>(a, b) =&gt; { return a + b; }</code></td>
<td><code>{}</code> 사용 — 여러 줄의 코드를 실행할 때, <code>return</code>을 <strong>명시적으로</strong> 써야 값이 반환됨</td>
</tr>
</tbody></table>
<p>즉 <code>{}</code>를 쓰는 순간부터는 &quot;이건 그냥 코드 블록이니, 값을 반환하려면 <code>return</code>을 직접 써라&quot;는 의미가 된다.</p>
<h3 id="이벤트란">이벤트란?</h3>
<p>사용자의 행동(클릭, 입력, 스크롤 등)에 반응해서 특정 코드를 실행하도록 등록하는 것. 오늘은 이벤트를 등록하는 두 가지 방식을 비교했다.</p>
<pre><code class="language-javascript">// 방식 1 - 프로퍼티 방식 (예전 방식)
document.querySelector(&quot;#handler&quot;).onclick = () =&gt; {
    window.alert(`signIn button click`);
};

// 방식 2 - addEventListener (표준 방식)
document.querySelector(&quot;#handler&quot;).addEventListener('click', () =&gt; {
    window.alert(`signIn button click`);
});</code></pre>
<p><strong><code>onclick</code> vs <code>addEventListener</code> 차이</strong></p>
<table>
<thead>
<tr>
<th>구분</th>
<th><code>onclick</code></th>
<th><code>addEventListener</code></th>
</tr>
</thead>
<tbody><tr>
<td>이벤트 등록 개수</td>
<td>하나만 등록 가능 (덮어씀)</td>
<td>같은 이벤트에 <strong>여러 개</strong> 등록 가능</td>
</tr>
<tr>
<td>표준 여부</td>
<td>예전 방식</td>
<td>현재 <strong>표준(권장)</strong> 방식</td>
</tr>
</tbody></table>
<p>여러 개의 클릭 이벤트를 동시에 등록하고 싶을 때는 <code>addEventListener</code>가 아니면 불가능하기 때문에, 지금은 이 방식이 기본이다.</p>
<hr />
<h2 id="3-헷갈렸던-점-🤔">3. 헷갈렸던 점 🤔</h2>
<ul>
<li><p><strong><code>{}</code> 안에서 배열/객체 값을 참조할 때 백틱(<code>`</code>)과 홑따옴표(<code>'</code>)를 헷갈림</strong>: 템플릿 리터럴 문법(<code>${}</code>)은 반드시 백틱 안에서만 동작하는데, 홑따옴표 안에 <code>${}</code>를 쓰면 그냥 문자 그대로 출력되어버려서 처음에 왜 값이 안 나오는지 헷갈렸음 (실습 코드의 <code>forEach</code> 부분에서 발견)</p>
</li>
<li><p><strong>DOM과 BOM의 경계가 헷갈림</strong>: <code>document</code>는 DOM, <code>window</code>/<code>location</code>은 BOM인데 둘 다 브라우저에서 제공하는 전역 객체다 보니 처음엔 구분이 잘 안 됐음</p>
</li>
<li><p><strong>함수관련(6.함수(fuction 개념정리) 참고</strong></p>
</li>
</ul>
<hr />
<h2 id="4-실습--적용">4. 실습 / 적용</h2>
<h3 id="실습-1-배열-반복--dom에-카드-리스트-렌더링">실습 1) 배열 반복 &amp; DOM에 카드 리스트 렌더링</h3>
<p>세 가지 반복문(<code>for</code>, <code>forEach</code>, <code>for~in</code>)으로 배열을 순회하는 방법을 비교하고, 객체 배열을 이용해 화면에 카드 형태의 HTML을 동적으로 그려봤다.</p>
<pre><code class="language-html">&lt;!DOCTYPE html&gt;
&lt;html lang=&quot;en&quot;&gt;
&lt;head&gt;
    &lt;meta charset=&quot;UTF-8&quot;&gt;
    &lt;title&gt;Document&lt;/title&gt;
&lt;/head&gt;
&lt;body&gt;
    &lt;div id=&quot;display&quot;&gt;&lt;/div&gt;

    &lt;script&gt;
        const loopAry = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

        // 1) for문
        for (let idx = 0; idx &lt; loopAry.length; idx++) {
            console.log(`idx = ${idx} , value = ${loopAry[idx]}`);
        }

        // 2) forEach
        console.log(`debug &gt;&gt;&gt;&gt;&gt; forEach ~ `);
        loopAry.forEach((value, idx) =&gt; {
            console.log(`idx = ${idx}, value = ${value}`);
        });

        // 3) for~in
        console.log(`debug &gt;&gt;&gt;&gt;&gt; for ~ in `);
        for (let idx in loopAry) {
            console.log(`idx = ${idx}, value = ${loopAry[idx]}`);
        }

        // 객체 배열로 카드 리스트 동적 생성
        let img = 'xxxxx.png';
        let heading = 'xxxxxx';
        let cardAry = [
            { img, heading },
            { img, heading }
        ];

        let divTag = &quot;&quot;;
        cardAry.forEach((obj) =&gt; {
            divTag += `&lt;div&gt;&lt;img src='${obj.img}'&gt;`;
            divTag += `&lt;h2&gt;${obj.heading}&lt;/h2&gt;&lt;/div&gt;`;
        });
        document.getElementById(&quot;display&quot;).innerHTML = divTag;
    &lt;/script&gt;
&lt;/body&gt;
&lt;/html&gt;</code></pre>
<p><strong>주요 포인트</strong></p>
<table>
<thead>
<tr>
<th>개념</th>
<th>설명</th>
</tr>
</thead>
<tbody><tr>
<td><code>{ img, heading }</code></td>
<td><strong>단축 속성명(Shorthand Property)</strong> — <code>{ img: img, heading: heading }</code>을 줄여 쓴 문법. 변수 이름과 key 이름이 같을 때 사용 가능</td>
</tr>
<tr>
<td><code>innerHTML</code></td>
<td>문자열로 만든 HTML 태그를 실제 DOM에 삽입할 때 사용 (<code>textContent</code>와 달리 태그가 그대로 해석됨)</td>
</tr>
<tr>
<td><code>for</code> vs <code>forEach</code> vs <code>for~in</code></td>
<td><code>for</code>는 인덱스를 직접 다뤄야 할 때, <code>forEach</code>는 배열 전용 반복 메서드로 콜백 함수를 사용, <code>for~in</code>은 인덱스(key)를 순회하며 객체에도 사용 가능</td>
</tr>
</tbody></table>
<blockquote>
<p>주의: <code>loopAry.forEach((value, idx) =&gt; { console.log('idx = ${idx}, value = ${value}'); });</code> 부분은 원래 <strong>홑따옴표(<code>'</code>)</strong>로 되어 있으면 템플릿 리터럴이 동작하지 않고 <code>${idx}</code>가 문자 그대로 찍힌다. 반드시 백틱(<code>`</code>)으로 감싸야 값이 정상 출력된다.</p>
</blockquote>
<hr />
<h3 id="실습-2-addeventlistener로-버튼-클릭-이벤트-등록">실습 2) <code>addEventListener</code>로 버튼 클릭 이벤트 등록</h3>
<p>Bootstrap 버튼에 두 가지 방식(<code>onclick</code> 프로퍼티 vs <code>addEventListener</code>)으로 이벤트를 등록해보고 비교했다.</p>
<pre><code class="language-html">&lt;!DOCTYPE html&gt;
&lt;html lang=&quot;en&quot;&gt;
&lt;head&gt;
    &lt;meta charset=&quot;UTF-8&quot;&gt;
    &lt;title&gt;Document&lt;/title&gt;
    &lt;link href=&quot;https://cdn.jsdelivr.net/npm/bootstrap@5.3.8/dist/css/bootstrap.min.css&quot; rel=&quot;stylesheet&quot;&gt;
    &lt;script src=&quot;https://cdn.jsdelivr.net/npm/bootstrap@5.3.8/dist/js/bootstrap.bundle.min.js&quot;&gt;&lt;/script&gt;
&lt;/head&gt;
&lt;body&gt;
    &lt;div class=&quot;container&quot;&gt;
        &lt;button type=&quot;button&quot; class=&quot;btn btn-outline-danger&quot; id=&quot;handler&quot;&gt;SignIn&lt;/button&gt;
    &lt;/div&gt;

    &lt;script&gt;
        const signInhandler = (event) =&gt; {
            window.alert(`signIn button click`);
            console.log(`debug &gt;&gt;&gt;&gt; event ${event.target}`);
            console.log(`debug &gt;&gt;&gt;&gt; event ${event.target.value}`);
        };

        // case 02 (표준 방식)
        document.querySelector(&quot;#handler&quot;).addEventListener('click', () =&gt; {
            window.alert(`signIn button click`);
        });
    &lt;/script&gt;
&lt;/body&gt;
&lt;/html&gt;</code></pre>
<p><strong>주요 포인트</strong></p>
<ul>
<li><code>event</code> 매개변수를 통해 <strong>어떤 요소에서 이벤트가 발생했는지</strong>(<code>event.target</code>) 확인할 수 있음</li>
<li><code>signInhandler</code> 함수는 <code>event</code>를 매개변수로 받도록 정의했지만, 실제 <code>addEventListener</code>에는 익명 화살표 함수를 등록해서 <code>signInhandler</code>는 아직 연결되지 않은 상태. 연결하려면 <code>addEventListener('click', signInhandler)</code>처럼 함수 이름을 그대로 넘겨야 함</li>
</ul>
<hr />
<h3 id="실습-3-조건문으로-로그인-폼-값-검증하기">실습 3) 조건문으로 로그인 폼 값 검증하기</h3>
<p>이메일/비밀번호가 비어있는지 조건문으로 검사하고, 값에 따라 에러 메시지를 보여주거나 다음 페이지로 이동시키는 로그인 폼을 만들었다.</p>
<pre><code class="language-html">&lt;!DOCTYPE html&gt;
&lt;html lang=&quot;en&quot;&gt;
&lt;head&gt;
    &lt;meta charset=&quot;UTF-8&quot;&gt;
    &lt;title&gt;Document&lt;/title&gt;
    &lt;link href=&quot;https://cdn.jsdelivr.net/npm/bootstrap@5.3.8/dist/css/bootstrap.min.css&quot; rel=&quot;stylesheet&quot;&gt;
    &lt;script src=&quot;https://cdn.jsdelivr.net/npm/bootstrap@5.3.8/dist/js/bootstrap.bundle.min.js&quot;&gt;&lt;/script&gt;
&lt;/head&gt;
&lt;body&gt;
    &lt;div class=&quot;container&quot;&gt;
        &lt;div class=&quot;mb-3 mt-3&quot;&gt;
            &lt;label for=&quot;email&quot; class=&quot;form-label&quot;&gt;Email:&lt;/label&gt;
            &lt;input type=&quot;email&quot; class=&quot;form-control&quot; id=&quot;email&quot; placeholder=&quot;Enter email&quot; name=&quot;email&quot;&gt;
        &lt;/div&gt;
        &lt;div class=&quot;mb-3&quot;&gt;
            &lt;label for=&quot;pwd&quot; class=&quot;form-label&quot;&gt;Password:&lt;/label&gt;
            &lt;input type=&quot;password&quot; class=&quot;form-control&quot; id=&quot;pwd&quot; placeholder=&quot;Enter password&quot; name=&quot;pswd&quot;&gt;
        &lt;/div&gt;

        &lt;div id=&quot;err_empty_email_pwd&quot; style=&quot;display: none;&quot;&gt;
            &lt;div class=&quot;err_msg&quot;&gt;이메일 또는 패스워드를 입력해 주세요!!&lt;/div&gt;
        &lt;/div&gt;

        &lt;div class=&quot;form-check mb-3&quot;&gt;
            &lt;label class=&quot;form-check-label&quot;&gt;
                &lt;input class=&quot;form-check-input&quot; type=&quot;checkbox&quot; name=&quot;remember&quot;&gt; Remember me
            &lt;/label&gt;
        &lt;/div&gt;
        &lt;button type=&quot;button&quot; class=&quot;btn btn-primary&quot; id=&quot;signInBtn&quot;&gt;Submit&lt;/button&gt;
    &lt;/div&gt;

    &lt;script&gt;
        // 서버에서 전달받았다고 가정한 사용자 데이터
        const user = {
            email: &quot;hsson@naver.com&quot;,
            password: '1234'
        };

        // 페이지 로드 시 입력창에 기본값 채우기
        document.querySelector(&quot;#email&quot;).value = user.email;
        document.querySelector(&quot;#pwd&quot;).value = user.password;

        document.querySelector(&quot;#signInBtn&quot;).onclick = () =&gt; {
            window.alert(`button click`);
            let email = document.querySelector(&quot;#email&quot;).value;
            let passwd = document.querySelector(&quot;#pwd&quot;).value;
            console.log(`debug &gt;&gt;&gt;&gt;&gt; email : ${email}`);
            console.log(`debug &gt;&gt;&gt;&gt;&gt; passwd : ${passwd}`);

            if (email == '' || passwd == '') {
                document.querySelector(&quot;#err_empty_email_pwd&quot;).style.display = 'block';
                document.querySelector(&quot;#err_empty_email_pwd&quot;).style.color = &quot;red&quot;;
            } else {
                location.href = &quot;thanks.html?email=&quot; + email + &quot;&amp;passwd=&quot; + passwd;
            }
        };
    &lt;/script&gt;
&lt;/body&gt;
&lt;/html&gt;</code></pre>
<p><strong>주요 포인트</strong></p>
<table>
<thead>
<tr>
<th>개념</th>
<th>설명</th>
</tr>
</thead>
<tbody><tr>
<td>조건문 (<code>if</code>)</td>
<td>주어진 조건을 만족할 때만 특정 코드를 실행</td>
</tr>
<tr>
<td><code>array.filter</code></td>
<td>배열에서 조건에 맞는 값만 걸러내는 메서드</td>
</tr>
<tr>
<td>삼항 연산자</td>
<td><code>조건식 ? 참일때 : 거짓일때</code> — <code>if/else</code>를 한 줄로 줄인 문법. 여러 개를 이어 쓰면 <code>switch</code>처럼도 활용 가능</td>
</tr>
<tr>
<td><code>location.href</code></td>
<td>이 값을 바꾸면 <strong>페이지 이동</strong>이 일어남. BOM에 속하는 기능</td>
</tr>
</tbody></table>
<hr />
<h2 id="5-참고-자료">5. 참고 자료</h2>
<ul>
<li><a href="https://www.w3schools.com/">w3schools.com</a></li>
</ul>
<hr />
<h2 id="6-함수function-개념-정리">6. 함수(Function) 개념 정리</h2>
<p>오늘 실습 코드 전반에 함수 관련 개념이 많이 섞여 있어서, 따로 모아서 정리한다.</p>
<h3 id="6-1-함수를-만드는-3가지-방법">6-1. 함수를 만드는 3가지 방법</h3>
<pre><code class="language-javascript">// 방법 1) 함수 선언식 (Function Declaration)
function add(a, b) {
    return a + b;
}

// 방법 2) 함수 표현식 (Function Expression)
let add = function(a, b) {
    return a + b;
};

// 방법 3) 화살표 함수 (Arrow Function, ES6)
let add = (a, b) =&gt; a + b;</code></pre>
<table>
<thead>
<tr>
<th>방식</th>
<th>특징</th>
</tr>
</thead>
<tbody><tr>
<td>함수 선언식</td>
<td><code>function 이름() {}</code> 형태. 코드 어디서든 먼저 호출해도 동작함(호이스팅)</td>
</tr>
<tr>
<td>함수 표현식</td>
<td>변수에 함수를 담는 형태. 선언된 줄 이후부터만 호출 가능</td>
</tr>
<tr>
<td>화살표 함수</td>
<td>ES6에 새로 나온 문법. 표현식처럼 변수에 담아 쓰지만 문법이 더 짧음</td>
</tr>
</tbody></table>
<p>오늘 실습 코드는 대부분 <strong>화살표 함수</strong>를 사용했다 (<code>const signInhandler = (event) =&gt; {...}</code> 형태).</p>
<h3 id="6-2-매개변수parameter-vs-인자argument">6-2. 매개변수(Parameter) vs 인자(Argument)</h3>
<pre><code class="language-javascript">function add(a, b) {   // a, b = 매개변수(parameter) - 함수를 정의할 때 받기로 정한 자리
    return a + b;
}
add(10, 30);            // 10, 30 = 인자(argument) - 실제로 넘겨주는 값</code></pre>
<h3 id="6-3-콜백-함수-callback-function">6-3. 콜백 함수 (Callback Function)</h3>
<p><strong>&quot;함수를 다른 함수의 인자로 넘겨서, 나중에 대신 실행되게 하는 함수&quot;</strong>. 오늘 배운 반복문/이벤트 코드가 대부분 이 콜백 함수 패턴이다.</p>
<pre><code class="language-javascript">// forEach의 콜백 함수
loopAry.forEach((value, idx) =&gt; {
    console.log(`idx = ${idx}, value = ${value}`);
});

// addEventListener의 콜백 함수
document.querySelector(&quot;#handler&quot;).addEventListener('click', () =&gt; {
    window.alert(`signIn button click`);
});</code></pre>
<table>
<thead>
<tr>
<th>상황</th>
<th>콜백 함수가 하는 일</th>
</tr>
</thead>
<tbody><tr>
<td><code>forEach((value, idx) =&gt; {...})</code></td>
<td>배열의 <strong>각 요소마다</strong> 이 함수를 대신 호출해줌</td>
</tr>
<tr>
<td><code>addEventListener('click', () =&gt; {...})</code></td>
<td><strong>클릭이 발생할 때마다</strong> 이 함수를 대신 호출해줌</td>
</tr>
</tbody></table>
<p>&quot;함수 안에 함수를 넣는다&quot;는 게 낯설 수 있지만, <strong>&quot;이 상황이 오면 대신 실행해줘&quot;라고 미리 맡겨두는 것</strong>이라고 이해하면 된다.</p>
<h3 id="6-4-함수를-호출하는-방법에-따른-차이">6-4. 함수를 호출하는 방법에 따른 차이</h3>
<pre><code class="language-javascript">document.querySelector(&quot;#handler&quot;).addEventListener('click', signInhandler);    // ✅ 함수 자체를 등록
document.querySelector(&quot;#handler&quot;).addEventListener('click', signInhandler());  // ❌ 함수를 즉시 실행한 결과를 등록</code></pre>
<p><code>()</code>를 붙이면 <strong>&quot;지금 당장 실행해라&quot;</strong>는 뜻이 되고, <code>()</code>를 안 붙이면 <strong>&quot;이 함수 자체를 나중에 실행할 대상으로 넘겨라&quot;</strong>는 뜻이다. 오늘 실습에서 <code>signInhandler</code>가 정의만 되고 실제로는 연결이 안 됐던 이유가 바로 이것.</p>
<pre><code class="language-javascript">const signInhandler = (event) =&gt; {
    window.alert(`signIn button click`);
    console.log(`debug &gt;&gt;&gt;&gt; event ${event.target}`);
};

// 이렇게 연결해야 signInhandler가 실제로 클릭 이벤트에 쓰임
document.querySelector(&quot;#handler&quot;).addEventListener('click', signInhandler);</code></pre>
<h3 id="6-5-event-매개변수--이벤트-발생-정보를-담은-객체">6-5. <code>event</code> 매개변수 — 이벤트 발생 정보를 담은 객체</h3>
<pre><code class="language-javascript">const signInhandler = (event) =&gt; {
    console.log(event.target);        // 클릭된 요소 자체
    console.log(event.target.value);  // 그 요소의 value 값 (input일 때 주로 씀)
};</code></pre>
<p>콜백 함수의 첫 번째 매개변수 자리에 <code>event</code>를 써두면, 브라우저가 <strong>&quot;어떤 요소에서, 무슨 일이 일어났는지&quot;</strong>에 대한 정보를 자동으로 넘겨준다. 이름은 <code>event</code>가 아니라 <code>e</code> 등 다른 이름을 써도 되고, <strong>몇 번째 자리에 있는지</strong>가 중요하다.</p>
<h3 id="6-6-배열-전용-함수들-배열이-소유하고-있는-함수">6-6. 배열 전용 함수들 (배열이 &quot;소유&quot;하고 있는 함수)</h3>
<p>배열은 아래와 같은 함수들을 기본으로 갖고 있다.</p>
<table>
<thead>
<tr>
<th>함수</th>
<th>하는 일</th>
<th>진행 상태</th>
</tr>
</thead>
<tbody><tr>
<td><code>forEach</code></td>
<td>배열의 각 요소를 순회하며 콜백 함수 실행 (반환값 없음)</td>
<td>✅ 오늘 실습함</td>
</tr>
<tr>
<td><code>map</code></td>
<td>각 요소를 가공해서 <strong>새 배열</strong>을 만들어 반환</td>
<td>다음에 배울 예정</td>
</tr>
<tr>
<td><code>filter</code></td>
<td>조건에 맞는 요소만 걸러서 <strong>새 배열</strong>로 반환</td>
<td>다음에 배울 예정</td>
</tr>
<tr>
<td><code>reduce</code></td>
<td>배열 전체를 하나의 값으로 압축(합계 등)</td>
<td>다음에 배울 예정</td>
</tr>
</tbody></table>
<h3 id="함수-개념-전체-흐름-정리">함수 개념 전체 흐름 정리</h3>
<pre><code>함수 만드는 방법        → 화살표 함수가 요즘 표준
    ↓
{} 유무                → return 필요 여부 결정
    ↓
콜백 함수로 활용        → forEach, addEventListener에 함수를 &quot;맡겨두기&quot;
    ↓
event 매개변수 활용     → 어떤 요소에서 무슨 일이 일어났는지 정보 받기</code></pre><hr />
<h2 id="7-내일을-위한-예습">7. 내일을 위한 예습</h2>
<ul>
<li>통신 (서버와 데이터 주고받기)</li>
<li>이벤트 렌더링</li>
</ul>