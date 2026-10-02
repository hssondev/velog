<blockquote>
<p><strong>한 줄 소개</strong></p>
<p>서울시 문화행사 공공데이터와 TourAPI 맛집 데이터를 결합해, 행사 시간에 맞춰 주변 식사·카페까지 묶은 하루 일정을 AI로 추천하는 웹 서비스. 6인 팀 프로젝트에서 <strong>문화행사 데이터 수집 배치, 행사 상세/관심 행사 API, 공통 예외 처리, DB 설계</strong>를 맡았다.</p>
</blockquote>
<ul>
<li><strong>기간</strong>: 2026.09.22 ~ 2026.09.29 (팀 전체 일정은 09.21~09.30, 개인 작업은 약 1주일)</li>
<li><strong>인원</strong>: 6인 (PM 2, 백엔드/프론트 역할 분담)</li>
<li><strong>리포지토리</strong>: <a href="https://github.com/hssondev/MINI_PROJECT_1_Local_Culture_Tour_Guide">github.com/hssondev/MINI_PROJECT_1_Local_Culture_Tour_Guide</a></li>
<li><strong>내 역할</strong>: 백엔드 — 데이터 수집 배치, 행사 도메인 API, 공통 예외 처리, DB 설계</li>
</ul>
<hr />
<h2 id="1-프로젝트-소개">1. 프로젝트 소개</h2>
<p>여행 일정을 짤 때 보통은 관광지부터 정하고 그 근처에서 할 일을 찾는다. 하지만 전시·공연 같은 문화행사는 날짜와 시간이 고정되어 있어서 순서가 반대가 되어야 한다. 행사 시간을 먼저 잡고, 그 앞뒤로 식사·카페를 맞춰 넣어야 하는 것이다. TripAI는 이 순서를 그대로 제품으로 만들었다.</p>
<pre><code>날짜·조건 입력(행사 분야/지역/무료 여부, 이동수단, 식사 취향)
        ↓
서버가 조건에 맞는 행사와 방문일을 1차 결정
        ↓
규칙 기반으로 &quot;항상 동작하는&quot; 기본 일정 생성
        ↓
AI(OpenAI)가 만든 일정이 서버 검사를 통과하면 그걸로 교체
        ↓
타임라인 + 지도 동선으로 결과 제공 (편집·재추천 가능)</code></pre><p>AI 응답이 늦거나 틀려도 사용자에게는 항상 규칙 기반으로 만든 유효한 일정이 나가도록 설계되어 있다. AI는 &quot;있으면 더 좋은&quot; 보조 수단이고, 핵심 로직은 서버가 검증한다.</p>
<hr />
<h2 id="2-기술-스택">2. 기술 스택</h2>
<table>
<thead>
<tr>
<th>구분</th>
<th>기술</th>
</tr>
</thead>
<tbody><tr>
<td>백엔드 (담당)</td>
<td>Java 17, Spring Boot 3.5, MyBatis</td>
</tr>
<tr>
<td>백엔드 (팀원 담당)</td>
<td>Spring Security, JWT</td>
</tr>
<tr>
<td>프론트엔드 (팀원 담당)</td>
<td>React 18, Vite 5, 카카오 지도 SDK</td>
</tr>
<tr>
<td>데이터베이스 (담당)</td>
<td>MariaDB</td>
</tr>
<tr>
<td>외부 API (담당)</td>
<td>서울시 문화행사 OpenAPI</td>
</tr>
<tr>
<td>외부 API (팀원 담당)</td>
<td>한국관광공사 TourAPI, OpenAI API</td>
</tr>
<tr>
<td>협업</td>
<td>GitHub(이슈·PR 템플릿, 브랜치 전략), Figma</td>
</tr>
</tbody></table>
<hr />
<h2 id="3-시스템-구성">3. 시스템 구성</h2>
<pre><code>사용자 브라우저
    ↓ (REST API + JWT)
React (Vite)  ──────────────→  카카오 지도 SDK
    ↓
Spring Boot 서버
    ├─ MyBatis ──→ MariaDB
    ├─ 매일 03:00 스케줄러 ──→ 서울시 문화행사 OpenAPI 수집   ← 내가 만든 부분
    └─ 일정 생성 시 ──→ OpenAI API 호출
한국관광공사 TourAPI ──(사전 수집, 시드로 적재)──→ MariaDB</code></pre><p>문화행사는 매일 새벽 3시 배치로 서울시 OpenAPI에서 수집하고, 음식점·카페는 TourAPI에서 미리 수집해 영업시간·브레이크타임까지 정리한 시드로 넣어 둔다. 팀 전체 기준 테이블 7개(<code>users</code>, <code>event</code>, <code>place</code>, <code>favorite_event</code>, <code>trip_plan</code>, <code>trip_plan_interest</code>, <code>trip_item</code>)로 구성된 통합 DDL이 최종 산출물이다.</p>
<hr />
<h2 id="4-내가-맡은-부분">4. 내가 맡은 부분</h2>
<pre><code>문화행사 수집 배치 (매일 03:00, 온라인 행사 숨김)
        ↓
행사 상세 조회 API
        ↓
관심 행사(즐겨찾기) 저장 · 해제 · 목록 조회
        ↓
행사 상세 → &quot;AI 일정 만들기&quot; / &quot;저장된 일정에 행사 추가&quot; 흐름 연결</code></pre><p>그 외에 팀 공통으로 쓰는 예외 처리 체계(<code>ErrorCode</code>/<code>CustomException</code>/<code>GlobalExceptionHandler</code>)와 최종 통합 DDL(v3.2.5), 와이어프레임 작성도 담당했다.</p>
<h3 id="4-1-서울시-문화행사-수집-배치">4-1. 서울시 문화행사 수집 배치</h3>
<p>서울시가 제공하는 오픈API는 날짜·시간·좌표가 전부 &quot;문자열&quot;로 내려온다. 이걸 그대로 쓰면 날짜 비교도, 거리 계산도 할 수 없기 때문에, API 응답을 우리 DB 스키마에 맞는 형태로 변환해서 저장하는 배치를 만들었다.</p>
<pre><code class="language-java">@Scheduled(cron = &quot;0 0 3 * * *&quot;, zone = &quot;Asia/Seoul&quot;)
public void runNightlyBatch() {
    eventBatchService.collectAndSave();
}</code></pre>
<p>매일 새벽 3시(한국 시간)에 전체 행사를 다시 받아와서 upsert(있으면 수정, 없으면 삽입)하는 구조다. <code>collectAndSave()</code> 내부 흐름은 이렇다.</p>
<pre><code class="language-java">public void collectAndSave() {
    List&lt;SeoulEventApiItem&gt; items = apiClient.fetchAll();   // ① 네트워크 호출은 트랜잭션 밖에서
    Stats stats = new Stats();
    transactionTemplate.executeWithoutResult(status -&gt; {    // ② 변환·저장만 트랜잭션 안에서
        for (SeoulEventApiItem item : items) {
            saveOne(item, stats);
        }
    });
}</code></pre>
<p>①에서 네트워크 호출을 트랜잭션 밖에 둔 이유는, API 응답을 기다리는 동안 DB 커넥션을 붙잡고 있지 않기 위해서다. DB 커넥션 풀은 한정된 자원이라, 느린 외부 호출과 묶어 두면 다른 요청이 커넥션을 못 받는 문제가 생길 수 있다. 대신 변환과 저장만 트랜잭션으로 묶어서, 한 건 처리 중 예외가 나도 <code>saveOne</code> 안에서 잡아서 그 건만 건너뛰고(<code>stats.skipped++</code>) 전체 배치는 계속 돌도록 했다.</p>
<pre><code class="language-java">private void saveOne(SeoulEventApiItem item, Stats stats) {
    try {
        // ... 변환 후 저장 ...
    } catch (Exception e) {
        stats.skipped++;
        log.warn(&quot;적재 실패, 건너뜀: {}&quot;, item.getTitle());
        // 한 건이 실패해도 배치 전체는 계속한다
    }
}</code></pre>
<p>약 19,000건 중 일부 데이터가 형식이 깨져 있어도 그 한 건만 건너뛰고 나머지는 정상 저장되도록 한 것이 포인트다.</p>
<p><strong>좌표 보정</strong> — 서울시 API의 위도·경도 값 중 일부가 뒤바뀌어 들어오는 경우가 있었다(위도 자리에 경도 값이 들어오는 식).</p>
<pre><code class="language-java">private boolean inKorea(BigDecimal lat, BigDecimal lng) {
    return lat.doubleValue() &gt;= 33 &amp;&amp; lat.doubleValue() &lt;= 39
            &amp;&amp; lng.doubleValue() &gt;= 124 &amp;&amp; lng.doubleValue() &lt;= 132;
}</code></pre>
<p>받은 좌표가 한국 범위를 벗어나면 위·경도를 서로 바꿔서 다시 검사하고, 그래도 범위 밖이면 좌표 없이 저장하도록 보정 로직을 넣었다. 좌표가 없으면 지도에는 안 뜨지만 행사 정보 자체는 버리지 않는 쪽을 선택한 것이다.</p>
<p><strong>온라인 행사 숨김 처리</strong> — 서울시 API에는 온라인 여부를 알려주는 필드가 따로 없어서, 제목·장소 텍스트에 &quot;온라인&quot;, &quot;비대면&quot;, &quot;유튜브&quot;, &quot;zoom&quot; 같은 키워드가 있으면 숨기되, &quot;오프라인&quot;, &quot;현장&quot;, &quot;병행&quot; 같은 단어가 같이 있으면 숨기지 않도록 규칙을 짰다. 온라인+오프라인 병행 행사는 현장 참여가 가능하기 때문이다.</p>
<pre><code class="language-java">private boolean isOnline(SeoulEventApiItem item) {
    String text = (safe(item.getTitle()) + &quot; &quot; + safe(item.getPlace())).toLowerCase();
    boolean online = ONLINE_KEYWORDS.stream().anyMatch(text::contains);
    boolean hybrid = HYBRID_KEYWORDS.stream().anyMatch(text::contains);
    return online &amp;&amp; !hybrid;
}</code></pre>
<h3 id="4-2-행사-상세-조회-api">4-2. 행사 상세 조회 API</h3>
<pre><code class="language-java">@GetMapping(&quot;/{eventId}&quot;)
public ApiResponse&lt;EventDetailResponse&gt; getEventDetail(
        @AuthenticationPrincipal Long userId,
        @PathVariable String eventId
) {
    return ApiResponse.success(eventService.getEventDetail(eventId, userId));
}</code></pre>
<p>행사 ID로 상세 정보를 조회하는 API인데, <code>@AuthenticationPrincipal</code>로 로그인한 사용자 ID도 같이 받는 게 포인트다. 단순히 행사 정보만 내려주는 게 아니라, <code>favorited</code> 필드에 &quot;이 사용자가 이 행사를 관심 행사로 저장해 뒀는지&quot;까지 한 번의 쿼리로 같이 내려주기 때문이다. 이렇게 하면 프론트에서 행사 상세 화면을 그릴 때 하트 아이콘의 초기 상태(채워짐/비어있음)를 추가 API 호출 없이 바로 표시할 수 있다.</p>
<p>없는 행사 ID로 조회하면 <code>CustomException(ErrorCode.EVENT_NOT_FOUND)</code>를 던지도록 해서, 404와 함께 일관된 에러 메시지가 내려가게 했다.</p>
<h3 id="4-3-관심-행사즐겨찾기-저장·해제·목록">4-3. 관심 행사(즐겨찾기) 저장·해제·목록</h3>
<pre><code class="language-java">public void addFavoriteEvent(Long userId, String eventContentId) {
    if (userId == null) throw new CustomException(ErrorCode.UNAUTHORIZED);
    if (eventContentId == null || eventContentId.isBlank())
        throw new CustomException(ErrorCode.INVALID_INPUT);
    if (!favoriteEventMapper.existsEvent(eventContentId))
        throw new CustomException(ErrorCode.EVENT_NOT_FOUND);
    try {
        if (favoriteEventMapper.insertFavorite(userId, eventContentId) == 0) {
            throw new CustomException(ErrorCode.ALREADY_FAVORITED);
        }
    } catch (DuplicateKeyException exception) {
        // 동시에 저장한 요청도 DB의 복합 PK와 같은 409 응답으로 처리한다.
        throw new CustomException(ErrorCode.ALREADY_FAVORITED);
    }
}</code></pre>
<p><code>favorite_event</code> 테이블의 PK를 <code>(user_id, event_content_id)</code> 복합키로 잡아 뒀는데, 이 덕분에 같은 사용자가 같은 행사를 두 번 저장하려고 하면 DB 레벨에서 <code>DuplicateKeyException</code>이 터진다. 애플리케이션 코드에서 &quot;이미 저장했는지&quot;를 먼저 조회로 확인하고 저장하는 방식(조회 → 저장 2단계)은 두 요청이 거의 동시에 들어오면 둘 다 조회 시점엔 &quot;없음&quot;으로 보여서 중복 저장이 생길 수 있는데(동시성 문제), DB 제약으로 막아 두면 이런 경우까지 자연스럽게 처리된다. 그래서 <code>insertFavorite</code>가 0건이면 애플리케이션 로직상 중복이고, <code>DuplicateKeyException</code>이 나면 동시 요청으로 인한 중복이라 둘 다 같은 <code>ALREADY_FAVORITED</code>(409) 응답으로 묶었다.</p>
<h3 id="4-4-행사-상세-→-일정-생성-흐름-연결">4-4. 행사 상세 → 일정 생성 흐름 연결</h3>
<p>행사 상세 화면에서 &quot;이 행사로 AI 일정 만들기&quot;를 누르면 일정 추천 화면으로, &quot;저장된 일정에 추가&quot;를 누르면 이미 저장해 둔 일정에 이 행사를 끼워 넣는 흐름을 프론트(<code>EventCreateModal</code>, <code>EventSavedPlanModal</code>, <code>EventSavedPlanPage</code>)와 백엔드 양쪽에서 연결했다. 이 부분은 작업 중 방향을 한 번 바꾼 지점이기도 하다(아래 트러블슈팅 참고).</p>
<h3 id="4-5-공통-예외-처리">4-5. 공통 예외 처리</h3>
<p>팀 전체가 같은 형태의 에러 응답을 쓸 수 있도록 <code>ErrorCode</code>(enum) → <code>CustomException</code>(런타임 예외) → <code>GlobalExceptionHandler</code>(<code>@RestControllerAdvice</code>) 3단 구조를 만들었다.</p>
<pre><code class="language-java">public enum ErrorCode {
    EVENT_NOT_FOUND(HttpStatus.NOT_FOUND, &quot;존재하지 않는 행사입니다.&quot;),
    ALREADY_FAVORITED(HttpStatus.CONFLICT, &quot;이미 저장된 행사입니다.&quot;),
    // ...
    private final HttpStatus status;
    private final String message;
}</code></pre>
<p>서비스 코드에서는 <code>throw new CustomException(ErrorCode.XXX)</code> 한 줄이면 되고, HTTP 상태 코드와 메시지는 <code>ErrorCode</code>에만 정의해 두면 <code>GlobalExceptionHandler</code>가 알아서 일관된 JSON 응답으로 바꿔 준다. 덕분에 팀원 6명이 각자 다른 화면·API를 만들면서도 에러 응답 형식이 흩어지지 않았다.</p>
<h3 id="4-6-데이터베이스-설계">4-6. 데이터베이스 설계</h3>
<p>테이블 7개로 구성된 통합 DDL(v3.2.5)을 최종 정리해서 올렸다.</p>
<ul>
<li><code>trip_item</code>(일정 항목)이 &quot;행사&quot;일 수도 &quot;장소&quot;일 수도 있어서, <code>event_content_id</code>/<code>place_content_id</code> 두 컬럼을 두고 <code>item_type</code>으로 구분하게 했다. 원래는 &quot;둘 중 하나만 값이 있어야 한다&quot;는 규칙을 DB <code>CHECK</code> 제약으로 걸려고 했는데, MariaDB 일부 버전에서 복수 컬럼을 참조하는 이 형태의 CHECK가 에러(ERROR 1901)를 내서 DB 레벨 제약은 포기하고 서비스/DTO 검증으로 옮겼다. 이 결정과 이유는 DDL 주석에 남겨 뒀다.</li>
<li><code>favorite_event</code>, <code>trip_plan_interest</code>는 식별 관계(부모 PK를 자기 PK의 일부로 갖는 관계)로 설계해서, 부모 없이는 존재할 수 없는 종속적인 데이터임을 스키마 자체로 드러냈다.</li>
<li>검색 패턴에 맞춰 인덱스를 미리 설계했다. 예: <code>IX_EVENT_SEARCH (display_yn, district_name, event_type)</code>는 행사 목록 조회, <code>IX_PLACE_COORD (mapy, mapx)</code>는 반경 검색을 염두에 둔 것이다.</li>
</ul>
<hr />
<h2 id="5-트러블슈팅">5. 트러블슈팅</h2>
<h3 id="5-1-행사-전용-일정-api를-만들었다가-팀-공통-api로-갈아탄-일">5-1. 행사 전용 일정 API를 만들었다가, 팀 공통 API로 갈아탄 일</h3>
<ul>
<li><strong>상황</strong>: &quot;이 행사로 AI 일정 만들기&quot; 기능을 위해, 처음엔 <code>EventPlanController</code>/<code>EventPlanService</code>/<code>EventPlanMapper</code>를 새로 만들어서 행사 전용 일정 초안 생성 로직을 통째로 구현했다.</li>
<li><strong>문제</strong>: 구현을 마치고 보니, 같은 주에 다른 팀원이 만들고 있던 공통 일정 추천 API(<code>PlanRecommendController</code> 등)와 로직이 거의 겹쳤다. 규칙 기반 일정 생성, 후보 맛집 조회, AI 검증 같은 핵심 로직을 두 군데서 따로 유지하게 되는 상황이었다.</li>
<li><strong>해결</strong>: 직접 만든 코드의 중복 로직(약 190줄)을 걷어내고, 행사 상세 화면에서도 팀 공통 일정 추천 API를 그대로 호출하도록 프론트(<code>EventCreateModal</code>, <code>planService.js</code>)를 수정했다.</li>
<li><strong>배운 점</strong>: 기능 단위로 브랜치를 나눠 병렬로 작업하다 보면 비슷한 로직이 중복 구현되기 쉽다는 걸 체감했다. 이후로는 새 API를 만들기 전에 팀 공통 API 목록(API 계약 문서)을 먼저 확인하는 습관을 들이게 됐다.</li>
</ul>
<h3 id="5-2-배치-통계-숫자가-실제-저장-결과와-어긋난-버그">5-2. 배치 통계 숫자가 실제 저장 결과와 어긋난 버그</h3>
<ul>
<li><p><strong>상황</strong>: 문화행사 수집 배치 로그에 &quot;저장 N건, 온라인 숨김 M건&quot;을 남기도록 만들었는데, <code>isOnline()</code> 판정 직후 바로 <code>stats.hiddenOnline++</code>을 올리고 있었다.</p>
</li>
<li><p><strong>문제</strong>: 온라인으로 판정된 행사라도 이후 유효성 검증(<code>validate()</code>)에서 걸러지거나 저장 중 예외가 나서 실제로는 DB에 반영되지 않는 경우가 있는데, 이런 건도 전부 &quot;숨김 처리됨&quot;으로 집계되고 있었다. 즉 로그의 숫자가 실제 DB 상태를 보장하지 못하는 상태였다.</p>
</li>
<li><p><strong>해결</strong>: 카운트 위치를 <code>eventBatchMapper.upsertEvent(row)</code>가 성공한 <strong>이후</strong>로 옮겨서, 실제로 저장에 성공한 행사 중 <code>display_yn = false</code>인 것만 세도록 고쳤다.</p>
<pre><code class="language-java">eventBatchMapper.upsertEvent(row);
stats.saved++;
if (Boolean.FALSE.equals(row.getDisplayYn())) {
    stats.hiddenOnline++;   // 저장 성공 이후에만 집계
}</code></pre>
</li>
<li><p><strong>배운 점</strong>: 로그·통계용 카운터라도 &quot;언제 올릴지&quot; 위치를 잘못 잡으면 실제 데이터와 어긋나는 숫자를 보게 된다는 걸 겪었다. 이후로는 카운터를 추가할 때 &quot;이 값이 어떤 시점의 성공을 의미해야 하는가&quot;를 먼저 따져보게 됐다.</p>
</li>
</ul>
<h3 id="5-3-테스트용으로-열어-둔-엔드포인트-정리">5-3. 테스트용으로 열어 둔 엔드포인트 정리</h3>
<p>개발 중 배치를 수동으로 실행해 보려고 <code>EventBatchTestController</code>를 임시로 만들어 뒀는데, 기능 검증이 끝난 뒤 이 컨트롤러를 그대로 두면 운영 환경에서 누구나 수집 배치를 수동으로 트리거할 수 있는 엔드포인트가 노출된 채로 남는 상황이었다. PR 리뷰 전에 직접 찾아서 제거했다.</p>
<hr />
<h2 id="6-성과-및-회고">6. 성과 및 회고</h2>
<ul>
<li>서울시 문화행사 약 19,000건을 매일 자동 수집·갱신하는 배치를 처음부터 끝까지(외부 API 연동 → 데이터 정제/보정 → 스케줄링) 혼자 설계하고 구현했다.</li>
<li>공통 예외 처리 체계를 팀에서 가장 먼저 만들어서 제공한 덕분에, 이후 팀원들이 각자 API를 만들 때 바로 가져다 쓸 수 있었다.</li>
<li>복합 PK, 식별/비식별 관계, DB 레벨 제약의 한계(MariaDB CHECK 이슈)까지 고려해서 통합 DDL을 최종 정리하면서, &quot;이론으로 배운 설계&quot;와 &quot;실제 DBMS에서 동작하는 설계&quot;의 차이를 직접 겪었다.</li>
<li>일정 추천처럼 여러 팀원이 동시에 건드리는 핵심 로직은 섣불리 새로 만들기보다, 먼저 공통 API 유무를 확인하는 게 전체 코드 품질에 더 낫다는 걸 트러블슈팅을 통해 배웠다.</li>
</ul>
<hr />
<h2 id="7-다른-팀원들이-작업한-기술-학습-메모">7. 다른 팀원들이 작업한 기술 (학습 메모)</h2>
<p>내가 직접 짠 코드는 아니지만, 코드를 읽으면서 &quot;이건 나중에 써먹어 봐야겠다&quot; 싶었던 부분을 정리해 둔다. 포트폴리오에는 안 넣더라도, 개인 복습용으로 남겨둔다.</p>
<h3 id="로그인jwt-tourapi-음식점-수집">로그인/JWT, TourAPI 음식점 수집</h3>
<p>로그인 토큰(JWT)을 만들고 검증하는 <code>JwtTokenProvider</code>를 담당했다. 눈여겨볼 부분은 비밀키 처리 방식이다.</p>
<pre><code class="language-java">public JwtTokenProvider(
        @Value(&quot;${jwt.secret}&quot;) String secret,
        @Value(&quot;${jwt.expiration}&quot;) long accessTokenExpirationMs
) {
    this.key = Keys.hmacShaKeyFor(sha256(secret));   // secret 길이와 상관없이 SHA-256으로 정규화
}</code></pre>
<p>JWT 서명에 쓰는 HS256 알고리즘은 키 길이에 최소 기준이 있는데, <code>.env</code>에 설정한 <code>jwt.secret</code> 문자열이 짧으면 바로 에러가 난다. 이 문제를 <code>secret</code> 값을 SHA-256으로 한 번 해시해서 항상 고정 길이(32바이트)로 만든 다음 키로 쓰는 방식으로 해결해 뒀다. <strong>배울 점</strong>: 설정값을 그대로 쓰지 않고 가공해서 &quot;항상 유효한 형태&quot;로 만들어 두면 운영 설정 실수를 줄일 수 있다는 패턴.</p>
<p>TourAPI에서 받아온 음식점 영업시간은 자유 형식 텍스트(<code>&quot;10:00~22:00(브레이크 15:00~17:00)&quot;</code> 같은)라서, 정규식으로 영업시간과 브레이크타임을 따로 뽑아내는 파서를 만들었다.</p>
<pre><code class="language-java">private static final Pattern TIME_RANGE = Pattern.compile(
    &quot;(?&lt;!\\d)([01]?\\d|2[0-3]|24):([0-5]\\d)\\s*(?:~|∼|-|–)\\s*([01]?\\d|2[0-3]|24):([0-5]\\d)(?!\\d)&quot;
);</code></pre>
<p>파싱이 안 되는 형식은 억지로 맞추지 않고 원문만 보존한 뒤 추천 후보에서 제외하는 방식도 참고할 만하다. <strong>배울 점</strong>: 외부 데이터는 전부 파싱 가능하다고 가정하지 말고, &quot;못 읽으면 안전하게 빼는&quot; 전략을 기본값으로 둘 것.</p>
<h3 id="마이페이지-api-ai-프롬프트-설계">마이페이지 API, AI 프롬프트 설계</h3>
<p>AI에게 일정을 만들어 달라고 요청하는 프롬프트를 설계했다. 그냥 &quot;일정 짜줘&quot;가 아니라, 지켜야 할 규칙을 번호를 매겨 명시하는 방식이다.</p>
<pre><code class="language-java">private static final String PLAN_GENERATION_PROMPT = &quot;&quot;&quot;
    당신은 서울 문화행사를 기준으로 당일 여행 일정을 짜는 도우미입니다.
    아래 INPUT_JSON에는 사용자의 조건과 서버가 미리 고른 후보가 들어 있습니다.
    오직 INPUT_JSON에 있는 후보만 사용해 하루 일정을 만들고, 결과는 JSON으로만 답하세요.

    [필수 규칙]
    1. 행사는 userConditions.requiredEventId와 eventContentId가 같은 행사 1개만 넣습니다.
    2. ...
    &quot;&quot;&quot;;</code></pre>
<p><strong>배울 점</strong>: AI가 &quot;서버가 미리 골라준 후보 안에서만&quot; 답하도록 input 자체를 제한해 두면, AI가 없는 장소를 지어내는 환각(hallucination) 문제를 프롬프트 단계에서부터 줄일 수 있다. 조건을 자연어 문단이 아니라 번호 매긴 규칙으로 쪼개 쓴 것도, 나중에 Spring AI/OpenAI 프롬프트 짤 때 참고할 만하다.</p>
<h3 id="일정-추천-엔진-ai규칙-기반-분리-설계">일정 추천 엔진, AI/규칙 기반 분리 설계</h3>
<p>이 프로젝트에서 가장 핵심이 되는 설계 하나가 <code>AiPlanClient</code>라는 인터페이스다.</p>
<pre><code class="language-java">public interface AiPlanClient {
    boolean isEnabled();                      // API 키가 있어서 AI 호출이 가능한지
    AiPlanResponse generate(AiPlanInput input);
}</code></pre>
<p>일정 추천 로직(<code>PlanRecommendService</code>)은 이 인터페이스만 보고 동작한다. 실제 구현체(<code>ApiPlanService</code>, OpenAI 호출)가 키가 없거나 호출에 실패하면, 호출한 쪽은 몰라도 되고 그냥 규칙 기반 추천으로 넘어가도록 되어 있다. <strong>배울 점</strong>: &quot;AI가 있으면 쓰고, 없으면 대체 로직으로&quot; 같은 분기 처리를 서비스 코드 여기저기에 if문으로 흩어 두지 않고, 인터페이스 하나로 추상화해 둔 것. 나중에 AI 제공자를 OpenAI에서 다른 걸로 바꾸더라도 이 인터페이스 구현체만 바꾸면 된다는 점도 눈여겨볼 설계다.</p>
<p>카카오 지도 SDK를 비동기로 한 번만 불러오는 로더도 참고할 만하다.</p>
<pre><code class="language-js">let loading = null
export function loadKakaoMaps() {
  if (window.kakao?.maps?.LatLng) return Promise.resolve(window.kakao)  // 이미 로드됐으면 바로 반환
  if (loading) return loading                                          // 로딩 중이면 그 Promise를 재사용
  loading = new Promise((resolve, reject) =&gt; { /* &lt;script&gt; 태그 동적 삽입 */ })
  return loading
}</code></pre>
<p><strong>배울 점</strong>: 외부 SDK를 여러 컴포넌트에서 동시에 요청해도 <code>&lt;script&gt;</code> 태그가 중복으로 삽입되지 않도록, 진행 중인 로딩 Promise를 모듈 스코프 변수에 캐싱해 두는 패턴.</p>
<hr />
<h2 id="다음에-할-일-포트폴리오-보완용-메모">다음에 할 일 (포트폴리오 보완용 메모)</h2>
<ul>
<li><input disabled="" type="checkbox" /> 각 API에 대한 요청/응답 예시와 에러 케이스를 캡처해서 포트폴리오 이미지로 추가</li>
<li><input disabled="" type="checkbox" /> &quot;온라인 행사 숨김&quot; 로직처럼 키워드 기반 휴리스틱의 한계(오탐 가능성)와 개선 방향을 한 문단으로 정리</li>
<li><input disabled="" type="checkbox" /> 자소서/면접용으로 5-1, 5-2 트러블슈팅을 STAR 포맷으로 더 간결하게 다듬기 (<code>mms-migration-star.md</code>와 같은 톤)</li>
</ul>