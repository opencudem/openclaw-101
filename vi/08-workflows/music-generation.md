# Tạo Nhạc (Music Generation)

> Tạo nhạc và âm thanh với AI sử dụng Google Lyria, MiniMax, hoặc ComfyUI workflows.

## Công cụ này giải quyết gì

Công cụ `music_generate` tạo nhạc và âm thanh gốc từ văn bản mô tả. Dùng cho:
- Nhạc nền cho nội dung
- Hiệu ứng âm thanh và âm vang
- Phác thảo ý tưởng âm nhạc
- Nhận diện âm thanh thương hiệu

## Các Providers

| Provider | Phù hợp cho | Ghi chú |
|----------|-------------|---------|
| **Google Lyria** | Nhạc chất lượng cao | Tốt nhất cho bản nhạc phức tạp |
| **MiniMax** | Tạo nhanh, thị trường Trung Quốc | Tốt cho vòng lặp đơn giản |
| **ComfyUI** | Workflow-driven audio | Kiểm soát đầy đủ qua workflow |

## Bắt Đầu Nhanh

### Qua Chat

```
User: Tạo một vòng lặp ambient synth 30 giây
Agent: [Sử dụng music_generate với prompt, gửi file audio khi hoàn thành]
```

### Qua Tool Call

```json
{
  "music_generate": {
    "prompt": "Warm ambient synth loop with soft tape texture, 30 seconds",
    "duration": 30
  }
}
```

## Cấu Hình

### Google Lyria

```json5
{
  models: {
    providers: {
      google: {
        apiKey: "${GOOGLE_API_KEY}",
      },
    },
  },
}
```

### MiniMax

```json5
{
  models: {
    providers: {
      minimax: {
        apiKey: "${MINIMAX_API_KEY}",
      },
    },
  },
}
```

### ComfyUI Workflow

```json5
{
  models: {
    providers: {
      comfy: {
        mode: "local", // hoặc "cloud"
        baseUrl: "http://127.0.0.1:8188",
        music: {
          workflowPath: "./workflows/music-api.json",
          promptNodeId: "3",
          outputNodeId: "18",
        },
      },
    },
  },
}
```

## Tham Số Công Cụ

| Tham số | Kiểu | Bắt buộc | Mô tả |
|---------|------|----------|-------|
| `prompt` | string | Có | Mô tả văn bản của nhạc mong muốn |
| `duration` | number | Không | Thời lượng mục tiêu (giây) - chỉ là gợi ý |
| `style` | string | Không | Gợi ý thể loại (jazz, electronic, ambient) |
| `instrumental` | boolean | Không | Bắt buộc nhạc không lời |

## Ghi Chú Theo Provider

### Google Lyria
- Chất lượng cao nhất cho bản nhạc phức tạp
- Có thể bỏ qua gợi ý không được hỗ trợ với cảnh báo
- Tốt nhất cho: nhạc giao hưởng, jazz, điện tử, ambient

### MiniMax
- Thời gian tạo nhanh hơn
- Tối ưu cho thị trường/use case Trung Quốc
- Tốt nhất cho: vòng lặp đơn giản, nhạc nền

### ComfyUI
- Kiểm soát workflow đầy đủ
- Hỗ trợ hình ảnh tham chiếu (cho image-to-audio)
- Tốt nhất cho: pipeline tùy chỉnh, tạo hàng loạt

## Theo Dõi Tác Vụ

Tạo nhạc là bất đồng bộ. Công cụ:
1. Gửi yêu cầu tạo
2. Trả về task ID ngay lập tức
3. Kiểm tra trạng thái hoàn thành
4. Gửi file audio khi sẵn sàng

**Timeout:** Cấu hình qua `agents.defaults.musicGenerateTimeout` (mặc định: 300 giây)

## Ví Dụ

### Beat điện tử
```json
{
  "music_generate": {
    "prompt": "Upbeat electronic dance track with strong bassline, 4/4 beat, energetic",
    "duration": 60,
    "style": "electronic"
  }
}
```

### Nhạc nền ambient
```json
{
  "music_generate": {
    "prompt": "Soft ambient pads, no percussion, meditative atmosphere",
    "duration": 120,
    "instrumental": true
  }
}
```

### Jazz trio
```json
{
  "music_generate": {
    "prompt": "Piano jazz trio, walking bass, brush drums, laid-back swing",
    "style": "jazz",
    "instrumental": true
  }
}
```

## Đầu Ra

Công cụ trả về:
- **File âm thanh** (MP3 hoặc WAV)
- **Metadata** (thời lượng, provider sử dụng, thời gian tạo)
- **Chi tiết tác vụ** (nếu cần giao hàng bất đồng bộ)

## Giới Hạn

- Thời lượng là gợi ý, không đảm bảo (providers có thể xấp xỉ)
- Một số providers không hỗ trợ lời hát (kiểm tra instrumental flag)
- File lớn có thể cần timeout dài hơn

## Liên Quan

- [Video Generation](./video-generation.md) — Tạo video với âm thanh phù hợp
- [Image Generation](../04-skills/) — Tạo ảnh bìa cho nhạc

---

**Phiên bản:** v2026.4.5+
**Providers:** Google Lyria, MiniMax, ComfyUI
**Công cụ:** `music_generate`