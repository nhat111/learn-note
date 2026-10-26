# Phỏng vấn: dự án ở công ty (bán lẻ, Shinsegae)

**Ưu tiên số 1.** Người phỏng vấn sẽ hỏi dự án này nhiều nhất. Side project ([phong-van-java-sql.md](phong-van-java-sql.md)) chỉ để làm ví dụ dự phòng.

| | |
|---|---|
| Nghiệp vụ | Bán lẻ, khách hàng Shinsegae |
| Backend | Java, Spring, MyBatis 2.2.2, Oracle |
| Client | App Android |
| Phần mình làm | Chức năng xác nhận huỷ |
| Chuyện kỹ thuật | Tối ưu N+1 query ở một chức năng cũ; app báo timeout nhưng server vẫn cập nhật (mục 9) |
| Team | 3 dev, waterfall |
| Cần điền thêm | Spring Boot `___`, Java `___`, Oracle `___`, thời gian tham gia `___` tháng |

> **NDA:** trước khi phỏng vấn, hỏi công ty xem được nói tên khách đến đâu. Không được thì nói "một tập đoàn bán lẻ lớn của Hàn Quốc". Không kể số liệu doanh thu hay dữ liệu khách hàng thật.

Cách dùng như file kia: tự trả lời thành tiếng trước, rồi mới mở gợi ý. Chỗ `___` là việc của bro: hỏi lại team, xem lại code hoặc ticket để điền **số thật**. Phỏng vấn mà có số thì khác hẳn.

---

## 1. Pitch 2 phút (điền vào rồi tập nói)

> Em đang làm backend cho hệ thống ___ (bán hàng / quản lý đơn / ...) của Shinsegae, phía client là app Android mà ___ (nhân viên cửa hàng / khách hàng) dùng để ___.
> Stack là Spring Boot ___ với MyBatis và Oracle. Team có 3 dev, làm theo waterfall.
> Phần em phụ trách chính là **chức năng xác nhận huỷ**: ___ (ai gửi yêu cầu huỷ, ai xác nhận, xác nhận xong hệ thống làm gì).
> Một việc em thấy đáng kể nhất là em **tối ưu N+1 query** ở chức năng ___ có sẵn: trước đó một lần gọi API chạy ___ câu query và mất ___ giây; em gộp lại thành ___ câu, còn ___ giây.
> Qua dự án này em học được ___ (ví dụ: đọc execution plan của Oracle, thiết kế transaction cho các thao tác đổi trạng thái).

---

## 2. Chức năng xác nhận huỷ

Trước khi tập, bro trả lời được cho mình mấy chuyện này:
- Ai gửi yêu cầu huỷ (khách, nhân viên)? Ai xác nhận? Xác nhận trên app Android hay trên web admin?
- Đơn hàng có những trạng thái nào? Huỷ được ở những trạng thái nào?
- Xác nhận xong thì hệ thống làm gì: đổi trạng thái, hoàn tiền, trả lại tồn kho, hoàn điểm/coupon, ghi lịch sử, gửi thông báo?
- Có huỷ một phần đơn (vài món trong đơn) không?

**Q1. Mô tả luồng xác nhận huỷ từ app Android tới DB.**
<details><summary>Gợi ý khung trả lời</summary>

Android gọi `POST /___` (gửi mã đơn, người xác nhận, lý do) → Controller kiểm tham số → Service kiểm quyền và trạng thái đơn → trong **một transaction**: cập nhật trạng thái đơn, ___ (tồn kho / điểm / thanh toán), ghi bảng lịch sử huỷ → commit → trả kết quả và mã lỗi cho app.
Vẽ được hình này trên giấy trong 1 phút.
</details>

**Q2. Hai nhân viên cùng bấm "xác nhận huỷ" một đơn cùng lúc thì sao?**
<details><summary>Gợi ý</summary>

Rủi ro: hoàn tiền hai lần, trả tồn kho hai lần. Cách chặn:
- **Update có điều kiện** (hay dùng nhất):
  ```sql
  UPDATE orders SET status = 'CANCEL_CONFIRMED', confirmed_by = #{userId}, confirmed_at = SYSDATE
  WHERE order_id = #{orderId} AND status = 'CANCEL_REQUESTED'
  ```
  Mapper trả số dòng bị ảnh hưởng. Bằng 0 nghĩa là người khác đã xử lý trước: ném lỗi, rollback, báo cho app.
- **Khoá dòng**: `SELECT ... FROM orders WHERE order_id = #{orderId} FOR UPDATE NOWAIT` (hoặc `WAIT 3`). Request thứ hai bị chặn hoặc nhận `ORA-00054` ngay.
- Dự án thật đang làm cách nào: `___`. Chưa có cách nào thì cũng nói thật, kèm cách mình sẽ sửa.

Liên quan: S12 (lost update) trong [phong-van-java-sql.md](phong-van-java-sql.md).
</details>

**Q3. Mạng ở cửa hàng chập chờn, app Android gửi lại request (retry, bấm hai lần). Xử lý thế nào?**
<details><summary>Gợi ý</summary>

API đổi trạng thái phải **idempotent**: gọi hai lần thì kết quả vẫn như một lần. Update có điều kiện ở Q2 đã chặn được việc huỷ hai lần. Lần gọi thứ hai nên trả "đã xác nhận rồi" (thành công) chứ không trả lỗi lạ. Phía app thì khoá nút sau lần bấm đầu. Nếu có gọi API thanh toán bên ngoài để hoàn tiền thì cần thêm idempotency key cho lời gọi đó.
</details>

**Q4. `@Transactional` với MyBatis hoạt động thế nào? Giữa chừng lỗi thì sao?**
<details><summary>Gợi ý</summary>

- `mybatis-spring` dùng `SqlSessionTemplate`, gắn `SqlSession` vào transaction của Spring (`DataSourceTransactionManager`). Mọi mapper được gọi trong method `@Transactional` đều dùng chung một connection.
- Mặc định chỉ rollback khi gặp `RuntimeException`. Ném checked exception thì vẫn commit, trừ khi khai báo `rollbackFor = Exception.class`.
- Gọi method `@Transactional` từ method khác **cùng class** thì không có transaction (self-invocation, vì Spring dùng proxy).
- Liên quan: C2, C4 trong [phong-van-java-sql.md](phong-van-java-sql.md).
</details>

**Q5. Có gọi hệ thống ngoài (thanh toán, POS, ERP) khi xác nhận huỷ không? Gọi trong transaction có vấn đề gì?**
<details><summary>Gợi ý</summary>

Gọi API ngoài **bên trong** transaction DB thì transaction bị giữ lâu (khoá dòng, chiếm connection). Tệ hơn: API ngoài báo hoàn tiền thành công mà DB lại rollback, hai bên lệch nhau. Các cách hay dùng:
- Commit trạng thái "đang hoàn tiền" trước, gọi API ngoài sau, rồi cập nhật kết quả.
- Ghi việc cần làm vào một bảng (outbox), để batch hoặc job xử lý.
- Có job đối soát (reconcile) để bắt các trường hợp lệch.

Dự án thật làm thế nào: `___`.
</details>

**Q6. Huỷ một phần đơn hàng thì khó ở chỗ nào?** *(chỉ ôn nếu dự án có)*
<details><summary>Gợi ý</summary>

Tính lại tiền: khuyến mãi theo đơn (mua 2 tặng 1, giảm giá khi đủ ngưỡng tiền) có thể không còn đủ điều kiện; coupon và điểm phải chia theo từng món; phí giao hàng. Trạng thái nằm ở cả mức đơn và mức từng món. Tiền nên lưu bằng `NUMBER` (Oracle), dùng `BigDecimal` trong Java, không dùng `double`.
</details>

**Q7. Bro test chức năng này thế nào?**
<details><summary>Gợi ý</summary>

Liệt kê các case: huỷ đúng trạng thái, sai trạng thái, không có quyền, hai người xác nhận cùng lúc, lỗi giữa chừng có rollback không, huỷ một phần (nếu có). Có test tự động (`JUnit` + `Mockito` cho service) hay chỉ test tay theo 단위테스트 결과서: `___`. Nói thật cách team đang làm, rồi nói thêm mình muốn cải thiện gì.
</details>

---

## 3. Câu chuyện N+1 (câu chuyện chính của bro)

**Điền khung này, đây là câu trả lời cho "kể một vấn đề kỹ thuật em đã giải quyết":**

| | |
|---|---|
| **Vấn đề** | Chức năng `___` (màn hình / API nào) chậm: ___ giây với ___ dòng dữ liệu. Ai phát hiện: ___ (khách báo / test / mình thấy trong log) |
| **Nguyên nhân** | Lấy danh sách `___` bằng 1 query, rồi **với mỗi dòng** lại gọi thêm 1 query để lấy `___` → 1 + N query (N = ___) |
| **Cách tìm ra** | ___ (bật log SQL, đếm số query; đo thời gian; xem execution plan) |
| **Cách sửa** | ___ (gộp thành JOIN + `resultMap`, hoặc 1 query `IN` với `<foreach>` rồi ghép lại trong Java) |
| **Kết quả** | ___ query → ___ query; ___ giây → ___ giây |
| **Rủi ro đã kiểm** | ___ (kết quả trước và sau có giống nhau không, phân trang, giới hạn 1000 phần tử của `IN`) |

**Q8. Trong MyBatis, N+1 xuất hiện ở những dạng nào?**
<details><summary>Gợi ý</summary>

1. **Vòng lặp trong Java**: `for (Order o : orders) { o.setItems(itemMapper.findByOrderId(o.getId())); }`
2. **Nested select trong `resultMap`**: `<collection property="items" select="findItemsByOrderId" column="order_id"/>`. MyBatis chạy câu select con cho **từng** dòng cha. Bật `lazyLoadingEnabled` cũng không bớt được số query, chỉ dời lúc chạy.

Code cũ bị dạng nào: `___`.
</details>

**Q9. Có những cách sửa nào? Bro chọn cách nào, vì sao?**
<details><summary>Gợi ý</summary>

- **JOIN + nested result map** (1 query):
  ```xml
  <resultMap id="orderWithItems" type="Order">
    <id property="orderId" column="order_id"/>
    <result property="status" column="status"/>
    <collection property="items" ofType="OrderItem">
      <id property="itemId" column="item_id"/>
      <result property="qty" column="qty"/>
    </collection>
  </resultMap>
  ```
  Bắt buộc có `<id>` thì MyBatis mới gom các dòng con vào đúng dòng cha. Nhược điểm: dữ liệu cha bị lặp trên mỗi dòng con, và **phân trang theo dòng cha bị sai** nếu `ROWNUM`/`FETCH FIRST` áp lên kết quả đã JOIN.
- **Hai query** (1 + 1): lấy danh sách cha (có phân trang), rồi `WHERE order_id IN (<foreach>)` lấy hết dòng con, ghép bằng `Map<orderId, List<Item>>` trong Java. Phân trang đúng, mỗi query đơn giản.
- Bro chọn `___` vì `___`.
</details>

**Q10. Dùng `IN (...)` với `<foreach>` trên Oracle có bẫy gì?**
<details><summary>Gợi ý</summary>

- Oracle giới hạn **1000 phần tử** trong một `IN` list (`ORA-01795`). Phải chia lô 1000, hoặc đổi sang JOIN / subquery / bảng tạm.
- Danh sách rỗng sinh ra `IN ()` là lỗi cú pháp, phải chặn trước bằng `<if test="list != null and list.size() > 0">`.
- Mỗi độ dài list khác nhau là một câu SQL khác nhau đối với Oracle (bị hard parse lại, cache câu lệnh kém hiệu quả). Chia lô cố định cũng giúp chỗ này.
</details>

**Q11. Bro đo "nhanh hơn" bằng gì? Làm sao chắc kết quả không đổi?**
<details><summary>Gợi ý</summary>

- Đếm query: log SQL (`log4jdbc`, `p6spy`, hoặc log level `DEBUG` cho package mapper).
- Đo thời gian API trước và sau, với **cùng bộ dữ liệu**, đo nhiều lần.
- Xem plan: `EXPLAIN PLAN FOR <query>;` rồi `SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY);` để biết có `INDEX RANGE SCAN` hay `TABLE ACCESS FULL`.
- Kết quả đúng: so output API cũ và mới trên cùng dữ liệu (số dòng, thứ tự, giá trị).
- Thiếu bước nào thì nói thật: "lúc đó em chỉ đo ___, giờ em sẽ làm thêm ___".
</details>

**Q12. Sửa xong N+1 mà vẫn chậm thì nhìn tiếp chỗ nào?**
<details><summary>Gợi ý</summary>

Index cho cột JOIN/WHERE (ví dụ `order_items(order_id)`); câu query có bọc cột trong hàm không (`WHERE TO_CHAR(order_date,'YYYYMMDD') = ...` làm mất index, nên viết thành khoảng `order_date >= ... AND order_date < ...`); có ép kiểu ngầm không (so cột `VARCHAR2` với số); có lấy thừa cột (`SELECT *`) không; có cần phân trang không.
</details>

---

## 4. MyBatis

**Q13. `#{}` khác `${}` thế nào?**
<details><summary>Gợi ý</summary>

`#{}` sinh bind variable `?` (PreparedStatement), nên an toàn với SQL injection và Oracle dùng lại được execution plan. `${}` là **dán chuỗi thẳng vào SQL**: chỉ dùng cho tên cột / `ORDER BY` động, và phải lọc qua whitelist. Có `${}` nhận input người dùng là lỗ SQL injection.
</details>

**Q14. Dynamic SQL: `<if>`, `<where>`, `<choose>`, `<foreach>`, `<set>` dùng khi nào?**
<details><summary>Gợi ý</summary>

Màn hình tìm kiếm có nhiều điều kiện không bắt buộc: `<where>` + `<if>` (`<where>` tự bỏ `AND` thừa ở đầu). `<set>` cho update một phần các cột. `<foreach>` cho `IN` list và insert nhiều dòng. Bẫy: tất cả `<if>` đều false thì query không còn điều kiện, quét cả bảng hoặc update cả bảng.
</details>

**Q15. Vì sao dự án dùng MyBatis mà không dùng JPA?**
<details><summary>Gợi ý</summary>

Rất phổ biến ở dự án SI Hàn Quốc: SQL nghiệp vụ phức tạp, DBA cần đọc và tune được từng câu, schema có sẵn từ lâu, team quen. MyBatis: tự viết SQL, tự kiểm soát. JPA: ít code CRUD, nhưng SQL sinh ra khó đoán, dễ dính N+1 mà không biết. Câu trả lời tốt là nói đúng **đánh đổi**, không chê bên nào.
</details>

**Q16. Insert hoặc update hàng nghìn dòng bằng MyBatis cho nhanh thế nào?**
<details><summary>Gợi ý</summary>

`ExecutorType.BATCH` (gom lệnh, gửi theo lô), hoặc `<foreach>` sinh `INSERT ALL ... SELECT 1 FROM DUAL` (Oracle). Commit theo lô (ví dụ 1000 dòng), không commit từng dòng, cũng không giữ một transaction quá lớn.
</details>

**Q17. Cache cấp 1 và cấp 2 của MyBatis là gì?**
<details><summary>Gợi ý</summary>

Cấp 1: trong cùng một `SqlSession` (với Spring là trong cùng một transaction), cùng câu cùng tham số thì không query lại. Cấp 2: theo namespace của mapper, mặc định tắt, bật bằng `<cache/>`. Ít dùng vì dễ trả dữ liệu cũ khi có hệ thống khác cùng ghi vào DB.
</details>

**Q18. MyBatis 2.2.2: muốn nâng lên Spring Boot 3 thì phải làm gì?**
<details><summary>Gợi ý</summary>

`mybatis-spring-boot-starter` 2.x đi với Spring Boot 2.x. Spring Boot 3 cần starter 3.x, Java 17 trở lên, đổi `javax.*` sang `jakarta.*`. Nói được điều này cho thấy bro biết phiên bản mình đang dùng nằm ở đâu trong vòng đời của nó (Spring Boot 2.7 đã hết hỗ trợ bản miễn phí).
</details>

---

## 5. Oracle

**Q19. Phân trang trên Oracle viết thế nào?**
<details><summary>Gợi ý</summary>

- Oracle 12c trở lên: `ORDER BY order_date DESC OFFSET #{offset} ROWS FETCH NEXT #{size} ROWS ONLY`
- Bản cũ: `ROWNUM` lồng hai lớp. Bẫy: `WHERE ROWNUM <= 10 ORDER BY ...` lấy 10 dòng **trước** rồi mới sắp xếp, nên phải `ORDER BY` ở subquery bên trong.
- Dữ liệu lớn, trang sâu thì offset chậm, nên dùng keyset (`WHERE order_id < #{lastId}`).
- Dự án dùng Oracle bản `___`, viết phân trang kiểu `___`.
</details>

**Q20. `MERGE INTO` dùng làm gì?**
<details><summary>Gợi ý</summary>

Upsert của Oracle: `MERGE INTO t USING (SELECT ... FROM DUAL) s ON (t.id = s.id) WHEN MATCHED THEN UPDATE ... WHEN NOT MATCHED THEN INSERT ...`. Tương đương `INSERT ... ON CONFLICT` của Postgres (S6 trong file kia).
</details>

**Q21. Các điểm khác thường của Oracle hay bị hỏi**
<details><summary>Gợi ý</summary>

- **Chuỗi rỗng `''` là `NULL`**: `WHERE col = ''` không bao giờ đúng, phải dùng `IS NULL`.
- `NVL`, `NVL2`, `DECODE` (riêng Oracle) so với `COALESCE`, `CASE` (chuẩn SQL).
- `DATE` có cả giờ phút giây; `TRUNC(date)` để bỏ phần giờ. `SYSDATE` so với `SYSTIMESTAMP`.
- Sinh khoá: `SEQUENCE` (`seq.NEXTVAL`), Oracle 12c trở lên có thêm cột `IDENTITY`. Trong MyBatis lấy khoá vừa sinh bằng `<selectKey order="BEFORE">`.
- Isolation mặc định là `READ COMMITTED`; đọc không chặn ghi (dùng undo segment).
- Lỗi hay gặp: `ORA-00001` (trùng unique), `ORA-01795` (IN quá 1000), `ORA-00054` (dòng đang bị khoá, với `NOWAIT`).
</details>

**Q22. Đọc execution plan: `TABLE ACCESS FULL`, `INDEX RANGE SCAN`, `NESTED LOOPS`, `HASH JOIN` nghĩa là gì?**
<details><summary>Gợi ý</summary>

Full scan đọc cả bảng (không phải lúc nào cũng xấu, với bảng nhỏ hoặc khi lấy phần lớn số dòng thì lại nhanh hơn). Range scan đi theo index. Nested loops hợp khi một bên ít dòng và bên kia có index ở cột join; hash join hợp khi join hai tập lớn. Hint (`/*+ INDEX(t idx_name) */`) chỉ dùng khi đã hiểu vì sao optimizer chọn sai, và nên hỏi DBA.
</details>

---

## 6. API cho Android

**Q23. Thiết kế API cho app Android khác API cho web chỗ nào?**
<details><summary>Gợi ý</summary>

App cũ vẫn chạy trên máy người dùng (không bắt cập nhật được ngay), nên **không được làm hỏng API cũ**: chỉ thêm field, không đổi/xoá field; cần versioning (`/v2/...`) khi phải đổi. Mạng yếu, nên payload gọn, có timeout, có retry, và API ghi phải idempotent (Q3). Mã lỗi rõ ràng để app hiện đúng thông báo. Không có CORS (đó là chuyện của browser).
</details>

**Q24. Xác thực request từ app thế nào?**
<details><summary>Gợi ý</summary>

Dự án dùng `___` (session / JWT / token riêng của khách). Biết giải thích token hết hạn thì làm gì, và token lưu ở đâu trên máy.
</details>

---

## 7. Quy trình, team, khách Hàn

**Q25. Team 3 dev làm waterfall: một chức năng đi từ yêu cầu tới production thế nào? Bro tham gia những giai đoạn nào?**
<details><summary>Gợi ý</summary>

요구사항 정의 (yêu cầu) → 설계 (thiết kế: 화면설계서, DB설계서, 인터페이스 정의서) → 개발 → 단위테스트 → 통합테스트 → UAT với khách → 오픈 (go-live). Bro tham gia giai đoạn `___`, viết tài liệu `___`.
</details>

**Q26. Đang code mà khách đổi yêu cầu (rất hay gặp với waterfall) thì sao?**
<details><summary>Gợi ý</summary>

Ghi lại thành change request, ước lượng ảnh hưởng (tới tài liệu, code, test, lịch), báo PM/BrSE để khách đồng ý về lịch và chi phí trước khi làm. Kể một lần thật nếu có: `___`.
</details>

**Q27. Waterfall và Agile, bro thấy ưu nhược thế nào?**
<details><summary>Gợi ý</summary>

Waterfall: yêu cầu và tài liệu rõ, hợp hợp đồng giá cố định và khách lớn như tập đoàn; nhưng phát hiện sai muộn (tới UAT mới thấy), đổi yêu cầu tốn kém. Agile: phản hồi sớm, nhưng cần khách tham gia thường xuyên. Đừng chê waterfall: công ty phỏng vấn có thể cũng đang làm waterfall.
</details>

**Q28. Làm việc với khách Hàn: giao tiếp qua ai, bằng gì? Có hiểu lầm nào không?**
<details><summary>Gợi ý</summary>

Qua `___` (BrSE / comtor / trực tiếp), bằng `___` (email, KakaoWork, Jira). Có một ví dụ hiểu lầm yêu cầu và cách bro xử lý là câu trả lời rất mạnh. Câu này cũng là chỗ khoe được tiếng Hàn (TOPIK 3): chuẩn bị sẵn 2–3 câu tiếng Hàn giới thiệu chức năng xác nhận huỷ.
</details>

---

## 8. Câu hỏi xoáy hay gặp

- Code đó em tự viết hay sửa từ code có sẵn? Phần nào là của em?
- Nếu làm lại chức năng xác nhận huỷ, em đổi gì?
- Dữ liệu bảng đơn hàng có bao nhiêu dòng? Query chậm nhất em từng thấy là gì?
- Có sự cố production nào liên quan tới chức năng của em chưa? Xử lý thế nào?
- Em review code của người khác thế nào? Có test tự động không?
- Tại sao muốn rời dự án / công ty hiện tại?

---

## 9. Câu chuyện: app báo timeout nhưng server vẫn cập nhật dữ liệu

**Chuyện thật:** app Android đặt timeout 30 giây. Người dùng bấm xác nhận huỷ, app rơi vào `catch` và báo lỗi. Nhưng backend vẫn chạy tiếp và cập nhật xong; người dùng tìm kiếm lại thì thấy đơn đã được huỷ.

**Vì sao:** timeout ở phía app chỉ làm **app ngừng chờ**. Nó không báo gì cho server, server cũng không biết app đã bỏ đi.
```
 0s  App gửi POST ───────────────>  Server mở transaction, bắt đầu xử lý
                                    ... chậm (chờ khoá, query chậm, gọi API ngoài) ...
30s  App hết read timeout → catch → báo "thất bại"
35s                                 Server commit xong, ghi response vào kết nối đã đóng
                                    → log Tomcat có ClientAbortException / Broken pipe
40s  Người dùng tìm lại → đơn đã huỷ (app vừa báo sai)
```
Bản chất: hết timeout **không có nghĩa là thất bại**, mà là **không biết kết quả**.

**Điền khung câu chuyện:**

| | |
|---|---|
| **Vấn đề** | App báo lỗi khi xác nhận huỷ, nhưng dữ liệu đã được cập nhật. Người dùng `___` (bấm lại / báo lỗi / ...) |
| **Nguyên nhân** | Khoảng 1/3 số lần, server mất `___` giây (> 30s timeout của app) ở câu `MERGE` vì `___` (chờ khoá / thiếu index). App coi timeout là thất bại |
| **Cách tìm ra** | `___` (log thời gian của API, `ClientAbortException` trong log, thời điểm commit so với thời điểm app báo lỗi) |
| **Cách sửa** | `___` (xem các lớp bên dưới) |
| **Kết quả** | API còn `___` giây; số lần báo sai `___` → `___` |

**Cách sửa, theo từng lớp (nên làm cả 4):**

1. **Tìm vì sao server chậm hơn 30 giây.** Đây mới là nguyên nhân gốc. Hay gặp nhất:
   - Chờ khoá: `SELECT ... FOR UPDATE` (không có `NOWAIT`/`WAIT`) đứng chờ một session khác (batch, người dùng khác) đang giữ dòng đó.
   - Query chậm hoặc N+1 (mục 3).
   - Gọi API ngoài (hoàn tiền, POS) bên trong transaction (`Q5`, `SP30`).

2. **Timeout của server phải ngắn hơn timeout của app**, để server tự rollback trước khi app bỏ đi:
   ```java
   @Transactional(timeout = 20)          // giây; MyBatis-Spring áp nó làm query timeout cho các câu SQL
   public CancelResult confirm(long orderId, String userId) { ... }
   ```
   ```xml
   <!-- hoặc đặt cho từng câu: -->
   <update id="confirmCancel" timeout="10"> ... </update>
   <!-- và không chờ khoá vô hạn: -->
   SELECT ... FROM orders WHERE order_id = #{orderId} FOR UPDATE WAIT 5
   ```
   Thứ tự nên là: timeout của câu SQL < timeout của transaction < timeout của proxy/load balancer < timeout của app.
   Lớp này **không loại hết** được vấn đề: server vẫn có thể commit ở giây 29,9 và response tới app chậm hơn một chút. Nên vẫn cần lớp 3 và 4.

3. **API phải idempotent:** gọi lại lần hai thì trả kết quả như lần đầu, không làm lại:
   ```java
   int updated = orderMapper.confirmCancel(orderId, userId);   // UPDATE ... WHERE status = 'CANCEL_REQUESTED'
   if (updated == 0) {
       Order o = orderMapper.findById(orderId);
       if ("CANCEL_CONFIRMED".equals(o.getStatus())) {
           return CancelResult.alreadyConfirmed(o);            // 200: lần trước đã làm xong rồi
       }
       throw new BusinessException(ErrorCode.INVALID_ORDER_STATUS);   // 409
   }
   ```
   Các việc kèm theo (hoàn tiền, trả tồn kho) cũng không được chạy lần hai. Việc này cũng che luôn trường hợp app hoặc thư viện HTTP tự gửi lại request.

4. **App: timeout thì hỏi lại kết quả, không báo thất bại ngay:**
   ```java
   try {
       CancelResult r = api.confirmCancel(orderId);
       showSuccess(r);
   } catch (SocketTimeoutException e) {
       // Không biết server đã làm xong chưa: hỏi lại trạng thái đơn
       Order o = api.getOrder(orderId);
       if (o.isCancelConfirmed()) showSuccess(o);
       else showError("Chưa xác nhận được, vui lòng thử lại");
   }
   ```
   Trong lúc chờ thì khoá nút bấm và hiện "đang xử lý". Phân biệt `setConnectTimeout` (không kết nối được, chắc chắn server chưa nhận gì) với `setReadTimeout` (đã gửi rồi, đang chờ trả lời: **không biết kết quả**).

5. *(Nếu việc này vốn dĩ lâu)* Đổi sang bất đồng bộ: API nhận yêu cầu, trả `202 Accepted` ngay, xử lý ở nền; app hỏi lại trạng thái hoặc nhận push khi xong.

### Chẩn đoán thực tế: lỗi 1/3 số lần, transaction có câu `MERGE`

**Bước 1: so thời gian của request thành công với request bị timeout** (chỉ cần log, chưa sửa code).

| Request thành công mất | Nhiều khả năng là | Hướng sửa |
|---|---|---|
| Cũng chậm, khoảng 20–30 giây | `MERGE` chậm (full scan): thời gian dao động quanh mốc 30s nên lúc qua lúc không | Index cho cột trong `ON (...)` |
| Nhanh (1–2 giây), chỉ lần lỗi mới vượt 30s | **Chờ khoá**: lần đó trùng lúc session khác (người dùng khác, batch) đang giữ dòng mà `MERGE` cần | Rút ngắn thời gian giữ khoá, `WAIT n` |

Muốn đo từng câu thì bật log SQL của mapper (`logging.level.<package mapper>=DEBUG`), hoặc log thời gian quanh từng lời gọi mapper trong `confirm`.

**Bước 2: hỏi Oracle** (cần quyền đọc `V$`, nhờ DBA nếu không có):
```sql
-- Thời gian trung bình của câu MERGE (đơn vị giây)
SELECT sql_id, executions,
       ROUND(elapsed_time / NULLIF(executions, 0) / 1e6, 2) AS avg_sec,
       ROUND(buffer_gets  / NULLIF(executions, 0))          AS gets_per_exec
FROM v$sql
WHERE sql_text LIKE 'MERGE INTO TEN_BANG%';

-- Execution plan: tìm TABLE ACCESS FULL trên bảng đích
SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY_CURSOR('<sql_id>'));

-- Chạy NGAY LÚC đang bị treo: session của app đang chờ gì, ai đang chặn nó
SELECT sid, sql_id, event, blocking_session, last_call_et AS sec
FROM v$session
WHERE username = 'APP_USER' AND status = 'ACTIVE';

-- Bảng đích có index trên các cột trong ON (...) chưa
SELECT index_name, column_name, column_position
FROM all_ind_columns
WHERE table_name = 'TEN_BANG'
ORDER BY index_name, column_position;
```
`event = 'enq: TX - row lock contention'` kèm `blocking_session` là chờ khoá. Còn `db file scattered read` / `db file sequential read` là đang đọc dữ liệu: query chậm.

**Bước 3: sửa theo thứ tự ít rủi ro nhất cho dự án maintain:**

1. **Không đổi code:** thiếu index cho cột trong `ON (...)` thì thêm index (qua DBA, theo quy trình change của khách). Thường đây là cách sửa lớn nhất mà rủi ro nhỏ nhất.
2. **Đổi một chút trong service:** chuyển câu `MERGE` (hoặc câu khoá dòng dùng chung, như tồn kho hay số liệu tổng hợp) xuống **cuối** transaction, để khoá bị giữ ngắn nhất; gọi API ngoài thì đưa ra khỏi transaction (`SP30`).
3. **Lưới an toàn (nhỏ, nên làm luôn):** API xác nhận trả "đã xác nhận" khi gọi lại (lớp 3); app bị read timeout thì gọi lại **API tìm kiếm có sẵn** để kiểm tra trạng thái (lớp 4), không cần làm API mới.
4. **Timeout ở server (lớp 2): chỉ làm sau khi đã sửa bước 1–2.** Đặt timeout khi gốc vẫn còn chậm thì 1/3 số lần "báo lỗi nhưng thật ra đã xong" sẽ thành 1/3 số lần **thất bại thật**, người dùng không xác nhận được.

**Bẫy riêng của `MERGE`:** hai session cùng `MERGE` một key **chưa có**: cả hai đều thấy "không khớp" nên cùng `INSERT`, và một bên dính `ORA-00001` (Spring báo `DuplicateKeyException`). Nếu key đó có thể được tạo đồng thời thì bắt lỗi này rồi chạy lại `MERGE` một lần (lần sau sẽ rơi vào nhánh `UPDATE`).

### Khi câu `MERGE` nằm trong stored procedure dài

Không cần đọc hết procedure. Oracle ghi lại từng câu SQL bên trong nó cùng **số dòng**:
```sql
-- Mọi câu SQL chạy từ procedure/package này, chậm nhất lên đầu
SELECT s.sql_id,
       s.program_line#                                          AS dong,
       s.executions                                             AS so_lan_chay,
       ROUND(s.elapsed_time / NULLIF(s.executions, 0) / 1e6, 3) AS avg_sec,
       ROUND(s.elapsed_time / 1e6)                              AS tong_sec,
       SUBSTR(s.sql_text, 1, 60)                                AS dau_cau
FROM v$sql s
JOIN all_objects o ON o.object_id = s.program_id
WHERE o.object_name = 'TEN_PROCEDURE_HOAC_PACKAGE'
ORDER BY s.elapsed_time DESC;

-- Đúng lúc đang treo: procedure đang đứng ở câu nào, chờ gì, ai chặn
SELECT sid, sql_id, event, blocking_session, last_call_et AS sec
FROM v$session
WHERE username = 'APP_USER' AND status = 'ACTIVE';
```
Đọc kết quả:
- Mở procedure tới đúng **`dong`** của câu đứng đầu danh sách. Chỉ cần đọc đoạn đó.
- `so_lan_chay` lớn hơn nhiều lần so với số lần gọi API: câu đó nằm **trong vòng lặp** (cursor `FOR` loop), xử lý từng dòng một.
- Lấy `sql_id` để xem plan: `DBMS_XPLAN.DISPLAY_CURSOR('<sql_id>')`.

**Lướt procedure tìm mấy thứ này** (tìm bằng Ctrl+F, không cần hiểu hết logic):

| Tìm | Vì sao quan trọng |
|---|---|
| `COMMIT`, `ROLLBACK`, `PRAGMA AUTONOMOUS_TRANSACTION` | Procedure tự commit thì `@Transactional` của Java **không rollback được** phần đã commit. Lúc đó đặt timeout ở server có thể để lại dữ liệu nửa chừng |
| `FOR UPDATE`, `LOCK TABLE` | Chỗ có thể đứng chờ khoá |
| `MERGE` / `UPDATE` nằm trong `LOOP` | Xử lý từng dòng (row-by-row), chậm theo số dòng |
| `@` (ví dụ `@DBLINK_POS`) | Gọi sang DB khác qua DB link: chậm và phụ thuộc hệ thống kia |
| `WHEN OTHERS THEN NULL` | Lỗi bị nuốt im lặng, Java tưởng là thành công |
| Hàm bọc cột trong `WHERE`/`ON` (`TO_CHAR(col)`, `NVL(col, ...)`, `TRUNC(col)`) | Làm mất tác dụng của index |

**Gọi procedure từ MyBatis và đặt timeout:**
```xml
<select id="confirmCancel" statementType="CALLABLE" timeout="20">
  { call PKG_ORDER.CONFIRM_CANCEL(
      #{orderId, mode=IN,  jdbcType=NUMERIC},
      #{userId,  mode=IN,  jdbcType=VARCHAR},
      #{result,  mode=OUT, jdbcType=VARCHAR}) }
</select>
```
Quá 20 giây thì JDBC huỷ lời gọi (Oracle báo `ORA-01013`), và Spring rollback phần chưa commit. Phần procedure **đã tự commit** thì vẫn giữ nguyên, nên phải kiểm bảng ở trên trước khi bật timeout.

### Mỗi lần gọi API, server sinh một `slip_no` mới rồi truyền vào procedure

Chi tiết này đổi hai chuyện:

**1. Gọi lại không còn an toàn.** Lần gọi lại sau timeout sinh ra một `slip_no` **mới**, nên procedure có thể coi đó là một phiếu mới và xử lý lần hai (hai phiếu huỷ, trả tồn kho hai lần, hoàn tiền hai lần). Trừ khi procedure tự kiểm trạng thái đơn trước khi làm. Cần kiểm:
- Trong procedure có đoạn kiểm "đơn đã huỷ rồi thì dừng" không? Đoạn kiểm đó có khoá dòng đơn (`FOR UPDATE`) không? Không khoá thì hai request cùng lúc vẫn lọt qua.
- Phía Java: kiểm trạng thái đơn **trước khi sinh `slip_no`**. Đã `CANCEL_CONFIRMED` thì trả kết quả cũ (kèm `slip_no` cũ), không sinh số mới, không gọi procedure.
- Chặn ở mức DB *(nếu được phép thêm)*: mỗi đơn chỉ có một phiếu huỷ. Oracle không có partial index như Postgres, nhưng làm được bằng function-based index:
  ```sql
  CREATE UNIQUE INDEX ux_cancel_slip_per_order
    ON slip_table (CASE WHEN slip_type = 'CANCEL' THEN order_id END);
  ```

**2. Cách sinh `slip_no` có thể chính là nguyên nhân "1/3 số lần bị".** Có ba cách hay gặp:

| Cách sinh số | Khi nhiều người làm cùng lúc |
|---|---|
| `SEQUENCE` (`seq.NEXTVAL`) | Không khoá gì, không chờ. Số có thể bị nhảy (gap) |
| `SELECT MAX(slip_no) + 1` | Hai người lấy trùng một số → `ORA-00001` hoặc trùng phiếu; còn phải quét bảng |
| Bảng đánh số (số cuối theo cửa hàng/ngày), `UPDATE ... SET last_no = last_no + 1`, hay **`MERGE`** (chưa có dòng của ngày hôm nay thì `INSERT`, có rồi thì `UPDATE`) | **Khoá dòng đó tới khi commit.** Mọi giao dịch cùng cửa hàng, cùng ngày phải xếp hàng |

Cách thứ ba rất phổ biến ở hệ thống bán lẻ Hàn (전표번호 theo cửa hàng, theo ngày). Nếu việc lấy số nằm **chung transaction** với phần xử lý dài, thì khoá bị giữ suốt transaction:
```
Người A: [lấy số: khoá dòng S001-20261026] ──── xử lý 20s ──── [commit, nhả khoá]
Người B:        [lấy số → CHỜ A ...............................] → xử lý → quá 30s → app timeout
```
Lúc vắng thì không ai chờ ai (thành công), lúc đông thì chờ nhau (timeout). Đúng kiểu "khoảng 3 lần bị 1 lần".

**Kiểm chứng:** chạy câu `v$session` đúng lúc đang treo. Nếu `blocking_session` có giá trị và `sql_id` của session bị chặn trỏ vào câu lấy số (`UPDATE`/`MERGE` vào bảng đánh số) thì đúng là nó.

**Sửa (ít đổi logic nhất):** lấy số trong một **transaction riêng, commit ngay**, để khoá chỉ bị giữ vài mili giây:
```java
@Service
public class SlipNoService {
    @Transactional(propagation = Propagation.REQUIRES_NEW)   // commit ngay khi method này xong
    public String next(String storeCode, LocalDate day) {
        return slipMapper.nextSlipNo(storeCode, day);        // câu UPDATE/MERGE lấy số như cũ
    }
}
// CancelService (bean khác, xem SP12) gọi slipNoService.next(...) rồi mới gọi procedure.
```
Đánh đổi: transaction chính lỗi thì số đã lấy bị bỏ, dãy số có lỗ hổng. **Phải hỏi khách/BA xem có được phép nhảy số không** (phiếu kế toán đôi khi bắt buộc liên tục). Nếu việc lấy số nằm **bên trong** procedure thì phải sửa procedure (tách ra procedure riêng có `PRAGMA AUTONOMOUS_TRANSACTION`, hoặc chuyển ra Java như trên), đi qua DBA.

### Thực tế ở dự án: SP tự `COMMIT`, sai trạng thái thì `ROLLBACK` trong SP và trả `-1`

**Vì sao thêm `@Transactional` mà không rollback được:** `COMMIT` trong SP commit **toàn bộ** những gì đã làm trên connection đó, kể cả các câu mà Java chạy trước đó trong cùng `@Transactional`. Khi Spring muốn rollback thì không còn gì để rollback. Thêm nữa, SP trả `-1` chứ không ném exception, nên Spring tưởng là thành công và commit luôn.

```
@Transactional (Spring) ─┬─ Java: INSERT/UPDATE ...      ┐
                          ├─ call SP ─┬─ ... xử lý ...    │ COMMIT trong SP commit hết cả khối này
                          │           └─ COMMIT ──────────┘
                          └─ Spring rollback ──> không còn gì để rollback
```
Kết luận: **ranh giới transaction ở đây là SP, không phải Spring.** Nhưng có một điểm hay: `COMMIT`/`ROLLBACK` của SP áp cho **cả connection**, nên khi có `@Transactional`, các câu `INSERT` Java chạy trước đó đi **cùng một khối** với SP:

| | Có `@Transactional` | Không có `@Transactional` |
|---|---|---|
| Java `INSERT` trước khi gọi SP | Chưa commit. Session khác (SQL Developer) **chưa thấy** | Commit ngay sau từng câu (mybatis-spring tự commit khi không có transaction), session khác thấy ngay |
| Connection | Java và SP dùng chung một connection | Mỗi lời gọi mapper có thể lấy một connection khác trong pool |
| SP thành công (`COMMIT`) | Commit cả Java `INSERT` lẫn phần của SP | Phần SP được commit; Java `INSERT` đã commit từ trước |
| SP trả `-1` (`ROLLBACK` trong SP) | **Huỷ cả Java `INSERT`**: dữ liệu sạch | Java `INSERT` **vẫn còn**: dòng mồ côi, không có SP đi kèm |
| Lỗi hoặc timeout giữa chừng SP | Spring rollback được phần chưa commit | Java `INSERT` vẫn còn |
| Khoá dòng do Java `INSERT`/`UPDATE` tạo ra | Giữ tới khi SP `COMMIT` | Nhả ngay |

Kiểm nhanh: query từ SQL Developer **trong lúc** request đang chạy. Có `@Transactional` thì chưa thấy các dòng Java vừa `INSERT`; SP `COMMIT` xong mới thấy. Đó chính là hiện tượng đã gặp.

Lưu ý: kết luận ở bảng trên dựa vào việc SP dùng `ROLLBACK` toàn phần. Nếu SP dùng `ROLLBACK TO SAVEPOINT ...` thì chỉ phần của SP bị huỷ, còn Java `INSERT` thì vẫn chờ Spring quyết định.

**Cách xử lý hợp với dự án maintain:**

1. **Giữ `@Transactional`** nếu các bảng Java `INSERT` trước SP không được phép còn lại khi SP thất bại (thường là vậy). Chính nó làm cho Java `INSERT` và SP thành một khối. Ghi comment để người sau khỏi hiểu lầm: `// Java INSERT + SP chạy chung một connection; SP tự COMMIT/ROLLBACK cả khối`. Chỉ bỏ nếu các dòng đó **cố ý** được giữ lại kể cả khi SP lỗi (ví dụ bảng log yêu cầu). Hỏi lại BA hoặc người viết cũ nếu không chắc.
   Cái giá: khoá do Java tạo ra (nhất là khi lấy `slip_no` từ bảng đánh số) bị giữ tới khi SP `COMMIT`. Nên lấy `slip_no` bằng `REQUIRES_NEW` ở bean riêng (xem phần `slip_no` ở trên), còn các `INSERT` nghiệp vụ thì để trong transaction chính.
2. **Đổi `-1` thành phản hồi đúng nghĩa, và coi lần gọi lại là thành công:**
   ```java
   // Java INSERT + SP chung một connection; SP tự COMMIT/ROLLBACK cả khối
   @Transactional
   public CancelResult confirm(long orderId, String userId) {
       Order order = orderMapper.findById(orderId);
       if (order.isCancelConfirmed()) {
           return CancelResult.alreadyConfirmed(order);      // lần gọi lại sau timeout: không sinh slip_no mới
       }

       String slipNo = slipNoService.next(order.getStoreCode());   // REQUIRES_NEW: nhả khoá bảng đánh số ngay
       insertBusinessRows(order, slipNo);                    // các INSERT nghiệp vụ như cũ, chưa commit
       ConfirmCancelParam p = new ConfirmCancelParam(orderId, userId, slipNo);
       orderMapper.callConfirmCancel(p);                     // OUT param resultCode được MyBatis gán vào p

       if (p.getResultCode() == -1) {                        // SP đã tự ROLLBACK (cả các INSERT ở trên)
           Order now = orderMapper.findById(orderId);
           if (now.isCancelConfirmed()) {
               return CancelResult.alreadyConfirmed(now);    // người khác hoặc lần trước đã xác nhận xong
           }
           throw new BusinessException(ErrorCode.INVALID_ORDER_STATUS);   // 409 cho app
       }
       return CancelResult.ok(slipNo);
   }
   ```
3. **App:** bị read timeout thì tìm kiếm lại để xem trạng thái (như đã nói ở lớp 4).
4. **Kiểm 2 điều trong SP** trước khi đặt timeout ở server:
   - **Có mấy chỗ `COMMIT`, nằm ở đâu?** Chỉ một chỗ ở cuối thì SP là "tất cả hoặc không gì cả": timeout cắt ngang giữa chừng thì phần chưa commit bị huỷ, an toàn. Có `COMMIT` ở giữa (ví dụ trong vòng lặp) thì cắt ngang sẽ để lại dữ liệu **nửa chừng**: không đặt timeout ở server.
   - **Đoạn kiểm trạng thái có khoá dòng không?** (`SELECT ... FOR UPDATE` hoặc `UPDATE ... WHERE status = ...`). Chỉ `SELECT` bình thường thì hai người bấm cùng lúc đều qua được, đơn bị xử lý hai lần với hai `slip_no` khác nhau.

**Góc phỏng vấn:** "Em thêm `@Transactional` nhưng rollback không chạy. Tìm ra là vì SP tự commit, nên ranh giới transaction thật sự nằm ở SP. Em không sửa SP (rủi ro cao, phải qua DBA) mà sửa ở Java: xử lý mã trả về `-1`, chặn gọi lại, và để app kiểm lại trạng thái khi timeout." Câu này cho thấy hiểu transaction ở mức connection, không chỉ biết dùng annotation.

**Câu hỏi phỏng vấn có thể đi từ chuyện này:**
- Timeout ở client có làm server dừng không? (Không. Servlet không biết client đã ngắt cho tới khi ghi response.)
- Vì sao không chỉ tăng timeout của app lên 60 giây? (Che triệu chứng; server chậm hơn 60 giây thì lại y như cũ; người dùng phải chờ lâu; vẫn có trường hợp app báo sai.)
- Idempotency là gì, làm ở tầng nào? (`Q3`, `SP21`.)
- Nếu bước hoàn tiền gọi hệ thống khác thì sao? (`Q5`, `SP30`: tách transaction, trạng thái trung gian, job đối soát.)

---

## Việc cần làm (thêm vào tuần 1–2 của plan)

- [ ] Hỏi team hoặc xem lại code và ticket để điền hết các chỗ `___` (nhất là số liệu N+1)
- [ ] Vẽ luồng xác nhận huỷ và sơ đồ trạng thái đơn trên giấy
- [ ] Kiểm tra code xác nhận huỷ: đã chặn được trường hợp xác nhận đồng thời chưa (Q2)?
- [ ] Viết lại câu chuyện N+1 theo bảng ở mục 3, tập nói dưới 2 phút
- [ ] Câu chuyện timeout (mục 9): xem log để biết server thực tế mất bao nhiêu giây và chậm ở đâu, điền bảng, tập nói dưới 2 phút
- [ ] Tập pitch ở mục 1 bằng tiếng Việt, rồi bằng tiếng Hàn (tuần 5)
