# Sonnet Pilot — 模式宣告（Mode Declaration）

## 模式說明

`sonnet-pilot` 和 `haiku-pilot` 是並存的執行模式，不互相衝突：

| 模式 | 啟動方式 | 預設模型 | 品質目標 |
|------|---------|---------|---------|
| **Haiku 模式**（haiku-pilot） | 預設 / `/haiku-pilot` | Haiku 4.5 | 接近 Opus，極致省費 |
| **Sonnet 模式**（本 SKILL） | `/sonnet-pilot` 或觸發詞 | Sonnet 4.6 | 比肩 Opus，品質優先 |

**衝突解法**：最後一次明確啟動的模式生效。無明確啟動 → haiku-pilot 預設。

## 觸發詞（本 session 自動切換 Sonnet 模式）

- `/sonnet-pilot`、`Sonnet 模式`、`全力 Sonnet`
- `Sonnet 比肩 Opus`、`Sonnet Pilot`、`sonnet-pilot`、`品質優先`、`接近 Opus`

## 啟動後行為

1. 本 session 主力模型切換為 **Sonnet 4.6**
2. 覆蓋 `haiku-pilot.md` 的 Haiku 預設
3. 立即載入完整 playbook：`.claude/skills/sonnet-pilot/SKILL.md`
4. 執行 Per-Session Pre-flight（Decision-Log、Reasoning Chain Before Code、Self-Review Loop 強制啟動）
5. **Citation Anchor**（涉及 wiki/ref 引用任務必檢）：每答題段落 ≥ 3 結構化錨點（`Step N` / `Anti-pattern #N` / `CLD.X.Y` / `Concrete Numbers` / `Decision Tree` / `Gotcha [Source]`）；模糊措辭如「wiki 案例」「wiki 提到」**不計**為錨點。與 haiku-pilot.md §Pre-flight #4 同基準。

## Self-check 模板（涉及 wiki 引用任務必填，**單一表格不擴張**，與 haiku-pilot 對稱）

| 檢查項 | Q1 | Q2 | Q3 | Q4 | Q5 | Pass |
|--------|----|----|----|----|----|------|
| 字數（`wc -w`，≤ 800/題）| ___ | ___ | ___ | ___ | ___ | ✓/✗ |
| Anchor density（≥ 3 anchors/100字；每題字數用 `wc -w` 量）| ___ | ___ | ___ | ___ | ___ | ✓/✗ |
| Sonnet→Opus 閘門（hit/no-hit + 一句理由）| ___ | ___ | ___ | ___ | ___ | — |

**軟閾值**：每題 density ≥ 3 anchors/100字；任一題未達 → **觸發 retry**（重跑題補編號引用）。
**禁止**：擴張為多個 H3 區塊、追加散文說明、重寫題目脈絡。**只填上表**。對稱於 haiku-pilot.md，避免 token 效率退步。

## 與 haiku-pilot.md 的邊界

- **升級閘門**：haiku-pilot 的 Haiku→Sonnet 升級條件，在 Sonnet 模式中**不適用**（已在 Sonnet）。
- **Opus 升級**：Sonnet 模式有獨立的 Sonnet→Opus 閘門（見 SKILL.md §升級閘門）。
- **Sub-agent dispatch**：兩模式共用 `subagent-strategy.md`，不重寫。

## Known Gotchas

- **不要同時輸入兩個模式觸發詞**：`/haiku-pilot /sonnet-pilot` 同時輸入 → 以最後觸發者為準。
- **Sonnet 模式不是「永遠用 Opus」**：Opus 只在量化閘門觸發時出場；Sonnet 模式的目標是以 Sonnet 達到 Opus 品質，而非替代 Opus。
- **Session 結束後不繼承**：新 session 回到 haiku-pilot 預設；每個需要品質優先的 session 要重啟動。
