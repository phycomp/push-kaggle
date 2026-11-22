---
name: push-kaggle
description: Use when publishing datasets or notebooks to Kaggle — dataset creation/versioning, notebook push, credentials setup, and Kaggle-specific limitations. Triggers on "kaggle", "kaggle dataset", "kaggle publish", "kaggle notebook", or when the user wants to upload data/documents to Kaggle.
---

# Push Kaggle — Kaggle 發布技能

Kaggle 只收 Notebooks / Datasets / Models，**不能託管互動網站**。角色是「資料與分析層」。

## When to Use

- 發布 Dataset（主題蒸餾文檔、資料集）
- 發布 Notebook（可重跑的 report）
- Kaggle 憑證設定
- Kaggle 發布問題排查

## Prerequisites

- Kaggle CLI 2.x：`pip install -U kaggle`
- 憑證：https://www.kaggle.com/settings → API → Create New Token → `~/.kaggle/kaggle.json`（chmod 600）
- 或環境變數：`KAGGLE_USERNAME` / `KAGGLE_KEY`

## Dataset 發布

### 基本流程

```bash
# 驗證憑證（唯讀，不印出 secret）
kaggle datasets list -m

# 打包 staging + 產生 metadata
bash scripts/push_kaggle_dataset.sh [slug] [--public] [--stock-only] [--dry-run]
```

### 腳本行為
1. 憑證檢查（`kaggle.json` 或 env vars，不印出 secret）
2. 打包 `data/`（`--stock-only` 只打包 `data/stock_topics` 9 檔）
3. 產生 `dataset-metadata.json`（CC-BY-4.0）與 `README.md`
4. 存在則 `datasets version -m`，否則 `datasets create`（預設 private，`--public` 公開）
5. 印出 `https://www.kaggle.com/datasets/<user>/<slug>`

### 純驗證打包
```bash
bash scripts/push_kaggle_dataset.sh [slug] --dry-run   # 無需憑證
```

## Notebook 發布

若要 Notebook（例如把台股本益比表格做成可重跑的 report）：

```bash
kaggle kernels push -p <dir>
```

`kernel-metadata.json` 需含：
- `id`, `title`, `code_file`, `language: python`, `kernel_type: script`
- `dataset_sources: ["<user>/<slug>"]`

**注意**：Notebook 執行環境沒有持久服務，不要期待跑起 Streamlit。

## 限制

- Kaggle **不提供 Streamlit 託管** — 只收 Notebooks / Datasets / Models
- 內容免責：投資文檔一律保留「歷史績效不代表未來」
- 免費帳號可用

## 驗證

```bash
kaggle datasets files <id>   # 核對檔案清單
```

## 常見問題

- **401/403**：`kaggle.json` 權限非 600 或 key 失效；先 `kaggle datasets list -m`（唯讀）驗證憑證
- **想要互動網站**：做不到 — Kaggle 沒有持久服務；互動版留在 Streamlit Cloud / HF Spaces，Kaggle 只發 Dataset/Notebook

## References

- `push-streamlit` skill — 完整推送流程（含 GitHub/Kaggle/HF）
- `scripts/push_kaggle_dataset.sh` — Kaggle 發布腳本
EOF