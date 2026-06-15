# PaiCLI

PaiCLI 是一个使用 Java 17 构建的本地 Agent 命令行工具。它以 ReAct 循环为默认执行方式，并提供计划执行、多 Agent 协作、长期记忆、代码检索、人工审批、MCP 扩展、浏览器操作和 Skill 系统。

当前 CLI 版本：`v15.0.0`

## 主要能力

- ReAct Agent：模型可在思考、调用工具和继续执行之间循环，直到完成任务。
- Plan-and-Execute：生成执行计划，经用户确认后按依赖关系执行任务。
- Multi-Agent：提供 Planner、Worker 和 Reviewer 角色协作。
- Memory：支持短期上下文、长期记忆、摘要压缩和相关记忆检索。
- RAG：使用 Embedding、SQLite 和 JavaParser 建立代码索引并进行语义检索。
- HITL：对写文件、执行命令、创建项目及 MCP 操作提供人工审批。
- 安全策略：包含项目路径围栏、危险命令拦截、资源限制和 JSONL 审计日志。
- 多模型：支持 GLM 和 DeepSeek，并可在运行时切换。
- Web 工具：支持互联网搜索、网页正文抓取和 Markdown 提取。
- MCP：支持 stdio 与 Streamable HTTP，可加载外部工具、resources 和 prompts。
- 浏览器能力：通过 Chrome DevTools MCP 操作网页，并可按需复用本机 Chrome 登录态。
- Skill 系统：支持内置、用户级和项目级 `SKILL.md`，由 Agent 按任务动态加载。
- 并行执行：同一轮独立工具调用及 DAG 中无依赖任务可以并行运行。

## 环境要求

- Java 17 或更高版本
- Maven 3.8+
- GLM 或 DeepSeek API Key
- 可选：Ollama，用于本地代码 Embedding
- 可选：Node.js 与 Chrome，用于 Chrome DevTools MCP

## 快速开始

克隆项目并进入目录：

```bash
git clone https://github.com/Try-C/AI-Agent-CLI.git
cd AI-Agent-CLI
```

创建本地配置：

```bash
cp .env.example .env
```

Windows PowerShell 可使用：

```powershell
Copy-Item .env.example .env
```

至少配置一个模型 Key：

```dotenv
GLM_API_KEY=your_api_key_here
# DEEPSEEK_API_KEY=your_api_key_here
```

构建并运行：

```bash
mvn clean package
java -jar target/paicli-1.0-SNAPSHOT.jar
```

也可以直接通过 Maven 启动：

```bash
mvn clean compile exec:java -Dexec.mainClass="com.paicli.cli.Main"
```

## 常用命令

| 命令 | 说明 |
| --- | --- |
| `/plan <任务>` | 使用 Plan-and-Execute 模式执行任务 |
| `/team <任务>` | 使用 Multi-Agent 模式执行任务 |
| `/model` | 查看当前模型 |
| `/model glm` | 切换到 GLM |
| `/model deepseek` | 切换到 DeepSeek |
| `/context` | 查看上下文窗口和记忆状态 |
| `/memory` | 查看长期记忆状态 |
| `/save <事实>` | 保存稳定事实或长期偏好 |
| `/index [路径]` | 建立代码库索引 |
| `/search <查询>` | 语义检索代码 |
| `/graph <类名>` | 查看代码关系图谱 |
| `/hitl on` | 启用危险操作人工审批 |
| `/policy` | 查看安全策略 |
| `/audit [N]` | 查看最近的危险操作审计记录 |
| `/mcp` | 查看 MCP Server 状态 |
| `/browser connect` | 连接允许远程调试的本机 Chrome |
| `/browser disconnect` | 切回隔离浏览器模式 |
| `/skill list` | 查看可用 Skill |
| `/skill show <name>` | 查看 Skill 内容 |
| `/clear` | 清空当前对话历史 |
| `/exit` | 退出程序 |

## 代码检索配置

默认使用本机 Ollama：

```dotenv
EMBEDDING_PROVIDER=ollama
EMBEDDING_MODEL=nomic-embed-text:latest
EMBEDDING_BASE_URL=http://localhost:11434
```

首次检索前执行：

```text
/index
/search Agent 如何执行工具调用
```

## Web 搜索

支持以下 Provider：

- `zhipu`：使用 `GLM_API_KEY`
- `serpapi`：使用 `SERPAPI_KEY`
- `searxng`：使用 `SEARXNG_URL`

可通过环境变量显式指定：

```dotenv
SEARCH_PROVIDER=zhipu
```

`web_fetch` 适合静态或服务端渲染页面。对于 SPA、需要交互或需要登录态的页面，Agent 可以转用 Chrome DevTools MCP。

## MCP 配置

PaiCLI 按以下顺序加载 MCP 配置：

1. 用户级：`~/.paicli/mcp.json`
2. 项目级：`.paicli/mcp.json`

项目级配置会按 Server 名覆盖用户级配置。示例：

```json
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": ["-y", "chrome-devtools-mcp@latest", "--isolated=true"]
    }
  }
}
```

## Skill 目录

Skill 按以下优先级加载，后加载的同名 Skill 会覆盖前者：

1. Jar 内置：`src/main/resources/skills/`
2. 用户级：`~/.paicli/skills/<name>/SKILL.md`
3. 项目级：`.paicli/skills/<name>/SKILL.md`

## 安全说明

- 文件工具默认只能访问项目根目录以内的路径。
- `write_file` 单文件上限为 5 MB。
- `execute_command` 默认超时为 60 秒，并限制输出长度。
- 危险工具会写入 `~/.paicli/audit/` 下的 JSONL 审计日志。
- HITL 默认关闭，处理重要项目时建议执行 `/hitl on`。
- 本项目的策略层不是容器或虚拟机沙箱，运行模型生成的命令前仍需自行判断风险。

## 测试

运行全部测试：

```bash
mvn test
```

运行指定测试：

```bash
mvn test -Dtest=ExecutionPlanTest
```

部分 RAG 测试需要本机 Embedding 服务，部分命令测试会受操作系统 Shell 行为影响。

## 项目结构

```text
src/main/java/com/paicli/
├── agent/      Agent、计划执行与多 Agent 协作
├── browser/    Chrome 会话与敏感页面策略
├── cli/        命令行入口和交互
├── context/    长上下文策略
├── hitl/       人工审批
├── llm/        模型客户端
├── mcp/        MCP 客户端与传输层
├── memory/     短期和长期记忆
├── policy/     路径、命令和审计策略
├── rag/        代码索引与检索
├── skill/      Skill 加载与状态管理
├── tool/       内置工具注册与执行
└── web/        搜索与网页抓取
```

## License

当前仓库未附带开源许可证。未经明确许可，不代表授予复制、修改或分发代码的权利。
