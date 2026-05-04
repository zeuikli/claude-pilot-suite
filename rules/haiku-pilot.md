# Haiku Pilot — 預設 Haiku，量化升級

> 來源：AgentOpt (arxiv 2604.06296, 2026-04) + Augment Code AGENTS.md eval (2026-04-22)。
> 詳細 playbook：`.claude/skills/haiku-pilot/SKILL.md`（觸發詞：「Haiku 模式」「全力 Haiku」「升級判斷」「Haiku Pilot」）。

## 原則（Thesis）

**弱 planner + 善委派 > 強 planner 全包。** AgentOpt 實證：Opus 全包 HotpotQA 31.71%；Ministral planner + Opus solver 74.27%。
→ Parent session 預設 Haiku；積極委派 sub-agent；**僅在量化閘門觸發時升級**。

## 模式切換

- **Haiku 模式**（本規則）：預設，成本優先。觸發詞：`Haiku 模式`、`節費執行`、`/haiku-pilot`
- **Sonnet 模式**：品質優先，覆蓋本預設。觸發詞：`Sonnet 模式`、`品質優先`、`/sonnet-pilot`
- 兩模式並存，最後明確啟動者生效。Sonnet playbook：`.claude/skills/sonnet-pilot/SKILL.md`

## 升級閘門（量化，不憑感覺）

| 觸發條件 | 升級到 |
|---------|-------|
| 同一問題 ≥ 3 次失敗 | Sonnet 4.6 |
| 任務涉及 ≥ 10 個獨立檔案 OR 跨模組設計 | Sonnet 4.6 |
| 觸發詞：「架構決策」「design review」「資安審查」「威脅建模」 | Opus 4.7 / `advisor()` |
| Code-gen 一次回 >300 行未經要求 | 停手，重新派 sonnet `implementer` |
| 用戶明示「用 Opus」「用 Sonnet」 | 照辦 |

**閘門外** → 留在 Haiku，依 `subagent-strategy.md` 委派分工。

## 每任務 Pre-flight（≤ 4 檢查）

1. **Reference Pattern**：能否指向既有檔案而非口述需求？（`src/auth/login.ts` > 「類似 login」）
2. **Think-Before-Coding**：實作前 1-2 句複述理解 + 列關鍵假設（compensates Haiku 跳過此步驟的傾向）
3. **Diff-Review pledge**：宣告完成前必跑 `git diff --stat` 並逐項說明（捕捉順手改動）
4. **Citation Anchor**（涉及 wiki/ref 引用任務必檢）：每個答題段落 ≥ 3 個結構化錨點（`Step N` / `Anti-pattern #N` / `CLD.X.Y` / `Concrete Numbers` / `Decision Tree` / `Gotcha [Source]`）；模糊措辭如「wiki 案例」「wiki 提到」**不計**為錨點。來源：2026-05-04 benchmark 對比 Haiku 8 vs Sonnet 45 anchor lines 的 5.6× 落差信號。

## Self-check 模板（涉及 wiki 引用任務必填，**單一表格不擴張**）

| 檢查項 | Q1 | Q2 | Q3 | Q4 | Q5 | Pass |
|--------|----|----|----|----|----|------|
| 字數（`wc -w`，≤ 800/題）| ___ | ___ | ___ | ___ | ___ | ✓/✗ |
| Anchor count（每題 ≥ 3）| ___ | ___ | ___ | ___ | ___ | ✓/✗ |
| 升級閘門（hit/no-hit + 一句理由）| ___ | ___ | ___ | ___ | ___ | — |

**軟閾值**：總 anchor 數 ≥ 15；< 12 → **觸發 retry**（重跑題補編號引用）。
**禁止**：擴張為多個 H3 區塊、追加散文說明、重寫題目脈絡。**只填上表**。
來源：2026-05-04 v3 Haiku Token 效率退步（self-check 自行擴張為 5 H3 區塊，字數 2.24× 膨脹）。

## P+S（Prohibition + Solution）

| Don't | Do |
|-------|-----|
| 直覺「複雜任務先升 Opus」 | 先用 Haiku + 上面 pre-flight；命中閘門才升 |
| Haiku 失敗 1 次就換模型 | 看是 context 問題（`/compact`）還是模型不足（達 3 次才升） |
| 沿用「Haiku 一定差」直覺 | SWE-bench Verified Haiku 4.5 = 73.3%，距 Sonnet 5pp，成本 1/3 |

## Known Gotchas（Haiku 4.5 specific）

- **跳過 assumption-disclosure**：Haiku 傾向直接寫 code；rule 已強制 §`core.md`「實作前假設顯露」。
- **註解冗長**：Haiku 比 Sonnet 多寫 1.5–2× 註解；遵循 `output-discipline.md` 預設無註解。
- **漏 regression check**：debug 時不會自動問「最近 N commits 哪些改動可能導致？」；觸發詞「regression」「最近改動」明確要求。
- **引用語意化偏移**：Haiku 在 wiki 引用任務傾向「wiki 提到」「wiki 案例清晰」等模糊措辭，非結構化錨點；2026-05-04 benchmark 量測引用密度 0.75/100w（vs Sonnet 3.43）。Pre-flight #4 強制錨點化。

> 詳細 per-task router、Haiku vs Sonnet 對比、實作 checklist：載入 `haiku-pilot` SKILL。
