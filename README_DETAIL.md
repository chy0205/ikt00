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
2. **單位分組 (Dynamic Dropdown)**:
   - 核心函數 `initDropdowns()`：過濾出不重複的單位名稱並排序。
   - **自動跳過**: 若全班僅有一個單位，選單將自動隱藏並展示名冊。

### B. QR Code 簽到邏輯 (Enhanced Scanner)
1. **沈浸式全螢幕介面**:
   - 透過 `fixed inset-0` 實現相機影像佔滿全螢幕。
   - 採用 `object-cover` 確保預覽影片比例正確且填滿視區。
2. **智慧預設變焦 (Default 2x Zoom)**:
   - 在 `initScanner` 階段調用 `getVideoTracks().getCapabilities()`。
   - 若硬體支援 `zoom`，程式會自動將 `zoom` 設定為 **2.0**，極大提升遠距離掃描的便利性。
3. **懸浮訊息系統 (Floating Overlay)**:
   - 簽到結果、載入狀態與錯誤提示皆透過 `absolute` 定位的懸浮卡片呈現。
   - 具備自動定時隱藏機制，確保掃描過程連貫。
4. **安全驗證機制**:
   - **二級防護 (定位)**: 計算使用者座標與設定地點之間的 Haversine 距離。
   - **管理員模式**: 輸入地點特定密碼可解除定位限制。

---

## 3. UI/UX 設計規範 (Design Tokens)

- **視覺風格**: 現代卡片式介面 + 沈浸式視訊背景。
- **色彩系統**:
  - 主色: `blue-600` (系統導航 / 定位驗證)
  - 成功: `green-600` (出席/簽到成功)
  - 警告: `amber-500` (管理員模式)
  - 錯誤: `red-600` (驗證失敗/超距)
- **響應式**: 適配行動裝置，QR 掃描頁面採用全螢幕佈局，最大化識別區域。

---

## 4. 開發者紀錄 (Changelog)
- **v2.2 (最新)**: 實作沈浸式全螢幕掃描、自動 2x 變焦以及懸浮訊息視窗。
- **v2.0**: 替換 ZXing 掃描核心，實現自動跳過單一選項邏輯。
- **v1.5**: 整合手動表單與 QR 簽到至單一 Tab 頁面。
