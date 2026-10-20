# 贡献指南

感谢愿意一起把「医疗器械法规知识」做成可检索、可溯源、可审查的公共基础设施。

本项目的定位是**零依赖的静态前端工具**：不连后端、不调大模型、不上传用户文档。
所有能力都在浏览器里跑完。这个定位决定了下面这些约定不是风格偏好，而是架构前提。

---

## 一、可以贡献什么

| 类型 | 说明 | 落到哪个文件 |
| --- | --- | --- |
| 法规条目 | 补充缺失法规，修正文号 / 日期 / 效力状态 | `data/regulations-cn-ext.js`、`data/regulations-intl.js` |
| 核心条款要点 | 为已有法规补充或修订条款要点 | 同上 |
| 审查规则 | 新增文档结构、要素覆盖、规范检查、术语、时限口径 | `data/review-rules.js` |
| 自检条目 | 新增申报前自检项 | `data/self-check.js` |
| 注册流程 | 补充申报路径、环节时限、资料清单 | `data/registration-flow.js` |
| 全文获取通道 | 新增官方全文来源（必须 https） | `data/fulltext.js` |
| 缺陷与建议 | 提 Issue | — |

---

## 二、法规条目的硬性要求

### 1. 16 个必填字段，一个都不能少

```
id / title / region / level / docNo / issuer / issueDate / effectiveDate
status / priority / tags / summary / articles / officialUrl / source / lastVerified
```

### 2. 日期必须可核验

以官方来源为准：NMPA 公告通告页、国家标准全文公开系统（SAC）、中国政府网、eCFR、EUR-Lex。

> **无法核验发布日期的条目，宁可不收录，也不要写推测值。**
> 溯源准确度优先于条目数量 —— 这是本项目最重要的一条原则。

### 3. 禁止 ASCII 双引号

数据文件是 JS 字符串字面量。条款文本里出现 ASCII 双引号 `"` 会截断字符串，
**导致整份数据文件语法错误**。中文引号统一用 `「」`，书名号统一用 `《》`。

### 4. 分卷撰写

- 国内新增一律写入 `data/regulations-cn-ext.js`
- 海外新增写入 `data/regulations-intl.js`
- `data/regulations-cn.js`（主卷）保持稳定，不因批量扩充而反复改写

前端按 `id` 去重合并各卷，重复 id 会被校验脚本拦下。

### 5. 不得复制法规原文

`articles` 字段只写**人工整理的核心条款要点**，不是官方逐字全文。

**不接受**把法规原文、标准全文或大段近似原文提交进仓库 —— 这涉及版权与权威来源问题。
平台对外提供的是官方获取通道路径与本地解析能力，不是原文转载。

### 6. 引用的法规 id 必须真实存在

`basis`（注册流程 / 审查规则 / 自检条目）、`covers`（集市服务包）都是**法规 id 数组**，
`tools/validate.js` 会强制校验每一个引用。修改任何法规 id 前，先反查这四卷是否引用了它。

---

## 三、提交前必须跑通回归

```bash
npm install     # 仅首次：安装测试用 jsdom，站点本身零依赖
npm run check   # 依次跑 validate → smoke → build → smoke-standalone
```

也可以分步执行：

```bash
npm run validate
npm test
npm run build
npm run test:standalone
```

顺序不可调换 —— 最后一步读的是第三步产出的便携版。

当前基线：

| 步骤 | 期望结果 |
| --- | --- |
| `npm run validate` | 6 个校验块全部通过 |
| `npm test` | 272 / 272 |
| `npm run build` | 产出 `dist/医械法规云-单文件版.html`，约 699 KB |
| `npm run test:standalone` | 41 / 41 |

**任何一项失败都不要提交。** GitHub Actions 会在每个 PR 上跑同一套检查。

---

## 四、新增数据卷：必须同步五处

漏掉任何一处都会造成**静默失真**：校验不报错、便携版缺数据、自测照旧全绿。

1. `index.html` 的 `<script src>`
2. `tools/validate.js` 的 `DATA_FILES` + 对应校验块
3. `tools/build-standalone.js` 的读取与内联正则（含 `dsTotal` 统计）
4. `tools/smoke.js` 的 `evalFile` + 断言
5. `tools/smoke-standalone.js` 的内联断言

---

## 五、代码约定

- **零外部依赖**：不引入 CDN、框架、打包工具。数据通过 `<script>` 全局变量注入，
  以兼容 `file://` 直接打开与单文件分发。
- **动态节点一律事件委托**：`document.addEventListener("click", ...)` + `closest()`，
  不要用逐元素绑定（只在渲染那一刻生效）。
- **对外接口块放在模块 `const` 声明之后**，写在之前会触发 TDZ，
  整个脚本不执行。
- **同一个文件不要并行编辑**：多个编辑并发会因读写竞态互相覆盖，串行进行。
- **数据里新增「按钮元数据」时，同步检查渲染层是否认识这个 id**。
  漏了会生成 `data-export="undefined"`，点击无反应且控制台无报错。

---

## 六、明确不接受的贡献

- 内嵌法规原文、标准全文、受版权保护的指南正文
- 真实机构的名称、报价、评分、履约数据
  （集市必须保持 `demo: true`，且 `notice` 同时写明「演示」与「不经手交易」）
- 未经核验的文号与日期
- 引入外部依赖、需要后端或数据库的改动
- 把用户文档上传到服务器、或调用大模型的改动（本地确定性审查是本项目的设计底线）

---

## 七、提交 Pull Request

1. Fork 仓库，从 `main` 开分支
2. 一次 PR 聚焦一个主题
3. 描述里写清：改了什么、依据的官方来源、回归结果
4. 法规类改动请附官方链接

---

## 八、免责

本项目为辅助工具，**不构成法律意见**，也不是官方文本来源。
所有合规判定与注册申报决策，请以监管机构最新公布口径为准。
