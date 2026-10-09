# NEXUS SEARCH OMEGA v2.0.0

English / 中文

A privacy-first local search engine that combines a persistent SQLite index, LRU cache, and parallel external source queries into one fast and lightweight search experience.

中文：这是一个隐私优先的本地搜索引擎，结合持久化 SQLite 索引、LRU 缓存和并行外部搜索源，提供快速、轻量且可本地部署的搜索体验。

---

## English

## Overview

NEXUS SEARCH OMEGA is a local multi-source search engine designed for speed, privacy, and offline-first usage. It stores indexed pages in SQLite, caches repeated queries in memory, and queries multiple public sources in parallel when needed.

Unlike a pure browser search, this project is designed to be a lightweight local engine that can be run on a machine and used to search across multiple sources while keeping the result pipeline fast and controllable.

### Core Features

- Local SQLite index for persistent results
- LRU query cache with TTL
- Fast local match before external fallback
- Parallel HTTP queries across multiple sources
- Privacy-first design with zero tracking and zero telemetry
- Web UI served locally on port 8902
- Multi-source search aggregation
- Local index reuse for repeated queries
- Runnable with plain Python standard library

### Included Search Sources

- DuckDuckGo
- Wikipedia
- GitHub
- Hacker News
- ArXiv
- Stack Overflow
- Wikidata

### Design Goals

- Keep search fast for repeated queries
- Reduce network latency by using local cache and local index
- Keep the system simple and self-hostable
- Avoid dependency-heavy stacks
- Improve result ranking with score boosting and deduplication

### Key Performance Characteristics

- LRU cache hit: under 1 ms
- Same query within 300 seconds: served from cache
- Local index hit: 1–5 ms
- Term match in SQLite: very fast
- External search fallback: 500–1200 ms typically
- Up to 7 external sources queried in parallel

---

## How It Works

The workflow is straightforward:

1. A user submits a query via the local web interface or API.
2. The engine checks the in-memory LRU cache.
3. If the query is not cached, it searches the local SQLite index.
4. If local matches are insufficient, it performs parallel queries across external sources.
5. Results are deduplicated, ranked, and returned to the user.
6. Popular results are indexed locally so future runs are faster.

This creates a fast search loop with a strong local-first strategy.

### Result Ranking

The ranking logic combines:

- source score
- title match bonus
- snippet match bonus
- deduplication
- source weight boost
- URL quality boost

The final result list is sorted by score and returned with metadata such as:

- title
- snippet
- URL
- source
- score
- execution time

---

## Local Architecture

The project organizes itself around a few core components:

- `LocalIndex`: SQLite-backed page and term index
- `CACHE`: LRU memory cache with TTL
- `http_get`: safe HTTP fetcher with gzip support
- `search()`: main orchestrator for local + external query flow
- `rank()`: scoring and ranking
- `Handler`: HTTP server and API endpoints
- `HTML`: built-in local interface

### Data Storage

The engine stores its local state under:

- `~/Documents/nexus_search_omega/`
- `index.db`
- `cache/`
- `logs/`

This keeps the project portable and easy to manage locally.

---

## Quick Start

### Requirements

- Python 3.8+
- Standard library only
- Internet access for external sources

### Run

```bash
python3 nexus_search_omega.py
```

Then open:

```text
http://localhost:8902/
```

### API Endpoints

```text
GET /
GET /api/search?q=your+query
GET /api/health
GET /api/stats
GET /api/sources
```

### Example

```bash
curl "http://localhost:8902/api/search?q=machine%20learning"
```

---

## Configuration

The project reads a default port from environment variables:

```bash
PORT=8902 python3 nexus_search_omega.py
```

You can also modify the application behavior by adjusting:

- cache size
- cache TTL
- timeout values
- external source list
- result limits

---

## Security and Privacy

This project is made with privacy in mind:

- no analytics tracking
- no telemetry
- no cloud sync
- no user profiling
- no external account needed

The search engine only requests public sources and stores local data on the machine.

---

## License

This project is distributed under the NEXUS-OPEN-2.0 policy and includes the repository license file.

---

## Author

Aissa Mohammedi (DGK)

---

## 中文

## 项目概览

NEXUS SEARCH OMEGA 是一个本地优先的多源搜索引擎，目标是提供快速、隐私友好、离线可用的搜索体验。它使用 SQLite 持久化索引、LRU 内存缓存，以及并行外部源查询来实现稳定而高效的搜索。

这个项目不是单一搜索引擎，而是一个轻量型本地搜索中枢，可以在本地机器上运行，并在多个公共搜索源之间进行整合与聚合。

### 核心功能

- 持久化 SQLite 索引
- TTL 机制的 LRU 查询缓存
- 本地快速命中优先，外部搜索作为补充
- 多个搜索源并行 HTTP 请求
- 隐私优先：无跟踪、无遥测、无云同步
- 本地 Web 界面，端口 8902
- 多源搜索结果聚合
- 重复查询可直接命中缓存
- 仅依赖 Python 标准库

### 支持的搜索源

- DuckDuckGo
- Wikipedia
- GitHub
- Hacker News
- ArXiv
- Stack Overflow
- Wikidata

### 设计目标

- 让重复查询更快
- 通过本地缓存和索引减少网络延迟
- 保持项目轻量、便于自托管
- 避免重型依赖栈
- 通过评分和去重提升结果质量

### 关键性能特征

- LRU 缓存命中：小于 1 ms
- 300 秒内同一查询：直接命中缓存
- 本地索引命中：1–5 ms
- SQLite 词项匹配：非常快
- 外部搜索回退：通常 500–1200 ms
- 最多 7 个外部源并行查询

---

## 工作原理

搜索流程包括以下步骤：

1. 用户在本地网页或 API 中提交查询。
2. 引擎优先检查内存中的 LRU 缓存。
3. 若未命中缓存，则搜索本地 SQLite 索引。
4. 如果本地结果不够，系统并行请求多个外部源。
5. 对结果去重、排序并返回给用户。
6. 常用结果会写入本地索引，供后续查询复用。

这形成了一个“本地优先 + 外部补充”的高性能搜索链路。

### 结果排序

排序逻辑综合考虑：

- 来源分数
- 标题匹配加成
- 摘要匹配加成
- 去重
- 来源权重加成
- URL 质量加成

最终返回列表按分数排序，并附带：

- 标题
- 摘要
- URL
- 来源
- 分数
- 耗时

---

## 本地架构

这个项目的核心组件包括：

- `LocalIndex`：基于 SQLite 的页面与词项索引
- `CACHE`：带 TTL 的 LRU 内存缓存
- `http_get`：带 gzip 支持的安全 HTTP 请求器
- `search()`：主搜索协调器
- `rank()`：打分与排序模块
- `Handler`：HTTP 服务和 API 处理器
- `HTML`：内置本地 Web 界面

### 数据存储

本地状态默认写入：

- `~/Documents/nexus_search_omega/`
- `index.db`
- `cache/`
- `logs/`

这样可以保证运行环境便携、可控、易维护。

---

## 快速开始

### 前提条件

- Python 3.8+
- 仅依赖标准库
- 外部搜索源需要网络连接

### 运行

```bash
python3 nexus_search_omega.py
```

然后打开：

```text
http://localhost:8902/
```

### API 接口

```text
GET /
GET /api/search?q=your+query
GET /api/health
GET /api/stats
GET /api/sources
```

### 示例

```bash
curl "http://localhost:8902/api/search?q=machine%20learning"
```

---

## 配置说明

项目默认使用环境变量控制端口：

```bash
PORT=8902 python3 nexus_search_omega.py
```

你也可以调整这些参数：

- 缓存大小
- 缓存 TTL
- 超时时间
- 外部搜索源列表
- 返回结果数量上限

---

## 安全与隐私

这个项目遵循隐私优先原则：

- 无分析追踪
- 无遥测收集
- 无云同步
- 无用户画像
- 无外部账号要求

搜索引擎只访问公开源，并将数据保存在本地机器中。

---

## 许可证

本项目依据 NEXUS-OPEN-2.0 规则发布，并附带仓库中的许可文件。

---

## 作者

Aissa Mohammedi (DGK)

---

## Summary

NEXUS SEARCH OMEGA is a high-performance local search engine that mixes local persistence, LRU caching, and aggregated multi-source search into a clean and privacy-conscious tool. It is lightweight, self-hostable, and easy to extend.

中文总结：NEXUS SEARCH OMEGA 是一个高性能的本地搜索引擎，它将本地持久化、LRU 缓存和多源聚合搜索结合起来，形成一个轻量、私密且易扩展的搜索工具。
