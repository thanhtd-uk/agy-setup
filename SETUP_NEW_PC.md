# Hướng Dẫn Thiết Lập Google Antigravity Chuẩn Trên Máy Windows Mới

Quy trình áp dụng cấu trúc tối ưu (`ripgrep` + `agy-fast` bản đã kiểm chứng `v0.3.1` tắt tự cập nhật ngầm) sang máy Windows mới trong ~2 phút mà không để lại thư mục clone thừa.

---

## Yêu cầu trước khi cài
- Đã cài **Google Antigravity** và đăng nhập tài khoản.
- Có **Node.js `>= 20`** (`node -v`) và **Git** (`git --version`).

---

## Cách 1: Cài trực tiếp bản đã kiểm chứng `v0.3.1` (Khuyên dùng — không cần mang file `.zip`)

Mở **PowerShell** và chạy lần lượt 2 bước:

### Bước 1: Cài `ripgrep` (`rg`) để `batch_search` chạy đạt tốc độ tối đa
```powershell
winget install --id BurntSushi.ripgrep.MSVC --accept-package-agreements --accept-source-agreements
```
*(Nếu `rg` vừa mới cài, mở lại cửa sổ PowerShell mới để nhận biến môi trường `PATH`.)*

### Bước 2: Cài đúng tag `v0.3.1` của `agy-fast` + khóa `auto-update = notify`
```powershell
git clone --branch v0.3.1 --depth 1 https://github.com/INK-Q/agy-fast.git "$env:TEMP\agy-fast"
node "$env:TEMP\agy-fast\scripts\install.mjs" install --auto-update notify
Remove-Item -Recurse -Force "$env:TEMP\agy-fast"
```

---

## Cách 2: Copy offline từ máy cũ sang máy mới (Nếu không muốn kéo từ GitHub)

Chép đúng **2 mục** từ máy cũ sang cùng vị trí trên máy mới:
1. `%USERPROFILE%\.gemini\config\plugins\agy-fast`
2. `%USERPROFILE%\.gemini\cache\agy-fast\config.json` *(chứa cấu hình `"autoUpdate": "notify"`)*

> **Lưu ý quan trọng:** Tuyệt đối **không** copy đè file `%USERPROFILE%\.gemini\config\config.json` giữa 2 máy khác nhau (vì chứa đường dẫn & quyền thư mục riêng của từng máy). Chỉ cần đặt plugin vào `plugins/` là Antigravity tự nhận diện.

---

## Bước 3: Kiểm tra sức khỏe & Kích hoạt trên Antigravity App

1. Chạy kiểm tra sức khỏe plugin:
   ```powershell
   node "$env:USERPROFILE\.gemini\config\plugins\agy-fast\scripts\install.mjs" doctor
   ```
   Xác nhận các dòng sau đều xanh/OK:
   - `Node.js >= 20 (OK)`
   - `ripgrep (rg): available (fast search active)`
   - `MCP server: OK (5 tools: batch_search, apply_edits, read_slices, quick_outline, run_checks)`
   - `SessionStart: OK`
   - `Auto-update: notify`
   - `agy validate: PASSED (--strict)`

2. **Tắt và mở lại cửa sổ Antigravity** 1 lần để nạp tiến trình MCP nền.
3. Vào **Customizations → Installed** xác nhận `agy-fast` đang bật.
4. Khi mở **New Conversation** đầu tiên trên dự án thật:
   - Chọn mặc định **`Orchestrator`** (`Gemini 4 Neon Medium` hoặc `Gemini 4 Carbon Medium`).
   - Ngay lần đầu agent gọi tool `mcp_agy-fast_fast_*`, bấm **Always Allow** 1 lần để các lượt sau chạy tự động không bị ngắt nhịp.

---

## Gỡ sạch 100% khi không muốn dùng nữa
1. Vào **Customizations → Installed → `agy-fast`** bấm **Uninstall** (hoặc đóng hẳn app Antigravity rồi chạy `node "$env:USERPROFILE\.gemini\config\plugins\agy-fast\scripts\install.mjs" uninstall`).
2. Xóa nốt thư mục cache còn sót:
   ```powershell
   Remove-Item -Recurse -Force "$env:USERPROFILE\.gemini\cache\agy-fast" -ErrorAction SilentlyContinue
   ```
