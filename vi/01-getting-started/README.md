# 01 - Bắt Đầu

> Cài đặt OpenClaw và nhận phản hồi đầu tiên trong vòng 30 phút.

## Module này giải quyết gì

Module này đưa bạn từ con số 0 đến một OpenClaw Gateway hoạt động với kênh đầu tiên được kết nối. Đến cuối module, bạn sẽ chat được với AI assistant.

## Khi nào dùng module này

- Bạn chưa từng cài OpenClaw
- Bạn muốn cài đặt sạch, đã được kiểm chứng
- Bạn cần hiểu kiến trúc cơ bản

## Bắt đầu nhanh
### Yêu cầu trước

- Node.js 18+ đã cài
- npm hoặc yarn
- Một tài khoản ứng dụng chat (Telegram, Discord, WhatsApp, hoặc Slack)

### Cài đặt

```bash
# Script cài đặt chính thức (khuyến nghị)
curl -fsSL https://openclaw.ai/install.sh | bash

# Hoặc: npm install
npm install -g openclaw@latest

# Kiểm tra cài đặt
openclaw --version
```

### Onboarding và thiết lập Gateway

```bash
# Onboarding tương tác với cài đặt daemon
openclaw onboard --install-daemon

# Hoặc luồng nhanh (ít hỏi hơn)
openclaw onboard --flow quickstart
```

**Cờ onboarding:**
- `--install-daemon`: Cài đặt service hệ thống (launchd/systemd/schtasks)
- `--flow quickstart|manual`: Chọn mức độ hỏi
- `--mode local|remote`: Vị trí Gateway
- `--non-interactive`: Cho automation/scripts
- `--accept-risk`: Xác nhận đã hiểu rủi ro bảo mật

### Chạy Gateway

```bash
# Chạy gateway ở foreground (development)
openclaw gateway

# Hoặc dùng alias
openclaw gateway run

# Kiểm tra trạng thái
openclaw gateway status
openclaw gateway status --require-rpc
```

### Kết nối kênh đầu tiên

**Lựa chọn A: Telegram (Dễ nhất)**

```bash
# 1. Tạo bot với @BotFather trên Telegram
# 2. Lấy bot token
# 3. Thêm kênh
openclaw channels add --channel telegram --token YOUR_BOT_TOKEN
```

**Lựa chọn B: Discord**

```bash
# 1. Tạo Discord application tại https://discord.com/developers/applications
# 2. Bật Bot scope, lấy token
# 3. Thêm kênh
openclaw channels add --channel discord --token YOUR_BOT_TOKEN
```

### Kiểm tra mọi thứ hoạt động

Gửi tin nhắn đến bot:

```
hello
```

Bạn sẽ nhận phản hồi trong vài giây.

## Cách hoạt động bên trong

```
┌─────────────────┐     ┌──────────────┐     ┌─────────────────┐
│  Ứng dụng chat  │◄───►│   Gateway    │◄───►│  AI Assistant   │
│ (Telegram/etc)  │     │  (openclaw)  │     │  (Claude/etc)   │
└─────────────────┘     └──────────────┘     └─────────────────┘
```

Gateway là trung gian: nhận tin nhắn từ ứng dụng chat, chuyển tiếp đến AI, và gửi phản hồi lại.

## Ví dụ copy-paste

### Kiểm tra sức khỏe Gateway

```bash
openclaw gateway status
openclaw gateway logs --tail 50
```

### Liệt kê các kênh đã kết nối

```bash
openclaw channels list
openclaw channels status
```

## Lỗi thường gặp và cách khắc phục

| Vấn đề | Cách giải quyết |
|--------|-----------------|
| "command not found: openclaw" | Kiểm tra npm global bin có trong PATH không |
| Gateway không khởi động | Kiểm tra port 18789 không bị chiếm: `lsof -i :18789` |
| Bot không phản hồi | Xác minh token đúng; kiểm tra gateway logs |
| Tin nhắn chậm | Kiểm tra kết nối internet và API credits |

## Pattern nâng cao

### Chạy Gateway trên port khác

```bash
openclaw gateway --port 8080
```

### Biến môi trường

Tạo file `.env` trong `~/.openclaw/.env`:

```env
OPENCLAW_PORT=18789
OPENCLAW_LOG_LEVEL=info
OPENCLAW_PROVIDER=anthropic
```

## Module liên quan và bước tiếp theo

- Tiếp theo: [02-channels](../02-channels/) — Thêm nhiều kênh
- Tham khảo: [Gateway CLI](../10-cli-and-reference/gateway.md)
- Xử lý lỗi: [Gateway không khởi động](../TROUBLESHOOTING.md#gateway-wont-start)

---

**Thời gian hoàn thành:** 30 phút  
**Yêu cầu trước:** Không  
**Kết quả:** OpenClaw đã cài đặt với kênh đầu tiên
