# 07 - Tự động hóa Trình duyệt

> Tự động hóa nghiên cứu web, điền form, và trích xuất dữ liệu với browser skills.

## Module này giải quyết gì

Browser automation cho phép OpenClaw tương tác với websites như con người: điều hướng trang, điền form, click nút, và trích xuất dữ liệu — tất cả qua lệnh ngôn ngữ tự nhiên.

## Khi nào dùng module này

- Bạn cần nghiên cứu chủ đề trên nhiều site
- Bạn muốn tự động hóa gửi form
- Bạn cần trích xuất dữ liệu từ websites
- Bạn muốn giám sát thay đổi website

## Bắt đầu nhanh

### Cài đặt Browser Skill

```bash
# Cài từ ClawHub
openclaw skills install browser-automation

# Kiểm tra trạng thái browser
openclaw browser status
```

### Cách sử dụng cơ bản

```
User: Go to example.com and find the pricing
Agent: [Mở browser, điều hướng, trích xuất thông tin giá]

User: Search for "OpenClaw tutorials" and summarize the first 3 results
Agent: [Thực hiện tìm kiếm, truy cập các trang, trích xuất nội dung, tóm tắt]

User: Fill out the contact form on example.com with my info
Agent: [Điều hướng đến form, điền các trường dùng bộ nhớ của bạn, submit]
```

## Lệnh CLI Browser

| Lệnh | Mô tả | Ví dụ |
|------|-------|-------|
| `openclaw browser status` | Kiểm tra trạng thái browser | `openclaw browser status` |
| `openclaw browser start` | Khởi động browser session | `openclaw browser start` |
| `openclaw browser stop` | Dừng browser session | `openclaw browser stop` |
| `openclaw browser tabs` | Liệt kê tab đang mở | `openclaw browser tabs` |
| `openclaw browser open <url>` | Mở URL trong tab mới | `openclaw browser open https://example.com` |
| `openclaw browser close <tab-id>` | Đóng tab cụ thể | `openclaw browser close tab-123` |
| `openclaw browser navigate <url>` | Điều hướng đến URL | `openclaw browser navigate https://example.com` |
| `openclaw browser screenshot` | Chụp ảnh màn hình | `openclaw browser screenshot` |
| `openclaw browser snapshot` | Chụp trạng thái trang | `openclaw browser snapshot` |
| `openclaw browser click <selector>` | Click element | `openclaw browser click "button.submit"` |
| `openclaw browser type <selector> <text>` | Gõ vào input | `openclaw browser type "#email" "user@example.com"` |
| `openclaw browser fill <selector> <value>` | Điền trường form | `openclaw browser fill "#country" "Vietnam"` |
| `openclaw browser press <key>` | Nhấn phím | `openclaw browser press "Enter"` |
| `openclaw browser wait <ms>` | Chờ milliseconds | `openclaw browser wait 1000` |
| `openclaw browser evaluate <script>` | Thực thi JavaScript | `openclaw browser evaluate "document.title"` |

## Ví dụ copy-paste

### Workflow nghiên cứu qua lệnh chat

Thay vì viết JavaScript, dùng ngôn ngữ tự nhiên với agent:

```
User: Open browser and navigate to google.com
Agent: [Dùng openclaw browser navigate]

User: Search for "OpenClaw tutorials" and visit the first 3 results
Agent: [Thực hiện tìm kiếm, điều hướng, trích xuất nội dung]

User: Extract all links from the current page
Agent: [Dùng openclaw browser extractAll]
```

### Tự động hóa form qua chat

```
User: Navigate to example.com/contact
Agent: [Mở trang contact]

User: Fill the name field with "John", email with "john@example.com", and message with "Hello"
Agent: [Dùng openclaw browser type cho mỗi trường]

User: Click the submit button
Agent: [Dùng openclaw browser click]

User: Take a screenshot to confirm
Agent: [Dùng openclaw browser screenshot]
```

### Dùng browser trong skills (qua exec)

Skills có thể kích hoạt browser commands qua exec tool:

```javascript
// ~/.openclaw/skills/browser-helper/scripts/navigate.js
const { execSync } = require('child_process');

module.exports = {
  async navigateTo({ url }) {
    // Dùng openclaw CLI để điều hướng
    const result = execSync(`openclaw browser navigate "${url}"`, { encoding: 'utf8' });
    return { success: true, result };
  },
  
  async takeScreenshot() {
    const result = execSync('openclaw browser screenshot', { encoding: 'utf8' });
    return { screenshot: result };
  }
};
```

### Giám sát giá qua chat

```
User: Navigate to example.com/product and extract the price
Agent: [Dùng openclaw browser navigate và extract]

User: Compare with last check and alert if below $50
Agent: [Duy trì lịch sử, tính toán thay đổi, gửi cảnh báo nếu cần]
```

Hoặc tạo cron job để kiểm tra định kỳ:

```bash
openclaw cron add \
  --name "price-check" \
  --cron "0 */6 * * *" \
  --message "Check price on example.com/product and alert if below $50" \
  --announce \
  --channel telegram
```

## Cân nhắc An toàn

⚠️ **Browser automation rất mạnh — dùng có trách nhiệm:**

1. **Rate limiting**: Đừng spam websites. Thêm delay giữa các request.
2. **Terms of service**: Tôn trọng robots.txt và điều khoản site.
3. **Dữ liệu nhạy cảm**: Không bao giờ tự động hóa đăng nhập tài chính/ngân hàng.
4. **DM policies**: Cho hành động quan trọng, gửi đến kênh bị hạn chế để phê duyệt

## Lỗi thường gặp và xử lý

| Vấn đề | Cách giải quyết |
|--------|-----------------|
| Trang không tải | Kiểm tra URL; xác minh site không chặn automation |
| Không tìm thấy element | Chờ trang tải; kiểm tra selector; thêm delay |
| CAPTCHA triggered | Dừng để giải thủ công; không tự động hóa CAPTCHA |
| Session hết hạn | Xác thực lại; dùng saved session nếu có |

## Pattern nâng cao

### Dùng browser với cron

Lên lịch browser tasks định kỳ:

```bash
# Kiểm tra giá hàng ngày
openclaw cron add \
  --name "daily-price-check" \
  --cron "0 9 * * *" \
  --message "Check prices on monitored sites and report changes" \
  --announce \
  --channel telegram
```

### Workflow browser nhiều bước

Chuỗi nhiều browser commands:

```
User: Navigate to dashboard.example.com, extract the metrics, and email the summary
Agent: [Dùng browser navigate, extract, sau đó skill email]
```

## Module liên quan và bước tiếp

- Trước đó: [06-automation](../06-automation/)
- Tiếp theo: [08-workflows](../08-workflows/)
- An toàn: [Approval patterns](../09-security-and-ops/approval-patterns.md)

---

**Thời gian hoàn thành:** 60 phút  
**Yêu cầu trước:** [06-automation](../06-automation/)  
**Kết quả:** Khả năng nghiên cứu và automation web, workflow trích xuất dữ liệu
