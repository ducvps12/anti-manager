# 🛡️ Nghiên Cứu Chống Bị Google Quét & Chiến Lược Anti-Ban
> **Mục tiêu**: Triển khai dịch vụ Claude CLI proxy cho khách hàng tránh bị Google phát hiện  
> **Ngày**: 12/03/2026 | **Phiên bản AG**: v4.1.29 | **Claude CLI**: v2.1.74

---

## 📊 Cơ Chế Phát Hiện Của Google

Google sử dụng **hệ thống ML (machine learning)** nhiều lớp để phát hiện lạm dụng:

| Lớp phát hiện | Google kiểm tra gì | Mức rủi ro |
|---|---|---|
| **TLS Fingerprint** | JA3/JA4 hash — nếu không khớp browser thật → nghi ngờ | 🔴 Cao |
| **HTTP Headers** | User-Agent, X-Client-Name, X-Client-Version, Machine-Id | 🔴 Cao |
| **OAuth Pattern** | Tốc độ đổi token, số account/IP, tần suất refresh | 🔴 Cao |
| **API Traffic** | Request pattern, RPM, TPM, burst detection | 🟡 Trung bình |
| **IP Reputation** | Nhiều account cùng IP, datacenter IP vs residential | 🟡 Trung bình |
| **Account Behavior** | Account mới tạo xong dùng API ngay → đáng ngờ | 🟡 Trung bình |

---

## 🔬 Phân Tích Các Lớp Bảo Vệ Hiện Tại (Antigravity v4.1.29)

### ✅ Đã có:
1. **JA3 Fingerprint Spoofing** (Chrome 123 via BoringSSL + `rquest`)
2. **Dynamic User-Agent** (đọc version thật từ local IDE)
3. **Full Header Injection** (X-Client-Name, X-Client-Version, X-Machine-Id, X-VSCode-SessionId)
4. **OAuth Clean Exchange** (xoá JA3 spoof khi đổi token, dùng UA sạch)
5. **Smart Version Fingerprint** (`max(local, remote, baseline)`)
6. **Auto 429/401 Retry + Account Rotation**
7. **403 Auto-Disable** (ngưng gửi request cho account bị ban)
8. **Atomic File Writes** (tránh corrupt data khi concurrent)

### ❌ Chưa có / Cần cải thiện:
1. **gRPC/gRPC-Web proxy** (đang test, chưa production)
2. **Residential proxy rotation** (mỗi account dùng IP khác)
3. **Request timing randomization** (giãn cách ngẫu nhiên)
4. **Account warm-up period** (account mới → chờ trước khi dùng heavy)
5. **Per-account rate limiting** (giới hạn RPM cho từng account)

---

## 🚀 Chiến Lược Mở Rộng Đề Xuất

### Tầng 1: Cấp Khách Hàng Cá Nhân (Chi phí thấp)

```
[Khách hàng] → ANTHROPIC_BASE_URL → [Antigravity Local Proxy :8045]
                                           ↓
                            [Google Account Pool] → [Google API]
```

**Thực hiện:**
- Khách cài Antigravity trên máy local
- Thêm 2-3 Google account = đủ dùng cá nhân
- Bật proxy, chạy Claude CLI trỏ vào localhost
- **Rủi ro**: Thấp nếu dùng ít account, không spam

### Tầng 2: Cấp Dịch Vụ Nhỏ (5-20 khách)

```
[Khách 1..N] → ANTHROPIC_BASE_URL → [VPS + Antigravity Docker :8045]
                                           ↓
                            [Account Pool 10-30] → [Residential Proxy → Google API]
```

**Thực hiện:**
- Deploy Antigravity Docker trên VPS
- Mở port 8045 + đặt `API_KEY` + `WEB_PASSWORD`
- Khách chỉ cần set env vars → xong
- **QUAN TRỌNG**: Dùng residential proxy để mỗi account exit từ IP khác nhau

### Tầng 3: Cấp Dịch Vụ Lớn (Scale)

```
[API Gateway / Load Balancer]
      ↓           ↓           ↓
[AG Instance 1] [AG Instance 2] [AG Instance 3]
      ↓           ↓           ↓
[Proxy Pool A]  [Proxy Pool B]  [Proxy Pool C]
      ↓           ↓           ↓
          [Google API Endpoints]
```

**Thực hiện:**
- Nhiều instance Antigravity trên nhiều VPS
- Mỗi instance quản lý 5-10 account
- Mỗi instance dùng pool proxy riêng
- Load balancer phân phối request

---

## 🔑 7 Nguyên Tắc Vàng Tránh Bị Ban

| # | Nguyên tắc | Chi tiết |
|---|---|---|
| 1 | **Giới hạn account/IP** | Tối đa 3-5 account trên 1 IP |
| 2 | **Residential proxy** | Dùng residential IP, tránh datacenter IP |
| 3 | **Request spacing** | Giãn cách 200-500ms giữa các request, tránh burst |
| 4 | **Account warm-up** | Account mới → chờ 24-48h trước khi dùng proxy heavy |
| 5 | **Tắt auto-refresh** | Tắt "Auto Refresh" + "Quota Protection" — giảm background noise |
| 6 | **Rotate account** | Không siết 1 account — luân phiên đều |
| 7 | **Dùng account switching** | Hướng an toàn nhất: chỉ switching, không proxy |

---

## 🛠️ Claude CLI Login Flow (Cách Hoạt Động)

```
Claude CLI → Kiểm tra ANTHROPIC_API_KEY + ANTHROPIC_BASE_URL
     ↓
Nếu có BASE_URL → Gửi request đến proxy (Antigravity)
     ↓
Antigravity nhận request → Protocol conversion
     ↓
Gửi đến Google API dưới dạng Gemini native request
     ↓
Google API trả response → AG convert lại Anthropic format
     ↓
Claude CLI nhận response → Hiển thị cho user
```

**Biến môi trường quan trọng:**
```bash
ANTHROPIC_API_KEY      # API key của Antigravity (ví dụ: sk-antigravity)
ANTHROPIC_BASE_URL     # URL proxy (ví dụ: http://127.0.0.1:8045)
HTTPS_PROXY            # Corporate/upstream proxy (tùy chọn)
HTTP_PROXY             # HTTP proxy (tùy chọn)
```

---

## 📦 Kịch Bản Triển Khai Cho Khách Hàng

### Quick Deploy Script (Windows khách hàng)
```powershell
# 1. Cài Claude CLI
npm install -g @anthropic-ai/claude-code

# 2. Set env vars (thay YOUR-KEY và YOUR-SERVER)
[System.Environment]::SetEnvironmentVariable("ANTHROPIC_API_KEY", "sk-YOUR-KEY", "User")
[System.Environment]::SetEnvironmentVariable("ANTHROPIC_BASE_URL", "http://YOUR-SERVER:8045", "User")

# 3. Khởi động lại terminal rồi chạy
claude
```

### Quick Deploy Script (macOS/Linux khách hàng)
```bash
# 1. Cài Claude CLI
npm install -g @anthropic-ai/claude-code

# 2. Thêm vào ~/.bashrc hoặc ~/.zshrc
echo 'export ANTHROPIC_API_KEY="sk-YOUR-KEY"' >> ~/.bashrc
echo 'export ANTHROPIC_BASE_URL="http://YOUR-SERVER:8045"' >> ~/.bashrc
source ~/.bashrc

# 3. Chạy
claude
```

---

## ⚡ Kế Hoạch Phát Triển Tiếp Theo

| Ưu tiên | Task | Mô tả |
|---|---|---|
| 🔴 P0 | **gRPC Proxy** | Theo dõi branch gRPC của Antigravity — đây là hướng đi an toàn nhất |
| 🟡 P1 | **Residential Proxy Integration** | Tích hợp dịch vụ residential proxy (Bright Data, Smartproxy) |
| 🟡 P1 | **Request Timing Randomizer** | Thêm random delay 100-500ms giữa các request |
| 🟢 P2 | **Account Health Monitor** | Dashboard theo dõi sức khoẻ từng account real-time |
| 🟢 P2 | **Auto warm-up scheduler** | Tự động warm-up account mới trước khi đưa vào pool |

---

## 📝 Kết Luận

> **Không có cách 100% tránh bị ban** khi dùng reverse proxy với Google OAuth session.  
> Tuy nhiên, bằng cách kết hợp nhiều lớp bảo vệ (JA3 spoof + header injection + residential proxy + request spacing + account rotation), xác suất bị phát hiện có thể **giảm đáng kể**.

**Hướng đi an toàn nhất theo thứ tự:**
1. 🟢 **Account Switching** (chỉ chuyển đổi, không proxy) — An toàn nhất
2. 🟡 **gRPC Proxy** (đang phát triển) — Gần giống native traffic  
3. 🟠 **REST Proxy + Full Camouflage** (hiện tại) — Được nhưng rủi ro  
4. 🔴 **REST Proxy không bảo vệ** — Chắc chắn bị ban nhanh
