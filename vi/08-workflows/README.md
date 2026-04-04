# 08 - Workflows

> Kết hợp channels, skills, integrations, và automation thành các hệ thống hoàn chỉnh.

## Module này giải quyết gì

Tính năng riêng lẻ có ích; workflow kết hợp thì mạnh mẽ. Module này hướng dẫn cách xây dựng các hệ thống hoàn chỉnh xử lý các kịch bản thực tế.

## Khi nào dùng module này

- Bạn muốn automation từ đầu đến cuối
- Bạn cần quy trình nhiều bước
- Bạn muốn logic điều kiện
- Bạn cần phối hợp nhiều công cụ

## Kiến trúc Workflow

```
┌─────────────────────────────────────────────────────────────┐
│                       WORKFLOW                              │
├─────────────────────────────────────────────────────────────┤
│  Trigger → Process → Decision → Action → Notify → Archive  │
│                                                             │
│  Ví dụ:                                                    │
│  9:00 AM → Kiểm tra PRs → Có pending? → Review → Slack → Log │
└─────────────────────────────────────────────────────────────┘
```

## Bắt đầu nhanh

### Workflow đơn giản qua Chat

Thay vì viết JavaScript, dùng ngôn ngữ tự nhiên:

```
User: Gửi tôi briefing buổi sáng với thời tiết và lịch hôm nay
Agent: [Dùng weather skill và calendar skill để tạo briefing]

User: Lên lịch briefing này mỗi ngày lúc 8 AM
Agent: [Tạo cron job: openclaw cron add --name "morning-brief" --cron "0 8 * * *" --message "Send morning briefing" --announce --channel telegram]
```

### Workflow phức tạp với nhánh qua Chat

Dùng ngôn ngữ tự nhiên cho logic điều kiện:

```
User: Kiểm tra PRs GitHub của tôi
Agent: [Liệt kê PRs đang mở với chi tiết]

User: Với PRs lớn, gửi yêu cầu review chi tiết đến Slack
Agent: [Xác định PRs lớn, thông báo kênh Slack]

User: Với PRs nhỏ có test pass, hỏi tôi có muốn approve không
Agent: [Xác định PRs nhỏ sạch, gửi yêu cầu approval đến DMs]

User: Với PRs có test fail, thông báo các tác giả
Agent: [Comment trên PRs đề cập tác giả về lỗi test]
```

Hoặc thiết lập cron jobs tự động cho các loại PR khác nhau:

```bash
# Large PRs - review chi tiết (mỗi 2 giờ)
openclaw cron add \
  --name "large-pr-review" \
  --cron "0 */2 * * 1-5" \
  --message "Find large PRs and request thorough reviews" \
  --announce \
  --channel slack

# Small clean PRs - approval queue (mỗi giờ)
openclaw cron add \
  --name "small-pr-approval" \
  --cron "0 * * * 1-5" \
  --message "Find small PRs with passing tests and queue for approval" \
  --announce \
  --channel telegram
```

## Ví dụ copy-paste

### Workflow template

```javascript
// ~/.openclaw/skills/workflow-template/scripts/run.js
/**
 * Workflow: [Tên]
 * Description: [Làm gì]
 * Trigger: [Kích hoạt như thế nào]
 */

module.exports = {
  async execute(context = {}) {
    try {
      // 1. INPUT - Thu thập
      const input = await this.gatherInput(context);
      
      // 2. PROCESS - Xử lý
      const result = await this.process(input);
      
      // 3. OUTPUT - Gửi
      await this.deliver(result);
      
      // 4. LOG - Ghi lại
      await this.log(result);
      
      return { success: true, result };
    } catch (error) {
      await this.handleError(error);
      return { success: false, error: error.message };
    }
  },
  
  async gatherInput(context) {
    return context;
  },
  
  async process(input) {
    return input;
  },
  
  async deliver(result) {
    console.log('Deliver:', result);
  },
  
  async log(result) {
    // Ghi vào file thay vì skills.memory
    const fs = require('fs').promises;
    await fs.appendFile(
      '/tmp/workflow.log',
      `${new Date().toISOString()}: ${JSON.stringify(result)}\n`
    );
  },
  
  async handleError(error) {
    // Dùng exec để gửi notification qua openclaw CLI
    const { execSync } = require('child_process');
    execSync(`openclaw channels send --channel telegram --message "Workflow failed: ${error.message}"`);
  }
};
```

### Workflow đa kênh qua cron và chat

Thiết lập escalation qua cron jobs:

```bash
# Low severity - Telegram only
openclaw cron add \
  --name "alert-low" \
  --cron "0 * * * *" \
  --message "Check low priority alerts and notify via telegram" \
  --announce \
  --channel telegram

# High severity - Multiple channels  
openclaw cron add \
  --name "alert-high" \
  --cron "*/15 * * * *" \
  --message "Check high priority alerts and notify via telegram, email, and slack" \
  --announce \
  --channel telegram
```

### Scheduled workflow với cron

Dùng `openclaw cron add` với messages mô tả:

```bash
# Morning briefing lúc 8 AM hàng ngày
openclaw cron add \
  --name "morning-routine" \
  --cron "0 8 * * *" \
  --message "Generate and send morning briefing with weather and calendar" \
  --announce \
  --channel telegram

# PR check lúc 9 AM và 2 PM các ngày trong tuần
openclaw cron add \
  --name "pr-check" \
  --cron "0 9,14 * * 1-5" \
  --message "Check pending GitHub PRs and send summary" \
  --announce \
  --channel slack

# Weekly report thứ Sáu lúc 5 PM
openclaw cron add \
  --name "weekly-report" \
  --cron "0 17 * * 5" \
  --message "Generate weekly activity summary" \
  --announce \
  --channel email
```

## Các Workflow Flagship

### Morning Briefing

- **Trigger:** Hàng ngày lúc 8:00 AM
- **Components:** Weather, Calendar, Tasks
- **Output:** Telegram message
- **Files:** [examples/personal-productivity/morning-briefing.md](../examples/personal-productivity/morning-briefing.md)

### Engineering Inbox

- **Trigger:** Cron + Webhook
- **Components:** GitHub, Discord, Linear
- **Output:** Channel notifications
- **Files:** [examples/engineering/pr-alerts.md](../examples/engineering/pr-alerts.md)

### Research Agent

- **Trigger:** Manual hoặc scheduled
- **Components:** Browser, Search, Summarization
- **Output:** Report to email/Notion
- **Files:** [examples/research-and-content/research-agent.md](../examples/research-and-content/research-agent.md)

## Lỗi thường gặp và xử lý

| Vấn đề | Cách giải quyết |
|--------|-----------------|
| Workflow quá phức tạp | Chia thành các sub-workflows nhỏ hơn |
| Partial failures | Dùng transactions; implement rollback |
| Thiếu ngữ cảnh | Thêm logging tại mỗi bước |
| Race conditions | Thêm locks; dùng sequential execution |

## Pattern nâng cao

### Workflow composition

Chuỗi nhiều cron jobs hoặc lệnh chat:

```
User: Chạy morning briefing, sau đó kiểm tra emails, sau đó tóm tắt lịch
Agent: [Thực thi mỗi workflow tuần tự, truyền ngữ cảnh giữa các bước]
```

Hoặc tạo dependent cron jobs với delays:

```bash
# Bước 1: Morning briefing lúc 8 AM
openclaw cron add \
  --name "step1-briefing" \
  --cron "0 8 * * *" \
  --message "Generate morning briefing" \
  --announce \
  --channel telegram

# Bước 2: Email check lúc 8:05 AM (5 phút sau briefing)
openclaw cron add \
  --name "step2-email" \
  --cron "5 8 * * *" \
  --message "Check and summarize emails" \
  --announce \
  --channel telegram

# Bước 3: Calendar summary lúc 8:10 AM
openclaw cron add \
  --name "step3-calendar" \
  --cron "10 8 * * *" \
  --message "Summarize today's calendar" \
  --announce \
  --channel telegram
```

### Dynamic workflow qua conditional cron

Dùng các cron jobs khác nhau dựa trên điều kiện:

```bash
# Research workflow (chạy nếu flag research-needed được set)
openclaw cron add \
  --name "research-check" \
  --cron "0 10 * * 1" \
  --message "If research-needed flag is set, perform web research and summarize findings" \
  --announce \
  --channel telegram

# Approval workflow (gửi đến kênh bị hạn chế cho approval người)
openclaw cron add \
  --name "approval-check" \
  --cron "0 */4 * * *" \
  --message "Send pending approval requests to admin channel" \
  --announce \
  --channel telegram \
  --to "admin-user-id"
```

## Phân Tích Tài Liệu PDF (v2026.3.2+)

> Xử lý PDF native với Anthropic/Google providers, fallback extraction cho các providers khác.

### Giải quyết gì

Tool `pdf` phân tích tài liệu PDF và trích xuất text mà không cần OCR pipelines phức tạp. Xử lý:
- **Research papers** — Trích xuất findings và summaries
- **Hóa đơn/biên lai** — Parse số tiền và ngày tháng
- **Báo cáo** — Generate executive summaries
- **Hợp đồng** — Xác định các điều khoản chính
- **So sánh multi-document** — So sánh các phiên bản cạnh nhau

### Hỗ trợ Provider

| Provider | Mode | Ghi chú |
|----------|------|---------|
| **Anthropic** | Native | Gửi raw PDF bytes qua DocumentBlockParam |
| **Google** | Native | Gửi raw PDF qua inlineData với MIME application/pdf |
| **OpenAI/Khác** | Fallback | Trích xuất text qua pdfjs-dist, images qua @napi-rs/canvas |

### Cấu hình

Thêm vào `openclaw.json`:

```json5
{
  agents: {
    defaults: {
      pdfModel: {
        primary: "anthropic/claude-opus-4-6",
        fallbacks: ["openai/gpt-5-mini"],
      },
      pdfMaxBytesMb: 10,
      pdfMaxPages: 20,
    },
  },
}
```

**Tùy chọn cấu hình:**
- `pdfModel` — Model phân tích PDF (fallback đến `imageModel` nếu không set)
- `pdfMaxBytesMb` — Giới hạn size mỗi PDF (mặc định: 10)
- `pdfMaxPages` — Số trang tối đa trích xuất (mặc định: 20)

### Ví dụ Sử dụng

**Phân tích một PDF:**
```json
{
  "pdf": "/home/reports/q1-2026.pdf",
  "prompt": "Summarize revenue trends and key metrics in 5 bullets"
}
```

**So sánh nhiều PDF:**
```json
{
  "pdfs": ["/home/proposals/v1.pdf", "/home/proposals/v2.pdf"],
  "prompt": "Compare pricing and timeline changes between versions"
}
```

**Trích xuất trang cụ thể (chỉ fallback mode):**
```json
{
  "pdf": "https://example.com/contract.pdf",
  "pages": "1-3,7",
  "prompt": "Extract all payment terms and deadlines"
}
```

**Qua chat:**
```
User: Đọc /home/invoices/march.pdf và cho tôi biết tổng số tiền
Agent: Hóa đơn từ Acme Corp ngày March 1, 2026 có tổng 
       $4,750.00 — $3,500 cho dịch vụ consulting và $1,250 
       cho software licenses.
```

### Tham số Input

| Tham số | Type | Mô tả |
|---------|------|-------|
| `pdf` | `string` | Đường dẫn hoặc URL PDF đơn |
| `pdfs` | `string[]` | Nhiều PDFs (tối đa 10) |
| `prompt` | `string` | Prompt phân tích (mặc định: "Analyze this PDF document.") |
| `pages` | `string` | Lọc trang: "1-5", "1,3,7-9" (chỉ fallback mode) |
| `model` | `string` | Ghi đè model: "provider/model" |
| `maxBytesMb` | `number` | Giới hạn size mỗi PDF (MB) |

### Các Mode Thực thi

**Native mode** (Anthropic/Google):
- Gửi raw PDF bytes trực tiếp đến provider APIs
- Tham số `pages` KHÔNG được hỗ trợ (trả về lỗi nếu dùng)
- Nhanh nhất, chính xác nhất

**Fallback mode** (Các providers khác):
1. Trích xuất text từ các trang đã chọn (tối đa `pdfMaxPages`)
2. Nếu text trích xuất < 200 chars, render trang thành PNG và include như images
3. Gửi combined content đến model

### Tích hợp Workflow

**Xử lý báo cáo hàng ngày:**
```bash
openclaw cron add \
  --name "process-reports" \
  --cron "0 9 * * *" \
  --message "Đọc /home/reports/daily.pdf và post summary lên Slack" \
  --announce \
  --channel slack \
  --to "#reports"
```

**Research workflow với nguồn PDF:**
```
Trigger: PDF mới trong ~/downloads/
Action: Trích xuất key findings → Thêm vào research-notes.md → Post summary
```

### Giới hạn

- Tối đa 10 PDFs mỗi lần gọi
- Native mode không hỗ trợ filter `pages`
- Workspace-only file policy có thể từ chối đường dẫn ngoài allowed roots
- Cần `pdfjs-dist` và `@napi-rs/canvas` cho fallback mode

## Module liên quan và bước tiếp

- Trước đó: [07-browser-automation](../07-browser-automation/)
- Tiếp theo: [09-security-and-ops](../09-security-and-ops/)
- Examples: [../examples/](../examples/)

---

**Thời gian hoàn thành:** 90 phút  
**Yêu cầu trước:** [07-browser-automation](../07-browser-automation/)  
**Kết quả:** Các hệ thống tự động hoàn chỉnh xử lý các kịch bản thực tế
