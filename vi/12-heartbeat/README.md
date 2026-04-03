# 12 — Cấu hình Heartbeat

> Cấu hình agent check-in định kỳ cho tự động hóa chủ động và workflow bảo trì.

## Module này giải quyết gì

Heartbeat chạy **periodic agent turns** để agent có thể đưa thông tin quan trọng lên bề mặt mà bạn không phải hỏi. Khác với cron jobs (chạy nền), heartbeat tạo **main session turn** nơi agent có thể:

- Kiểm tra các follow-up khẩn cấp (emails, calendar, notifications)
- Review và cập nhật documentation
- Chạy maintenance workflows (như DocLoop)
- Đưa ra các cảnh báo cần sự chú ý của người

**Khác biệt chính với cron:**
- **Cron** = Thực thi task nền, tạo task records, không thể kích hoạt agent
- **Heartbeat** = Main session turn, tương tác trực tiếp agent, không có task records

## Khi nào dùng module này

- Bạn muốn **check-in chủ động** không cần prompt thủ công
- Bạn cần **bảo trì định kỳ** (documentation, cleanup, monitoring)
- Bạn muốn **batch nhiều kiểm tra** vào một lần (inbox + calendar + weather)
- Bạn cần **ngữ cảnh hội thoại** từ tin nhắn gần đây

**Dùng cron thay vào đó khi:**
- Thời gian chính xác quan trọng ("9:00 AM sharp every Monday")
- Task cần cô lập khỏi main session history
- One-shot reminders ("nhắc tôi sau 20 phút")
- Output nên gửi trực tiếp không qua main session

## Yêu cầu

**OpenClaw Version:** v0.31+ (cho cấu hình `heartbeat`)

**Bắt buộc:** Gateway đang hoạt động với ít nhất một channel được cấu hình

## Bắt đầu nhanh

### Bước 1: Tạo HEARTBEAT.md Checklist

Tạo checklist nhỏ trong workspace:

```bash
cat > ~/.openclaw/workspace/HEARTBEAT.md << 'EOF'
# Heartbeat Checklist

## DocLoop Maintenance
- [ ] Kiểm tra releases OpenClaw mới
- [ ] Review AUDIT_REPORT.md cho các khoảng trống
- [ ] Cập nhật documentation nếu cần

## System Health
- [ ] Quick scan: inbox hoặc notifications khẩn cấp
- [ ] Kiểm tra gateway status

## Response Rules
- Nếu không cần chú ý → reply HEARTBEAT_OK
- Nếu có việc → làm việc và báo cáo kết quả
- Nếu bị block → ghi chú cần gì và hỏi người
EOF
```

Giữ nó **nhỏ** để tránh token burn.

### Bước 2: Thêm Heartbeat Configuration

Sửa `~/.openclaw/openclaw.json`:

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",              // Chạy mỗi 30 phút
        target: "discord",         // Gửi đến Discord
        accountId: "default",    // Dùng Discord account này
        to: "YOUR_CHANNEL_ID",     // Channel cụ thể (tùy chọn)
        
        // Tối ưu chi phí (khuyến nghị cho maintenance workflows):
        lightContext: true,        // Chỉ load HEARTBEAT.md
        isolatedSession: true,     // Session mới mỗi lần
        
        prompt: "Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.",
      },
    },
  },
}
```

### Bước 3: Restart Gateway

```bash
openclaw gateway restart
```

### Bước 4: Xác minh

Chờ 30 phút hoặc kích hoạt thủ công:

```bash
openclaw system event --text "Test heartbeat" --mode now
```

## Tham khảo Cấu hình

### `every` — Khoảng thời gian

```json5
heartbeat: {
  every: "30m",    // 30 phút (mặc định)
  // every: "1h",  // 1 giờ
  // every: "0m",  // Tắt heartbeat
}
```

**Mặc định:** `30m` (hoặc `1h` cho Anthropic OAuth/setup-token mode)

### `target` — Đích gửi

| Giá trị | Hành vi |
|---------|---------|
| `"none"` (mặc định) | Chạy nhưng không gửi ra ngoài |
| `"last"` | Gửi đến kênh ngoài cuối cùng được dùng |
| `"discord"` | Gửi đến Discord |
| `"telegram"` | Gửi đến Telegram |
| `"whatsapp"` | Gửi đến WhatsApp |

**Với account và channel:**

```json5
heartbeat: {
  target: "discord",
  accountId: "docloop",
  to: "1488561209300095230",
}
```

### `lightContext` — Bootstrap Tối thiểu

| Giá trị | File Bootstrap | Chi phí Token | Use Case |
|---------|----------------|---------------|----------|
| `false` (mặc định) | Tất cả: `AGENTS.md`, `SOUL.md`, `USER.md`, `TOOLS.md`, `HEARTBEAT.md` | ~10-50K tokens | Full persona context |
| `true` | **Chỉ `HEARTBEAT.md`** | ~1-5K tokens | Checklist-only workflows |

**Khuyến nghị cho:**
- DocLoop maintenance (chỉ cần checklist)
- Inbox/calendar checks (không cần personality)
- System monitoring (tác vụ kỹ thuật)

### `isolatedSession` — Session Mới Mỗi Lần Chạy

| Giá trị | Session Behavior | Chi phí Token |
|---------|------------------|---------------|
| `false` (mặc định) | Tiếp tục main session với full history | ~50-100K+ tokens |
| `true` | **Session mới, không có prior context** | ~2-5K tokens |

**Khuyến nghị cho:**
- Maintenance workflows (mỗi lần chạy độc lập)
- Testing/validation (không có stale context)
- Setups nhạy cảm với chi phí (~90% token savings)

**⚠️ Trade-off:** Không thể truy cập công việc prior hoặc conversation history.

### `activeHours` — Khung giờ Hoạt động

Hạn chế heartbeats trong giờ làm việc:

```json5
heartbeat: {
  every: "30m",
  activeHours: {
    start: "09:00",              // 9 AM
    end: "22:00",                // 10 PM
    timezone: "Asia/Ho_Chi_Minh",
  },
}
```

Ngoài khung này, heartbeats được bỏ qua.

### `prompt` — Hướng dẫn Tùy chỉnh

Override prompt mặc định:

```json5
heartbeat: {
  prompt: "Check GitHub releases for new OpenClaw versions. If v2026.4.3+ released, start DocLoop Phase 1. Otherwise reply HEARTBEAT_OK.",
}
```

## Response Contract

### Quy tắc Agent

| Điều kiện | Phản hồi |
|-----------|----------|
| Không cần chú ý | `HEARTBEAT_OK` (chỉ vậy) |
| Có việc | Báo cáo kết quả, KHÔNG include HEARTBEAT_OK |
| Bị block/cần người | Alert với yêu cầu cụ thể |

## Cost Optimization

| Chiến lược | Config | Tiết kiệm |
|------------|--------|-----------|
| Dùng `lightContext: true` | Chỉ `HEARTBEAT.md` | ~80% |
| Dùng `isolatedSession: true` | Session mới | ~90% |
| Kết hợp cả hai | `lightContext + isolatedSession` | ~95% |
| Model rẻ hơn | `model: "ollama/llama3.2:1b"` | ~99% |

## Các Lỗi Thường Gặp

| Lỗi | Tại sao Xấu | Giải pháp |
|-----|-------------|-----------|
| Không có config `heartbeat` | Heartbeat không bao giờ chạy | Thêm `agents.defaults.heartbeat` hoặc per-agent config |
| `target: "none"` và mong đợi tin nhắn | Hoạt động âm thầm | Dùng `target: "last"` hoặc kênh cụ thể |
| `lightContext: true` nhưng cần `SOUL.md` | Agent hành xử generic | Dùng `lightContext: false` cho persona context |
| `isolatedSession: true` và cần history | Không thể truy cập prior work | Dùng `isolatedSession: false` hoặc lưu trong file |
| `HEARTBEAT.md` trống | Heartbeat skip | Thêm các mục checklist thực sự |

## Heartbeat vs Cron: Khi nào Dùng Cái nào

| Use Case | Heartbeat | Cron |
|----------|-----------|------|
| Nhiều kiểm tra batched | ✅ Yes | ❌ No (một task mỗi cron) |
| Cần conversational context | ✅ Yes | ❌ No (isolated) |
| Thời gian có thể drift (~30 min) | ✅ Yes | ❌ Cần chính xác |
| Lịch trình chính xác yêu cầu | ❌ No | ✅ Yes |
| One-shot reminders | ❌ No | ✅ Yes |
| Model/thinking level khác | ❌ No | ✅ Yes |
| Gửi không qua main session | ❌ No | ✅ Yes |

**Mẹo:** Batch các kiểm tra periodic tương tự vào `HEARTBEAT.md` thay vì tạo nhiều cron jobs.

## Per-Agent Heartbeats

Nếu bất kỳ `agents.list[]` nào có block `heartbeat`, **chỉ các agent đó** chạy heartbeats:

```json5
{
  agents: {
    defaults: {
      heartbeat: { every: "30m", target: "last" },  // Shared defaults
    },
    list: [
      { id: "main" },                              // Không heartbeat
      {
        id: "docloop",
        heartbeat: {                               // Chạy heartbeats
          every: "1h",
          target: "discord",
          accountId: "docloop",
        }
      },
    ],
  },
}
```

Per-agent block merge trên defaults.

## Ví dụ Thực tế: DocLoop

```json5
{
  agents: {
    list: [
      {
        id: "docloop",
        name: "DocLoop",
        workspace: "~/.openclaw/workspace-docloop",
        heartbeat: {
          every: "30m",
          target: "discord",
          accountId: "docloop",
          to: "1488561209300095230",
          lightContext: true,
          isolatedSession: true,
          prompt: "Read HEARTBEAT.md if it exists. Follow the DocLoop 6-Phase Workflow strictly. Do not infer tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.",
        },
      },
    ],
  },
}
```

**HEARTBEAT.md:**
```markdown
# DocLoop Documentation Maintenance

## 6-Phase Checklist
- [ ] PHASE 1: Scan for new releases, skills, use cases
- [ ] PHASE 2: Audit gaps against existing docs
- [ ] PHASE 3: Research authoritative sources
- [ ] PHASE 4: Create/update documentation
- [ ] PHASE 5: Validate (keep-or-discard gate)
- [ ] PHASE 6: Commit & report

## Rules
1. Official docs = ground truth only — NO invention
2. Never push to main — always PR
3. Failed validation → PENDING_REVIEW.md
```

## Manual Wake (Testing)

Kích hoạt heartbeat ngay lập tức:

```bash
# Immediate wake
openclaw system event --text "Check for urgent follow-ups" --mode now

# Next scheduled tick
openclaw system event --text "Check for urgent follow-ups" --mode next-heartbeat
```

Nếu nhiều agents có `heartbeat` được cấu hình, manual wake chạy tất cả.

## Heartbeat vs Cron: Khi nào Dùng Cái nào

| Use Case | Heartbeat | Cron |
|----------|-----------|------|
| Nhiều kiểm tra batched | ✅ Yes | ❌ No (một task mỗi cron) |
| Cần conversational context | ✅ Yes | ❌ No (isolated) |
| Thời gian có thể drift (~30 min) | ✅ Yes | ❌ Cần chính xác |
| Lịch trình chính xác yêu cầu | ❌ No | ✅ Yes |
| One-shot reminders | ❌ No | ✅ Yes |
| Model/thinking level khác | ❌ No | ✅ Yes |
| Gửi không qua main session | ❌ No | ✅ Yes |

**Mẹo:** Batch các kiểm tra periodic tương tự vào `HEARTBEAT.md` thay vì tạo nhiều cron jobs.

## Module Liên quan

- Trước đó: [11-creating-agents](../11-creating-agents/)
- Tự động hóa: [06-automation](../06-automation/)
- Cron Jobs: [06-automation](../06-automation/)

---

**Thời gian hoàn thành:** 15 phút
**Yêu cầu trước:** Gateway đang hoạt động, một channel được cấu hình
**Kết quả:** Agent check-in chủ động cho bảo trì và cảnh báo

**Status:** ✅ VALIDATED — Phase 5 Complete
**DocLoop Phase:** 6 Ready (Commit & Report)
