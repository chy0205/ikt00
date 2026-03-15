# 崇正班期點名整合系統 - 詳細技術文件 (README_DETAIL)

本文件提供「崇正班期點名整合系統」的深度技術分析，包含邏輯流程、UI 設計細節以及後端通訊規範。

## 1. 系統概覽 (System Overview)
本系統是一個專為班期點名設計的輕量化 Web App，採用「無伺服器架構 (Serverless)」，核心邏輯完全在瀏覽器運行，並利用 Google Apps Script 作為後端資料庫介面。

- **Host**: 部署於 GitHub Pages / GAS 網頁發布。
- **Frontend**: HTML5, CSS3 (Tailwind), ES6 JavaScript.
- **Backend API**: Google Apps Script (GAS) 處理 Google Sheets 讀寫。

---

## 2. 核心功能模組 (Technical Features)

### A. 手動點名邏輯 (Manual Attendance Flow)
1. **初始化 (Initialization)**:
   - 啟動時發送 `action=init` 請求獲取班級清單。
   - **自動優化**: 若班級數量為 1，系統會通過 `selectClass()` 自動跳過首頁按鈕選擇。
2. **數據加載**:
   - `loadFormData()` 依據所選班級抓取名冊。
   - 名冊包含：`memberId`, `name`, `gender`, `seatNum`, `area` (單位)。
3. **單位分組 (Dynamic Dropdown)**:
   - 核心函數 `initDropdowns()`：過濾出不重複的單位名稱並排序。
   - **自動跳過**: 若全班僅有一個單位，選單將自動隱藏並展示名冊。
4. **狀態提交**:
   - 使用 `submitFormData()` 封裝 POST 請求。
   - 支援「出席」與「請假」（包含 Modal 彈窗收集額外理由）。

### B. QR Code 簽到邏輯 (High-Performance QR Scanner)
1. **解碼引擎 (ZXing Library)**:
   - 引入 `@zxing/library` 取代 html5-qrcode。
   - 配置 `hints.set(ZXing.DecodeHintType.TRY_HARDER, true)` 以應對低畫質、遠距離或複雜編碼。
2. **高品質影像請求**:
   - 請求 `1080p` 理想解析度。
   - 鎖定 `facingMode: "environment"` (後置相機)。
3. **安全驗證機制**:
   - **一級防護 (密碼)**: 進入 QR 頁面前需輸入 Password。
   - **二級防護 (定位)**: 計算使用者座標與設定地點 (`currentLoc.lat/lng`) 之間的 Haversine 距離。若超過 `limit` 則拒絕簽到。
   - **管理員模式**: 輸入地點特定密碼可解除定位限制，方便室內或訊號不佳時手動簽到。
4. **掃描緩衝與查重**:
   - 設定 3 秒內禁止重複掃描相同 ID，防止因畫面晃動導致多次提交。

---

## 3. UI/UX 設計規範 (Design Tokens)

- **視覺風格**: 現代卡片式介面 (Card-based UI)，使用玻璃擬態 (Glassmorphism) 陰影效果。
- **色彩系統**:
  - 主色: `blue-600` (系統導航)
  - 成功: `green-600` (出席/簽到成功)
  - 警告: `amber-500` (管理員/請假)
  - 錯誤: `red-600` (驗證失敗/超距)
- **響應式**: 完全適配行動裝置，採用 `max-w-md` 限制桌面端寬度，確保在手機瀏覽器上呈現最舒適的點擊範圍。

---

## 4. API 介面定義 (API Reference)

### GET 請求
- `?action=init`: 獲取 `locations` 與 `classes` 基礎配置。
- `?action=getMemberList&className={name}`: 獲取特定班級的名冊 CSV。

### POST 請求 (JSON Body)
- `action: "signin"`: QR Code 自動簽到。
- `action: "updateStatus"`: 手動更新出席狀態。

---

## 5. 開發者紀錄 (Changelog)
- **v2.0 (最新)**: 更換 ZXing 掃描核心，實現自動跳過單一選項邏輯，增加管理員免定位模式。
- **v1.5**: 整合手動表單與 QR 簽到至單一 Tab 頁面。
- **v1.0**: 初始版本，基礎數據對接。
