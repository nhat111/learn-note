# Plan 6 tuần: Java + AI, dùng AI chuyên nghiệp, phỏng vấn

**26/10 → 5/12/2026** · khoảng 8 tiếng/tuần (1 tiếng mỗi tối ngày thường + 3 tiếng cuối tuần).
Thiếu thời gian thì bỏ các mục *(mở rộng)*.

Mục tiêu: từ **bậc 1** (prompting, vibe coding) lên **bậc 3** (tự viết spec, review, đo bằng số).

| Bậc | Nghĩa là | Trạng thái |
|---|---|---|
| 1. Prompting | Nhờ AI, xem kết quả, sửa prompt | ✅ |
| 2. Context | Cho AI tài liệu dự án (`CLAUDE.md`, `ASSUMPTIONS.md`) | ✅ |
| 3. Spec + kiểm chứng | Tự viết yêu cầu và tiêu chí xong; review; đo bằng số | ⬜ mục tiêu đến 5/12 |
| 4. Workflow | Plan trước code, eval, review tự động, chia việc song song | ⬜ chạm ở tuần 5 |

---

## Tuần 1 (26/10–1/11): Hiểu cái mình đang có

**Java + AI**
- [ ] Đọc backend korean-practice: `AiProvider`, `ClaudeProvider`, `AiConfig`, filter CORS và access key
- [ ] Chạy `./mvnw test`, hiểu cách `ClaudeProviderTest` dựng server HTTP giả
- [ ] Vẽ đường đi của request `/api/email/check`: frontend → `client.ts` → CORS filter → access key filter → controller → provider → Claude → structured output → frontend
- [ ] Giải thích được vì sao CORS filter phải đứng đầu

**Dùng AI**
- [ ] Review và merge crypto-signal PR #54 theo hướng dẫn review
- [ ] Review và merge korean-practice PR #84
- [ ] Bắt đầu nhật ký PR (mẫu ở cuối file)

**PV**
- [ ] **Ưu tiên:** pitch 2 phút cho dự án công ty, điền các chỗ `___` trong [du-an-cong-ty.md](du-an-cong-ty.md)
- [ ] Pitch 2 phút cho crypto-signal (tiếng Việt)
- [ ] Pitch 2 phút cho korean-practice (tiếng Việt)
- [ ] [Java/SQL](phong-van-java-sql.md): A1–A6, S1–S5

## Tuần 2 (2–8/11): Java 21 / Spring Boot 3, tự tay

**Java**
- [ ] Java 21: record, sealed interface + `switch` pattern matching
- [ ] Virtual threads: vì sao hợp với lời gọi AI có thể chặn tới 120 giây
- [ ] Spring: `@ConfigurationProperties`, filter khác interceptor thế nào, `@ControllerAdvice`, test với MockMvc
- [ ] **Tự viết** (AI chỉ giải thích khái niệm): giới hạn số request cho `/api/roleplay` theo access key, kèm test
- [ ] Đưa Claude review bài đó như một senior

**Dùng AI**
- [ ] Mọi task đều viết spec trước khi prompt (mẫu ở cuối file)
- [ ] Dùng plan mode: sửa plan của AI trước khi cho viết code

**PV**: viết câu chuyện theo dạng Vấn đề → Quyết định → Đo lường → Kết quả
- [ ] Câu chuyện 0 (chính): tối ưu N+1 ở dự án công ty ([mục 3](du-an-cong-ty.md#3-câu-chuyện-n1-câu-chuyện-chính-của-bro)), có số trước và sau
- [ ] [Dự án công ty](du-an-cong-ty.md): Q1–Q12 (xác nhận huỷ, N+1)
- [ ] Câu chuyện 1: replay setup −0.39R, tắt alert
- [ ] Câu chuyện 2: Binance chặn IP Render nên chuyển qua OKX
- [ ] Câu chuyện 3: audio làm đầy 10GB của Vercel nên chuyển qua GitHub Pages + jsDelivr
- [ ] [Java/SQL](phong-van-java-sql.md): B1–B8, S6–S10

## Tuần 3 (9–15/11): Cốt lõi AI engineering là eval

**AI**
- [ ] Nắm các khái niệm: token, context window, temperature, structured output / JSON schema, tool use, prompt caching, timeout/retry, chi phí
- [ ] **Bộ eval cho `/api/email/check`**: lấy `draft` và `corrections[].wrong` trong `emails.json` làm lỗi đã biết; viết script đo AI bắt được bao nhiêu %
- [ ] So 2 phiên bản prompt bằng con số. Kết quả: A = ___ %, B = ___ %

**Dùng AI**
- [ ] Tự viết test case bằng lời trước, AI chỉ implement

**PV**
- [ ] Câu chuyện 4: iOS chặn `play()` sau một lần fetch chậm
- [ ] Câu chuyện 5: Kronos / TimesFM không tìm ra hướng giá
- [ ] Câu chuyện 6: bộ eval email (vừa làm)
- [ ] [Java/SQL](phong-van-java-sql.md): A7–A11, bài tập SQL 1–4

## Tuần 4 (16–22/11): Đưa app AI lên production đàng hoàng

**Java + AI**
- [ ] Log latency, số token và chi phí cho mỗi request
- [ ] Fallback về `NoAiProvider` khi Claude lỗi hoặc quá timeout
- [ ] Thử prompt injection ("bỏ qua hướng dẫn trên" trong email người dùng gửi)
- [ ] *(mở rộng)* Viết 1 trang so sánh Spring AI / LangChain4j với cách gọi SDK trực tiếp

**Dùng AI**
- [ ] 1 task "người trước": tự làm xong, rồi cho AI làm cùng task đó, so sánh hai bản

**PV**
- [ ] Mock interview #1 với Claude (tiếng Việt, hỏi xoáy kiến trúc và cách dùng AI)
- [ ] [Java/SQL](phong-van-java-sql.md): B9–B12, S11–S20

## Tuần 5 (23–29/11): Tiếng Hàn và chiều sâu

**Java**
- [ ] Ôn câu hỏi nền: collections, concurrency, JVM memory/GC, `@Transactional`, JPA N+1 ([phần C và câu nền](phong-van-java-sql.md))
- [ ] Bài tập SQL 5–8, đọc `EXPLAIN ANALYZE` của 1 query

**Dùng AI**
- [ ] Chạy `/code-review` trên PR của mình trước khi merge
- [ ] Khi thấy AI lặp lại một lỗi, viết thành luật trong `CLAUDE.md`

**PV**
- [ ] Dịch pitch và 3 câu chuyện sang tiếng Hàn (하십시오체)
- [ ] Dán vào "Bộ câu hỏi của tôi" trong app để luyện nói
- [ ] Mock #2 (một phần bằng tiếng Hàn)

## Tuần 6 (30/11–5/12): Đóng gói

- [ ] README crypto-signal: sơ đồ kiến trúc, quyết định chính, mục "Cách tôi làm việc với AI"
- [ ] README korean-practice: như trên
- [ ] Mock #3 (đầy đủ, như phỏng vấn thật), có 15 câu Java/SQL trộn và 2 bài SQL viết tay
- [ ] Chấm theo mốc 5/12 bên dưới

---

## Mốc 5/12

- [ ] Vẽ được kiến trúc 2 project mà không cần mở code
- [ ] Tự viết 1 feature Spring Boot có test, AI không viết code
- [ ] Có bộ eval với con số cụ thể
- [ ] 6 câu chuyện, pitch có cả tiếng Việt và tiếng Hàn
- [ ] Từ 80% PR trở lên chỉ merge sau khi trả lời đúng câu hỏi tự kiểm tra
- [ ] Đã qua 3 buổi mock interview
- [ ] Bảng ⚠️ trong [Java/SQL](phong-van-java-sql.md) không còn câu nào chưa ôn lại

## Câu hỏi phỏng vấn cần chuẩn bị

- [ ] Giới thiệu **dự án công ty** trong 2 phút ([du-an-cong-ty.md](du-an-cong-ty.md))
- [ ] Giới thiệu side project trong 2 phút
- [ ] Vẽ kiến trúc giúp anh/chị
- [ ] AI viết bao nhiêu? Bạn làm phần nào?
- [ ] Làm sao biết code AI viết là đúng?
- [ ] Một bug khó nhất bạn từng gặp
- [ ] Một quyết định bạn đã thay đổi, và vì sao
- [ ] App gọi AI xử lý lỗi, timeout và chi phí thế nào?
- [ ] Làm lại từ đầu thì bạn đổi gì?
- [ ] Java, Spring, SQL theo project: xem [phong-van-java-sql.md](phong-van-java-sql.md)

---

## Thói quen

- Không merge khi chưa trả lời được câu hỏi tự kiểm tra trong PR
- Mỗi PR ghi 3 dòng vào nhật ký
- Mỗi tuần làm 1 task "người trước"
- Hỏi AI "vì sao không làm cách khác?" ít nhất một lần mỗi PR

### Mẫu spec (viết trước mỗi prompt)

```
Mục tiêu:   ...
Ràng buộc:  ...
Không làm:  ...
Xong khi:   ...
Kiểm tra:   ...
```

### Nhật ký PR

| Ngày | Repo / PR | Đổi gì | Học được gì | Chỗ còn mờ |
|---|---|---|---|---|
| | | | | |
