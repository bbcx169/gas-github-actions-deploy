---
name: gas-github-actions-deploy
description: >-
  引導在 Google Apps Script (GAS) 專案中建立 GitHub Actions CI/CD 工作流程，實現本機 git push 自動執行 clasp push 部署 GAS。
  當使用者要求「透過 GitHub Actions 自動部署 GAS」、「git push 自動啟動 clasp push」、需要設定 GAS CI/CD 或 GitHub Secrets (如 CLASPRC_JSON、CLASP_JSON)，或是討論 git commit / push 部署 GAS 時使用。
---

# GAS GitHub Actions CI/CD 自動部署指南

本 Skill 旨在協助在專案中建立 GitHub Actions 工作流程，實現本機只要執行 `git push`，GitHub 就能在雲端虛擬環境自動執行 `clasp push` 更新 Google Apps Script (GAS)。

---

## 觸發時機 (When to Use)

- 使用者要求「透過 GitHub Actions 自動部署 GAS」
- 使用者提及「git push 自動啟動 clasp push」
- 使用者需要設定 CI/CD 或 GAS 的 GitHub Secrets
- 使用者執行 `git commit` 或 `git push` 時希望建立自動化部署

---

## CI/CD 流程架構

```
[本機開發]
   │
   ├─► 1. git add . && git commit -m "update"
   ├─► 2. git push origin main
   │
[GitHub Actions (雲端 Ubuntu 環境)]
   │
   ├─► 3. 觸發 push 事件啟動 Workflow
   ├─► 4. actions/checkout 下載專案程式碼
   ├─► 5. actions/setup-node 安裝 Node.js 環境
   ├─► 6. 安裝 @google/clasp 工具
   ├─► 7. 從 GitHub Secrets 注入憑證至 ~/.clasprc.json 與 .clasp.json
   └─► 8. 執行 clasp push -f 強制覆寫部署至 GAS
```

---

## 執行步驟 (Execution Steps)

### 步驟 1：說明 CI/CD 流程與事前準備

在開始建立檔案前，向使用者簡要說明運作原理，並確認以下事前要求：
1. 本機已安裝並登入 clasp（即存在 `~/.clasprc.json`）。
2. Google 帳號已啟用 **Google Apps Script API**：
   - 前往 [Google Apps Script 偏好設定](https://script.google.com/home/usersettings)
   - 將「Google Apps Script API」切換為 **開啟 (ON)**。
3. 專案已關聯 GitHub Repository。

---

### 步驟 2：建立 GitHub Actions 工作流程設定檔

在專案根目錄建立 `.github/workflows/deploy.yml`。

#### 工作流程設定檔範本：

```yaml
name: Deploy Google Apps Script

on:
  push:
    branches:
      - main
      - master

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install clasp
        run: sudo npm install -g @google/clasp

      - name: Create clasprc credentials
        run: |
          echo "$CLASPRC_JSON" > ~/.clasprc.json
        env:
          CLASPRC_JSON: ${{ secrets.CLASPRC_JSON }}

      - name: Create .clasp.json if secret exists
        if: env.CLASP_JSON != ''
        run: |
          echo "$CLASP_JSON" > .clasp.json
        env:
          CLASP_JSON: ${{ secrets.CLASP_JSON }}

      - name: Push to Google Apps Script
        run: clasp push -f
```

> **提示**：若專案的 `.clasp.json` 已有加入 Git 版控，則 `Create .clasp.json if secret exists` 步驟會被跳過或覆寫；但建議將 `.clasp.json` 納入 `.gitignore`，改由 GitHub Secret 注入，以維護 scriptId 隱私。

---

### 步驟 3：設定 .gitignore 避免機密洩漏

引導使用者檢查專案根目錄的 `.gitignore`，確保包含以下內容：

```gitignore
.clasprc.json
.clasp.json
node_modules/
```

---

### 步驟 4：引導取得憑證並設定 GitHub Secrets

引導使用者至 GitHub Repository 設定金鑰：
1. 進入 GitHub 專案頁面 ➔ **Settings** ➔ 左側選單 **Secrets and variables** ➔ **Actions**。
2. 點擊 **New repository secret**，新增以下 Secrets：

#### 1. `CLASPRC_JSON` (必要)
- **說明**：本機端執行 `clasp login` 後產生的全域認證檔。
- **取得方式**：
  - **Windows (PowerShell)**：
    ```powershell
    Get-Content ~/.clasprc.json | Set-Clipboard
    ```
    *(或手動用記事本開啟 `C:\Users\<你的使用者名稱>\.clasprc.json`)*
  - **macOS / Linux**：
    ```bash
    cat ~/.clasprc.json | pbcopy   # macOS
    cat ~/.clasprc.json            # Linux
    ```
- **內容範例**：
  ```json
  {
    "token": {
      "access_token": "ya29...",
      "refresh_token": "1//...",
      "scope": "https://www.googleapis.com/auth/script.projects ...",
      "token_type": "Bearer",
      "expiry_date": 1234567890
    },
    "oauth2ClientSettings": {
      "clientId": "...",
      "clientSecret": "...",
      "redirectUri": "http://localhost"
    },
    "isLocalCreds": false
  }
  ```
- 將檔案的**完整 JSON 內容**貼入 Secret 值中。

#### 2. `CLASP_JSON` (建議)
- **說明**：專案目標 GAS Script ID 設定檔。
- **取得方式**：
  專案根目錄中的 `.clasp.json` 內容。
- **內容範例**：
  ```json
  {
    "scriptId": "1a2b3c4d5e...",
    "rootDir": "./src"
  }
  ```
  *(若原始碼就在根目錄，`rootDir` 為 `"./"` 或省略)*

---

### 步驟 5：後續驗證與除錯 (Verification)

1. **推送程式碼測試**：
   ```bash
   git add .
   git commit -m "ci: add GitHub Actions workflow for GAS deploy"
   git push origin main
   ```
2. **觀察部署狀態**：
   - 前往 GitHub Repo 頂部的 **Actions** 頁籤。
   - 點進最新觸發的 **Deploy Google Apps Script** 工作流程。
   - 確認各步驟（尤其是 `Push to Google Apps Script`）是否顯示綠色打勾。
3. **驗證 GAS 專案**：
   - 開啟目標 Google Apps Script 編輯器，重新整理頁面，確認最新程式碼已成功同步。

---

## 常見問題與注意事項 (Gotchas & Troubleshooting)

| 問題現象 | 原因分析 | 解決方式 |
| :--- | :--- | :--- |
| **User has not enabled the Apps Script API** | Google 帳號未開啟 API 權限 | 前往 [GAS User Settings](https://script.google.com/home/usersettings) 開啟「Google Apps Script API」。 |
| **Could not read API credentials / invalid_grant** | Token 過期或認證失效 | Refresh Token 可能因長期未用或 Google 安全政策失效。請本機重新執行 `clasp login`，重新複製 `~/.clasprc.json` 並更新 GitHub Secret `CLASPRC_JSON`。 |
| **CI 執行卡在 y/n 確認提示** | `clasp push` 在無互動環境下偵測到檔案衝突 | 務必在命令中加上 `-f`（即 `clasp push -f`），強制覆寫目標專案。 |
| **JSON Parse error / Invalid JSON** | Secret 複製時遺漏大括號或格式截斷 | 檢查 GitHub Secret 是否完整包含最外層 `{` 與 `}`，不要有多餘的換行或引號。 |
| **Manifest file not found** | 找不到 `appsscript.json` | 確保專案中（或 `rootDir` 指定的目錄內）包含 `appsscript.json` 專案設定檔。 |
