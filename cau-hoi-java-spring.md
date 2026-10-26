# Câu hỏi phỏng vấn Java + Spring

Bộ câu hỏi chung, gắn vào stack đang làm: **Spring Boot 2.x, MyBatis, Oracle, API cho app Android**.
Mã câu: `J` là Java, `SP` là Spring. Câu nào đã có trong file khác thì chỉ ghi mã để khỏi lặp: `A1`, `B3` ở [phong-van-java-sql.md](phong-van-java-sql.md), `Q4` ở [du-an-cong-ty.md](du-an-cong-ty.md).

**Mức độ:** ⭐ gần như chắc chắn bị hỏi · ⭐⭐ hay gặp ở vòng kỹ thuật · ⭐⭐⭐ hỏi khi muốn đào sâu (mid/senior)

Phần Spring (mục 5–13) có thêm ví dụ code, 🧪 thử tay và 📖 link tài liệu chính thức.

**Thứ tự ôn:** hết ⭐ trước (tuần 2), rồi tới ⭐⭐ (tuần 3–5), còn dư thời gian mới làm ⭐⭐⭐. Câu nào có dòng **"Ở dự án"** thì tập trả lời bằng ví dụ ở công ty.

---

## 1. Java: OOP và nền tảng

**J1. ⭐ 4 tính chất của OOP là gì? Cho ví dụ trong code của bro.**
<details><summary>Gợi ý</summary>

Đóng gói (field `private`, chỉ đổi trạng thái qua method), kế thừa, đa hình (một interface, nhiều cách cài đặt), trừu tượng (giấu chi tiết sau interface). Ví dụ có sẵn: `AiProvider` có 3 cài đặt (`ClaudeProvider`, `OllamaProvider`, `NoAiProvider`); mapper interface của MyBatis.
</details>

**J2. ⭐ Overloading khác overriding thế nào?**
<details><summary>Gợi ý</summary>

Overloading: cùng tên, khác tham số, chọn lúc **compile**. Overriding: lớp con định nghĩa lại method của lớp cha, cùng signature, chọn lúc **chạy** theo kiểu thật của object. Overriding không được giảm quyền truy cập, và không được ném thêm checked exception rộng hơn.
</details>

**J3. ⭐ Abstract class khác interface thế nào? Khi nào dùng cái nào?**
<details><summary>Gợi ý</summary>

Interface: một class implement được nhiều interface, từ Java 8 có `default` và `static` method, không có state (chỉ có hằng số). Abstract class: có field, có constructor, nhưng chỉ kế thừa được một lớp. Dùng interface để định nghĩa "làm được gì" (hợp đồng); dùng abstract class khi các lớp con dùng chung code và trạng thái (Template Method).
</details>

**J4. ⭐ `==` khác `equals()`? Vì sao `Integer a = 127, b = 127; a == b` là `true`, còn với 128 lại là `false`?**
<details><summary>Gợi ý</summary>

`==` so địa chỉ (với object), `equals` so nội dung. `Integer` cache sẵn các giá trị từ -128 tới 127, nên 127 là cùng một object, 128 thì không. Bug thật hay gặp: so `Long` id trả về từ mapper bằng `==`. Luôn dùng `equals` hoặc `Objects.equals(a, b)` (an toàn với `null`).
</details>

**J5. ⭐ Hợp đồng `equals`/`hashCode` là gì? Override `equals` mà quên `hashCode` thì sao?**
<details><summary>Gợi ý</summary>

Hai object `equals` nhau thì phải có cùng `hashCode` (chiều ngược lại không bắt buộc). Quên override thì `HashMap`/`HashSet` coi hai object "bằng nhau" là hai key khác nhau: `set.contains(x)` trả `false` dù vừa `add` một object bằng `x`. Record tự sinh cả hai method.
</details>

**J6. ⭐ Vì sao `String` là immutable? `StringBuilder` khác `StringBuffer` chỗ nào?**
<details><summary>Gợi ý</summary>

Bất biến thì dùng chung an toàn được (String pool), an toàn giữa nhiều thread, cache được `hashCode` (key `HashMap` hay là String), và an toàn khi làm tham số (đường dẫn, câu SQL). Cộng chuỗi trong vòng lặp sinh ra nhiều object, nên dùng `StringBuilder`. `StringBuffer` có `synchronized` nên chậm hơn, hiếm khi cần.
</details>

**J7. ⭐ Java truyền tham số theo tham trị hay tham chiếu?**
<details><summary>Gợi ý</summary>

Luôn là **tham trị**. Với object thì giá trị được copy là *tham chiếu*: trong method sửa field của object thì bên ngoài thấy, nhưng gán biến tham số sang object mới thì bên ngoài không thấy.
</details>

**J8. ⭐⭐ `final`, `finally`, `finalize` khác nhau thế nào? `finally` có `return` thì sao?**
<details><summary>Gợi ý</summary>

`final`: biến không gán lại được, method không override được, class không kế thừa được. `finally`: luôn chạy sau `try`/`catch`. `finalize`: cơ chế dọn dẹp cũ, đã deprecated, không dùng. `return` trong `finally` **nuốt mất exception** và ghi đè giá trị return của `try`, nên không bao giờ viết vậy. Đóng tài nguyên thì dùng try-with-resources (`AutoCloseable`).
</details>

**J9. ⭐⭐ `static` dùng khi nào? Vì sao không nên giữ state trong biến `static`?**
<details><summary>Gợi ý</summary>

`static` thuộc về class, không thuộc object: dùng cho hằng số, hàm tiện ích không có state, factory method. Một biến `static` có thể sửa được là state dùng chung giữa mọi thread và mọi request: dễ có race condition, khó test, và dễ rò bộ nhớ (`static Map` cứ lớn dần).
</details>

**J10. ⭐⭐ SOLID là gì? Cho ví dụ chữ D (Dependency Inversion).**
<details><summary>Gợi ý</summary>

S: mỗi class chỉ một lý do để sửa. O: mở để mở rộng, đóng với sửa đổi. L: lớp con thay được cho lớp cha. I: interface nhỏ, đúng việc. D: phụ thuộc vào abstraction chứ không vào cài đặt cụ thể. Ví dụ chữ D: service gọi interface `OrderMapper`, MyBatis tự sinh cài đặt. Ở korean-practice: `TutorService` chỉ biết interface `AiProvider`, không biết là Claude hay Ollama.
</details>

**J11. ⭐⭐ Exception: checked với unchecked, `Error` với `Exception`.** → xem `A5`. Thêm: `Error` (`OutOfMemoryError`, `StackOverflowError`) thì không nên `catch`. Đừng `catch (Exception e) {}` rồi bỏ đó: mất dấu lỗi và mất luôn rollback (`Q4`).

---

## 2. Java: Collections và Stream

**J12. ⭐ `ArrayList` khác `LinkedList` thế nào? Thực tế dùng cái nào?**
<details><summary>Gợi ý</summary>

`ArrayList`: mảng động, lấy theo index O(1), chèn hoặc xoá ở giữa O(n). `LinkedList`: lấy theo index O(n), chèn hoặc xoá ở đầu và cuối O(1), tốn bộ nhớ hơn, cache CPU kém. Gần như luôn dùng `ArrayList`; cần hàng đợi thì dùng `ArrayDeque`.
</details>

**J13. ⭐ `HashMap` hoạt động bên trong thế nào?**
<details><summary>Gợi ý</summary>

Mảng các bucket. `hash(key)` chọn bucket, va chạm thì xếp thành danh sách liên kết; từ Java 8, khi một bucket có từ 8 phần tử trở lên (và mảng có ít nhất 64 bucket) thì chuyển thành cây đỏ đen. Load factor 0.75: vượt ngưỡng thì gấp đôi mảng và chia lại (rehash). Cho phép 1 key `null`. Không an toàn khi nhiều thread cùng ghi. Key nên là immutable: key bị đổi sau khi `put` thì không tìm lại được.
</details>

**J14. ⭐⭐ `HashMap`, `LinkedHashMap`, `TreeMap`, `ConcurrentHashMap`: dùng khi nào?**
<details><summary>Gợi ý</summary>

`LinkedHashMap`: giữ thứ tự chèn (làm LRU cache được với `accessOrder = true` + `removeEldestEntry`). `TreeMap`: sắp theo key, có `floorKey`/`ceilingKey`, O(log n). `ConcurrentHashMap`: nhiều thread đọc ghi; từ Java 8 dùng CAS và khoá theo từng bucket, không khoá cả map; không nhận `null`. `Collections.synchronizedMap` khoá cả map nên chậm hơn.
</details>

**J15. ⭐ Vì sao `for (Item i : list) { if (...) list.remove(i); }` ném `ConcurrentModificationException`?**
<details><summary>Gợi ý</summary>

Iterator là fail-fast: nó thấy list bị đổi cấu trúc ngoài nó. Sửa: `list.removeIf(i -> ...)`, hoặc dùng `Iterator.remove()`, hoặc tạo list mới bằng stream `filter`. Bẫy cùng loại: `Arrays.asList(...)` có kích thước cố định (`add` ném lỗi), `List.of(...)` thì bất biến hoàn toàn.
</details>

**J16. ⭐ `Comparable` khác `Comparator` thế nào?**
<details><summary>Gợi ý</summary>

`Comparable`: thứ tự tự nhiên, nằm trong chính class (`compareTo`). `Comparator`: thứ tự đặt từ bên ngoài, có nhiều cách sắp được: `Comparator.comparing(Order::getOrderDate).reversed().thenComparing(Order::getId)`. Không viết `a - b` để so số (tràn số), dùng `Integer.compare`.
</details>

**J17. ⭐ Stream: `map` khác `flatMap`? Thao tác trung gian (intermediate) khác thao tác kết thúc (terminal)?**
<details><summary>Gợi ý</summary>

`map`: 1 phần tử ra 1 phần tử. `flatMap`: 1 phần tử ra 0..n phần tử rồi trải phẳng (danh sách đơn → mọi món trong các đơn). Intermediate (`filter`, `map`, `sorted`) là lazy; chỉ khi gặp terminal (`collect`, `toList`, `count`, `findFirst`) thì mới chạy. Một stream chỉ dùng được một lần.
</details>

**J18. ⭐⭐ Viết bằng stream: tổng tiền theo từng cửa hàng, từ `List<Order>`.**
<details><summary>Gợi ý</summary>

```java
Map<String, BigDecimal> totalByStore = orders.stream()
    .collect(Collectors.groupingBy(Order::getStoreCode,
             Collectors.reducing(BigDecimal.ZERO, Order::getAmount, BigDecimal::add)));
```
Hỏi xoáy: vì sao `BigDecimal` mà không `double`? Vì sao `new BigDecimal("0.1")` đúng còn `new BigDecimal(0.1)` sai? (Dùng `BigDecimal.valueOf(0.1)` cũng được.) Lưu ý thêm: `BigDecimal.equals` so cả scale (`2.0` khác `2.00`), nên so giá trị bằng `compareTo`.
</details>

**J19. ⭐⭐ `Optional` dùng đúng và dùng sai thế nào?**
<details><summary>Gợi ý</summary>

Đúng: làm kiểu trả về khi có thể không có kết quả (`findById`), rồi `map`/`orElse`/`orElseThrow`. Sai: làm field, làm tham số method, gọi `get()` mà không kiểm tra, dùng `Optional` bọc collection (trả list rỗng là đủ). `orElse(x)` luôn tính `x`, còn `orElseGet(() -> x)` chỉ tính khi cần.
</details>

**J20. ⭐⭐⭐ Parallel stream có nên dùng trong web app không?**
<details><summary>Gợi ý</summary>

Thường là không: nó chạy trên `ForkJoinPool.commonPool()` dùng chung cho cả JVM, việc chậm (gọi DB, gọi API) chặn luôn chỗ khác; với dữ liệu nhỏ thì chi phí chia việc còn lớn hơn lợi. Chỉ hợp với việc thuần CPU, dữ liệu lớn, không có side effect.
</details>

---

## 3. Java: Concurrency

**J21. ⭐ Tạo thread bằng cách nào? Vì sao nên dùng thread pool?**
<details><summary>Gợi ý</summary>

`new Thread(runnable)`, `ExecutorService`, `CompletableFuture`, và trong Spring thì `@Async`. Pool để dùng lại thread (tạo thread tốn kém) và **giới hạn** số thread chạy cùng lúc. Không dùng `Executors.newCachedThreadPool()` cho việc không giới hạn được số lượng, cũng không dùng `newFixedThreadPool` với hàng đợi vô hạn khi việc đến nhanh hơn xử lý: hàng đợi phình lên tới hết RAM.
</details>

**J22. ⭐⭐ `ThreadPoolExecutor` có những tham số nào? Khi pool đầy thì sao?**
<details><summary>Gợi ý</summary>

`corePoolSize`, `maximumPoolSize`, `keepAliveTime`, `workQueue`, `RejectedExecutionHandler`. Thứ tự: dùng hết core thread → **đưa vào hàng đợi** → hàng đợi đầy mới tạo thêm thread tới max → vẫn đầy thì bị từ chối (`AbortPolicy` ném lỗi, `CallerRunsPolicy` bắt chính thread gọi chạy việc đó). Bẫy: hàng đợi vô hạn thì không bao giờ vượt quá core thread.
</details>

**J23. ⭐ `synchronized`, `volatile`, `AtomicInteger` khác nhau thế nào?**
<details><summary>Gợi ý</summary>

`volatile`: đảm bảo **thấy được** giá trị mới (visibility) và không bị sắp xếp lại lệnh, nhưng **không nguyên tử**: `count++` với `volatile` vẫn sai. `AtomicInteger`: nguyên tử bằng CAS, hợp cho bộ đếm. `synchronized`: khoá, cho một đoạn code chỉ một thread chạy, và cũng đảm bảo visibility. Có thêm `ReentrantLock` khi cần `tryLock` có timeout hoặc khoá công bằng.
</details>

**J24. ⭐⭐ Deadlock là gì? Phòng tránh thế nào? (Cả trong Java lẫn trong DB.)**
<details><summary>Gợi ý</summary>

Hai bên mỗi bên giữ một khoá và chờ khoá của bên kia. Phòng tránh: **luôn lấy khoá theo cùng một thứ tự**, có timeout (`tryLock`, `FOR UPDATE WAIT 3`), giữ khoá thật ngắn. Trong Oracle: hai transaction update hai dòng theo thứ tự ngược nhau sẽ bị `ORA-00060`. Ví dụ ở dự án: xác nhận huỷ nhiều đơn cùng lúc thì update theo thứ tự `order_id` tăng dần.
</details>

**J25. ⭐⭐ `CompletableFuture`: gọi song song 3 API rồi gộp kết quả.**
<details><summary>Gợi ý</summary>

```java
var a = CompletableFuture.supplyAsync(() -> stockApi.get(id), executor);
var b = CompletableFuture.supplyAsync(() -> pointApi.get(id), executor);
var c = CompletableFuture.supplyAsync(() -> couponApi.get(id), executor);
CompletableFuture.allOf(a, b, c).orTimeout(3, TimeUnit.SECONDS).join();
return new Summary(a.join(), b.join(), c.join());
```
Luôn truyền executor riêng (không dùng common pool), xử lý lỗi bằng `exceptionally`/`handle`. `orTimeout` có từ Java 9. Transaction và `ThreadLocal` (cả MDC log) **không đi theo** sang thread khác.
</details>

**J26. ⭐⭐ `ThreadLocal` là gì? Vì sao dùng với thread pool phải cẩn thận?**
<details><summary>Gợi ý</summary>

Mỗi thread một bản giá trị riêng. Spring dùng nó để giữ transaction, `SecurityContext`, `RequestContextHolder`. Với thread pool, thread được dùng lại: quên `remove()` thì request sau thấy dữ liệu của request trước (lộ dữ liệu người khác) và bị rò bộ nhớ. Luôn `remove()` trong `finally`.
</details>

**J27. ⭐⭐⭐ Virtual threads (Java 21).** → xem `A8`, `A9`.

---

## 4. Java: JVM và phiên bản

**J28. ⭐⭐ Bộ nhớ JVM gồm những vùng nào? GC hoạt động thế nào?**
<details><summary>Gợi ý</summary>

Heap (chia young/old: object mới sinh ở young, sống lâu thì lên old), stack của từng thread, metaspace (metadata của class). GC tìm object không còn ai tham chiếu (tính từ các GC root). G1 là mặc định từ Java 9; ZGC/Shenandoah cho thời gian dừng rất ngắn. Minor GC dọn young (nhanh), full GC dừng cả app lâu hơn. Xem thêm `A11`.
</details>

**J29. ⭐⭐ Java có rò bộ nhớ không? Tìm thế nào?**
<details><summary>Gợi ý</summary>

Có, khi object không còn dùng nhưng vẫn bị tham chiếu: `static` collection lớn dần, cache không giới hạn, `ThreadLocal` không `remove`, listener không gỡ, `ResultSet`/stream không đóng. Tìm bằng heap dump (`jmap -dump`, hoặc tự dump khi lỗi với `-XX:+HeapDumpOnOutOfMemoryError`) rồi mở bằng Eclipse MAT, xem object nào chiếm nhiều nhất và ai đang giữ nó. Quan sát bằng biểu đồ heap sau mỗi lần GC: đáy cứ cao dần là có rò.
</details>

**J30. ⭐⭐ Java 8 có gì mới? Từ Java 11, 17, 21 có gì đáng dùng?**
<details><summary>Gợi ý</summary>

- **8**: lambda, functional interface, Stream, `Optional`, `java.time` (thay `Date`/`Calendar`), `default` method trong interface.
- **11**: `var` (từ 10), `HttpClient`, `String.isBlank()`/`strip()`/`lines()`, `List.of` (từ 9).
- **17**: record, sealed class, text block, switch expression, pattern matching cho `instanceof`.
- **21**: virtual threads, pattern matching cho `switch`, record pattern, sequenced collections (`getFirst()`/`getLast()`).

Ở dự án đang dùng Java `___`. Biết vì sao chưa nâng cấp (Spring Boot 2.x, thư viện của khách, quy trình kiểm duyệt) là điểm cộng.
</details>

**J31. ⭐ Vì sao dùng `java.time` (`LocalDateTime`, `ZonedDateTime`, `Instant`) thay `Date`?**
<details><summary>Gợi ý</summary>

`Date` và `SimpleDateFormat` có thể sửa được và **không an toàn thread** (một `SimpleDateFormat` `static` dùng chung là bug kinh điển). `java.time` bất biến, rõ ràng: `LocalDate` (chỉ ngày), `LocalDateTime` (không có múi giờ), `Instant` (thời điểm tuyệt đối), `ZonedDateTime`. Hệ thống Hàn Quốc thì chú ý `Asia/Seoul` (UTC+9) so với giờ server.
</details>

---

## Trước khi đọc phần Spring

**Ví dụ dùng chung.** Mọi đoạn code bên dưới dùng cùng một nghiệp vụ cho dễ theo dõi: `CancelService` xác nhận huỷ đơn, gọi `OrderMapper` (MyBatis) và `CancelHistoryMapper`. Code là ví dụ minh hoạ, không phải code của dự án thật.

**Đọc tài liệu.** Link 📖 dẫn tới tài liệu chính thức bản mới nhất (Spring Framework 6, Spring Boot 3). Với Spring Boot 2.x như dự án đang dùng thì khái niệm vẫn y như vậy, chỉ khác mấy chỗ:
- `javax.*` (Boot 2) đổi thành `jakarta.*` (Boot 3), ví dụ `javax.validation.Valid` → `jakarta.validation.Valid`.
- Spring Security 5.7 trở về trước dùng `WebSecurityConfigurerAdapter` và `antMatchers`; bản mới dùng bean `SecurityFilterChain` và `requestMatchers`.
- Muốn đọc đúng bản 2.7 thì vào [Spring Boot 2.7 reference](https://docs.spring.io/spring-boot/docs/2.7.x/reference/html/).

**🧪 Thử tay** là những chỗ nên tự chạy. Tạo nhanh một project ở [start.spring.io](https://start.spring.io) (chọn Spring Web, JDBC API, H2), chép code vào rồi chạy. Tự thấy log in ra thì nhớ lâu hơn đọc nhiều.

---

## 5. Spring Core: IoC, DI, Bean

**SP1. ⭐ IoC và DI là gì? Lợi ích thật là gì?**
<details><summary>Gợi ý</summary>

IoC: container (`ApplicationContext`) tạo object và nối chúng với nhau, thay vì code tự `new`. DI là một cách làm IoC: phụ thuộc được truyền vào từ bên ngoài.

**Ví dụ:**
```java
// Không DI: CancelService tự tạo SmsSender, nên
// - muốn đổi sang push notification phải sửa CancelService
// - test thì gửi SMS thật
public class CancelService {
    private final SmsSender sender = new SmsSender();
}

// Có DI: CancelService chỉ biết interface, Spring truyền cài đặt vào
public interface NotificationSender { void send(String to, String msg); }

@Component
public class PushSender implements NotificationSender { ... }

@Service
public class CancelService {
    private final NotificationSender sender;
    public CancelService(NotificationSender sender) { this.sender = sender; }
}

// Trong test thì không cần Spring, truyền bản giả vào:
CancelService service = new CancelService((to, msg) -> {});
```
Lợi ích: đổi cài đặt mà không sửa code dùng nó, test dễ, và vì Spring giữ object nên nó bọc thêm được proxy (transaction, AOP).

📖 [Dependency Injection](https://docs.spring.io/spring-framework/reference/core/beans/dependencies/factory-collaborators.html)
</details>

**SP2. ⭐ Inject qua constructor, setter hay field? Vì sao nên dùng constructor?**
<details><summary>Gợi ý</summary>

```java
// Field injection: gọn, nhưng phụ thuộc bị giấu, field không final được,
// test phải dùng reflection hoặc dựng cả Spring
@Service
public class CancelService {
    @Autowired private OrderMapper orderMapper;
    @Autowired private CancelHistoryMapper historyMapper;
}

// Constructor injection: nên dùng
@Service
public class CancelService {
    private final OrderMapper orderMapper;
    private final CancelHistoryMapper historyMapper;

    // Chỉ có một constructor thì không cần @Autowired (từ Spring 4.3)
    public CancelService(OrderMapper orderMapper, CancelHistoryMapper historyMapper) {
        this.orderMapper = orderMapper;
        this.historyMapper = historyMapper;
    }
}

// Viết gọn bằng Lombok: sinh constructor cho mọi field final
@Service
@RequiredArgsConstructor
public class CancelService {
    private final OrderMapper orderMapper;
    private final CancelHistoryMapper historyMapper;
}
```
Lợi ích: field là `final`; thiếu bean thì app báo lỗi ngay lúc khởi động; test chỉ cần `new CancelService(mockA, mockB)`; constructor có tới 8 tham số là dấu hiệu class đang ôm quá nhiều việc.

📖 [Constructor-based hay setter-based DI](https://docs.spring.io/spring-framework/reference/core/beans/dependencies/factory-collaborators.html)
</details>

**SP3. ⭐ `@Component`, `@Service`, `@Repository`, `@Controller` khác nhau thế nào? `@Bean` thì sao?**
<details><summary>Gợi ý</summary>

Cả bốn đều là `@Component` để component scan tìm thấy, chỉ khác ở ý nghĩa. `@Repository` có thêm việc đổi exception của DB sang `DataAccessException` của Spring.

`@Bean` dùng khi object là của **thư viện bên ngoài** (không sửa class đó để gắn annotation được), hoặc khi việc tạo object cần cấu hình:
```java
@Configuration
public class HttpConfig {
    // RestTemplate là class của Spring, không gắn @Component vào được.
    // Tạo bằng @Bean để đặt timeout một chỗ, mọi nơi inject dùng chung.
    @Bean
    public RestTemplate paymentRestTemplate(RestTemplateBuilder builder) {
        return builder
                .rootUri("https://payment.example.com")
                .setConnectTimeout(Duration.ofSeconds(3))
                .setReadTimeout(Duration.ofSeconds(10))
                .build();
    }
}
```

📖 [`@Bean` và `@Configuration`](https://docs.spring.io/spring-framework/reference/core/beans/java/basic-concepts.html)
</details>

**SP4. ⭐ Có 2 bean cùng kiểu thì Spring inject cái nào?**
<details><summary>Gợi ý</summary>

Không làm gì thì app không khởi động được, lỗi `NoUniqueBeanDefinitionException: expected single matching bean but found 2`.

**Cách 1: `@Primary` hoặc `@Qualifier`**
```java
@Component @Primary
public class PushSender implements NotificationSender { ... }

@Component("smsSender")
public class SmsSender implements NotificationSender { ... }

@Service
public class CancelService {
    // Không ghi gì thì nhận PushSender (@Primary); muốn SMS thì chỉ đích danh
    public CancelService(@Qualifier("smsSender") NotificationSender sender) { ... }
}
```

**Cách 2: lấy tất cả rồi chọn theo key (Strategy pattern).** Hợp với hoàn tiền theo từng phương thức thanh toán:
```java
public interface RefundHandler {
    String paymentMethod();          // "CARD", "POINT", ...
    void refund(Order order);
}

@Component class CardRefundHandler  implements RefundHandler { ... }
@Component class PointRefundHandler implements RefundHandler { ... }

@Service
public class RefundService {
    private final Map<String, RefundHandler> handlers;

    public RefundService(List<RefundHandler> all) {   // Spring truyền mọi bean RefundHandler
        this.handlers = all.stream()
                .collect(Collectors.toMap(RefundHandler::paymentMethod, h -> h));
    }

    public void refund(Order order) {
        RefundHandler h = handlers.get(order.getPaymentMethod());
        if (h == null) throw new IllegalStateException("No refund handler for " + order.getPaymentMethod());
        h.refund(order);
    }
}
```
Thêm phương thức thanh toán mới thì chỉ thêm một class, không sửa `RefundService`.

**Với MyBatis, hay gặp khi có 2 DB:** mỗi DB một `SqlSessionFactory`, và mapper được gắn với factory theo package:
```java
@MapperScan(basePackages = "com.x.mapper.main", sqlSessionFactoryRef = "mainSqlSessionFactory")
```

📖 [`@Primary` và `@Qualifier`](https://docs.spring.io/spring-framework/reference/core/beans/annotation-config/autowired-qualifiers.html)
</details>

**SP5. ⭐ Bean scope có những loại nào? Mặc định là gì?**
<details><summary>Gợi ý</summary>

`singleton` (mặc định: cả app chỉ có một object), `prototype` (mỗi lần lấy là một object mới), và trong web có thêm `request`, `session`, `application`.

Vì service là singleton nên **mọi request dùng chung một object**. Giữ dữ liệu của request trong field là bug:
```java
@Service
public class CancelService {
    private Order current;              // BUG: field dùng chung cho mọi request

    public void confirm(long orderId) {
        current = orderMapper.findById(orderId);
        // ... trong lúc này request khác gọi confirm(), ghi đè current ...
        current.setStatus("CANCEL_CONFIRMED");
        orderMapper.update(current);    // có thể đang update nhầm đơn của người khác
    }
}

// Đúng: dữ liệu của request để trong biến cục bộ (mỗi thread một bản riêng)
public void confirm(long orderId) {
    Order order = orderMapper.findById(orderId);
    order.setStatus("CANCEL_CONFIRMED");
    orderMapper.update(order);
}
```
Bug này chỉ lộ ra khi có nhiều người dùng cùng lúc, nên test một mình thì không thấy. Khi maintain code cũ, gặp field không phải `final` trong `@Service` hay `@Controller` thì nên dừng lại kiểm tra.

📖 [Bean scopes](https://docs.spring.io/spring-framework/reference/core/beans/factory-scopes.html)
</details>

**SP6. ⭐⭐ Inject một bean `prototype` vào bean `singleton` thì sao?**
<details><summary>Gợi ý</summary>

```java
@Component
@Scope("prototype")
public class CancelReportBuilder {      // có state, mỗi lần dùng cần một object mới
    private final List<String> lines = new ArrayList<>();
    public void add(String line) { lines.add(line); }
    public String build() { return String.join("\n", lines); }
}

@Service
public class ReportService {
    private final CancelReportBuilder builder;   // SAI: chỉ inject MỘT lần khi tạo ReportService
    // => mọi lần gọi đều dùng chung một builder, các dòng cứ dồn lại
}

@Service
public class ReportService {
    private final ObjectProvider<CancelReportBuilder> builders;   // ĐÚNG

    public String report(List<Order> orders) {
        CancelReportBuilder b = builders.getObject();   // mỗi lần gọi là một object mới
        orders.forEach(o -> b.add(o.getOrderNo()));
        return b.build();
    }
}
```

📖 [Singleton bean dùng prototype bean](https://docs.spring.io/spring-framework/reference/core/beans/factory-scopes.html#beans-factory-scopes-sing-prot-interaction)
</details>

**SP7. ⭐⭐ Vòng đời của một bean.**
<details><summary>Gợi ý</summary>

Tạo object → inject phụ thuộc → các callback `*Aware` → `BeanPostProcessor` (trước init) → `@PostConstruct` → `BeanPostProcessor` (sau init: **chỗ Spring bọc proxy** cho `@Transactional`, `@Async`) → dùng → `@PreDestroy` khi tắt app.

**Ví dụ hay gặp:** nạp bảng mã cửa hàng vào bộ nhớ khi khởi động.
```java
@Component
public class StoreCodeCache {
    private final StoreMapper storeMapper;
    private volatile Map<String, String> names = Map.of();

    public StoreCodeCache(StoreMapper storeMapper) { this.storeMapper = storeMapper; }

    @PostConstruct      // Boot 2: javax.annotation.PostConstruct; Boot 3: jakarta.annotation.PostConstruct
    void load() {       // chạy một lần, sau khi storeMapper đã được inject
        names = storeMapper.findAll().stream()
                .collect(Collectors.toMap(Store::getCode, Store::getName));
    }

    @PreDestroy
    void close() { /* giải phóng tài nguyên nếu có */ }
}
```
Bẫy: trong `@PostConstruct`, proxy của chính bean đó chưa được tạo, nên gọi method `@Transactional` của chính nó ở đây sẽ không có transaction. Việc nặng chạy lúc khởi động thì dùng `ApplicationRunner` hoặc nghe event `ApplicationReadyEvent`.

📖 [Lifecycle callbacks](https://docs.spring.io/spring-framework/reference/core/beans/factory-nature.html)
</details>

**SP8. ⭐⭐ Circular dependency là gì? Spring Boot xử lý thế nào?**
<details><summary>Gợi ý</summary>

```java
@Service
public class OrderService {
    public OrderService(CancelService cancelService) { ... }   // Order cần Cancel
}

@Service
public class CancelService {
    public CancelService(OrderService orderService) { ... }    // Cancel lại cần Order
}
```
Từ Spring Boot 2.6, app **không khởi động được** và in ra:
```
The dependencies of some of the beans in the application context form a cycle:
┌─────┐
|  orderService
↑     ↓
|  cancelService
└─────┘
```
Sửa đúng: tách phần mà hai bên cùng cần ra một bean thứ ba (ví dụ `OrderQueryService` chỉ đọc đơn), hoặc dùng event (SP10) để một bên không cần biết bên kia. `@Lazy` hay `spring.main.allow-circular-references=true` chỉ là cách chữa cháy. Code cũ chạy được trên Boot 2.5 mà lên 2.6 thì chết là vì lý do này.

📖 [Circular dependencies](https://docs.spring.io/spring-framework/reference/core/beans/dependencies/factory-collaborators.html) · [Spring Boot 2.6 release notes](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.6-Release-Notes)
</details>

**SP9. ⭐⭐⭐ `@Configuration` khác `@Component` khi chứa method `@Bean` ở chỗ nào?**
<details><summary>Gợi ý</summary>

```java
@Configuration          // Spring bọc class này bằng CGLIB
public class AppConfig {
    @Bean public Clock clock() { return Clock.system(ZoneId.of("Asia/Seoul")); }

    @Bean public CancelPolicy cancelPolicy() {
        return new CancelPolicy(clock());   // vẫn nhận lại ĐÚNG bean clock trong container
    }
}
```
Đổi `@Configuration` thành `@Component` (hoặc `@Configuration(proxyBeanMethods = false)`) thì `clock()` là lời gọi Java bình thường và tạo ra một `Clock` **mới**, khác với bean `clock` đã đăng ký. Với object có state (connection pool chẳng hạn) thì thành ra có hai pool.

📖 [`@Configuration`](https://docs.spring.io/spring-framework/reference/core/beans/java/configuration-annotation.html)
</details>

**SP10. ⭐⭐ `ApplicationEvent` và `@TransactionalEventListener` dùng làm gì?** *(gắn với xác nhận huỷ)*
<details><summary>Gợi ý</summary>

```java
public record CancelConfirmedEvent(long orderId, String customerId) {}

@Service
@RequiredArgsConstructor
public class CancelService {
    private final OrderMapper orderMapper;
    private final ApplicationEventPublisher events;

    @Transactional
    public void confirm(long orderId, String userId) {
        Order order = orderMapper.findById(orderId);
        // ... cập nhật trạng thái, trả tồn kho, ghi lịch sử ...
        events.publishEvent(new CancelConfirmedEvent(orderId, order.getCustomerId()));
        // Không gửi push ở đây: lỡ phía dưới lỗi và rollback thì khách
        // đã nhận thông báo "huỷ thành công" cho một đơn chưa huỷ.
    }
}

@Component
@RequiredArgsConstructor
public class CancelNotificationListener {
    private final NotificationSender sender;

    @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)  // chỉ chạy khi DB đã commit
    public void on(CancelConfirmedEvent e) {
        sender.send(e.customerId(), "Đơn " + e.orderId() + " đã được huỷ");
    }
}
```
Lợi ích thêm: `CancelService` không cần biết có những việc gì chạy sau khi huỷ (push, SMS, ghi thống kê). Thêm việc mới thì thêm một listener. Muốn listener không làm chậm response thì thêm `@Async`.

📖 [Transaction-bound events](https://docs.spring.io/spring-framework/reference/data-access/transaction/event.html) · [ApplicationContext và event](https://docs.spring.io/spring-framework/reference/core/beans/context-introduction.html)
</details>

---

## 6. Spring AOP và proxy

**SP11. ⭐ AOP là gì? Spring dùng nó ở đâu?**
<details><summary>Gợi ý</summary>

Tách các việc lặp lại ở nhiều chỗ (transaction, log, đo thời gian, kiểm quyền, cache) ra khỏi code nghiệp vụ. Spring tự dùng nó cho `@Transactional`, `@Async`, `@Cacheable`, `@PreAuthorize`.

**Ví dụ:** log thời gian chạy của mọi method trong service, không phải sửa từng method (cần `spring-boot-starter-aop`):
```java
@Aspect
@Component
@Slf4j
public class TimingAspect {
    @Around("execution(* com.x.service..*(..))")   // mọi method trong package service
    public Object time(ProceedingJoinPoint pjp) throws Throwable {
        long start = System.nanoTime();
        try {
            return pjp.proceed();                    // gọi method thật
        } finally {
            long ms = (System.nanoTime() - start) / 1_000_000;
            if (ms > 500) log.warn("{} took {} ms", pjp.getSignature().toShortString(), ms);
        }
    }
}
```
Đây cũng là cách đơn giản để tìm ra chức năng chậm (trước khi đi tìm N+1).

📖 [@AspectJ trong Spring](https://docs.spring.io/spring-framework/reference/core/aop/ataspectj.html)
</details>

**SP12. ⭐ Vì sao gọi method `@Transactional` (hoặc `@Async`, `@Cacheable`) từ method khác trong cùng class thì không có tác dụng?**
<details><summary>Gợi ý</summary>

Bean bro inject vào controller thực ra là một **proxy** bọc ngoài object thật. Proxy mở transaction rồi mới gọi vào object thật. Còn `this.method()` là gọi thẳng trong object thật, không đi qua proxy:

```
Controller ──> [Proxy: mở transaction] ──> CancelService thật
                                              │
                                              └── this.saveHistory()  ← không đi qua proxy
```

```java
@Service
public class CancelService {
    public void confirmAll(List<Long> ids) {           // không có @Transactional
        for (long id : ids) {
            this.confirmOne(id);                       // SAI: gọi thẳng, KHÔNG có transaction
        }
    }

    @Transactional
    public void confirmOne(long id) {
        orderMapper.confirmCancel(id);
        stockMapper.restore(id);                       // lỗi ở đây: dòng trên KHÔNG rollback
    }
}
```
Sửa sạch nhất: chuyển `confirmOne` sang một bean khác (ví dụ `CancelWorker`) rồi inject vào. Cách khác: dùng `TransactionTemplate`:
```java
transactionTemplate.executeWithoutResult(status -> {
    orderMapper.confirmCancel(id);
    stockMapper.restore(id);
});
```

🧪 **Thử tay:** in ra trong method xem có transaction không:
```java
log.info("tx active = {}", TransactionSynchronizationManager.isActualTransactionActive());
```
Gọi từ controller (qua proxy) thì in `true`, gọi qua `this.` thì in `false`. Hoặc bật log `logging.level.org.springframework.transaction.interceptor=TRACE` để xem dòng `Getting transaction for [...]`.

📖 [Understanding AOP proxies](https://docs.spring.io/spring-framework/reference/core/aop/proxying.html) (có đúng ví dụ self-invocation) · [Dùng `@Transactional`](https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/annotations.html)
</details>

**SP13. ⭐⭐ JDK dynamic proxy khác CGLIB thế nào?**
<details><summary>Gợi ý</summary>

JDK proxy chỉ proxy được qua interface. CGLIB tạo một **lớp con** của class, nên không proxy được method `final`/`private` và class `final`. Spring Boot 2 trở lên mặc định dùng CGLIB.

🧪 **Thử tay:**
```java
@Component
@RequiredArgsConstructor
class ProxyCheck implements ApplicationRunner {
    private final CancelService cancelService;
    public void run(ApplicationArguments args) {
        System.out.println(cancelService.getClass());
        // Spring 5 (Boot 2): class com.x.CancelService$$EnhancerBySpringCGLIB$$1a2b3c
        // Spring 6 (Boot 3): class com.x.CancelService$$SpringCGLIB$$0
        // Không có @Transactional/@Async nào thì in ra đúng class gốc: không có proxy
    }
}
```
Hệ quả: `@Transactional` đặt trên method `private` hay `final` bị **bỏ qua mà không báo lỗi gì**.

📖 [Proxying mechanisms](https://docs.spring.io/spring-framework/reference/core/aop/proxying.html)
</details>

---

## 7. Spring Boot

**SP14. ⭐ Spring Boot khác Spring Framework chỗ nào? `@SpringBootApplication` gồm gì?**
<details><summary>Gợi ý</summary>

Boot = Spring + auto-configuration + starter (bộ phụ thuộc đi kèm nhau) + server nhúng (Tomcat) + cấu hình bên ngoài + Actuator.

`@SpringBootApplication` = `@Configuration` + `@EnableAutoConfiguration` + `@ComponentScan`. Component scan quét **từ package của class main trở xuống**:
```
com.x.retail
├── RetailApplication.java   ← @SpringBootApplication ở đây
├── order/CancelService.java     ✓ được quét
└── config/WebConfig.java        ✓ được quét
com.x.common
└── util/DateUtil.java           ✗ KHÔNG được quét (nằm ngoài com.x.retail)
```
Bean nằm ngoài thì báo lỗi `required a bean of type '...' that could not be found`.

📖 [Dùng `@SpringBootApplication`](https://docs.spring.io/spring-boot/reference/using/using-the-springbootapplication-annotation.html)
</details>

**SP15. ⭐⭐ Auto-configuration hoạt động thế nào?**
<details><summary>Gợi ý</summary>

Boot có sẵn rất nhiều class cấu hình, mỗi class kèm điều kiện. Ví dụ, viết lại cho dễ hiểu (không phải code thật của Boot):
```java
@AutoConfiguration
@ConditionalOnClass(SqlSessionFactory.class)       // classpath có MyBatis không?
@ConditionalOnSingleCandidate(DataSource.class)    // có đúng một DataSource không?
public class MybatisAutoConfiguration {
    @Bean
    @ConditionalOnMissingBean                       // người dùng chưa tự khai báo thì mới tạo
    public SqlSessionFactory sqlSessionFactory(DataSource ds) { ... }
}
```
Vì vậy chỉ cần thêm `mybatis-spring-boot-starter` và cấu hình `spring.datasource.*` là đã có `SqlSessionFactory`. Tự khai báo bean `SqlSessionFactory` thì bản của Boot lùi lại.

🧪 **Thử tay:** chạy app với `--debug` (hoặc đặt `debug=true` trong `application.yml`). Log in ra `CONDITIONS EVALUATION REPORT`, chia hai phần `Positive matches` (cấu hình nào đã chạy và vì sao) và `Negative matches` (cấu hình nào bị bỏ và vì sao). Đây là cách nhanh nhất để biết vì sao một bean "tự nhiên có" hoặc "tự nhiên không có".

📖 [Viết auto-configuration](https://docs.spring.io/spring-boot/reference/features/developing-auto-configuration.html)
</details>

**SP16. ⭐ Cấu hình theo môi trường (dev, stg, prod) làm thế nào? Thứ tự ưu tiên ra sao?**
<details><summary>Gợi ý</summary>

```yaml
# application.yml: phần dùng chung
mybatis:
  mapper-locations: classpath:mapper/**/*.xml
  configuration:
    map-underscore-to-camel-case: true      # cột order_no ↔ field orderNo

# application-dev.yml
spring:
  datasource:
    url: jdbc:oracle:thin:@localhost:1521/XEPDB1
    username: dev
    password: dev

# application-prod.yml: không ghi mật khẩu thật vào file
spring:
  datasource:
    url: ${DB_URL}
    username: ${DB_USER}
    password: ${DB_PASSWORD}
```
Chạy: `java -jar app.jar --spring.profiles.active=prod`, hoặc đặt biến môi trường `SPRING_PROFILES_ACTIVE=prod`.

Ưu tiên từ cao xuống (cái trên ghi đè cái dưới): tham số dòng lệnh → biến môi trường → `application-{profile}.yml` → `application.yml`. Vì vậy trên server chỉ cần đặt biến môi trường `SPRING_DATASOURCE_PASSWORD` là ghi đè được giá trị trong file.

📖 [Externalized configuration](https://docs.spring.io/spring-boot/reference/features/external-config.html) (có danh sách đủ thứ tự ưu tiên) · [Profiles](https://docs.spring.io/spring-boot/reference/features/profiles.html)
</details>

**SP17. ⭐⭐ Actuator dùng làm gì? Có rủi ro gì?**
<details><summary>Gợi ý</summary>

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics      # chỉ mở những gì cần
  endpoint:
    health:
      show-details: when-authorized       # chi tiết (trạng thái DB...) chỉ cho người đã đăng nhập
```
Gọi `GET /actuator/health` trả `{"status":"UP"}`. Load balancer dùng nó để biết server nào còn sống. Health tự kiểm cả kết nối Oracle.

Rủi ro: `include: "*"` sẽ mở `/actuator/env` (lộ cấu hình), `/actuator/heapdump` (tải về được cả bộ nhớ, trong đó có mật khẩu và dữ liệu khách).

📖 [Actuator endpoints](https://docs.spring.io/spring-boot/reference/actuator/endpoints.html)
</details>

**SP18. ⭐⭐ Log trong Spring Boot: SLF4J, Logback, MDC.**
<details><summary>Gợi ý</summary>

```java
log.info("Cancel confirmed: order=" + orderId);   // SAI: cộng chuỗi kể cả khi level INFO đang tắt
log.info("Cancel confirmed: order={}", orderId);  // ĐÚNG: placeholder
log.error("Refund failed: order={}", orderId, e); // exception để cuối cùng thì in được stack trace
```

**MDC:** gắn một mã cho mỗi request để lọc được toàn bộ log của một lần xác nhận huỷ:
```java
@Component
@Order(Ordered.HIGHEST_PRECEDENCE)
public class RequestIdFilter extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(HttpServletRequest req, HttpServletResponse res, FilterChain chain)
            throws ServletException, IOException {
        String id = Optional.ofNullable(req.getHeader("X-Request-Id"))      // app Android gửi lên
                .orElse(UUID.randomUUID().toString());
        MDC.put("requestId", id);
        try {
            chain.doFilter(req, res);
        } finally {
            MDC.remove("requestId");   // thread được dùng lại cho request khác (xem J26)
        }
    }
}
```
```yaml
logging:
  pattern:
    level: "%5p [%X{requestId}]"     # mọi dòng log có thêm [requestId]
```

📖 [Logging](https://docs.spring.io/spring-boot/reference/features/logging.html)
</details>

---

## 8. Spring MVC và REST API

**SP19. ⭐ Một request đi qua Spring MVC thế nào (`DispatcherServlet`)?**
<details><summary>Gợi ý</summary>

```
App Android
   │  POST /api/orders/123/cancel-confirmation
   ▼
Filter (RequestIdFilter, Spring Security...)       ← tầng Servlet
   ▼
DispatcherServlet
   ├─ HandlerMapping: URL này → CancelController.confirm()
   ├─ Interceptor.preHandle()                       ← tầng Spring MVC
   ├─ HandlerAdapter: đổi JSON → DTO (Jackson), chạy @Valid, gọi method
   ├─ Kết quả trả về → HttpMessageConverter → JSON
   ├─ Interceptor.postHandle()
   └─ Interceptor.afterCompletion()
   (Có exception thì @ExceptionHandler / @RestControllerAdvice xử lý)
```

Ví dụ interceptor đo thời gian theo từng controller method:
```java
public class TimingInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest req, HttpServletResponse res, Object handler) {
        req.setAttribute("start", System.currentTimeMillis());
        return true;                     // false thì dừng, không gọi controller
    }
    @Override
    public void afterCompletion(HttpServletRequest req, HttpServletResponse res, Object handler, Exception ex) {
        long ms = System.currentTimeMillis() - (long) req.getAttribute("start");
        log.info("{} {} -> {} ({} ms)", req.getMethod(), req.getRequestURI(), res.getStatus(), ms);
    }
}

@Configuration
public class WebMvcConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new TimingInterceptor()).addPathPatterns("/api/**");
    }
}
```

📖 [DispatcherServlet](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-servlet.html) · [Interceptors](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-servlet/handlermapping-interceptor.html)
</details>

**SP20. ⭐ `@Controller` khác `@RestController`? `@RequestParam`, `@PathVariable`, `@RequestBody` khác nhau thế nào?**
<details><summary>Gợi ý</summary>

```java
@RestController                         // = @Controller + @ResponseBody: trả dữ liệu (JSON), không trả trang HTML
@RequestMapping("/api/orders")
@RequiredArgsConstructor
public class CancelController {
    private final CancelService cancelService;

    // GET /api/orders/cancel-requests?storeCode=S001&page=0
    @GetMapping("/cancel-requests")
    public PageResponse<CancelRequestDto> list(
            @RequestParam String storeCode,                      // ?storeCode=...
            @RequestParam(defaultValue = "0") int page) { ... }

    // POST /api/orders/123/cancel-confirmation   body: {"reason":"..."}
    @PostMapping("/{orderId}/cancel-confirmation")
    public ResponseEntity<CancelResult> confirm(
            @PathVariable long orderId,                          // một phần của URL
            @Valid @RequestBody CancelConfirmRequest body) {     // body JSON → object
        return ResponseEntity.ok(cancelService.confirm(orderId, body));
    }
}
```

📖 [Tham số của handler method](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-methods.html)
</details>

**SP21. ⭐ Chọn HTTP method và status code thế nào? API xác nhận huỷ nên trả gì?**
<details><summary>Gợi ý</summary>

| Tình huống | Status |
|---|---|
| Xác nhận thành công | `200 OK` (kèm trạng thái mới) |
| Tạo yêu cầu huỷ mới | `201 Created` |
| Body thiếu field, sai định dạng | `400 Bad Request` |
| Chưa đăng nhập, token hết hạn | `401 Unauthorized` |
| Đăng nhập rồi nhưng không có quyền xác nhận | `403 Forbidden` |
| Không có đơn này | `404 Not Found` |
| Đơn đã xác nhận rồi, hoặc không ở trạng thái chờ huỷ | `409 Conflict` |
| Lỗi không lường trước | `500 Internal Server Error` |

```java
// Trong service
int updated = orderMapper.confirmCancel(orderId, userId);   // UPDATE ... WHERE status = 'CANCEL_REQUESTED'
if (updated == 0) {
    throw new BusinessException(ErrorCode.INVALID_ORDER_STATUS);   // advice đổi thành 409 (xem SP22)
}
```
Bấm hai lần do mạng chậm: trả `200` kèm "đã xác nhận" (idempotent, `Q3`) hay trả `409` đều được, miễn là thống nhất với app Android và ghi trong tài liệu API.

📖 [MDN: HTTP response status codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)
</details>

**SP22. ⭐ Xử lý lỗi tập trung thế nào? Trả lỗi cho app Android ra sao?**
<details><summary>Gợi ý</summary>

```java
@Getter
@RequiredArgsConstructor
public enum ErrorCode {
    ORDER_NOT_FOUND(HttpStatus.NOT_FOUND, "Không tìm thấy đơn hàng"),
    INVALID_ORDER_STATUS(HttpStatus.CONFLICT, "Đơn không ở trạng thái chờ huỷ"),
    NO_PERMISSION(HttpStatus.FORBIDDEN, "Không có quyền xác nhận huỷ");

    private final HttpStatus status;
    private final String message;
}

@Getter
public class BusinessException extends RuntimeException {   // unchecked: transaction rollback (Q4)
    private final ErrorCode code;
    public BusinessException(ErrorCode code) { super(code.getMessage()); this.code = code; }
}

public record ErrorResponse(String code, String message) {}

@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {
    @ExceptionHandler(BusinessException.class)
    public ResponseEntity<ErrorResponse> business(BusinessException e) {
        return ResponseEntity.status(e.getCode().getStatus())
                .body(new ErrorResponse(e.getCode().name(), e.getMessage()));
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> invalid(MethodArgumentNotValidException e) {
        String msg = e.getBindingResult().getFieldErrors().stream()
                .map(f -> f.getField() + ": " + f.getDefaultMessage())
                .findFirst().orElse("invalid request");
        return ResponseEntity.badRequest().body(new ErrorResponse("INVALID_REQUEST", msg));
    }

    @ExceptionHandler(Exception.class)          // lưới cuối cùng
    public ResponseEntity<ErrorResponse> unknown(Exception e) {
        log.error("Unhandled error", e);        // stack trace chỉ nằm trong log
        return ResponseEntity.internalServerError()
                .body(new ErrorResponse("INTERNAL_ERROR", "Có lỗi xảy ra, vui lòng thử lại"));
    }
}
```
App Android nhận `{"code": "INVALID_ORDER_STATUS", "message": "..."}` và hiển thị theo `code`.

📖 [`@ExceptionHandler`](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-exceptionhandler.html)
</details>

**SP23. ⭐ Validate input: `@Valid` khác `@Validated` thế nào?**
<details><summary>Gợi ý</summary>

```java
public class CancelConfirmRequest {
    @NotBlank @Size(max = 200)
    private String reason;

    @NotNull @Positive
    private Integer quantity;

    @Valid                          // không có dòng này thì các item bên trong KHÔNG được kiểm
    @NotEmpty
    private List<CancelItem> items;
}
```
`@Valid` (chuẩn Jakarta, Boot 2 là `javax.validation`): kiểm `@RequestBody` và đi vào các object lồng bên trong.

`@Validated` (của Spring) làm được thêm hai việc:
```java
// 1. Kiểm tham số đơn lẻ: phải đặt @Validated trên class
@RestController
@Validated
public class CancelController {
    @GetMapping("/cancel-requests")
    public List<CancelRequestDto> list(@RequestParam @Min(0) int page,
                                       @RequestParam @Max(100) int size) { ... }
}

// 2. Validation group: luật cho tạo mới khác luật cho cập nhật
public interface OnCreate {}
public interface OnUpdate {}

public class StoreDto {
    @Null(groups = OnCreate.class) @NotNull(groups = OnUpdate.class)
    private Long id;
}

@PostMapping public void create(@Validated(OnCreate.class) @RequestBody StoreDto dto) { ... }
@PutMapping  public void update(@Validated(OnUpdate.class) @RequestBody StoreDto dto) { ... }
```

📖 [Validation trong Spring MVC](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-validation.html) · [Bean Validation](https://docs.spring.io/spring-framework/reference/core/validation/beanvalidation.html)
</details>

**SP24. ⭐⭐ Thay đổi API mà không làm hỏng app Android bản cũ thế nào?**
<details><summary>Gợi ý</summary>

```java
// v1: app cũ đang gửi
public class CancelConfirmRequest {
    private String reason;
}

// Thêm field mới: KHÔNG bắt buộc, app cũ không gửi vẫn chạy
public class CancelConfirmRequest {
    private String reason;
    private String reasonCode;      // có thể null; service tự xử lý khi null
}

// Response: chỉ thêm field. App cũ gặp field lạ thì bỏ qua
// (Spring Boot mặc định cấu hình Jackson với FAIL_ON_UNKNOWN_PROPERTIES = false;
//  phía Android dùng Gson thì mặc định cũng bỏ qua field lạ).
```
Những việc **không được làm** với API đang có app dùng: đổi tên field, đổi kiểu (`"qty": "1"` sang `"qty": 1`), xoá field, biến field không bắt buộc thành bắt buộc. Cần đổi lớn thì làm `/api/v2/...` và giữ v1 cho tới khi phần lớn người dùng đã cập nhật app.

📖 [Spring Boot: cấu hình Jackson](https://docs.spring.io/spring-boot/reference/web/servlet.html)
</details>

**SP25. ⭐⭐ Phân trang API danh sách đơn thế nào?**
<details><summary>Gợi ý</summary>

```java
public record PageResponse<T>(List<T> items, int page, int size, long totalCount) {}

@GetMapping("/cancel-requests")
public PageResponse<CancelRequestDto> list(@RequestParam String storeCode,
                                           @RequestParam(defaultValue = "0") int page,
                                           @RequestParam(defaultValue = "20") int size) {
    size = Math.min(size, 100);                      // chặn app xin 1 triệu dòng
    int offset = page * size;
    List<CancelRequestDto> items = cancelMapper.findRequests(storeCode, offset, size);
    long total = cancelMapper.countRequests(storeCode);
    return new PageResponse<>(items, page, size, total);
}
```
```xml
<select id="findRequests" resultType="CancelRequestDto">
  SELECT order_id, order_no, requested_at, reason
  FROM cancel_requests
  WHERE store_code = #{storeCode}
  ORDER BY requested_at DESC, order_id DESC   <!-- thêm cột duy nhất để thứ tự không đổi giữa các trang -->
  OFFSET #{offset} ROWS FETCH NEXT #{size} ROWS ONLY   <!-- Oracle 12c+, bản cũ dùng ROWNUM (Q19) -->
</select>
```
App cuộn vô hạn thì không cần `totalCount` (bớt được một câu `COUNT(*)`), trả `hasNext` là đủ.
</details>

**SP26. ⭐⭐ Upload file (ảnh chứng từ huỷ chẳng hạn) cần chú ý gì?**
<details><summary>Gợi ý</summary>

```yaml
spring:
  servlet:
    multipart:
      max-file-size: 5MB
      max-request-size: 10MB
```
```java
@PostMapping(value = "/{orderId}/cancel-evidence", consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
public void upload(@PathVariable long orderId, @RequestParam("file") MultipartFile file) throws IOException {
    String type = file.getContentType();          // app gửi lên, không tin hoàn toàn
    if (!List.of("image/jpeg", "image/png").contains(type)) {
        throw new BusinessException(ErrorCode.INVALID_FILE);
    }
    String key = "cancel/" + orderId + "/" + UUID.randomUUID() + ".jpg";   // không dùng file.getOriginalFilename()
    storage.save(key, file.getInputStream());     // S3, NAS..., không lưu vào DB hay thư mục của app
}
```
Tên file gốc có thể là `../../etc/passwd` (path traversal), nên luôn tự đặt tên.

📖 [Spring Boot: multipart](https://docs.spring.io/spring-boot/reference/web/servlet.html)
</details>

---

## 9. Transaction (Spring)

Phần cơ bản đã có ở `Q4` (MyBatis), `C2`, `C4`, `C6`. Các câu sau đào sâu hơn.

**SP27. ⭐ Các loại propagation, và ví dụ dùng từng loại.**
<details><summary>Gợi ý</summary>

```java
@Service
@RequiredArgsConstructor
public class CancelService {
    private final OrderMapper orderMapper;
    private final AuditService auditService;     // bean KHÁC (xem SP12)

    @Transactional                                // REQUIRED (mặc định)
    public void confirm(long orderId, String userId) {
        auditService.log(orderId, userId, "CONFIRM_TRY");   // REQUIRES_NEW: commit ngay, riêng
        orderMapper.confirmCancel(orderId, userId);
        stockMapper.restore(orderId);             // lỗi ở đây: rollback confirmCancel,
                                                  // nhưng dòng audit VẪN CÒN
    }
}

@Service
public class AuditService {
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void log(long orderId, String userId, String action) {
        auditMapper.insert(orderId, userId, action);
    }
}
```

| Propagation | Ý nghĩa | Ví dụ |
|---|---|---|
| `REQUIRED` | Có thì dùng chung, chưa có thì tạo mới | Hầu hết method service |
| `REQUIRES_NEW` | Luôn tạo mới, tạm dừng cái đang có | Audit log, lịch sử thao tác phải lưu dù nghiệp vụ lỗi |
| `NESTED` | Savepoint trong transaction đang có | Huỷ từng món trong đơn, món nào lỗi thì chỉ bỏ món đó |
| `SUPPORTS`, `MANDATORY`, `NOT_SUPPORTED`, `NEVER` | Ít dùng | Biết tên là đủ |

Lưu ý: `REQUIRES_NEW` lấy thêm **một connection nữa** từ pool trong lúc connection cũ vẫn bị giữ. Dùng nhiều dưới tải cao có thể làm cạn pool (SP36). `NESTED` chạy được với `DataSourceTransactionManager` (MyBatis/JDBC), JPA thì không.

📖 [Transaction propagation](https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/tx-propagation.html) · [MyBatis-Spring: transactions](https://mybatis.org/spring/transactions.html)
</details>

**SP28. ⭐⭐ Lỗi `UnexpectedRollbackException: Transaction silently rolled back because it has been marked as rollback-only` là gì?**
<details><summary>Gợi ý</summary>

```java
@Service
public class CancelService {
    @Transactional
    public void confirmAll(List<Long> ids) {
        for (long id : ids) {
            try {
                cancelWorker.confirmOne(id);      // bean khác, cũng @Transactional (REQUIRED)
            } catch (BusinessException e) {
                log.warn("skip order {}", id);    // tưởng là bỏ qua đơn lỗi, làm tiếp đơn khác
            }
        }
    }   // ← tới đây commit thì ném UnexpectedRollbackException, KHÔNG đơn nào được lưu
}
```
Vì sao: `confirmOne` dùng **chung** transaction với `confirmAll`. Khi nó ném `RuntimeException`, Spring đánh dấu cả transaction là "chỉ được rollback". `confirmAll` bắt lỗi rồi chạy tiếp, nhưng tới lúc commit thì Spring phát hiện và rollback hết.

Sửa, tuỳ nghiệp vụ muốn gì:
- Muốn mỗi đơn độc lập (đơn lỗi thì bỏ, đơn khác vẫn lưu): cho `confirmOne` dùng `REQUIRES_NEW`, hoặc bỏ `@Transactional` ở `confirmAll`.
- Muốn tất cả hoặc không gì cả: đừng `catch`, cho lỗi bay lên.

📖 [Rolling back](https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/rolling-back.html) · [Propagation: phần "rollback-only"](https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/tx-propagation.html)
</details>

**SP29. ⭐⭐ Isolation level: Spring khai báo thế nào? Oracle hỗ trợ những mức nào?**
<details><summary>Gợi ý</summary>

```java
@Transactional(isolation = Isolation.READ_COMMITTED)   // mặc định của Oracle, thường không cần ghi
```

| Hiện tượng | Ví dụ |
|---|---|
| Dirty read | Đọc được dữ liệu người khác **chưa commit** (Oracle không bao giờ cho) |
| Non-repeatable read | Trong một transaction, đọc lại cùng một dòng mà giá trị đã khác |
| Phantom read | Đọc lại cùng một điều kiện mà có thêm hoặc mất dòng |

Oracle chỉ có `READ COMMITTED` (mặc định) và `SERIALIZABLE`. Thực tế hầu như để mặc định, còn tranh chấp thì giải bằng update có điều kiện hoặc `SELECT ... FOR UPDATE` (`Q2`). Nói được câu này là đủ cho phần lớn buổi phỏng vấn.

📖 [`@Transactional` settings](https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/annotations.html)
</details>

**SP30. ⭐⭐ Transaction dài gây ra vấn đề gì? `timeout` dùng thế nào?**
<details><summary>Gợi ý</summary>

```java
// SAI: gọi API thanh toán (có thể mất 10 giây) bên trong transaction
@Transactional
public void confirm(long orderId) {
    orderMapper.lockForUpdate(orderId);          // SELECT ... FOR UPDATE: dòng bị khoá
    paymentClient.refund(orderId);               // trong 10 giây này dòng bị khoá, connection bị giữ
    orderMapper.confirmCancel(orderId);
}

// TỐT HƠN: transaction ngắn, gọi bên ngoài sau khi commit
public void confirm(long orderId) {
    cancelWorker.markRefunding(orderId);         // @Transactional: đổi trạng thái → REFUNDING, commit
    RefundResult r = paymentClient.refund(orderId);   // không có transaction nào đang mở
    cancelWorker.finish(orderId, r);             // @Transactional: CANCEL_CONFIRMED hoặc REFUND_FAILED
}
// Thêm một job đối soát quét các đơn kẹt ở REFUNDING quá lâu.
```
Đặt chặn trên: `@Transactional(timeout = 5)` (đơn vị là giây).

📖 [`@Transactional` settings](https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/annotations.html)
</details>

**SP31. ⭐⭐⭐ `@Transactional` đặt ở controller, service hay mapper?**
<details><summary>Gợi ý</summary>

Ở **service**, vì đó là ranh giới của một nghiệp vụ:
```java
@Transactional          // xác nhận huỷ = 3 việc, phải cùng thành công hoặc cùng huỷ
public void confirm(long orderId, String userId) {
    orderMapper.confirmCancel(orderId, userId);
    stockMapper.restore(orderId);
    historyMapper.insert(orderId, userId, "CONFIRMED");
}
```
Đặt ở controller: transaction ôm luôn việc đổi JSON, kiểm tra đầu vào, và những việc không liên quan. Không đặt gì cả (mỗi câu SQL tự commit): dòng thứ 2 lỗi thì dòng thứ 1 đã commit rồi, dữ liệu lệch. Đặt trên class thì áp cho mọi method `public`; đặt trên method thì ghi đè cấu hình của class. Method chỉ đọc thì thêm `readOnly = true` (`C6`).
</details>

---

## 10. Spring Security

**SP32. ⭐ Authentication khác authorization thế nào? Spring Security chạy ở đâu trong request?**
<details><summary>Gợi ý</summary>

Authentication: bạn là ai (đăng nhập). Authorization: bạn được làm gì (vai trò, quyền).

```
Request ─> DelegatingFilterProxy ─> FilterChainProxy ─> [các filter của Security] ─> DispatcherServlet
                                                          ├─ đọc token / session
                                                          ├─ xác thực → SecurityContextHolder
                                                          └─ kiểm quyền theo URL
```
Cấu hình kiểu mới (Spring Security 5.7 trở lên, Boot 2.7 trở lên):
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf().disable()                                   // API stateless (SP34)
            .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            .and()
            .authorizeHttpRequests(auth -> auth
                .antMatchers("/actuator/health", "/api/auth/login").permitAll()   // Security 6: requestMatchers
                .antMatchers("/api/orders/*/cancel-confirmation").hasRole("STORE_MANAGER")
                .anyRequest().authenticated())
            .addFilterBefore(jwtFilter, UsernamePasswordAuthenticationFilter.class);
        return http.build();
    }
}
```
Lấy người đang đăng nhập trong controller: `@AuthenticationPrincipal UserDetails user`.

📖 [Kiến trúc Spring Security (servlet)](https://docs.spring.io/spring-security/reference/servlet/architecture.html) · [Phân quyền theo URL](https://docs.spring.io/spring-security/reference/servlet/authorization/authorize-http-requests.html)
</details>

**SP33. ⭐⭐ API cho app di động: session hay JWT? Ưu nhược điểm?**
<details><summary>Gợi ý</summary>

```
1. App: POST /api/auth/login {id, password}
2. Server: kiểm mật khẩu → trả accessToken (sống 30 phút) + refreshToken (sống 14 ngày)
3. App: mọi request gửi header  Authorization: Bearer <accessToken>
4. Server (jwtFilter): kiểm chữ ký + hạn → đặt user vào SecurityContextHolder
5. accessToken hết hạn → app gọi POST /api/auth/refresh với refreshToken → nhận token mới
```

| | Session | JWT |
|---|---|---|
| State | Nằm ở server | Nằm trong token (stateless) |
| Nhiều server | Phải chia sẻ session (Redis) | Server nào cũng tự kiểm được |
| Thu hồi ngay (khoá tài khoản) | Dễ: xoá session | Khó: phải chờ hết hạn, hoặc giữ danh sách đen |

JWT chỉ **ký**, không mã hoá: nội dung chỉ là base64, ai cũng đọc được (thử dán vào jwt.io). Không bỏ dữ liệu nhạy cảm vào. Ở dự án dùng `___`.
</details>

**SP34. ⭐⭐ Lưu mật khẩu thế nào? Vì sao API cho app di động thường tắt CSRF?**
<details><summary>Gợi ý</summary>

```java
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}

String hash = encoder.encode("raw-password");        // $2a$10$... mỗi lần ra một chuỗi khác (có salt)
boolean ok = encoder.matches("raw-password", hash);  // so bằng matches, không encode rồi so chuỗi
```
Không dùng MD5, SHA-256 trần (quá nhanh, dễ dò), không mã hoá hai chiều.

CSRF: trang web xấu lừa browser gửi request tới server của mình, và **browser tự đính kèm cookie** đăng nhập. App Android gửi token qua header `Authorization` (app tự gắn, không có gì tự đính kèm), nên không bị kiểu tấn công này, tắt CSRF được. Còn nếu đăng nhập bằng cookie session (web admin chẳng hạn) thì phải bật.

📖 [Password storage](https://docs.spring.io/spring-security/reference/features/authentication/password-storage.html) · [CSRF: khi nào cần](https://docs.spring.io/spring-security/reference/servlet/exploits/csrf.html)
</details>

**SP35. ⭐⭐ Phân quyền "chỉ quản lý cửa hàng mới được xác nhận huỷ" làm thế nào?**
<details><summary>Gợi ý</summary>

Có hai lớp, **cần cả hai**:
```java
// Lớp 1: theo vai trò (người này có được làm việc này không?)
@Configuration
@EnableMethodSecurity            // Security 5.5 trở về trước: @EnableGlobalMethodSecurity(prePostEnabled = true)
public class MethodSecurityConfig {}

@PreAuthorize("hasRole('STORE_MANAGER')")
public void confirm(long orderId, LoginUser user) { ... }

// Lớp 2: theo dữ liệu (đơn này có thuộc cửa hàng của người này không?)
// Quản lý cửa hàng A có role STORE_MANAGER, nhưng không được xác nhận đơn của cửa hàng B.
```
```xml
<update id="confirmCancel">
  UPDATE orders
  SET status = 'CANCEL_CONFIRMED', confirmed_by = #{userId}, confirmed_at = SYSDATE
  WHERE order_id = #{orderId}
    AND status = 'CANCEL_REQUESTED'
    AND store_code = #{storeCode}       <!-- lấy từ thông tin đăng nhập, KHÔNG lấy từ request -->
</update>
```
Thiếu lớp 2 là lỗi bảo mật kiểu IDOR: chỉ cần đổi `orderId` trên URL là thao tác được đơn của cửa hàng khác.

📖 [Method security](https://docs.spring.io/spring-security/reference/servlet/authorization/method-security.html)
</details>

---

## 11. Hiệu năng và vận hành

**SP36. ⭐⭐ Connection pool (HikariCP): vì sao cần, cấu hình gì, lỗi nào hay gặp?**
<details><summary>Gợi ý</summary>

```yaml
spring:
  datasource:
    hikari:
      maximum-pool-size: 10            # mặc định 10
      minimum-idle: 10
      connection-timeout: 3000         # chờ lấy connection tối đa 3s (mặc định 30s)
      max-lifetime: 1800000            # 30 phút, nên ngắn hơn timeout của DB/firewall
      leak-detection-threshold: 20000  # connection bị giữ quá 20s thì log ra stack trace chỗ giữ
```
Lỗi hay gặp khi pool cạn:
```
HikariPool-1 - Connection is not available, request timed out after 30000ms.
```
Nguyên nhân thường gặp: transaction dài (SP30), query chậm, gọi API ngoài trong transaction, dùng `REQUIRES_NEW` lồng nhau (SP27), hoặc code tự lấy connection mà không trả lại. Tăng `maximum-pool-size` thường **không** giải quyết được, có khi còn làm DB chậm hơn.

📖 [Spring Boot: DataSource và pool](https://docs.spring.io/spring-boot/reference/data/sql.html) · [HikariCP: About Pool Sizing](https://github.com/brettwooldridge/HikariCP/wiki/About-Pool-Sizing)
</details>

**SP37. ⭐⭐ `@Async` dùng thế nào? Có những bẫy gì?**
<details><summary>Gợi ý</summary>

```java
@Configuration
@EnableAsync
public class AsyncConfig {
    @Bean(name = "notificationExecutor")
    public Executor notificationExecutor() {
        ThreadPoolTaskExecutor ex = new ThreadPoolTaskExecutor();
        ex.setCorePoolSize(4);
        ex.setMaxPoolSize(8);
        ex.setQueueCapacity(500);                       // có giới hạn (xem J22)
        ex.setThreadNamePrefix("notify-");              // nhìn log biết ngay thread nào
        ex.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        ex.initialize();
        return ex;
    }
}

@Service
public class NotificationService {
    @Async("notificationExecutor")
    public void sendCancelPush(long orderId) { ... }    // gọi xong trả về ngay, chạy ở thread khác

    @Async("notificationExecutor")
    public CompletableFuture<Boolean> sendSms(String phone) { ... }   // cần kết quả thì trả CompletableFuture
}
```
Bẫy: gọi trong cùng class thì không async (SP12); method `void` ném exception thì exception **mất** (không ai bắt, cần `AsyncUncaughtExceptionHandler`); thread mới không có transaction, không có `SecurityContext`, không có MDC của request gốc.

📖 [Spring Boot: task execution](https://docs.spring.io/spring-boot/reference/features/task-execution-and-scheduling.html) · [`@Async`](https://docs.spring.io/spring-framework/reference/integration/scheduling.html)
</details>

**SP38. ⭐⭐ `@Scheduled` cho batch (đối soát đơn huỷ cuối ngày chẳng hạn) có bẫy gì?**
<details><summary>Gợi ý</summary>

```java
@Configuration
@EnableScheduling
public class SchedulingConfig {}

@Component
public class CancelReconcileJob {
    // 2 giờ sáng giờ Hàn Quốc. Cron của Spring có 6 trường: giây phút giờ ngày tháng thứ
    @Scheduled(cron = "0 0 2 * * *", zone = "Asia/Seoul")
    public void reconcile() { ... }

    @Scheduled(fixedDelay = 60_000)   // chạy xong rồi nghỉ 60s mới chạy lại
    public void retryFailedRefunds() { ... }
    // fixedRate = 60_000: cứ 60s chạy một lần, tính từ lúc BẮT ĐẦU lần trước
}
```
```yaml
spring:
  task:
    scheduling:
      pool:
        size: 4       # mặc định 1: một job chạy lâu làm mọi job khác bị trễ
```
Chạy 2 server thì job chạy **trên cả 2**, đối soát hai lần. Dùng [ShedLock](https://github.com/lukas-krecan/ShedLock) để chỉ một server chạy:
```java
@Scheduled(cron = "0 0 2 * * *", zone = "Asia/Seoul")
@SchedulerLock(name = "cancelReconcile", lockAtMostFor = "PT30M")   // khoá nằm trong một bảng của DB
public void reconcile() { ... }
```

📖 [Scheduling](https://docs.spring.io/spring-framework/reference/integration/scheduling.html) · [Spring Boot: scheduling](https://docs.spring.io/spring-boot/reference/features/task-execution-and-scheduling.html)
</details>

**SP39. ⭐⭐ Cache với `@Cacheable`: dùng khi nào, cẩn thận gì?**
<details><summary>Gợi ý</summary>

```java
@Configuration
@EnableCaching
public class CacheConfig {}

@Service
public class StoreService {
    @Cacheable(cacheNames = "store", key = "#code")        // lần đầu query DB, lần sau lấy trong cache
    public Store findByCode(String code) { return storeMapper.findByCode(code); }

    @CacheEvict(cacheNames = "store", key = "#store.code") // sửa dữ liệu thì xoá cache của key đó
    public void update(Store store) { storeMapper.update(store); }
}
```
```yaml
spring:
  cache:
    type: caffeine
    caffeine:
      spec: maximumSize=1000,expireAfterWrite=10m    # giới hạn số lượng + tự hết hạn
```
Hợp với dữ liệu đọc nhiều, ít đổi (cửa hàng, danh mục, mã code chung). **Không** cache trạng thái đơn hàng. Cache trong bộ nhớ (Caffeine) thì mỗi server một bản: server A đã xoá, server B vẫn giữ dữ liệu cũ tới khi hết hạn. Cần đồng bộ thì dùng Redis.

📖 [Cache annotations](https://docs.spring.io/spring-framework/reference/integration/cache/annotations.html) · [Spring Boot: caching](https://docs.spring.io/spring-boot/reference/io/caching.html)
</details>

**SP40. ⭐⭐⭐ API đang chạy bỗng chậm trên production. Bro điều tra theo thứ tự nào?**
<details><summary>Gợi ý</summary>

1. **Phạm vi:** mọi API hay chỉ một API, từ lúc nào, có trùng thời điểm deploy hay giờ cao điểm không.
2. **Log và metric:** thời gian của từng request (interceptor ở SP19), số connection đang dùng (`/actuator/metrics/hikaricp.connections.active`), CPU, heap, GC.
3. **DB:** câu SQL nào chậm (Oracle: AWR, `V$SQL`), có bị khoá chờ không, plan có bị đổi không.
4. **Thread dump**, chụp 2–3 lần cách nhau vài giây:
   ```bash
   jcmd <pid> Thread.print > dump1.txt     # hoặc: jstack <pid>
   ```
   Nhiều thread cùng đứng ở `HikariPool.getConnection` là pool cạn; đứng ở `SocketInputStream.read` là đang chờ DB hoặc API ngoài; ở trạng thái `BLOCKED` là đang chờ khoá.
5. Sửa xong thì **đo lại bằng số**.

Kể một lần thật ở dự án là tốt nhất (chuyện N+1 ở mục 3 của [du-an-cong-ty.md](du-an-cong-ty.md)).
</details>

---

## 12. Test

**SP41. ⭐ Unit test khác integration test thế nào? Bro viết test cho service thế nào?**
<details><summary>Gợi ý</summary>

Unit test: một class, phụ thuộc là mock, không có Spring, chạy rất nhanh.
```java
@ExtendWith(MockitoExtension.class)
class CancelServiceTest {
    @Mock OrderMapper orderMapper;
    @Mock StockMapper stockMapper;
    @InjectMocks CancelService cancelService;     // tạo CancelService, truyền 2 mock vào

    @Test
    void confirm_restoresStock_whenStatusIsCancelRequested() {
        when(orderMapper.confirmCancel(1L, "u1")).thenReturn(1);    // update được 1 dòng

        cancelService.confirm(1L, "u1");

        verify(stockMapper).restore(1L);           // đã trả tồn kho
    }

    @Test
    void confirm_throws_andDoesNotRestoreStock_whenAlreadyConfirmed() {
        when(orderMapper.confirmCancel(1L, "u1")).thenReturn(0);    // 0 dòng: người khác đã xử lý

        assertThatThrownBy(() -> cancelService.confirm(1L, "u1"))
                .isInstanceOf(BusinessException.class);
        verify(stockMapper, never()).restore(anyLong());           // không trả tồn kho lần nữa
    }
}
```
Integration test: chạy cùng Spring và DB thật, kiểm cả SQL, transaction, cấu hình.

📖 [Spring Boot: testing](https://docs.spring.io/spring-boot/reference/testing/spring-boot-applications.html)
</details>

**SP42. ⭐⭐ `@SpringBootTest`, `@WebMvcTest`, `@MybatisTest` khác nhau thế nào?**
<details><summary>Gợi ý</summary>

```java
// Chỉ tầng web: controller, advice, filter. Service là mock.
@WebMvcTest(CancelController.class)
class CancelControllerTest {
    @Autowired MockMvc mvc;
    @MockBean CancelService cancelService;        // Boot 3.4 trở lên: @MockitoBean

    @Test
    void returns409_whenStatusInvalid() throws Exception {
        doThrow(new BusinessException(ErrorCode.INVALID_ORDER_STATUS))
                .when(cancelService).confirm(eq(1L), any());

        mvc.perform(post("/api/orders/1/cancel-confirmation")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content("{\"reason\":\"khách đổi ý\"}"))
           .andExpect(status().isConflict())
           .andExpect(jsonPath("$.code").value("INVALID_ORDER_STATUS"));
    }
}

// Chỉ mapper + DataSource, để kiểm câu SQL (thư viện mybatis-spring-boot-starter-test)
@MybatisTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)   // dùng DB thật, không thay bằng DB nhúng
class OrderMapperTest {
    @Autowired OrderMapper orderMapper;

    @Test
    void confirmCancel_updatesOnlyWhenCancelRequested() { ... }
}
```
`@SpringBootTest` dựng cả app: chậm nhất, nhưng gần với thật nhất. SQL viết riêng cho Oracle (`ROWNUM`, `MERGE`, `NVL`) thì nên test trên Oracle thật (Testcontainers với image Oracle XE), H2 không chạy giống hệt được.

📖 [Spring Boot: test slices](https://docs.spring.io/spring-boot/reference/testing/spring-boot-applications.html) · [`@MybatisTest`](https://mybatis.org/spring-boot-starter/mybatis-spring-boot-test-autoconfigure/)
</details>

**SP43. ⭐⭐ Test có `@Transactional` thì có gì đặc biệt?**
<details><summary>Gợi ý</summary>

```java
@SpringBootTest
@Transactional                     // mỗi test chạy xong Spring tự ROLLBACK
class CancelServiceIT {
    @Test
    void confirm() {
        cancelService.confirm(1L, "u1");
        // kiểm DB ở đây thấy dữ liệu mới (cùng transaction)
    }   // rollback: DB sạch lại cho test sau
}
```
Bẫy: vì không bao giờ commit, những thứ chỉ chạy khi commit (`@TransactionalEventListener(AFTER_COMMIT)` ở SP10) **không chạy trong test**; code chạy ở thread khác (`@Async`) không thấy dữ liệu chưa commit. Muốn test những thứ đó thì bỏ `@Transactional` ở test và tự dọn dữ liệu.

📖 [Transaction trong test](https://docs.spring.io/spring-framework/reference/testing/testcontext-framework/tx.html)
</details>

---

## 13. Design pattern trong Spring

**SP44. ⭐⭐ Spring dùng những design pattern nào? Bro đã dùng pattern nào?**
<details><summary>Gợi ý</summary>

| Pattern | Spring dùng ở đâu | Ví dụ trong file này |
|---|---|---|
| Singleton | Bean mặc định | SP5 |
| Factory | `BeanFactory`, `FactoryBean` (`SqlSessionFactoryBean` của MyBatis) | |
| Proxy | `@Transactional`, `@Async`, AOP | SP12, SP13 |
| Template Method | `JdbcTemplate`, `RestTemplate`, `TransactionTemplate`, `SqlSessionTemplate` | SP12 |
| Strategy | Inject `List<Interface>` rồi chọn theo key | `RefundHandler` ở SP4 |
| Observer | `ApplicationEvent` | SP10 |
| Front Controller | `DispatcherServlet` | SP19 |
| Chain of Responsibility | Filter chain, Spring Security | SP32 |

Khi được hỏi, chọn **một** pattern bro từng dùng thật và kể vì sao dùng nó, hơn là kể tên cả danh sách.
</details>

**SP45. ⭐⭐⭐ Spring MVC khác Spring WebFlux thế nào? Khi nào nên chuyển?**
<details><summary>Gợi ý</summary>

MVC: mỗi request giữ một thread cho tới khi xong (blocking), code đọc từ trên xuống dễ hiểu, hợp với JDBC/MyBatis. WebFlux: non-blocking, ít thread mà phục vụ được rất nhiều kết nối, nhưng cả chuỗi phải non-blocking. JDBC và MyBatis là blocking, nên dùng WebFlux với chúng thì mất hết lợi ích mà code lại khó đọc, khó debug hơn. Với Java 21, virtual threads giúp MVC chịu được nhiều kết nối mà không phải viết lại code. Dự án dùng MyBatis + Oracle thì MVC là lựa chọn đúng.

📖 [Spring WebFlux overview](https://docs.spring.io/spring-framework/reference/web/webflux/new-framework.html)
</details>

---

## Tự kiểm tra nhanh (5 phút, trả lời trong 1 câu)

1. Gọi method `@Transactional` từ method khác trong cùng class thì sao? (`SP12`)
2. Checked exception có làm transaction rollback không? (`Q4`)
3. `#{}` hay `${}`, cái nào chống SQL injection? (`Q13`)
4. `HashMap` có an toàn khi nhiều thread cùng ghi không? (`J13`)
5. Vì sao nên inject qua constructor? (`SP2`)
6. `volatile` có làm `count++` nguyên tử được không? (`J23`)
7. Bean mặc định có scope gì, và vì sao điều đó quan trọng? (`SP5`)
8. Hai server cùng chạy một job `@Scheduled` thì sao? (`SP38`)
9. Pool connection cạn thì người dùng thấy gì? (`SP36`)
10. So hai `Long` bằng `==` thì sai ở đâu? (`J4`)

Ghi các câu trả lời chưa trôi vào bảng ⚠️ ở cuối [phong-van-java-sql.md](phong-van-java-sql.md).
