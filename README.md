# OpenClaw Chat Platform

基于 LangGraph 的 AI Agent 多轮对话平台，支持工具调用、长期记忆、安全审查和可观测性追踪。

## 架构概览

```
┌─────────────┐     SSE      ┌──────────────┐    Tool Call    ┌────────────┐
│   Frontend   │◄──────────►│    Backend    │◄──────────────►│   LLM API  │
│  (Next.js)   │  /api/chat  │  (FastAPI)    │                │  (智谱GLM)  │
└─────────────┘              └──────┬───────┘                └────────────┘
                                    │
                          ┌─────────┼─────────┐
                          ▼         ▼         ▼
                     ┌────────┐ ┌────────┐ ┌─────────┐
                     │Guardian│ │ Memory │ │  Tools   │
                     │ 安全审查│ │ 记忆模块│ │ 工具执行  │
                     └────────┘ └────────┘ └─────────┘
```

## 核心功能

| 模块 | 说明 |
|------|------|
| **Agent Loop** | 基于 LangGraph 的 ReAct 循环，支持多轮工具调用 |
| **Guardian 安全审查** | 对用户输入做前置安全检查，支持超时降级（open/closed 模式） |
| **长期记忆 v2** | 会话蒸馏 + 混合检索（BM25 关键词 + Dense 向量），跨会话记忆持久化 |
| **Skills 系统** | 可插拔技能包（天气查询、RAG 知识库、网页搜索等），启动时自动扫描注册 |
| **工具调用** | 内置 Terminal、PythonREPL、FetchURL、ReadFile、search_memory 等工具 |
| **Langfuse 追踪** | 可选接入，完整追踪 Agent 每一步决策和工具调用链路 |
| **流式响应** | 后端 SSE 流式输出，前端实时渲染对话和思考链 |

## 技术栈

**后端：**
- Python 3.10+ / FastAPI / Uvicorn
- LangGraph（Agent 编排）
- Chroma / PostgreSQL + pgvector（向量存储）
- Langfuse（可观测性）

**前端：**
- Next.js 14 / React / TypeScript
- Tailwind CSS

**LLM 支持：**
- 智谱 (Zhipu/GLM) / 百炼 (Bailian/Qwen) / DeepSeek / OpenAI 兼容接口

## 快速开始

### 1. 克隆项目

```bash
git clone https://github.com/pacer12321/openclaw_chat_platform.git
cd openclaw_chat_platform
```

### 2. 启动后端

```bash
cd backend

# 创建并激活虚拟环境
python -m venv .venv
# Windows:
.venv\Scripts\activate
# Linux/Mac:
source .venv/bin/activate

# 安装依赖
pip install -r requirements.txt

# 配置环境变量
copy config\.env.example config\.env
# 编辑 config\.env，填入 LLM_API_KEY 等必填项

# 启动服务
uvicorn app:app --host 0.0.0.0 --port 8002 --reload
```

### 3. 启动前端

```bash
cd frontend
npm install
npm run dev
```

### 4. 访问

浏览器打开 http://localhost:3000

## 环境变量

必填项：

| 变量 | 说明 |
|------|------|
| `LLM_PROVIDER` | LLM 提供商：zhipu / bailian / deepseek / openai |
| `LLM_MODEL` | 模型名称，如 glm-5 |
| `LLM_API_KEY` | 对应提供商的 API Key |

可选项详见 `backend/config/.env.example`。

## API 端点

| 端点 | 方法 | 说明 |
|------|------|------|
| `/health` | GET | 健康检查 |
| `/api/chat` | POST | 对话（SSE 流式） |
| `/api/sessions` | GET | 会话列表 |
| `/api/sessions/{id}` | GET | 会话详情 |
| `/api/files` | GET | 文件列表 |
| `/api/config` | GET/PUT | 配置读写 |

## 项目结构

```
openclaw_chat_platform/
├── backend/
│   ├── api/              # FastAPI 路由层
│   ├── config/           # 配置管理 & .env
│   ├── graph/            # LangGraph Agent 编排
│   │   ├── agent.py      # Agent 主循环
│   │   ├── guardian.py   # 安全审查模块
│   │   └── llm.py        # LLM 适配层
│   ├── memory_module_v1/ # 记忆模块 v1 (Chroma RAG)
│   ├── memory_module_v2/ # 记忆模块 v2 (蒸馏 + 混合检索)
│   ├── skills/           # 可插拔技能包
│   ├── tools/            # 内置工具
│   └── app.py            # 应用入口
├── frontend/
│   └── src/
│       ├── app/          # Next.js 页面
│       ├── components/   # UI 组件
│       └── lib/          # API & 状态管理
└── .gitignore
```

## License

MIT