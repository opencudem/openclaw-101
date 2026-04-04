# Hướng Dẫn Nâng Cấp Lên OpenClaw v2026.4.2

> Thay đổi đột phá: Đường dẫn cấu hình plugin xAI và Firecrawl đã thay đổi. Sử dụng hướng dẫn này để nâng cấp mà không bị gián đoạn.

---

## Có Gì Thay Đổi

OpenClaw v2026.4.2 thực thi ranh giới nghiêm ngặt giữa cấu hình hệ thống lõi và cấu hình plugin. Trước đây, cấu hình plugin nằm trong không gian tên lõi (`tools.web.*`). Giờ đây chúng nằm trong không gian tên plugin riêng (`plugins.entries.*`).

Đây là **thay đổi đột phá** — đường dẫn cấu hình cũ sẽ bị bỏ qua sau khi nâng cấp.

### Các Tính Năng Bị Ảnh Hưởng

| Tính Năng | Đường Dẫn Cũ | Đường Dẫn Mới | Khóa Xác Thực |
|-----------|--------------|---------------|---------------|
| Tìm Kiếm xAI (Grok) | `tools.web.x_search.*` | `plugins.entries.xai.config.xSearch.*` | `plugins.entries.xai.config.webSearch.apiKey` |
| Trích Xuất Web Firecrawl | `tools.web.fetch.firecrawl.*` | `plugins.entries.firecrawl.config.webFetch.*` | `plugins.entries.firecrawl.config.apiKey` |

---

## Trước Khi Nâng Cấp

### 1. Sao Lưu Cấu Hình

```bash
# Sao lưu cấu hình hiện tại
cp ~/.openclaw/openclaw.json ~/.openclaw/openclaw.json.backup.$(date +%Y%m%d)
```

### 2. Kiểm Tra Cấu Hình Cũ

```bash
# Kiểm tra cấu hình xAI cũ
grep -q "tools.web.x_search" ~/.openclaw/openclaw.json && echo "Cần nâng cấp xAI"

# Kiểm tra cấu hình Firecrawl cũ
grep -q "tools.web.fetch.firecrawl" ~/.openclaw/openclaw.json && echo "Cần nâng cấp Firecrawl"
```

---

## Các Bước Nâng Cấp

### Cách 1: Nâng Cấp Tự Động (Khuyến Nghị)

```bash
# Cập nhật OpenClaw trước
npm install -g openclaw@latest

# Chạy công cụ sửa lỗi tự động
openclaw doctor --fix
```

Lệnh `doctor --fix` sẽ:
- Phát hiện đường dẫn cấu hình cũ
- Di chuyển giá trị sang không gian tên plugin mới
- Sao lưu cấu hình gốc
- Xác thực cấu hình mới

### Cách 2: Nâng Cấp Thủ Công

Nếu công cụ tự động lỗi hoặc bạn muốn kiểm soát:

**Cấu Hình xAI Cũ:**
```json5
{
  tools: {
    web: {
      x_search: {
        enabled: true,
        apiKey: "xai-...",
        model: "grok-2"
      }
    }
  }
}
```

**Cấu Hình xAI Mới:**
```json5
{
  plugins: {
    entries: {
      xai: {
        config: {
          xSearch: {
            enabled: true,
            model: "grok-2"
          },
          webSearch: {
            apiKey: "xai-..."  // hoặc dùng biến môi trường XAI_API_KEY
          }
        }
      }
    }
  }
}
```

**Cấu Hình Firecrawl Cũ:**
```json5
{
  tools: {
    web: {
      fetch: {
        firecrawl: {
          apiKey: "fc-...",
          baseUrl: "https://api.firecrawl.dev"
        }
      }
    }
  }
}
```

**Cấu Hình Firecrawl Mới:**
```json5
{
  plugins: {
    entries: {
      firecrawl: {
        config: {
          webFetch: {
            apiKey: "fc-...",  // hoặc dùng biến môi trường FIRECRAWL_API_KEY
            baseUrl: "https://api.firecrawl.dev"
          }
        }
      }
    }
  }
}
```

---

## Xác Minh Nâng Cấp

### Kiểm Tra Tìm Kiếm xAI

```bash
# Kiểm tra tìm kiếm web với Grok
openclaw tool web_search --query "OpenClaw v2026.4.2 changes" --provider grok
```

### Kiểm Tra Trích Xuất Firecrawl

```bash
# Kiểm tra trích xuất web với Firecrawl
openclaw tool web_fetch --url "https://docs.openclaw.ai" --provider firecrawl
```

### Kiểm Tra Trạng Thái Plugin

```bash
# Xác minh plugin được tải đúng
openclaw plugins list
```

---

## Xử Lý Sự Cố

### "Tìm kiếm xAI ngừng hoạt động sau nâng cấp — không có lỗi"

**Nguyên nhân**: Đường dẫn cấu hình cũ `tools.web.x_search` bị bỏ qua.

**Sửa**: Chạy `openclaw doctor --fix` hoặc nâng cấp thủ công.

### "Firecrawl trả về kết quả rỗng"

**Nguyên nhân**: Đường dẫn cấu hình cũ không được nhận diện.

**Sửa**: Xác minh nâng cấp bằng:
```bash
openclaw config get plugins.entries.firecrawl.config.webFetch.apiKey
```

### Lệnh nâng cấp lỗi

**Kiểm tra**: Đảm bảo OpenClaw đang ở phiên bản v2026.4.2 hoặc mới hơn:
```bash
openclaw --version
```

**Giải pháp**: Dùng các bước nâng cấp thủ công ở trên.

---

## Hỗ Trợ Biến Môi Trường

Bạn vẫn có thể dùng biến môi trường thay vì khóa API trong file:

| Dịch Vụ | Biến Môi Trường |
|---------|-----------------|
| xAI | `XAI_API_KEY` |
| Firecrawl | `FIRECRAWL_API_KEY` |

Ví dụ với biến môi trường:
```json5
{
  plugins: {
    entries: {
      xai: {
        config: {
          xSearch: { enabled: true },
          webSearch: {
            // bỏ qua apiKey — sẽ dùng XAI_API_KEY
          }
        }
      }
    }
  }
}
```

---

## Danh Sách Kiểm Tra Sau Nâng Cấp

- [ ] Đã sao lưu cấu hình gốc
- [ ] Đã chạy `openclaw doctor --fix` (hoặc nâng cấp thủ công)
- [ ] Đã xác minh tìm kiếm xAI hoạt động: `openclaw tool web_search --query test --provider grok`
- [ ] Đã xác minh trích xuất Firecrawl hoạt động (nếu đang dùng)
- [ ] Đã kiểm tra `openclaw plugins list` hiển thị trạng thái đúng
- [ ] Đã xóa các mục `tools.web.x_search` và `tools.web.fetch.firecrawl` cũ khỏi cấu hình
- [ ] Đã cập nhật tài liệu và runbook tham chiếu đường dẫn cũ

---

## Tại Sao Thay Đổi

Việc tách biệt không gian tên plugin:
- **Ngăn phình to cấu hình lõi** khi thêm nhiều plugin
- **Cho phép quản lý phiên bản riêng** cho từng plugin
- **Làm rõ quyền sở hữu** — lõi sở hữu `tools.web.*`, plugin sở hữu `plugins.entries.*`
- **Cho phép xác thực plugin riêng** mà không cần thay đổi hệ thống lõi

---

## Tài Liệu Liên Quan

- [Mô-đun 05: Tích Hợp](./README.md)
- [OpenClaw Doctor](../../10-cli-and-reference/cli-reference.md#doctor)
- [Cấu Hình Tìm Kiếm Web](../../10-cli-and-reference/config-reference.md#web-search)
- [Hướng Dẫn Nâng Cấp Chính Thức (Tiếng Anh)](https://www.xugj520.cn/en/archives/openclaw-2026-migration-configuration-security-task-flow.html)

---

**Cập Nhật Lần Cuối**: 4 tháng 4, 2026  
**Áp Dụng Cho**: OpenClaw v2026.4.2+  
**Thay Đổi Đột Phá**: Có — cần hành động khi nâng cấp
