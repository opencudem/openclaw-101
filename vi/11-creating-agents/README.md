# 11 - Tạo Các Agent Bổ Sung

> Tạo các agent chuyên biệt với workspace riêng cho các tác vụ cụ thể.

## Module này giải quyết gì

Khi đã có agent OpenClaw chính chạy, bạn có thể muốn các agent bổ sung cho mục đích cụ thể:
- **Documentation agent** duy trì wiki của bạn
- **Research agent** giám sát các nguồn và báo cáo phát hiện
- **Coding agent** xử lý các dự án cụ thể
- **Personal assistant** cho các ngữ cảnh khác nhau (work vs. personal)

Mỗi agent có workspace, cấu hình, và bộ nhớ riêng — hoàn toàn cô lập với agent chính.

## Khi nào dùng module này

- Bạn muốn các agent chuyên biệt cho các tác vụ khác nhau
- Bạn cần ngữ cảnh cô lập (work vs. personal)
- Bạn muốn thử nghiệm không ảnh hưởng đến setup chính
- Bạn đang xây dựng các hệ thống tự động hóa

## Yêu cầu

**OpenClaw Version:** v0.31+ (cho wizard `openclaw agents add`)

**Bắt buộc:** Agent chính đang hoạt động ([01-getting-started](../01-getting-started/))

## Bắt đầu nhanh

### Bước 1: Tạo Agent Mới

Dùng agent wizard:

```bash
openclaw agents add work
```

Tạo ra:
- Thư mục agent: `~/.openclaw/agents/work/`
- Thư mục state: `~/.openclaw/agents/work/agent/`
- Session store: `~/.openclaw/agents/work/sessions/`
- Auth profiles: `~/.openclaw/agents/work/agent/auth-profiles.json`

### Bước 2: Cấu hình Workspace

Tạo các file cấu hình bắt buộc trong workspace:

**AGENTS.md** - Identity và scope của agent:
```markdown
# AGENTS.md - Work Agent

## Agent Identity
**Name:** WorkBot
**Purpose:** Xử lý tác vụ work và coding
**Task:** Review PRs hàng ngày, documentation code

## Scope
- Giám sát GitHub repositories
- Tóm tắt thay đổi code
- Post vào kênh Slack work

## Constraints
- Chỉ truy cập work repositories
- Không bao giờ sửa file personal
```

**SOUL.md** - Nguyên tắc cốt lõi:
```markdown
# SOUL.md

## Core Identity
Tôi là WorkBot, tập trung vào các tác vụ development và code review.

## Principles
- Phân tích code kỹ lưỡng
- Documentation rõ ràng
- Security-conscious
```

**USER.md** - Người này agent giúp:
```markdown
# USER.md

## User
**Name:** [Tên bạn]
**Work hours:** 9 AM - 5 PM
**Channel:** Slack #engineering
```

**HEARTBEAT.md** - Tác vụ định kỳ:
```markdown
# HEARTBEAT.md

## Daily Tasks
- [ ] Kiểm tra PRs overnight
- [ ] Tóm tắt thay đổi
- [ ] Post vào Slack
```

**TOOLS.md** - Integrations khả dụng:
```markdown
# TOOLS.md

## GitHub
- Token: [qua biến env GITHUB_TOKEN_WORK]
- Repos: company/backend, company/frontend

## Slack
- Channel: #engineering
- Token: [qua biến env SLACK_TOKEN_WORK]
```

### Bước 3: Thiết lập Channel Bindings

Route các kênh cụ thể đến agent này:

```bash
# Bind work Slack đến work agent
openclaw agents bind --agent work --bind slack:engineering

# Bind work GitHub đến work agent
openclaw agents bind --agent work --bind github:company

# Xác minh bindings
openclaw agents list --bindings
```

### Bước 4: Cấu hình Channel Credentials (QUAN TRỌNG)

**Binding route tin nhắn đến agent, nhưng mỗi agent cần credentials riêng để phản hồi.**

Cho **multi-agent Discord setups**, thêm per-account tokens vào main config (`~/.openclaw/openclaw.json`):

```json5
{
  channels: {
    discord: {
      enabled: true,
      groupPolicy: "allowlist",
      accounts: {
        default: {
          token: "MAIN_BOT_TOKEN_HERE",
          guilds: {
            "YOUR_GUILD_ID": {
              users: ["YOUR_USER_ID"],
              channels: {
                "MAIN_CHANNEL_ID": { allow: true, requireMention: false }
              }
            }
          }
        },
        work: {
          token: "WORK_BOT_TOKEN_HERE",
          guilds: {
            "YOUR_GUILD_ID": {
              channels: {
                "WORK_CHANNEL_ID": { allow: true, requireMention: false }
              }
            }
          }
        }
      }
    }
  }
}
```

**⚠️ Quan trọng:** Mỗi agent cần **Discord bot riêng** (application riêng tại https://discord.com/developers/applications). Chia sẻ tokens giữa các agent gây lỗi routing.

**Tại sao cần cấu trúc này:**
- Binding (`--bind`) = Route tin nhắn **đến** agent
- Per-account `token` trong main config = Cho phép agent gửi phản hồi **đi**
- Main config's `channels.discord.accounts` ưu tiên hơn per-agent auth profiles

**Cho các kênh khác** (Slack, Telegram, v.v.), auth profiles có thể vẫn hoạt động — kiểm tra tài liệu kênh cụ thể.

### Bước 5: Cấu hình Identity

Đặt tên, avatar, và theme của agent:

```bash
openclaw agents set-identity --agent work \
  --name "WorkBot" \
  --emoji "💼" \
  --avatar avatars/workbot.png
```

Hoặc load từ IDENTITY.md:
```bash
openclaw agents set-identity --agent work --from-identity
```

### Bước 6: Restart và Xác minh

```bash
# Restart gateway để áp dụng thay đổi
openclaw gateway restart

# Xác minh agent đang chạy
openclaw agents list --bindings

# Kiểm tra channel status
openclaw channels status --probe
```

## Kiến trúc Multi-Agent

```
Main Agent (default)
├── Workspace: ~/.openclaw/workspace/
├── Agent ID: main
├── State: ~/.openclaw/agents/main/
└── General purpose

Work Agent
├── Workspace: ~/.openclaw/workspace-work/
├── Agent ID: work
├── State: ~/.openclaw/agents/work/
├── Slack: #engineering
├── GitHub: company repos
└── Specialized: Code reviews

Personal Agent
├── Workspace: ~/.openclaw/workspace-personal/
├── Agent ID: personal
├── State: ~/.openclaw/agents/personal/
├── Telegram: @personalbot
└── Specialized: Life admin
```

## Giải thích Cấu trúc Thư mục

```
~/.openclaw/
├── openclaw.json              # Main configuration
├── agents/
│   ├── main/
│   │   ├── agent/             # State và auth
│   │   │   └── auth-profiles.json
│   │   └── sessions/          # Chat history
│   ├── work/
│   │   ├── agent/
│   │   │   └── auth-profiles.json
│   │   └── sessions/
│   └── personal/
│       ├── agent/
│       └── sessions/
├── workspace/                 # Default workspace (main agent)
├── workspace-work/           # Work agent workspace
└── workspace-personal/      # Personal agent workspace
```

**Quy tắc quan trọng:**
- **Không bao giờ reuse agentDir** giữa các agent (gây xung đột auth/session)
- Mỗi agent có **auth profiles riêng** và **sessions cô lập**
- Workspaces là **default cwd**, không phải hard sandboxes

## Giao tiếp Giữa Các Agent

### Pattern 1: File-Based (Shared Memory)

Các agent đọc/ghi file chia sẻ:

```
~/.openclaw/shared/
├── work-tasks.md      # Work agent ghi, main agent đọc
├── research-notes.md  # Research agent ghi, tất cả agents đọc
└── daily-report.md    # Tóm tắt hàng ngày tổng hợp
```

### Pattern 2: Message-Based

Main agent kích hoạt các agent khác qua ngôn ngữ tự nhiên:

```
User: Nhờ work agent review PRs
Main Agent: [Nhắn tin đến channel của work agent]
Work Agent: [Xử lý và phản hồi trực tiếp]
```

### Pattern 3: Cron-Based (Scheduled)

Phối hợp với time offsets:

```bash
# 8:00 AM - Work agent kiểm tra PRs
openclaw cron add \
  --name "work-check-prs" \
  --cron "0 8 * * 1-5" \
  --agent work \
  --message "Check overnight PRs and summarize"

# 8:05 AM - Main agent tổng hợp
openclaw cron add \
  --name "main-digest" \
  --cron "5 8 * * 1-5" \
  --message "Read work-tasks.md and post daily summary"
```

## Ví dụ Thực tế

### Ví dụ 1: Solo Founder Team (4 agents)

| Agent | Vai trò | Tasks | Kênh |
|-------|--------|-------|------|
| Milo | Strategy Lead | Daily standups, thiết lập priority | Telegram @milo |
| Josh | Business | Metrics, giám sát đối thủ | Telegram @josh |
| Marketing | Creative | Content ideas, research trends | Telegram @marketing |
| Dev | Technical | CI/CD monitoring, PR reviews | Telegram @dev |

**Telegram routing:**
- `@milo` → Strategy agent
- `@josh` → Business agent
- `@marketing` → Marketing agent
- `@dev` → Dev agent

### Ví dụ 2: Content Factory (3 agents)

| Agent | Kênh | Task | Lịch trình |
|-------|------|------|-----------|
| Research | Discord #research | Tìm chủ đề trending | 8 AM hàng ngày |
| Writing | Discord #scripts | Viết drafts | 9 AM hàng ngày |
| Design | Discord #thumbnails | Tạo images | 10 AM hàng ngày |

## Best Practices

### 1. Workspace Isolation

Tạo các thư mục riêng biệt:
```bash
mkdir -p ~/.openclaw/workspace-{work,personal,research}
```

### 2. Separate Auth Profiles

Mỗi agent có credentials riêng:
```bash
# Work agent dùng work GitHub token
export GITHUB_TOKEN_WORK="ghp_work_token"

# Personal agent dùng personal token
export GITHUB_TOKEN_PERSONAL="ghp_personal_token"
```

### 3. Skill Scoping

Mỗi workspace có thư mục `skills/` riêng:
```
workspace-work/skills/
├── github-work/       # Work GitHub integration
└── slack-work/        # Work Slack integration

workspace-personal/skills/
├── telegram-personal/ # Personal Telegram
└── calendar-personal/ # Personal calendar
```

### 4. Clear Boundaries

Document agent nào làm gì:
```markdown
# AGENTS.md

## Agent này LÀM:
- Giám sát work GitHub repos
- Post vào #engineering Slack
- Tóm tắt thay đổi code

## Agent này KHÔNG LÀM:
- Truy cập personal repositories
- Gửi emails
- Sửa file main agent
```

## Các Lỗi Thường Gặp

| Lỗi | Tại sao Xấu | Giải pháp |
|-----|-------------|-----------|
| Reusing agentDir | Xung đột auth/session | Dùng `openclaw agents add` cho mỗi agent |
| Chia sẻ workspaces | Memory bleed | Thư mục riêng biệt |
| **Same channel tokens** | **Cross-routing chaos** | **Một token cho mỗi agent mỗi kênh** |
| **Missing auth-profiles.json** | **Agent không thể phản hồi** | **Tạo `~/.openclaw/agents/<name>/agent/auth-profiles.json`** |
| Không có bindings | Tất cả agents nhận tất cả tin nhắn | Explicit `agents bind` |
| Unclear scope | Agents overlap | Document trong AGENTS.md |

## Xử lý Lỗi: Agent Không Phản hồi trong Kênh

**Vấn đề:** Bạn đã bind agent đến Discord, nhưng main agent vẫn phản hồi (hoặc không có phản hồi).

**Các bước chẩn đoán:**

**Bước 1: Kiểm tra binding tồn tại**
```bash
openclaw agents list --bindings
# Nên hiển thị: work → discord:work (account-based binding)
```

**Bước 2: Xác minh main config có per-account token**
```bash
cat ~/.openclaw/openclaw.json | grep -A5 '"work"'
# Nên hiển thị trường "token" dưới accounts.work
```

**Bước 3: Thêm per-account token vào main config**

Token phải trong `~/.openclaw/openclaw.json` dưới `channels.discord.accounts.<accountId>.token`:

```json5
{
  channels: {
    discord: {
      accounts: {
        work: {
          token: "WORK_BOT_TOKEN_HERE",
          guilds: { ... }
        }
      }
    }
  }
}
```

**⚠️ Lưu ý:** Per-agent `auth-profiles.json` KHÔNG hoạt động cho Discord — main config's `channels.discord.accounts` ưu tiên hơn.

**Bước 4: Lấy Discord bot token riêng**
- Vào https://discord.com/developers/applications
- Tạo **New Application** (không reuse bot hiện có)
- Bot → Add Bot → Copy Token
- Bật **Message Content Intent**
- Mời bot mới vào Discord server

**Bước 5: Restart gateway**
```bash
openclaw gateway restart
```

**Tại sao xảy ra:**
- **Binding** (`--bind`) = Route tin nhắn **đến** agent ✅
- **Per-account token trong main config** = Cho phép phản hồi **đi** ❌ (thường bị quên!)
- Mỗi agent cần **bot token riêng** để gửi tin nhắn
- Main config ưu tiên hơn per-agent auth profiles cho Discord

## CLI Reference

```bash
# Tạo và quản lý agents
openclaw agents add <name>                    # Tạo agent mới
openclaw agents list                          # Liệt kê tất cả agents
openclaw agents list --bindings              # Liệt kê với channel bindings
openclaw agents delete <name>                 # Xóa agent

# Cấu hình identity
openclaw agents set-identity --agent <name> \
  --name "Display Name" \
  --emoji "💼" \
  --avatar avatars/bot.png

# Channel routing
openclaw agents bind --agent <name> --bind <channel:account>
openclaw agents unbind --agent <name> --bind <channel:account>
openclaw agents bindings                      # Hiển thị tất cả bindings

# Xác minh
openclaw channels status --probe              # Kiểm tra channel health
```

## Nâng cao: Một WhatsApp, Nhiều Người

Route các WhatsApp DM khác nhau đến các agent khác nhau:

```json
{
  "agents": {
    "list": [
      { "id": "alex", "workspace": "~/.openclaw/workspace-alex" },
      { "id": "mia", "workspace": "~/.openclaw/workspace-mia" }
    ]
  },
  "bindings": [
    {
      "agentId": "alex",
      "match": { "channel": "whatsapp", "peer": { "kind": "direct", "id": "+84123456789" } }
    },
    {
      "agentId": "mia",
      "match": { "channel": "whatsapp", "peer": { "kind": "direct", "id": "+84987654321" } }
    }
  ],
  "channels": {
    "whatsapp": {
      "dmPolicy": "allowlist",
      "allowFrom": ["+84123456789", "+84987654321"]
    }
  }
}
```

## Module Liên quan

- Trước đó: [10-cli-and-reference](../10-cli-and-reference/)
- Bảo mật: [09-security-and-ops](../09-security-and-ops/)
- Tự động hóa: [06-automation](../06-automation/)
- Tích hợp: [05-integrations](../05-integrations/)

---

**Thời gian hoàn thành:** 45 phút  
**Yêu cầu trước:** [01-getting-started](../01-getting-started/), OpenClaw v0.31+  
**Kết quả:** Nhiều agent chuyên biệt với workspace cô lập và workflow phối hợp
