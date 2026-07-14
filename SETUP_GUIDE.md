# 週度宏觀分析 Routine 部署指南

## 架構總覽

```
GitHub repo: macro-routine
├── SPEC.md              ← v5.1 完整規格(分析引擎)
├── ROUTINE_PROMPT.md    ← Routine 執行指令(貼進 Instructions 欄位)
├── CLAUDE.md            ← 極簡上下文(每次執行都計費,刻意留薄)
├── SETUP_GUIDE.md       ← 本文件
└── reports/             ← 每週報告落地處,同時是 routine 的「記憶」
    └── YYYY-MM-DD_macro.md
```

核心設計:routine 每次執行是無狀態 fresh session,但 repo 提供持久性。
每份報告 commit 進 `reports/`,下次執行先讀上週報告,產出 WEEKLY DELTA
(週對週快照變化、regime 是否切換、上週五個資產觀點的記分、情景樹複盤)。
報告歷史同時就是你的判斷績效紀錄——execution compliance 可以直接回測。

## 部署步驟

### 1. 建立 repo
```bash
gh repo create macro-routine --private
cd macro-routine
# 放入本資料夾的四個檔案 + 空的 reports/ 目錄(放一個 .gitkeep)
git add -A && git commit -m "init macro routine" && git push
```

### 2. 建立 Routine
前往 **claude.ai/code/routines**(或 Desktop app → New Task → New Remote Task,
或 CLI 內 `/schedule`——注意 CLI 只能建排程觸發,API trigger 要事後去網頁補):

| 欄位 | 設定 |
|---|---|
| Name | `weekly-macro-report` |
| Repository | `macro-routine` |
| Instructions | 貼上 `ROUTINE_PROMPT.md` 全文 |
| Trigger | Schedule → Weekly |
| 時間 | 週日 10:00(Asia/Taipei)——見下方時區說明 |
| Connectors | 全部移除不需要的(預設會掛上你所有 connector;本 routine 只需 web search + repo,Notion/Gmail 除非要做投遞否則移除,減少攻擊面與 token 消耗) |

**排程時間的邏輯**:台北週日上午 = 美東週六。此時週五收盤的 FRED 日頻序列
(H.15 殖利率、breakevens)與有一天延遲的 HY OAS 全部到位,報告在週一亞洲
開盤前就緒。若改排週六上午,會拿到週四的 HY OAS——可接受但次佳。

### 3. 首次執行驗證(必做)
按 **Run now**,檢查五件事:
1. Web search / fetch 在雲端環境確實可用(Routines 是 research preview,首次必驗)
2. 報告有 commit 進 `reports/` 且格式完整(17 個 section + DATA SNAPSHOT)
3. DATA GAPS 段落誠實——沒有逐行「數據尚未搜尋到」,沒有捏造值
4. 快照表每個值都帶日期
5. 首次執行會標「首次執行,無前週基準」——第二次執行才會出現 WEEKLY DELTA

### 4.(可選)加 API trigger,接進 FICC PRO v4
Routine 建好後在網頁編輯頁開啟 API trigger,會拿到專屬 endpoint + auth token。
FICC PRO 加一顆「刷新宏觀報告」按鈕:

```javascript
// 注意:token 不要放前端。走你現有的 Cloudflare Worker 代理,
// 跟 Gemini CORS proxy 同一個模式。
await fetch('https://your-worker.workers.dev/trigger-macro', { method: 'POST' });
// Worker 內部才持有 routine token,轉發到 routine endpoint,
// 回傳的 session URL 可以直接開新分頁看執行過程。
```

用途:FOMC 之夜、CPI 超預期、荷姆茲式突發——盤中隨時手動觸發一份新鮮報告,
不用等週日。

## 用量與成本

- 週報 = 每週 1 次執行,遠低於每日上限(Pro 5 次/日、Max 15 次/日)
- Routine 執行消耗訂閱用量,與互動 session 同一池——單次完整分析約 12–14 次
  搜尋加長輸出,是重型 session,Pro 方案下留意當日剩餘量
- API trigger 手動觸發也計入每日 routine 次數

## 已知限制(誠實清單)

1. **Research preview**——行為、限制、API 介面都可能變;官方文件:
   claude.com/blog/introducing-routines-in-claude-code
2. **無人值守 = prompt 承擔全部認知負載**——這正是 SPEC.md 的驗證協議與
   缺數據紀律存在的理由,不要刪減它們來省 token
3. **雲端環境拿不到你的本機檔案**——一切狀態走 repo
4. **失敗模式已定義**:數據覆蓋率 <70% 時仍會 commit 一份誠實的降級報告
   (conviction 全部封頂 2、無 asymmetric setup),而不是靜默失敗或捏造
5. 前兩週建議每份報告人工複核快照表的日期欄——這是「亂找數據」的檢測哨,
   校準完成後再放手

## 維護

- 改分析邏輯 → 只改 `SPEC.md`,commit 後下次執行自動生效(routine 每次
  clone 最新 repo)
- 改執行流程(輸出路徑、delta 邏輯)→ 改 routine 的 Instructions 欄位
- 報告累積一季後,可以把 `reports/` 餵回互動 session 做判斷績效複盤:
  regime call 命中率、invalidation 觸發率、情景樹校準度——這本身就是
  你 DYL 框架裡 execution compliance rate 的宏觀版本
