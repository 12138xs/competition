# 基于 Ray+MindSpore+Milvus 实现开箱即用的分布式 RAG

[![License](https://img.shields.io/badge/license-Apache%202.0-blue.svg)](LICENSE)
[![Python](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/)
[![Docker](https://img.shields.io/badge/docker-compose-blue.svg)](https://docs.docker.com/compose/)

一个基于微服务架构的分布式检索增强生成（RAG）系统，集成了 MindSpore 深度学习框架、Milvus 向量数据库和 Ray 分布式计算框架，提供开箱即用的 RAG 解决方案。

## 🚀 项目特色

- **微服务架构**: 采用 Docker Compose 编排，服务解耦，易于扩展和维护
- **国产化支持**: 基于华为 MindSpore 深度学习框架，支持昇腾 AI 处理器
- **高性能向量检索**: 集成 Milvus 向量数据库，支持大规模向量相似性搜索
- **分布式计算**: 预留 Ray 框架支持，为大规模分布式计算做准备
- **开箱即用**: 一键启动，无需复杂配置
- **多格式文档支持**: 支持 PDF、Markdown、TXT 等多种文档格式

## 🏗️ 系统架构

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Main App      │    │  Embedding      │    │   LLM Server    │
│  (Streamlit)    │◄──►│   Server        │◄──►│  (MindSpore)    │
│                 │    │  (MindNLP)      │    │                 │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         │                       │                       │
         ▼                       ▼                       ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│     MinIO       │    │     Milvus      │    │      etcd       │
│  (对象存储)      │    │   (向量数据库)    │    │   (元数据存储)    │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### 核心组件

- **Main App**: 基于 Streamlit 的 Web 界面，提供文档上传和问答功能
- **Embedding Server**: 基于 MindNLP 的文本向量化服务
- **LLM Server**: 基于 MindSpore 的大语言模型推理服务
- **Milvus**: 高性能向量数据库，用于存储和检索文档向量
- **MinIO**: 对象存储服务，用于存储原始文档
- **etcd**: 分布式键值存储，为 Milvus 提供元数据服务

## 📋 环境要求

- **操作系统**: Linux (推荐 Ubuntu 20.04+)
- **Docker**: 20.10+
- **Docker Compose**: 2.0+
- **内存**: 至少 8GB RAM (推荐 16GB+)
- **存储**: 至少 10GB 可用空间
- **网络**: 需要访问 Hugging Face 模型仓库

## 🛠️ 快速开始

### 一键启动完整环境

```bash
# 1. 克隆项目
git clone <repository-url>
cd DistributedRAG

# 2. 一键启动所有服务
docker-compose up -d

# 3. 等待服务启动完成（首次启动需要下载模型，约5-10分钟）
docker-compose logs -f
```

### 访问应用

启动完成后，打开浏览器访问：
- **主应用界面**: http://localhost:7860
- **MinIO 控制台**: http://localhost:9001 (用户名/密码: minioadmin/minioadmin)

### 开发模式

如需修改代码，直接编辑以下文件即可：
- `main_app/main.py` - 主应用逻辑
- `embedding_server/app.py` - 向量化服务
- `llm_server/app.py` - LLM推理服务

修改后重启对应服务：
```bash
# 重启特定服务
docker-compose restart main_app
docker-compose restart embedding-server
docker-compose restart llm-server
```

### 服务端口说明

- **7860**: 主应用 (Streamlit)
- **8001**: Embedding服务
- **8002**: LLM服务  
- **9000/9001**: MinIO存储
- **19530**: Milvus向量数据库

## 📖 使用指南

### 文档上传与处理

1. 打开浏览器访问 http://localhost:7860
2. 在左侧面板上传文档（支持 PDF、Markdown、TXT 格式）
3. 系统会自动处理文档：
   - 提取文本内容
   - 分块处理
   - 生成向量嵌入
   - 存储到 Milvus 向量数据库

### 智能问答

1. 在右侧问答区域输入问题
2. 系统会：
   - 将问题转换为向量
   - 在向量数据库中检索相关文档片段
   - 结合上下文生成回答

### 支持的文档格式

- **PDF**: 自动提取文本内容
- **Markdown**: 转换为纯文本
- **TXT**: 直接读取文本内容

## 🔧 配置说明

### 环境变量

主要配置在 `docker-compose.yml` 中：

```yaml
# 服务地址配置
EMBEDDING_SERVER_URL: http://embedding-server/embed
LLM_SERVER_URL: http://llm-server/generate

# MinIO 配置
MINIO_HOST: minio:9000
MINIO_ACCESS_KEY: minioadmin
MINIO_SECRET_KEY: minioadmin

# Milvus 配置
MILVUS_HOST: standalone
MILVUS_PORT: 19530
```

### 模型配置

#### Embedding 模型
- **默认模型**: `BAAI/bge-base-zh-v1.5`
- **向量维度**: 768
- **语言**: 中文优化

#### LLM 模型
- **默认模型**: `openbmb/MiniCPM-2B-dpo-bf16`

## 📁 项目结构

```
DistributedRAG/
├── docker-compose.yml          # Docker 编排配置
├── main_app/                   # 主应用服务
│   ├── main.py                # Streamlit 主程序
│   ├── requirements.txt       # Python 依赖
│   └── Dockerfile            # Docker 镜像配置
├── embedding_server/           # 向量化服务
│   ├── app.py                # FastAPI 服务
│   ├── requirements.txt       # Python 依赖
│   └── Dockerfile            # Docker 镜像配置
├── llm_server/               # LLM 推理服务
│   ├── app.py                # FastAPI 服务
│   ├── requirements.txt       # Python 依赖
│   └── Dockerfile            # Docker 镜像配置
├── test_data/                # 测试数据
│   └── test.txt              # 示例文档
└── volumes/                  # 数据卷
    ├── etcd/                 # etcd 数据
    ├── milvus/               # Milvus 数据
    └── minio/                # MinIO 数据
```

## 🔄 开发模式

### 代码修改

项目采用微服务架构，主要代码文件：
- `main_app/main.py` - 主应用逻辑和Web界面
- `embedding_server/app.py` - 文本向量化服务
- `llm_server/app.py` - 大语言模型推理服务

### 服务重启

修改代码后重启对应服务：
```bash
# 重启主应用
docker-compose restart main_app

# 重启向量化服务  
docker-compose restart embedding-server

# 重启LLM服务
docker-compose restart llm-server

# 查看服务状态
docker-compose ps
```

### 日志查看

```bash
# 查看所有服务日志
docker-compose logs -f

# 查看特定服务日志
docker-compose logs -f main_app
docker-compose logs -f embedding-server
docker-compose logs -f llm-server
```

## 📄 许可证

本项目采用 Apache 2.0 许可证 - 查看 [LICENSE](LICENSE) 文件了解详情。

## 🙏 致谢

- [MindSpore](https://www.mindspore.cn/) - 华为开源深度学习框架
- [MindNLP](https://github.com/mindspore-lab/mindnlp) - 自然语言处理工具包
- [Milvus](https://milvus.io/) - 向量数据库
- [MinIO](https://min.io/) - 对象存储服务
- [Streamlit](https://streamlit.io/) - Web 应用框架

## 📞 联系方式

如有问题或建议，请通过以下方式联系：

- 提交 Issue
- 发送邮件至 [your-email@example.com]
---

**注意**: 首次启动时，系统会自动下载模型文件，这可能需要较长时间，请耐心等待。