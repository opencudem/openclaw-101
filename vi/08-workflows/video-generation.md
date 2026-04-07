# Tạo Video Bằng AI

> Tạo video từ lời nhắc văn bản và hình ảnh bằng công cụ video_generate tích hợp của OpenClaw. Hỗ trợ xAI (Grok), Runway và Alibaba Model Studio.

**Phiên Bản Giới Thiệu:** v2026.4.5 (6 tháng 4, 2026)  
**Công Cụ:** `video_generate`  
**Nhà Cung Cấp:** xAI, Runway, Model Studio (Wan)

---

## Tổng Quan

Công cụ `video_generate` cho phép đại lý tạo video ngắn thông qua các nhà cung cấp được cấu hình. Video được tạo không đồng bộ và được giao khi sẵn sàng.

**Các Trường Hợp Sử Dụng:**
- Tạo nội dung (clip mạng xã hội, video xem trước)
- Kể chuyện trực quan từ mô tả văn bản
- Biến thể video từ hình ảnh tham chiếu
- Tạo mẫu và trực quan hóa khái niệm

**Các Nhà Cung Cấp Được Hỗ Trợ:**
| Nhà Cung Cấp | Mô Hình | Tính Năng |
|--------------|---------|-----------|
| xAI | `grok-imagine-video` | Văn bản-thành-video, tạo nhanh |
| Runway | `gen-3-alpha` | Chất lượng cao, hỗ trợ hình ảnh-thành-video |
| Model Studio | `wan-2.1` | Mô hình video Wan mã nguồn mở |

---

## Bắt Đầu Nhanh

### 1. Cấu Hình Nhà Cung Cấp Video

**xAI (Grok):**
```json5
{
  providers: {
    video: {
      xai: {
        apiKey: "xai-...",  // hoặc sử dụng biến môi trường XAI_API_KEY
        model: "grok-imagine-video"
      }
    }
  }
}
```

**Runway:**
```json5
{
  providers: {
    video: {
      runway: {
        apiKey: "rw-...",  // hoặc sử dụng biến môi trường RUNWAY_API_KEY
        model: "gen-3-alpha"
      }
    }
  }
}
```

**Alibaba Model Studio:**
```json5
{
  providers: {
    video: {
      modelstudio: {
        apiKey: "sk-...",  // hoặc sử dụng biến môi trường MODELSTUDIO_API_KEY
        model: "wan-2.1",
        baseUrl: "https://api.modelstudio.ai/v1"
      }
    }
  }
}
```

### 2. Tạo Video

```bash
# Văn bản-thành-video
openclaw tool video_generate --prompt "Một hồ núi thanh bình lúc bình minh, sương mù từ mặt nước" --provider xai

# Hình ảnh-thành-video (Runway)
openclaw tool video_generate --prompt "Camera chậm di chuyển sang phải qua cảnh quan" --image ./landscape.jpg --provider runway
```

### 3. Kiểm Tra Trạng Thái Tạo

```bash
# Liệt kê các tác vụ video đang chờ
openclaw tasks list --type video

# Lấy trạng thái tác vụ cụ thể
openclaw tasks get <task-id>
```

---

## Cấu Hình Đại Lý

Bật tạo video cho các đại lý cụ thể:

```json5
{
  agents: {
    contentCreator: {
      tools: {
        video_generate: {
          enabled: true,
          defaultProvider: "xai",  // nhà cung cấp dự phòng
          maxDurationSeconds: 10,
          requireConfirmation: false  // đặt true cho các thao tác tốn kém
        }
      }
    }
  }
}
```

**Cách Sử Dụng Đại Lý:**
```
Người Dùng: Tạo video 5 giây về robot nhảy múa
Đại Lý: [sử dụng công cụ video_generate với lời nhắc "Một robot hình người nhảy múa năng động trong studio"]
```

---

## Tham Chiếu Công Cụ

### video_generate

Tạo video từ lời nhắc văn bản và hình ảnh tham chiếu tùy chọn.

**Tham Số:**

| Tham Số | Kiểu | Bắt Buộc | Mô Tả |
|---------|------|----------|-------|
| `prompt` | chuỗi | Có | Mô tả văn bản của video mong muốn |
| `provider` | chuỗi | Không | Ghi đè nhà cung cấp (xai, runway, modelstudio) |
| `image` | chuỗi | Không | Đường dẫn đến hình ảnh tham chiếu cho hình ảnh-thành-video |
| `durationSeconds` | số | Không | Thời lượng mục tiêu (5-10 giây, phụ thuộc nhà cung cấp) |
| `aspectRatio` | chuỗi | Không | "16:9", "9:16", "1:1" (phụ thuộc nhà cung cấp) |

**Phản Hồi:**
```json
{
  "taskId": "vid_abc123",
  "status": "pending",
  "estimatedSeconds": 45,
  "provider": "xai"
}
```

**Giao Hàng Không Đồng Bộ:**
Khi hoàn tất, tệp video được giao:
- Trong chat: dưới dạng đính kèm video
- Qua webhook: với URL tải xuống
- Qua API: điểm cuối trạng thái bao gồm URL tải xuống

---

## Chi Tiết Theo Nhà Cung Cấp

### xAI (Grok Imagine Video)

**Tốt nhất cho:** Tạo nhanh, video đa mục đích

**Cấu Hình:**
```json5
{
  providers: {
    video: {
      xai: {
        apiKey: "${XAI_API_KEY}",
        model: "grok-imagine-video",
        defaultDuration: 5,
        defaultAspectRatio: "16:9"
      }
    }
  }
}
```

**Giới Hạn:**
- Thời lượng tối đa: 5 giây
- Tỷ lệ khung hình: 16:9, 9:16, 1:1
- Không hỗ trợ hình ảnh-thành-video

### Runway Gen-3

**Tốt nhất cho:** Đầu ra chất lượng cao, hình ảnh-thành-video

**Cấu Hình:**
```json5
{
  providers: {
    video: {
      runway: {
        apiKey: "${RUNWAY_API_KEY}",
        model: "gen-3-alpha",
        defaultDuration: 10,
        defaultAspectRatio: "16:9"
      }
    }
  }
}
```

**Giới Hạn:**
- Thời lượng tối đa: 10 giây
- Hỗ trợ hình ảnh tham chiếu cho khung hình đầu tiên
- Chất lượng cao hơn, thời gian tạo lâu hơn

### Alibaba Model Studio (Wan)

**Tốt nhất cho:** Mô hình mã nguồn mở, hiệu quả chi phí

**Cấu Hình:**
```json5
{
  providers: {
    video: {
      modelstudio: {
        apiKey: "${MODELSTUDIO_API_KEY}",
        model: "wan-2.1",
        baseUrl: "https://api.modelstudio.ai/v1",
        defaultDuration: 5
      }
    }
  }
}
```

**Giới Hạn:**
- Thời lượng tối đa: 5 giây
- Yêu cầu tài khoản Model Studio

---

## Theo Dõi Tác Vụ

Tạo video là không đồng bộ. Theo dõi các tác vụ:

```bash
# Liệt kê tất cả các tác vụ video
openclaw tasks list --type video --status pending

# Chờ hoàn thành
openclaw tasks watch <task-id>

# Hủy tạo đang chờ
openclaw tasks cancel <task-id>
```

**Các Trạng Thái Tác Vụ:**
- `pending` — Đang xếp hàng để tạo
- `generating` — Đang tạo chủ động
- `complete` — Video sẵn sàng để tải xuống
- `failed` — Lỗi tạo (kiểm tra nhật ký)

---

## Nâng Cao: Quy Trình Làm Việc ComfyUI

Cho các quy trình video tùy chỉnh, sử dụng plugin ComfyUI:

```json5
{
  plugins: {
    entries: {
      comfy: {
        config: {
          baseUrl: "http://localhost:8188",
          workflows: {
            videoGenerate: {
              path: "./workflows/video-gen.json",
              inputs: {
                prompt: "{{prompt}}",
                width: 1024,
                height: 576
              }
            }
          }
        }
      }
    }
  }
}
```

Xem [Tích Hợp ComfyUI](../05-integrations/comfyui.md) để biết chi tiết.

---

## Mẹo Kỹ Thuật Lời Nhắc

**Lời nhắc video hiệu quả:**
- Cụ thể về chuyển động: "camera chậm zoom vào", "vật thể xoay"
- Mô tả ánh sáng và bầu không khí
- Bao gồm các yếu tố thời gian: "trong 5 giây", "dần dần biến đổi"
- Tránh: nhân vật có bản quyền, nội dung bạo lực (bộ lọc phụ thuộc nhà cung cấp)

**Ví Dụ Lời Nhắc Tốt:**
```
"Một cây hoa anh đào đung đưa nhẹ trong gió xuân, cánh hoa rơi chậm, ánh sáng giờ vàng"

"Mô phỏng chất lỏng trừu tượng, màu sắc ánh kim chuyển từ xanh sang tím, camera quay chậm"

"Thành phố tương lai về đêm, xe bay đi qua giữa các tòa nhà được chiếu sáng bằng neon, mưa trên ống kính"
```

---

## Xử Lý Sự Cố

### "Không tìm thấy nhà cung cấp"

**Nguyên nhân**: Nhà cung cấp chưa được cấu hình trong `providers.video.*`

**Khắc phục**: Thêm cấu hình nhà cung cấp với khóa API.

### "Hết thời gian tạo video"

**Nguyên nhân**: Tạo đang mất nhiều thời gian hơn dự kiến.

**Khắc phục**: Kiểm tra trạng thái tác vụ với `openclaw tasks get <id>`. Một số nhà cung cấp (Runway) mất 60-120 giây.

### "Tỷ lệ khung hình không hợp lệ"

**Nguyên nhân**: Nhà cung cấp không hỗ trợ tỷ lệ được yêu cầu.

**Khắc phục**: Sử dụng các tỷ lệ được hỗ trợ:
- xAI: 16:9, 9:16, 1:1
- Runway: 16:9, 9:16, 4:3, 1:1
- Model Studio: 16:9, 1:1

### "Khóa API không hợp lệ"

**Kiểm tra**: Đảm bảo khóa API hợp lệ và có quyền tạo video:
```bash
# Kiểm tra với nhà cung cấp trực tiếp
openclaw provider test --provider xai --type video
```

---

## Cân Nhắc Về Giá

Tạo video sử dụng API nhà cung cấp với chi phí liên quan:

| Nhà Cung Cấp | Chi Phí Mỗi Video | Ghi Chú |
|--------------|-------------------|---------|
| xAI | ~$0.10-0.30 | Thay đổi theo thời lượng |
| Runway | ~$0.50-1.00 | Chất lượng cao hơn, chi phí cao hơn |
| Model Studio | ~$0.05-0.15 | Hiệu quả chi phí nhất |

**Kiểm Soát Chi Phí:**
```json5
{
  agents: {
    myAgent: {
      tools: {
        video_generate: {
          enabled: true,
          requireConfirmation: true,  // Hỏi trước khi tạo
          dailyLimit: 10  // Tối đa tạo mỗi ngày
        }
      }
    }
  }
}
```

---

## Tài Liệu Liên Quan

- [Tạo Nhạc](./music-generation.md) — Tạo nhạc bằng AI
- [Tạo Hình Ảnh](./image-generation.md) — Tạo hình ảnh bằng AI
- [Tích Hợp ComfyUI](../05-integrations/comfyui.md) — Quy trình làm việc phương tiện tùy chỉnh
- [Quản Lý Tác Vụ](../06-automation/task-flow.md) — Xử lý tác vụ nền
- [Cấu Hình Nhà Cung Cấp](../10-cli-and-reference/config-reference.md#providers)

---

**Cập Nhật Lần Cuối**: 7 tháng 4, 2026  
**Áp Dụng Cho**: OpenClaw v2026.4.5+
