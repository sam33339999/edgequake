# EdgeQuake

> **高性能 Graph-RAG 框架（Rust 實現）**  
> 將文件轉化為智慧知識圖譜，實現卓越的檢索與生成能力

[![版本](https://img.shields.io/badge/version-0.4.0-blue.svg?style=flat)](CHANGELOG.md)
[![Rust](https://img.shields.io/badge/rust-1.78+-orange.svg?style=flat&logo=rust)](https://www.rust-lang.org)
[![授權](https://img.shields.io/badge/license-Apache%202.0-blue.svg?style=flat)](LICENSE)
[![建構狀態](https://img.shields.io/badge/build-passing-brightgreen.svg?style=flat)](https://github.com/raphaelmansuy/edgequake)
[![文件](https://img.shields.io/badge/docs-available-blue.svg?style=flat)](docs/README.md)

> **英文**：[README.md](README.md)

---

![EdgeQuake 前端截圖](docs/assets/01-screenshot.png)

## 為什麼選擇 EdgeQuake？

傳統的 RAG 系統僅使用向量相似度來檢索文件片段。這對於簡單查詢有效，但對於需要多跳推理（「X 如何通過 Z 與 Y 相關？」）、主題性問題（「主要主題是什麼？」）以及關係查詢則會失敗。核心問題在於：**向量捕捉了語義相似性，但丟失了概念之間的結構關係**。

**EdgeQuake** 透過在 Rust 中實現 [LightRAG 算法](https://arxiv.org/abs/2410.05779) 來解決這個問題：文件不僅僅是被切片和嵌入——它們被分解為由實體和關係構成的**知識圖譜**。在查詢時，系統同時遍歷向量空間和圖結構，結合向量搜尋的速度與圖遍歷的推理能力。

### EdgeQuake 的優勢

- **知識圖譜**：LLM 驅動的實體提取和關係映射，創建對文件的結構化理解——而不僅僅是關鍵字匹配
- **6 種查詢模式**：從快速的簡單向量搜尋到圖遍歷的混合查詢，每種模式針對不同問題類型進行優化
- **Rust 高性能**：以 Tokio 為基礎的異步架構，零拷貝操作——可處理數千個並發請求
- **PDF LLM 視覺管道 ✅ 0.4.0 新功能**：多模態 LLM（GPT-4o、Claude、Gemini）以圖片方式讀取 PDF 頁面——開箱即用地處理掃描文件、複雜表格和多欄佈局
- **生產就緒**：OpenAPI 3.0 REST API、SSE 串流、健康檢查、多租戶工作區隔離
- **現代前端**：React 19 搭配互動式 Sigma.js 圖譜視覺化

---

## 快速開始

### 前置條件

- **Rust**：1.78 或更高版本（[安裝 Rust](https://rustup.rs)）
- **Node.js**：18+ 或 Bun 1.0+（[安裝 Node](https://nodejs.org)）
- **Docker**：用於 PostgreSQL（[安裝 Docker](https://www.docker.com/get-started)）
- **Ollama**：用於本地 LLM（可選，[安裝 Ollama](https://ollama.ai)）

### 安裝（5 分鐘）

```bash
# 1. 克隆倉庫
git clone https://github.com/raphaelmansuy/edgequake.git
cd edgequake

# 2. 安裝依賴
make install

# 3. 啟動完整堆疊（PostgreSQL + 後端 + 前端）
make dev
```

**完成！** 🎉

- **後端**：http://localhost:8080
- **前端**：http://localhost:3000
- **Swagger UI**：http://localhost:8080/swagger-ui
- **LLM 提供者**：Ollama（本地，免費）

---

## 如何建構專案

### 建構後端（Rust）

```bash
# 進入 edgequake 目錄
cd edgequake

# 開發版建構
cargo build

# 正式版建構（優化）
cargo build --release

# 執行所有測試
cargo test

# 程式碼檢查（Lint）
cargo clippy

# 程式碼格式化
cargo fmt
```

### 建構前端（React / TypeScript）

```bash
# 進入前端目錄
cd edgequake_webui

# 安裝依賴（使用 Bun）
bun install

# 啟動開發伺服器
bun run dev

# 建構正式版
bun run build

# 執行前端測試
bun test
```

### 使用 Makefile 一鍵管理

EdgeQuake 提供統一的 Makefile，涵蓋所有常見開發任務：

```bash
# ── 完整開發堆疊 ──────────────────────────────────────────
make dev              # 啟動所有服務（PostgreSQL + 後端 + 前端）
make dev-bg           # 在後台啟動（適用於自動化/CI）
make stop             # 停止所有服務
make status           # 查看服務狀態

# ── 僅後端 ────────────────────────────────────────────────
make backend-dev      # 以 PostgreSQL 模式啟動後端
make backend-bg       # 在後台啟動後端
make backend-test     # 執行後端測試

# ── 僅前端 ────────────────────────────────────────────────
make frontend-dev     # 啟動前端開發伺服器
make frontend-build   # 建構前端正式版

# ── 資料庫 ────────────────────────────────────────────────
make db-start         # 啟動 PostgreSQL 容器
make db-stop          # 停止 PostgreSQL 容器

# ── 品質管控 ──────────────────────────────────────────────
make test             # 執行所有測試
make lint             # 檢查所有程式碼
make format           # 格式化所有程式碼
make clean            # 清除建構產物
```

### 環境變數

| 變數                           | 必填 | 用途                      | 範例                                                  |
| ------------------------------ | ---- | ------------------------- | ----------------------------------------------------- |
| `DATABASE_URL`                 | ✅   | PostgreSQL 連線字串       | `postgres://edgequake:edgequake@localhost/edgequake`  |
| `OPENAI_API_KEY`               | 可選 | 啟用 OpenAI 提供者        | `sk-proj-...`                                         |
| `EDGEQUAKE_LLM_PROVIDER`       | 可選 | 覆蓋 LLM 提供者           | `openai`、`ollama`、`lmstudio`、`mock`                |
| `EDGEQUAKE_EMBEDDING_PROVIDER` | 可選 | 混合模式：獨立嵌入提供者  | `ollama`                                              |
| `OLLAMA_HOST`                  | 可選 | Ollama 伺服器 URL         | `http://localhost:11434`                              |
| `RUST_LOG`                     | 可選 | 日誌級別                  | `debug`、`info`、`warn`                               |

---

## 功能特色

### 🚀 高性能

- **異步優先**：基於 Tokio 的運行時，最大化並發能力
- **零拷貝**：利用 Rust 所有權系統實現高效記憶體管理
- **並行處理**：多執行緒實體提取與嵌入計算
- **快速存儲**：PostgreSQL AGE 圖存儲 + pgvector 向量嵌入

### 知識圖譜

- **實體提取**：自動檢測人物、組織、地點、概念、事件、技術和產品（7 種可配置類型）
- **關係映射**：LLM 驅動的關係識別，附帶關鍵字標籤
- **多輪提取（Gleaning）**：多輪提取比單輪多捕獲 15-25% 的實體
- **社群偵測**：Louvain 模組化優化，將相關實體聚類以支援主題查詢
- **圖譜視覺化**：互動式 Sigma.js 前端，支援縮放/平移

### 📄 PDF 處理（0.4.0 已生產就緒）

- **文字模式**：快速的 pdfium 提取（預設，零配置）
- **視覺模式** ✨：LLM 以圖片方式讀取每頁——支援 GPT-4o、Claude 3.5+、Gemini 2.5
- **自動回退**：視覺模式失敗時自動回退到文字提取
- **嵌入式 pdfium**：無需設置 `PDFIUM_DYNAMIC_LIB_PATH` 環境變數

### 🔍 6 種查詢模式

1. **Naive（簡單）**：向量相似度搜尋——最快速（約 100-300ms）
2. **Local（局部）**：以實體為中心，遍歷局部圖鄰域——最適合具體關係查詢（約 200-500ms）
3. **Global（全局）**：基於社群的語義搜尋——最適合主題性/高層次問題（約 300-800ms）
4. **Hybrid（混合）**_（預設）_：結合局部實體上下文與全局社群上下文（約 400-1000ms）
5. **Mix（混合加強）**：可配置比例的簡單向量結果與圖增強結果的加權混合
6. **Bypass（繞過）**：不進行 RAG 檢索，直接將問題傳給 LLM

---

## 架構概覽

```
┌──────────────────────────────────────────────────────────┐
│                     EdgeQuake 系統                        │
└──────────────────────────────────────────────────────────┘

前端（React 19 + TypeScript）
  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐
  │  文件上傳     │  │  查詢介面    │  │  圖譜視覺化   │
  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘
         └──────────────────┴──────────────────┘
                            │
                            ▼
       REST API（Axum） — OpenAPI 3.0 / SSE 串流
                            │
                            ▼
後端（Rust — 11 個 Crate）
  edgequake-core      │ 流程協調與管道
  edgequake-llm       │ OpenAI、Ollama、LM Studio、Mock
  edgequake-storage   │ PostgreSQL AGE、記憶體適配器
  edgequake-api       │ REST API 伺服器
  edgequake-pipeline  │ 文件攝取管道
  edgequake-query     │ 查詢引擎（6 種模式）
  edgequake-pdf       │ PDF 提取（文字/視覺/混合）
                            │
              ┌─────────────┴─────────────┐
              ▼                           ▼
       LLM 提供者                    存儲後端
  • OpenAI (gpt-4.1-nano)        • PostgreSQL 15+
  • Ollama (gemma3:12b)          • Apache AGE（圖）
  • LM Studio（本地）            • pgvector（向量）
  • Mock（測試用）
```

---

## 文件索引

- [快速開始](docs/getting-started/quick-start.md)
- [安裝指南](docs/getting-started/installation.md)
- [REST API 參考](docs/api-reference/rest-api.md)
- [架構概覽](docs/architecture/overview.md)
- [LightRAG 算法深入解析](docs/deep-dives/lightrag-algorithm.md)
- [常見問題（FAQ）](docs/faq.md)
- [常見問題排除](docs/troubleshooting/common-issues.md)

---

## 貢獻

EdgeQuake 由 **Raphaël MANSUY** 創建的 **edgecode** SOTA 編碼助理開發，遵循規範驅動開發（Specification-Driven Development）方法，所有更改均在 `specs/` 目錄中規範後再實施。

- **GitHub Issues**：回報錯誤和請求功能
- **GitHub Discussions**：提問和分享想法
- **直接聯繫**：重大貢獻請聯繫 [@raphaelmansuy](https://github.com/raphaelmansuy)

詳見 [CONTRIBUTING.md](CONTRIBUTING.md)。

---

## 授權

本專案依據 Apache License 2.0 授權。  
詳情請參閱：http://www.apache.org/licenses/LICENSE-2.0
