# iMessage qua BlueBubbles — Hướng Dẫn Thiết Lập Đầy Đủ

> Kết nối OpenClaw với iMessage bằng máy chủ BlueBubbles trên macOS. Gửi và nhận iMessage, thả cảm xúc, đính kèm tệp, và tin nhắn nhóm qua OpenClaw agent của bạn.

---

## Tổng Quan

**BlueBubbles** là máy chủ mã nguồn mở trên macOS, giúp iMessage được truy cập qua REST API. OpenClaw kết nối để:

- ✅ Gửi/nhận iMessage (cá nhân và nhóm)
- ✅ Thả cảm xúc (👍❤️😂 etc.)
- ✅ Chỉnh sửa và thu hồi tin nhắn (macOS 13+)
- ✅ Đính kèm tệp và ghi âm thoại
- ✅ Quản lý nhóm (thêm/xóa thành viên)
- ✅ Biên nhận đã đọc và chỉ báo đang nhập

**Trạng thái**: Sẵn sàng sản xuất | **Yêu cầu**: Mac vật lý chạy macOS 12+

---

## ⚠️ Yêu Cầu Quan Trọng

**Mac là BẮT BUỘC.** iMessage là hạ tầng độc quyền của Apple. Bạn cần:

- Mac chạy macOS Sequoia (15) hoặc mới hơn (macOS 26 Tahoe hỗ trợ với lưu ý)
- Messages.app đã đăng nhập Apple ID
- Mac phải bật nguồn và kết nối internet liên tục
- Không có giải pháp Docker — BlueBubbles chạy trực tiếp trên macOS

---

## Bước 1: Cài Đặt Máy Chủ BlueBubbles

### 1.1 Tải Và Cài Đặt

1. Truy cập <https://bluebubbles.app/install>
2. Tải file `.dmg` mới nhất cho macOS
3. Mở DMG và kéo BlueBubbles vào Applications
4. Khởi chạy BlueBubbles từ Applications

### 1.2 Cấp Quyền cho macOS

Khi khởi chạy lần đầu, macOS sẽ yêu cầu nhiều quyền. **Cấp tất cả**:

| Quyền | Tại Sao Cần |
|-------|-------------|
| **Automation** | Điều khiển Messages.app để gửi/nhận tin nhắn |
| **Full Disk Access** | Đọc cơ sở dữ liệu Messages (`chat.db`) |
| **Accessibility** | Tương tác với UI Messages cho tính năng nâng cao |

Vào **System Settings → Privacy & Security** để xác nhận tất cả quyền đã được cấp.

### 1.3 Hoàn Thành Trình Thuật Dẫn

Trình thuật dẫn BlueBubbles sẽ hướng dẫn bạn qua:

1. **Tài khoản iMessage** — Dùng mặc định (Apple ID đã đăng nhập)
2. **Mật khẩu máy chủ** — Đặt mật khẩu mạnh (bạn sẽ cần cho OpenClaw)
3. **Web API** — Bật tính năng này (bắt buộc cho OpenClaw)
4. **Private API** — Bật để có thả cảm xúc, chỉnh sửa, thu hồi

### 1.4 Kiểm Tra Máy Chủ Đang Chạy

Mở Terminal trên Mac và kiểm tra:

```bash
curl http://localhost:1234/api/v1/ping
```

Kết quả mong đợi:
```json
{"status":"ok","timestamp":1715530507}
```

Nếu có phản hồi, BlueBubbles đang chạy. Ghi nhớ cổng (mặc định: 1234).

---

## Bước 2: Truy Cập Mạng

OpenClaw cần kết nối đến Mac của bạn. Chọn cách thiết lập:

### Lựa Chọn A: Cùng Máy (OpenClaw + BlueBubbles trên cùng Mac)

Dùng `localhost` — thiết lập đơn giản nhất.

### Lựa Chọn B: Máy Khác Nhau (OpenClaw trên VPS, BlueBubbles trên Mac)

Mac của bạn cần được truy cập từ máy chủ OpenClaw:

| Phương Pháp | Thiết Lập | Phù Hợp Với |
|--------|-------|----------|
| **Tailscale** | Cài Tailscale trên cả hai, dùng Tailscale IP | Bảo mật nhất, khuyến nghị |
| **ngrok** | `ngrok http 1234` trên Mac, dùng ngrok URL | Kiểm tra nhanh |
| **Public IP** | Mở port 1234 trên router | Máy chủ tại nhà có IP tĩnh |

**Khuyến nghị**: Tailscale. Tạo VPN mesh bảo mật giữa các thiết bị.

---

## Bước 3: Cấu Hình OpenClaw

### 3.1 Thiết Lập Tương Tác (Khuyến Nghị)

Chạy trình thuật dẫn:

```bash
openclaw onboard
```

Chọn **BlueBubbles** khi được hỏi. Bạn cần:
- Server URL (vd: `http://192.168.1.100:1234` hoặc `http://mac-mini.tailnet.ts.net:1234`)
- Mật khẩu (từ BlueBubbles settings)

### 3.2 Cấu Hình Thủ Công

Chỉnh sửa `~/.openclaw/openclaw.json` hoặc `config.json5`:

```json5
{
  channels: {
    bluebubbles: {
      enabled: true,
      serverUrl: "http://192.168.1.100:1234",  // IP Mac hoặc địa chỉ Tailscale
      password: "your-strong-password-here",  // Từ BlueBubbles settings
      webhookPath: "/bluebubbles-webhook",
      dmPolicy: "pairing",  // Yêu cầu phê duyệt cho DM mới
      sendReadReceipts: true,
      actions: {
        reactions: true,     // Thả cảm xúc (👍❤️😂)
        edit: true,          // Chỉnh sửa tin nhắn (macOS 13+)
        unsend: true,        // Thu hồi tin nhắn (macOS 13+)
        reply: true,         // Trả lời theo chuỗi
        sendWithEffect: true, // Hiệu ứng iMessage (slam, loud, gentle)
        renameGroup: true,   // Đổi tên nhóm
        setGroupIcon: true,  // Đặt ảnh nhóm
        addParticipant: true,
        removeParticipant: true,
        leaveGroup: true,
        sendAttachment: true, // Gửi tệp/hình ảnh
      },
    },
  },
}
```

### 3.3 Thêm Kênh Qua CLI

```bash
openclaw channels add bluebubbles --http-url http://192.168.1.100:1234 --password "your-password"
```

---

## Bước 4: Cấu Hình Webhook

iMessage đến qua webhook. Bạn phải cho BlueBubbles biết gửi đến đâu.

### 4.1 Lấy Gateway URL

Gateway OpenClaw cần URL công khai hoặc có thể truy cập. Định dạng:

```
https://your-gateway-host:3000/bluebubbles-webhook?password=YOUR_WEBHOOK_PASSWORD
```

**Thực hành bảo mật tốt**: Dùng mật khẩu khác cho webhook và API.

### 4.2 Cấu Hình Webhook Trong BlueBubbles

Trong BlueBubbles Server settings:

1. Vào **Settings → Webhook**
2. Bật **Webhook**
3. Đặt **Webhook URL** là gateway URL của bạn (trên)
4. Thêm mật khẩu làm query param hoặc header

### 4.3 Kiểm Tra Webhook

Gửi tin nhắn thử từ thiết bị khác. Kiểm tra log OpenClaw:

```bash
openclaw logs --follow
```

Bạn sẽ thấy sự kiện tin nhắn đến.

---

## Bước 5: Ghép Đôi và Kiểm Soát Truy Cập

### 5.1 Ghép Đôi DM (Mặc Định: Bật)

Theo mặc định, người gửi mới phải được phê duyệt. Khi có người nhắn bạn:

```bash
# Liệt kê ghép đôi đang chờ
openclaw pairing list bluebubbles

# Phê duyệt người gửi
openclaw pairing approve bluebubbles <CODE>

# Từ chối người gửi
openclaw pairing deny bluebubbles <CODE>
```

Mã ghép đôi hết hạn sau **1 giờ**.

### 5.2 Danh Sách Cho Phép (Bỏ Qua Ghép Đôi)

Để tự động phê duyệt các liên hệ cụ thể:

```json5
{
  channels: {
    bluebubbles: {
      dmPolicy: "allowlist",
      allowFrom: [
        "+15555550123",        // Số điện thoại (định dạng E.164)
        "friend@example.com",    // Email
        "chat_id:123",          // Chat ID cụ thể
      ],
    },
  },
}
```

### 5.3 Cài Đặt Nhóm Chat

```json5
{
  channels: {
    bluebubbles: {
      groupPolicy: "allowlist",  // hoặc "open" cho mọi nhóm
      groupAllowFrom: ["+15555550123"],
      groups: {
        "*": { requireMention: true },  // Mặc định: phải @mention bot
        "iMessage;-;chat123": { requireMention: false },  // Ngoại lệ
      },
    },
  },
}
```

---

## Bước 6: Giữ Messages.app Luôn Hoạt Động

Trên Mac headless hoặc VM, Messages.app có thể "ngủ" và ngừng nhận sự kiện.

### Giải Pháp: Kích Hoạt Messages Mỗi 5 Phút

Tạo `~/Scripts/poke-messages.scpt`:

```applescript
try
  tell application "Messages"
    if not running then
      launch
    end if
    set _chatCount to (count of chats)
  end tell
on error
  -- Bỏ qua lỗi tạm thời
end try
```

Tạo `~/Library/LaunchAgents/com.user.poke-messages.plist`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" 
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
  <dict>
    <key>Label</key>
    <string>com.user.poke-messages</string>
    <key>ProgramArguments</key>
    <array>
      <string>/bin/bash</string>
      <string>-lc</string>
      <string>/usr/bin/osascript "$HOME/Scripts/poke-messages.scpt"</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
    <key>StartInterval</key>
    <integer>300</integer>
  </dict>
</plist>
```

Tải nó:

```bash
launchctl load ~/Library/LaunchAgents/com.user.poke-messages.plist
```

---

## Xử Lý Sự Cố

### "Không có tin nhắn đến"

| Kiểm Tra | Lệnh |
|-------|---------|
| BlueBubbles đang chạy? | `curl http://localhost:1234/api/v1/ping` |
| Gateway có thể truy cập? | Kiểm tra webhook URL đúng |
| Ghép đôi đang chờ? | `openclaw pairing list bluebubbles` |
| Firewall chặn? | Xác nhận port 1234 mở |

### "Thả cảm xúc không hoạt động"

Bật Private API trong BlueBubbles settings. Thả cảm xúc cần nó.

### "Chỉnh sửa/thu hồi bị lỗi trên macOS Tahoe (26)"

Vấn đề đã biết — private API thay đổi trên Tahoe. Tắt nó:

```json5
{
  channels: {
    bluebubbles: {
      actions: {
        edit: false,
      },
    },
  },
}
```

### "Messages.app bị ngủ"

Dùng giải pháp LaunchAgent ở trên (Bước 6).

### Chẩn Đoán Đầy Đủ

```bash
openclaw status --deep
openclaw channels status --probe
```

---

## Nâng Cao: ACP Bindings

Biến bất kỳ cuộc trò chuyện iMessage nào thành ACP workspace:

```bash
# Trong DM hoặc nhóm BlueBubbles
/acp spawn codex --bind here
```

Tin nhắn sau sẽ được định tuyến đến ACP session đã spawn.

Hoặc cấu hình binding cố định trong `openclaw.json`:

```json5
{
  bindings: [
    {
      type: "acp",
      agentId: "codex",
      match: {
        channel: "bluebubbles",
        peer: { kind: "dm", id: "+15555550123" },
      },
    },
  ],
}
```

---

## Tài Liệu Tham Khảo Cấu Hình

| Tùy Chọn | Kiểu | Mặc Định | Mô Tả |
|--------|------|---------|-------------|
| `enabled` | boolean | false | Bật kênh |
| `serverUrl` | string | "" | URL REST API BlueBubbles |
| `password` | string | "" | Mật khẩu API |
| `webhookPath` | string | "/bluebubbles-webhook" | Điểm cuối webhook |
| `dmPolicy` | string | "pairing" | "pairing", "allowlist", "open", "disabled" |
| `groupPolicy` | string | "allowlist" | "allowlist", "open", "disabled" |
| `sendReadReceipts` | boolean | true | Gửi biên nhận đã đọc |
| `blockStreaming` | boolean | false | Truyền phản hồi theo khối |
| `mediaMaxMb` | number | 8 | Kích thước tệp đính kèm tối đa (MB) |
| `textChunkLimit` | number | 4000 | Số ký tự tối đa mỗi tin nhắn |

---

## Bước Tiếp Theo

- [Ghép Đôi & Kiểm Soát Truy Cập](../README.md#access-control)
- [Hành Vi Nhóm Chat](../README.md#groups)
- [Định Tuyến Kênh](../README.md#routing)
- [Thực Hành Bảo Mật Tốt Nhất](../../09-security-and-ops/README.md)

---

*Cập nhật lần cuối: 4 tháng 4, 2026 — OpenClaw v2026.4.2*
