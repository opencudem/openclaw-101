# Task Flow: Điều Phối Nền

> Điều phối tác vụ nền bền vững với chế độ đồng bộ được quản lý/phản chiếu, theo dõi phiên bản, và phục hồi mượt mà.

**Phiên bản giới thiệu**: v2026.4.2  
**Cấp độ**: Trung cấp đến Nâng cao  
**Yêu cầu**: [06-automation: Cron Jobs](./cron-jobs.md), [11-creating-agents](../11-creating-agents/)

---

## Task Flow Là Gì?

Task Flow là lớp nền tảng của OpenClaw để điều phối các tác vụ nền dài hạn. Khác với cron job đơn giản "phát và quên", Task Flow cung cấp:

- **Trạng thái bền vững** tồn tại sau khi gateway khởi động lại
- **Theo dõi phiên bản** cho điểm phục hồi chính xác
- **Tạo tác vụ con** với quản lý vòng đời phối hợp
- **Ý định hủy dính** cho việc tắt máy mượt mà
- **Truy cập API plugin** để xây dựng khả năng điều phối

Dùng Task Flow khi bạn cần:
- Quy trình nhiều bước phải tồn tại qua gián đoạn
- Quan hệ tác vụ cha-con với hủy bỏ phối hợp
- Pipeline nền (trích xuất dữ liệu → xử lý → báo cáo)
- Tự động hóa phức tạp cần kiểm tra trạng thái và phục hồi

---

## Chế Độ Đồng Bộ

Task Flow hỗ trợ hai chế độ đồng bộ:

### Chế Độ Được Quản Lý (Mặc Định)

OpenClaw quản lý và duy trì trạng thái flow hoàn toàn. Vòng đời flow được tách biệt khỏi bất kỳ hệ thống điều phối bên ngoài.

**Dùng khi**: Bạn muốn OpenClaw là nguồn sự thật cho điều phối tác vụ.

```json5
{
  taskFlows: {
    myPipeline: {
      mode: "managed",
      steps: [
        { name: "extract", tool: "web_fetch" },
        { name: "process", agent: "data-processor" },
        { name: "report", channel: "slack" }
      ]
    }
  }
}
```

### Chế Độ Phản Chiếu

Trạng thái flow phản chiếu từ hệ thống điều phối bên ngoài. OpenClaw căn chỉnh trạng thái của nó với thay đổi trạng thái bên ngoài.

**Dùng khi**: Bạn có bộ điều phối bên ngoài (Airflow, Temporal, v.v.) và OpenClaw thực thi tác vụ trong framework đó.

```json5
{
  taskFlows: {
    externalPipeline: {
      mode: "mirrored",
      externalId: "airflow-dag-123",
      syncInterval: "30s"
    }
  }
}
```

---

## Task Flow Cơ Bản

### Tạo Flow

```bash
# Tạo workflow 3 bước đơn giản
openclaw taskflow create \
  --name daily-report \
  --steps "fetch-data,process,notify"
```

### Kiểm Tra Flow

```bash
# Liệt kê tất cả flow đang hoạt động
openclaw flows list

# Lấy trạng thái chi tiết của flow cụ thể
openclaw flows status daily-report

# Xem lịch sử phiên bản
openclaw flows revisions daily-report
```

### Kích Hoạt và Hủy

```bash
# Bắt đầu flow
openclaw flows start daily-report

# Hủy với ý định dính (chờ tác vụ con)
openclaw flows cancel daily-report --graceful

# Hủy bắt buộc (dừng ngay lập tức)
openclaw flows cancel daily-report --force
```

---

## Tạo Tác Vụ Con

Task Flow hỗ trợ quan hệ cha-con nơi tác vụ con kế thừa ngữ cảnh nhưng có quản lý vòng đời độc lập.

### Flow Cha Với Tác Vụ Con

```json5
{
  taskFlows: {
    contentPipeline: {
      mode: "managed",
      steps: [
        {
          name: "research",
          spawn: {
            agent: "researcher",
            isolated: true,
            timeout: "10m"
          }
        },
        {
          name: "write",
          dependsOn: ["research"],
          spawn: {
            agent: "writer",
            context: "{{steps.research.output}}"
          }
        },
        {
          name: "thumbnail",
          parallel: true,  // Chạy song song với "write"
          spawn: {
            agent: "designer",
            task: "generate thumbnail"
          }
        }
      ]
    }
  }
}
```

### Ý Định Hủy Dính

Khi bạn hủy flow cha, tác vụ con nhận ý định hủy nhưng có thể hoàn thành mượt mà:

```bash
# Cha báo hiệu hủy, con hoàn thành công việc hiện tại rồi dừng
openclaw flows cancel contentPipeline --graceful

# Kiểm tra trạng thái tác vụ con
openclaw flows children contentPipeline
```

---

## Phục Hồi và Khả Năng Phục Hồi

### Phục Hồi Tự Động

Nếu gateway khởi động lại, Task Flow tự động phục hồi về phiên bản đã commit cuối:

```bash
# Kiểm tra trạng thái phục hồi sau khi khởi động lại
openclaw flows status daily-report

# Kết quả mong đợi:
# Flow: daily-report
# State: recovered
# Revision: 42
# LastStep: process
# ChildTasks: 2 active
```

### Phục Hồi Thủ Công

Nếu phục hồi tự động lỗi hoặc bạn cần tiếp tục từ điểm cụ thể:

```bash
# Liệt kê điểm phục hồi có sẵn
openclaw flows revisions daily-report --failed

# Tiếp tục từ phiên bản cụ thể
openclaw flows resume daily-report --from-revision 38

# Thử lại bước lỗi
openclaw flows retry daily-report --step notify
```

---

## Truy Cập API Plugin

Plugin có thể tạo và quản lý Task Flow qua `api.runtime.taskFlow`:

```typescript
// Ví dụ code plugin
export async function execute(context) {
  const { taskFlow } = context.api.runtime;
  
  // Tạo flow được quản lý
  const flow = await taskFlow.create({
    name: "plugin-orchestrated",
    mode: "managed",
    steps: [
      { tool: "web_search", query: context.input.topic },
      { agent: "summarizer", input: "{{step0.results}}" }
    ]
  });
  
  // Bắt đầu và theo dõi
  await flow.start();
  const status = await flow.waitForCompletion({ timeout: "5m" });
  
  return { status, output: status.output };
}
```

Điều này cho phép plugin điều phối workflow phức tạp mà không cần định danh bên ngoài mỗi lần gọi.

---

## So Sánh Task Flow và Cron Jobs

| Tính Năng | Cron Jobs | Task Flow |
|-----------|-----------|-----------|
| **Kích hoạt** | Dựa trên thời gian | Sự kiện, API, hoặc thời gian |
| **Trạng thái** | Không trạng thái | Bền vững với theo dõi phiên bản |
| **Phục hồi** | Không | Phục hồi tự động từ khởi động lại |
| **Tác vụ con** | Không | Tạo tác vụ con được quản lý |
| **Độ phức tạp** | Đơn giản một lần | Điều phối nhiều bước |
| **Trường hợp dùng** | Thông báo, kiểm tra đơn giản | Pipeline, workflow, đa agent |

---

## Ví Dụ Thực Tế: Nhà Máy Nội Dung

Pipeline sản xuất nội dung đa agent:

```json5
{
  taskFlows: {
    newsletterFactory: {
      mode: "managed",
      schedule: "0 7 * * 1",  // Thứ Hai lúc 7h sáng
      steps: [
        {
          name: "curate",
          spawn: {
            agent: "researcher",
            task: "Tìm 5 tin tức AI hàng đầu từ tuần qua"
          }
        },
        {
          name: "draft",
          dependsOn: ["curate"],
          spawn: {
            agent: "writer",
            task: "Viết bản thảo newsletter từ {{steps.curate.articles}}"
          }
        },
        {
          name: "review",
          dependsOn: ["draft"],
          spawn: {
            agent: "editor",
            task: "Xem xét và chỉnh sửa: {{steps.draft.content}}"
          }
        },
        {
          name: "publish",
          dependsOn: ["review"],
          channel: "email",
          to: "subscribers@example.com",
          body: "{{steps.review.finalContent}}"
        }
      ],
      onFailure: {
        notify: "admin@example.com",
        retry: { maxAttempts: 3, backoff: "exponential" }
      }
    }
  }
}
```

**Kịch bản phục hồi**: Gateway khởi động lại trong bước "draft" → Flow tiếp tục ở "draft" với ngữ cảnh nghiên cứu còn nguyên.

---

## Tham Khảo CLI

| Lệnh | Mô Tả |
|------|-------|
| `openclaw flows list` | Liệt kê tất cả flow |
| `openclaw flows status <tên>` | Lấy trạng thái và phiên bản flow |
| `openclaw flows revisions <tên>` | Hiển thị lịch sử phiên bản |
| `openclaw flows start <tên>` | Kích hoạt flow |
| `openclaw flows cancel <tên>` | Hủy với tắt máy mượt mà |
| `openclaw flows resume <tên>` | Tiếp tục từ phiên bản cuối |
| `openclaw flows retry <tên>` | Thử lại bước lỗi |
| `openclaw flows children <tên>` | Liệt kê tác vụ con |
| `openclaw flows logs <tên>` | Xem log thực thi |

---

## Xử Lý Sự Cố

### "Trạng thái flow mất sau khi khởi động lại"

**Kiểm tra**: Đảm bảo flow đang dùng `mode: "managed"` hoặc `mode: "mirrored"`, không phải chế độ không trạng thái cũ.

**Sửa**: Cập nhật config với khai báo chế độ rõ ràng.

### "Tác vụ con mồ côi sau khi hủy"

**Kiểm tra**: Xác minh bạn đang dùng flag `--graceful` cho ý định hủy dính.

**Sửa**: 
```bash
openclaw flows cancel <tên> --graceful
# Chờ tác vụ con, rồi xác minh:
openclaw flows children <tên>
```

### "Không thể tiếp tục từ phiên bản"

**Kiểm tra**: Phiên bản có thể đã bị xóa. Kiểm tra cài đặt giữ lại.

**Sửa**: Điều chỉnh giữ lại hoặc thử lại từ phiên bản có sẵn:
```bash
openclaw flows revisions <tên> --available
openclaw flows resume <tên> --from-revision <id>
```

---

## Bước Tiếp Theo

- [Kết hợp với Cron](./cron-jobs.md) cho điều phối có lịch
- [Xây Dựng Workflow Đa Agent](../11-creating-agents/multi-agent-patterns.md)
- [Tạo Plugin Tùy Chỉnh](../11-creating-agents/plugin-development.md) dùng Task Flow API

---

**Cập Nhật Lần Cuối**: 4 tháng 4, 2026  
**Áp Dụng Cho**: OpenClaw v2026.4.2+
