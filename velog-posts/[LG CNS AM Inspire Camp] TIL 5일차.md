<hr />
<h3 id="📌-오늘의-한-줄-요약">📌 오늘의 한 줄 요약</h3>
<p>React의 구조와 코드에 대해 알아본 시간. JSX 문법과 컴포넌트에 자주 쓰이는 최신 JS 문법(Spread, 구조분해, Optional Chaining, 널 병합 연산자)을 함께 익혔다.</p>
<hr />
<h3 id="🧠-오늘-배운-개념-정리">🧠 오늘 배운 개념 정리</h3>
<h4 id="1-jsx--javascript--xmlhtml">1) JSX = JavaScript + XML/HTML</h4>
<p>JS 코드 안에 HTML처럼 생긴 마크업을 바로 쓸 수 있게 해주는 문법. <code>{ }</code> 안에 자바스크립트 표현식을 넣으면, HTML 태그 안에서 변수·함수 결과를 자유롭게 꺼내 쓸 수 있다. (오늘 실습 <code>Librarypage.jsx</code>의 <code>{books.filter(...).map(...)}</code> 부분이 그 예시.)</p>
<h4 id="2-virtual-dom이란">2) Virtual DOM이란?</h4>
<p>실제 DOM을 조작하는 대신, 같은 내용을 담은 자바스크립트 객체(가상 DOM)를 메모리에 두고 먼저 비교한 뒤 달라진 부분만 실제 DOM에 반영하는 방식이다. 브라우저 화면을 직접 그리는 작업은 비용이 크기 때문에, 변경 전/후 가상 DOM을 비교(재조정, Reconciliation)해서 바뀐 부분만 골라 업데이트하면 훨씬 가볍고 빠르다.</p>
<pre><code>state 변경
   │
   ▼
새 Virtual DOM 생성 ──비교(Reconciliation)──▶ 이전 Virtual DOM
   │
   ▼
달라진 부분만 실제 DOM에 반영 (리렌더링)</code></pre><h4 id="3-구조분해-할당destructuring">3) 구조분해 할당(Destructuring)</h4>
<p>객체나 배열 안의 값을 변수에 바로 꺼내 담는 문법. <code>const { bookName, price } = book;</code> 처럼 쓰면 매번 <code>book.bookName</code>으로 접근하지 않아도 되어 코드가 짧아진다.</p>
<h4 id="4-spread-operator-전개-연산자">4) Spread Operator (전개 연산자)</h4>
<p><code>...</code>을 붙여서 배열이나 객체의 요소를 하나씩 풀어 펼치는 문법. 배열/객체를 복사하거나 합칠 때, 또는 <code>props</code>를 통째로 컴포넌트에 넘길 때 자주 쓰인다.</p>
<h4 id="5-props내장-객체란">5) props(내장 객체)란?</h4>
<p>부모 컴포넌트가 자식 컴포넌트에게 값을 전달할 때 쓰는 객체. 오늘 실습에서 <code>&lt;Book bookName={book.bookName}/&gt;</code>처럼 부모(<code>Librarypage</code>)가 자식(<code>Book</code>)에게 <code>bookName</code>을 넘기면, 자식은 <code>props.bookName</code>으로 그 값을 받아 화면에 표시한다.</p>
<h4 id="6-삼항-연산자">6) 삼항 연산자</h4>
<p><code>조건 ? 참일때값 : 거짓일때값</code> 형태로, 간단한 <code>if/else</code>를 한 줄로 줄여 쓰는 문법.</p>
<h4 id="7-optional-chaining--vs-널-병합-연산자-">7) Optional Chaining (<code>?.</code>) vs 널 병합 연산자 (<code>??</code>)</h4>
<p>둘 다 값이 없을 때를 안전하게 처리하기 위한 문법이지만 역할이 다르다.</p>
<table>
<thead>
<tr>
<th>구분</th>
<th>Optional Chaining <code>?.</code></th>
<th>널 병합 연산자 <code>??</code></th>
</tr>
</thead>
<tbody><tr>
<td>주 사용 대상</td>
<td>객체의 중첩 속성 접근</td>
<td>변수의 기본값 지정</td>
</tr>
<tr>
<td>동작</td>
<td>참조가 <code>null</code>/<code>undefined</code>면 에러 대신 <code>undefined</code>를 반환하고 이후 접근을 멈춤</td>
<td>왼쪽 값이 <code>null</code>/<code>undefined</code>일 때만 오른쪽 값을 반환 (<code>0</code>, <code>''</code>처럼 falsy이지만 유효한 값은 그대로 유지)</td>
</tr>
<tr>
<td>예시</td>
<td><code>obj.address?.city</code></td>
<td><code>value ?? '기본값'</code></td>
</tr>
</tbody></table>
<p>→ <code>||</code>(논리 OR)는 <code>0</code>이나 <code>''</code>도 무조건 걸러버리지만, <code>??</code>는 진짜 <code>null</code>/<code>undefined</code>일 때만 기본값을 넣어준다는 점이 핵심 차이다.</p>
<hr />
<h3 id="🤔-헷갈렸던-점">🤔 헷갈렸던 점</h3>
<ul>
<li><strong>Optional Chaining(<code>?.</code>)</strong>: 어디까지가 안전하게 처리되는지, <code>.</code>(점 표기법)과 헷갈렸음 → 참조가 nullish일 때만 <code>undefined</code>로 멈춘다는 점을 확인</li>
<li><strong>널 병합 연산자(<code>??</code>)</strong>: <code>||</code>랑 비슷해 보였지만, <code>0</code>처럼 falsy하지만 유효한 값을 다르게 처리한다는 차이를 정확히 몰랐음 → 위 표로 정리하며 구분 완료</li>
</ul>
<hr />
<h3 id="💻-실습--적용">💻 실습 / 적용</h3>
<h4 id="--librarypagejsx">--Librarypage.jsx</h4>
<pre><code class="language-jsx">import Book from &quot;../../components/sample/Book&quot;
const Librarypage = () =&gt; {
        /*
        jsx = script + html
        */
        // Script
        const books = [
        {category : 'it',       bookName :'java',    price : '10,000원'},
        {category : 'it',       bookName :'java',    price : '10,000원'},
        {category : 'lang',     bookName :'kor',     price : '10,000원'},
        {category : 'lang',     bookName :'eng',     price : '10,000원'},
        {category : 'essay',    bookName :'xxxx',    price : '10,000원'},
        {category : 'essay',    bookName :'xxxx',    price : '10,000원'},
         ];
        // UI
        // html에서 스크립트 변수를 {}를 이용해서 자유롭게 사용 가능
        return(
            &lt;div&gt;
                {
                    books
                        .filter( book =&gt; book.category === 'lang')
                        .map((book,idx) =&gt; {
                                return &lt;Book bookName={book.bookName}/&gt;
                        })
                }
            &lt;/div&gt;
        );

}
export default Librarypage;</code></pre>
<h4 id="--bookjsx">--Book.jsx</h4>
<pre><code class="language-jsx">const Book = (props) =&gt; {
    return(
        &lt;div&gt;
            &lt;div&gt;
                &lt;span&gt;책이름 : {props.bookName} &lt;/span&gt;
            &lt;/div&gt;
        &lt;/div&gt;
    );
}
export default Book ;</code></pre>
<blockquote>
<p>💡 <code>Librarypage</code>는 <code>books</code> 배열을 <code>filter</code>로 걸러 <code>category === 'lang'</code>인 항목만 골라내고, <code>map</code>으로 하나씩 <code>&lt;Book&gt;</code> 컴포넌트에 <code>props</code>로 넘겨서 화면에 그린다. 컴포넌트를 재사용해서 여러 개의 책 카드를 찍어내는 구조를 실습으로 확인했다.</p>
</blockquote>
<hr />
<h3 id="📚-참고-자료">📚 참고 자료</h3>
<ul>
<li><a href="https://www.airbnb.co.kr/">https://www.airbnb.co.kr/</a></li>
<li><a href="https://reactjs.org/docs/create-a-new-react-app.html">https://reactjs.org/docs/create-a-new-react-app.html</a></li>
</ul>
<hr />
<h3 id="🔜-복습자료">🔜 복습자료</h3>
<ul>
<li>오늘 배운 내용 복습을 위한 무료 강의
<a href="https://www.inflearn.com/course/amazing-javascript-%EC%9E%85%EB%AC%B8">https://www.inflearn.com/course/amazing-javascript-%EC%9E%85%EB%AC%B8</a></li>
</ul>