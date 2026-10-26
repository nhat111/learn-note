# Phỏng vấn Java + SQL, bám theo 2 project

Câu hỏi lấy từ code thật:
- **korean-practice** `backend/`: Java 21, Spring Boot 3.5, Anthropic Java SDK. Không có DB, chỉ lưu file JSON.
- **crypto-signal**: TypeScript, nhưng **PostgreSQL 16** với 42 migration và mọi câu SQL nằm ở `packages/db/src`.

Nên khi phỏng vấn: **Java/Spring lấy ví dụ từ korean-practice**, **SQL lấy ví dụ từ crypto-signal**.
Câu "nếu viết lại bằng Spring/JPA thì sao" (phần C) là cầu nối giữa hai bên.

Cách dùng: đọc câu hỏi, **tự trả lời thành tiếng trước**, rồi mới mở phần gợi ý. Câu nào trả lời không trôi thì đánh dấu ⚠️ và ôn lại cuối tuần.

---

## Plan ôn (chạy song song plan 6 tuần)

Khoảng 2–3 tiếng/tuần, lấy từ phần *PV* và *Java* của [plan 6 tuần](plan-den-5-12.md).

| Tuần | Java / Spring | SQL | Xong khi |
|---|---|---|---|
| 1 (26/10–1/11) | A1–A6: Java core qua code korean-practice | S1–S5: schema, constraint, index | Trả lời trôi 10/11 câu, không mở code |
| 2 (2–8/11) | B1–B8: Spring (filter, config, exception, test) | S6–S10: upsert, `DISTINCT ON`, partial index | Tự viết lại `WebConfig` + `AccessKeyFilter` trên giấy |
| 3 (9–15/11) | A7–A10: concurrency, virtual threads | Bài tập SQL 1–4 trên schema thật | 4 bài chạy đúng trên Postgres local (`docker compose up postgres`) |
| 4 (16–22/11) | B9–B12: timeout, retry, lỗi production | S11–S16: transaction, lock, migration, retention | Giải thích được migration 008 và advisory lock trong 2 phút |
| 5 (23–29/11) | C1–C8: JPA, `@Transactional`, N+1 | Bài tập SQL 5–8 + `EXPLAIN ANALYZE` | Đọc được 1 query plan, chỉ ra index nào được dùng |
| 6 (30/11–5/12) | Mock: trộn ngẫu nhiên 15 câu | Mock: 2 bài SQL viết tay trong 15 phút | Mock #3 không có câu nào ⚠️ |

**Mỗi buổi 30–45 phút:** 5 câu hỏi (nói thành tiếng, có hẹn giờ 2 phút/câu) → mở gợi ý → ghi chỗ sai vào bảng ⚠️ cuối file.
Có thể dán cả phần câu hỏi cho Claude và bảo: *"hỏi mình từng câu, đừng đưa đáp án, hỏi xoáy tiếp khi mình trả lời"*.

---

## A. Java core (qua korean-practice)

**A1. `AppProperties`, `AiRequest`, `Correction` đều là `record`. Record là gì, khác class thường chỗ nào?**
<details><summary>Gợi ý</summary>

- Class dữ liệu bất biến: field `private final`, tự sinh constructor, accessor (`ai()` chứ không `getAi()`), `equals`/`hashCode`/`toString`.
- Không kế thừa được class khác (ngầm `extends Record`), nhưng implement interface được, có compact constructor để validate.
- "Bất biến nông": `List<Correction>` bên trong vẫn có thể là list sửa được, trừ khi copy bằng `List.copyOf`.
- Trong project: `@ConfigurationProperties` bind thẳng vào record qua constructor (Spring Boot 3 không cần `@ConstructorBinding`).
</details>

**A2. `<T> T generate(AiRequest<T> request)` trong `AiProvider`. Vì sao phải truyền `Class<T> responseType`, không dùng `T` là đủ?**
<details><summary>Gợi ý</summary>

- **Type erasure**: lúc chạy `T` bị xoá thành `Object`, Jackson/SDK không biết parse JSON thành gì.
- `Class<T>` là "type token": `mapper.readValue(content, request.responseType())` trong `OllamaProvider`.
- Với generic lồng (`List<Correction>`) thì `Class` không đủ, cần `TypeReference` của Jackson.
</details>

**A3. `AiConfig` dùng `switch` có `->` và `yield`. Khác `switch` cổ điển thế nào?**
<details><summary>Gợi ý</summary>

- Switch expression (Java 14+): trả về giá trị, không fall-through, không cần `break`.
- `yield` dùng khi một nhánh có nhiều lệnh (nhánh `default` log warn rồi `yield new NoAiProvider()`).
- Với sealed interface + pattern matching (Java 21), compiler bắt buộc phủ hết case nên không cần `default`.
</details>

**A4. `ApiExceptionHandler` viết `e instanceof MethodArgumentNotValidException m ? ...`. Đây là gì?**
<details><summary>Gợi ý</summary>

Pattern matching cho `instanceof` (Java 16): kiểm tra kiểu và gán biến `m` trong một bước, khỏi ép kiểu thủ công. Java 21 có thêm pattern trong `switch` và record pattern (`case Correction(var o, var c, ...)`).
</details>

**A5. `AiDisabledException` và `AiProviderException` kế thừa `RuntimeException`. Checked với unchecked exception khác nhau thế nào, sao chọn unchecked?**
<details><summary>Gợi ý</summary>

- Checked (`IOException`) bắt buộc `throws`/`catch`; unchecked thì không.
- Chọn unchecked vì service/controller không tự xử lý được; để nó bay lên `@RestControllerAdvice`, map thành 503/502 ở **một chỗ**.
- `ProgressStore` bọc `IOException` thành `UncheckedIOException` cũng vì lý do đó. Luôn giữ `cause` (`new AiProviderException(msg, e)`) để không mất stack trace.
</details>

**A6. `TutorService.cleanCorrections` dùng Stream: `filter → filter → map → toList()`. `toList()` khác `collect(Collectors.toList())` chỗ nào? `List.of` thì sao?**
<details><summary>Gợi ý</summary>

- `Stream.toList()` (Java 16) trả list **không sửa được**; `Collectors.toList()` hiện trả `ArrayList` sửa được (không cam kết).
- `List.of(...)` cũng bất biến và **không nhận `null`**.
- Stream lazy: không có terminal operation (`toList`, `findFirst`) thì không chạy gì.
- `ClaudeProvider` dùng `flatMap(block -> block.text().stream())`: biến `Optional` thành stream 0 hoặc 1 phần tử.
</details>

**A7. `ProgressStore.load()` và `save()` đều `synchronized`. Bảo vệ được gì, không bảo vệ được gì?**
<details><summary>Gợi ý</summary>

- Bean là singleton nên hai request đồng thời không ghi chéo nhau **trong cùng một JVM**.
- Chạy 2 instance (scale ngang) thì vô dụng: cần lock ngoài (DB, file lock, advisory lock như crypto-signal).
- Ghi file tạm rồi `Files.move(..., ATOMIC_MOVE)`: crash giữa chừng không để lại file ghi dở. Đây là chuyện **atomic**, khác chuyện **đồng thời**.
- Câu hỏi xoáy: dùng `ReentrantLock` hay `ReadWriteLock` thì hơn gì? (đọc song song được; có `tryLock` timeout).
</details>

**A8. Tomcat cấu hình `threads.max: 20`, lời gọi AI có timeout 90 giây (`AI_TIMEOUT`) và `maxRetries(2)`. 25 người gọi `/api/roleplay` cùng lúc thì sao?**
<details><summary>Gợi ý</summary>

- 20 thread bị chặn chờ Claude, 5 request còn lại xếp hàng (accept queue), kể cả `/api/health` cũng phải chờ.
- Trường hợp xấu một request giữ thread rất lâu: tối đa 3 lần thử × timeout.
- Hướng sửa: virtual threads (`spring.threads.virtual.enabled=true`), giới hạn số request AI đồng thời (`Semaphore`), rate limit theo access key (bài tuần 2), giảm retry.
</details>

**A9. Virtual thread là gì? Vì sao hợp với app này? Có bẫy gì?**
<details><summary>Gợi ý</summary>

- Thread nhẹ do JVM quản lý, gắn tạm lên vài "carrier" platform thread. Khi bị chặn I/O thì nhả carrier, nên có thể có hàng nghìn request chờ AI mà không tốn hàng nghìn thread OS.
- Hợp với việc "chủ yếu là chờ" (I/O-bound); không làm CPU-bound chạy nhanh hơn.
- Bẫy: trước Java 24, chặn bên trong `synchronized` sẽ **ghim** (pin) carrier (`ProgressStore` là ví dụ, nhưng nó chỉ ghi file nhỏ nên ít ảnh hưởng). `ThreadLocal` nặng nhân lên theo số thread. Không pool virtual thread.
</details>

**A10. `AccessKeyFilter` so key bằng `MessageDigest.isEqual`, không dùng `String.equals`. Vì sao?**
<details><summary>Gợi ý</summary>

So sánh thời gian hằng (constant-time): `equals` dừng ở ký tự sai đầu tiên, kẻ tấn công đo thời gian phản hồi để đoán dần từng ký tự (timing attack). `isEqual` vẫn lộ **độ dài** nếu hai mảng khác độ dài; chặt hơn thì băm cả hai bằng SHA-256 rồi mới so.
</details>

**A11. JVM: heap, stack, metaspace là gì? App chạy free tier Render ít RAM thì chỉnh gì?**
<details><summary>Gợi ý</summary>

- Heap: object (GC dọn). Stack: mỗi thread một stack, chứa frame và biến cục bộ. Metaspace: metadata của class (Spring nạp nhiều class).
- Trong container: `-XX:MaxRAMPercentage=75` thay vì `-Xmx` cứng; ít thread (đã để `max: 20`); Serial GC hợp RAM nhỏ, G1 là mặc định cho máy lớn hơn.
- Biết kể `OutOfMemoryError: Java heap space` khác `Metaspace` khác `unable to create native thread`.
</details>

**Câu nền hay bị hỏi (không gắn project, ôn tuần 5):**
`HashMap` hoạt động thế nào (bucket, hash, treeify khi va chạm nhiều) · `equals`/`hashCode` contract · `ArrayList` vs `LinkedList` · `ConcurrentHashMap` vs `Collections.synchronizedMap` · `volatile` vs `synchronized` vs `AtomicInteger` · `CompletableFuture` · `String` immutable và String pool · `final`/`finally`/`finalize` · interface `default` method vs abstract class.

---

## B. Spring Boot (qua korean-practice)

**B1. Vẽ đường đi request `POST /api/email/check` từ browser tới Claude và ngược lại.**
<details><summary>Gợi ý</summary>

Browser (preflight `OPTIONS` trước) → Tomcat → `CorsFilter` (order `HIGHEST_PRECEDENCE`) → `AccessKeyFilter` (`+10`) → `RequestSizeLimit` (`@Component`, order mặc định) → `DispatcherServlet` → `ApiController.checkEmail` (`@Valid` kiểm `EmailCheckRequest`) → `TutorService.checkEmail` (dựng prompt) → `AiProvider` (bean được chọn ở `AiConfig`) → `ClaudeProvider` (structured output theo `EmailCheckResponse.class`) → `cleanCorrections` + kẹp `score` 0–100 → Jackson → response. Lỗi đi qua `ApiExceptionHandler`.
</details>

**B2. Vì sao `CorsFilter` phải đứng trước `AccessKeyFilter`? Đảo thứ tự thì người dùng thấy gì?**
<details><summary>Gợi ý</summary>

Response 401 từ `AccessKeyFilter` sẽ không có header `Access-Control-Allow-Origin`, browser chặn không cho JS đọc response: frontend thấy "CORS error / network error" chứ không thấy 401, nên không biết là sai key. Preflight `OPTIONS` không mang header custom, nên `AccessKeyFilter.shouldNotFilter` cho `OPTIONS` đi qua. Test `missingKeyIs401WithCorsHeaders` khoá đúng điều này.
</details>

**B3. Filter khác `HandlerInterceptor` khác AOP thế nào? Vì sao access key làm bằng filter?**
<details><summary>Gợi ý</summary>

- Filter: tầng Servlet, trước `DispatcherServlet`, thấy mọi request (kể cả static, 404).
- Interceptor: tầng Spring MVC, biết handler (controller method) nào sẽ chạy.
- AOP: quanh method của bean bất kỳ.
- Access key và CORS là việc chặn cửa cho cả `/api/*`, không cần biết controller, nên filter hợp. Rate limit theo endpoint thì interceptor cũng được.
</details>

**B4. `RequestSizeLimit` chặn theo `getContentLengthLong()`. Có lách được không?**
<details><summary>Gợi ý</summary>

Có: request gửi `Transfer-Encoding: chunked` không có `Content-Length`, hàm trả `-1`, nên lọt qua. Cách sửa: bọc input stream đếm byte và cắt khi vượt ngưỡng, hoặc giới hạn ở tầng proxy/Tomcat. Câu này hay vì cho thấy bro **đọc code để tìm điểm yếu**, không chỉ kể nó làm gì.
</details>

**B5. `@ConfigurationProperties` khác `@Value` thế nào? Biến môi trường `AI_TIMEOUT=PT90S` biến thành `Duration` ra sao?**
<details><summary>Gợi ý</summary>

- `@ConfigurationProperties`: gom cả nhóm config vào một object có kiểu, validate được, IDE gợi ý; `@Value` lấy lẻ từng giá trị.
- `application.yml` có `${AI_TIMEOUT:PT90S}` (env var, có mặc định); Spring converter đổi chuỗi ISO-8601 (hoặc `90s`) sang `Duration`.
- Secret không nằm trong file: `ANTHROPIC_API_KEY` để SDK tự đọc từ env.
</details>

**B6. `AiConfig` chọn provider bằng `switch` trong một `@Bean`. Còn cách nào khác? Vì sao thiếu API key lại trả `NoAiProvider` thay vì cho app chết?**
<details><summary>Gợi ý</summary>

- Cách khác: `@ConditionalOnProperty(name = "app.ai.provider", havingValue = "claude")` trên từng bean, hoặc `@Profile`.
- Fail soft: thiếu key thì phần sync progress vẫn chạy, `/api/health` báo `aiEnabled=false`, AI endpoint trả 503 `ai_disabled`. Đánh đổi: lỗi cấu hình không lộ ra lúc deploy, nên có log warn.
- Đây là **Strategy pattern** + **Null Object pattern** (`NoAiProvider`).
</details>

**B7. Vì sao `AiDisabledException` → 503 còn `AiProviderException` → 502? Frontend dùng gì để phân biệt lỗi?**
<details><summary>Gợi ý</summary>

503 Service Unavailable: server mình chưa bật tính năng. 502 Bad Gateway: server mình ổn nhưng **upstream** (Claude/Ollama) lỗi hoặc trả rác. Body chung dạng `{"error": code, "message": text}`; frontend switch trên `error`, không trên chữ của `message`.
</details>

**B8. `@Valid` trên `EmailCheckRequest` kiểm những gì? Thiếu `@Valid` ở `List<ChatTurn> history` thì sao?**
<details><summary>Gợi ý</summary>

`@NotBlank @Size(max = 4000)` cho `text`, `@Pattern(regexp = "hasipsio|haeyo")` cho `politeness` (null vẫn hợp lệ vì `@Pattern` bỏ qua null). `@Valid` trên `history` mới làm Bean Validation đi vào **từng** `ChatTurn`; thiếu thì `role` sai hay `text` 10.000 ký tự vẫn lọt. Lỗi → `MethodArgumentNotValidException` → 400 `bad_request`.
</details>

**B9. Test: `ApiIntegrationTest` dùng `@SpringBootTest` + MockMvc, còn `ClaudeProviderTest` dựng `FakeHttpServer`. Vì sao không mock `AnthropicClient` bằng Mockito?**
<details><summary>Gợi ý</summary>

- Server HTTP giả kiểm cả những thứ mock bỏ qua: header thật (`anthropic-beta`), body JSON thật SDK gửi đi, cách SDK parse response. Mock interface chỉ kiểm "mình gọi hàm nào".
- `@SpringBootTest` dựng cả context: filter, CORS, validation, exception handler chạy thật. Nhanh hơn thì `@WebMvcTest` (chỉ tầng web).
- `@DynamicPropertySource` để set property cho từng test.
</details>

**B10. Claude trả `stop_reason = max_tokens` hoặc `refusal`. Code xử lý thế nào, vì sao không trả luôn phần text có được?**
<details><summary>Gợi ý</summary>

Cả hai ném `AiProviderException` → 502. Với structured output, text bị cắt là JSON dở, parse sẽ lỗi hoặc ra object thiếu field; trả nó cho frontend là để lỗi lộ ra ở chỗ khó debug hơn. Sau đó `TutorService` vẫn kiểm `reply`/`corrected` rỗng thì cũng báo lỗi: **không tin output của AI**, luôn validate lại.
</details>

**B11. Timeout và retry: SDK đặt `timeout(90s)` và `maxRetries(2)`. Retry kiểu gì là an toàn?**
<details><summary>Gợi ý</summary>

Chỉ retry lỗi tạm (429, 5xx, timeout kết nối), có backoff + jitter; không retry 400. Lời gọi AI ở đây không đổi trạng thái gì nên retry an toàn (idempotent). Với lời gọi có side effect (trừ tiền, gửi mail) cần idempotency key. Đắt: mỗi retry tốn token, cộng dồn thời gian chờ của người dùng.
</details>

**B12. Muốn thêm DB cho korean-practice (lưu lịch sử roleplay, nhiều người dùng) thì thiết kế thế nào?**
<details><summary>Gợi ý</summary>

Postgres + Flyway migration; bảng `users`, `roleplay_sessions`, `roleplay_turns (session_id FK, role, text, created_at)`, index `(session_id, created_at)`; `corrections` để JSONB hoặc bảng riêng nếu cần thống kê theo `type`. Thay `ProgressStore` file bằng bảng `progress (user_id PK, data JSONB, updated_at)` với upsert. Bỏ `synchronized` vì DB lo đồng thời (optimistic lock bằng `updated_at`/`@Version`).
</details>

---

## S. SQL / PostgreSQL (qua crypto-signal)

**S1. `market_candles` có `id BIGSERIAL PRIMARY KEY` **và** `UNIQUE (symbol, market, timeframe, open_time)`. Sao cần cả hai?**
<details><summary>Gợi ý</summary>

Surrogate key (`id`) để tham chiếu gọn; natural key (bộ 4 cột) là **luật nghiệp vụ**: một nến chỉ có một dòng. Unique constraint là thứ làm `ON CONFLICT` hoạt động, nên ingest chạy lại bao nhiêu lần cũng không nhân đôi dữ liệu. Thiếu nó thì dedupe phải làm ở code và sẽ có lúc quên.
</details>

**S2. Index `(symbol, market, timeframe, open_time DESC)`. Query nào dùng được, query nào không?**
<details><summary>Gợi ý</summary>

Quy tắc **leftmost prefix**: dùng tốt cho `WHERE symbol = ? AND market = ? AND timeframe = ? ORDER BY open_time DESC LIMIT n` (lấy n nến mới nhất, không cần sort). Chỉ lọc `WHERE timeframe = '1h'` thì không tận dụng được. Thứ tự cột: cột lọc bằng `=` trước, cột range/sort sau cùng. Lưu ý: unique constraint cũng đã tạo một index trên đúng 4 cột này; hỏi xoáy được là index thứ hai có thừa không (khác ở chiều `DESC`, nhưng Postgres quét ngược B-tree được).
</details>

**S3. `CHECK (market IN ('spot','futures'))`, `CHECK (severity IN (...))`. Vì sao không để code kiểm?**
<details><summary>Gợi ý</summary>

DB là lớp phòng thủ cuối: mọi đường ghi (worker, script, migration, psql tay) đều bị chặn. Thay thế: kiểu `ENUM` của Postgres (khó thêm/xoá giá trị hơn) hoặc bảng tra cứu + FK.
</details>

**S4. Giá lưu `DOUBLE PRECISION`. Ngân hàng thì có làm vậy không?**
<details><summary>Gợi ý</summary>

Không. Số thực nhị phân có sai số (`0.1 + 0.2 ≠ 0.3`); tiền thì dùng `NUMERIC(p, s)` (Java: `BigDecimal`, không `double`). Ở đây chấp nhận được vì là chỉ báo phân tích (CVD, % thay đổi), không phải sổ cái; `NUMERIC` chậm hơn và tốn chỗ hơn. Nói được đánh đổi này là điểm cộng.
</details>

**S5. Mọi thời gian là `TIMESTAMPTZ`. Khác `TIMESTAMP` thế nào?**
<details><summary>Gợi ý</summary>

`TIMESTAMPTZ` lưu một thời điểm tuyệt đối (nội bộ là UTC), hiển thị theo timezone của session; `TIMESTAMP` chỉ là "giờ trên đồng hồ" không có múi giờ, dễ lệch khi server và người dùng khác múi. Project quy ước UTC (helper trong `shared`). Java: map sang `Instant`/`OffsetDateTime`, không phải `LocalDateTime`.
</details>

**S6. `upsertCandle`: `INSERT ... ON CONFLICT (...) DO UPDATE SET close = EXCLUDED.close ...`. `EXCLUDED` là gì? Khác "SELECT rồi INSERT hoặc UPDATE" ở code chỗ nào?**
<details><summary>Gợi ý</summary>

`EXCLUDED` là dòng định chèn nhưng bị trùng. Upsert là **một câu, nguyên tử**: không có khe giữa SELECT và INSERT để request khác chen vào (race condition → lỗi duplicate key hoặc mất cập nhật). Còn có thêm `WHERE` sau `DO UPDATE` (`keepLiveOverBackfill`): dữ liệu live không bao giờ bị dữ liệu backfill ghi đè.
</details>

**S7. `ON CONFLICT DO NOTHING` (ví dụ `bot_settings`, `coin_watch_intraday_sent`) dùng khi nào?**
<details><summary>Gợi ý</summary>

Khi dòng đã có là đúng và không cần sửa: tạo setting mặc định lần đầu, đánh dấu "đã gửi alert hôm nay" để không gửi trùng (bảng chính là khoá idempotency). Lưu ý: `DO NOTHING` + `RETURNING` không trả dòng đã tồn tại, nên `startFarm` phải `SELECT` lại sau khi insert.
</details>

**S8. Partial unique index: `CREATE UNIQUE INDEX ... ON gem_watches (chat_id, chain_id, token_address) WHERE status = 'active'`. Giải bài toán gì?**
<details><summary>Gợi ý</summary>

"Mỗi người chỉ có **một** watch đang active cho một token, nhưng được giữ nhiều watch đã đóng trong lịch sử." `UNIQUE` thường không làm được vì các dòng `closed` sẽ trùng nhau. Muốn dùng nó với upsert phải ghi đúng điều kiện: `ON CONFLICT (chat_id, project_key) WHERE status = 'active'` (`airdrop.ts`). Partial index thường (`WHERE status = 'open'`) còn nhỏ hơn hẳn index toàn bảng vì chỉ chứa vài dòng đang mở.
</details>

**S9. Lấy dòng mới nhất cho mỗi nhóm: `DISTINCT ON` vs `ROW_NUMBER()` vs `GROUP BY` + `MAX`.**
<details><summary>Gợi ý</summary>

- `SELECT DISTINCT ON (symbol) ... ORDER BY symbol, timestamp DESC`: riêng Postgres, ngắn, nhanh khi có index phù hợp (`markPrice.ts`).
- `ROW_NUMBER() OVER (PARTITION BY symbol ORDER BY timestamp DESC) = 1` trong subquery/CTE: chuẩn SQL, lấy được top-N.
- `GROUP BY` + `MAX(timestamp)` rồi join lại: chạy mọi DB nhưng dài và có thể ra nhiều dòng khi trùng timestamp.
</details>

**S10. Bug trong `gems.ts`: `DISTINCT ON` + `LIMIT` trả sai top gem khi có 2 chain. Vì sao, sửa thế nào?**
<details><summary>Gợi ý</summary>

`DISTINCT ON` bắt `ORDER BY` bắt đầu bằng khoá dedupe `(chain_id, token_address)`, nên `LIMIT` cắt theo thứ tự **địa chỉ**, không theo **điểm**: chain đứng sau bị mất top pick mà danh sách vẫn trông như "xếp theo điểm". Sửa: dedupe ở subquery, sắp `gem_score DESC` và `LIMIT` ở query ngoài. Đây là **một câu chuyện bug** rất tốt cho phỏng vấn: lỗi im lặng, chỉ lộ khi có dữ liệu thứ hai.
</details>

**S11. `saveCoinScan` mở transaction bằng `pool.connect()` → `BEGIN` → `DELETE` → nhiều `INSERT` → `COMMIT`. Vì sao không viết `pool.query('BEGIN')` cho gọn?**
<details><summary>Gợi ý</summary>

Mỗi `pool.query` có thể lấy một connection **khác**; transaction gắn với connection, nên `BEGIN` và `INSERT` có thể nằm ở hai connection khác nhau (transaction vô nghĩa, hoặc kẹt connection đang mở transaction). Phải giữ một `client`, `ROLLBACK` trong `catch`, `release()` trong `finally`. Mục đích: một ngày scan là **một** lần chạy, không bao giờ là nửa cũ nửa mới. Trong Spring, `@Transactional` lo việc giữ connection này cho bro (C3).
</details>

**S12. Isolation level mặc định của Postgres là gì? Lost update xảy ra thế nào và chặn ra sao?**
<details><summary>Gợi ý</summary>

`READ COMMITTED`: mỗi câu thấy dữ liệu đã commit tại lúc câu đó bắt đầu. Lost update: hai transaction cùng đọc `x = 10`, cùng ghi `x = 11`. Chặn bằng: update nguyên tử (`SET x = x + 1`), `SELECT ... FOR UPDATE` (pessimistic), cột version (optimistic), hoặc `SERIALIZABLE` + retry. Postgres dùng MVCC: đọc không chặn ghi, ghi không chặn đọc.
</details>

**S13. `migrate.mjs` dùng `pg_advisory_lock`. Bài toán gì?**
<details><summary>Gợi ý</summary>

Cả `worker` và `api` chạy migration lúc khởi động (để api không bao giờ lên trước schema). Hai runner chạy song song có thể cùng áp một file. Advisory lock là lock theo **số do app tự đặt**, không gắn với bảng nào: runner thứ hai chờ, rồi thấy mọi migration đã áp nên không làm gì. Flyway/Liquibase cũng có cơ chế khoá tương tự.
</details>

**S14. Migration là forward-only, "never edit an applied migration" (comment trong `001_init.sql`). Vì sao? Migration 008 xoá kết quả `signal_outcomes` để tính lại: có nguy hiểm không?**
<details><summary>Gợi ý</summary>

- Sửa file đã chạy thì môi trường đã áp (prod) và môi trường mới (local) ra hai schema khác nhau mà không ai biết. Muốn đổi thì viết migration mới (003 đổi nghĩa `bot_settings.symbols` rỗng thành "mọi coin", 001 giữ nguyên).
- 008: tracker cũ lấy giá của nến **mới nhất lúc job chạy**, không phải nến tại `thời điểm signal + horizon`, nên khi worker chậm thì số đo sai cửa sổ và không phân biệt được dòng nào sai. Set về `NULL` là an toàn vì `market_candles` không bị xoá nên tính lại được; dòng thiếu nến thì để `NULL`: "chưa có kết quả" trung thực hơn "có số nhưng đo sai".
</details>

**S15. Retention: `DELETE FROM liquidations WHERE order_trade_time < now() - ($1 || ' days')::interval`. Lên vài trăm triệu dòng thì còn ổn không?**
<details><summary>Gợi ý</summary>

`DELETE` lớn sinh dead tuple (MVCC), làm bảng phình (bloat) cho tới khi `VACUUM` dọn, giữ lock và WAL lớn. Cách tốt hơn ở quy mô lớn: **partition theo thời gian** (`PARTITION BY RANGE (order_trade_time)`) rồi `DROP` partition cũ (gần như tức thì), hoặc xoá theo lô nhỏ. `($1 || ' days')::interval` vẫn tham số hoá an toàn; viết rõ hơn là `make_interval(days => $1)`.
</details>

**S16. Truy vấn có `$1, $2`. Chống SQL injection thế nào? Khi nào tham số hoá không đủ?**
<details><summary>Gợi ý</summary>

Prepared statement gửi câu SQL và giá trị riêng nhau, giá trị không bao giờ được hiểu là SQL. Không tham số hoá được **tên cột/bảng** và `ORDER BY` động: phải dùng whitelist (`outcomes.ts` map horizon → tên cột cố định bằng `Record<OutcomeHorizon, string>`). Java: `PreparedStatement`, `JdbcTemplate` với `?`, JPA `:param`.
</details>

**S17. `WHERE farm_id = ANY($1::bigint[])` trong `airdrop.ts`. Vì sao không query từng farm?**
<details><summary>Gợi ý</summary>

Đó là cách tránh **N+1**: 1 query lấy danh sách farm + 1 query lấy mọi action của tất cả farm, gom ở code; thay vì 1 + N query. Trong JPA: `JOIN FETCH`, `@EntityGraph`, `@BatchSize`, hoặc `WHERE id IN (:ids)`.
</details>

**S18. `reasons JSONB`, `metrics JSONB`, `health_components JSONB`. Khi nào JSONB, khi nào tách cột?**
<details><summary>Gợi ý</summary>

JSONB khi dữ liệu chủ yếu để hiển thị, cấu trúc hay đổi theo loại signal, ít khi lọc/aggregate. Tách cột khi cần lọc, sort, constraint, FK, hay thống kê (vì vậy `health_score`, `risk_score` là cột riêng dù đã có trong components). Lọc JSONB thì cần GIN index (`USING GIN (metrics jsonb_path_ops)`).
</details>

**S19. `PG_POOL_MAX` mặc định 10, chạy trên free tier Supabase/Neon. Pool size chọn thế nào?**
<details><summary>Gợi ý</summary>

DB free tier giới hạn số connection; image all-in-one chạy hai pool tới cùng DB, nên tổng `max` của mọi process phải nhỏ hơn giới hạn đó. Connection Postgres đắt (mỗi cái một process), to hơn chưa chắc nhanh hơn. Java: HikariCP `maximum-pool-size`; nhiều instance thì đặt PgBouncer phía trước.
</details>

**S20. Query chậm, bro làm gì đầu tiên?**
<details><summary>Gợi ý</summary>

`EXPLAIN (ANALYZE, BUFFERS)`: xem `Seq Scan` hay `Index Scan`, số dòng ước lượng so với thực tế (lệch nhiều thì `ANALYZE` bảng), sort có tràn ra đĩa không. Rồi mới thêm index đúng thứ tự cột, bật `pg_stat_statements` để tìm query tốn nhất tổng thể. Index không miễn phí: làm chậm ghi, tốn chỗ.
</details>

**Câu nền hay bị hỏi:** `INNER`/`LEFT`/`FULL` JOIN · `WHERE` vs `HAVING` · `UNION` vs `UNION ALL` · `NULL` trong `NOT IN` (bẫy: có một `NULL` là ra rỗng, dùng `NOT EXISTS`) · `COUNT(*)` vs `COUNT(col)` · chuẩn hoá 1NF/2NF/3NF và khi nào phi chuẩn hoá · ACID · clustered vs non-clustered index (MySQL InnoDB vs Postgres heap) · view vs materialized view.

---

## Bài tập SQL tự viết trên schema thật

Chạy Postgres local trong repo crypto-signal: `docker compose up postgres migrate`, rồi `psql postgres://crypto:crypto@localhost:5432/crypto_market_health`. Viết tay trước, mở đáp án sau.

**1. Snapshot mới nhất khung `1h` của mỗi symbol** (`market_health_snapshots`).
<details><summary>Đáp án</summary>

```sql
SELECT DISTINCT ON (symbol) symbol, timestamp, health_score, risk_score
FROM market_health_snapshots
WHERE timeframe = '1h'
ORDER BY symbol, timestamp DESC;
```
</details>

**2. Mỗi `signal_type`: số signal, tỉ lệ giá tăng sau 4h, chỉ lấy loại có ≥ 30 mẫu đã có kết quả.**
<details><summary>Đáp án</summary>

```sql
SELECT s.signal_type,
       COUNT(*)                                             AS n,
       AVG((o.move_after_4h_pct > 0)::int)                  AS up_rate,
       percentile_cont(0.5) WITHIN GROUP (ORDER BY o.move_after_4h_pct) AS median_move
FROM market_signals s
JOIN signal_outcomes o ON o.signal_id = s.signal_id
WHERE o.move_after_4h_pct IS NOT NULL
GROUP BY s.signal_type
HAVING COUNT(*) >= 30
ORDER BY up_rate DESC;
```
Hỏi xoáy: vì sao phải loại `NULL` trước khi tính (migration 008)? Vì sao 30 mẫu vẫn có thể là may mắn (xem `edge.ts`: khoảng tin cậy)?
</details>

**3. Tìm lỗ hổng dữ liệu: nến `5m` futures BTCUSDT bị thiếu (hai nến liên tiếp cách nhau > 5 phút).**
<details><summary>Đáp án</summary>

```sql
SELECT open_time, prev_open, open_time - prev_open AS gap
FROM (
  SELECT open_time, LAG(open_time) OVER (ORDER BY open_time) AS prev_open
  FROM market_candles
  WHERE symbol = 'BTCUSDT' AND market = 'futures' AND timeframe = '5m'
) t
WHERE open_time - prev_open > interval '5 minutes'
ORDER BY open_time;
```
</details>

**4. Top 3 signal có `confidence` cao nhất của mỗi symbol trong 7 ngày qua.**
<details><summary>Đáp án</summary>

```sql
WITH ranked AS (
  SELECT symbol, signal_type, confidence, timestamp,
         ROW_NUMBER() OVER (PARTITION BY symbol ORDER BY confidence DESC, timestamp DESC) AS rn
  FROM market_signals
  WHERE timestamp >= now() - interval '7 days'
)
SELECT * FROM ranked WHERE rn <= 3 ORDER BY symbol, rn;
```
Hỏi xoáy: `RANK()` và `DENSE_RANK()` khác `ROW_NUMBER()` thế nào khi trùng `confidence`?
</details>

**5. Kiểm cooldown: đã gửi alert cùng `(symbol, timeframe, signal_type)` trong 30 phút gần nhất chưa?** Index nào được dùng?
<details><summary>Đáp án</summary>

```sql
SELECT EXISTS (
  SELECT 1 FROM alert_events
  WHERE symbol = $1 AND timeframe = $2 AND signal_type = $3
    AND sent_at >= now() - interval '30 minutes'
);
```
Dùng `idx_alert_events_cooldown (symbol, timeframe, signal_type, sent_at DESC)`: 3 cột `=` rồi tới range. Chạy `EXPLAIN` để thấy `Index Only Scan`/`Index Scan`.
</details>

**6. Người dùng bật alert nhưng chưa từng nhận alert nào.**
<details><summary>Đáp án</summary>

```sql
SELECT u.chat_id, u.username
FROM bot_users u
JOIN bot_settings bs ON bs.chat_id = u.chat_id AND bs.alerts_enabled
WHERE NOT EXISTS (SELECT 1 FROM alert_events a WHERE a.chat_id = u.chat_id);
```
Hỏi xoáy: viết bằng `LEFT JOIN ... WHERE a.id IS NULL` được không? Vì sao `NOT IN (SELECT chat_id FROM alert_events)` có thể trả rỗng (`chat_id` cho phép `NULL`)?
</details>

**7. Số alert mỗi ngày (theo giờ Việt Nam) trong 14 ngày qua, ngày không có alert vẫn hiện số 0.**
<details><summary>Đáp án</summary>

```sql
SELECT d::date AS day, COUNT(a.id) AS alerts
FROM generate_series(
       date_trunc('day', now() AT TIME ZONE 'Asia/Ho_Chi_Minh') - interval '13 days',
       date_trunc('day', now() AT TIME ZONE 'Asia/Ho_Chi_Minh'),
       interval '1 day') AS d
LEFT JOIN alert_events a
  ON date_trunc('day', a.sent_at AT TIME ZONE 'Asia/Ho_Chi_Minh') = d
GROUP BY d
ORDER BY d;
```
Điểm chính: `COUNT(a.id)` chứ không `COUNT(*)` (không thì ngày trống đếm ra 1); đổi múi giờ trước khi cắt ngày.
</details>

**8. Viết migration thêm cột `muted` (mặc định `false`) cho `market_signals` khi bảng đã lớn, không khoá bảng lâu.**
<details><summary>Đáp án</summary>

```sql
ALTER TABLE market_signals ADD COLUMN muted BOOLEAN NOT NULL DEFAULT false;
CREATE INDEX CONCURRENTLY idx_market_signals_muted ON market_signals (symbol, timestamp DESC) WHERE muted;
```
So với migration thật `009_data_provenance.sql` (thêm cột `source` cho nhiều bảng). Từ Postgres 11, `ADD COLUMN ... DEFAULT` hằng số không ghi lại cả bảng. `CREATE INDEX CONCURRENTLY` không chặn ghi nhưng **không chạy được trong transaction**. Mà `db/migrate.mjs` bọc mỗi file trong `BEGIN`/`COMMIT`, nên đưa câu này vào repo sẽ lỗi ngay: phải thêm cách cho một file chạy ngoài transaction (Flyway có `executeInTransaction=false`).
</details>

---

## C. Cầu nối: nếu viết lại phần DB bằng Spring

**C1. Port `upsertCandle` sang Java: JPA `save()`, `JdbcTemplate`, hay jOOQ?**
<details><summary>Gợi ý</summary>

`save()` với entity đã có id sẽ `SELECT` rồi `UPDATE` (merge): hai câu, không nguyên tử, và JPA không có `ON CONFLICT`. Ingest số lượng lớn hợp `JdbcTemplate.batchUpdate` với câu `INSERT ... ON CONFLICT` viết tay (hoặc `@Query(nativeQuery = true)` + `@Modifying`). Đọc/ghi CRUD bình thường thì JPA ổn. Câu trả lời tốt: **dùng cả hai, mỗi cái đúng chỗ**.
</details>

**C2. `@Transactional` hoạt động thế nào? Vì sao gọi một method `@Transactional` từ method khác **cùng class** thì không có transaction?**
<details><summary>Gợi ý</summary>

Spring bọc bean bằng **proxy**; transaction mở khi gọi đi qua proxy. Gọi `this.method()` bên trong class là gọi thẳng, bỏ qua proxy (self-invocation). Cũng không chạy với method `private`. Mặc định chỉ rollback với `RuntimeException`/`Error`, checked exception thì commit (trừ khi `rollbackFor`).
</details>

**C3. `saveCoinScan` (DELETE ngày + INSERT nhiều dòng) viết bằng Spring thế nào?**
<details><summary>Gợi ý</summary>

Một method `@Transactional` trong service: `repo.deleteByDay(day)` + `jdbc.batchUpdate(...)`. Spring giữ một connection suốt method (đúng việc `pool.connect()` làm thủ công trong TS), lỗi thì rollback tự động.
</details>

**C4. Propagation `REQUIRED` vs `REQUIRES_NEW`. Ví dụ dùng `REQUIRES_NEW` trong crypto-signal?**
<details><summary>Gợi ý</summary>

`REQUIRED` (mặc định): nhập vào transaction đang có. `REQUIRES_NEW`: tạm dừng cái đang có, mở cái mới, commit độc lập. Ví dụ: ghi `job_health` (job chạy lỗi) phải được lưu kể cả khi transaction chính của job bị rollback.
</details>

**C5. N+1 trong JPA: entity `Farm` có `@OneToMany List<FarmAction> actions`. List 50 farm thì chạy mấy query? Sửa thế nào?**
<details><summary>Gợi ý</summary>

`LAZY` (mặc định cho `@OneToMany`): 1 query lấy farm + 50 query lấy action khi truy cập `actions`. Sửa: `JOIN FETCH`, `@EntityGraph(attributePaths = "actions")`, `@BatchSize`/`default_batch_fetch_size` (ra `WHERE farm_id IN (...)`, giống hệt `ANY($1)` ở S17). Phát hiện: bật log SQL, đếm query trong test. Đừng đổi sang `EAGER` cho xong.
</details>

**C6. `@Transactional(readOnly = true)` có tác dụng gì thật?**
<details><summary>Gợi ý</summary>

Hibernate bỏ dirty checking/flush (nhẹ hơn), driver có thể đánh dấu connection read-only, và có thể dùng để định tuyến sang replica. Không tự chặn được mọi câu ghi ở mức DB nếu driver không hỗ trợ.
</details>

**C7. Optimistic lock bằng `@Version`: dùng cho `trade_journal` thế nào?**
<details><summary>Gợi ý</summary>

Cột `version` tăng mỗi lần update; `UPDATE ... WHERE id = ? AND version = ?` ảnh hưởng 0 dòng thì JPA ném `OptimisticLockException`. Hợp khi xung đột hiếm (một người sửa journal của mình từ web và Telegram cùng lúc). Xung đột nhiều thì `SELECT ... FOR UPDATE` (`@Lock(PESSIMISTIC_WRITE)`).
</details>

**C8. Test tầng DB: crypto-signal có file `*.db.test.ts` chạy với Postgres thật. Java làm tương đương thế nào?**
<details><summary>Gợi ý</summary>

Testcontainers (`@Testcontainers` + `PostgreSQLContainer`, hoặc `@ServiceConnection` từ Spring Boot 3.1) để chạy đúng Postgres 16, không dùng H2 (khác cú pháp: `ON CONFLICT`, `DISTINCT ON`, JSONB không có). `@DataJpaTest` cho tầng repository, chạy migration Flyway trước test.
</details>

---

## Câu tình huống trộn (dùng cho mock)

1. Request `/api/email/check` lúc nào cũng mất 40 giây. Bro điều tra từ đâu, đo ở những chỗ nào?
2. Dashboard `/performance` của crypto-signal hiện tỉ lệ thắng 72%. Bro tin con số này tới đâu? (live vs backfill, số mẫu, NULL, sai cửa sổ như migration 008)
3. Có 2 instance korean-practice sau load balancer. Cái gì hỏng đầu tiên? (`ProgressStore` file + `synchronized`, đĩa Render tạm thời)
4. Bảng `market_candles` lên 500 triệu dòng. Thay đổi gì? (partition, retention, index thừa, aggregate sẵn)
5. AI trả về một `Correction` có `original` không hề có trong email của người dùng. Bắt ở đâu? (hiện `cleanCorrections` chưa kiểm; có thể thêm `source.contains(original)`, và đưa vào bộ eval tuần 3)
6. Ai đó tìm ra URL backend và spam `/api/roleplay`. Những lớp nào chặn được, lớp nào chưa có? (access key, size limit, chưa có rate limit, chi phí AI)

---

## Bảng ⚠️ (câu trả lời chưa trôi)

| Ngày | Mã câu | Mình đã nói gì | Đúng ra là | Ôn lại ngày |
|---|---|---|---|---|
| | | | | |
