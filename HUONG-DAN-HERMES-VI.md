# Hướng dẫn Hermes Agent cho người mới ☤

> Bản này viết cho máy **hmacs-mac-mini** (đã cài sẵn). Cập nhật: 2026-06-26.
> Hermes Agent v0.17.0 — framework AI agent tự cải thiện của Nous Research (MIT license).

---

## 0. Hermes là gì (1 phút)

Hermes là một **AI agent chạy trong terminal** (giống Claude Code), nhưng có thêm:

- **Tự học (learning loop):** tự tạo skill từ việc đã làm, tự cải thiện skill khi dùng, nhớ bạn qua nhiều phiên.
- **Sống ở mọi nơi:** chat qua Terminal, hoặc qua Telegram / Discord / Slack / WhatsApp / Signal / Email (gateway).
- **Đa model:** OpenAI, Anthropic, OpenRouter, Qwen, GLM, DeepSeek, Nous Portal... đổi bằng 1 lệnh, không sửa code.
- **Cron built-in:** hẹn giờ chạy task tự động (báo cáo sáng, backup đêm...).
- **40+ tool:** đọc/ghi file, chạy terminal, search web, vision, delegate subagent.

So sánh nhanh với Claude Code: cùng là agent terminal + tool-calling, nhưng Hermes **mở (open-source)**, tự host được, đa nền tảng chat, và **gắn được vào model bất kỳ** (kể cả Qwen/DeepSeek giá rẻ).

---

## 1. Trạng thái trên máy này

Đã cài sẵn — **không cần cài lại**:

| Thành phần | Trạng thái |
|---|---|
| Hermes Agent v0.17.0 | ✅ `agents/hermes-agent/` |
| CLI `hermes` | ✅ `~/.local/bin/hermes` (đã thêm vào PATH ở `~/.zshrc`) |
| Python 3.11.15 + venv | ✅ (uv quản lý) |
| 72 skills | ✅ đã sync |
| **Model / API key** | ⚠️ **CHƯA có** → đây là việc cần làm đầu tiên |

**Nếu gõ `hermes` báo "command not found":** chạy `source ~/.zshrc` hoặc mở tab terminal mới.

---

## 2. Việc đầu tiên: gắn model (BẮT BUỘC)

Hermes chưa chạy được vì chưa có model. Có 2 đường:

### Cách A — Nous Portal (dễ nhất, 1 lệnh)
1 subscription = 300+ model (Claude, GPT, Gemini...) + sẵn web search / image / TTS:
```bash
hermes setup --portal      # login OAuth qua browser, tự set model + tool gateway
```

### Cách B — Tự mang key (bring your own key)
```bash
hermes model               # wizard chọn provider + model, dán API key
```
Chọn provider (OpenRouter / OpenAI / Anthropic / Qwen / GLM...) → dán key → chọn model → xong.

> **Quy tắc vàng:** chừng nào Hermes chưa chat được 1 lần sạch sẽ thì **đừng** bật thêm gateway/cron/voice. Cho 1 cuộc chat chạy ngon trước, rồi mới thêm tính năng.

### Trường hợp key Qwen của anh (DashScope)
Endpoint OpenAI-compatible, đã test key xác thực OK nhưng **mọi model trả `AccessDenied.Unpurchased`** — workspace Alibaba chưa kích hoạt/mua model. Cần làm bên Alibaba Model Studio (region Singapore): activate model service + bật 1 model text (vd `qwen-plus`) + add billing. Sau khi 1 model gọi được, gắn vào Hermes bằng `hermes model` → chọn "custom OpenAI-compatible endpoint" → nhập base URL `.../compatible-mode/v1` + key + tên model.

### Verify đã chạy
```bash
hermes status              # xem provider/model/key — phải hết ✗
hermes -z "say PONG"       # chạy 1 prompt nhanh, phải trả lời
```

---

## 3. Lệnh CLI cơ bản (gõ ở terminal)

| Lệnh | Tác dụng |
|---|---|
| `hermes` | Mở giao diện chat tương tác (TUI) |
| `hermes -c` | Tiếp tục **phiên gần nhất** (giữ nguyên lịch sử) |
| `hermes -r "tên phiên"` | Resume phiên theo tên |
| `hermes -z "câu hỏi"` | Chạy **1 prompt** rồi thoát (one-shot, hợp cho script) |
| `hermes -m <model>` | Chạy với model chỉ định cho phiên này |
| `hermes model` | Đổi provider/model mặc định |
| `hermes tools` | Bật/tắt từng tool |
| `hermes status` | Trạng thái cấu hình (model, key, tool) |
| `hermes doctor` | Chẩn đoán lỗi cài đặt |
| `hermes update` | Cập nhật Hermes lên bản mới |
| `hermes setup` | Wizard cấu hình toàn bộ |
| `hermes --help` | Xem toàn bộ lệnh con |

Các nhóm lệnh nâng cao: `hermes gateway` (chat đa nền tảng), `hermes cron` (hẹn giờ), `hermes skills` (quản lý skill), `hermes memory` (bộ nhớ), `hermes mcp` (gắn MCP server), `hermes backup` / `hermes import` (sao lưu/khôi phục).

---

## 4. Trong cuộc chat — slash command & phím tắt

Khi đã ở trong `hermes` (giao diện chat), gõ:

| Slash command | Tác dụng |
|---|---|
| `/help` | Danh sách lệnh |
| `/new` hoặc `/reset` | Bắt đầu hội thoại mới (xóa context) |
| `/model [provider:model]` | Đổi model giữa chừng |
| `/skills` | Duyệt skill; hoặc gõ thẳng `/<tên-skill>` để chạy |
| `/tools` | Xem/bật tool |
| `/usage` | Xem token đã dùng |
| `/compress` | Nén context khi hội thoại dài |
| `/insights [--days N]` | Thống kê sử dụng |
| `/sessions` | Danh sách phiên cũ |
| `/personality [tên]` | Đổi tính cách agent |
| `/retry`, `/undo` | Thử lại / hoàn tác lượt cuối |
| `/agents` | Quản lý subagent |
| `/voice` | Chế độ voice |
| `/status` | Trạng thái phiên |

**Phím tắt hữu ích:**
- `Alt+Enter` / `Ctrl+J` / `Shift+Enter` — xuống dòng không gửi
- `Ctrl+C` (1 lần) — ngắt giữa chừng để đổi hướng; (2 lần trong 2s) — thoát
- `Ctrl+V` — dán ảnh từ clipboard (agent dùng vision để đọc ảnh/screenshot)
- Dán code/traceback nhiều dòng — tự gộp thành 1 message, không gửi từng dòng

---

## 5. Các tính năng chính (khi đã quen)

- **Skills (kỹ năng):** quy trình đóng gói sẵn. Gõ `/skills` để duyệt, `/<tên>` để chạy. Hermes còn **tự tạo skill mới** sau khi làm xong task phức tạp.
- **Memory (bộ nhớ):** Hermes nhớ bạn qua các phiên (file `MEMORY.md` / `USER.md`). Quản lý bằng `hermes memory`.
- **Context file `AGENTS.md`:** đặt file `AGENTS.md` trong thư mục dự án → Hermes đọc tự động mỗi phiên. Để chỉ dẫn lặp lại ("dùng pytest", "API ở /api/v2", "tab không phải space").
- **Gateway (chat đa nền tảng):** `hermes gateway setup` rồi `hermes gateway start` → nhắn agent qua Telegram/Discord/Slack...
- **Cron (hẹn giờ):** `hermes cron` → task tự động chạy nền (báo cáo, backup) bằng ngôn ngữ tự nhiên.
- **MCP:** `hermes mcp` → gắn MCP server bất kỳ để mở rộng tool.
- **Subagent:** Hermes spawn subagent chạy song song nhiều việc.

---

## 6. Mẹo dùng hiệu quả

1. **Nói cụ thể.** "Sửa TypeError ở `api/handlers.py` dòng 47 — `process_request()` nhận `None` từ `parse_body()`" tốt hơn nhiều "sửa code đi".
2. **Đưa context trước.** Dán đường dẫn file, lỗi traceback ngay trong message đầu — đỡ phải hỏi đi hỏi lại.
3. **Để agent tự dùng tool.** Nói "tìm và sửa test đang fail" thay vì chỉ tay từng bước.
4. **Có skill thì dùng skill.** Trước khi viết prompt dài, `/skills` xem có sẵn không.
5. **Hội thoại dài thì `/compress`** để khỏi tốn token và tránh "quên" đầu hội thoại.
6. **Resume bằng `hermes -c`** thay vì kể lại từ đầu.

---

## 7. Khi gặp lỗi (troubleshooting)

| Triệu chứng | Cách xử lý |
|---|---|
| `hermes: command not found` | `source ~/.zshrc` hoặc mở tab mới |
| Không trả lời / lỗi auth | `hermes status` xem key; `hermes doctor` chẩn đoán |
| `AccessDenied` / model lỗi | Model chưa được provider bật — đổi model `hermes model` hoặc bật bên provider |
| Cấu hình rối | `hermes dump` (xuất summary để debug) |
| Muốn làm lại từ đầu | `hermes setup` |
| Cập nhật | `hermes update` |

---

## 8. Đường tới thành thạo (gợi ý thứ tự)

1. Gắn model (`hermes setup --portal` hoặc `hermes model`) → chat thử 1 câu.
2. Dùng CLI hằng ngày: `hermes`, `hermes -c`, `/model`, `/skills`.
3. Tạo `AGENTS.md` cho dự án hay làm.
4. Bật gateway nếu muốn chat qua Telegram/Slack.
5. Cron hóa việc lặp lại.
6. Gắn MCP / tạo skill riêng.

**Doc gốc đầy đủ:** https://hermes-agent.nousresearch.com/docs/
**Doc trong repo:** `agents/hermes-agent/website/docs/` (CLI, TUI, configuration, skills, memory, security...).
