# Routine Instructions (paste this into the routine's Instructions field)

You are executing the weekly FICC macro analysis routine. Follow these steps in order. You run unattended — never wait for human input, never ask questions, always finish with a committed report.

## Step 1 — Load the specification
Read `SPEC.md` in the repository root. It is the complete and authoritative specification: role, reasoning principles, data sourcing strategy (tiered sources, FRED series IDs, proxy map, search budget), verification protocol, analysis framework, and the 17-section output specification. Follow it exactly.

## Step 2 — Load last week's context
List the files in `reports/` and read the MOST RECENT report (filenames are `YYYY-MM-DD_macro.md`, sort by date). Extract from it: last week's regime call and conviction, the five asset views with their invalidation levels, the scenario tree probabilities, and the key snapshot values (10Y, DXY, Gold, WTI, SPX, VIX, HY OAS, hike odds). If `reports/` is empty, note "首次執行,無前週基準" and skip delta logic.

## Step 3 — Execute the analysis
Run the full data gathering (search budget 10–14 calls per SPEC.md), verification checks, and produce the complete output in Traditional Chinese per the SPEC.md output specification.

## Step 4 — Append the WEEKLY DELTA section
After the final SPEC section, append one additional section:

=== WEEKLY DELTA(週對週變化)===
- 快照變化表:本週 vs 上週的 10Y、DXY、Gold、WTI、SPX、VIX、HY OAS、升息機率,各附變化量與單位
- Regime 判定:延續或切換?若切換,說明哪個 lens 的證據翻轉了
- 資產觀點記分:上週五個觀點各自的狀態 — 朝目標推進 / 觸發 invalidation / 區間內。觸發 invalidation 的必須明確承認,不准靜默替換
- 情景樹複盤:上週機率最高的情景是否兌現?一句話
- 對稱性檢查變化:哪些關係從 HOLDING 轉 BREAKING 或反向

## Step 5 — Commit the report
Write the complete output (Steps 3 + 4) to `reports/YYYY-MM-DD_macro.md` using today's date (Asia/Taipei). Commit with message `weekly macro report YYYY-MM-DD` and push to the default branch.

## Failure handling
- If web search/fetch is unavailable or Tier 1 data coverage falls below 70%: still write and commit a report, with the DATA GAPS section stating what failed, all convictions capped at 2, and no asymmetric setup. A short honest report beats no report and beats a fabricated one.
- Never fabricate a data point. Never invent calendar dates. Per SPEC.md, unverified values appear once in the snapshot as UNVERIFIED and nowhere else.
- If the previous report cannot be parsed, proceed without the delta section and note why.

## Hard boundaries
- Write ONLY inside `reports/`. Never modify SPEC.md, CLAUDE.md, or this file.
- One commit per run. No force-push. No branch creation.
- Output language: Traditional Chinese; tickers and FRED IDs stay in English.
