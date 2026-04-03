# 03 - Bộ Nhớ

> Giúp OpenClaw nhớ bạn là ai, bạn thích gì, và bạn đang làm gì.

## Module này giải quyết gì

Không có bộ nhớ, mỗi cuộc trò chuyện bắt đầu từ đầu. Với bộ nhớ, trợ lý của bạn biết sở thích, cuộc trò chuyện trước, và dự án đang làm — khiến nó thực sự hữu ích.

## Khi nào dùng module này

- Bạn muốn phản hồi được cá nhân hóa
- Bạn đang làm dự án dài hạn
- Bạn muốn trợ lý học phong cách của bạn

## Vị trí bộ nhớ

OpenClaw đọc bộ nhớ từ các vị trí này (theo thứ tự ưu tiên):

| Vị trí | Mục đích | Ưu tiên |
|--------|----------|---------|
| `MEMORY.md` | Bộ nhớ dài hạn được chọn lọc | Cao nhất |
| `memory/*.md` | Ghi chú hàng ngày, ngữ cảnh dự án | Cao |
| `~/.openclaw/memory/` | Bộ nhớ toàn cục xuyên workspace | Trung bình |

## Lệnh CLI Bộ nhớ

OpenClaw cung cấp lệnh CLI để quản lý và tìm kiếm bộ nhớ:

```bash
# Kiểm tra trạng thái bộ nhớ
openclaw memory status

# Index file bộ nhớ (làm mới chỉ mục tìm kiếm)
openclaw memory index

# Tìm kiếm bộ nhớ
openclaw memory search "quyết định database"
openclaw memory search --query "chúng ta đã quyết định gì về postgres"
```

## Bắt đầu nhanh

### 1. Kiểm tra trạng thái bộ nhớ

```bash
openclaw memory status
```

Hiển thị file bộ nhớ nào được load và trạng thái của chúng.

### 2. Tạo file bộ nhớ

Tạo file bộ nhớ mà OpenClaw tự động đọc:

```bash
# Tạo thư mục bộ nhớ
mkdir -p ~/.openclaw/memory

# Thêm file bộ nhớ
cat > ~/.openclaw/memory/about-me.md << 'EOF'
# Về tôi

- Tên: [Tên bạn]
- Vị trí: [Nơi bạn ở]
- Công việc: [Bạn làm gì]
- Sở thích:
  - Phong cách giao tiếp: [trực tiếp/chi tiết/thân thiện]
  - Giờ làm việc: [khi nào bạn làm việc]
  - Múi giờ: [múi giờ của bạn]
EOF
```

Trợ lý tự động đọc các file này khi bắt đầu cuộc trò chuyện.

### 3. Index và tìm kiếm

```bash
# Index file bộ nhớ
openclaw memory index

# Tìm kiếm thông tin cụ thể
openclaw memory search "truy vấn của bạn ở đây"
```

## Các pattern bộ nhớ

### Pattern ghi chú hàng ngày

Tạo ghi chú có ngày tháng cho ngữ cảnh đang diễn ra:

```bash
# Tạo ghi chú hôm nay
mkdir -p ~/.openclaw/memory/daily
date +%Y-%m-%d.md

# Cấu trúc:
# ## 2026-03-31
# ### Ưu tiên
# - Ra mắt tính năng mới
# - Review PRs
# 
# ### Quyết định
# - Chọn PostgreSQL thay vì MongoDB
#
# ### Ghi chú
# - Nhớ hỏi về giới hạn API
```

### Bộ nhớ dự án

Giữ ngữ cảnh riêng cho từng dự án:

```bash
mkdir -p ~/.openclaw/memory/projects

# ~/.openclaw/memory/projects/openclaw-101.md
# ## Dự án OpenClaw-101
# 
# ### Mục tiêu
# - Tạo hướng dẫn thân thiện với người mới
# - 10 module học tập
# 
# ### Công nghệ
# - Markdown
# - GitHub Pages
# 
# ### Trạng thái hiện tại
# - Phase 1: Module chính (đang thực hiện)
```

### Bộ nhớ workspace

Cho bộ nhớ riêng dự án, dùng workspace root:

```bash
# Trong workspace dự án của bạn
cat > MEMORY.md << 'EOF'
# Ngữ cảnh dự án

## Mục tiêu hiện tại
Hoàn thành tính năng X vào thứ Sáu

## Quyết định chính
- Dùng Redis cho caching
- Giới hạn API: 100 req/phút

## Team
- Trưởng nhóm: Jane
- Backend: Bob
EOF
```

## Bộ nhớ trong thực tế

### Đề cập bộ nhớ trong chat

```
User: Nhớ rằng tôi thích phản hồi ngắn gọn
Agent: Đã hiểu — tôi sẽ giữ câu trả lời súc tích.

[Cái này có thể được lưu vào file bộ nhớ của bạn]
```

### Truy vấn bộ nhớ

```bash
# Tìm kiếm quyết định trước đó
openclaw memory search "quyết định database"

# Agent cũng có thể tìm kiếm trong cuộc trò chuyện
```

### Cập nhật sở thích

```
User: Thực ra, tôi đã đổi ý về database. Dùng SQLite thay vào đó.
Agent: Đã cập nhật. Tôi đã ghi chú chuyển sang SQLite trong bộ nhớ dự án của bạn.
```

## Ví dụ Copy-Paste

### Mẫu file bộ nhớ

```markdown
# Hồ sơ người dùng

## Cá nhân
- Tên: 
- Vai trò:
- Múi giờ:

## Phong cách giao tiếp
- Độ dài phản hồi: [ngắn gọn/chi tiết]
- Giọng điệu: [chuyên nghiệp/thân mật]
- Định dạng ưa thích: [gạch đầu dòng/đoạn văn]

## Dự án hiện tại
- [Tên dự án]: [mô tả ngắn]

## Ngữ cảnh hữu ích
- [Bất kỳ điều gì agent nên biết]
```

### Tìm kiếm bộ nhớ

```bash
# Tìm kiếm chủ đề cụ thể trong bộ nhớ
openclaw memory search "postgres"

# Tìm kiếm bằng ngôn ngữ tự nhiên
openclaw memory search --query "chúng ta đã chọn database gì"
```

## Lỗi thường gặp và xử lý

| Vấn đề | Cách giải quyết |
|--------|-----------------|
| Bộ nhớ không được đọc | Kiểm tra vị trí file: phải ở `MEMORY.md`, `memory/*.md`, hoặc `~/.openclaw/memory/` |
| Tìm kiếm không trả kết quả | Chạy `openclaw memory index` để làm mới chỉ mục |
| Quá nhiều ngữ cảnh cũ | Lưu trữ ghi chú hàng ngày cũ; chỉ giữ những cái gần đây |
| Thông tin nhạy cảm bị lộ | Dùng kênh bị hạn chế; tách bộ nhớ cá nhân/công việc |

## Pattern nâng cao

### Bộ nhớ có cấu trúc với JSON

Cho truy cập lập trình, bạn cũng có thể dùng JSON:

```json
{
  "user": {
    "name": "Alex",
    "preferences": {
      "notifications": "summarized",
      "timezone": "Asia/Ho_Chi_Minh"
    }
  },
  "projects": {
    "openclaw-101": {
      "status": "active",
      "priority": "high"
    }
  }
}
```

### Tự động hóa index bộ nhớ

Thêm vào shell profile để tự động index khi đăng nhập:

```bash
# ~/.bashrc hoặc ~/.zshrc
alias om="openclaw memory"
alias oms="openclaw memory search"
```

## Module liên quan và bước tiếp theo

- Trước đó: [02-channels](../02-channels/)
- Tiếp theo: [04-skills](../04-skills/) — Mở rộng khả năng với skills
- Liên quan: [Workflows với bộ nhớ](../08-workflows/README.md)

---

**Thời gian hoàn thành:** 30 phút  
**Yêu cầu trước:** [02-channels](../02-channels/)  
**Kết quả:** Trợ lý cá nhân hóa nhớ ngữ cảnh
