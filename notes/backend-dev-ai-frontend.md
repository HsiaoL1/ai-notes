# 后端开发者用 AI 写前端：资源清单与工作流

> 面向有后端经验、但不熟悉前端工程的人。目标不是"学会前端"，而是**用 AI 交付一个能上线的前端**，并且知道什么时候它在胡说。

记录时间：2026-11-21

---

## 0. 先明确一件事

不存在一份官方的"后端转 AI 前端"统一教程。市面上的资源可以拆成三个层次，缺一层就会卡住：

| 层次 | 解决什么 | 典型资源 |
|---|---|---|
| **方法层** | 整体流程、先后顺序、如何描述需求 | aicodeguide、现代 Web 指南、中文实战帖 |
| **约束层** | 让 AI 每次都按你的规范写，而不是每次重新猜 | `.cursorrules` / `AGENTS.md` / 规则库 |
| **观测层** | 让 AI 能"看见"渲染结果，而不是盲改 | Chrome DevTools MCP、截图比对工具 |

大多数人只用了第一层，所以会陷入"改了十遍还是不对"的循环。**观测层是分水岭。**

---

## 1. 一条可执行的主链路

```
定栈 → 立规矩 → 后端先行 → AI 生成骨架（mock 数据）→ 接真接口 → 看渲染结果 → 定点修 → 循环
```

对应到具体工具：

```
生成骨架      v0 / 21st.dev / Bolt.new
接真接口      你的后端 + 一份 endpoint 文档
看渲染结果    chrome-devtools-mcp
定点修        imugi（截图比对）或 DevTools 里的 computed style
```

**关键顺序：前端先用 mock 数据跑起来，别等后端。** 反过来做，你会被迫在接口还没稳定时反复改前端。

---

## 2. 技术栈：别自己选

后端最容易踩的坑是按自己的审美挑框架（"我用 Go，那前端用 Svelte 吧"）。AI 对主流栈的训练数据密度高一个量级，**选小众栈等于主动降低 AI 的成功率**。

推荐组合，理由从"AI 友好度"出发：

| 选择 | 理由 |
|---|---|
| **Next.js**（App Router） | AI 生成质量最高的 React 框架，模板和规则库最全 |
| **Tailwind CSS** | 样式用类名表达，AI 不用理解 CSS 层叠，你也不用。改样式 = 改字符串 |
| **shadcn/ui** | 组件是**复制进项目**的源码，不是黑盒依赖。AI 可以直接改它，你也能看懂 |
| **TypeScript** | 类型错误是 AI 最好的自检信号，比运行时白屏好排查一百倍 |

替代方案：不想碰 React 生态就用 **FastAPI + Jinja2/HTMX**，AI 生成质量同样不错，且和你后端同语言。真要 3D 或重交互再考虑别的。

---

## 3. 资源清单

### 3.1 方法与路线图

**automata/aicodeguide** — https://github.com/automata/aicodeguide

最贴合"后端想用 AI 搭前端"这个具体处境的仓库，通篇是实操建议而非概念：

- 前后端分两个目录，或两个仓库都挂进同一个 Cursor workspace，让 agent 能跨端引用文件
- **先让 AI 用 mock/dummy 数据写前端**，后端完成后再切换
- 接 Playwright / browser-use MCP，让 agent 自己抓截图和报错，省掉复制粘贴
- 后端是端到端测试的好目标——让 agent 给每个新功能写测试并跑通
- 后端完成后，把 HTTP 接口文档作为输入喂给前端 agent，用于从 mock 切真实数据

**GoogleChrome/modern-web-guidance** — https://github.com/GoogleChrome/modern-web-guidance-src

Chrome / Edge / Firefox / MDN 团队共同维护。解决"AI 写的前端像活在五年前"：明明原生 `<dialog>`、Popover API、容器查询都能用，它非要引一堆库和 polyfill。

收录 262 个现代 Web 特性、204 个真实用例，且**刻意剔除了模型已经会的常识**，所以注入上下文时很省 token。还带 A/B 评测看板，公开了接入前后的代码通过率。

```bash
npx modern-web-guidance@latest install      # 交互式安装
npx modern-web-guidance@latest search "form validation"
npx modern-web-guidance@latest retrieve "ime-safe-enter-submit"
npx modern-web-guidance@latest legacy-scan .   # 扫出代码里的过时写法
```

一个很懂中文场景的细节：`ime-safe-enter-submit` 指南专门教 AI 判断 `isComposing`，避免用户用中文输入法按回车选词时，表单被提前提交。这种 bug 你手写也很难想到。

**中文实战帖**

- 《Vibe Coding 时代，后端程序员开发前端的最佳实践》(CSDN) — 核心四条：不手写 CSS、全交给 Tailwind；需求描述精确到"栈 + 组件库 + 交互 + 接口 + 适配端"；用官方模板而非从零搭；建规则文件防 AI 放飞
- 《一人公司失败指北——0 代码基础 AI 开发简易工程化》(人人都是产品经理) — 把调试拆成"观测 → 描述 → 猜想 → 验证 → 修改 → 沉淀"，是所有资源里对"为什么反复改都改不对"解释得最透的

### 3.2 规则文件（约束层，收益最高的一件事）

规则文件写一次，之后每次生成都自动遵守，省掉重复交代。

| 仓库 | 说明 |
|---|---|
| [PatrickJS/awesome-cursorrules](https://github.com/PatrickJS/awesome-cursorrules) | 最大的规则库，按技术栈分类，直接复制 `.cursorrules` |
| [blefnk/awesome-cursor-rules](https://github.com/blefnk/awesome-cursor-rules) | 对齐 Next.js 15 / React 19 / Tailwind 4 / shadcn/ui，和后端最省心的组合重合 |
| [linlang781/awesome-cursorrules-zh](https://github.com/linlang781/awesome-cursorrules-zh) | 中文翻译版，132 条规则 |

现在更通用的载体是项目根目录的 `AGENTS.md`（Claude Code、Codex、Cursor 都认）。模板见 [第 4 节](#4-落地步骤)。

### 3.3 观测层工具（关键）

**chrome-devtools-mcp（官方，优先装）** — https://github.com/ChromeDevTools/chrome-devtools-mcp

让 agent 控制一个真实的 Chrome：读 DOM、computed style、console（带 sourcemap 堆栈）、网络请求，还能录制性能 trace。

```json
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": ["-y", "chrome-devtools-mcp@latest"]
    }
  }
}
```

官方给出的示例提示词，正好覆盖你的两个需求：

- 改样式：`The page on localhost:8080 looks strange and off. Check what's happening there.`
- 性能：`Localhost:8080 is loading slowly. Make it load faster.`

Chrome 144+ 还支持远程调试自动接管：在 Elements 面板里选好出问题的元素，直接让 agent 接手排查。这是人工定位和自动化修复之间最好用的衔接方式。

**imugi（视觉比对，专治还原度）** — https://github.com/M00N7682/imugi

给它一张设计稿，它循环执行：截图 → SSIM + 像素 diff → 标出差异区域 → 抽取 DOM 的 computed style → 让 AI 改 → 再比，直到相似度 95%+。

```bash
npm install -g imugi-ai
imugi init          # 自动装 Playwright、识别框架、生成配置
```

```json
{ "mcpServers": { "imugi": { "command": "npx", "args": ["-y", "imugi-ai", "mcp"] } } }
```

解决的问题很具体：AI 一次能做出"差不多"，但剩下两小时的 padding、字号、颜色、对齐，以前只能人肉熬。配了 `FIGMA_TOKEN` 还能直接从 Figma API 拉设计值做 diff（`fontSize: design=42px → your code=48px`），不用从像素猜。

**shadcn MCP** — 让 AI 装组件，而不是编组件

```bash
pnpm dlx shadcn@latest mcp init --client claude
```

免费、无需 key。装进来的是被 review 过的代码、依赖声明完整、落在 `components/ui/` 统一管理；编出来的则是没人看过的。这个区别在长期维护上差很远。

**21st.dev Magic MCP** — 5000+ 现成区块，可按提示词生成变体

```bash
npx @21st-dev/cli login
npx @21st-dev/cli init --client claude --write
```

需要账号，免费额度有限。只在 shadcn 生态里找不到所需区块时才动用。

### 3.4 生成初版：v0 / Lovable / Bolt.new

| | 擅长 | 短板 |
|---|---|---|
| **v0**（Vercel） | 组件质量最高，Next.js + shadcn/ui + Tailwind，适合塞进已有项目做局部 | 纯前端，不支持 Vue/Svelte |
| **Lovable** | 整站最快，自带 Supabase + auth + Stripe | 会自作主张改结构，依赖偏重 |
| **Bolt.new** | 浏览器里跑完整 Node 环境，框架不限，全栈原型最快 | 组件质量一般，不用 shadcn |

**后端视角的建议**：v0 出单页或组件，Bolt 出可跑原型，然后**一律导出到 GitHub，回到 Cursor / Claude Code 里改**。Web 沙盒承载不了你真实的业务逻辑，越早回到本地越好。

---

## 4. 落地步骤

### Step 0 · 建骨架

```bash
npx create-next-app@latest my-app --tailwind --typescript --app --eslint
cd my-app
pnpm dlx shadcn@latest init
```

用官方模板，别从零搭。AI 对标准模板的适配度显著高于自定义结构。

### Step 1 · 写 AGENTS.md

放在项目根目录。这一份文件的投入产出比最高：

```markdown
# 项目规范

## 技术栈
- Next.js 15 App Router + TypeScript strict
- Tailwind CSS v4，样式一律用工具类，不写自定义 CSS 文件
- shadcn/ui 组件位于 components/ui/，业务组件放 components/

## 硬性约束
- 颜色只用语义 token：bg-background / text-foreground / text-muted-foreground
  / border-border / bg-primary。禁止硬编码 bg-zinc-100 这类调色板类名
- 禁止安装新依赖，需要时先问我
- 所有组件默认 Server Component；用到 useState/useEffect 才加 'use client'
- 表单用 react-hook-form + zod，校验 schema 与后端共用
- 数据请求统一走 lib/api.ts，不要在组件里直接 fetch

## 交互要求
- 每个异步操作都要有 loading / empty / error 三种状态
- 删除等破坏性操作必须二次确认，不用 window.confirm
- 移动端断点 768px，必须先看小屏效果

## 交付方式
- 一次只改一个关注点，改完告诉我改了什么、为什么
- 不确定的地方先问，不要猜着写
```

### Step 2 · 后端先行，前端用 mock

后端接口定好后，**先写一份 `docs/api.md`**（endpoint、参数、返回示例、错误码），再让 agent 按这份文档生成 `lib/api.ts` 的类型和调用函数。这一步做完，前后端就解耦了。

### Step 3 · 生成骨架

让 agent 一次只做一个页面或区块，别一次要整站：

```
用 Next.js + shadcn/ui 实现用户管理页：
- 顶部搜索框（防抖 300ms）+ 新建按钮
- 表格列：姓名、邮箱、角色、创建时间、操作
- 服务端分页，每页 20 条，排序状态写进 URL query
- 新建/编辑用 Dialog，表单 zod 校验，必填项失焦后才报错
- 数据从 lib/api.ts 的 listUsers() 取，先用 mock 数据
- 需要 loading 骨架屏和空状态
```

注意最后两条：**明确写出空状态和加载态**，否则 AI 默认只做"有数据"这一条路径，你上线才发现空列表是一片白。

### Step 4 · 接真接口

把 `docs/api.md` 和真实返回贴给 agent，让它替换 mock。重点检查三处：字段名大小写、错误响应结构、分页参数命名。这三个地方 AI 最容易按自己的想象写。

### Step 5 · 配观测层，进入修改循环

装好 chrome-devtools-mcp，然后不要再用"页面不对，你看看"这种话。见下一节。

---

## 5. 提示词模板

### 5.1 改样式

```
页面 http://localhost:3000/users 的表格区域有问题：
- 操作列在 1280px 下被挤成两行
- 表头和内容的左对齐不一致

用 DevTools 看一下这个表格的实际布局，先告诉我原因
（是列宽分配、flex 收缩还是 padding 导致的），
不要直接改代码。原因确认后给最小改动方案。
```

### 5.2 修 Bug

```
现象：点"保存"后弹窗关闭，但列表没有刷新，刷新页面才出现新数据。
期望：关闭弹窗后列表立即更新。
复现：1. 打开 /users 2. 点新建 3. 填完保存 4. 看列表

证据：
- console 无报错
- Network 里 POST /api/users 返回 201，响应体 {id: 12, ...}
- 列表请求没有再发出

相关代码：[贴 lib/api.ts 的 listUsers 和页面组件的 fetch 部分]

先列出可能原因并按概率排序，禁止修改代码。
```

### 5.3 让 AI 先诊断，而不是先动手

这是最重要的一条提示词，能挡掉大量假修复：

```
有 bug：<描述>。
禁止修改代码。按以下步骤诊断：
1. 这属于哪一类问题？（组件状态 / 数据流 / 样式层叠 / 接口契约 / 构建配置）
2. 列出所有可能原因，按概率从高到低排序
3. 对每个原因，说明用什么证据可以排除它
```

AI 最常见的坏习惯是看到报错就加 `?.` 或 try-catch，把错误吞掉。**"禁止修改代码"这句话本身就是最好的防线。**

---

## 6. 调试三件套

任何一次求助，这三样缺一不可：

1. **报错原文** — 完整堆栈，不要摘要，不要"它报了个错"
2. **代码上下文** — 出错的那几行 + 它依赖的定义
3. **期望行为** — 你原本想让它做什么

再加两条经验：

- **一次只改一处。** 一轮改五处，你就永远不知道哪处生效了。
- **改完自己去浏览器看。** 不要相信"已经修复了"这句话，AI 无法验证自己的输出。

---

## 7. 避坑清单

**AI 目前明显不擅长的：**

- **视觉审美判断** — 间距节奏、层级对比、配色气质。它能做出"正确"的页面，做不出"好看"的页面
- **React hydration 报错** — 服务端/客户端渲染不一致，它经常给出看似合理但无效的补丁
- **复杂状态同步** — 多个组件共享状态时的时序问题，容易越改越乱
- **跨浏览器兼容** — 它假设一切都跑在最新 Chrome 上
- **静默错误** — 用可选链和默认值把问题藏起来，而不是解决

**几个高频陷阱：**

| 陷阱 | 表现 | 对策 |
|---|---|---|
| 幻觉依赖 | `import { debounce } from 'lodash'` 但没装，或 API 根本不存在 | 规则文件里写"禁止安装新依赖"；每次核对 package.json |
| 造组件 | 需要个 Dialog，它从零写一个，而不是用已有的 | 装 shadcn MCP；规则里写"先查 components/ui/" |
| 只做happy path | 没有空状态、没有错误态、没有 loading | 需求里逐条写明三种状态 |
| 过度抽象 | 三个页面抽了五层 hook | 明确要求"先写直白的实现，重复三处以上再抽" |
| 样式硬编码 | `bg-zinc-100` 满天飞，暗色模式全崩 | 规则里锁死语义 token |

---

## 8. 参考资料

**方法**
- https://github.com/automata/aicodeguide
- https://github.com/GoogleChrome/modern-web-guidance-src
- https://developer.chrome.com/docs/modern-web-guidance

**规则**
- https://github.com/PatrickJS/awesome-cursorrules
- https://github.com/blefnk/awesome-cursor-rules
- https://github.com/linlang781/awesome-cursorrules-zh

**观测 / 调试**
- https://github.com/ChromeDevTools/chrome-devtools-mcp
- https://developer.chrome.com/blog/chrome-devtools-mcp
- https://github.com/M00N7682/imugi
- https://ui.shadcn.com/docs/mcp

**生成**
- https://v0.dev
- https://lovable.dev
- https://bolt.new
- https://21st.dev

**中文阅读**
- 《Vibe Coding 时代，后端程序员开发前端的最佳实践》
- 《一人公司失败指北——0 代码基础 AI 开发简易工程化》
- 《AI 辅助前端开发 5：Bug 调试篇》
