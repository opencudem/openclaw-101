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

Tạo các file cấu hình bắt buộc trong workspace.

### Bước 3: Thiết lập Channel Bindings

Route các kênh cụ thể đến agent này:

```bash
# Bind work Slack đến work agent
openclaw agents bind --agent work --bind slack:engineering

# Xác minh bindings
openclaw agents list --bindings
```

### Bước 4: Cấu hình Channel Credentials

Mỗi agent cần credentials riêng để phản hồi.

### Bước 5: Cấu hình Identity

```bash
openclaw agents set-identity --agent work --name "WorkBot" --emoji "💼"
```

### Bước 6: Restart và Xác minh

```bash
openclaw gateway restart
openclaw agents list --bindings
```

## Kiến trúc Multi-Agent

Các agent cô lập với workspace, state, và sessions riêng.

## CLI Reference

```bash
openclaw agents add <name>
openclaw agents list --bindings
openclaw agents bind --agent <name> --bind <channel:account>
```

---

**Thời gian hoàn thành:** 45 phút  
**Yêu cầu trước:** [01-getting-started](../01-getting-started/), OpenClaw v0.31+  
**Kết quả:** Nhiều agent chuyên biệt với workspace cô lập
