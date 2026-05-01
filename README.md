# 建筑智能化 RAG MCP Agent

> 面向建筑智能化行业的可插拔、可观测 RAG 知识库服务框架。以楼宇自控（BAS）、建筑管理系统（BMS）、智能弱电系统操作手册、施工验收规范（GB/T 国标）、设备参数手册等技术文档为知识源，通过 MCP（Model Context Protocol）协议对外暴露工具接口，支持 AI 助手直接语义检索私有技术文档库。

---

## 📖 目录

- [项目概述](#-项目概述)
- [快速开始](#-快速开始)
- [目录结构](#-目录结构)
- [配置说明](#-配置说明)
- [MCP 集成](#-mcp-集成)
- [常见问题](#-常见问题)

---

## 🏗️ 项目概述

本项目针对建筑智能化系统集成场景下技术文档分散、工程师现场查询效率低的痛点，将**混合检索（Hybrid Search + Rerank）**、**多模态视觉处理（Image Captioning）**、**RAG 评估（Ragas + Custom）**、**生成（LLM Response）** 与 **MCP（Model Context Protocol）** 协议串联为完整的可运行工程。

系统支持将设备手册、施工验收规范、BMS 操作指南等 PDF 技术文档摄取为向量知识库，通过语义检索对外提供问答服务，可直接对接 GitHub Copilot、Claude Desktop 等 MCP Client。

得益于全链路可插拔架构，LLM / Embedding / Reranker / VectorStore 等后端均可通过配置文件一键切换，无需修改代码。

### 核心能力一览

| 模块 | 能力 | 说明 |
|------|------|------|
| **Ingestion Pipeline** | PDF → Markdown → Chunk → Transform → Embedding → Upsert | 全链路数据摄取，支持多模态图片描述（Image Captioning） |
| **Hybrid Search** | Dense (向量) + Sparse (BM25) + RRF Fusion + Rerank | 粗排召回 + 精排重排的两段式检索架构 |
| **MCP Server** | 标准 MCP 协议暴露 Tools | `query_knowledge_hub`、`list_collections`、`get_document_summary` |
| **Dashboard** | Streamlit 六页面管理平台 | 系统总览 / 数据浏览 / Ingestion 管理 / 摄取追踪 / 查询追踪 / 评估面板 |
| **Evaluation** | Ragas + Custom 评估体系 | 支持 Golden Test Set 回归测试，量化对比调优前后效果 |
| **Observability** | 全链路白盒化追踪 | Ingestion 与 Query 两条链路的每一个中间状态透明可见 |

### 技术亮点

**🔌 全链路可插拔架构**：LLM / Embedding / Reranker / Splitter / VectorStore / Evaluator 每一个核心环节均定义了抽象接口，通过配置文件一键切换后端，零代码修改。

**🔍 混合检索 + 重排**：BM25 稀疏检索解决专有名词精确匹配（如设备型号、标准编号）+ Dense Embedding 解决语义近义匹配，RRF 融合后可选 Cross-Encoder / LLM Rerank 精排，平衡查全率与查准率。

**🖼️ 多模态图像处理**：采用 Image-to-Text 策略，利用 Vision LLM 自动生成设备接线图、系统拓扑图等图片描述并缝合进 Chunk，复用纯文本 RAG 链路即可实现「搜文字出图」。

**📡 MCP 生态集成**：遵循 Model Context Protocol 标准，可直接对接 GitHub Copilot、Claude Desktop 等 MCP Client，零前端开发，一次开发处处可用。

**📊 可视化管理 + 自动化评估**：Streamlit Dashboard 提供完整的数据管理与链路追踪能力，集成 Ragas 等评估框架，建立基于数据的迭代反馈回路。

**🧪 三层测试体系**：Unit / Integration / E2E 分层测试，覆盖独立模块逻辑、模块间交互、完整链路（MCP Client / Dashboard），累计 1200+ 测试用例。

> 📖 详细架构设计与模块说明请参阅 [DEV_SPEC.md](DEV_SPEC.md)

---

## 🚀 快速开始

### 环境要求

- Python 3.11+
- [uv](https://docs.astral.sh/uv/)（推荐）或 pip
- LLM / Embedding API Key（支持 OpenAI、Azure OpenAI、DeepSeek、Ollama 等）

### 安装依赖

```bash
git clone https://github.com/xanderc123/Zhizhu-Tong-BA-RAG.git
cd Zhizhu-Tong-BA-RAG
uv sync
```

### 配置

复制并编辑配置文件：

```bash
cp config/test_credentials.yaml.example config/credentials.yaml
```

编辑 `config/settings.yaml`，填入 LLM / Embedding Provider 信息（详见[配置说明](#️-配置说明)）。

### 启动 Dashboard

```bash
uv run python scripts/start_dashboard.py
```

浏览器打开 `http://localhost:8501`，通过 Dashboard 上传 PDF 文档并触发摄取。

也可以使用内置的 **Setup Skill**，在 VS Code Copilot 对话框中输入：

```
setup
```

Agent 会自动引导完成全部配置流程。

### 摄取文档（命令行）

```bash
uv run python scripts/ingest.py --file /path/to/your/document.pdf
```

### 查询

```bash
uv run python scripts/query.py --query "GB 50339 对中央管理工作站响应时间的要求"
```

---

## 📁 目录结构

```
.
├── config/
│   ├── settings.yaml          # 主配置文件（LLM / Embedding / 检索参数）
│   └── prompts/               # Prompt 模板（支持独立迭代）
├── src/
│   ├── core/                  # 核心类型、设置、查询引擎
│   ├── ingestion/             # 摄取流水线（Load → Split → Transform → Embed → Upsert）
│   ├── libs/                  # 可插拔组件（LLM / Embedding / Reranker / VectorStore 等）
│   ├── mcp_server/            # MCP Server 实现
│   └── observability/         # 追踪、Dashboard
├── scripts/                   # 常用脚本（摄取、查询、评估、启动 Dashboard）
└── tests/                     # 三层测试（unit / integration / e2e）
```

---

## ⚙️ 配置说明

所有配置集中在 `config/settings.yaml`。

### LLM Provider

```yaml
llm:
  provider: "ollama"       # openai | azure | ollama | deepseek
  model: "qwen3:8b"
  base_url: "http://localhost:11434/v1"
  api_key: "ollama"
```

### Embedding Provider

```yaml
embedding:
  provider: "ollama"       # openai | azure | ollama
  model: "qwen3-embedding:8b"
  dimensions: 4096
```

### Reranker（可选）

```yaml
rerank:
  enabled: true
  provider: "cross_encoder"   # none | cross_encoder | llm
  model: "cross-encoder/ms-marco-MiniLM-L-6-v2"
  top_k: 5
```

### Vision LLM（可选，用于图片描述）

```yaml
vision_llm:
  enabled: true
  provider: "openai"
  model: "gpt-4o"
  api_key: "YOUR_API_KEY"
```

切换任意 Provider 只需修改对应字段，无需改动代码。

---

## 📡 MCP 集成

项目实现了标准 MCP Server（Stdio Transport），暴露以下三个工具：

| Tool | 功能 |
|------|------|
| `query_knowledge_hub` | 语义检索知识库，返回相关 Chunk 及 Citation |
| `list_collections` | 列出当前所有知识库集合 |
| `get_document_summary` | 获取指定文档的摘要信息 |

### 接入 GitHub Copilot（VS Code）

在 `.vscode/mcp.json` 中添加：

```json
{
  "servers": {
    "building-rag": {
      "type": "stdio",
      "command": "uv",
      "args": ["run", "python", "main.py"],
      "cwd": "/path/to/project"
    }
  }
}
```

### 接入 Claude Desktop

在 `claude_desktop_config.json` 中添加：

```json
{
  "mcpServers": {
    "building-rag": {
      "command": "uv",
      "args": ["run", "python", "main.py"],
      "cwd": "/path/to/project"
    }
  }
}
```

---

## ❓ 常见问题

### 如何切换 LLM / Embedding Provider？

修改 `config/settings.yaml` 中对应的 `provider` 字段即可，无需改代码。项目基于工厂模式实现，新增 Provider 只需：① 在 `src/libs/` 下新增 Provider 类；② 在工厂中注册；③ 更新配置。

### 如何支持 PDF 以外的文档格式？

Loader 层基于 `BaseLoader` 抽象接口，当前实现了 PDF Loader。扩展其他格式（Word、Markdown、HTML 等）只需新增对应 Loader 实现，参考 `src/libs/loader/` 目录。

### Custom Evaluator 和 Cross-Encoder Reranker 的状态？

这两个模块框架代码已完成，但尚未经过完整端到端测试：

| 模块 | 状态 |
|------|------|
| Custom Evaluator | 框架已有，需补充评估数据集 |
| Cross-Encoder Reranker | 框架已有，需下载本地重排模型 |

### 如何重置系统数据？

```bash
uv run python scripts/reset_system.py
```

该操作会清空 Chroma 向量库、BM25 索引和摄取历史记录。
