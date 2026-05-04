---
description: Context Window 監控 + 1M context GA + Prompt Caching（進階 session 管理見 session-management.md）
tier: auto
---

# Context Window 管理

## 理論錨點

**Context Engineering** (Karpathy 2025-06-25) = 「filling the context window with just the right information for the next step」。雙刃劍：太少/錯誤格式 → 效能不足；太多/不相關 → 成本上升且品質下降。歸檔：`research/tweets/2025-06-25-@karpathy-607626.md`。

## 監控

- `/usage` 查看 session token/cost 即時用量。
- **Compact 三層觸發**（優先序：行為信號 > 數字閾值 > 定時器）：
  1. **行為信號**：模型出現「請提供更多上下文」「你想做什麼？」等迷失問句 → 立即 `/rewind` 或 `/compact`
  2. **數字閾值**：一般任務 **70%**；初學者 ~**60%**；長 agentic **30–35%** 主動 compact
  3. **定時器**：複雜 agentic 每 **300–400K token** 主動 compact（200K context 用戶的 30–35% ≈ 60–70K）
- `/compact <hint>` 繼續任務；`/clear` 切換任務。
- **核心原理**：compact 重新聚焦注意力（「Lost in the Middle」U 型曲線），**不是省錢**。走偏後修正成本 ≈ 重開 2–3 倍。

## 壓縮層級決策表

根據 context % 調整行動：0–40% 無限制、40–70% 聚焦、70–85% 主動 compact、85–95% 停止新任務、95%+ 立即 clear。詳細表格與操作說明見 `.claude/refs/context-monitor-table.md`。

## Auto Memory（跨 session 記憶）

- `.claude/settings.json` 設 `"autoMemoryEnabled": true`
- Claude 自動累積於 `~/.claude/projects/<project>/memory/`
- `/memory` 查看或編輯；`/clear` 與 `/compact` 不影響 Auto Memory

> 1M context 已 GA：**Max / Team / Enterprise** 訂閱含 Opus 1M（Sonnet 1M 需額外用量費）；**Pro** 方案兩者均需額外付費；**API / pay-as-you-go** 標準費率（200K+ 無溢價）。compaction 壓力低。

> **Context rot 閾值**（社群觀察，未經官方確認）：約 **300–400k tokens** 開始影響輸出品質，高度依賴任務類型（來源：Thariq @trq212, 2026-04-16；Wisely Chen AI, 2026-04-24）。複雜 agentic 任務以 **30–35% 主動 compact** 為操作基準；一般任務仍以 70% 提醒為準。

> **進階 compact / rewind 策略**：按需載入 `.claude/refs/session-management.md`。

## Prompt Caching 架構規則

Prompt caching 架構規則詳見 `.claude/refs/prompt-caching-rules.md`（按需載入；觸發詞：`cache`、`prompt caching`）。核心原則：Static first, dynamic last；工具列表和模型 mid-session 不可改變。

## Known Gotchas

- **Compact 不總是解決問題**：若摘要時模型已走偏，摘要品質也差。出現「模型迷失」信號時立即 rewind，不要等 context % 才 compact。
- **Context rot 與 token 消耗無直接關聯**：> 3,500 token 不代表一定 rot；focus 在 context engineering（填充「恰好所需的資訊」），而非壓行數。
- **`@-import` 無法省 token**：@ 指令是載入期間展開，非 lazy load；只把確實需要的規則放 auto-loaded。
- **70% 警告易被無視**：把 70% 當「開始蒐集摘要素材」的信號，而非可忽視的黃燈。
