# gas-github-actions-deploy

這個 repository 存放了 Google Apps Script (GAS) 的 GitHub Actions CI/CD 自動部署技能 (Skill)。

實現本機只要執行 `git push`，GitHub Actions 就能在雲端自動執行 `clasp push -f` 部署更新 GAS。

## 安裝與使用方式

把這行貼給您的 AI 助理（如 Antigravity / Claude Code）：

```
這是 GAS GitHub Actions 自動部署技能 https://github.com/bbcx169/gas-github-actions-deploy
請讀取 repo 內容並載入此技能。
```

AI 助理會自動讀取 `SKILL.md` 並學習如何引導您在專案中建立 GitHub Actions CI/CD 工作流程與設定 GitHub Secrets。

## 功能特點

- 🚀 **自動化 CI/CD**：`git push` 自動觸發虛擬環境並執行 `clasp push -f`
- 🔒 **憑證防護**：指引安全設定 `CLASPRC_JSON` 與 `CLASP_JSON`，避免敏感金鑰外洩
- ⚡ **開箱即用範本**：提供標準 `.github/workflows/deploy.yml` 工作流程設定檔（參見 [examples/deploy.yml](./examples/deploy.yml)）
- 🛠️ **常見問題排查**：內建 Token 過期、無互動環境卡死、Apps Script API 權限等除錯指引

## 目錄結構

```text
gas-github-actions-deploy/
├── SKILL.md            # Skill 主要說明與引導流程
├── README.md           # 專案介紹與載入說明
├── .gitignore          # 排除暫存檔與敏感憑證
└── examples/
    └── deploy.yml      # GitHub Actions 工作流程範本
```
