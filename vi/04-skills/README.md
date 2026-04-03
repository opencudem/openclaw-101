# 04 - Skills

> Dạy OpenClaw các thủ thuật mới: cài đặt, tạo, và quản lý skills từ ClawHub và nơi khác.

## Module này giải quyết gì

Skills là cách OpenClaw học làm những việc mới. Module này bao gồm tìm skills từ ClawHub, hiểu cách chúng hoạt động, và tạo skills của riêng bạn.

## Khi nào dùng module này

- Bạn muốn mở rộng khả năng OpenClaw
- Bạn cần tích hợp tùy chỉnh
- Bạn muốn tự động hóa công việc cụ thể

## Kiến trúc Skill

```
Nguồn Skills (theo thứ tự ưu tiên):
├── 1. Bundled Skills    (đi kèm OpenClaw)
├── 2. Workspace Skills  (~/.openclaw/skills/)
├── 3. ClawHub Skills    (cài qua openclaw skills install)
└── 4. Custom Entries    (đường dẫn chỉ định trong config)
```

Nguồn sau ghi đè nguồn trước. Điều này cho phép bạn tùy chỉnh skills đi kèm.

### Thứ tự tải Skill (Cao đến Thấp)

Khi nhiều skills có cùng tên, OpenClaw dùng thứ tự ưu tiên này:

1. **`<workspace>/skills`** - Workspace skills (ưu tiên cao nhất)
2. **`<workspace>/.agents/skills`** - Project agent skills
3. **`~/.agents/skills`** - Personal agent skills
4. **`~/.openclaw/skills`** - Managed/local skills
5. **Bundled skills** - Skills đi kèm OpenClaw
6. **`skills.load.extraDirs`** - Thư mục extra từ config (ưu tiên thấp nhất)

Thứ tự này đảm bảo workspace và agent-specific skills ghi đè mặc định hệ thống.

## Bắt đầu nhanh

### Tìm kiếm Skills

Duyệt skills có sẵn:

```bash
# Tìm kiếm ClawHub
openclaw skills search weather

# Liệt kê skills đã cài
openclaw skills list
openclaw skills list --eligible

# Lấy thông tin skill
openclaw skills info weather

# Kiểm tra vấn đề skill
openclaw skills check
```

### Cài đặt Skills

```bash
# Cài từ ClawHub
openclaw skills install <slug>
openclaw skills install weather

# Cài phiên bản cụ thể
openclaw skills install <slug>@version
openclaw skills install weather@1.2.0

# Cập nhật skill
openclaw skills update weather
openclaw skills update --all

# Để publish/cập nhật qua ClawHub CLI
clawhub sync --all
```

**Vị trí cài đặt:** `~/.openclaw/skills/` (cho managed/local skills)  
**Lưu ý:** Workspace skills tại `<workspace>/skills` có ưu tiên cao nhất

### Cài đặt qua npx (cho phát triển)

```bash
# Cài nhanh không cần thêm vào ClawHub
npx openclaw-skill-weather
```

## Cấu trúc Skill

Một skill là thư mục với cấu trúc này:

```
my-skill/
├── SKILL.md          # Tài liệu và spec
├── package.json      # Dependencies (tùy chọn)
├── references/       # File tham khảo
│   └── api-docs.md
└── scripts/          # Script thực thi được
    ├── get-weather.js
    └── forecast.js
```

### Định dạng SKILL.md (tương thích AgentSkills)

Frontmatter YAML bắt buộc với metadata OpenClaw:

```markdown
---
name: my-skill
description: Mô tả ngắn về skill này làm gì
metadata:
  { "openclaw": { "requires": { "bins": ["uv"], "env": ["API_KEY"], "config": ["browser.enabled"] }, "primaryEnv": "API_KEY" } }
---

# My Skill

## Description
Giải thích chi tiết skill này làm gì và khi nào dùng.

## Tools

### get-weather
Lấy thời tiết hiện tại cho một địa điểm.

**Parameters:**
- location (string, required): Tên thành phố

**Example:**
```javascript
getWeather({ location: "New York" })
```
```

**Bắt buộc:**
- YAML frontmatter với `name` và `description`
- Chỉ key một dòng
- `metadata` là JSON object một dòng với `openclaw.requires`

**Tùy chọn gating:**
- `requires.bins`: binary phải có trong PATH
- `requires.env`: biến môi trường phải được đặt
- `requires.config`: đường dẫn config phải tồn tại
- `requires.anyBins`: ít nhất một trong các binary này
- `always: true`: bỏ qua tất cả gates

**Key frontmatter tùy chọn:**
- `homepage`: Website URL
- `user-invocable: true|false`: Expose dưới dạng slash command (mặc định: true)
- `disable-model-invocation: true|false`: Loại khỏi model prompt (mặc định: false)

## Tạo Skill đầu tiên

```bash
# Tạo thư mục skill
mkdir -p ~/.openclaw/skills/my-first-skill
cd ~/.openclaw/skills/my-first-skill

# Tạo SKILL.md
cat > SKILL.md << 'EOF'
# Hello World Skill

## Description
Skill chào hỏi đơn giản để test.

## Tools

### greet
Chào ai đó.

**Parameters:**
- name (string, required): Người để chào

**Example:**
```javascript
greet({ name: "World" })
// Returns: "Hello, World!"
```
EOF

# Tạo script
mkdir -p scripts
cat > scripts/greet.js << 'EOF'
module.exports = async function greet({ name }) {
  return `Hello, ${name}!`;
};
EOF
```

Test nó:

```
User: Greet Alice
Agent: [Dùng skill greet] Hello, Alice!
```

## Ví dụ copy-paste

### Skill wrapper thời tiết

```javascript
// ~/.openclaw/skills/local-weather/scripts/weather.js
const fetch = require('node-fetch');

async function getWeather({ location }) {
  const response = await fetch(
    `https://wttr.in/${location}?format=j1`
  );
  const data = await response.json();
  
  return {
    temp: data.current_condition[0].temp_C,
    condition: data.current_condition[0].weatherDesc[0].value,
    humidity: data.current_condition[0].humidity
  };
}

module.exports = { getWeather };
```

### Skill helper Git

```javascript
// ~/.openclaw/skills/git-helper/scripts/repo.js
const { execSync } = require('child_process');

module.exports = {
  getStatus: () => {
    return execSync('git status --short', { encoding: 'utf8' });
  },
  
  getLastCommit: () => {
    return execSync('git log -1 --oneline', { encoding: 'utf8' });
  },
  
  getBranch: () => {
    return execSync('git branch --show-current', { encoding: 'utf8' }).trim();
  }
};
```

## Lỗi thường gặp và xử lý

| Vấn đề | Cách giải quyết |
|--------|-----------------|
| Skill không hiện | Kiểm tra cú pháp SKILL.md; xác minh cấu trúc thư mục |
| Lỗi script | Kiểm tra quyền file; đảm bảo có shebang line |
| Thiếu dependencies | Chạy `npm install` trong thư mục skill |
| Không tìm thấy skill | Xác minh skill trong search path; restart gateway |

## Pattern nâng cao

### Dependencies cho skill

```json
// package.json trong thư mục skill
{
  "name": "my-skill",
  "dependencies": {
    "axios": "^1.0.0",
    "cheerio": "^1.0.0"
  }
}
```

Sau đó chạy `npm install` trong thư mục skill.

### Skills điều kiện

Tải skills chỉ trong ngữ cảnh cụ thể:

```yaml
# ~/.openclaw/config.yaml
skills:
  entries:
    work-git:
      path: ./skills/work-git
      condition: "channel.type === 'slack'"
```

### Kết hợp skills

Skills có thể dùng skills khác:

```javascript
// Trong skill script của bạn
const weather = await skills.weather.getWeather({ location });
const calendar = await skills.calendar.getToday();

return `It's ${weather.temp}°C. You have ${calendar.events.length} events today.`;
```

## Module liên quan và bước tiếp theo

- Trước đó: [03-memory](../03-memory/)
- Tiếp theo: [05-integrations](../05-integrations/)
- Tham khảo: [Skills Config](https://docs.openclaw.ai/tools/skills-config)
- Duyệt: [ClawHub](https://clawhub.ai)

---

**Thời gian hoàn thành:** 60 phút  
**Yêu cầu trước:** [03-memory](../03-memory/)  
**Kết quả:** Skills tùy chỉnh đã cài và tạo, khả năng trợ lý được mở rộng
