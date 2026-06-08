# method-api-doc-html

![Claude Code Skill](https://img.shields.io/badge/Claude%20Code-Skill-7357ff) ![Output](https://img.shields.io/badge/output-HTML%20%C2%B7%20Markdown-1677ff) ![Lang](https://img.shields.io/badge/docs-Java%20%C2%B7%20%E4%B8%AD%E6%96%87-17b26a)

> 为**单个 Java 方法**生成一份**格式固定、面向调用方**的接口契约文档，**默认输出为精美的自包含单文件 HTML**（左侧导航、代码高亮、复制按钮、响应式），也可 `--format md` 退回 Markdown。

这是一个 [Claude Code](https://claude.com/claude-code) 个人 skill。它把某个对外暴露的 `Service / Facade / RPC / Controller` 方法，整理成给其他开发**直接调用**的契约：怎么调、传什么、返回什么、什么时候抛异常——固定 7 个章节，不增不减、不画图、不做架构分析。

---

## ✨ 特性

- **默认精美 HTML** —— 单文件、自包含（CSS/JS 全内联，无外链），双击即可在浏览器打开或直接分享。
- **固定 7 章节契约** —— 方法说明 / 方法定义 / 入参 / 返回值 / 调用示例 / 返回示例 / 异常说明，表头与顺序不可改，保证一致性。
- **HTTP 与 RPC 双形态** —— 模板内置 Controller(HTTP) 与 Service(RPC) 两套骨架，按目标方法二选一。
- **线性图标 + 分色导航** —— 7 章各配一枚 Lucide 风格内联 SVG 图标，侧边导航随滚动高亮。
- **可回退 Markdown** —— `--format md` 输出标准 Markdown，方便贴进 wiki / PR。
- **诚实不臆造** —— 必填性拿不准时标 `?`「待确认」，绝不编造约束。

---

## 📦 安装

skill 需要被放进 Claude Code 的个人 skills 目录 `~/.claude/skills/`。推荐用**软链接**：源码留在本 git 仓库里（可版本管理、可推送），Claude 通过链接加载。

```bash
# 1. 克隆到任意位置（例如你的 skills 工作区）
git clone git@github.com:qinghuazs/method-api-doc-html.git

# 2. 软链接到 Claude Code 个人 skills 目录
ln -s "$(pwd)/method-api-doc-html" ~/.claude/skills/method-api-doc-html

# 3. 重启 Claude Code 会话，skill 即被加载
```

验证：

```bash
ls -la ~/.claude/skills/method-api-doc-html
# 应显示：method-api-doc-html -> /path/to/method-api-doc-html
```

> **为什么用软链接？** 直接把目录放进 `~/.claude/skills/` 也能用，但软链接让真实文件留在 git 仓库中——改了 `SKILL.md` 或模板可以直接 `git commit && git push`，而 Claude 始终加载最新版本。

---

## 🚀 使用方法

安装后，在 Claude Code 中用自然语言触发，或用简写命令：

```
api-doc <方法定位> [输出位置] [--mode replace|append] [--format html|md]
```

**触发词**：`api-doc xxx`、"为 xxx 生成 HTML 接口文档"、"生成网页版/精美接口文档"、"把这个方法整理成能分享的调用说明"……

### 参数

| 参数 | 必填 | 说明 |
|------|------|------|
| 方法定位 | 是 | 方法名 `payBill` ／ 全限定 `com.x.Y#m` ／ 带签名 `com.x.Y#m(String,Long)`（同名重载会让你选） |
| 输出位置 | 否 | 目录或文件路径；不给时在对话中展示，要求落盘时通常放 `docs/` 下 |
| `--mode` | 否 | `replace`（默认，覆盖）／ `append`（追加，前置 `## 修订 vN`） |
| `--format` | 否 | `html`（**默认**，自包含单文件 HTML）／ `md`（Markdown） |

### 示例

```
api-doc com.example.account.OpenApiForAccountBalanceController#occupyAccountBalance docs/api/
```

Claude 会定位该方法源码 → 采集 7 章节所需信息 → 套用 `html-template.html` → 用 heredoc 写出 `occupyAccountBalance.html`。

---

## 📄 输出长什么样

生成的 HTML 是一张带侧边导航的单页文档：

```
┌──────────────┬─────────────────────────────────────────────┐
│              │   渐变 Hero：方法中文名 + 英文名 + 一句话用途    │
│  ▸ 方法说明   ├─────────────────────────────────────────────┤
│  ▸ 方法定义   │   概览卡：请求方式 / 接口地址 / Content-Type    │
│  ▸ 入参说明   │           / 成功返回码 / 更新时间               │
│  ▸ 返回值说明 │   实现卡：Controller  ·  实现类#方法            │
│  ▸ 调用示例   │  ┌── ① 方法说明（大图标 + 适用场景）─────────┐  │
│  ▸ 返回示例   │  ├── ② 方法定义（mac 风格代码框 + 复制按钮）─┤  │
│  ▸ 异常说明   │  │   ③ 入参表  ④ 返回表（必填/选填彩色 chip）│  │
│              │  │   ⑤ 调用示例  ⑥ 返回 JSON  ⑦ 异常卡片网格 │  │
│  💡 小贴士    │  └──────────────────────────────────────────┘  │
└──────────────┴─────────────────────────────────────────────┘
```

7 个固定章节：

| # | 章节 | 内容 |
|---|------|------|
| 1 | 方法说明 | 用途 + 适用业务场景 |
| 2 | 方法定义 | 完整签名（HTTP 接口则给「方法 + 路径 + Content-Type」） |
| 3 | 入参说明 | 参数名 / 类型 / 必填 / 示例值 / 含义约束 |
| 4 | 返回值说明 | 字段名 / 类型 / 示例值 / 含义 |
| 5 | 调用示例 | 最小可用调用（HTTP 请求 或 Java 注入调用） |
| 6 | 返回示例 | 与第 4 章同源的真实形态 JSON |
| 7 | 异常说明 | 异常/错误码 / 触发场景 / 处理建议 |

页面内置：滚动高亮导航、代码/JSON 一键复制、返回顶部按钮、`1180px / 680px` 响应式断点。

---

## 🎨 自定义 HTML 模板

`html-template.html` 是带完整内联 CSS/JS 的骨架，正文以「内部账户支出实占」为 HTTP 范例。文件顶部注释列出了所有 `{{占位符}}` 与替换说明。要调整外观（配色、字体、布局）时改这个文件即可。

几条约定（生成文档时由 skill 自动遵守，手改模板时也请保持）：

- **图标是固定 UI 资产** —— 侧边导航与「方法说明」的图标是内置 Lucide 内联 SVG（`stroke="currentColor"` 自动继承分色），**不要替换成 emoji / Unicode 字符**（字形尺寸不一会错位）。
- **HTTP / RPC 二选一** —— meta 卡片和第 2 章各有 (A)HTTP / (B)RPC 两版（一版是 HTML 注释），按目标方法保留一版、删另一版。
- **保持自包含** —— CSS/JS 全内联，不要引外部资源。

---

## 📁 目录结构

```
method-api-doc-html/
├── SKILL.md            # skill 定义：触发条件、执行步骤、固定模板、质量要求
├── html-template.html  # 精美 HTML 模板（内联 CSS/JS + 7 章节骨架 + 占位符）
├── README.md           # 本文件
└── .gitignore          # 忽略 .DS_Store、同步软件临时文件等
```

---

## 📝 说明

- 个人 / 团队使用的 Claude Code skill。生成的文档全程中文，遵循项目约定。
- 改了 `SKILL.md` 或模板后，`git push` 即可；通过软链接安装的 Claude 会自动用到最新版。
