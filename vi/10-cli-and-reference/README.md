# 10 - CLI và Tham khảo

> Tài liệu tham khảo đầy đủ các lệnh cho thao tác OpenClaw hàng ngày.

## Module này giải quyết gì

Tra cứu nhanh các lệnh bạn dùng mỗi ngày. Không có câu chuyện, chỉ có tài liệu tham khảo.

---

## Gateway Commands

```bash
# Status và điều khiển
openclaw gateway status              # Kiểm tra đang chạy không
openclaw gateway start               # Khởi động Gateway daemon
openclaw gateway stop                # Dừng Gateway
openclaw gateway restart             # Restart Gateway

# Logs
openclaw gateway logs                # Xem logs
openclaw gateway logs --tail 100     # 100 dòng cuối
openclaw gateway logs --follow       # Stream logs
```

---

## Channel Commands

```bash
# List và status
openclaw channels list               # Hiển thị tất cả channels
openclaw channels status             # Kiểm tra channel status
openclaw channels capabilities       # Kiểm tra channels hỗ trợ gì

# Thêm channels
openclaw channels add --channel telegram --token <bot-token>
openclaw channels add --channel discord --token <bot-token>
openclaw channels add --channel slack --token <xoxb-token>
openclaw channels add --channel whatsapp  # QR pairing tương tác

# Xóa channels
openclaw channels remove --channel <name> --delete

# Xác thực
openclaw channels login --channel whatsapp   # QR pairing
openclaw channels logout --channel <name>    # Logout
```

---

## Pairing Commands

```bash
# Liệt kê yêu cầu ghép đôi đang chờ
openclaw pairing list
openclaw pairing list telegram       # Liệt kê cho channel cụ thể

# Phê duyệt yêu cầu ghép đôi
openclaw pairing approve <channel> <code>
openclaw pairing approve telegram ABC123
```

---

## Skill Commands

```bash
# Khám phá
openclaw skills search <query>        # Tìm kiếm ClawHub
openclaw skills list                  # Liệt kê skills đã cài
openclaw skills list --eligible       # Liệt kê skills khả dụng
openclaw skills info <name>           # Lấy thông tin skill
openclaw skills check                 # Kiểm tra vấn đề skill

# Cài đặt/quản lý
openclaw skills install <slug>        # Cài từ ClawHub
openclaw skills install <slug> --version <version>
openclaw skills update <slug>         # Cập nhật skill
openclaw skills update --all          # Cập nhật tất cả skills

# Xóa thủ công từ ~/.openclaw/skills/
```

---

## Agent Commands

```bash
# List và quản lý
openclaw agents list                  # Liệt kê tất cả agents
openclaw agents add <name>            # Thêm agent
openclaw agents delete <name>         # Xóa agent
openclaw agents set-identity <agent> --name "Display Name"

# Bindings
openclaw agents bindings              # Hiển thị agent-channel bindings
openclaw agents bind <agent> --channel <channel>
openclaw agents unbind <agent> --channel <channel>
```

---

## Cron Commands

```bash
# List và status
openclaw cron list                    # Liệt kê tất cả jobs
openclaw cron status                  # Kiểm tra cron status
openclaw cron runs --id <job-id>      # Kiểm tra lịch sử chạy job

# Thêm jobs
openclaw cron add \
  --name "job-name" \
  --cron "0 9 * * *" \
  --message "What to do"

# Job một lần
openclaw cron add \
  --name "reminder" \
  --at "2026-04-05 14:00" \
  --message "Meeting in 15 minutes"

# Điều khiển jobs
openclaw cron enable <job-id>
openclaw cron disable <job-id>
openclaw cron rm <job-id>             # Xóa job
openclaw cron run <job-id>            # Chạy job thủ công

# Sửa jobs
openclaw cron edit <job-id> \
  --announce \
  --channel telegram \
  --to "123456789"
```

---

## Memory Commands

```bash
openclaw memory status                # Kiểm tra memory status
openclaw memory index                 # Index file memory
openclaw memory search "<query>"      # Tìm kiếm memory
openclaw memory search --query "<query>"
```

---

## Config Commands

```bash
# Xem config
openclaw config file                  # Hiển thị đường dẫn file config
openclaw config schema                # Hiển thị config schema
openclaw config get <key>             # Lấy giá trị config
openclaw config get                   # Lấy tất cả config (pretty-printed)

# Set giá trị
openclaw config set <key> <value>
openclaw config set browser.executablePath "/usr/bin/google-chrome"

# Set với SecretRef
openclaw config set channels.discord.token \
  --ref-provider default \
  --ref-source env \
  --ref-id DISCORD_BOT_TOKEN

# Set batch
openclaw config set --batch-file ./config.batch.json

# Unset giá trị
openclaw config unset <key>

# Validate
openclaw config validate
openclaw config validate --json
```

---

## Doctor Commands

```bash
openclaw doctor                       # Chạy diagnostics
openclaw doctor --deep                # Deep check
openclaw doctor --fix                 # Fix issues tự động
```

---

## Security Commands

```bash
openclaw security audit                 # Security audit
openclaw security audit --deep        # Deep audit
openclaw security audit --fix         # Auto-fix issues
```

---

## Browser Commands

```bash
openclaw browser status               # Kiểm tra browser status
openclaw browser start                # Khởi động browser
openclaw browser stop                 # Dừng browser

openclaw browser tabs                 # Liệt kê tabs
openclaw browser open <url>           # Mở URL
openclaw browser close <tab-id>       # Đóng tab

openclaw browser navigate <url>
openclaw browser screenshot
openclaw browser snapshot
openclaw browser click <selector>
openclaw browser type <selector> <text>
openclaw browser fill <selector> <value>
openclaw browser press <key>
openclaw browser wait <ms>
openclaw browser evaluate <script>
```

---

## Plugin Commands

```bash
openclaw plugins list                 # Liệt kê plugins
openclaw plugins list --json          # JSON output
openclaw plugins install <spec>
openclaw plugins uninstall <plugin>
```

---

## Session Commands

```bash
openclaw sessions list                # Liệt kê active sessions
openclaw sessions status              # Kiểm tra session status
openclaw sessions kill <id>           # Kill session
```

---

## Update Commands

```bash
openclaw update                       # Cập nhật lên latest
openclaw update --channel stable      # stable | beta | dev
openclaw migrate                      # Migrate config sau upgrade
```

---

## Global Flags

Các flags này hoạt động với hầu hết các lệnh:

```bash
--dev                     # Dùng dev environment (state riêng)
--profile <name>          # Dùng profile đặt tên (state riêng)
--json                    # Output dạng JSON
--no-color                # Tắt ANSI colors
--verbose                 # Verbose logging
--log-level debug         # Set log level (debug|info|warn|error)
```

---

## Environment Variables

| Variable | Mô tả | Default |
|----------|-------------|---------|
| `OPENCLAW_PORT` | Gateway port | 18789 |
| `OPENCLAW_GATEWAY_TOKEN` | Gateway auth token | - |
| `OPENCLAW_LOG_LEVEL` | Logging level | info |
| `OPENCLAW_CONFIG_PATH` | Vị trí file config | ~/.openclaw/openclaw.json |
| `OPENCLAW_HOME` | Base directory | ~/.openclaw |
| `OPENCLAW_SECRET_*` | Secret references | - |

### Provider API Keys (tham chiếu qua ${VAR} trong config)

```bash
ANTHROPIC_API_KEY=sk-ant-...
OPENAI_API_KEY=sk-...
OPENROUTER_API_KEY=sk-or-...
XAI_API_KEY=...
```

---

## Vị trí File

```
~/.openclaw/
├── openclaw.json           # Main configuration
├── gateway.log             # Gateway logs
├── skills/                 # Custom skills
│   └── my-skill/
│       ├── SKILL.md
│       └── scripts/
├── memory/                 # Memory files
│   ├── about-me.md
│   └── daily/
├── cron/
│   └── jobs.json           # Cron job definitions
├── audit/                  # Audit logs
└── backups/                # Backup files
```

---

## Quick Recipes

### Restart everything fresh

```bash
openclaw gateway stop
openclaw gateway start
openclaw channels list
```

### Debug một channel

```bash
openclaw gateway logs --tail 50
openclaw channels status
```

### Update everything

```bash
openclaw update
openclaw skills update --all
openclaw gateway restart
```

### Export setup của bạn

```bash
# Backup config và skills
tar -czf openclaw-backup-$(date +%Y%m%d).tar.gz \
  ~/.openclaw/openclaw.json \
  ~/.openclaw/skills \
  ~/.openclaw/memory
```

### Import setup của bạn

```bash
tar -xzf openclaw-backup-20260331.tar.gz -C ~/
openclaw gateway restart
```

---

## Tham khảo Cấu hình

```yaml
# ~/.openclaw/openclaw.json (JSON5 format)

{
  // Gateway
  gateway: {
    port: 18789,
    mode: "local",
    bind: "loopback"
  },

  // AI Models
  models: {
    mode: "merge",
    providers: {
      anthropic: {
        apiKey: "${ANTHROPIC_API_KEY}"
      }
    }
  },

  // Agents
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-opus-4-5"
      },
      heartbeat: {
        every: "30m"
      }
    },
    list: []
  },

  // Channels
  channels: {
    telegram: {
      botToken: "${TELEGRAM_BOT_TOKEN}",
      dmPolicy: "pairing"
    }
  },

  // Skills
  skills: {
    entries: {}
  },

  // Cron
  cron: {
    enabled: true
  }
}
```

---

## Exit Codes

| Code | Ý nghĩa |
|------|---------|
| 0 | Success |
| 1 | General error |
| 2 | Invalid arguments |
| 3 | Configuration error |
| 4 | Gateway not running |
| 5 | Permission denied |

---

## Module liên quan

- Trước đó: [09-security-and-ops](../09-security-and-ops/)
- Xử lý lỗi: [TROUBLESHOOTING.md](../../TROUBLESHOOTING.md)
- Full docs: https://docs.openclaw.ai

---

**Thời gian hoàn thành:** 30 phút (reference)  
**Yêu cầu trước:** None (dùng bất cứ lúc nào)  
**Kết quả:** Tra cứu lệnh nhanh cho thao tác hàng ngày
