<h3 id="어제보다-deep-한-script-🚀">어제보다 DEEP 한 Script 🚀</h3>
<hr />
<h3 id="📌-오늘의-한-줄-요약">📌 오늘의 한 줄 요약</h3>
<p>어제는 Script의 기초를 배웠다면, 오늘은 <strong>Promise</strong>와 <strong>axios</strong>를 이용해 서버 데이터를 화면에 렌더링하는 법을 배웠다.</p>
<hr />
<h3 id="🧠-오늘-배운-개념-정리">🧠 오늘 배운 개념 정리</h3>
<h4 id="1-컴포넌트component란">1) 컴포넌트(Component)란?</h4>
<p>화면을 이루는 UI를 재사용 가능한 작은 단위로 쪼갠 조각을 말한다. 오늘 실습한 상품 카드(<code>img</code> + <code>title</code> + <code>price</code> + 삭제 버튼)처럼, 반복되는 구조를 하나의 틀로 만들어 데이터만 바꿔가며 여러 번 찍어내는 방식으로 이해하면 쉽다.</p>
<h4 id="2-serveropen-api-server--legacy">2) server(open api server) / legacy</h4>
<ul>
<li><strong>open api server</strong>: 누구나 호출해서 데이터를 받아올 수 있게 열어둔 서버. 오늘 실습에서 사용한 <code>fakestoreapi.com</code>이 그 예시.</li>
<li><strong>legacy</strong>: 예전 방식으로 만들어져서 지금도 유지·운영되고 있는 코드나 시스템을 뜻함.</li>
</ul>
<h4 id="3-textcontent-vs-innerhtml-vs-innertext">3) textContent vs innerHTML vs innerText</h4>
<p>셋 다 요소의 텍스트를 읽거나 바꾸는 데 쓰이지만 성격이 다르다.</p>
<table>
<thead>
<tr>
<th>속성</th>
<th>소속</th>
<th>특징</th>
</tr>
</thead>
<tbody><tr>
<td><code>innerHTML</code></td>
<td>Element</td>
<td>태그까지 포함한 마크업 전체를 가져오거나 설정. HTML 문자열을 파싱하므로 XSS 위험과 성능 비용이 있음</td>
</tr>
<tr>
<td><code>innerText</code></td>
<td>HTMLElement</td>
<td>화면에 실제로 렌더링된(보이는) 텍스트만 반환. CSS로 숨긴 내용은 제외되고, 값을 읽을 때 리플로우가 발생해 상대적으로 비용이 큼</td>
</tr>
<tr>
<td><code>textContent</code></td>
<td>Node</td>
<td><code>&lt;script&gt;</code>, <code>&lt;style&gt;</code>, <code>display:none</code> 요소까지 포함해서 노드 안의 모든 텍스트를 그대로 반환. 단순 텍스트만 바꿀 땐 이게 가장 가볍고 안전함</td>
</tr>
</tbody></table>
<p>→ 오늘 실습(<code>scriptProperty.html</code>)처럼 <strong>단순 텍스트만 바꿀 때는 <code>textContent</code></strong> 를 쓰는 게 정석이라는 걸 확인했다.</p>
<h4 id="4-promise란">4) Promise란?</h4>
<p>비동기 작업(서버 요청처럼 시간이 걸리는 일)이 나중에 끝났을 때 결과를 돌려주겠다는 &quot;약속&quot; 객체.</p>
<p><strong>상태(state) 흐름</strong></p>
<pre><code>pending(대기, 초기 상태)
   │
   ├─ 성공 → resolve() 호출 → fulfilled(이행)
   └─ 실패 → reject()  호출 → rejected(실패)</code></pre><ul>
<li><code>fulfilled</code>, <code>rejected</code>는 한 번 정해지면 되돌릴 수 없고, 이 두 상태를 합쳐서 <strong>settled(처리 완료)</strong> 라고 부른다.</li>
<li><code>result</code>는 처음엔 <code>undefined</code>이며, <code>resolve(value)</code>가 호출되면 서버가 준 json data 같은 값으로 채워진다.</li>
<li>결과를 받는 방법: <code>.then()</code>(성공 처리) → <code>.catch()</code>(실패 처리) → <code>.finally()</code>(성공/실패 관계없이 항상 실행)</li>
</ul>
<h4 id="5-axios-vs-fetch-script들의-정의">5) axios vs fetch (script들의 정의)</h4>
<table>
<thead>
<tr>
<th>구분</th>
<th>fetch</th>
<th>axios</th>
</tr>
</thead>
<tbody><tr>
<td>제공 방식</td>
<td>브라우저 내장 API</td>
<td>별도 설치가 필요한 라이브러리</td>
</tr>
<tr>
<td>JSON 변환</td>
<td><code>response.json()</code>을 직접 한 번 더 호출해야 함</td>
<td>응답을 자동으로 JSON으로 파싱해 <code>response.data</code>에 담아줌</td>
</tr>
<tr>
<td>에러 처리</td>
<td>4xx·5xx 응답도 성공(resolve)으로 처리되어 별도 분기 필요</td>
<td>4xx·5xx 응답을 자동으로 reject 처리해 <code>.catch()</code> 하나로 처리 가능</td>
</tr>
<tr>
<td>부가 기능</td>
<td>기본 기능만 제공</td>
<td>타임아웃, 인터셉터, 요청 취소 등 편의 기능 제공</td>
</tr>
</tbody></table>
<p>→ 오늘 실습에서 <code>axios.get()</code>으로 상품 목록을 받아오면서, <code>response.json()</code> 없이 바로 <code>response.data</code>를 쓸 수 있다는 점이 fetch와 가장 크게 체감되는 차이였다.</p>
<hr />
<h3 id="🤔-헷갈렸던-점">🤔 헷갈렸던 점</h3>
<ul>
<li><code>scriptProperty.html</code> 퀴즈(버튼 클릭 시 <code>h1</code> 텍스트와 <code>img</code> 속성을 바꾸는 문제)를 풀 때 어떤 속성으로 값을 바꿔야 할지 헷갈렸음</li>
<li>→ <code>textContent</code>(텍스트 변경)와 <code>src</code>(이미지 속성 변경)를 구분해서 써야 한다는 걸 확인</li>
<li>관련 코드는 아래 4번 <strong><code>scriptProperty.html</code></strong> 항목에 표시해둠</li>
</ul>
<hr />
<h3 id="💻-실습--적용">💻 실습 / 적용</h3>
<h4 id="--scriptaxioshtml">--scriptAxios.html</h4>
<pre><code class="language-html">&lt;!DOCTYPE html&gt;
&lt;html lang=&quot;en&quot;&gt;
    &lt;head&gt;
        &lt;meta charset=&quot;UTF-8&quot;&gt;
        &lt;meta name=&quot;viewport&quot; content=&quot;width=device-width, initial-scale=1.0&quot;&gt;
        &lt;title&gt;Document&lt;/title&gt;
        &lt;script src=&quot;https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js&quot;&gt;&lt;/script&gt;
        &lt;link href=&quot;../css/axios.css&quot; rel=&quot;stylesheet&quot;&gt;
    &lt;/head&gt;

    &lt;body&gt;
        &lt;h1 class=&quot;title&quot;&gt;상품목록&lt;/h1&gt;

        &lt;div class=&quot;container&quot; id=&quot;container&quot;&gt;&lt;/div&gt;

        &lt;script&gt;
            // image(사이즈 조절), title, price
            ary = [];
             const loadData = async (e) =&gt; {
                await axios.get('https://fakestoreapi.com/products')
                    .then( response =&gt; {
                        console.log(`debug &gt;&gt;&gt;&gt;&gt; response` ,response);
                        ary = response.data;
                    })
                    .catch( error =&gt; {
                        console.log(`debug &gt;&gt;&gt;&gt;&gt; error ${error}`);
                    })
                    .finally( () =&gt; {
                        console.log(`debug &gt;&gt;&gt;&gt;&gt; request completed`);
                    });
                 makeProduct();
             }

             const makeProduct = (e) =&gt; {

                container.innerHTML ='';
                ary.forEach((object,idx) =&gt; {
                    const card = document.createElement('div');
                    card.className = 'card';

                    const img           = document.createElement('img');
                    img.src             = object.image;
                    const title         = document.createElement('h3');
                    title.textContent   = object.title ;
                    const price         = document.createElement('p');
                    price.textContent   = `${object.price}$`;

                    //
                    const delBtn = document.createElement('button');
                    delBtn.className = 'btn'
                    delBtn.textContent =&quot;삭제&quot;;


                    delBtn.onclick = (e) =&gt; {
                        window.alert('button click')
                        ary.splice(idx,1) ;
                        makeProduct();
                    }

                    card.appendChild(img);
                    card.appendChild(title);
                    card.appendChild(price);
                    card.appendChild(delBtn);

                    container.appendChild(card) ;
                });
             }



            const init = async (e) =&gt; {
                await loadData();
                makeProduct();
             }
             init();

        &lt;/script&gt;
    &lt;/body&gt;
&lt;/html&gt;</code></pre>
<h4 id="--scriptpromisehtml">--scriptPromise.html</h4>
<pre><code class="language-html">&lt;!DOCTYPE html&gt;
&lt;html lang=&quot;en&quot;&gt;
    &lt;head&gt;
        &lt;meta charset=&quot;UTF-8&quot;&gt;
        &lt;meta name=&quot;viewport&quot; content=&quot;width=device-width, initial-scale=1.0&quot;&gt;
        &lt;title&gt;Document&lt;/title&gt;
    &lt;/head&gt;
    &lt;body&gt;
        &lt;button type =&quot;button&quot; id =&quot;btn&quot;&gt;users&lt;/button&gt;

        &lt;script&gt;
            // const promise = new Promise( (resolve, reject) =&gt;{
            //     setTimeout( () =&gt; {
            //         // resolve({ status : 200, data : {message : 'success'}});
            //         reject({ status : 404 });
            //     },5000)
            // });

            // promise
            //     .then( (response ) =&gt; {
            //         console.log('debug &gt;&gt;&gt;&gt;&gt; response ', response);
            //     })
            //     .catch((error ) =&gt; {
            //         console.log('debug &gt;&gt;&gt;&gt;&gt; error ', error);
            //     });

            // script : fetch api, third parts : axios lib
            document.querySelector(&quot;#btn&quot;).onclick = async (e) =&gt; {
                await fetch('../server/users.json')
                // fetch('../server/data.json')
                // fetch(https://fakestoreapi.com/products)
                    .then( response =&gt; response.json())
                    .then( data =&gt; {
                        console.log('debug &gt;&gt;&gt;&gt;&gt; data ', data);
                    })
                    .catch( error =&gt; {
                        console.log('debug &gt;&gt;&gt;&gt;&gt; error ${eroor}');
                    });
            }
        &lt;/script&gt;
    &lt;/body&gt;
&lt;/html&gt;</code></pre>
<h4 id="--scriptpropertyhtml-🤔-헷갈렸던-점-관련-코드">--scriptProperty.html 🤔 (헷갈렸던 점 관련 코드)</h4>
<pre><code class="language-html">&lt;!DOCTYPE html&gt;
&lt;html lang=&quot;en&quot;&gt;
    &lt;head&gt;
        &lt;meta charset=&quot;UTF-8&quot;&gt;
        &lt;meta name=&quot;viewport&quot; content=&quot;width=device-width, initial-scale=1.0&quot;&gt;
        &lt;title&gt;Document&lt;/title&gt;
        &lt;link href=&quot;../css/property.css&quot; rel=&quot;stylesheet&quot;&gt;
    &lt;/head&gt;

    &lt;body&gt;

        &lt;div&gt;
            &lt;h1&gt;Inspire Coffee&lt;/h1&gt;
            &lt;img src=&quot;../img/coffee-blue.jpg&quot;
                width=&quot;200&quot;
                height=&quot;200&quot;&gt;

        &lt;/div&gt;

        &lt;div&gt;
            &lt;!-- &lt;button type=&quot;button&quot; onclick=&quot;handler('Lgcns Coffee', 'coffee-pink.jpg')&quot;&gt;헤딩과 이미지의 속성을 변경&lt;/button&gt; --&gt;
            &lt;button type=&quot;button&quot;
            id=&quot;btn&quot;&gt;헤딩과 이미지의 속성을 변경&lt;/button&gt;
        &lt;/div&gt;

        &lt;script&gt;
            /*
            Q)
            - step01 : 버튼의 이벤트를 감지하는 함수정의
            - step02 : 이벤트 발생시 h1, img 태그에 접근하여
            - step03 : 이미지 src 속성에 접근하여  coffee-pink.jpg 변경
            - step04 : h1 접근해서 텍스트를  Lgcns Coffee 변경
            - 필요에 따라서 태그에 선택자를 정의할 수 있음.

            hint) 특정영역에 text 삽입시 : textContent, innerHTML, innerText

            - step05 : 테스트 완료되면 ai agent 도움을 받아서 css 작성하고 꾸미기
            */

               const title = document.querySelector(&quot;h1&quot;);
                const img = document.querySelector(&quot;img&quot;);
                const btn = document.querySelector(&quot;#btn&quot;);

                let isBlue = true;

                btn.addEventListener(&quot;click&quot;, (e) =&gt; {

                    console.log(e.target.textContent);
                    alert(&quot;signIn button click&quot;);

                    if (isBlue) {
                        title.textContent = &quot;Lgcns Coffee&quot;;
                        img.src = &quot;../img/coffee-pink.jpg&quot;;
                    } else {
                        title.textContent = &quot;Inspire Coffee&quot;;
                        img.src = &quot;../img/coffee-blue.jpg&quot;;
                    }

                    isBlue = !isBlue;
                });

        &lt;/script&gt;
    &lt;/body&gt;
&lt;/html&gt;</code></pre>
<h4 id="--mediaqueryhtml">--mediaQuery.html</h4>
<pre><code class="language-html">&lt;!DOCTYPE html&gt;
&lt;html lang=&quot;en&quot;&gt;
    &lt;head&gt;
        &lt;meta charset=&quot;UTF-8&quot;&gt;
        &lt;meta name=&quot;viewport&quot; content=&quot;width=device-width, initial-scale=1.0&quot;&gt;
        &lt;title&gt;Document&lt;/title&gt;
        &lt;link href=&quot;../css/media.css&quot; rel=&quot;stylesheet&quot;&gt;
    &lt;/head&gt;
    &lt;body&gt;
        &lt;div&gt;
            &lt;h1&gt;안녕하세요~ 미디어 쿼리입니다!&lt;/h1&gt;
        &lt;/div&gt;

        &lt;script&gt;
            /*
            미디어 쿼리?
            - 해상도에 따른 컴포넌트의 재 배치
            mobile : 320px ~ 430px ~ 767px
            tablet : 768px ~ 1023px
            desktop : 1024px~
            */


        &lt;/script&gt;
    &lt;/body&gt;
&lt;/html&gt;</code></pre>
<blockquote>
<p>💡 promise와 axios는 &quot;비동기로 데이터를 받아오는 것&quot;이라는 목적은 같지만, axios는 그 위에 자동 JSON 파싱·에러 처리 같은 편의 기능을 얹은 라이브러리라는 관계로 이해하면 헷갈리지 않는다.</p>
</blockquote>
<hr />
<h3 id="📚-참고-자료">📚 참고 자료</h3>
<ul>
<li><a href="https://openrouter.ai/?ref=openrouter.com">https://openrouter.ai/?ref=openrouter.com</a></li>
<li><a href="https://axios.rest">https://axios.rest</a></li>
</ul>
<hr />
<h3 id="🔜-내일을-위한-예습">🔜 내일을 위한 예습</h3>
<ul>
<li><input disabled="" type="checkbox" /> Skeleton UI</li>
<li><input disabled="" type="checkbox" /> React</li>
</ul>