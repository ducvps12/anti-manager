# 📘 Hướng Dẫn Cài Đặt Claude CLI + Antigravity Manager
> **Phiên bản**: v4.1.29 | **Cập nhật**: 12/03/2026  
> Gửi khách hàng — Tự cài đặt nhanh trong 5 phút

---

## ⚠️ CẢNH BÁO QUAN TRỌNG — Google Đang Siết Chặt Phong Toả

> [!CAUTION]
> **Từ v4.1.28+**, Google đã tăng cường kiểm soát rủi ro. Việc sử dụng công cụ bên thứ 3 (OAuth proxy) **có thể dẫn đến tài khoản bị đình chỉ hoặc chấm dứt**.
> - Nếu tài khoản bị khoá nhầm, gửi đơn kháng cáo tại: [Google Appeal Form](https://forms.gle/hGzM9MEUv2azZsrb9)
> - Theo dõi cập nhật mới nhất: [Telegram Channel](https://t.me/AntigravityManager)

---

## 📋 Yêu Cầu Trước Khi Bắt Đầu

| Yêu cầu | Chi tiết |
|----------|----------|
| **Hệ điều hành** | Windows 10/11, macOS, hoặc Linux |
| **Node.js** | v18+ (để cài Claude CLI) |
| **Tài khoản Google** | Với Google One AI Premium hoặc Gemini Code Assist |
| **Tải Antigravity** | [GitHub Releases](https://github.com/lbjlaq/Antigravity-Manager/releases) |

---

## 🚀 Bước 1: Cài Đặt Antigravity Manager

### Windows (PowerShell)
```powershell
irm https://raw.githubusercontent.com/lbjlaq/Antigravity-Manager/main/install.ps1 | iex
```

### macOS / Linux
```bash
curl -fsSL https://raw.githubusercontent.com/lbjlaq/Antigravity-Manager/v4.1.29/install.sh | bash
```

### Hoặc tải file thủ công
- **Windows**: `.msi` hoặc `.zip` portable  
- **macOS**: `.dmg`  
- **Linux**: `.deb` / `.AppImage`

> macOS báo "App bị hỏng"? Chạy: `sudo xattr -rd com.apple.quarantine "/Applications/Antigravity Tools.app"`

---

## 🔐 Bước 2: Thêm Tài Khoản Google (OAuth)

1. Mở Antigravity → **Accounts** → **Thêm tài khoản** → **OAuth**
2. Hệ thống sẽ tạo sẵn **link xác thực** — copy link đó
3. Mở link trong trình duyệt → **đăng nhập Google** → cho phép quyền truy cập
4. Trình duyệt sẽ chuyển đến trang callback hiển thị **"✅ Đã xác thực thành công!"**
5. Quay lại ứng dụng — tài khoản được lưu tự động

> **Lưu ý**: Luôn dùng link mới nhất từ popup. Nếu app không chạy khi auth, trình duyệt sẽ báo `localhost refused connection`.

---

## ⚡ Bước 3: Bật API Proxy

1. Vào tab **"API Phản Đại"** (API Proxy)
2. Nhấn nút **Bật dịch vụ** (Start Service)
3. Proxy sẽ chạy tại: `http://127.0.0.1:8045`

### Các endpoint hỗ trợ:
| Định dạng | Endpoint | Dùng cho |
|-----------|----------|----------|
| **Anthropic** | `/v1/messages` | Claude Code CLI |
| **OpenAI** | `/v1/chat/completions` | Cherry Studio, NextChat, v.v. |
| **Gemini** | Google SDK format | Gemini CLI |

---

## 🔧 Bước 4: Cài Đặt Claude Code CLI

### Cài Claude CLI qua npm
```bash
npm install -g @anthropic-ai/claude-code
```

### Cấu hình kết nối với Antigravity Proxy
```bash
# Windows (PowerShell)
$env:ANTHROPIC_API_KEY = "sk-antigravity"
$env:ANTHROPIC_BASE_URL = "http://127.0.0.1:8045"
claude

# macOS / Linux (Bash/Zsh)
export ANTHROPIC_API_KEY="sk-antigravity"
export ANTHROPIC_BASE_URL="http://127.0.0.1:8045"
claude
```

### Hoặc: Đồng bộ tự động từ Antigravity
1. Trong Antigravity, vào tab **"Đồng bộ CLI"**
2. Tìm card **"Cấu hình Claude Code"**
3. Nhấn **"Đồng bộ ngay"** → Tự động thiết lập env vars

---

## 🛡️ Bước 5: Cấu Hình Chống Phát Hiện (Anti-Detection)

### Tính năng bảo vệ đã tích hợp sẵn trong Antigravity:

| Tính năng | Mô tả |
|-----------|-------|
| **JA3 Fingerprint Spoofing** | Giả lập TLS fingerprint Chrome 123 (BoringSSL) |
| **Dynamic User-Agent** | Tự động đọc version thật của IDE để build UA |
| **Full Header Injection** | Bổ sung `X-Client-Name`, `X-Client-Version`, `X-Machine-Id`, `X-VSCode-SessionId` |
| **OAuth Clean Exchange** | Khi đổi token, xoá JA3 giả lập → request sạch, giống native client |
| **Smart Version Fingerprint** | Luôn dùng phiên bản mới nhất: `max(local, remote, baseline)` |
| **429 / 401 Auto-Retry** | Tự động xoay vòng tài khoản khi bị rate limit |
| **403 Auto-Disable** | Tự động tắt tài khoản bị cấm, không gửi request thừa |

### ⚙️ Cài đặt khuyến nghị để tránh bị quét:

1. **Tắt "Bảo vệ Quota"** và **"Tự động làm mới nền"** → giảm request thừa
2. **Bật User-Agent Override** trong Settings
3. **Không dùng quá nhiều tài khoản** trên cùng IP
4. Sử dụng **proxy pool** nếu có nhiều tài khoản
5. **Giãn cách request** — tránh burst quá nhiều request cùng lúc
6. Cân nhắc dùng **gRPC protocol** (đang test) thay vì 2api thông thường

> [!WARNING]
> Tính năng 2api (reverse proxy) có xác suất bị Google phát hiện cao hơn.  
> Hướng đi an toàn hơn: **gRPC / gRPC-Web proxy** (đang trong giai đoạn thử nghiệm, theo dõi TG channel để cập nhật).

---

## 🔗 Các CLI Khác Được Hỗ Trợ

### Gemini CLI
```bash
# Đồng bộ từ Antigravity → Card "Cấu hình Gemini CLI" → Đồng bộ ngay
```

### OpenCode
```bash
# Antigravity → Card "OpenCode Sync" → Sync
# Tự tạo file ~/.config/opencode/opencode.json
opencode run "test" --model antigravity-manager/claude-sonnet-4-5-thinking
```

### Codex AI / Droid
```bash
# Tương tự — dùng card tương ứng trong Antigravity
```

### Python SDK
```python
import openai

client = openai.OpenAI(
    api_key="sk-antigravity",
    base_url="http://127.0.0.1:8045/v1"
)

response = client.chat.completions.create(
    model="gemini-3-flash",
    messages=[{"role": "user", "content": "Xin chào!"}]
)
print(response.choices[0].message.content)
```

---

## 🐳 Triển Khai VPS/Server (Docker)

Dành cho việc chạy proxy trên server để khách truy cập từ xa:

```bash
docker run -d --name antigravity-manager \
  -p 8045:8045 \
  -e API_KEY=sk-your-custom-key \
  -e WEB_PASSWORD=admin-password \
  -e ABV_MAX_BODY_SIZE=104857600 \
  -v ~/.antigravity_tools:/root/.antigravity_tools \
  lbjlaq/antigravity-manager:latest
```

**Khách hàng chỉ cần:**
```bash
export ANTHROPIC_API_KEY="sk-your-custom-key"
export ANTHROPIC_BASE_URL="http://your-server-ip:8045"
claude
```

---

## 🔍 Xử Lý Sự Cố

| Lỗi | Giải pháp |
|------|----------|
| `403 Forbidden` | Kiểm tra email có link xác minh Google không. Tài khoản có thể cần verify. Chờ một lúc rồi thử lại. |
| `429 Too Many Requests` | Hệ thống tự xoay tài khoản. Nếu tất cả tài khoản hết quota, chờ reset. |
| `400 INVALID_ARGUMENT` | Cập nhật Antigravity lên phiên bản mới nhất. Kiểm tra model mapping. |
| `404 Not Found` | Kiểm tra project ID. Thử xoá và thêm lại tài khoản. |
| Tài khoản bị ban | Gửi kháng cáo: [Google Appeal](https://forms.gle/hGzM9MEUv2azZsrb9). Tránh dùng OAuth proxy, chuyển sang chức năng "chuyển đổi tài khoản" thay thế. |

---

## 📢 Liên Hệ & Cập Nhật

- **Telegram**: [t.me/AntigravityManager](https://t.me/AntigravityManager)
- **GitHub**: [github.com/lbjlaq/Antigravity-Manager](https://github.com/lbjlaq/Antigravity-Manager)
- **Phiên bản hiện tại**: v4.1.29

---

> **Lưu ý cuối cùng**: Dự án này sử dụng session Google để chuyển đổi thành API. Đây là hành vi vi phạm TOS của Google. Sử dụng có trách nhiệm và chấp nhận rủi ro. Khuyến nghị: chỉ dùng tính năng **chuyển đổi tài khoản (account switching)** thay vì reverse proxy nếu muốn an toàn nhất.
