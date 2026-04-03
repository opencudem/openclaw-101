# 06 - Tự động hóa

> Thiết lập công việc định kỳ, briefing, và nhắc nhở với cron tích hợp của OpenClaw.

## Module này giải quyết gì

Cron là scheduler tích hợp của OpenClaw. Nó lưu jobs, đánh thức agent đúng lúc, và thực thi tasks — ngay cả khi bạn không đang chat.

## Khi nào dùng module này

- Bạn muốn briefing hàng ngày/tuần
- Bạn cần nhắc nhở định kỳ
- Bạn muốn health check tự động
- Bạn muốn thu thập dữ liệu theo lịch trình

## Cách Cron Hoạt động

```
┌─────────────┐     ┌──────────┐     ┌─────────────┐
│  Cron Job   │────►│  Gateway │────►│  Agent Run  │
│  (scheduled)│     │  (wakes) │     │  (executes) │
└─────────────┘     └──────────┘     └─────────────┘
                                           │
                                           ▼
                                    ┌─────────────┐
                                    │   Channel   │
                                    │  (notifies) │
                                    └─────────────┘
```

## Tổng quan Lệnh Cron

```bash
# Liệt kê tất cả cron jobs
openclaw cron list

# Thêm job định kỳ
openclaw cron add \
  --name "job-name" \
  --cron "0 9 * * *" \
  --message "What to do"

# Thêm job một lần
openclaw cron add \
  --name "reminder" \
  --at "2026-04-05 14:00" \
  --message "Meeting in 15 minutes"

# Sửa job
openclaw cron edit <job-id> --announce --channel telegram

# Bật/tắt job
openclaw cron enable <job-id>
openclaw cron disable <job-id>

# Chạy job thủ công
openclaw cron run <job-id>

# Kiểm tra job runs
openclaw cron runs --id <job-id>

# Xóa job
openclaw cron rm <job-id>
```

## Bắt đầu nhanh

### Tạo Cron Job đầu tiên

```bash
# Thêm briefing hàng ngày với delivery đến Telegram
openclaw cron add \
  --name "daily-briefing" \
  --cron "0 9 * * *" \
  --message "Send morning briefing with weather, calendar, and priority tasks" \
  --announce \
  --channel telegram \
  --to "123456789"
```

### Liệt kê Cron Jobs

```bash
openclaw cron list
```

Output hiển thị job ID, tên, lịch trình, lần chạy tiếp theo, và trạng thái.

### Kiểm tra Job Runs

```bash
# Kiểm tra kết quả của job cụ thể
openclaw cron runs --id <job-id>
```

### Xóa Job

```bash
openclaw cron rm <job-id>
```

## Cú pháp Cron

Cờ `--cron` dùng định dạng biểu thức cron chuẩn:

| Biểu thức | Ý nghĩa |
|-----------|---------|
| `0 9 * * *` | Mỗi ngày lúc 9:00 AM |
| `0 */6 * * *` | Mỗi 6 giờ |
| `0 9 * * 1` | Mỗi thứ Hai lúc 9:00 AM |
| `0 0 1 * *` | Ngày đầu tiên mỗi tháng |
| `*/15 * * * *` | Mỗi 15 phút |

Định dạng: `phút giờ ngày tháng thứ`

## Các Loại Session

Cron jobs có thể chạy trong các chế độ session khác nhau:

| Loại Session | Mô tả | Trường hợp sử dụng |
|--------------|-------|-------------------|
| `main` | Chạy trong heartbeat, chia sẻ ngữ cảnh | Task âm thầm, công việc nền |
| `isolated` | Session mới mỗi lần chạy (mặc định) | Job nhẹ, không cần lịch sử |
| `session:custom-id` | Session đặt tên persistent | Jobs cần nhớ state |

### Ví dụ Session

```bash
# Main session - chạy trong heartbeat
openclaw cron add \
  --name "health-check" \
  --cron "0 */4 * * *" \
  --message "Check service health" \
  --session main

# Isolated session - context mới mỗi lần chạy (mặc định)
openclaw cron add \
  --name "morning-brief" \
  --cron "0 8 * * *" \
  --message "Generate morning briefing" \
  --session isolated \
  --announce \
  --channel telegram

# Custom persistent session
openclaw cron add \
  --name "data-collector" \
  --cron "0 */6 * * *" \
  --message "Collect metrics" \
  --session session:metrics-collector
```

## Các Pattern Tự động hóa Thông dụng

### Morning Briefing

```bash
openclaw cron add \
  --name "morning-brief" \
  --cron "0 8 * * *" \
  --message "Send morning briefing: weather, calendar, and priority tasks" \
  --announce \
  --channel telegram \
  --to "YOUR_CHAT_ID"
```

### Daily Standup Reminder

```bash
openclaw cron add \
  --name "standup" \
  --cron "0 9 * * 1-5" \
  --message "Daily standup time! Check GitHub PRs and report status." \
  --announce \
  --channel slack \
  --to "#standup-channel"
```

### Weekly Review

```bash
openclaw cron add \
  --name "weekly-review" \
  --cron "0 17 * * 5" \
  --message "Weekly review: summarize completed tasks, check upcoming priorities" \
  --announce \
  --channel discord \
  --to "YOUR_USER_ID"
```

### Health Check

```bash
openclaw cron add \
  --name "health-check" \
  --cron "0 */4 * * *" \
  --message "Run health check on all services and report any issues" \
  --session isolated \
  --no-deliver
```

### One-Shot Reminder

```bash
# Lên lịch nhắc nhở một lần
openclaw cron add \
  --name "meeting-reminder" \
  --at "2026-04-05 14:00" \
  --message "Meeting with team in 15 minutes" \
  --announce \
  --channel telegram
```

## Ví dụ Copy-Paste

### Job với Light Context

Cho thực thi nhanh với bootstrap tối thiểu:

```bash
openclaw cron add \
  --name "quick-check" \
  --cron "0 * * * *" \
  --message "Quick status check" \
  --light-context \
  --no-deliver
```

### Chỉnh sửa Jobs

```bash
# Thêm delivery vào job hiện có
openclaw cron edit <job-id> \
  --announce \
  --channel telegram \
  --to "123456789"

# Tắt delivery
openclaw cron edit <job-id> --no-deliver

# Bật light context
openclaw cron edit <job-id> --light-context
```

### Chạy Thủ công

```bash
# Kích hoạt job thủ công
openclaw cron run <job-id>
# Returns: { ok: true, enqueued: true, runId }

# Kiểm tra kết quả
openclaw cron runs --id <job-id>
```

## Chính sách Retry

Cron jobs có tự động retry khi thất bại:

| Lần thử | Delay |
|---------|-------|
| 1st retry | 30 giây |
| 2nd retry | 1 phút |
| 3rd retry | 5 phút |
| 4th retry | 15 phút |
| 5th retry | 60 phút |

Jobs thất bại sau tất cả các lần retry được đánh dấu failed. Kiểm tra `openclaw cron runs --id <job-id>` để biết chi tiết.

## Lưu trữ Job

Cron jobs được lưu tại:
- **Vị trí:** `~/.openclaw/cron/jobs.json`
- **Định dạng:** JSON với job definitions
- **Persistence:** Sống sót qua restart

## Lỗi Thường Gặp và Xử lý

| Vấn đề | Cách giải quyết |
|--------|-----------------|
| Job không chạy | Kiểm tra gateway đang chạy: `openclaw gateway status` |
| Sai múi giờ | Đặt biến môi trường `TZ` |
| Job chạy nhưng không output | Xác minh cờ `--announce` và kết nối kênh |
| Missed executions | Gateway đã tắt; jobs không backfill mặc định |
| "Invalid cron expression" | Kiểm tra định dạng: `phút giờ ngày tháng thứ` |
| Không thể sửa job | Dùng job ID từ `openclaw cron list`, không phải tên |

## Pattern Nâng cao

### Job Dependencies

Chuỗi jobs với lịch trình offset:

```bash
# Fetch data mỗi 6 giờ
openclaw cron add \
  --name "fetch-data" \
  --cron "0 */6 * * *" \
  --message "Fetch metrics from API" \
  --no-deliver

# Process data 5 phút sau fetch
openclaw cron add \
  --name "process-data" \
  --cron "5 */6 * * *" \
  --message "Process collected metrics" \
  --announce \
  --channel slack
```

### Logic điều kiện trong Messages

Cron `--message` là prompt cho agent. Bạn có thể đưa vào hướng dẫn điều kiện:

```bash
openclaw cron add \
  --name "smart-briefing" \
  --cron "0 9 * * *" \
  --message "Check calendar and weather. If it's raining, mention umbrella. If calendar is empty, suggest a focus day." \
  --announce \
  --channel telegram
```

### Kiểm tra Trạng thái Cron

```bash
# Kiểm tra trạng thái tổng thể của hệ thống cron
openclaw cron status
```

## Module Liên quan và Bước Tiếp

- Trước đó: [05-integrations](../05-integrations/)
- Tiếp theo: [07-browser-automation](../07-browser-automation/)
- Tham khảo: [Cron documentation](https://docs.openclaw.ai/cli/cron)

---

**Thời gian hoàn thành:** 45 phút  
**Yêu cầu trước:** [05-integrations](../05-integrations/)  
**Kết quả:** Công việc định kỳ tự động, briefing theo lịch, workflow hands-off
