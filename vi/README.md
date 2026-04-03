# OpenClaw 101 — Tiếng Việt

> Hướng dẫn thực hành về channels, skills, automations, và workflows.

[![GitHub stars](https://img.shields.io/github/stars/opencudem/openclaw-101?style=social)](https://github.com/opencudem/openclaw-101)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](../LICENSE)
[![Discord](https://img.shields.io/badge/Discord-Join%20Chat-7289DA?logo=discord)](https://discord.com/invite/clawd)

**Trạng thái bản dịch:** 🚧 Đang thực hiện (1/12 module)

---

[🚀 Bắt đầu nhanh](#bắt-đầu-trong-5-phút) · [📚 Lộ trình học](#lộ-trình-học)

---

## Tại sao có hướng dẫn này

OpenClaw rất mạnh — kết nối WhatsApp, Telegram, Discord, Slack và nhiều hơn nữa. Nó có thể duyệt web, chạy lệnh, quản lý lịch, và tự động hóa công việc lặp lại. Nhưng sự đa dạng đó có thể khiến người mới choáng ngợp.

Hướng dẫn này là **con đường được chọn lọc** mà chúng tôi ước gì đã có khi bắt đầu: từ cài đặt đầu tiên đến các workflow hữu ích, theo một trình tự dễ dự đoán.

## Khác biệt so với tài liệu chính thức

| Tài liệu chính thức | Hướng dẫn này |
|---------------------|---------------|
| Tài liệu tham khảo toàn diện | Học thực hành, tiến bộ từng bước |
| Tập trung vào tính năng | Tập trung vào workflow |
| Giả định bạn biết mình muốn gì | Giúp bạn khám phá khả năng |
| Hoàn hảo để tra cứu | Hoàn hảo để bắt đầu |

Dùng cả hai: bắt đầu từ đây, sau đó dùng tài liệu chính thức để tra cứu.

## Bắt đầu trong 5 phút

```bash
# 1. Cài đặt OpenClaw Gateway
npm install -g openclaw

# 2. Khởi động Gateway
openclaw gateway start

# 3. Kết nối kênh đầu tiên
openclaw channel add telegram

# 4. Chat với agent
# Gửi "hello" đến bot Telegram của bạn
```

👉 Tiếp tục với [01-getting-started](01-getting-started/) để có hướng dẫn cài đặt đầy đủ.

## Lộ trình học

| Cấp độ | Module | Thời gian | Mục tiêu |
|--------|--------|-----------|----------|
| 🌱 Cơ bản | 01-03 | 2-3 giờ | Chạy OpenClaw với ứng dụng chat yêu thích |
| 🌿 Trung cấp | 04-06 | 4-6 giờ | Dạy skills và tự động hóa công việc hàng ngày |
| 🌳 Nâng cao | 07-11 | 6-8 giờ | Xây dựng workflow phức tạp và multi-agent |
| 🔧 Bảo trì | 12 | Liên tục | Giám sát chủ động và cập nhật tài liệu |

**Mục tiêu cuối tuần:** Hoàn thành Cơ bản + Trung cấp để có giá trị ngay lập tức.

## Bản đồ Module

| Module | Nội dung | Thời gian |
|--------|----------|-----------|
| [01-getting-started](01-getting-started/) | Cài đặt, kênh đầu tiên, lệnh cơ bản | 30 phút |
| [02-channels](02-channels/) | Thiết lập WhatsApp, Telegram, Discord, Slack | 45 phút |
| [03-memory](03-memory/) | Cá nhân hóa, ngữ cảnh, bộ nhớ file | 30 phút |
| [04-skills](04-skills/) | Kiến trúc skill, ClawHub, local vs workspace | 60 phút |
| [05-integrations](05-integrations/) | GitHub, lịch, email, API | 45 phút |
| [06-automation](06-automation/) | Cron jobs, công việc định kỳ, briefing | 45 phút |
| [07-browser-automation](07-browser-automation/) | Nghiên cứu, điền form, trích xuất dữ liệu | 60 phút |
| [08-workflows](08-workflows/) | Kết hợp mọi thứ thành hệ thống thực | 90 phút |
| [09-security-and-ops](09-security-and-ops/) | Phê duyệt, quyền hạn, độ tin cậy | 45 phút |
| [10-cli-and-reference](10-cli-and-reference/) | Tài liệu tham khảo lệnh đầy đủ | 30 phút |
| [11-creating-agents](11-creating-agents/) | Kiến trúc multi-agent và routing | 45 phút |
| [12-heartbeat](12-heartbeat/) | Tự động hóa chủ động và bảo trì | 15 phút |

## Workflow Mẫu

Ví dụ sẵn sàng triển khai kết hợp nhiều tính năng:

| Workflow | Thành phần | Độ khó |
|----------|------------|--------|
| Morning Briefing | Cron + Channels + Thời tiết + Lịch | 🟢 Dễ |
| Engineering Inbox | Discord/Slack + GitHub skill | 🟡 Trung bình |
| Personal Follow-up System | Bộ nhớ + Messaging + Nhắc nhở | 🟡 Trung bình |
| Research Agent | Browser + Tìm kiếm + Tóm tắt | 🔴 Nâng cao |
| Approval-Gated Actions | Channels + Phê duyệt + Cách ly | 🔴 Nâng cao |

👉 Xem tất cả workflow trong [examples/](examples/)

## Tài liệu tham khảo

- [📖 CATALOG.md](CATALOG.md) — Skills và integrations được chọn lọc
- [🗺️ LEARNING-ROADMAP.md](LEARNING-ROADMAP.md) — Nội dung sắp tới
- [⚡ QUICK_REFERENCE.md](QUICK_REFERENCE.md) — Cheat sheet sử dụng hàng ngày
- [🔧 TROUBLESHOOTING.md](TROUBLESHOOTING.md) — Các vấn đề thường gặp và cách khắc phục
- [💡 USE_CASES.md](USE_CASES.md) — Tìm module phù hợp với mục tiêu

## FAQ

**H: Tôi cần kinh nghiệm lập trình không?**
Đ: Cơ bản thì không. Skills và workflow tùy chỉnh cần JavaScript/TypeScript cơ bản.

**H: Đây có phải tài liệu chính thức OpenClaw?**
Đ: Không, đây là hướng dẫn cộng đồng. Tài liệu chính thức tại [docs.openclaw.ai](https://docs.openclaw.ai).

**H: Tôi có thể đóng góp không?**
Đ: Có! Xem [CONTRIBUTING.md](../CONTRIBUTING.md) và [TRANSLATION.md](../TRANSLATION.md).

## Đóng góp bản dịch

Người đóng góp: @hoangmrb  
Cập nhật lần cuối: 2026-04-03

---

[🇺🇸 English version](../en/) · [📋 Hướng dẫn dịch](../TRANSLATION.md)

---

<p align="center">Made with 🐱 cho cộng đồng OpenClaw</p>
