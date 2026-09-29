# X老中医传承工作室 · 独立站

为老中医打造的「学术展示 + 患者服务」独立网站（结构安排参考 I:\linjun-website）。包含：风采展示、学术成就、理论心得、从业经历、患者寄语五大内容板块，以及基于 Cloudflare Workers AI 的「AI 健康助手」在线咨询（模型 `@cf/zai-org/glm-4.7-flash`）。

## 目录结构

```
I:\zhongyi\
├── index.html              # 独立站主页（单文件，含全部样式与逻辑）
├── functions\api\chat.js   # Cloudflare Pages Function：AI 助手接口 /api/chat
├── _routes.json            # Pages Functions 路由配置（/api/*）
├── hero-doctor.png         # 主视觉：X老在中医诊室
├── portrait.png            # 肖像：风采展示 / AI助手头像
├── academic.png            # 学术：伏案著述
├── clinic.png              # 诊室：切脉看诊（患者寄语板块）
├── herbs.png               # 中药静物（理论心得板块）
└── README.md               # 本说明
```

## 一、本地预览

直接用浏览器打开 `I:\zhongyi\index.html` 即可预览。未部署时 `/api/chat` 不可用，AI 助手会自动回退为**内置演示回复**（消息会标注「演示回复」）。

## 二、部署（Cloudflare Pages，与 I:\linjun-website 相同方式）

1. 打开 Cloudflare Dashboard → Workers 和 Pages → 创建应用程序 → Pages → 连接到 Git 仓库（或将 `I:\zhongyi` 整个目录直接上传）。
2. 无需构建配置，构建输出目录填根目录 `/`；入口文件为 `index.html`。
3. 项目设置 → Functions → 绑定 **Workers AI**（变量名填 `AI`）→ 在 AI 设置中允许模型 `@cf/zai-org/glm-4.7-flash`。
4. 部署后即可访问；`functions/api/chat.js` 会自动生成 `/api/chat` 接口，与页面 AI 助手直接接通，无需改动代码。

## 三、修改 AI 助手人设

编辑 `functions/api/chat.js` 中的 system 提示词即可调整口吻、知识边界与合规规则。建议随X老公开资料的丰富持续更新。

## 四、上线前需要替换的内容（占位标记说明）

页面中所有标注 `.ph` 样式（书名、篇数、年份、地址、出诊时间等）及「示例」字样均为**占位内容**，需按真实资料替换：

| 位置 | 待替换内容 |
| --- | --- |
| 全文 | 「X老」等人物称谓、品牌名 |
| 学术成就 | 著作/论文/讲座/带教的具体信息 |
| 理论心得 | 四则示例主张 → 替换为X老本人认可的学术表述 |
| 从业经历 | 六个阶段的真实年份与经历 |
| 患者寄语 | 寄语全文与三条示例留言 → 替换为真实内容 |
| 预约门诊 | 地址、出诊时间、企业微信二维码 |
| 图片 | 所有生成图为示意，可替换为X老本人照片 |

## 五、合规要点（已在页面与 Function 中落实）

- 对外明确标识「AI 健康助手」，不冒称X老本人；
- AI 不提供诊断与处方，急症与诊疗需求一律引导至门诊/正规医疗机构（Function 内置急症关键词预检）；
- 页面页脚与对话窗口均有免责声明；
- 医案、学术内容、肖像与姓名权等知识产权归X老所有，网站仅作展示与科普。

## 六、成本参考（以 Cloudflare 官方为准）

- Workers 免费额度可覆盖小流量场景；`glm-4.7-flash` 单价约 $0.0605/M 输入 token、$0.40/M 输出 token（见 Cloudflare 模型页）。
- 域名约 ¥100/年左右；如需企业微信认证约 ¥300/年（账号归X老方所有）。

## 七、FAQ：AI 助手只回「演示回复」/ 没有真实 AI 回复

**症状**：对话框有界面，但发送后只返回「演示回复」或提示接口不可用，`/api/chat` 404。

**原因**：生产仓库里缺少 `functions/` 目录——AI 请求没有服务端代码接收。AI binding 只是「连接通道」，必须由 Pages Function 实际调用 `context.env.AI.run()`。

**修复（推送清单，逐项核对）**：

1. 用 `I:\zhongyi` 整个目录替换 GitHub 仓库 `kenstella86/zhongyi` 根目录内容，**必须包含以下文件并一起提交**：
   ```
   functions/api/chat.js   # Pages Function：接收 /api/chat，调用 glm-4.7-flash
   _routes.json            # 路由：{"version":1,"include":["/api/*"],"exclude":[]}
   index.html              # 前端（已与 chat.js 配对：发 {question} 收 {answer}）
   ```
2. 前端与后端参数已配对（`{ question: 问题 }` → 返回 `{ answer: 回复 }`），**不要混用 `{message}/{reply}` 写法**，否则会不匹配。
3. 提交并推送到 `main` 分支，Cloudflare Pages 自动重新部署（约 1–2 分钟）。
4. 验证：浏览器打开部署站点 → 开发者工具 Network 里请求 `POST /api/chat`，若返回 `{"answer":"..."}` 即成功；若 404，说明 `functions/` 未进仓库。
5. 本地测试 Pages Function（无需部署）：
   ```bash
   npx wrangler pages dev I:\zhongyi
   ```
   在 `http://localhost:8788` 打开站点即可联调 `/api/chat`（需本机有 Workers AI 凭据配置）。

