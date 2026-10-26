# Câu hỏi phỏng vấn Java + Spring

Bộ câu hỏi chung, gắn vào stack đang làm: **Spring Boot 2.x, MyBatis, Oracle, API cho app Android**.
Mã câu: `J` là Java, `SP` là Spring. Câu nào đã có trong file khác thì chỉ ghi mã để khỏi lặp: `A1`, `B3` ở [phong-van-java-sql.md](phong-van-java-sql.md), `Q4` ở [du-an-cong-ty.md](du-an-cong-ty.md).

**Mức độ:** ⭐ gần như chắc chắn bị hỏi · ⭐⭐ hay gặp ở vòng kỹ thuật · ⭐⭐⭐ hỏi khi muốn đào sâu (mid/senior)

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

## 5. Spring Core: IoC, DI, Bean

**SP1. ⭐ IoC và DI là gì? Lợi ích thật là gì?**
<details><summary>Gợi ý</summary>

IoC: container (`ApplicationContext`) tạo và nối các object thay vì code tự `new`. DI là một cách làm IoC: phụ thuộc được truyền vào. Lợi ích: thay cài đặt mà không sửa code dùng nó, test dễ (truyền mock vào), cấu hình nằm một chỗ, và Spring bọc thêm được proxy (transaction, AOP).
</details>

**SP2. ⭐ Inject qua constructor, setter hay field? Vì sao nên dùng constructor?**
<details><summary>Gợi ý</summary>

Constructor: field để `final` được, thiếu phụ thuộc thì báo lỗi ngay lúc khởi động, test chỉ cần `new Service(mock)` mà không cần Spring, và class có quá nhiều tham số tự lộ ra là đang làm quá nhiều việc. Field injection (`@Autowired` lên field) gọn nhưng giấu phụ thuộc và khó test. Lombok `@RequiredArgsConstructor` sinh constructor cho mọi field `final`.
</details>

**SP3. ⭐ `@Component`, `@Service`, `@Repository`, `@Controller` khác nhau thế nào? `@Bean` thì sao?**
<details><summary>Gợi ý</summary>

Cả bốn đều là `@Component` (để component scan tìm thấy), khác nhau ở ý nghĩa; `@Repository` còn được Spring đổi exception của DB sang `DataAccessException`. `@Bean` đặt trên method trong `@Configuration`: dùng khi tạo object của thư viện bên ngoài (không gắn annotation vào class đó được) hoặc khi việc tạo cần logic (`AiConfig` trong korean-practice).
</details>

**SP4. ⭐ Có 2 bean cùng kiểu thì Spring inject cái nào?**
<details><summary>Gợi ý</summary>

Không làm gì thì lỗi `NoUniqueBeanDefinitionException`. Cách giải: `@Primary` (bean mặc định), `@Qualifier("name")` tại chỗ inject, hoặc inject `List<Interface>`/`Map<String, Interface>` để lấy hết (Strategy pattern: mỗi phương thức thanh toán một bean, chọn theo key). Ví dụ: hai `DataSource` (Oracle chính và DB khác) → hai `SqlSessionFactory`.
</details>

**SP5. ⭐ Bean scope có những loại nào? Mặc định là gì?**
<details><summary>Gợi ý</summary>

`singleton` (mặc định, một object cho cả container), `prototype` (mỗi lần lấy một object mới), và với web thêm `request`, `session`, `application`. Singleton dùng chung cho mọi request, nên **không được giữ state của request trong field** của service/controller.
</details>

**SP6. ⭐⭐ Inject một bean `prototype` vào bean `singleton` thì sao?**
<details><summary>Gợi ý</summary>

Chỉ inject một lần lúc tạo singleton, nên prototype trở thành "một object duy nhất" luôn. Muốn mỗi lần dùng một object mới thì inject `ObjectProvider<T>` rồi gọi `getObject()`, hoặc dùng `@Lookup`, hoặc scoped proxy.
</details>

**SP7. ⭐⭐ Vòng đời của một bean.**
<details><summary>Gợi ý</summary>

Tạo object → inject phụ thuộc → các callback `*Aware` → `BeanPostProcessor` (trước init) → `@PostConstruct` / `afterPropertiesSet` / `init-method` → `BeanPostProcessor` (sau init: **chỗ Spring tạo proxy** cho `@Transactional`, `@Async`) → dùng → `@PreDestroy` khi tắt app. Vì proxy được tạo sau `@PostConstruct`, gọi method `@Transactional` của chính nó trong `@PostConstruct` sẽ không có transaction.
</details>

**SP8. ⭐⭐ Circular dependency là gì? Spring Boot xử lý thế nào?**
<details><summary>Gợi ý</summary>

A cần B, B lại cần A. Từ Spring Boot 2.6, mặc định **cấm** và báo lỗi lúc khởi động (`spring.main.allow-circular-references=false`). Với constructor injection thì không bao giờ giải được. Sửa đúng là tách phần dùng chung ra một bean thứ ba, hoặc dùng event; `@Lazy` chỉ là cách chữa cháy.
</details>

**SP9. ⭐⭐⭐ `@Configuration` khác `@Component` khi chứa method `@Bean` ở chỗ nào?**
<details><summary>Gợi ý</summary>

Spring bọc class `@Configuration` bằng CGLIB, nên một method `@Bean` gọi một method `@Bean` khác vẫn nhận lại **đúng singleton** trong container. Trong `@Component` (gọi là "lite mode", hoặc `@Configuration(proxyBeanMethods = false)`) thì đó là lời gọi Java bình thường, mỗi lần gọi tạo một object mới.
</details>

**SP10. ⭐⭐ `ApplicationEvent` và `@TransactionalEventListener` dùng làm gì?** *(gắn với xác nhận huỷ)*
<details><summary>Gợi ý</summary>

Bắn event để tách các việc phụ ra khỏi service chính: `publisher.publishEvent(new CancelConfirmedEvent(orderId))`. Dùng `@TransactionalEventListener(phase = AFTER_COMMIT)` để việc gửi thông báo (push tới app Android, gửi SMS) chỉ chạy **sau khi DB đã commit**: transaction rollback thì không gửi nhầm "đã huỷ thành công". Muốn không chặn request thì thêm `@Async`.
</details>

---

## 6. Spring AOP và proxy

**SP11. ⭐ AOP là gì? Spring dùng nó ở đâu?**
<details><summary>Gợi ý</summary>

Tách các việc cắt ngang nhiều chỗ (transaction, log, đo thời gian, kiểm quyền, cache) ra khỏi code nghiệp vụ. Spring tự dùng cho `@Transactional`, `@Async`, `@Cacheable`, `@PreAuthorize`. Tự viết: `@Aspect` + `@Around("execution(* com.x.service..*(..))")` để log thời gian chạy của mọi service.
</details>

**SP12. ⭐ Vì sao gọi method `@Transactional` (hoặc `@Async`, `@Cacheable`) từ method khác trong cùng class thì không có tác dụng?**
<details><summary>Gợi ý</summary>

Annotation có tác dụng nhờ **proxy** bọc ngoài bean. Gọi `this.method()` là gọi thẳng vào object thật, không đi qua proxy. Sửa: tách method ra một bean khác (cách sạch nhất), hoặc tự inject chính mình (`@Lazy` self), hoặc dùng `TransactionTemplate`. Đây là câu bị hỏi **rất nhiều**.
</details>

**SP13. ⭐⭐ JDK dynamic proxy khác CGLIB thế nào?**
<details><summary>Gợi ý</summary>

JDK proxy: chỉ proxy được qua interface. CGLIB: tạo lớp con của class, nên không proxy được method `final`/`private` và class `final`. Spring Boot 2 trở lên mặc định dùng CGLIB (`proxyTargetClass = true`). Hệ quả: `@Transactional` trên method `private` hay `final` bị bỏ qua, không báo lỗi gì.
</details>

---

## 7. Spring Boot

**SP14. ⭐ Spring Boot khác Spring Framework chỗ nào? `@SpringBootApplication` gồm gì?**
<details><summary>Gợi ý</summary>

Boot = Spring + **auto-configuration** + starter (gói phụ thuộc đi kèm nhau) + server nhúng (Tomcat) + cấu hình bên ngoài + Actuator. `@SpringBootApplication` = `@Configuration` + `@EnableAutoConfiguration` + `@ComponentScan` (quét từ package của class đó trở xuống: đặt class main sai package là không thấy bean).
</details>

**SP15. ⭐⭐ Auto-configuration hoạt động thế nào?**
<details><summary>Gợi ý</summary>

Boot nạp danh sách các class cấu hình tự động (Boot 2.7 trở lên: file `META-INF/spring/...AutoConfiguration.imports`; trước đó là `spring.factories`). Mỗi class có điều kiện: `@ConditionalOnClass` (có thư viện trong classpath không), `@ConditionalOnMissingBean` (người dùng đã tự khai báo chưa), `@ConditionalOnProperty`. Ví dụ: thêm `mybatis-spring-boot-starter` và có `DataSource` thì `SqlSessionFactory` tự được tạo. Xem bean nào được tạo và vì sao: chạy với `--debug` (in ra conditions report).
</details>

**SP16. ⭐ Cấu hình theo môi trường (dev, stg, prod) làm thế nào? Thứ tự ưu tiên ra sao?**
<details><summary>Gợi ý</summary>

`application-{profile}.yml` + `spring.profiles.active=prod`. Ưu tiên từ cao xuống: tham số dòng lệnh → biến môi trường → file profile → `application.yml`. Mật khẩu không để trong file commit lên git: dùng biến môi trường, hoặc Vault, hoặc mã hoá (Jasypt hay gặp ở dự án Hàn). Gom cấu hình bằng `@ConfigurationProperties` (xem `B5`).
</details>

**SP17. ⭐⭐ Actuator dùng làm gì? Có rủi ro gì?**
<details><summary>Gợi ý</summary>

`/actuator/health` (load balancer kiểm tra app còn sống không, có cả health của DB), `/metrics`, `/info`, `/loggers` (đổi log level lúc đang chạy). Rủi ro: mở `/env`, `/heapdump` ra ngoài là lộ mật khẩu và dữ liệu. Chỉ mở những endpoint cần, để sau xác thực hoặc mạng nội bộ.
</details>

**SP18. ⭐⭐ Log trong Spring Boot: SLF4J, Logback, MDC.**
<details><summary>Gợi ý</summary>

Code gọi SLF4J (interface), Logback là cài đặt mặc định. Viết `log.info("order {}", id)` (có placeholder) chứ không cộng chuỗi. MDC: gắn `requestId` (lấy từ header app Android gửi lên, hoặc tự sinh) vào mọi dòng log của một request bằng một filter, nhờ đó tra được toàn bộ log của một lần xác nhận huỷ. Không log mật khẩu, số thẻ, số điện thoại của khách.
</details>

---

## 8. Spring MVC và REST API

**SP19. ⭐ Một request đi qua Spring MVC thế nào (`DispatcherServlet`)?**
<details><summary>Gợi ý</summary>

Filter → `DispatcherServlet` → `HandlerMapping` (tìm controller method theo URL) → interceptor `preHandle` → `HandlerAdapter` gọi method (bind tham số, chạy validate) → kết quả trả về được `HttpMessageConverter` (Jackson) đổi thành JSON → `postHandle` → `afterCompletion`. Có exception thì `@ExceptionHandler`/`@ControllerAdvice` xử lý. Filter khác interceptor: xem `B3`.
</details>

**SP20. ⭐ `@Controller` khác `@RestController`? `@RequestParam`, `@PathVariable`, `@RequestBody` khác nhau thế nào?**
<details><summary>Gợi ý</summary>

`@RestController` = `@Controller` + `@ResponseBody` (trả dữ liệu, không trả view). `@PathVariable`: một phần của URL (`/orders/{id}`). `@RequestParam`: query string (`?status=CANCEL`). `@RequestBody`: body JSON đổi thành object. `ResponseEntity` để tự chọn status code và header.
</details>

**SP21. ⭐ Chọn HTTP method và status code thế nào? API xác nhận huỷ nên trả gì?**
<details><summary>Gợi ý</summary>

`GET` (chỉ đọc), `POST` (tạo mới hoặc hành động), `PUT` (thay toàn bộ, idempotent), `PATCH` (sửa một phần), `DELETE`. Status: 200, 201 (đã tạo), 204, 400 (dữ liệu sai), 401 (chưa đăng nhập), 403 (không có quyền), 404, **409 (xung đột trạng thái: đơn đã được xác nhận hoặc đã bị huỷ)**, 500. Xác nhận huỷ: `POST /orders/{id}/cancel-confirmation`. Thành công trả 200; đơn đã xác nhận rồi thì trả 200 kèm trạng thái hiện tại (idempotent, `Q3`) hoặc 409, miễn là thống nhất với app Android.
</details>

**SP22. ⭐ Xử lý lỗi tập trung thế nào? Trả lỗi cho app Android ra sao?**
<details><summary>Gợi ý</summary>

`@RestControllerAdvice` + `@ExceptionHandler` cho từng loại exception (business exception có mã lỗi riêng, lỗi validate, lỗi không lường trước). Body thống nhất: `{"code": "ORDER_ALREADY_CANCELLED", "message": "..."}`. App hiển thị theo `code`, không theo `message`. Lỗi 500 không được trả stack trace ra ngoài. Ví dụ có sẵn: `ApiExceptionHandler` (`B7`).
</details>

**SP23. ⭐ Validate input: `@Valid` khác `@Validated` thế nào?**
<details><summary>Gợi ý</summary>

`@Valid` (chuẩn Jakarta/Javax): kích hoạt validate cho `@RequestBody` và đi vào các object lồng bên trong. `@Validated` (của Spring): hỗ trợ **validation group** (luật cho tạo mới khác luật cho cập nhật) và validate tham số method khi đặt trên class (`@Min` trên `@RequestParam`). Annotation hay dùng: `@NotNull`, `@NotBlank`, `@Size`, `@Pattern`, `@Positive`. Xem `B8`.
</details>

**SP24. ⭐⭐ Thay đổi API mà không làm hỏng app Android bản cũ thế nào?**
<details><summary>Gợi ý</summary>

Chỉ **thêm** field, không đổi tên hay xoá field. Spring Boot cấu hình Jackson mặc định bỏ qua field lạ (`FAIL_ON_UNKNOWN_PROPERTIES = false`), nên phía server nhận thêm field không bị lỗi; phía app cũng nên bỏ qua field lạ. Cần đổi lớn thì làm version mới (`/v2/...`) và giữ bản cũ tới khi phần lớn người dùng đã cập nhật app. Xem `Q23`.
</details>

**SP25. ⭐⭐ Phân trang API danh sách đơn thế nào?**
<details><summary>Gợi ý</summary>

Nhận `page`, `size` (giới hạn `size` tối đa, ví dụ 100), trả `items` + `totalCount` (hoặc `hasNext`). Phía DB: `OFFSET/FETCH` hoặc `ROWNUM` (xem `Q19`). App cuộn vô hạn hoặc dữ liệu lớn thì dùng cursor/keyset (`?lastId=...`), vì `COUNT(*)` trên bảng lớn cũng tốn.
</details>

**SP26. ⭐⭐ Upload file (ảnh chứng từ huỷ chẳng hạn) cần chú ý gì?**
<details><summary>Gợi ý</summary>

`MultipartFile`, giới hạn dung lượng (`spring.servlet.multipart.max-file-size`, `max-request-size`), kiểm tra loại file theo nội dung chứ không chỉ theo đuôi, đặt lại tên file (không dùng tên người dùng gửi để tránh path traversal `../`), lưu ra storage riêng (S3, NAS) chứ không lưu trong DB hay trong thư mục của app.
</details>

---

## 9. Transaction (Spring)

Phần cơ bản đã có ở `Q4` (MyBatis), `C2`, `C4`, `C6`. Thêm các câu hay bị đào sâu:

**SP27. ⭐ Các loại propagation, và ví dụ dùng từng loại.**
<details><summary>Gợi ý</summary>

- `REQUIRED` (mặc định): có transaction thì dùng chung, không có thì tạo mới.
- `REQUIRES_NEW`: luôn tạo transaction mới, tạm dừng cái đang có. Ví dụ: ghi log lỗi hoặc lịch sử thao tác phải được lưu dù transaction chính rollback.
- `NESTED`: savepoint bên trong transaction đang có; phần con rollback được mà phần cha không bị ảnh hưởng. Chạy được với `DataSourceTransactionManager` (MyBatis/JDBC), JPA thì không.
- `SUPPORTS`, `MANDATORY`, `NOT_SUPPORTED`, `NEVER`: ít dùng, biết tên là đủ.
</details>

**SP28. ⭐⭐ Lỗi `UnexpectedRollbackException: Transaction silently rolled back because it has been marked as rollback-only` là gì?**
<details><summary>Gợi ý</summary>

Method ngoài (`REQUIRED`) gọi method trong (`REQUIRED`, dùng chung transaction). Method trong ném `RuntimeException`, nên transaction bị **đánh dấu rollback-only**. Method ngoài `catch` lỗi rồi chạy tiếp, tưởng sẽ commit được, nhưng tới lúc commit thì Spring ném lỗi này. Sửa: đừng nuốt lỗi; nếu phần trong được phép thất bại riêng thì cho nó `REQUIRES_NEW` hoặc `NESTED`.
</details>

**SP29. ⭐⭐ Isolation level: Spring khai báo thế nào? Oracle hỗ trợ những mức nào?**
<details><summary>Gợi ý</summary>

`@Transactional(isolation = Isolation.READ_COMMITTED)`. Các hiện tượng: dirty read, non-repeatable read, phantom read. Oracle chỉ có `READ COMMITTED` (mặc định) và `SERIALIZABLE` (cùng chế độ `READ ONLY`); không có dirty read. Thực tế hầu như để mặc định, xử lý tranh chấp bằng update có điều kiện hoặc `FOR UPDATE` (`Q2`).
</details>

**SP30. ⭐⭐ Transaction dài gây ra vấn đề gì? `timeout` dùng thế nào?**
<details><summary>Gợi ý</summary>

Giữ khoá dòng lâu (người khác phải chờ hoặc bị deadlock), giữ connection (pool cạn thì toàn bộ API treo), undo của Oracle lớn dần. Không gọi API ngoài, không gửi mail, không xử lý file trong transaction (`Q5`). `@Transactional(timeout = 5)` để chặn trên. Batch lớn thì chia commit theo lô.
</details>

**SP31. ⭐⭐⭐ `@Transactional` đặt ở controller, service hay mapper?**
<details><summary>Gợi ý</summary>

Ở **service**: đó là ranh giới của một nghiệp vụ (xác nhận huỷ = đổi trạng thái + hoàn tồn kho + ghi lịch sử). Ở controller thì transaction ôm cả việc chuyển JSON và các việc không liên quan; ở mapper thì mỗi câu SQL là một transaction riêng, mất tính nguyên tử. Đặt trên class thì áp cho mọi method `public`; đặt trên method thì ghi đè cấu hình của class.
</details>

---

## 10. Spring Security

**SP32. ⭐ Authentication khác authorization thế nào? Spring Security chạy ở đâu trong request?**
<details><summary>Gợi ý</summary>

Authentication: bạn là ai (đăng nhập). Authorization: bạn được làm gì (vai trò, quyền). Spring Security là một **chuỗi filter** (`FilterChainProxy`, nằm trong servlet filter chain qua `DelegatingFilterProxy`), chạy trước `DispatcherServlet`. Kết quả xác thực nằm trong `SecurityContextHolder` (dùng `ThreadLocal`, xem `J26`).
</details>

**SP33. ⭐⭐ API cho app di động: session hay JWT? Ưu nhược điểm?**
<details><summary>Gợi ý</summary>

Session: state nằm ở server, thu hồi dễ, nhưng nhiều server thì phải chia sẻ session (Redis). JWT: stateless, server nào cũng tự kiểm được bằng chữ ký, nhưng **khó thu hồi** trước khi hết hạn, nên access token để ngắn hạn + có refresh token. Trong JWT không bỏ dữ liệu nhạy cảm (chỉ được mã hoá base64, ai cũng đọc được). Ở dự án dùng `___`.
</details>

**SP34. ⭐⭐ Lưu mật khẩu thế nào? Vì sao API cho app di động thường tắt CSRF?**
<details><summary>Gợi ý</summary>

Mật khẩu: hash bằng `BCryptPasswordEncoder` (có salt, cố tình chậm), không dùng MD5 hay SHA-256 trần, không bao giờ mã hoá hai chiều. CSRF là kiểu tấn công lợi dụng việc **browser tự gửi cookie**; API stateless nhận token qua header `Authorization` thì không bị, nên tắt được. Còn dùng cookie session thì phải bật.
</details>

**SP35. ⭐⭐ Phân quyền "chỉ quản lý cửa hàng mới được xác nhận huỷ" làm thế nào?**
<details><summary>Gợi ý</summary>

Theo URL trong `SecurityFilterChain` (`.antMatchers("/orders/*/cancel-confirmation").hasRole("STORE_MANAGER")`; Spring Security 5.8 trở lên đổi tên thành `requestMatchers`), hoặc theo method với `@PreAuthorize("hasRole('STORE_MANAGER')")` (cần `@EnableMethodSecurity`; bản cũ là `@EnableGlobalMethodSecurity`). Còn phải kiểm thêm **theo dữ liệu**: quản lý cửa hàng A không được xác nhận đơn của cửa hàng B. Việc này nằm ở service hoặc ngay trong câu SQL (`WHERE store_code = #{myStore}`).
</details>

---

## 11. Hiệu năng và vận hành

**SP36. ⭐⭐ Connection pool (HikariCP): vì sao cần, cấu hình gì, lỗi nào hay gặp?**
<details><summary>Gợi ý</summary>

Mở connection tới Oracle rất tốn, nên giữ sẵn một nhóm để dùng lại. HikariCP là mặc định từ Spring Boot 2, `maximum-pool-size` mặc định 10. Lỗi hay gặp: `Connection is not available, request timed out after 30000ms` khi pool cạn, do transaction dài, query chậm, hoặc connection bị rò (lấy ra mà không trả lại). Bật `leak-detection-threshold` để log chỗ code giữ connection quá lâu. Pool to hơn chưa chắc đã nhanh hơn.
</details>

**SP37. ⭐⭐ `@Async` dùng thế nào? Có những bẫy gì?**
<details><summary>Gợi ý</summary>

Cần `@EnableAsync`. Bẫy: gọi trong cùng class thì không async (`SP12`); không cấu hình executor thì dùng pool mặc định (Boot 2.1 trở lên tự tạo `ThreadPoolTaskExecutor` có hàng đợi vô hạn); method `void` ném exception thì exception bị mất (cần `AsyncUncaughtExceptionHandler`); không có transaction và không có `SecurityContext` của request gốc. Trả `CompletableFuture<T>` nếu cần lấy kết quả.
</details>

**SP38. ⭐⭐ `@Scheduled` cho batch (đối soát đơn huỷ cuối ngày chẳng hạn) có bẫy gì?**
<details><summary>Gợi ý</summary>

Mặc định tất cả job chạy trên **một thread duy nhất**: job này chậm thì job kia bị trễ (tăng `spring.task.scheduling.pool.size`). Chạy nhiều server thì **mỗi server đều chạy job** (chạy trùng): dùng ShedLock (khoá trong DB) hoặc chỉ bật job ở một server. Nhớ múi giờ: `@Scheduled(cron = "0 0 2 * * *", zone = "Asia/Seoul")`. `fixedRate` khác `fixedDelay`: tính từ lúc bắt đầu hay từ lúc chạy xong.
</details>

**SP39. ⭐⭐ Cache với `@Cacheable`: dùng khi nào, cẩn thận gì?**
<details><summary>Gợi ý</summary>

Hợp với dữ liệu đọc nhiều, ít đổi: mã cửa hàng, danh mục, bảng mã code chung. Cần `@EnableCaching` và một provider (Caffeine trong máy, Redis dùng chung giữa các server). Bẫy: gọi trong cùng class (`SP12`), quên xoá cache khi dữ liệu đổi (`@CacheEvict`), cache trong máy thì mỗi server một bản khác nhau, không có TTL thì dữ liệu cũ nằm mãi. Không cache dữ liệu trạng thái đơn hàng.
</details>

**SP40. ⭐⭐⭐ API đang chạy bỗng chậm trên production. Bro điều tra theo thứ tự nào?**
<details><summary>Gợi ý</summary>

1. Phạm vi: mọi API hay chỉ một API, từ lúc nào, có trùng thời điểm deploy hay giờ cao điểm không.
2. Log và metric: thời gian từng request, số connection đang dùng trong pool, CPU, heap, số lần GC.
3. DB: câu SQL nào chậm (Oracle: AWR, `V$SQL`), có bị khoá chờ không, có mất index hay plan bị đổi không.
4. Thread dump (`jstack`) chụp 2–3 lần: thread đang kẹt ở đâu (chờ connection, chờ API ngoài, chờ khoá).
5. Sửa xong thì đo lại bằng số.

Kể một lần thật ở dự án là tốt nhất (chuyện N+1 ở mục 3 của [du-an-cong-ty.md](du-an-cong-ty.md)).
</details>

---

## 12. Test

**SP41. ⭐ Unit test khác integration test thế nào? Bro viết test cho service thế nào?**
<details><summary>Gợi ý</summary>

Unit: một class, các phụ thuộc là mock (JUnit 5 + Mockito: `@ExtendWith(MockitoExtension.class)`, `@Mock`, `@InjectMocks`, `when(...).thenReturn(...)`, `verify(...)`), chạy rất nhanh. Integration: chạy cùng Spring và DB thật. Với service xác nhận huỷ: mock mapper, kiểm tra trường hợp sai trạng thái thì ném lỗi, trường hợp update trả 0 dòng thì ném lỗi, và mapper được gọi đúng số lần.
</details>

**SP42. ⭐⭐ `@SpringBootTest`, `@WebMvcTest`, `@MybatisTest` khác nhau thế nào?**
<details><summary>Gợi ý</summary>

`@SpringBootTest`: dựng cả app, chậm nhưng thật nhất. `@WebMvcTest`: chỉ tầng web (controller, advice, filter), service thì mock bằng `@MockBean` (Boot 3.4 trở lên là `@MockitoBean`), test bằng `MockMvc`. `@MybatisTest` (thư viện `mybatis-spring-boot-starter-test`): chỉ dựng mapper và DataSource, để test câu SQL. Test SQL viết cho Oracle thì nên chạy trên Oracle thật (Testcontainers với image Oracle XE), không chạy trên H2.
</details>

**SP43. ⭐⭐ Test có `@Transactional` thì có gì đặc biệt?**
<details><summary>Gợi ý</summary>

Spring tự **rollback** sau mỗi test, nên DB sạch lại. Bẫy: vì không commit, những thứ chỉ chạy khi commit (`@TransactionalEventListener(AFTER_COMMIT)`, trigger, constraint kiểu deferred) **không được test tới**; và code chạy ở thread khác (`@Async`) không thấy dữ liệu chưa commit.
</details>

---

## 13. Design pattern trong Spring

**SP44. ⭐⭐ Spring dùng những design pattern nào? Bro đã dùng pattern nào?**
<details><summary>Gợi ý</summary>

Singleton (bean), Factory (`BeanFactory`, `FactoryBean`), Proxy (`@Transactional`, AOP), Template Method (`JdbcTemplate`, `RestTemplate`, `SqlSessionTemplate`), Strategy (inject `List<PaymentHandler>`), Observer (`ApplicationEvent`), Front Controller (`DispatcherServlet`), Adapter (`HandlerAdapter`). Tự dùng: Strategy cho nhiều cách hoàn tiền theo phương thức thanh toán; Null Object (`NoAiProvider`, xem `B6`).
</details>

**SP45. ⭐⭐⭐ Spring MVC khác Spring WebFlux thế nào? Khi nào nên chuyển?**
<details><summary>Gợi ý</summary>

MVC: mỗi request giữ một thread (blocking), code dễ hiểu, hợp với JDBC/MyBatis. WebFlux: non-blocking, ít thread mà phục vụ được nhiều kết nối, nhưng cả chuỗi phải non-blocking (JDBC và MyBatis là blocking, nên dùng WebFlux với chúng thì mất hết lợi ích), và debug khó hơn. Với Java 21, virtual threads giúp MVC chịu được nhiều kết nối mà không phải viết lại code. Dự án dùng MyBatis + Oracle thì MVC là lựa chọn đúng.
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
