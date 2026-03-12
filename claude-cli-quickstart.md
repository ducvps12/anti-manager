# ⚡ QUICK START — Claude CLI + Antigravity (5 phút)

## 1️⃣ Tải Antigravity Manager
```powershell
# Windows PowerShell
irm https://raw.githubusercontent.com/lbjlaq/Antigravity-Manager/main/install.ps1 | iex
```

## 2️⃣ Thêm tài khoản Google
`Accounts → Thêm tài khoản → OAuth → Copy link → Đăng nhập Google`

## 3️⃣ Bật Proxy
`API Proxy → Start Service`  
Proxy chạy tại: `http://127.0.0.1:8045`

## 4️⃣ Chạy Claude CLI
```powershell
$env:ANTHROPIC_API_KEY = "sk-antigravity"
$env:ANTHROPIC_BASE_URL = "http://127.0.0.1:8045"
claude
```

## 5️⃣ Đồng bộ nhanh (tùy chọn)
Vào Antigravity → tab **Đồng bộ CLI** → Card **Claude Code** → **Đồng bộ ngay**

---
📖 Hướng dẫn chi tiết: xem file `claude-cli-setup-guide.md`  
⚠️ Google đang siết chặt — tắt "Auto Refresh" và "Quota Protection" để giảm rủi ro.
