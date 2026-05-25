# AI Knowledge Graph — 架构设计

> 产品定位：用户输入笔记/文档 → AI 自动提取概念并建立关联 → 生成可交互图谱 → 支持自然语言提问。

---

## 功能规划

### Phase 1 — MVP
- [ ] 文本输入界面（粘贴 / 上传 Markdown）
- [ ] AI 自动提取实体节点与关系边
- [ ] 可交互知识图谱可视化
- [ ] 基础语义搜索

### Phase 2 — 问答能力
- [ ] 自然语言提问（RAG Pipeline）
- [ ] 图谱节点点击 → 显示相关原文
- [ ] Markdown 文件批量导入

### Phase 3 — 产品化
- [ ] 用户账户系统
- [ ] 多知识库管理
- [ ] 导出为 Obsidian / JSON
- [ ] 对外开放 / SaaS 化

---

## 技术架构

```
用户输入文本
    │
    ▼
┌─────────────────────────────────┐
│  后端 (Python FastAPI)           │
│                                 │
│  1. 文本切块 (Chunker)           │
│       │                         │
│       ├──▶ Embedding            │
│       │    └──▶ ChromaDB        │  ← 向量存储（语义搜索用）
│       │                         │
│       └──▶ LLM 实体抽取          │
│            └──▶ NetworkX Graph  │  ← 图结构存储（图谱渲染用）
│                                 │
│  2. RAG 问答 Pipeline            │
│     问题 → 向量检索 → 拼上下文    │
│     → GPT 回答                  │
└─────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────┐
│  前端 (Vue.js)                   │
│                                 │
│  InputPanel  — 文本输入          │
│  GraphView   — 图谱可视化        │
│  ChatPanel   — AI 问答           │
└─────────────────────────────────┘
```

---

## 技术选型

| 层级 | 技术 | 理由 |
|------|------|------|
| 前端框架 | Vue.js 3 | 团队已有经验 |
| 图谱可视化 | `force-graph` | 轻量、交互流畅 |
| 后端框架 | Python FastAPI | 简单快速，生态丰富 |
| 向量数据库 | ChromaDB | 本地零配置，易上手 |
| 图计算 | NetworkX | 标准 Python 图库 |
| LLM | GPT-4o-mini | 低成本，能力足够 |
| Embedding | text-embedding-3-small | 低成本，效果好 |

---

## 目录结构（代码仓库）

```
ai-knowledge-graph/         ← 独立代码仓库（不在 IdeaLab 里）
│
├── backend/
│   ├── main.py             ← FastAPI 入口
│   ├── ingestion.py        ← 文本切块 + Embedding + ChromaDB
│   ├── extraction.py       ← GPT-4o-mini 实体/关系抽取
│   ├── graph.py            ← NetworkX 图结构管理
│   ├── qa.py               ← RAG 问答 Pipeline
│   ├── requirements.txt
│   └── .env.example
│
└── frontend/
    ├── src/
    │   ├── App.vue
    │   ├── components/
    │   │   ├── InputPanel.vue
    │   │   ├── GraphView.vue
    │   │   └── ChatPanel.vue
    │   └── api/index.js
    └── package.json
```

---

## API 接口设计

| 方法 | 路径 | 说明 |
|------|------|------|
| POST | `/ingest` | 提交文本 → 切块 + Embedding + 实体抽取 → 返回更新后的图 |
| GET | `/graph` | 获取完整图谱数据（nodes + edges） |
| POST | `/search` | 语义搜索，返回最相关文本片段 |
| POST | `/ask` | 自然语言提问，返回 AI 回答 |
| DELETE | `/graph` | 清空图谱 |

---

## 核心数据格式

```json
{
  "nodes": [
    { "id": "rag", "label": "RAG", "type": "technology" },
    { "id": "vector_db", "label": "向量数据库", "type": "technology" }
  ],
  "edges": [
    { "source": "rag", "target": "vector_db", "relation": "depends on" }
  ]
}
```

---

*Last updated: 2026-05-25*
