# mcp-audit

一个面向前端工程的安全审计工具，支持：

- 本地项目审计（读取本地 `package.json`）
- 远程 GitHub 仓库审计（读取仓库 `package.json`）
- 输出标准 Markdown 审计报告
- 以 MCP Tool 形式对外提供服务（`auditPackage`）

核心目标是：对项目本身以及所有直接/间接依赖进行漏洞扫描，并生成可直接展示或归档的审计结果。

## 功能特性

- 支持本地路径和 GitHub URL 两种输入
- 自动生成临时 `package-lock.json`，保证审计依赖树可解析
- 调用 `npm audit --json` 获取依赖漏洞
- 补充“当前项目本身”在 npm 审计接口下的漏洞信息
- 漏洞按严重级别归类：`critical` / `high` / `moderate` / `low`
- 使用 EJS 模板渲染 Markdown 报告
- 提供 MCP Server（stdio）供 AI 客户端或工具链集成

## 技术栈

- Node.js（ESM）
- `@modelcontextprotocol/sdk`
- `zod`
- `ejs`

## 项目结构

```text
src/
  audit/          # npm audit 调用、结果规格化、依赖链分析、当前项目审计
  parseProject/   # 本地/远程项目 package.json 解析
  generateLock/   # 生成临时 package-lock.json
  render/         # EJS 模板渲染 Markdown
  entry/          # 审计主流程入口 auditPackage
  workDir/        # 临时工作目录创建与清理
  mcpServer.js    # MCP 服务入口（注册 auditPackage 工具）
```

## 工作流程

`auditPackage(projectRoot, savePath)` 的执行链路如下：

1. 创建临时工作目录
2. 解析目标项目 `package.json`（本地或远程）
3. 在工作目录写入 `package.json` 并生成 `package-lock.json`
4. 执行 `npm audit --json`
5. 规格化漏洞结果，并补充当前项目本身漏洞信息
6. 渲染为 Markdown 字符串
7. 删除临时工作目录
8. 写入 `savePath`

## 安装

```bash
npm install
```

## 使用方式

### 1) 作为库函数调用

你可以直接调用入口函数 `auditPackage`：

```js
import { auditPackage } from './src/entry/index.js';

await auditPackage(
  'D:/workspace/my-project',
  'D:/workspace/my-project-audit.md'
);
```

远程仓库示例（GitHub）：

```js
import { auditPackage } from './src/entry/index.js';

await auditPackage(
  'https://github.com/webpack/webpack-dev-server/tree/v4.9.3',
  'D:/workspace/webpack-dev-server-audit.md'
);
```

### 2) 作为 MCP Server 运行

启动 MCP 服务（stdio transport）：

```bash
node src/mcpServer.js
```

服务会注册工具：`auditPackage`

工具入参：

- `projectRoot`: 本地项目根目录绝对路径，或 GitHub 仓库 URL
- `savePath`: 审计报告输出路径（建议绝对路径，如 `D:/audit.md`）

调用成功后返回文本：`审计完成，结果已保存到: <savePath>`

## 审计报告内容

输出为 Markdown，通常包含：

- 项目信息（名称、版本）
- 漏洞汇总（total / critical / high / moderate / low）
- 各级别漏洞详情
- 每条漏洞的问题描述（来源、范围、CVSS/CWE、链接等）
- 依赖链信息（帮助定位由谁引入）

## 输入说明

### 本地项目

- 必须存在可读取的 `package.json`
- 建议使用绝对路径

### 远程项目

- 当前实现仅支持 `github.com` URL
- 可使用仓库地址或 `tree/<tag|branch>` 路径

## 注意事项与限制

- 运行环境需要可用的 `npm` 命令
- 审计依赖 npm 官方安全接口与网络连通性
- 远程解析目前聚焦 GitHub，不支持 GitLab/Gitee
- 项目当前 `package.json` 未配置完整的 `test/lint/typecheck` 脚本，可按团队规范补充

## 代码入口参考

- 审计主流程：[src/entry/index.js](file:///e:/davon-project/MCP从入门到实战-资料1/MCP从入门到实战-资料/mcp-audit/src/entry/index.js#L1-L28)
- MCP 服务入口：[src/mcpServer.js](file:///e:/davon-project/MCP从入门到实战-资料1/MCP从入门到实战-资料/mcp-audit/src/mcpServer.js#L1-L43)
- 项目解析：[parseProject](file:///e:/davon-project/MCP从入门到实战-资料1/MCP从入门到实战-资料/mcp-audit/src/parseProject/index.js#L1-L18)
- 审计聚合：[audit](file:///e:/davon-project/MCP从入门到实战-资料1/MCP从入门到实战-资料/mcp-audit/src/audit/index.js#L1-L28)
- 报告渲染：[render](file:///e:/davon-project/MCP从入门到实战-资料1/MCP从入门到实战-资料/mcp-audit/src/render/index.js#L1-L24)
