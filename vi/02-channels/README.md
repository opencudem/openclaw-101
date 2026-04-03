# 02 - Channels

> Kết nối OpenClaw với nơi bạn đang chat: WhatsApp, Telegram, Discord, và Slack.

## Module này giải quyết gì

Tính năng đặc trưng của OpenClaw là gặp gỡ bạn ở nơi bạn đang ở. Module này hướng dẫn cách kết nối nhiều nền tảng nhắn tin để bạn có thể nói chuyện với trợ lý từ bất kỳ thiết bị nào.

## Khi nào dùng module này

- Bạn muốn dùng OpenClaw từ điện thoại
- Team bạn dùng nhiều nền tảng chat
- Bạn muốn hành vi riêng cho từng kênh

## Các kênh được hỗ trợ

| Kênh | Phù hợp nhất cho | Độ khó |
|------|------------------|--------|
| **Telegram** | Bắt đầu nhanh, truy cập mobile | 🟢 Dễ |
| **Discord** | Cộng tác team, cộng đồng | 🟢 Dễ |
| **WhatsApp** | Cá nhân, danh bạ hiện có | 🟡 Trung bình |
| **Slack** | Tích hợp nơi làm việc | 🟡 Trung bình |
| **iMessage** | Hệ sinh thái Apple | 🟡 Trung bình |
| **Signal** | Người dùng quan tâm riêng tư | 🟡 Trung bình |

## Tổng quan lệnh Channels

Tất cả quản lý kênh dùng lệnh `channels` (số nhiều):

```bash
# Liệt kê các kênh đã kết nối
openclaw channels list

# Kiểm tra khả năng của kênh
openclaw channels capabilities

# Thêm kênh (ví dụ cho từng nền tảng bên dưới)
openclaw channels add --channel <name> --token <token>

# Xóa kênh
openclaw channels remove --channel <name> --delete

# Đăng nhập tương tác (cho ghép đôi QR)
openclaw channels login --channel <name>

# Đăng xuất
openclaw channels logout --channel <name>

# Giải quyết tên thành ID
openclaw channels resolve --channel slack "#general" "@jane"
```

## Bắt đầu nhanh theo từng kênh

### Telegram

```bash
# 1. Nhắn tin @BotFather trên Telegram
# 2. Gửi /newbot và làm theo hướng dẫn
# 3. Copy bot token (dạng 123456789:ABCdefGHIjklMNOpqrsTUVwxyz)

# 4. Thêm vào OpenClaw
openclaw channels add --channel telegram --token YOUR_BOT_TOKEN

# 5. Cấu hình chính sách DM (tùy chọn)
# Sửa ~/.openclaw/config.yaml hoặc dùng biến môi trường
# 6. Bắt đầu chat với bot
```

**Chính sách DM cho Telegram:**

Telegram hỗ trợ tin nhắn trực tiếp. Cấu hình ai có thể nhắn tin cho agent của bạn:

```yaml
channels:
  telegram:
    botToken: "${TELEGRAM_BOT_TOKEN}"
    dmPolicy: pairing  # Tùy chọn: pairing, allowlist, open, disabled
```

**Các tùy chọn chính sách DM:**
- `pairing` (mặc định): Yêu cầu mã ghép đôi một lần để phê duyệt
- `allowlist`: Chỉ người trong danh sách `allowFrom` có thể nhắn tin
- `open`: Cho phép tất cả inbound (yêu cầu `allowFrom: ["*"]`)
- `disabled`: Chặn tất cả tin nhắn trực tiếp

**Lệnh ghép đôi:**

Khi dùng `dmPolicy: pairing`, người dùng phải ghép đôi trước khi nhắn tin:

```bash
# Liệt kê các yêu cầu ghép đôi đang chờ
openclaw pairing list telegram

# Phê duyệt yêu cầu ghép đôi
openclaw pairing approve telegram <CODE>

# Mã ghép đôi hết hạn sau 1 giờ
```

**Mẹo chuyên nghiệp:** Tạo lệnh bot với `/setcommands` trong BotFather để truy cập nhanh.

---

### Discord

```bash
# 1. Vào https://discord.com/developers/applications
# 2. Tạo New Application → Bot → Add Bot
# 3. Copy Bot Token
# 4. Bật Message Content Intent (bắt buộc để bot đọc tin nhắn)
# 5. Tạo OAuth2 URL với bot scope và quyền gửi tin nhắn
# 6. Mời bot vào server
# 7. Thêm vào OpenClaw
openclaw channels add --channel discord --token YOUR_BOT_TOKEN
```

**Thiết lập Discord bắt buộc:**

1. **Privileged Intents** (bắt buộc):
   - Bật `MESSAGE CONTENT INTENT` trong Bot → Privileged Gateway Intents
   - Không có cái này, bot không đọc được nội dung tin nhắn

2. **OAuth2 Scopes** cho URL mời:
   - `bot` scope
   - Quyền `Send Messages`
   - Quyền `Read Message History`

**Chính sách DM cho Discord:**

Discord hỗ trợ DM. Cấu hình kiểm soát truy cập:

```yaml
channels:
  discord:
    botToken: "${DISCORD_BOT_TOKEN}"
    dmPolicy: allowlist
    allowFrom:
      - "discord:123456789"  # Discord user ID của bạn
```

**Mẹo chuyên nghiệp:** Tạo kênh riêng chỉ cho bạn và bot, hoặc dùng DM để giao tiếp an toàn.

---

### WhatsApp

WhatsApp yêu cầu ghép đôi mã QR (quá trình tương tác):

```bash
# 1. Thêm kênh WhatsApp (không cần token ban đầu)
openclaw channels add --channel whatsapp

# 2. Đăng nhập để bắt đầu ghép đôi QR
openclaw channels login --channel whatsapp
# Mã QR sẽ xuất hiện - quét bằng ứng dụng WhatsApp

# 3. Sau khi ghép đôi, session của bạn được lưu
```

**Cấu hình WhatsApp:**

```yaml
channels:
  whatsapp:
    allowFrom:
      - "+14155552671"  # Số điện thoại bạn ở định dạng E.164
    groupPolicy: allowlist  # Tùy chọn: allowlist, open, disabled
    groupAllowFrom:
      - "group:123456789"  # ID nhóm cụ thể
```

**Lưu ý quan trọng về WhatsApp:**

- **Định dạng E.164**: Số điện thoại phải có `+` và mã quốc gia (ví dụ: `+84123456789`)
- **Ghép đôi QR**: Chạy `openclaw channels login --channel whatsapp` để ghép đôi lại nếu session hết hạn
- **Giới hạn tốc độ**: WhatsApp có giới hạn tốc độ nghiêm ngặt. Đừng spam.
- **Số điện thoại riêng**: Dùng số thứ hai, không phải số chính

**Hỗ trợ nhóm WhatsApp:**

```yaml
# Cho phép các nhóm cụ thể
channels:
  whatsapp:
    groupPolicy: allowlist
    groupAllowFrom:
      - "group:123456789@g.us"
```

---

### Slack

Slack hỗ trợ hai chế độ: **Socket Mode** (mặc định, khuyến nghị) và **HTTP Mode**.

```bash
# 1. Vào https://api.slack.com/apps
# 2. Tạo New App → From scratch
# 3. Thêm features: Bots, Permissions
# 4. Trong OAuth & Permissions, thêm Bot Token Scopes:
#    - chat:write
#    - im:history
#    - im:read
#    - channels:history (để truy cập kênh)
# 5. Cài đặt vào workspace, copy Bot User OAuth Token (bắt đầu bằng xoxb-)
# 6. Cho Socket Mode, cũng cần App-Level Token (bắt đầu bằng xapp-)

# 7. Thêm vào OpenClaw
openclaw channels add --channel slack --token xoxb-YOUR-BOT-TOKEN
```

**Cấu hình Slack:**

```yaml
channels:
  slack:
    botToken: "${SLACK_BOT_TOKEN}"
    appToken: "${SLACK_APP_TOKEN}"  # Bắt buộc cho Socket Mode
    mode: socket  # hoặc 'http' cho webhook mode
    dmPolicy: allowlist
    allowFrom:
      - "slack:U123456789"  # Slack user ID của bạn
```

**Các Bot Token Scopes bắt buộc:**
- `chat:write` - Gửi tin nhắn
- `im:history` - Truy cập lịch sử DM
- `im:read` - Đọc DM
- `channels:history` - Truy cập lịch sử kênh (tùy chọn)

**Socket Mode vs HTTP Mode:**

| Chế độ | Phù hợp nhất cho | Token cần thiết |
|--------|------------------|-----------------|
| **Socket Mode** | Phát triển local, sau firewall | Bot Token + App Token |
| **HTTP Mode** | Server có public URL | Bot Token only |

**Mẹo chuyên nghiệp:** Bot cần được mời vào các kênh trước khi có thể thấy tin nhắn. Dùng `/invite @YourBot` trong mỗi kênh.

---

## Thiết lập đa kênh

Bạn có thể kết nối nhiều kênh cùng lúc:

```bash
# Thêm tất cả kênh của bạn
openclaw channels add --channel telegram --token TELEGRAM_TOKEN
openclaw channels add --channel discord --token DISCORD_TOKEN
openclaw channels add --channel slack --token xoxb-SLACK_TOKEN

# Liệt kê tất cả kênh đã kết nối
openclaw channels list

# Kiểm tra từng kênh hỗ trợ gì
openclaw channels capabilities
```

Tin nhắn gửi đến bất kỳ kênh nào cũng được chuyển đến cùng một agent. Agent biết bạn đang dùng kênh nào.

---

## Chính sách DM (Bảo mật tin nhắn trực tiếp)

Kiểm soát ai có thể gửi tin nhắn trực tiếp đến agent của bạn:

### Các loại chính sách

| Chính sách | Hành vi | Trường hợp sử dụng |
|------------|---------|-------------------|
| **pairing** | Phê duyệt một lần qua mã ghép đôi | Truy cập an toàn, có kiểm soát |
| **allowlist** | Chỉ ID được chỉ định có thể nhắn tin | Chỉ người quen |
| **open** | Ai cũng có thể nhắn tin (cẩn thận) | Bot công khai |
| **disabled** | Chặn tất cả DM | Chỉ dùng kênh |

### Ví dụ cấu hình

**Chế độ Pairing (an toàn nhất):**
```yaml
channels:
  telegram:
    botToken: "${TELEGRAM_BOT_TOKEN}"
    dmPolicy: pairing
```

Người dùng phải yêu cầu ghép đôi:
```
User: Can I talk to you?
Bot: Pairing required. Reply with: pair ABC123
User: pair ABC123
# Admin chạy: openclaw pairing approve telegram ABC123
```

**Chế độ Allowlist:**
```yaml
channels:
  discord:
    botToken: "${DISCORD_BOT_TOKEN}"
    dmPolicy: allowlist
    allowFrom:
      - "discord:123456789"      # Discord ID của bạn
      - "discord:987654321"      # Discord ID của bạn bè
```

**Chế độ Open (bot công khai):**
```yaml
channels:
  telegram:
    botToken: "${TELEGRAM_BOT_TOKEN}"
    dmPolicy: open
    allowFrom: ["*"]  # Bắt buộc khi dùng 'open'
```

---

## Lệnh ghép đôi

Cho các kênh dùng `dmPolicy: pairing`:

```bash
# Liệt kê tất cả yêu cầu ghép đôi đang chờ
openclaw pairing list

# Liệt kê yêu cầu cho kênh cụ thể
openclaw pairing list telegram

# Phê duyệt yêu cầu ghép đôi
openclaw pairing approve telegram ABC123

# Phê duyệt cho Discord
openclaw pairing approve discord XYZ789
```

**Luồng ghép đôi:**
1. Người dùng nhắn tin cho bot và yêu cầu ghép đôi
2. Bot cung cấp mã ghép đôi (ví dụ: `ABC123`)
3. Admin chạy `openclaw pairing approve <channel> <code>`
4. Người dùng có thể nhắn tin cho bot tự do
5. Mã ghép đôi hết hạn sau 1 giờ

---

## Hành vi riêng cho từng kênh

### Phản hồi khác nhau theo kênh

Cấu hình trong cài đặt OpenClaw:

```yaml
# ~/.openclaw/config.yaml
channels:
  telegram:
    personality: "casual, emoji-friendly"
  discord:
    personality: "technical, markdown-friendly"
  slack:
    personality: "professional, concise"
```

### Hạn chế kênh

Giới hạn kênh nào có thể thực thi lệnh nhạy cảm:

```yaml
security:
  allowed_channels:
    - "telegram:12345678"   # Chat riêng của bạn
    - "discord:98765432"    # Server riêng của bạn
```

---

## Ví dụ copy-paste

### Lấy kênh ID

```bash
# Gửi tin nhắn từ kênh
# Kiểm tra logs để tìm channel identifier
openclaw gateway logs | grep "channel"

# Hoặc dùng lệnh resolve
openclaw channels resolve --channel slack "#general"
```

### Xóa kênh

```bash
# Xóa Telegram
openclaw channels remove --channel telegram --delete

# Xóa Discord
openclaw channels remove --channel discord --delete

# Cờ --delete xóa kênh khỏi config
```

### Kiểm tra trạng thái kênh

```bash
# Liệt kê tất cả kênh và trạng thái
openclaw channels list

# Kiểm tra kênh hỗ trợ gì
openclaw channels capabilities --channel telegram
```

---

## Lỗi thường gặp và xử lý

| Vấn đề | Cách giải quyết |
|--------|-----------------|
| Bot Telegram không phản hồi | Kiểm tra bạn có nhắn đúng bot không; xác minh token; kiểm tra `openclaw gateway logs` |
| Bot Discord offline | Kiểm tra Message Content Intent đã bật trong Bot settings chưa |
| Mã QR WhatsApp lỗi | Dùng số điện thoại riêng; xóa ghép đôi cũ bằng `openclaw channels logout --channel whatsapp` |
| Bot Slack không DM được | Bot cần được mời vào kênh trước; kiểm tra scopes có `im:read` không |
| "Invalid command: channel" | Dùng `channels` (số nhiều), không phải `channel` |
| Mã ghép đôi hết hạn | Mã hết hạn sau 1 giờ; người dùng phải yêu cầu mã mới |
| DMs bị chặn | Kiểm tra cài đặt `dmPolicy` và danh sách `allowFrom` |

---

## Pattern nâng cao

### Chế độ webhook (cho server)

Thay vì polling, nhận webhook:

```yaml
channels:
  telegram:
    mode: webhook
    webhook_url: https://your-server.com/webhook/telegram
```

### Quy tắc định tuyến kênh

Định tuyến loại tin nhắn khác nhau đến kênh khác nhau:

```yaml
routing:
  alerts:
    channels: [slack, discord]
  personal:
    channels: [telegram]
```

### Dùng SecretRef cho token

Lưu token an toàn thay vì trong file config:

```yaml
channels:
  discord:
    botToken:
      secretRef: discord-bot-token  # Tham chiếu biến môi trường OPENCLAW_SECRET_discord_bot_token
```

---

## Module liên quan và bước tiếp theo

- Trước đó: [01-getting-started](../01-getting-started/)
- Tiếp theo: [03-memory](../03-memory/) — Cá nhân hóa với bộ nhớ
- Liên quan: [Xử lý lỗi kênh](../TROUBLESHOOTING.md#channels)

---

**Thời gian hoàn thành:** 45 phút  
**Yêu cầu trước:** [01-getting-started](../01-getting-started/)  
**Kết quả:** Nhiều kênh được kết nối, nhắn tin từ mọi nền tảng
