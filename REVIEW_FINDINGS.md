# Claude Plugins Official — 代码检视报告

> **检视日期**: 2026-06-26  
> **检视范围**: 全部 36 个内置插件 + 15 个外部插件 + marketplace.json  
> **检视方法**: 结构化代码审查 + 安全审计 + 一致性校验

---

## 问题总览

| 严重级别 | 数量 | 类别 |
|----------|------|------|
| 🔴 Critical | 13 | 缺失 plugin.json manifest |
| 🟠 High | 12 | 弃用字段、缺失 README、文档不足 |
| 🟡 Medium | 4 | 规范性问题、配置缺失 |
| 🟢 Low | 3 | 改进建议 |

---

## 🔴 Critical: 缺失 plugin.json manifest（13 个）

以下插件在 `marketplace.json` 中已注册但缺少 `.claude-plugin/plugin.json` manifest 文件，**无法被插件系统正确发现和安装**：

### LSP 插件（12 个）

| 插件 | 现有文件 | 缺失 |
|------|----------|------|
| `clangd-lsp` | LICENSE + README.md | plugin.json |
| `csharp-lsp` | LICENSE + README.md | plugin.json |
| `gopls-lsp` | LICENSE + README.md | plugin.json |
| `jdtls-lsp` | LICENSE + README.md | plugin.json |
| `kotlin-lsp` | LICENSE + README.md | plugin.json |
| `lua-lsp` | LICENSE + README.md | plugin.json |
| `php-lsp` | LICENSE + README.md | plugin.json |
| `pyright-lsp` | LICENSE + README.md | plugin.json |
| `ruby-lsp` | LICENSE + README.md | plugin.json |
| `rust-analyzer-lsp` | LICENSE + README.md | plugin.json |
| `swift-lsp` | LICENSE + README.md | plugin.json |
| `typescript-lsp` | LICENSE + README.md | plugin.json |

### 技能插件（1 个）

| 插件 | 现有文件 | 缺失 |
|------|----------|------|
| `session-report` | LICENSE + skills/SKILL.md + template | plugin.json |

**修复建议**：为每个 LSP 插件添加 `.claude-plugin/plugin.json`，参考格式：

```json
{
  "name": "clangd-lsp",
  "description": "C/C++ language server providing code intelligence via clangd. Requires clangd installed locally.",
  "author": { "name": "Anthropic" }
}
```

对于 `session-report`，添加 manifest 并声明其 skill。如果是 skill-bundle 类型（无 manifest），需在 marketplace.json 中将 `strict` 设为 `false` 并显式声明 `skills`。

---

## 🟠 High

### 1. marketplace.json 使用了弃用字段 `commit`（2 处）

`schema` 定义使用 `sha` 字段指定提交版本，但以下插件使用了 `commit`：

```json
// fullstory
"source": { "source": "github", "repo": "fullstorydev/fullstory-skills", "commit": "1ec5865..." }

// jfrog  
"source": { "source": "github", "repo": "jfrog/claude-plugin", "commit": "259c8e7..." }
```

**修复**：将 `commit` 改为 `sha`。

### 2. 10 个外部插件缺少 README.md

| 插件 | Manifest | .mcp.json | README.md |
|------|----------|-----------|-----------|
| asana | ✅ | ✅ | ❌ |
| context7 | ✅ | ✅ | ❌ |
| firebase | ✅ | ✅ | ❌ |
| github | ✅ | ✅ | ❌ |
| gitlab | ✅ | ✅ | ❌ |
| laravel-boost | ✅ | ✅ | ❌ |
| linear | ✅ | ✅ | ❌ |
| playwright | ✅ | ✅ | ❌ |
| serena | ✅ | ✅ | ❌ |
| terraform | ✅ | ✅ | ❌ |

这些插件是 MCP 服务器类型（都有 `.mcp.json`），但用户无法通过 README 了解安装后如何配置、需要哪些环境变量、有哪些功能。

**修复**：为每个外部插件添加 README.md，至少包含：
- 插件功能简介
- 所需环境变量（如 `GITHUB_PERSONAL_ACCESS_TOKEN`）
- 配置说明
- 官方文档链接

### 3. 安全性：环境变量命名不一致

`.mcp.json` 中引用环境变量的命名规范不统一：

```json
// github → 正确的 kebab-case + 前缀
"Authorization": "Bearer ${GITHUB_PERSONAL_ACCESS_TOKEN}"

// terraform → 使用不同的命名风格
"-e", "TFE_TOKEN=${TFE_TOKEN}"
```

**建议**：统一推荐使用 `*_TOKEN` 或 `*_API_KEY` 后缀，并在 README 中明确说明。

---

## 🟡 Medium

### 4. `.gitignore` 不完整

当前 `.gitignore`:
```
*.DS_Store
.claude/
```

缺少常见的开发环境忽略项。

**修复建议**：
```gitignore
*.DS_Store
.claude/
node_modules/
__pycache__/
*.pyc
.env
.env.local
*.log
.vscode/
.idea/
```

### 5. LSP 插件 README 过于简陋

12 个 LSP 插件中，仅 `kotlin-lsp` 有一行描述，其余仅包含标题 `# plugin-name`。缺少：

- 前置依赖说明（需要安装的 LSP 二进制）
- 功能范围说明
- 配置方法
- 已知限制

### 6. `code-simplifier` 缺少 README.md

`plugins/code-simplifier/` 仅包含 LICENSE 和 agents 目录，没有 README。该插件的 agent prompt 硬编码了 JavaScript/TypeScript 特定的编码规范（ES modules、React 组件），但对非 JS 项目适用性差。

**修复建议**：
- 添加 README.md 说明适用范围
- Agent prompt 中的语言特定规范应改为通用指导，或注明仅适用于 JS/TS 项目

### 7. `learning-output-style`/`explanatory-output-style` 使用非标准 hooks 格式

这两个插件使用 `hooks/hooks.json` + shell 脚本的方式定义 hooks，但 Claude Code 标准 hooks 配置应该在 `settings.json` 中。`hooks.json` 不是标准文件格式。

**建议**：将 hooks 配置移至 `.claude-plugin/plugin.json` 的 hooks 声明中，或提供文档说明这种格式是特定于插件的。

---

## 🟢 Low

### 8. `code-review` 命令的 `disable-model-invocation: false`

`plugins/code-review/commands/code-review.md` 设置 `disable-model-invocation: false`。这意味着执行该命令时模型会重新解读命令内容，可能产生意外的二次行为。大多数命令应设为 `true` 以直接执行命令逻辑。

**建议**：确认是否为有意设置，如果是，在命令文档中说明原因。

### 9. 外部插件依赖的远端仓库可能不可用

marketplace.json 中 189 个插件指向外部 Git 仓库（`github`、`url`、`git-subdir` 源），这些仓库可能被删除、重命名或变为私有。当前没有监控机制检测这些外部依赖的可用性。

**建议**：在 CI 中添加定期巡检，检测 404/403 的远端仓库。

### 10. GitHub MCP 依赖特定服务端点

`external_plugins/github/.mcp.json` 指向 `https://api.githubcopilot.com/mcp/`，这是一个 GitHub Copilot 特定的 API 端点。如果该服务变更或被取代，插件会完全失效。没有备用的开源 GitHub MCP server 实现。

---

## ✅ 正面发现

- **无硬编码密钥泄露**：全库扫描未发现 API key、token 硬编码
- **CI/CD 完善**：8 个工作流覆盖验证、许可检查、SHA bump、安全检查
- **许可证合规**：所有内置插件均有 Apache 2.0 LICENSE 文件
- **插件结构规范**：大部分插件遵循标准的 `.claude-plugin/plugin.json` + commands/agents/skills 结构
- **marketplace.json 无重复**：240 个插件均唯一注册
- **安全审计插件内置**：`security-guidance` 插件提供注入、XSS、SSRF 等 25+ 漏洞类型检测

---

## 修改优先级建议

1. **立即修复**（本周内）：为 13 个缺失 manifest 的插件添加 `plugin.json`
2. **尽快修复**：将 `commit` → `sha`、为 10 个外部插件添加 README
3. **下个迭代**：完善 LSP 插件文档、补充 `.gitignore`、统一环境变量命名
4. **持续改进**：外部依赖监控、`code-review` 命令审查、hooks 格式标准化
