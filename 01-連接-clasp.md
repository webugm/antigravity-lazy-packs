---
name: antigravity-gas-clasp
description: 在 AntiGravity 連接 Google Apps Script (clasp)。說「連接 clasp」「設定 clasp」「開發 GAS」「GAS 環境」時載入。
---

# 連接 Google Apps Script (clasp)（AntiGravity 版）

本指南協助 AI Agent 與開發者自動檢查、安裝並設定 Google Apps Script (GAS) 開發工具鏈，支援本地端代碼編寫與雲端雙向同步。

---

## 🛠️ 開發 GAS 必備工具清單

1. **Node.js & npm**：基礎執行環境與套件管理工具。
2. **@google/clasp**：Google 官方 Apps Script CLI 工具，負責本地代碼與雲端專案雙向同步 (`push`/`pull`)。
3. **Google 帳號憑證**：執行 `clasp login` 完成 OAuth 授權。
4. **Google Apps Script API 啟用**：必須在雲端控制台啟用 API 開關。
5. **@types/google-apps-script**（強烈建議）：提供本地 IDE / Agent 編寫 GAS 的 API 型別提示與語法檢查。

---

## 🤖 Agent 自動檢查流程

當使用者觸發本技能時，Agent 請依序執行以下步驟進行檢測：

### 步驟 1：檢測 Node.js 與 npm
執行終端機命令：
```bash
node -v
npm -v
```
- 若成功回傳版本號（如 `v20.x` 或更高），標記 ✅ 通過。
- 若提示找不到指令，進入「未安裝教學 - 安裝 Node.js」。

### 步驟 2：檢測 clasp CLI
執行終端機命令：
```bash
clasp -v
```
- 若成功回傳版本（如 `3.3.0`），標記 ✅ 通過。
- 若提示找不到指令，進入「未安裝教學 - 安裝 clasp」。

### 步驟 3：檢測 clasp 登入狀態
執行終端機命令：
```bash
clasp show-authorized-user
```
- 若回傳 `You are logged in as <email>`，標記 ✅ 登入有效。
- 若回傳未登入或錯誤，提示使用者執行 `clasp login`。

### 步驟 4：檢查 Google Apps Script API 開啟狀態
- 提示使用者確認是否已開啟 Google Apps Script API：
  👉 前往：https://script.google.com/home/usersettings
  👉 將「Google Apps Script API」切換為 **開啟 (ON)**（若未開啟，`clasp push` 會報錯 403 / API disabled）。

### 步驟 5：回報檢查總表
將檢查結果整理成清楚的 Markdown 檢查清單回報給使用者。

---

## 💻 跨平台安裝教學指引

若檢測發現工具未安裝，請根據使用者的作業系統提供對應說明：

### 一、Windows 系統安裝指引

#### 1. 安裝 Node.js & npm
- **推薦方式 (PowerShell 命令列)**：
  ```powershell
  winget install OpenJS.NodeJS.LTS
  ```
- **手動安裝方式**：
  前往 [Node.js 官方網站](https://nodejs.org/) 下載 **LTS (長期維護版)** 的 `.msi` 安裝檔，依預設步驟安裝到底，安裝完成後重啟終端機。

#### 2. 全域安裝 clasp
開啟 PowerShell 或 CMD 執行：
```powershell
npm install -g @google/clasp
```

#### 3. 登入 Google 帳號
```powershell
clasp login
```
- 瀏覽器會自動彈出 Google 登入驗證頁面，登入欲開發 GAS 的 Google 帳號並允許存取權限。
- 驗證成功後，憑證將自動儲存於 `~/.clasprc.json`。

---

### 二、macOS 系統安裝指引

#### 1. 安裝 Node.js & npm
- **推薦方式 (使用 Homebrew)**：
  ```bash
  brew install node
  ```
- **或使用 nvm (Node 版本管理器)**：
  ```bash
  curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
  nvm install --lts
  ```

#### 2. 安裝 clasp
```bash
npm install -g @google/clasp
```
> 若遇到 `EACCES` 權限錯誤，建議不要使用 `sudo npm`，可設定 npm 全域目錄：
> `mkdir ~/.npm-global && npm config set prefix '~/.npm-global'`
> 並將 `export PATH=~/.npm-global/bin:$PATH` 加入 `~/.zshrc`。

#### 3. 登入 Google 帳號
```bash
clasp login
```

---

### 三、iOS / iPadOS 行動裝置環境指引

> 由於 iOS/iPadOS 系統沙盒限制，無法直接在本機原生常駐 Node.js CLI。建議採用以下方案：

#### 方案 A：使用 GitHub Codespaces / 雲端容器（⭐ 最佳推薦）
1. 在 iPad 瀏覽器開啟 [GitHub Codespaces](https://github.com/codespaces) 或 [VS Code for Web](https://vscode.dev)。
2. 啟動一個 Linux 雲端環境。
3. 雲端終端機內已內建 Node.js，直接執行：
   ```bash
   npm install -g @google/clasp
   clasp login --no-localhost
   ```
   > 依終端機提示將 URL 貼至瀏覽器授權，並將取得的授權碼貼回終端機完成登入。

#### 方案 B：使用 iPad 終端機 App（如 a-Shell 或 iSH）
1. 至 App Store 下載 **a-Shell**（原生支援 Node/npm）或 **iSH Shell**（Alpine Linux 模擬器）。
2. 在 **iSH Shell** 中執行：
   ```bash
   apk update && apk add nodejs npm
   npm install -g @google/clasp
   clasp login --no-localhost
   ```

---

## 🚀 新專案初始化標準流程 (GAS Project Setup)

當環境安裝就緒，初始化一個 GAS 本地專案的標準步驟：

### 1. 建立專案資料夾並進入
```bash
mkdir my-gas-project
cd my-gas-project
```

### 2. 建立新專案或複製現有專案
- **新建雲端專案 (二選一)**：
  ```bash
  clasp create --title "我的 GAS 專案" --type standalone
  # 或綁定至現有試算表: --type sheets
  ```
- **拉取現有雲端專案**：
  ```bash
  clasp clone <YOUR_SCRIPT_ID>
  ```

### 3. 配置本地開發環境 (TypeScript 型別支援)
```bash
npm init -y
npm install --save-dev @types/google-apps-script typescript
```

### 4. 建立 `.claspignore`（必備，避免上傳無關檔案）
建立 `.claspignore` 並寫入：
```gitignore
**/**
!src/**
!appsscript.json
!**/*.js
!**/*.ts
!**/*.html
.git/**
node_modules/**
package.json
package-lock.json
tsconfig.json
README.md
```

### 5. 日常開發同步指令
- **推送本地程式碼至雲端**：
  ```bash
  clasp push
  ```
- **拉取雲端程式碼至本地**：
  ```bash
  clasp pull
  ```
- **直接用瀏覽器開啟雲端 Apps Script 編輯器**：
  ```bash
  clasp open
  ```
- **啟動 clasp MCP 伺服器 (若支援)**：
  ```bash
  clasp mcp
  ```

---

## 🔒 安全規則
- 嚴禁將 `~/.clasprc.json`（包含 Google OAuth Token）與 `.clasp.json`（若包含機密 ID）commit 到公開儲存庫。
- 專案務必配置 `.gitignore`，包含 `.clasprc.json`、`node_modules/` 與敏感憑證檔案。
- 變更送出前先檢查 `git diff`，不無差別 `git add .`。
