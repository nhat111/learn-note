# Vibe coding hiệu quả cho dev

Mục tiêu: từ "prompt rồi xem kết quả" lên **dùng AI có quy trình**: tự viết spec, review, kiểm chứng.
Simon Willison gọi cách này là *vibe engineering* / *agentic engineering*, để phân biệt với vibe coding.

---

## 1. Setup một lần cho mỗi repo

- [ ] **`CLAUDE.md` / `AGENTS.md`**: bản đồ dự án, quy ước, lệnh chạy test/lint/build, luật review. Viết ngắn, chỉ chỗ cần đọc, không chép lại cả tài liệu. (`/init` tạo bản nháp đầu tiên.)
- [ ] **Một lệnh kiểm tra tất cả**: ví dụ `npm run check`, `./mvnw verify`. AI cần cách tự biết mình làm đúng hay sai.
- [ ] **CI chạy trên mỗi lần push**, và cấm merge khi CI đỏ.
- [ ] **Hooks** (tuỳ chọn): tự chạy lint/format sau mỗi lần AI sửa file.

## 2. Quy trình cho mỗi task (10 bước)

| # | Bước | Bro làm | AI làm |
|---|---|---|---|
| 1 | **Explore** | Hỏi: "đọc phần X, tóm tắt cách nó chạy, **chưa sửa gì**" | Đọc và tóm tắt |
| 2 | **Spec** | Viết mẫu 5 dòng (bên dưới) | Hỏi lại chỗ mơ hồ |
| 3 | **Plan** | Dùng plan mode; sửa plan; chia nhỏ (1 PR = 1 việc) | Đề xuất plan |
| 4 | **Test trước** | Liệt kê case bằng lời: "nếu A thì B" | Viết test theo các case đó |
| 5 | **Implement** | Theo dõi, ngắt khi thấy đi sai hướng | Code từng bước nhỏ |
| 6 | **Verify** | Chạy app thật, xem tận mắt | Chạy test/lint/build |
| 7 | **Review** | Đọc "Hướng dẫn review" + file lõi; trả lời câu hỏi tự kiểm tra | `/code-review`, `/security-review` |
| 8 | **Merge** | Merge, rồi kiểm tra trên production | — |
| 9 | **Ghi lại** | 3 dòng vào nhật ký PR | — |
| 10 | **Cải tiến** | Thấy AI lặp lại một lỗi thì viết thành luật trong `CLAUDE.md` hoặc thành skill | Soạn luật hoặc skill |

**Mẫu spec:**
```
Mục tiêu:   ...
Ràng buộc:  ...
Không làm:  ...
Xong khi:   ...
Kiểm tra:   ...
```

## 3. Giữ context sạch

- Task mới thì mở session mới, hoặc `/clear`. Context cũ dễ làm AI "nhớ nhầm".
- Đưa file và lỗi cụ thể (log, stack trace, ảnh chụp). Đừng chỉ nói "nó bị lỗi".
- Task dài thì bắt AI ghi tiến độ ra file (`TODO.md`), để session sau đọc tiếp được.

## 4. Những lỗi hay gặp (anti-pattern)

- ❌ "fix giúp" mà không nói người dùng thấy gì và mong muốn gì
- ❌ Merge vì CI xanh mà không hiểu code (test cũng do AI viết)
- ❌ Để AI **sửa test cho xanh** thay vì sửa code
- ❌ PR khổng lồ trộn nhiều việc, không ai review nổi
- ❌ Tin câu "đã xong" mà không tự kiểm tra
- ❌ Cài skill, plugin hay MCP lạ mà không đọc trước: chúng chạy lệnh trên máy bro

---

## 5. Skills và plugins nên biết (Claude Code)

**Có sẵn**
- `/init`: tạo `CLAUDE.md` từ codebase
- Plan mode: bắt AI lên plan trước khi viết code
- `/code-review`, `/security-review`, `/simplify`: review diff trước khi merge
- Hooks, subagents, MCP: dùng khi đã quen quy trình

**Công khai trên GitHub** (đọc README trước khi cài)
- [anthropics/skills](https://github.com/anthropics/skills): bộ skill chính thức của Anthropic. Chuẩn định dạng skill ở [agentskills.io](https://agentskills.io)
- [anthropics/claude-plugins-official](https://github.com/anthropics/claude-plugins-official): danh mục plugin chính thức
- [obra/superpowers](https://github.com/obra/superpowers): quy trình brainstorm → plan → TDD (red-green-refactor) → review. Rất hợp với bước 2–7 ở trên
- [github/spec-kit](https://github.com/github/spec-kit): spec-driven development, `/specify` → `/plan` → `/tasks` → implement. Dùng được với Claude Code, Copilot, Gemini CLI
- Tự viết skill: dùng skill `skill-creator` (trong anthropics/skills) để đóng gói quy trình của riêng bro

## 6. Nguồn tài liệu cập nhật liên tục

**Chính thức**
- [Claude Code docs](https://code.claude.com/docs): đọc trang *Best practices* trước
- [anthropics/claude-code](https://github.com/anthropics/claude-code): bấm Watch → Releases để theo dõi CHANGELOG
- [Anthropic Engineering blog](https://www.anthropic.com/engineering): các bài *Building effective agents*, *Effective context engineering*
- [anthropics/claude-cookbooks](https://github.com/anthropics/claude-cookbooks): code mẫu (tool use, RAG, eval, structured output)
- [agents.md](https://agents.md): chuẩn chung cho file hướng dẫn agent

**Danh sách cộng đồng**
- [hesreallyhim/awesome-claude-code](https://github.com/hesreallyhim/awesome-claude-code): danh sách lớn nhất (skills, hooks, agents, plugins)
- [travisvn/awesome-claude-skills](https://github.com/travisvn/awesome-claude-skills): tập trung vào skills

**Người nên theo dõi**
- [Simon Willison](https://simonwillison.net): viết gần như mỗi ngày về LLM và agentic engineering. Có RSS
- Addy Osmani: sách *Beyond Vibe Coding* và các bài về AI-assisted engineering
- [Hamel Husain](https://hamel.dev): **eval** cho app LLM (liên quan tuần 3 trong plan)
- Kent Beck: *augmented coding*, TDD khi làm việc cùng AI

**Java + AI** (liên quan backend korean-practice)
- [anthropics/anthropic-sdk-java](https://github.com/anthropics/anthropic-sdk-java): SDK bro đang dùng
- [spring-projects/spring-ai](https://github.com/spring-projects/spring-ai)
- [langchain4j/langchain4j](https://github.com/langchain4j/langchain4j)

**Cách theo dõi mà không bị ngợp:** 30 phút mỗi tuần (ví dụ tối Chủ nhật). Đọc CHANGELOG của Claude Code và 1–2 bài mới, rồi chọn **đúng 1 thứ** để thử vào project trong tuần.
