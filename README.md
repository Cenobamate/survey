# 翼思生物 · 市场调研问卷平台

> CNB（西诺氨酯/翼弗瑞®）& SOL（索安非托/翼朗清®）问卷调研网站
>
> 纯静态 HTML/CSS/JS，可直接部署到 GitHub Pages，无需后端。

---

## 📁 文件结构

```
survey-website/
├── index.html      # 主入口页（CNB/SOL 问卷入口 + 管理后台入口）
├── survey.html     # 问卷填写页（URL参数区分产品和模式）
├── admin.html      # 管理后台（数据汇总、图表、导出Excel、二维码）
└── README.md       # 本说明文档
```

---

## 🚀 部署到 GitHub Pages

### 方法一：通过 GitHub 网页上传

1. **登录 GitHub**，点击右上角 `+` → `New repository`
2. 填写仓库名称（如 `ignis-survey`），选择 **Public**，点击 `Create repository`
3. 在仓库页面点击 `uploading an existing file` 链接
4. 将 `index.html`、`survey.html`、`admin.html`、`README.md` 四个文件拖入上传区域
5. 填写 commit message（如"初始化问卷网站"），点击 `Commit changes`
6. 进入仓库 **Settings** → 左侧 **Pages**
7. 在 **Source** 下选择 `Deploy from a branch`
8. Branch 选择 `main`，文件夹选择 `/ (root)`，点击 `Save`
9. 等待 1-2 分钟，页面顶部会显示访问地址，格式为：
   ```
   https://<你的用户名>.github.io/ignis-survey/
   ```

### 方法二：通过 Git 命令行

```bash
# 克隆仓库
git clone https://github.com/<你的用户名>/ignis-survey.git
cd ignis-survey

# 复制四个文件到仓库目录
cp /path/to/index.html .
cp /path/to/survey.html .
cp /path/to/admin.html .
cp /path/to/README.md .

# 提交并推送
git add .
git commit -m "初始化问卷网站"
git push origin main
```

推送后进入 **Settings → Pages** 开启 GitHub Pages（Source 选 `main` 分支 `/ root`）。

### ⚠️ 重要：子路径注意事项

如果你的仓库不是 `<用户名>.github.io` 这种格式，网站会部署在子路径下（如 `https://xxx.github.io/ignis-survey/`）。所有页面间的链接已使用相对路径，无需额外配置。

---

## 📱 使用方式

### 问卷填写

访问主入口页后，选择对应产品（CNB/SOL）和模式（实名/匿名）即可填写。

也可以直接通过 URL 参数访问：

| 问卷 | URL |
|------|-----|
| CNB 实名 | `survey.html?product=cnb&mode=real` |
| CNB 匿名 | `survey.html?product=cnb&mode=anon` |
| SOL 实名 | `survey.html?product=sol&mode=real` |
| SOL 匿名 | `survey.html?product=sol&mode=anon` |

### 管理后台

访问 `admin.html`，输入管理密码（默认：`ignis2026`）即可进入。

后台功能：
- 📊 **数据汇总**：查看总填写数、实名/匿名分布
- 📈 **图表展示**：各题选项分布柱状图
- 📋 **填写明细**：逐条查看每份问卷
- 📥 **导出 Excel**：一键导出 .xlsx 文件
- 📱 **二维码**：生成并下载各问卷入口的二维码 PNG

---

## ✏️ 自定义问卷题目

打开 `survey.html`，找到 `SURVEY_CONFIG` 变量（约第 200 行），按以下格式修改：

```javascript
var SURVEY_CONFIG = {
  cnb: {
    productName: '西诺氨酯（翼弗瑞®）',
    productCode: 'CNB',
    intro: '感谢您参与…',
    questions: [
      {
        id: 'q1',              // 题目ID（唯一，不可重复）
        type: 'single',        // 题型：single(单选) | multi(多选) | scale(量表) | text(文本)
        title: '题目内容',
        required: true,        // 是否必填
        options: ['选项A', '选项B', '选项C']  // single/multi类型需要
      },
      {
        id: 'q2',
        type: 'scale',         // 量表题
        title: '满意度评分',
        required: true,
        scaleMin: 1,           // 最小分值
        scaleMax: 5,           // 最大分值
        scaleMinLabel: '非常不满意',
        scaleMaxLabel: '非常满意'
      },
      {
        id: 'q3',
        type: 'text',          // 文本题
        title: '其他意见',
        required: false,
        placeholder: '请输入…'
      }
    ]
  },
  sol: { /* 同上格式 */ }
};
```

> ⚠️ 修改题目后，需同步更新 `admin.html` 中的 `PRODUCTS` 变量（约第 230 行），保持题目配置一致，否则后台图表和导出会不匹配。

### 修改管理密码

打开 `admin.html`，找到 `ADMIN_PASSWORD` 变量（约第 215 行）：

```javascript
var ADMIN_PASSWORD = 'ignis2026'; // 修改为你自己的密码
```

---

## 🔗 对接真实后端（可选）

当前版本使用 **localStorage** 存储数据，数据仅保存在填写者本地浏览器中。如需集中收集数据，可对接真实后端。

### 方案一：Google Sheets（推荐，免费）

1. 创建一个 Google Sheet，表头设为：`product | mode | timestamp | name | hospital | dept | answers`
2. 打开 Google Sheets → `扩展程序` → `Apps Script`
3. 粘贴以下代码：

```javascript
function doPost(e) {
  var sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName('Responses') || 
              SpreadsheetApp.getActiveSpreadsheet().insertSheet('Responses');
  
  if (sheet.getLastRow() === 0) {
    sheet.appendRow(['product', 'mode', 'timestamp', 'name', 'hospital', 'dept', 'answers']);
  }
  
  var data = JSON.parse(e.postData.contents);
  sheet.appendRow([
    data.product,
    data.mode,
    data.timestamp,
    data.answers._name || '',
    data.answers._hospital || '',
    data.answers._dept || '',
    JSON.stringify(data.answers)
  ]);
  
  return ContentService.createTextOutput(JSON.stringify({status: 'ok'}))
    .setMimeType(ContentService.MimeType.JSON);
}
```

4. 点击 `部署` → `新建部署` → 类型选 `Web 应用`
5. 设置：执行身份=自己，访问权限=任何人
6. 复制部署 URL
7. 打开 `survey.html`，找到 `submitSurvey()` 函数中的 webhook 部分（约第 350 行），取消注释并填入 URL：

```javascript
var WEBHOOK_URL = 'https://script.google.com/macros/s/YOUR_DEPLOYMENT_ID/exec';
fetch(WEBHOOK_URL, { method: 'POST', mode: 'no-cors', body: JSON.stringify(record) });
```

### 方案二：Airtable

1. 创建 Airtable Base，添加字段对应问卷各题
2. 获取 Airtable API Key 和 Base ID
3. 在 `survey.html` 的 `submitSurvey()` 中添加：

```javascript
fetch('https://api.airtable.com/v0/YOUR_BASE_ID/Survey', {
  method: 'POST',
  headers: {
    'Authorization': 'Bearer YOUR_API_KEY',
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({ records: [{ fields: record.answers }] })
});
```

### 方案三：自建后端 API

在 `survey.html` 的 `submitSurvey()` 中替换存储逻辑：

```javascript
// 替换 localStorage 存储为后端提交
fetch('https://your-api.com/survey/submit', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify(record)
})
.then(res => res.json())
.then(data => {
  // 显示成功页
  document.getElementById('surveyContainer').style.display = 'none';
  document.getElementById('submitBar').style.display = 'none';
  document.getElementById('successPage').style.display = 'block';
  window.scrollTo(0, 0);
});
```

> 对接后端后，`admin.html` 的数据读取也需要改为从后端 API 获取（将 `getRecords()` 函数中的 localStorage 读取替换为 fetch 调用）。

---

## 🎨 技术栈

| 功能 | 库 | CDN |
|------|------|-----|
| 图表 | Chart.js 4.4.1 | `cdn.jsdelivr.net/npm/chart.js` |
| Excel导出 | SheetJS (xlsx) 0.18.5 | `cdn.jsdelivr.net/npm/xlsx` |
| 二维码 | qrcode.js 1.0.0 | `cdn.jsdelivr.net/npm/qrcodejs` |

所有库通过 CDN 引入，无需本地安装。

---

## 📐 设计说明

- **移动端优先**：医生通过手机扫码填写，界面针对小屏优化
- **品牌色**：CNB 使用深蓝色（`#1a3a5c`），SOL 使用翼思橙（`#e8732a`）
- **中文界面**：全中文，符合医药行业使用习惯
- **无依赖**：纯静态文件，无需 Node.js / 数据库 / 服务器

---

## ❓ 常见问题

**Q: 数据存在 localStorage，换设备/浏览器能看到吗？**
A: 不能。localStorage 是浏览器本地存储，数据不跨设备同步。如需集中管理，请对接真实后端（见上方说明）。

**Q: 二维码扫描后打不开？**
A: 确认 GitHub Pages 已正确部署，且二维码 URL 与实际访问地址一致。后台生成的二维码 URL 基于当前 `admin.html` 的地址自动推导。

**Q: 如何修改问卷题目？**
A: 编辑 `survey.html` 中的 `SURVEY_CONFIG` 变量，同时同步更新 `admin.html` 中的 `PRODUCTS` 变量。详见上方"自定义问卷题目"章节。

**Q: 管理密码忘了？**
A: 默认密码为 `ignis2026`。如已修改并遗忘，直接打开 `admin.html` 源码查看 `ADMIN_PASSWORD` 变量即可。

---

## 📄 License

© 2026 翼思生物医药（上海）有限公司 · Ignis Therapeutics
仅供内部市场调研使用。
