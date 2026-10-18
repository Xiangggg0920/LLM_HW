# AimPulse - FPS 靈敏度換算與反應靶場 系統規格書 (v1.7)

## 1. 專案基本資訊
* **專案名稱**：AimPulse (FPS Sensitivity Converter & Mini Aim Trainer)
* **專案類型**：純前端互動式 Web 應用程式 (Single Page Application)
* **實作架構**：單一檔案架構 (`index.html` 內嵌 HTML、CSS、JavaScript)
* **運行環境**：主流現代瀏覽器 (Chrome, Edge, Firefox, Safari)
* **部屬方式**：GitHub Pages 靜態託管

---

## 2. 專案背景與開發目標
射擊遊戲 (FPS) 玩家常面臨跨遊戲遊玩時手感不一致的問題，需透過靈敏度與硬體 DPI 的精準轉換維持肌肉記憶。同時，玩家在調整設定後往往需要立即驗證手感與反應狀況。

本專案將「跨遊戲靈敏度精算」與「沉浸式 2D 點擊反應靶場」整合於單一畫面中。針對視覺可見度與練習手感，全面優化標靶尺寸與全螢幕專注機制，並加入「開局 3 秒預備倒數 (3-2-1 Countdown)」機制、實時平均反應時間 (ms) 數據追蹤、完整的結算面板流程與一鍵分享功能。

---

## 3. 系統功能規格

### 3.1 靈敏度換算模組 (Sensitivity Converter)
提供主流第一人稱射擊遊戲的靈敏度雙向換算與基礎物理移動距離估算。

1. **支援遊戲清單與視覺識別**：
   * Counter-Strike 2 (CS2)
   * Valorant
   * Apex Legends
   * Overwatch 2
2. **輸入參數**：
   * 來源遊戲 (Source Game)
   * 硬體滑鼠靈敏度 (Mouse DPI，如 400, 800, 1600 等)
   * 來源遊戲內靈敏度 (In-game Sensitivity)
   * 目標遊戲 (Target Game)
3. **計算演算法與公式**：
   * 基準 Yaw 值定義：
     * CS2 / Apex Legends：$Yaw = 0.022^\circ$
     * Valorant：$Yaw = 0.07^\circ$
     * Overwatch 2：$Yaw = 0.0066^\circ$
   * 換算邏輯：
     * 換算係數（例如 Valorant 轉 CS2）：$$\text{Sens}_{\text{CS2}} = \text{Sens}_{\text{Val}} \times \left(\frac{0.07}{0.022}\right) \approx \text{Sens}_{\text{Val}} \times 3.1818$$
     * 有效 DPI ($eDPI$)：$$eDPI = DPI \times \text{Sens}$$
     * 360 度轉身物理距離 ($cm/360^\circ$)：$$cm/360^\circ = \frac{360 \times 2.54}{DPI \times \text{Sens} \times Yaw} = \frac{914.4}{DPI \times \text{Sens} \times Yaw}$$
4. **輸出結果**：
   * 目標遊戲推薦靈敏度（四捨五入至小數點後 3 位）。
   * 當前設定的 $eDPI$ 與 $cm/360^\circ$ 物理指標。

---

### 3.2 互動式 Canvas 點擊靶場模組 (Mini Aim Trainer)
採用原生 HTML5 Canvas 繪製，提供流暢的點擊目標、即時反應回饋與放大沉浸體驗。

1. **開局 3 秒預備倒數機制 (3-2-1 Ready Countdown)**：
   * **啟動觸發**：玩家點擊「立即開始」或「全螢幕放大專注開始」或「再練一局」時，先進入 3 秒預備狀態。
   * **視覺呈現**：畫布中央顯示全螢幕半透明遮罩，以發光霓虹大字依序倒數「3」、「2」、「1」與「GO!」，每秒伴隨彈跳放大動畫。
   * **音效提示**：
     * 倒數 3、2、1 時：發出 440Hz 預備短嗶聲（Web Audio Sine Wave）。
     * 倒數結束 GO! 時：發出 880Hz 高亢發令音，同步隱藏遮罩。
   * **狀態保護**：在 3 秒倒數期間畫布不生成標靶，且滑鼠點擊不計入命中與脫靶，給予玩家握好滑鼠、對齊視野的準備時間。
2. **視窗放大與全螢幕專注機制 (Focus Mode)**：
   * **雙軌顯示架構**：原生 Fullscreen API 與劇院覆蓋備援模式 (`theater-mode`)。
   * **標靶尺寸規範**：
     * 常規視窗模式：標靶基礎半徑加大至 $r = 33 \sim 45\text{ px}$（直徑約 $66 \sim 90\text{ px}$）。
     * 全螢幕/劇院放大模式：再放大 2.0 倍，標靶半徑擴展至 $r = 66 \sim 90\text{ px}$（直徑約 $132 \sim 180\text{ px}$）。
   * **遊戲結束自動退出全螢幕**：30 秒結束時自動退出全螢幕，無縫切換到結算彈窗。
3. **即時數據追蹤（頂部 HUD 儀表板）**：
   * **剩餘時間 (Time Left)**：30 秒高精確度倒數（小數點後 1 位）。
   * **命中 / 脫靶數 (Hits / Misses)**：命中與失誤次數對比。
   * **準確率 (Accuracy %)**：$$\text{Accuracy} = \frac{\text{Hits}}{\text{Hits} + \text{Misses}} \times 100\%$$
   * **即時平均反應時間 (Avg RT)**：$$\overline{RT} = \frac{1}{N}\sum_{i=1}^N \Delta t_i \quad (\text{ms})$$（未命中前顯示 `--ms`）。

---

### 3.3 成績結算與分享模組 (Result Modal)
1. **結算彈窗操作**：
   * **一鍵複製成績**：格式化字串複製到剪貼簿。
   * **再練一局 (Play Again)**：關閉彈窗並再次觸發 3 秒預備倒數後開始。
   * **結束測試 · 返回主畫面 (Finish & Exit)**：點擊右上角關閉鈕或下方結束按鈕，關閉結算彈窗，重置數據顯示（反應時間重設為 `--ms`），並退回初始引導狀態。
2. **段位評級機制**：
   * $Hits \ge 45$ 且 $Accuracy \ge 90\%$：**S 級 (Radiant / The Global Elite)**
   * $Hits \ge 35$ 且 $Accuracy \ge 80\%$：**A 級 (Diamond / Master)**
   * $Hits \ge 25$：**B 級 (Gold / Platinum)**
   * $Hits < 25$：**C 級 (Bronze / Silver)**

---

## 4. 技術實現規範
1. **單一檔案要求**：全部 HTML、CSS、JavaScript 封裝於 `index.html`。
2. **免外部多媒體依賴**：向量 SVG 與 Web Audio API 即時頻率振盪器，零外部音檔或圖片請求。
3. **高精確度計時生命週期**：倒數 3 秒使用間隔定時器鎖定操作，結束後無縫銜接主迴圈。