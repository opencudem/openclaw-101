# 09 - Bảo mật và Vận hành

> Bảo mật OpenClaw deployment với DM policies, pairing, và security auditing.

## Module này giải quyết gì

Automation mạnh mẽ cần guardrails. Module này bao gồm các tính năng bảo mật tích hợp: DM policies, pairing, security audits, và best practices vận hành.

## Khi nào dùng module này

- Bạn chạy OpenClaw trong production
- Bạn xử lý dữ liệu nhạy cảm
- Bạn cần kiểm soát ai có thể nhắn tin cho agent
- Bạn muốn audit deployment của mình

## Các Lớp Bảo mật

```
┌─────────────────────────────────────────┐
│           PERIMETER                     │
│  - DM policies (pairing/allowlist)      │
│  - Channel allowlists                   │
├─────────────────────────────────────────┤
│           ACCESS CONTROL                │
│  - Pairing system                       │
│  - Group policies                       │
│  - Sandbox mode                         │
├─────────────────────────────────────────┤
│           CREDENTIALS                   │
│  - SecretRef cho secrets                │
│  - Environment variables                │
├─────────────────────────────────────────┤
│           AUDIT                         │
│  - Security audit command               │
│  - Doctor diagnostics                   │
└─────────────────────────────────────────┘
```

## Bắt đầu nhanh

### Security Audit

Chạy security audit để kiểm tra cấu hình:

```bash
# Basic security audit
openclaw security audit

# Deep audit với checks chi tiết
openclaw security audit --deep

# Auto-fix issues khi có thể
openclaw security audit --fix
```

### Doctor Diagnostics

Kiểm tra các issues phổ biến và vấn đề bảo mật:

```bash
# Chạy diagnostics
openclaw doctor

# Deep check
openclaw doctor --deep

# Fix issues tự động
openclaw doctor --fix
```

## DM Policies (Bảo mật Tin nhắn Trực tiếp)

Kiểm soát ai có thể gửi tin nhắn trực tiếp đến agent:

### Các Loại Policy

| Policy | Hành vi | Mức độ Bảo mật |
|--------|---------|----------------|
| `pairing` | Phê duyệt một lần qua mã ghép đôi | Cao |
| `allowlist` | Chỉ ID được chỉ định có thể nhắn tin | Trung bình |
| `open` | Ai cũng có thể nhắn tin (cẩn thận) | Thấp |
| `disabled` | Chặn tất cả DMs | Tối đa |

### Ví dụ Cấu hình

**Chế độ Pairing (Khuyến nghị cho production):**
```yaml
# ~/.openclaw/config.yaml
channels:
  telegram:
    botToken: "${TELEGRAM_BOT_TOKEN}"
    dmPolicy: pairing  # Yêu cầu phê duyệt một lần
```

Người dùng phải yêu cầu mã ghép đôi và bạn phải phê duyệt:
```bash
# Liệt kê các yêu cầu ghép đôi đang chờ
openclaw pairing list telegram

# Phê duyệt yêu cầu
openclaw pairing approve telegram ABC123
```

**Chế độ Allowlist:**
```yaml
channels:
  discord:
    botToken: "${DISCORD_BOT_TOKEN}"
    dmPolicy: allowlist
    allowFrom:
      - "discord:123456789"      # Discord ID của bạn
      - "discord:987654321"      # Bạn bè tin cậy
```

**Chế độ Disabled (chỉ channels):**
```yaml
channels:
  slack:
    botToken: "${SLACK_BOT_TOKEN}"
    dmPolicy: disabled  # Không cho phép DMs
```

## Group Policies

Cho các kênh hỗ trợ nhóm (WhatsApp, v.v.), kiểm soát truy cập nhóm:

```yaml
channels:
  whatsapp:
    dmPolicy: allowlist
    allowFrom:
      - "+14155552671"  # Số điện thoại của bạn
    groupPolicy: allowlist  # Tùy chọn: allowlist, open, disabled
    groupAllowFrom:
      - "group:123456789@g.us"  # ID nhóm cụ thể
```

## Channel Allowlists

Hạn chế các kênh nào có thể truy cập agent:

```yaml
# ~/.openclaw/config.yaml
security:
  allowed_channels:
    - "telegram:12345678"      # Chat riêng của bạn
    - "discord:987654321"      # Server riêng của bạn
    - "slack:U123456789"       # Slack user của bạn
```

## Secure Credentials với SecretRef

Không bao giờ lưu secrets plain text. Dùng SecretRef:

```yaml
# ~/.openclaw/config.yaml
channels:
  telegram:
    botToken:
      secretRef: TELEGRAM_BOT_TOKEN  # Đọc từ OPENCLAW_SECRET_TELEGRAM_BOT_TOKEN
  discord:
    botToken:
      secretRef: DISCORD_BOT_TOKEN

skills:
  entries:
    github:
      config:
        token:
          secretRef: GITHUB_TOKEN
```

**Đặt biến môi trường:**
```bash
export OPENCLAW_SECRET_TELEGRAM_BOT_TOKEN="your-token-here"
export OPENCLAW_SECRET_DISCORD_BOT_TOKEN="your-token-here"
export OPENCLAW_SECRET_GITHUB_TOKEN="ghp_xxxxxxxx"
```

## Sandboxing

Kiểm soát sandbox restrictions của agent:

```yaml
# ~/.openclaw/config.yaml
agents:
  defaults:
    sandbox:
      mode: restricted  # Tùy chọn: full, restricted, off
      network: false    # Tắt network access
      filesystem: read-only  # Filesystem chỉ đọc
```

| Mode | Mô tả |
|------|-------|
| `full` | Full sandbox restrictions |
| `restricted` | Partial restrictions (mặc định) |
| `off` | Không sandbox (nguy hiểm) |

## Ví dụ Copy-Paste

### Production Security Checklist

```yaml
# ~/.openclaw/config.yaml - Production Hardening

# 1. Dùng pairing cho tất cả DM channels
channels:
  telegram:
    botToken:
      secretRef: TELEGRAM_BOT_TOKEN
    dmPolicy: pairing
  
  discord:
    botToken:
      secretRef: DISCORD_BOT_TOKEN
    dmPolicy: pairing

# 2. Hạn chế allowed_channels
security:
  allowed_channels:
    - "telegram:${TELEGRAM_USER_ID}"
    - "discord:${DISCORD_USER_ID}"

# 3. Bật sandboxing
agents:
  defaults:
    sandbox:
      mode: restricted

# 4. Dùng SecretRef cho tất cả credentials
```

### Pairing Approval Workflow

```bash
# 1. User gửi tin nhắn đến bot
# 2. Bot phản hồi với mã ghép đôi: "Reply with: pair ABC123"
# 3. User trả lời: pair ABC123
# 4. Admin kiểm tra yêu cầu đang chờ:
openclaw pairing list telegram

# 5. Admin phê duyệt yêu cầu:
openclaw pairing approve telegram ABC123

# 6. User có thể nhắn tin tự do
```

### Security Audit Script

```bash
#!/bin/bash
# ~/.openclaw/scripts/security-check.sh

echo "Running OpenClaw security checks..."

# Chạy security audit
openclaw security audit

# Kiểm tra các issues phổ biến
openclaw doctor

# Liệt kê tất cả các kênh đã cấu hình
openclaw channels list

echo "Security check complete!"
```

## Các Pattern Vận hành

### Health Checks

```bash
# Thêm health check tự động
openclaw cron add \
  --name "security-check" \
  --cron "0 */6 * * *" \
  --message "Run openclaw doctor and report any issues" \
  --session isolated
```

### Backup Configuration

```bash
# Backup script cho OpenClaw config
cat > ~/.openclaw/scripts/backup.sh << 'EOF'
#!/bin/bash
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="$HOME/.openclaw/backups/$DATE"

mkdir -p "$BACKUP_DIR"
cp "$HOME/.openclaw/openclaw.json" "$BACKUP_DIR/"
cp -r "$HOME/.openclaw/skills" "$BACKUP_DIR/"
cp -r "$HOME/.openclaw/memory" "$BACKUP_DIR/"

echo "Backup saved to $BACKUP_DIR"
EOF

chmod +x ~/.openclaw/scripts/backup.sh
```

### Recovery Procedures

```markdown
# ~/.openclaw/RECOVERY.md

## Gateway Không Khởi động

1. Kiểm tra logs: `openclaw gateway logs --tail 100`
2. Kiểm tra xung đột port: `lsof -i :18789`
3. Chạy diagnostics: `openclaw doctor`
4. Fix issues: `openclaw doctor --fix`
5. Restore từ backup: `cp ~/.openclaw/backups/latest/openclaw.json ~/.openclaw/`

## Mất Kết nối Kênh

1. Kiểm tra token hợp lệ
2. Xác minh bot vẫn active trong kênh
3. Thêm lại kênh: `openclaw channels remove --channel <name> --delete`
4. Sau đó: `openclaw channels add --channel <name> --token <new-token>`

## Nghi ngờ Security Breach

1. Ngay lập tức disable kênh: Đổi dmPolicy thành disabled
2. Revoke tất cả tokens
3. Chạy security audit: `openclaw security audit --deep`
4. Kiểm tra pairing list: `openclaw pairing list <channel>`
5. Xóa các ghép đôi không được phép
```

## Lỗi thường gặp và xử lý

| Vấn đề | Cách giải quyết |
|--------|-----------------|
| Truy cập trái phép | Đặt dmPolicy thành pairing; xem xét lại allowed_channels |
| Token lộ trong config | Dùng SecretRef thay vì plain text |
| Pairing spam | Đặt dmPolicy thành allowlist với các ID cụ thể |
| Gateway security issues | Chạy `openclaw security audit --fix` |
| Sandbox chặn skills | Kiểm tra sandbox.mode setting |
| Secrets không load | Xác minh biến môi trường OPENCLAW_SECRET_* |

## Best Practices Bảo mật

1. **Luôn dùng pairing hoặc allowlist** cho DM policies trong production
2. **Không bao giờ commit tokens** - dùng SecretRef với environment variables
3. **Hạn chế allowed_channels** chỉ cho các account cá nhân của bạn
4. **Chạy security audits định kỳ** - `openclaw security audit`
5. **Cập nhật OpenClaw** - `openclaw update`
6. **Backup định kỳ** - Giữ backup của openclaw.json
7. **Xem xét skills trước khi cài** - `openclaw skills check`
8. **Dùng sandboxing** - Không tắt sandbox trong production

## Module liên quan và bước tiếp

- Trước đó: [08-workflows](../08-workflows/)
- Tiếp theo: [10-cli-and-reference](../10-cli-and-reference/)
- Tham khảo: [Security documentation](https://docs.openclaw.ai/gateway/security)

---

**Thời gian hoàn thành:** 45 phút  
**Yêu cầu trước:** [08-workflows](../08-workflows/)  
**Kết quả:** OpenClaw deployment production-ready với access controls phù hợp
