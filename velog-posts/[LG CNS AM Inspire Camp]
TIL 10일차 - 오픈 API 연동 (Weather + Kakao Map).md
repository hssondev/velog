<h2 id="1-오늘의-맥락--외부open-api를-붙여서-실시간-데이터-가져오기">1. 오늘의 맥락 — 외부(Open) API를 붙여서 실시간 데이터 가져오기</h2>
<p>지금까지는 json-server(내가 만든 가짜 서버)와만 통신했다면, 오늘은 진짜 외부 서비스인 <strong>OpenWeatherMap</strong>과 <strong>Kakao Map</strong>을 붙여봤다.</p>
<p>처음 목표는 단순했다. 도시 이름 버튼(서울/부산/…)을 누르면 그 도시 날씨만 보여주는 것. 그런데 여기서부터 이미 한 번 방향이 바뀌었다 — 도시 이름을 영문(<code>&quot;Seoul, KR&quot;</code>)으로 넘기다가, 나중에 카카오맵의 주소 검색(Geocoder)을 쓰기로 하면서 한글 지명(<code>&quot;서울&quot;</code>)으로 배열 자체를 바꿔야 했다. 그리고 여기서 그치지 않고, 지도를 아예 화면에 띄워서 <strong>직접 클릭한 위치</strong>의 날씨까지 볼 수 있도록 카카오맵을 얹었다. 결과적으로 하루 사이에 &quot;버튼으로만 날씨 보기&quot;에서 &quot;지도 클릭으로도 날씨 보기&quot;까지 기능이 한 겹 더 쌓인 셈이다.</p>
<hr />
<h2 id="2-코드-흐름-한눈에-보기">2. 코드 흐름 한눈에 보기</h2>
<pre><code class="language-null">WeatherPage (부모, 데이터와 로직 담당)
   │
   ├─ 마운트 시 useEffect → getCurrentLocation()
   │      └─ navigator.geolocation → 내 현재 위치(lat, lon) 확인
   │            └─ getCurrentWeather(lat, lon) → fetch로 날씨 요청 → setWeather()
   │
   ├─ WeatherButton (자식) — 도시 버튼 클릭
   │      └─ cityHandler(e, city) → setCity(city) + getCoordsByCity(city)
   │            └─ (카카오 geocoder로 도시명 → 좌표 변환)
   │                  └─ getWeatherByCoords(lat, lng) → fetch → setWeather()
   │
   ├─ KaKaoMap (자식) — 지도 클릭
   │      └─ 지도 클릭 이벤트 → setWeatherByCoords(lat, lng) (=getWeatherByCoords, props로 전달받음)
   │
   └─ WeatherBox (자식) — weather state를 받아 화면에 표시만 함</code></pre>
<p>한 마디로 정리하면, <strong>&quot;어떤 방식으로든 위도·경도(lat, lng)를 알아낸다 → 그 좌표로 날씨 API를 fetch한다 → weather state를 갱신한다 → 화면이 자동으로 다시 그려진다&quot;</strong>는 흐름이 여러 경로(버튼 클릭 / 지도 클릭 / 최초 마운트 시 현재 위치)로 반복되는 구조다. 처음엔 이 세 경로를 각각 따로 짤 뻔했는데, 결국 &quot;좌표만 알아내면 나머지는 같은 함수(<code>getWeatherByCoords</code>)로 합칠 수 있다&quot;는 걸 깨닫고 나서 코드가 훨씬 정리됐다.</p>
<hr />
<h2 id="3-weatherpage--데이터를-불러오는-세-가지-경로">3. WeatherPage — 데이터를 불러오는 세 가지 경로</h2>
<p><strong>JSX</strong> · <code>WeatherPage.jsx</code> — 상태 선언과 도시 버튼 핸들러</p>
<pre><code class="language-jsx">const key = process.env.REACT_APP_WEATHER_API_KEY ;

const cities = [&quot;서울&quot;, &quot;부산&quot;, &quot;대전&quot;, &quot;인천&quot;, &quot;광주광역시&quot;, &quot;여수&quot;] ;

const [city, setCity] = useState('');
const [weather, setWeather] = useState({});

// handler
const cityHandler = (e, city) =&gt; {
    setCity(city)
    // 기존 api 대신 좌표기반으로 변경
    getCoordsByCity(city);
}</code></pre>
<p><strong>코드 리뷰</strong></p>
<ul>
<li>API 키(<code>key</code>)는 <code>.env</code> 파일에 <code>REACT_APP_WEATHER_API_KEY</code>로 저장해두고, <code>process.env</code>로 꺼내 쓴다. 코드 안에 키를 직접 적지 않는 이유는, 키가 그대로 저장소(github 등)에 노출되는 걸 막기 위해서다.</li>
<li><code>cities</code> 배열은 원래 영문(<code>&quot;Seoul, KR&quot;</code>)이었다. OpenWeather API는 도시 이름을 영문으로 넘기면 그 자체로 조회가 됐기 때문이다. 그런데 카카오맵의 Geocoder(주소 검색)를 쓰기로 하면서 문제가 생겼다 — 카카오 Geocoder는 한글 지명 위주로 잘 인식하고, 영문으로 넘기면 결과가 잘 안 돌아왔다. 그래서 배열 자체를 한글(<code>&quot;서울&quot;</code>)로 바꿨다. 두 API가 기대하는 입력 형식이 다르다는 걸 직접 부딪히고 나서야 알게 된 부분이다.</li>
<li><code>cityHandler</code>는 버튼을 클릭했을 때 두 가지 일을 한다: ① <code>city</code> state를 바꾸고, ② <code>getCoordsByCity(city)</code>를 호출해서 그 도시의 좌표를 구하러 간다. 처음엔 이 함수 안에서 바로 <code>fetch</code>로 날씨까지 가져오려고 했는데, 지도 클릭 쪽 로직과 겹치는 부분이 많아서 &quot;좌표를 구하는 일&quot;과 &quot;날씨를 가져오는 일&quot;을 분리하는 쪽으로 정리했다.</li>
</ul>
<p><strong>JSX</strong> · <code>WeatherPage.jsx</code> — 좌표로 날씨 가져오기 + 현재 위치 감지</p>
<pre><code class="language-jsx">const getWeatherByCoords = async (lat, lng) =&gt; {
    let endPoint = `https://api.openweathermap.org/data/2.5/weather?lat=${lat}&amp;lon=${lng}&amp;appid=${key}`;
    await fetch(endPoint)
        .then( response =&gt; {
            return response.json() ;
        })
        .then( data =&gt; {
            setWeather(data);
        })
        .catch( error =&gt; {
            console.log(`debug &gt;&gt;&gt;&gt; fetch error ` , error);
        }) ;
};

const [mapCenter, setMapCenter] = useState(null);

const getCoordsByCity = (cityName) =&gt; {
    const geocoder = new window.kakao.maps.services.Geocoder();
    geocoder.addressSearch(cityName,(result, status)=&gt;{
        if(status === window.kakao.maps.services.Status.OK) {
            const lat = parseFloat(result[0].y) ;
            const lng = parseFloat(result[0].x) ;
            setMapCenter({lat: lat , lng : lng, time : Date.now()});
            getWeatherByCoords(lat,lng);
        }else{
            console.log(`debug &gt;&gt;&gt;&gt; getCoordsByCity 좌표변환실패`);
        }
    });
}</code></pre>
<p><strong>코드 리뷰</strong></p>
<ul>
<li><code>getWeatherByCoords</code>는 도시 이름이 아니라 <strong>위도(lat)·경도(lng)</strong> 기준으로 OpenWeather API를 호출하는 공통 함수다. 도시 버튼을 눌렀을 때도, 지도를 클릭했을 때도 결국 이 함수 하나로 합쳐진다는 게 오늘 가장 중요한 설계 판단이었다.</li>
<li><code>getCoordsByCity</code>는 카카오맵 SDK가 제공하는 <code>Geocoder</code>(주소 ↔ 좌표 변환기)를 이용해서, 도시 이름을 좌표로 바꾼다. <code>result[0].y</code>가 위도, <code>result[0].x</code>가 경도인데, 이름만 보면 순서가 직관과 반대라 처음엔 <code>x</code>/<code>y</code>를 헷갈려서 좌표가 엉뚱한 곳을 가리켰다. 콘솔에 <code>result</code>를 찍어서 실제 구조를 확인하고 나서야 바로잡을 수 있었다.</li>
<li>⚠️ <code>result[0].y</code>, <code>result[0].x</code>는 <strong>문자열</strong>로 내려온다. 그래서 <code>parseFloat</code>로 숫자로 바꿔주지 않으면 좌표 계산이 이상하게 동작한다. 처음엔 이 캐스팅을 빠뜨려서 지도가 이상한 위치로 움직이는 걸 보고서야 원인을 알아챘다.</li>
<li><code>mapCenter</code>에 <code>time: Date.now()</code>를 같이 넣은 것도 시행착오의 결과다. 처음엔 <code>{lat, lng}</code>만 넣었는데, 같은 도시를 두 번 연속 클릭하면 <code>useEffect</code>의 의존성 배열이 &quot;값이 안 바뀌었다&quot;고 판단해서 지도가 안 움직이는 문제가 있었다. <code>time</code> 값을 매번 다르게 넣어서 <code>mapCenter</code> 객체 자체가 항상 새로운 값으로 인식되게 만들어 해결했다.</li>
</ul>
<p><strong>JSX</strong> · <code>WeatherPage.jsx</code> — 현재 위치 자동 감지</p>
<pre><code class="language-jsx">const getCurrentLocation = () =&gt; {
    navigator.geolocation.getCurrentPosition((position) =&gt; {
        let lat = position.coords.latitude ;
        let lon = position.coords.longitude ;
        getCurrentWeather(lat, lon);
    });
}

useEffect(() =&gt; {
    getCurrentLocation();
}, [])</code></pre>
<p><strong>코드 리뷰</strong></p>
<ul>
<li>브라우저 내장 기능인 <code>navigator.geolocation.getCurrentPosition()</code>으로 사용자의 현재 위치(위도/경도)를 물어본다. 브라우저가 위치 권한을 사용자에게 물어보는 팝업이 뜨는 그 기능인데, 처음 테스트할 때 권한을 거부해버려서 한동안 아무 반응이 없는 걸 보고 당황했다 — 권한을 다시 허용으로 바꾸고 나서야 정상 동작을 확인했다.</li>
<li><code>useEffect(() =&gt; {...}, [])</code>로 감싸서, 페이지가 처음 열릴 때 딱 한 번만 현재 위치를 조회하고, 그 위치의 날씨를 자동으로 보여주도록 했다.</li>
<li>⚠️ 이 함수 이름을 <code>getCurrentLocation</code>이라고 지었는데, 정작 안에서 부르는 브라우저 API는 <code>getCurrentPosition</code>이다. 이름만 보면 헷갈릴 수 있는 부분이라 다음에 다시 볼 때 주의가 필요하다.</li>
</ul>
<hr />
<h2 id="4-weatherbutton--weatherbox--화면-표시를-맡는-자식-컴포넌트">4. WeatherButton / WeatherBox — 화면 표시를 맡는 자식 컴포넌트</h2>
<p><strong>JSX</strong> · <code>WeatherButton.jsx</code> — 도시 선택 버튼</p>
<pre><code class="language-jsx">import {Button} from 'react-bootstrap';

const WeatherButton = ({cities, city, handler}) =&gt; {
    return(
        &lt;div className='button-group'&gt;
            {
                cities.map((item, idx)=&gt;{
                    return(
                        &lt;Button key ={idx}
                                className={`btn ${city===item ? 'active' :'' }`}
                                onClick={ (e) =&gt; handler(e, item)}&gt;
                            {item}
                        &lt;/Button&gt;
                    )
                })
            }
        &lt;/div&gt;
    );
}

export default WeatherButton;</code></pre>
<p><strong>코드 리뷰</strong></p>
<ul>
<li><code>cities</code> 배열을 <code>map</code>으로 돌면서 버튼을 여러 개 찍어낸다. 오늘까지 계속 반복해서 보고 있는 &quot;부모가 배열을 내려주고, 자식이 <code>map</code>으로 그린다&quot;는 패턴이 여기서도 그대로다.</li>
<li><code>city===item ? 'active' : ''</code>로, 현재 선택된 도시와 이 버튼이 같으면 <code>active</code> 클래스를 붙여서 CSS로 강조 표시한다.</li>
<li>버튼 클릭 시 자기가 판단하지 않고 <code>handler(e, item)</code>으로 이벤트와 클릭한 도시 이름만 부모(<code>WeatherPage</code>)에 넘긴다. &quot;무엇을 클릭했는지&quot;만 알려주고 &quot;그래서 뭘 할지&quot;는 부모가 정하는 역할 분담이다.</li>
</ul>
<p><strong>JSX</strong> · <code>WeatherBox.jsx</code> — 날씨 데이터 표시</p>
<pre><code class="language-jsx">const WeatherBox = ({weather}) =&gt; {
    return(
        &lt;div className='weather-box'&gt;
            &lt;div className='weather-city'&gt;{weather.name} &lt;/div&gt;
            &lt;div className='weather-temp'&gt;
            {weather?.main?.temp
             ?(weather?.main?.temp - 273.15).toFixed(1)
             : &quot;로딩중&quot;
             }
            &lt;/div&gt;
            &lt;div className='weather-desc'&gt;
            {weather?.weather?.[0]?.description}
            &lt;/div&gt;
        &lt;/div&gt;
    );
}

export default WeatherBox;</code></pre>
<p><strong>코드 리뷰</strong></p>
<ul>
<li>처음엔 <code>weather.main.temp</code>처럼 Optional Chaining 없이 바로 접근했다가, 페이지가 처음 열려서 <code>weather</code>가 아직 빈 객체(<code>{}</code>)인 순간에 <code>Cannot read properties of undefined</code> 에러가 났다. <code>?.</code>를 연달아 붙여서(<code>weather?.main?.temp</code>) 값이 없으면 에러 대신 <code>undefined</code>가 되도록 고치고, 삼항 연산자로 그 자리에 <code>&quot;로딩중&quot;</code>을 대신 보여주게 했다.</li>
<li>⚠️ OpenWeather API가 돌려주는 온도 단위는 <strong>켈빈(K)</strong>이다. 처음엔 이걸 모르고 그대로 화면에 찍어서 &quot;300도&quot;짜리 날씨가 나오는 걸 보고 단위를 다시 확인했다. <code>- 273.15</code>로 섭씨로 변환하고 <code>.toFixed(1)</code>로 소수점 첫째 자리까지만 표시하도록 고쳤다.</li>
<li><code>weather?.weather?.[0]?.description</code>처럼 배열 인덱스(<code>[0]</code>)에도 Optional Chaining을 함께 쓸 수 있다는 걸 오늘 처음 써봤다. 응답 구조를 콘솔로 찍어보니 <code>weather</code> 필드 자체가 배열이라, <code>weather.weather.description</code>으로 접근했을 때는 계속 <code>undefined</code>가 나왔던 원인이 바로 이거였다.</li>
</ul>
<hr />
<h2 id="5-kakaomap--지도를-그리고-클릭-지점의-날씨를-받아오기">5. KaKaoMap — 지도를 그리고 클릭 지점의 날씨를 받아오기</h2>
<p><strong>JSX</strong> · <code>KaKaoMap.jsx</code> — 지도 초기화와 클릭 이벤트</p>
<pre><code class="language-jsx">import { useEffect, useRef } from &quot;react&quot;;

const KaKaoMap = ({ setWeatherByCoords, mapCenter }) =&gt; {

    const mapRef = useRef(null);
    const markerRef = useRef(null);

    useEffect(() =&gt; {
        window.kakao.maps.load(() =&gt; {
            navigator.geolocation.getCurrentPosition((position) =&gt; {
                let lat = position.coords.latitude;
                let lng = position.coords.longitude;

                const container = document.getElementById('map');

                const centerPosition = new window.kakao.maps.LatLng(37.5665, 126.9780);
                const map = new window.kakao.maps.Map(container, {
                    center: centerPosition,
                    level: 3
                });
                const marker = new window.kakao.maps.Marker({
                    position: centerPosition
                });
                marker.setMap(map);

                mapRef.current = map;
                markerRef.current = marker;

                window.kakao.maps.event.addListener(map, &quot;click&quot;, function (mouseEvent) {
                    const lat = mouseEvent.latLng.getLat();
                    const lng = mouseEvent.latLng.getLng();

                    marker.setPosition(new window.kakao.maps.LatLng(lat, lng));
                    setWeatherByCoords(lat, lng);
                });
            })
        });
    }, [])

    useEffect(() =&gt; {
        if (!mapCenter) return;
        if (!mapRef.current || !markerRef.current) return;

        const lat = parseFloat(mapCenter.lat);
        const lng = parseFloat(mapCenter.lng);

        const position = new window.kakao.maps.LatLng(lat, lng);

        mapRef.current.setCenter(position);
        markerRef.current.setPosition(position);
    }, [mapCenter]);

    return (
        &lt;div id='map' style={{ width: '100%', height: '400px' }}&gt;&lt;/div&gt;
    );
}

export default KaKaoMap;</code></pre>
<h3 id="왜-usestate가-아니라-useref를-썼는가">왜 useState가 아니라 useRef를 썼는가</h3>
<p>지도와 마커를 어디에 담을지 처음엔 <code>useState</code>로 시작했다. 그런데 도시 버튼을 눌러 지도 중심을 옮길 때마다 지도가 통째로 다시 생성되는 이상한 현상이 있었다. <strong>지도는 &quot;다시 그려야 하는 데이터&quot;가 아니라 &quot;이미 그려진 걸 이동시키기만 하면 되는 객체&quot;</strong>라는 걸 깨닫고 나서야 원인이 보였다. <code>useState</code>로 관리하면 값이 바뀔 때마다 컴포넌트가 리렌더링되고, 그 리렌더링 과정에서 지도 생성 로직이 다시 실행될 위험이 있었던 것이다.</p>
<p>그래서 <code>mapRef</code>, <code>markerRef</code>를 <code>useRef(null)</code>로 바꿨다. <code>useRef</code>로 관리하는 값은 바뀌어도 리렌더링을 유발하지 않기 때문에, 지도 인스턴스를 &quot;기억&quot;만 해두고 필요할 때 <code>.current</code>로 꺼내 쓰는 용도로 딱 맞았다.</p>
<p><strong>코드 리뷰</strong></p>
<ul>
<li>첫 번째 <code>useEffect</code>(의존성 배열 <code>[]</code>)는 <strong>최초 마운트 시 1번만</strong> 실행돼서 지도를 딱 한 번 생성한다. 지도를 생성한 뒤, 지도 위 클릭 이벤트를 등록해서 클릭한 지점의 좌표(<code>mouseEvent.latLng</code>)로 마커를 옮기고 <code>setWeatherByCoords(lat, lng)</code>(부모의 <code>getWeatherByCoords</code>)를 호출한다.</li>
<li>두 번째 <code>useEffect</code>(의존성 배열 <code>[mapCenter]</code>)는 <strong>도시 버튼을 눌러서 <code>mapCenter</code>가 바뀔 때마다</strong> 실행돼서, 이미 만들어진 지도의 중심과 마커 위치만 옮긴다. 지도를 새로 만드는 게 아니라 <code>mapRef.current.setCenter(...)</code>로 기존 지도 인스턴스를 그대로 재사용한다.</li>
<li><code>mapRef.current</code>가 아직 없을 수도 있는 시점(지도가 완전히 준비되기 전에 도시 버튼을 먼저 누르는 경우)을 대비해서 <code>if (!mapRef.current || !markerRef.current) return;</code>으로 방어 코드를 넣어뒀다. 이 방어 코드가 없었을 때, 페이지 로딩 직후 빠르게 버튼을 누르면 <code>Cannot read properties of null</code>류의 에러가 났었다.</li>
</ul>
<hr />
<h2 id="6-사용한-코드-개념-따로-정리">6. 사용한 코드 개념 따로 정리</h2>
<h3 id="6-1-useref란">6-1) <code>useRef</code>란?</h3>
<p><code>useState</code>가 &quot;값이 바뀌면 화면도 다시 그려야 하는 값&quot;을 관리한다면, <code>useRef</code>는 <strong>&quot;값이 바뀌어도 화면을 다시 그릴 필요는 없지만, 계속 기억은 해둬야 하는 값&quot;</strong>을 관리한다. <code>const ref = useRef(초기값)</code>으로 만들면 <code>ref.current</code>에 값을 넣고 꺼내 쓸 수 있고, 이 값은 컴포넌트가 여러 번 리렌더링돼도 사라지지 않는다.</p>
<table>
<thead>
<tr>
<th>구분</th>
<th><code>useState</code></th>
<th><code>useRef</code></th>
</tr>
</thead>
<tbody><tr>
<td>값이 바뀌면</td>
<td>리렌더링 발생</td>
<td>리렌더링 없음</td>
</tr>
<tr>
<td>주요 용도</td>
<td>화면에 표시할 데이터</td>
<td>DOM 요소 참조, 외부 라이브러리 인스턴스 보관 등</td>
</tr>
<tr>
<td>값 읽기/쓰기</td>
<td><code>state</code> / <code>setState(값)</code></td>
<td><code>ref.current</code> / <code>ref.current = 값</code></td>
</tr>
</tbody></table>
<h3 id="6-2-fetch-api란">6-2) <code>fetch</code> API란?</h3>
<p>브라우저에 내장된, 서버에 요청을 보내고 응답을 받는 함수다. axios처럼 별도 설치가 필요 없다는 게 큰 차이. 다만 axios와 달리 응답을 바로 JSON으로 안 주기 때문에, <code>.then(response =&gt; response.json())</code>처럼 한 단계를 더 거쳐야 실제 데이터를 꺼낼 수 있다.</p>
<h3 id="6-3-navigatorgeolocation이란">6-3) <code>navigator.geolocation</code>이란?</h3>
<p>브라우저가 기본으로 제공하는 <strong>위치 정보 API</strong>다. <code>getCurrentPosition(콜백함수)</code>를 호출하면, 브라우저가 사용자에게 위치 접근 권한을 물어보는 팝업을 띄우고, 허용하면 콜백함수의 <code>position.coords</code>에 위도(<code>latitude</code>)·경도(<code>longitude</code>)를 담아 돌려준다.</p>
<h3 id="6-4-put-vs-patch-헷갈렸던-점에서-이어지는-개념">6-4) PUT vs PATCH (헷갈렸던 점에서 이어지는 개념)</h3>
<p>둘 다 서버에 있는 기존 데이터를 &quot;수정&quot;할 때 쓰는 HTTP 메서드지만 범위가 다르다.</p>
<table>
<thead>
<tr>
<th>구분</th>
<th>PUT</th>
<th>PATCH</th>
</tr>
</thead>
<tbody><tr>
<td>수정 범위</td>
<td>리소스 <strong>전체</strong>를 새 값으로 교체</td>
<td>보낸 필드만 <strong>부분</strong> 수정</td>
</tr>
<tr>
<td>안 보낸 필드는?</td>
<td>사라지거나 초기화될 수 있음</td>
<td>그대로 유지됨</td>
</tr>
<tr>
<td>예시</td>
<td>사용자 정보 전체를 새 객체로 덮어씀</td>
<td>댓글 내용 하나만 수정</td>
</tr>
</tbody></table>
<p>→ 오늘 헷갈렸던 예시(<code>api.patch('/comments/:id', {comment: mention})</code>)처럼, 댓글 하나에서 <strong><code>comment</code> 필드 하나만</strong> 바꾸고 싶을 때는 <code>PUT</code>이 아니라 <code>PATCH</code>를 써야, 다른 필드(작성자 이메일 등)가 날아가지 않는다.</p>
<hr />
<h2 id="7-헷갈렸던-점-다음-복습-포인트-🤔">7. 헷갈렸던 점 (다음 복습 포인트) 🤔</h2>
<ul>
<li><strong><code>useEffect</code>, <code>useMemo</code>의 필요성</strong>: 오늘 실습에는 <code>useMemo</code>가 직접 등장하진 않았지만, <code>useEffect</code>가 여러 군데(마운트 시 위치 감지, <code>mapCenter</code> 변경 감지)에서 서로 다른 목적으로 쓰인 걸 보면서 &quot;이 <code>useEffect</code>가 정확히 언제, 왜 실행되는지&quot;를 하나씩 짚어봐야 했다.</li>
<li><strong>PUT vs PATCH</strong>: 위 6-4 표로 정리한 대로, &quot;전체 교체&quot;와 &quot;부분 수정&quot;이라는 목적 차이를 오늘 처음 명확히 구분했다.</li>
<li><strong><code>useRef</code></strong>: <code>useState</code>와 뭐가 다른지 헷갈렸는데, 지도 인스턴스가 리렌더링 때마다 다시 생성되는 문제를 직접 겪고 나서야 &quot;리렌더링을 유발하지 않으면서 값을 기억해두는 상자&quot;라는 개념이 왜 필요한지 체감했다.</li>
</ul>
<hr />
<h2 id="8-아직-남겨둔-것">8. 아직 남겨둔 것</h2>
<ul>
<li><strong>영문 도시명 지원</strong> — 카카오 Geocoder가 한글 지명에서만 안정적으로 동작해서, 영문 도시명(<code>&quot;Seoul, KR&quot;</code>)을 다시 지원하려면 별도 처리(예: OpenWeather의 도시명 검색을 그대로 쓰고 좌표만 따로 넘기는 방식)가 더 필요해 보인다.</li>
<li><strong><code>KaKaoMap.jsx</code>의 <code>window.kakao.maps.load((setWeatherByCoords, moveTo) =&gt; {...})</code> 부분</strong> — <code>kakao.maps.load</code>의 콜백 함수는 원래 인자를 받지 않는데, 여기서는 <code>(setWeatherByCoords, moveTo)</code>처럼 매개변수를 적어뒀다. 실제로는 아무 값도 안 넘어오기 때문에 이 두 매개변수는 항상 <code>undefined</code>가 되고, 지금 코드에서도 실제로는 쓰이지 않고 있어 정리가 필요해 보인다.</li>
<li><strong><code>getCurrentLocation</code> 함수명</strong> — 실제로 호출하는 브라우저 API는 <code>getCurrentPosition</code>인데 함수 이름은 <code>getCurrentLocation</code>으로 지어서, 나중에 다시 볼 때 헷갈릴 수 있는 부분이다.</li>
</ul>
<hr />
<h2 id="9-다음에-할-일">9. 다음에 할 일</h2>
<ul>
<li><input disabled="" type="checkbox" /> <code>getCurrentLocation</code> 함수명을 실제 동작에 맞게 정리하거나 주석으로 보완하기</li>
<li><input disabled="" type="checkbox" /> <code>KaKaoMap.jsx</code>의 불필요한 콜백 매개변수 정리하기</li>
<li><input disabled="" type="checkbox" /> 카카오 API 설정 방법 다시 읽어보기 (<a href="https://apis.map.kakao.com/web/guide/">https://apis.map.kakao.com/web/guide/</a>)</li>
</ul>
<hr />
<h2 id="오늘의-한-줄-요약">오늘의 한 줄 요약</h2>
<p>OpenWeatherMap과 Kakao Map을 연동해, 버튼 클릭·지도 클릭·현재 위치 감지라는 세 가지 경로로 날씨 데이터를 가져와 화면에 반영하는 흐름을 만들어봤고, 지도가 리렌더링될 때마다 다시 생성되는 문제를 직접 겪은 뒤에야 <code>useRef</code>로 지도 인스턴스를 유지하는 이유를 제대로 이해한 날이었다.</p>
<hr />
<h2 id="마무리-회고">마무리 회고</h2>
<p>오늘은 처음으로 &quot;우리 서버(json-server)&quot;가 아니라 <strong>진짜 바깥 세상의 API</strong>를 붙여봤다는 점에서 느낌이 달랐다. 도시 이름을 눌렀는데 그 도시의 실제 지금 날씨가 뜨는 걸 보니, 지금까지 만든 게 &quot;화면 안에서만 도는 앱&quot;이 아니라 실제로 외부와 연결된 앱이 될 수 있다는 감각이 생겼다.</p>
<p>돌아보면 오늘 겪은 문제들은 대부분 &quot;값의 형식을 확인하지 않고 그냥 될 거라 가정&quot;해서 생긴 것들이었다. 좌표가 문자열로 오는 걸 몰라서 <code>parseFloat</code>를 빠뜨렸고, 온도가 켈빈 단위인 걸 몰라서 300도짜리 날씨를 봤고, 응답의 <code>weather</code> 필드가 배열인 걸 몰라서 계속 <code>undefined</code>를 받았다. 세 번 다 콘솔에 데이터를 직접 찍어보고 나서야 원인을 알 수 있었다.</p>
<p><code>useRef</code>도 마찬가지였다. 지도가 계속 다시 그려지는 문제를 실제로 겪고 나서야 &quot;리렌더링이 필요 없는 값도 있다&quot;는 개념이 몸으로 와닿았다. 내일은 오늘 미뤄둔 부분(영문 도시명, 불필요한 콜백 매개변수)부터 정리하고 넘어가야겠다.</p>