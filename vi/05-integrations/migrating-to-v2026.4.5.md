# Chuyển Đổi Lên OpenClaw v2026.4.5

> Thay đổi đột phá: Bí danh cấu hình cũ đã bị xóa. Sử dụng hướng dẫn này để chuyển đổi mà không bị gián đoạn.

**Phiên Bản Giới Thiệu:** v2026.4.5 (6 tháng 4, 2026)  
**Thay Đổi Đột Phá:** Có — yêu cầu hành động khi nâng cấp  
**Lệnh Di Chuyển:** `openclaw doctor --fix`

---

## Có Gì Thay Đổi

OpenClaw v2026.4.5 xóa các bí danh cấu hình công khai cũ để ưu tiên đường dẫn chuẩn. Mặc dù cấu hình cũ vẫn tải được (với cảnh báo), các đường dẫn cũ đã bị ngừng hỗ trợ và sẽ bị xóa trong phiên bản chính sắp tới.

Lệnh `openclaw doctor --fix` xử lý việc di chuyển tự động.

### Các Đường Dẫn Cấu Hình Bị Ảnh Hưởng

| Đường Dẫn Cũ | Đường Dẫn Mới | Thành Phần |
|--------------|---------------|------------|
| `talk.voiceId` | `plugins.entries.tts.config.voiceId` | Chọn giọng TTS |
| `talk.apiKey` | `providers.tts.*.apiKey` | Khóa API nhà cung cấp TTS |
| `agents.*.sandbox.perSession` | `agents.*.sandbox.strategy` | Chiến lược sandbox |
| `browser.ssrfPolicy.allowPrivateNetwork` | `browser.security.allowPrivateNetwork` | Chính sách SSRF trình duyệt |
| `hooks.internal.handlers` | `hooks.handlers` | Trình xử lý hook |
| Công tắc `allow` kênh/nhóm/phòng | Cờ `enabled` | Bật kênh |

---

## Trước Khi Nâng Cấp

### 1. Sao Lưu Cấu Hình

```bash
# Sao lưu cấu hình hiện tại với dấu thời gian
cp ~/.openclaw/openclaw.json ~/.openclaw/openclaw.json.backup.$(date +%Y%m%d)
```

### 2. Kiểm Tra Xem Bạn Có Bị Ảnh Hưởng Không

```bash
# Kiểm tra đường dẫn cấu hình cũ
grep -E "(talk\.(voiceId|apiKey)|perSession|ssrfPolicy|hooks\.internal)" ~/.openclaw/openclaw.json && echo "Cần di chuyển"
```

---

## Các Bước Di Chuyển

### Tùy Chọn 1: Di Chuyển Tự Động (Khuyến Nghị)

```bash
# Nâng cấp OpenClaw trước
npm install -g openclaw@latest

# Xác minh phiên bản
openclaw --version

# Chạy sửa lỗi tự động
openclaw doctor --fix
```

Lệnh `doctor --fix` sẽ:
- Phát hiện bí danh cấu hình cũ
- Di chuyển giá trị sang đường dẫn chuẩn
- Tạo bản sao lưu của cấu hình gốc
- Xác thực cấu hình mới
- Báo cáo các vấn đề cần chú ý thủ công

### Tùy Chọn 2: Di Chuyển Thủ Công

Nếu di chuyển tự động thất bại hoặc bạn muốn kiểm soát:

**Cấu Hình TTS Cũ:**
```json5
{
  talk: {
    voiceId: "nova",
    apiKey: "sk-..."
  }
}
```

**Cấu Hình TTS Mới:**
```json5
{
  plugins: {
    entries: {
      tts: {
        config: {
          voiceId: "nova"
        }
      }
    }
  },
  providers: {
    tts: {
      openai: {
        apiKey: "sk-..."  // hoặc sử dụng biến môi trường OPENAI_API_KEY
      }
    }
  }
}
```

**Cấu Hình Sandbox Cũ:**
```json5
{
  agents: {
    myAgent: {
      sandbox: {
        perSession: true
      }
    }
  }
}
```

**Cấu Hình Sandbox Mới:**
```json5
{
  agents: {
    myAgent: {
      sandbox: {
        strategy: "perSession"  // hoặc "shared", "isolated"
      }
    }
  }
}
```

**Cấu Hình SSRF Trình Duyệt Cũ:**
```json5
{
  browser: {
    ssrfPolicy: {
      allowPrivateNetwork: false
    }
  }
}
```

**Cấu Hình SSRF Trình Duyệt Mới:**
```json5
{
  browser: {
    security: {
      allowPrivateNetwork: false
    }
  }
}
```

**Cấu Hình Hooks Cũ:**
```json5
{
  hooks: {
    internal: {
      handlers: ["default"]
    }
  }
}
```

**Cấu Hình Hooks Mới:**
```json5
{
  hooks: {
    handlers: ["default"]
  }
}
```

**Công Tắc Cho Phép Kênh Cũ:**
```json5
{
  channels: {
    telegram: {
      groups: {
        myGroup: {
          allow: true
        }
      }
    }
  }
}
```

**Cờ Enabled Kênh Mới:**
```json5
{
  channels: {
    telegram: {
      groups: {
        myGroup: {
          enabled: true
        }
      }
    }
  }
}
```

---

## Xác Minh Việc Di Chuyển

### Kiểm Tra Cấu Hình Tải

```bash
# Xác thực cấu hình tải không có cảnh báo
openclaw config validate
```

### Kiểm Tra TTS (nếu đã cấu hình)

```bash
# Kiểm tra chức năng TTS
openclaw tool tts_speak --text "Di chuyển hoàn tất" --provider openai
```

### Kiểm Tra Chiến Lược Sandbox

```bash
# Xác minh cấu hình sandbox đại lý
openclaw config get agents.myAgent.sandbox.strategy
```

### Xem Lại Kết Quả Doctor

```bash
# Chạy doctor không có --fix để xem trạng thái hiện tại
openclaw doctor
```

---

## Xử Lý Sự Cố

### "Cảnh báo xác thực cấu hình sau khi nâng cấp"

**Nguyên nhân**: Còn bí danh cũ trong cấu hình.

**Khắc phục**: Chạy `openclaw doctor --fix` hoặc xóa thủ công đường dẫn cũ.

### "TTS ngừng hoạt động — không có đầu ra giọng nói"

**Nguyên nhân**: `talk.voiceId` và `talk.apiKey` chưa được di chuyển.

**Khắc phục**: Xác minh di chuyển với:
```bash
openclaw config get plugins.entries.tts.config.voiceId
openclaw config get providers.tts.openai.apiKey
```

### "Hành vi sandbox thay đổi bất ngờ"

**Nguyên nhân**: `perSession` boolean được di chuyển sang `strategy` enum.

**Khắc phục**: Kiểm tra giá trị strategy:
```bash
openclaw config get agents.*.sandbox.strategy
```

Giá trị hợp lệ: `"perSession"`, `"shared"`, `"isolated"`

### "Yêu cầu trình duyệt đến mạng riêng thất bại"

**Nguyên nhân**: Đường dẫn `ssrfPolicy` thay đổi thành `security`.

**Khắc phục**: Cập nhật cấu hình trình duyệt:
```json5
{
  browser: {
    security: {
      allowPrivateNetwork: true  // nếu cần cho trường hợp của bạn
    }
  }
}
```

### Lệnh di chuyển thất bại

**Kiểm tra**: Đảm bảo OpenClaw là v2026.4.5 trở lên:
```bash
openclaw --version
```

**Dự phòng**: Sử dụng các bước di chuyển thủ công ở trên.

---

## Hỗ Trợ Biến Môi Trường

Tất cả khóa API vẫn có thể sử dụng biến môi trường thay vì giá trị được mã hóa cứng:

| Đường Dẫn Cấu Hình | Biến Môi Trường |
|-------------------|----------------|
| `providers.tts.openai.apiKey` | `OPENAI_API_KEY` |
| `providers.tts.elevenlabs.apiKey` | `ELEVENLABS_API_KEY` |
| `providers.tts.google.apiKey` | `GOOGLE_API_KEY` |

Ví dụ với biến môi trường:
```json5
{
  providers: {
    tts: {
      openai: {
        // apiKey được bỏ qua — sẽ sử dụng biến môi trường OPENAI_API_KEY
      }
    }
  }
}
```

---

## Danh Sách Kiểm Tra Sau Di Chuyển

- [ ] Sao lưu cấu hình gốc trước khi di chuyển
- [ ] Chạy `openclaw doctor --fix` (hoặc tương đương thủ công)
- [ ] Xác minh `openclaw config validate` vượt qua không có cảnh báo
- [ ] Kiểm tra TTS nếu đã cấu hình trước đó
- [ ] Xác minh giá trị chiến lược sandbox đại lý là chính xác
- [ ] Kiểm tra cài đặt bảo mật trình duyệt nếu sử dụng công cụ trình duyệt
- [ ] Xem lại cấu hình trình xử lý hook
- [ ] Cập nhật `allow` thành `enabled` cho kênh/nhóm nếu áp dụng
- [ ] Cập nhật tài liệu tham khảo đến đường dẫn cấu hình cũ

---

## Có Gì Mới Trong v2026.4.5

Xem các hướng dẫn mới sau đây cho các tính năng được giới thiệu trong phiên bản này:

- [Hướng Dẫn Tạo Video](../../08-workflows/video-generation.md) — Tạo video AI với xAI, Runway và Model Studio
- [Hướng Dẫn Tạo Nhạc](../../08-workflows/music-generation.md) — Nhạc AI với Google Lyria, MiniMax và ComfyUI
- [Amazon Bedrock Mantle](../../05-integrations/bedrock.md#mantle) — Khám phá và thiết lập tự động hồ sơ suy luận
- [Nhà Cung Cấp SearXNG](../../05-integrations/searxng.md) — Tìm kiếm web tập trung vào quyền riêng tư

---

## Tại Sao Thay Đổi Này

Chuẩn hóa đường dẫn cấu hình:
- **Giảm nhầm lẫn** — một đường dẫn chuẩn cho mỗi cài đặt
- **Cho phép xác thực tốt hơn** — mỗi đường dẫn có kiểu nghiêm ngặt
- **Chuẩn bị cho tính năng tương lai** — không gian tên sạch hơn cho các bổ sung sắp tới
- **Cải thiện độ rõ ràng tài liệu** — nguồn duy nhất cho mỗi cấu hình

Việc di chuyển là quá trình một lần. Các phiên bản tương lai sẽ duy trì khả năng tương thích ngược với đường dẫn cấu hình v2026.4.5+.

---

## Tài Liệu Liên Quan

- [Mô-đun 05: Tích Hợp](./README.md)
- [Mô-đun 08: Quy Trình Làm Việc](../../08-workflows/README.md)
- [OpenClaw Doctor](../../10-cli-and-reference/cli-reference.md#doctor)
- [Cấu Hình TTS](../../10-cli-and-reference/config-reference.md#tts)
- [Cấu Hình Sandbox](../../10-cli-and-reference/config-reference.md#sandbox)
- [Cấu Hình Trình Duyệt](../../10-cli-and-reference/config-reference.md#browser)
- [Di Chuyển v2026.4.2](./migrating-to-v2026.4.2.md) — Nếu nâng cấp từ các phiên bản cũ hơn

---

**Cập Nhật Lần Cuối**: 7 tháng 4, 2026  
**Áp Dụng Cho**: OpenClaw v2026.4.5+  
**Di Chuyển Trước**: [v2026.4.2](./migrating-to-v2026.4.2.md)
