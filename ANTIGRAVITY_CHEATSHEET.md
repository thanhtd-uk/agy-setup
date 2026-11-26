# Sổ Tay Sử Dụng Nhanh Google Antigravity + `agy-fast`

Tài liệu tra cứu nhanh để giữ lâu dài cho mọi dự án: cách chọn Model, luân phiên giữ Prompt Cache & Quota, chọn đúng Slash Command, và quản lý plugin `agy-fast`.

---

## 1. Bản Đồ Chọn Model & Mức Suy Luận (`Thinking Level`)

| Model | Điểm mạnh cốt lõi | Khi nào nên chọn? |
| :--- | :--- | :--- |
| **`Orchestrator`**<br>*(Planner: `Neon` hoặc `Carbon`)* | Tự động bắt cặp **2 model trong cùng 1 lượt**: con lớn (`Neon`/`Carbon`) đọc code, lập kế hoạch & nghiệm thu; con nhanh (`Hydrogen`) chạy tìm kiếm, sửa file, chạy test. Nếu vấp lỗi 2 lần tự trả về con lớn sửa kế hoạch. | **Mặc định hằng ngày (~80% công việc):** làm tính năng, sửa bug, refactor nhiều file — vừa giữ chất lượng cao nhất vừa nhanh và đỡ tốn quota. |
| **`Gemini 4 Neon`**<br>*(`Low` / `Medium` / `High` / `Extra`)* | Flagship đầu bảng, cửa sổ ngữ cảnh tới **512K tokens**, có thêm nấc suy luận tối đa **`Extra`** (`xhigh`). | Khi cần **1 bộ não duy nhất suy nghĩ cực sâu xuyên suốt mọi bước**: thiết kế kiến trúc hệ thống từ đầu, gỡ race-condition/deadlock hiểm, hoặc chạy `/boost`. |
| **`Gemini 4 Carbon`**<br>*(`Low` / `Medium` / `High`)* | Frontier chủ lực song hành cùng `Neon`, phong cách trực diện, đầm và chắc tay khi code/debug kỹ thuật dài hơi. | Code backend, hệ thống, bảo mật, refactor phức tạp nhiều bước (hoặc chọn làm Planner trong `Orchestrator`). |
| **`Gemini 4 Hydrogen`**<br>*(`Low` / `Medium` / `High` / `Extra`)* | Nhanh nhất, nhẹ quota nhất dòng Gemini 4. | Việc rõ ràng không cần lập kế hoạch riêng: viết script nhanh, tạo boilerplate, nâng cấp package, sửa UI nhỏ, viết unit test cho hàm đã rõ. |

> **Lưu ý lịch chuyển giao:** Nhóm gắn nhãn `Leaving Soon` / `Notice` (`Gemini 4 Argon`, `Gemini 3.8 Flash`, `Claude 4.6`, `GPT-OSS 120B`) ngừng phục vụ vào **30/11/2026**. Cứ xoay quanh bộ tứ **`Orchestrator` · `Neon` · `Carbon` · `Hydrogen`** làm trục chính.

---

## 2. Bốn Nguyên Tắc Luân Phiên Model & Giữ Cache/Quota

1. **Không đổi Model (kể cả gạt đổi nấc `Thinking Level`) ngay giữa phiên chat đang dài (>50k–100k tokens):**
   - Trong Antigravity, mỗi nấc suy luận (`Low`/`Medium`/`High`/`Extra`) thay đổi trực tiếp phần đầu của System Prompt (Prefix) và định danh model (`-low`, `-medium`, `-high`). Do Prompt Cache khớp từ đầu xuống cuối (Prefix Matching), cả việc đổi model lẫn gạt đổi nấc `Thinking Level` giữa phiên đang nóng đều **làm mất Prompt Cache ở lượt kế tiếp**.
   - *Cách chuẩn:* Chọn sẵn đúng nấc ngay từ lúc mở phiên mới. Muốn vừa có não lớn lập kế hoạch vừa có tay nhanh viết code tự động mà không phải gạt tay → chọn **`Orchestrator`** ngay từ đầu.
2. **Chọn đúng nấc `Thinking Level` ngay từ đầu phiên theo độ khó công việc:**
   - **`Medium`**: Điểm cân bằng tốt nhất cho đa số việc hằng ngày.
   - **`High` / `Extra`**: Chỉ bật khi giải bài toán khó, kiến trúc phức tạp, bug ngầm hiểm hóc.
   - **`Low`**: Khi cần phản hồi tức thì cho thao tác vặt (đổi tên, format, viết tài liệu ngắn).
3. **Chia tải thông minh qua 2 bể Quota độc lập:**
   - **Bể 1 (Họ Gemini):** `Orchestrator`, `Neon`, `Carbon`, `Hydrogen` dùng chung 1 hạn mức Gemini.
   - **Bể 2 (Đối tác ngoài Gemini):** Dòng `Claude` (Opus / Sonnet) dùng bể hạn mức riêng biệt. Khi cày dự án lớn làm vơi bể Gemini, mở phiên mới với dòng Claude để chia tải.
4. **Quy trình chuẩn khi hết Quota phải tắt App đổi sang Account khác (`agy-auth`):**
   - **Cơ chế Cache:** Lịch sử chat & Artifacts lưu trên ổ cứng nên còn nguyên, nhưng KV-cache trên server bị cô lập theo từng tài khoản. Lượt đầu tiên sau khi đổi account chắc chắn bị **Cache Miss 1 lần** (chậm hơn ~4–12s và tính lại token lịch sử), nhưng **từ bước gọi tool thứ 2 trở đi trong cùng lượt đó cache sẽ tự ấm lại 90–95%** trên account mới.
   - **Đánh thức tiến trình nền:** Tắt mở lại app sẽ làm dừng các `dev server` chạy nền, background tasks và đưa các worker của `/swarm` về trạng thái ngủ (`stopped`). Thay vì chỉ gõ `"continue"`, hãy nhắn:
     > *"Kiểm tra lại trạng thái file/tiến trình hiện tại rồi tiếp tục hoàn thành phần việc đang dở."*
   - **Tránh ngốn mạnh Quota của Account mới khi phiên cũ đã quá dài (`>180k` tokens):**
     - *Nếu đang làm dở giữa chừng:* Mở lại phiên cũ trên account mới → gõ **`/compact`** để nén gọn ngữ cảnh trước → rồi mới bảo làm tiếp.
     - *Nếu vừa xong một cột mốc tròn trịa (VD: vừa chốt xong `/plan`):* Mở hẳn **New Conversation** trên account mới rồi dùng `@` đính kèm file Plan/tài liệu vừa chốt để thi công từ ngữ cảnh sạch `0 token`.

---

## 3. Khi Nào Gõ Lệnh `/` Nào?

| Lệnh | Bản chất | Tình huống sử dụng chuẩn nhất |
| :--- | :--- | :--- |
| **`/plan <việc>`** | Quét repo và xuất tài liệu **Implementation Plan Artifact** để duyệt/góp ý trước khi sửa code | Khi bắt đầu tính năng mới hoặc thay đổi chạm tới nhiều file cần chốt thiết kế trước |
| **`/grill-me <ý tưởng>`** | Agent chủ động phỏng vấn ngược lại bạn từng góc khuất thiết kế trước khi chốt kế hoạch | Khi ý tưởng còn sơ khai, nhiều điểm chưa quyết định rõ |
| **`/boost <bài toán>`** | Dây chuyền đa agent **Làm thử → Tự phản biện tìm lỗ hổng → Sửa → Kiểm định độc lập** | Bài toán **hẹp nhưng cực khó về chiều sâu** (race condition, memory leak, tối ưu thuật toán, root-cause bug lạ) |
| **`/swarm <việc>`** | Coordinator chia việc cho nhiều Worker làm **song song** trên bảng tin chung + `@verifier` nghiệm thu | Việc **rộng về chiều ngang**, tách được thành nhiều mảng/module độc lập (Backend + Frontend + Tests, hoặc sửa đồng loạt chục module) |
| **`/teamwork <mục tiêu>`** | Đội hình nhiều vai trò chuyên biệt chạy qua từng Milestone có cổng kiểm duyệt khắt khe | Dự án quy mô lớn kéo dài hàng giờ/nhiều ngày (migration cả repo lớn, xây hệ thống lớn từ đầu) |
| **`/browser <yêu cầu>`** | Cử **Browser Subagent** mở trình duyệt tích hợp, tự click/nhập liệu/soi lỗi Console & chụp ảnh | Kiểm thử tự động giao diện Web `localhost` hoặc thao tác trên ứng dụng web thực tế |
| **`/goal <mục tiêu>`** | Giữ agent tự làm bền bỉ đến cùng không dừng sớm cho tới khi đạt trọn mục tiêu | Tác vụ chạy dài (ví dụ sửa hết toàn bộ lỗi typecheck/test của một dự án lớn) |
| **`/compact`** | Tóm tắt nén gọn lịch sử hội thoại để giải phóng cửa sổ ngữ cảnh | Khi phiên làm việc đã kéo dài >150k tokens hoặc xong một cột mốc lớn |

> **Bài học thực chiến về `/boost` (Tránh mất 30–50 phút & tốn x4 Quota oan):**
> 1. **Tuyệt đối không lạm dụng `/boost` để viết tính năng mới có phạm vi mở:** Các vòng phản biện dễ rơi vào bẫy *Over-engineering* (tự chế thêm cơ chế phụ rồi lại tự bắt lỗi edge-case nhỏ của chính phần phụ đó, đẻ thêm Round 2–3 kéo dài hàng chục phút). Viết tính năng mới (kể cả phức tạp vừa phải) cứ chạy **`Orchestrator`** hoặc `Neon`/`Carbon` `High` thường — xong trong **2–4 phút**.
> 2. **Cơ chế nhân bản Token ngầm:** Mỗi Worker (`Layer 0`, `Improvement`, `Round 2`...) là một conversation độc lập chỉ nhận *Handoff message* tóm tắt — giữ sạch khung chat chính nhưng **không dùng chung Prompt Cache** với Worker trước (mỗi con đều phải tự đọc lại bộ file code từ đầu), cộng thêm con điều phối chờ lâu sẽ nguội cache → tốn xấp xỉ **3–5 lần quota** so với làm 1 mạch.
> 3. **Khi nào nên rút `/boost` & Mẹo khóa biên:** Chỉ dùng khi **diệt bug hiểm mà chat thường sửa 2 lượt không dứt** hoặc **tối ưu thuật toán có tiêu chí chấm rõ ràng**. Khi viết prompt cho `/boost`, luôn chốt thêm câu khóa biên: *"Chỉ giải quyết đúng phạm vi yêu cầu, không tự ý thêm cơ chế ngoài đề bài"* để tránh đẻ vòng thừa.

---

## 4. Playbook Kịch Bản 1: Dự Án Có Sẵn, Đang Dang Dở Hoặc Từ Hệ Agent Khác Sang

Đây là tình huống chiếm phần lớn công việc thực tế. Mục tiêu cốt lõi là **nắm đúng hiện trạng, đồng bộ quy tắc cũ và không phá vỡ code đang chạy**.

### Tình huống 1A: Repo từng làm bằng hệ Agent/IDE khác (Cursor, Claude Code, Codex...)
Các repo này thường có sẵn `.cursorrules`, `.cursor/rules/*.mdc`, `CLAUDE.md`, `.claude/`, hoặc cấu hình MCP cũ. Antigravity tự đọc `AGENTS.md` / `GEMINI.md`, và có sẵn công cụ di trú tự động các thiết lập còn lại.

- **Model khuyên dùng:** `Orchestrator` (Medium)
- **Prompt mẫu mở đầu:**
  ```text
  Kiểm tra toàn bộ các file cấu hình/rule của công cụ AI cũ trong repo này (như .cursor, .cursorrules, CLAUDE.md, .claude, AGENTS.md, mcp...). Hãy tóm tắt những quy tắc quan trọng đang áp dụng cho dự án, chuyển đổi hoặc gom gọn chúng về chuẩn .agents/ của Antigravity nếu cần (tránh trùng lặp gây tốn context), và báo cáo trước cho tôi duyệt trước khi xóa/sửa file gốc.
  ```

### Tình huống 1B: Dự án đang làm dở dang (In-Progress / Handoff giữa chừng)
Khi nhảy vào một nhánh code đang viết dở (từ phiên cũ, đồng nghiệp, hoặc công cụ khác để lại), tuyệt đối không bảo agent *"làm tiếp đi"* ngay khi chưa đối chiếu hiện trạng.

- **Model khuyên dùng:** `Orchestrator` (Medium / High)
- **Prompt mẫu "Khám nghiệm hiện trường & Lên Plan chặng cuối":**
  ```text
  /plan Hãy khảo sát hiện trạng dang dở của repo này trước khi viết thêm code:
  1. Kiểm tra git status, git diff, 10 commit gần nhất và các chỗ đánh dấu TODO/FIXME hoặc file mới tạo chưa hoàn thiện.
  2. Chạy thử build/typecheck/test hiện tại xem đang pass hay gãy ở đâu.
  3. Đối chiếu để xác định rõ: phần nào đã làm xong, phần nào đang viết dở hoặc còn thiếu test/edge-case, rồi lập bản kế hoạch hoàn thiện nốt các hạng mục còn lại.
  ```

### Tình huống 1C: Lần đầu tiếp nhận một Codebase lớn / Lạ lẫm (Onboarding Audit)
Tận dụng thế mạnh `quick_outline` + `batch_search` của `agy-fast` để vẽ bản đồ dự án vào một file **Artifact** mà không làm bẩn khung chat chính hay đụng chạm code.

- **Model khuyên dùng:** `Orchestrator` (Medium)
- **Prompt mẫu:**
  ```text
  Khảo sát kiến trúc toàn bộ dự án này (chỉ đọc, không sửa bất kỳ file nào) và xuất ra một Artifact báo cáo gồm:
  1. Sơ đồ Mermaid luồng khởi chạy chính, các tầng module và cơ sở dữ liệu/external services.
  2. Các lệnh chạy dev, build, test thực tế và trạng thái sức khỏe hiện tại của bộ test/typecheck.
  3. Danh sách 3-5 điểm rủi ro kỹ thuật, nút thắt hiệu năng hoặc nợ kỹ thuật đáng chú ý nhất kèm link tới file/dòng code cụ thể.
  ```

### Tình huống 1D: Thêm tính năng mới hoặc Diệt Bug khó trên dự án đang chạy ổn định
- **Nếu thêm tính năng / refactor chạm nhiều file:**
  1. Chạy `/plan <mô tả tính năng + yêu cầu không làm gãy hành vi cũ>` để duyệt danh sách file sẽ chạm tới.
  2. Nếu các phần độc lập nhau (nhiều API/UI/tests tách rời) → gõ `/swarm` thi công theo bản Plan kèm `@verifier` gác cổng test hồi quy. Nếu logic xoắn chặt vào nhau → để nguyên `Orchestrator` thi công tuần tự.
  3. Nếu là Web App → chốt bằng `/browser` để tự mở `localhost` kiểm tra luồng cũ + luồng mới.
- **Nếu săn Bug chập chờn / Race-condition / Tụt hiệu năng khó hiểu:**
  - Mở phiên mới với `Gemini 4 Neon` (High) hoặc `Gemini 4 Carbon` (High) và gõ:
    ```text
    /boost Truy tìm nguyên nhân gốc rễ (root-cause) của lỗi [mô tả hiện tượng lỗi], viết test tái hiện đúng lỗi đó trước, sau đó sửa triệt để và chạy lại toàn bộ test suite để đảm bảo không gây lỗi hồi quy sang các module khác.
    ```

---

## 5. Khung Quy Trình Nhanh Cho Kịch Bản 2: Dựng Dự Án Mới Từ Số 0 (Greenfield)

Khi bắt đầu từ một thư mục trống (như bài thực hành `pulse-ops`):
1. **Ý tưởng còn mở / chưa chốt hết chi tiết:** Gõ `/grill-me <ý tưởng>` để agent hỏi xoáy làm rõ các quyết định kiến trúc, tech stack, schema.
2. **Chốt thiết kế khung:** Gõ `/plan <đề bài>` để duyệt sơ đồ kiến trúc và phân tách cấu trúc thư mục rõ ràng (tách đủ module để dễ làm song song).
3. **Thi công tốc độ cao:** Gõ `/swarm` để nhiều worker dựng song song Backend + Frontend + Test Suite, có `@verifier` ghép nối và chạy test.
4. **Kiểm thử giao diện thực tế:** Gõ `/browser` để agent tự chạy server nền, mở trình duyệt bấm test từng luồng và chụp ảnh nghiệm thu.
5. **Gia cố các cụm logic cực khó (Tùy chọn có chọn lọc):** Chỉ dùng `/boost` cho bài toán thuật toán hóc búa hoặc khi phát sinh bug đồng thời/race-condition khó gỡ (các tính năng thông thường cứ dùng `Orchestrator` để hoàn thành trong vài phút thay vì chờ nhiều vòng phản biện).

---

## 6. Các Lệnh Quản Lý `agy-fast` Thường Dùng (PowerShell)

```powershell
# Kiểm tra sức khỏe plugin & phiên bản hiện tại
node "$env:USERPROFILE\.gemini\config\plugins\agy-fast\scripts\install.mjs" doctor

# Kiểm tra xem có bản cập nhật mới không (và hỏi xác nhận trước khi nâng cấp)
node "$env:USERPROFILE\.gemini\config\plugins\agy-fast\scripts\install.mjs" update --prompt

# Đo hiệu quả thực tế Trước vs. Sau khi dùng agy-fast trên máy (100% offline)
node "$env:USERPROFILE\.gemini\config\plugins\agy-fast\dist\benchmark.mjs" analyze --local --split-usage

# Gỡ cài đặt sạch 100% (nhớ tắt plugin trong Customizations hoặc đóng app Antigravity trước)
node "$env:USERPROFILE\.gemini\config\plugins\agy-fast\scripts\install.mjs" uninstall
Remove-Item -Recurse -Force "$env:USERPROFILE\.gemini\cache\agy-fast" -ErrorAction SilentlyContinue
```
