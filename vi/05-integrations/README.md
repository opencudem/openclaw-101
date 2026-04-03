# 05 - Tích hợp

> Kết nối OpenClaw với công cụ và dịch vụ bên ngoài: GitHub, Lịch, Email, và hơn nữa.

## Module này giải quyết gì

Tích hợp cho phép OpenClaw tương tác với dịch vụ bên ngoài của bạn - GitHub repos, Google Calendar, tài khoản email, database, và APIs. Module này bao gồm cách cấu hình các kết nối này một cách an toàn.

## Khi nào dùng module này

- Bạn muốn tự động hóa workflow GitHub (PRs, issues, CI)
- Bạn cần truy cập lịch (Google, iCloud, Outlook)
- Bạn muốn gửi/nhận email
- Bạn cần kết nối database
- Bạn muốn tích hợp API (Slack, Notion, v.v.)

## Cách Tích hợp Hoạt động

Tích hợp được cấu hình qua:
1. **Biến môi trường** - Cho API keys và tokens
2. **OpenClaw config** - `~/.openclaw/config.yaml` hoặc `openclaw.json`
3. **Skill configs** - File cấu hình riêng cho từng skill

Agent dùng các credentials này để xác thực với dịch vụ bên ngoài.

---

## Cơ bản về Cấu hình

### 1. Biến Môi trường (Khuyến nghị cho tokens)

Lưu credentials nhạy cảm trong shell:

```bash
# ~/.bashrc hoặc ~/.zshrc
export GITHUB_TOKEN="ghp_your_token_here"
export OPENAI_API_KEY="sk-your_key_here"
export ANTHROPIC_API_KEY="sk-ant-your_key_here"
export SMTP_PASSWORD="your_email_password"
export CALENDAR_PASSWORD="your_calendar_app_password"
```

Sau đó reload:
```bash
source ~/.bashrc
```

### 2. SecretRef trong Config (An toàn nhất)

Dùng `secretRef` để tham chiếu biến môi trường trong config:

```yaml
# ~/.openclaw/config.yaml
models:
  providers:
    anthropic:
      apiKey:
        secretRef: ANTHROPIC_API_KEY  # Đọc từ env var
        
skills:
  entries:
    github:
      config:
        token:
          secretRef: GITHUB_TOKEN
```

**Định dạng SecretRef:**
```yaml
key:
  secretRef: ENV_VAR_NAME  # Agent tìm OPENCLAW_SECRET_ENV_VAR_NAME hoặc ENV_VAR_NAME
```

### 3. Config Trực tiếp (Không khuyến nghị cho production)

Chỉ cho config không nhạy cảm:

```yaml
# ~/.openclaw/config.yaml
gateway:
  port: 18789
  
tools:
  web:
    search:
      provider: brave
```

---

## Các Tích hợp đã được Xác minh

Các tích hợp này đã được kiểm tra và test:

### Tích hợp GitHub

**Làm gì:** Truy cập GitHub repos, PRs, issues, CI status qua `gh` CLI

**Thiết lập:**
```bash
# 1. Cài gh CLI
brew install gh  # macOS
# hoặc apt install gh  # Linux

# 2. Xác thực
gh auth login

# 3. Đặt token trong env (nếu dùng trong container/headless)
export GITHUB_TOKEN="ghp_your_token"
```

**Dùng qua chat:**
```
User: Liệt kê PR đang mở của tôi trong opencudem/openclaw-101
Agent: [Dùng gh pr list để hiển thị PRs]

User: Kiểm tra trạng thái PR #42
Agent: [Dùng gh pr view 42 --json state,checks]

User: Comment "LGTM" trên PR #42
Agent: [Dùng gh pr comment 42 --body "LGTM"]
```

---

### Tích hợp Lịch (CalDAV)

**Làm gì:** Đọc/ghi sự kiện lịch qua giao thức CalDAV

**Thiết lập:**
```bash
# 1. Cài binary cần thiết
pip install vdirsyncer
pip install khal

# 2. Cấu hình vdirsyncer
# ~/.config/vdirsyncer/config
[pair my_calendar]
a = my_local
b = my_remote
collections = ["from a", "from b"]

[storage my_local]
type = filesystem
path = ~/.calendars/
fileext = .ics

[storage my_remote]
type = caldav
url = https://caldav.fastmail.com/dav/calendars/
username = your@email.com
password = your_app_password
```

**Dùng qua chat:**
```
User: Hôm nay lịch của tôi có gì?
Agent: [Dùng khal list today để hiển thị sự kiện]

User: Thêm cuộc họp ngày mai lúc 2pm với Alice
Agent: [Dùng khal new để tạo sự kiện]
```

---

### Tích hợp Email (IMAP/SMTP)

**Làm gì:** Đọc inbox và gửi email

**Thiết lập:**
```bash
# Đặt credentials trong environment
export EMAIL_IMAP_SERVER="imap.gmail.com"
export EMAIL_IMAP_USER="you@gmail.com"
export EMAIL_IMAP_PASS="your_app_password"
export EMAIL_SMTP_SERVER="smtp.gmail.com"
export EMAIL_SMTP_USER="you@gmail.com"
export EMAIL_SMTP_PASS="your_app_password"
```

**Dùng qua chat:**
```
User: Kiểm tra email chưa đọc của tôi
Agent: [Kết nối IMAP, lấy số chưa đọc]

User: Gửi email đến team@company.com về bản release
Agent: [Soạn qua chat, gửi qua SMTP]
```

---

### Tích hợp Tìm kiếm Web

**Làm gì:** Tìm kiếm web qua Brave, Google, hoặc provider khác

**Thiết lập:**
```bash
# Cho Brave Search
export BRAVE_API_KEY="your_brave_key"

# Cho Google Custom Search
export GOOGLE_SEARCH_API_KEY="your_key"
export GOOGLE_SEARCH_CX="your_cx"
```

**Cấu hình trong openclaw.json:**
```json
{
  "tools": {
    "web": {
      "search": {
        "provider": "brave",
        "apiKey": { "secretRef": "BRAVE_API_KEY" }
      }
    }
  }
}
```

**Dùng qua chat:**
```
User: Tìm tutorial OpenClaw
Agent: [Dùng web_search để tìm kết quả]

User: Framework AI agent mới nhất là gì?
Agent: [Tìm kiếm, trích xuất, tóm tắt]
```

---

## Các Pattern Cấu hình Tích hợp

### Pattern 1: Biến Môi trường Đơn giản

Cho thiết lập nhanh:

```bash
export API_KEY="your_key"
```

Sau đó trong chat:
```
User: Dùng API key của tôi để kiểm tra thời tiết
Agent: [Đọc API_KEY từ env, gọi API]
```

### Pattern 2: SecretRef với Fallback

Cho production:

```yaml
# config.yaml
integrations:
  github:
    token:
      secretRef: GITHUB_TOKEN
    # Fallback to direct value (không khuyến nghị)
    # token: "ghp_xxx"
```

### Pattern 3: Nhiều Tích hợp

Cấu hình nhiều cái cùng lúc:

```yaml
# config.yaml
models:
  providers:
    anthropic:
      apiKey: { secretRef: ANTHROPIC_API_KEY }
    openai:
      apiKey: { secretRef: OPENAI_API_KEY }

integrations:
  github: { token: { secretRef: GITHUB_TOKEN } }
  calendar: { password: { secretRef: CALENDAR_PASSWORD } }
  email: { 
    imap: { user: "you@gmail.com", pass: { secretRef: EMAIL_PASSWORD } },
    smtp: { user: "you@gmail.com", pass: { secretRef: EMAIL_PASSWORD } }
  }
```

---

## Kiểm tra Tích hợp

### Test GitHub
```bash
gh auth status
gh repo list
```

### Test Lịch
```bash
khal list today
```

### Test Email
```bash
# Dùng openclaw để test
openclaw config get tools.email.imap.user
```

---

## Lỗi Cấu hình Thường Gặp

| Lỗi | Cách giải quyết |
|-----|-----------------|
| Token trong config plain text | Dùng `secretRef` thay thế |
| Sai tên biến env | Kiểm tra `openclaw config get <key>` |
| Thiếu `export` trong .bashrc | Đảm bảo vars được export, không chỉ set |
| API key không đúng scopes | Xác minh token có quyền cần thiết |
| Credentials trong git repo | Thêm vào `.gitignore`, dùng env vars |

---

## Best Practices Bảo mật

1. **Không commit credentials** - Luôn dùng env vars hoặc SecretRef
2. **Dùng app passwords** - Cho email/calendar, không phải password chính
3. **Xoay token định kỳ** - Đặt lịch nhắc xoay API keys
4. **Scope token hẹp** - GitHub tokens nên có quyền tối thiểu
5. **Chạy `openclaw security audit`** - Chạy định kỳ để tìm credentials lộ

---

## Module Liên quan và Bước Tiếp

- Trước đó: [04-skills](../04-skills/) - Cách cài đặt và dùng skills
- Tiếp theo: [06-automation](../06-automation/) - Tự động hóa với cron
- Bảo mật: [09-security-and-ops](../09-security-and-ops/) - Bảo mật production
- Tham khảo: `openclaw config --help`

---

**Thời gian hoàn thành:** 30 phút  
**Yêu cầu trước:** [04-skills](../04-skills/)  
**Kết quả:** Dịch vụ bên ngoài được kết nối qua cấu hình an toàn
