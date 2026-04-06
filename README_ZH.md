# rico-memory-ops

讓 AI 不只是記得更多，而是能在時間拉長後，仍然像同一個它。

英文版請見：[README.md](README.md)

## 這是什麼

大多數 AI 記憶系統，是幫 agent 記得更多事情。

`rico-memory-ops` 想處理的是一件更難的事：

`幫助一個 agent 在跨時間、跨 session、跨交接之後，仍然保持相對一致。`

這不是單純的筆記儲存，也不是一般的 semantic recall。

這個 repo 更在意的是 continuity 背後真正的 operating layer：

- 讓判斷在跨 session 後仍然保持一致
- 喚醒對的上下文，而不是每次把整包內容重載
- 在房間、agent 或 session 之間交接工作時，不丟主線
- 把熱工作狀態和可長期保存的記憶分開

它適合給想做這幾種東西的人：

- persistent agents
- continuity systems
- 長期運行的人機協作 workflow

## 為什麼這個專案存在

很多 agent 系統可以存下大量筆記，
卻還是會在更難的問題上失敗：

`它們記得事實，卻沒有辦法維持一致性。`

實際上，這通常會表現在：

- 每個 session 都像重新開始判斷
- 一交接就斷線
- 喚醒太多上下文，或喚醒錯的上下文
- 把熱狀態、已封存記憶、身份層級的工作方式混在一起

`rico-memory-ops` 想解的，就是這些 operating problems。

## 什麼情況下會有用

這類工具會在下面這些情況變得很有價值：

- 一個 agent 要把工作交給另一個 agent，但判斷風格不該整個斷掉
- 一個長期陪跑的 assistant，應該像同一個腦袋在接著做事，而不是每次都像新來的
- 系統需要為眼前任務叫醒對的上下文，而不是把整個 archive 全塞回 prompt
- 團隊希望 continuity 來自可重複的方法，而不是臨場拼 prompt

## 這個 repo 跟一般 memory system 有什麼不同

這個 repo 不是想做一個巨大的記憶資料庫，
也不是想做一個人格包裝器。

它真正想讓三件事更可靠：

1. `跨 session 的 continuity`
2. `跨時間的一致判斷`
3. `能保留身份層級工作風格的記憶，而不只是保留事實`

它的核心想法很簡單：

> 光有記憶還不夠。  
> 一個 agent 還需要 continuity。

## 目前包含什麼

- `RICO`
  - `Roadmap + Intent + Cue + Origin`
- 用於 recall 與 handoff 的輕量記憶語義
- continuity 與 handoff 模板
- synthetic examples
- 適合 method-first open source 的發佈指引

## 目前不在範圍內的

- 巨大的記憶語料庫
- 人格包裝
- 為了做大而做大的 graph infrastructure

## starter 內容

- [`docs/PUBLIC_PRIVATE_BOUNDARY.md`](docs/PUBLIC_PRIVATE_BOUNDARY.md)
- [`docs/APPLICATION_CHECKLIST.md`](docs/APPLICATION_CHECKLIST.md)
- [`docs/CODEX_FOR_OSS_DRAFT.md`](docs/CODEX_FOR_OSS_DRAFT.md)
- [`examples/rico-example.yaml`](examples/rico-example.yaml)
- [`templates/session-bootstrap.md`](templates/session-bootstrap.md)
- [`templates/task-handoff.md`](templates/task-handoff.md)

## 這個 repo 可能幫到誰

- 維護多 session AI workflow 的 maintainer
- 正在做 agent memory 或 continuity layer 的團隊
- 想讓 AI 不再每次都像新助手一樣的人
- 需要在人與 agent 之間做更乾淨交接的團隊

## 為什麼把它開源

我們想公開的，是別人真的能拿去用的方法層：

- schemas
- recall patterns
- handoff templates
- starter examples

如果你在做的，不只是讓 AI 記得更多，
而是想讓 AI 在跨時間後還像同一個它，
這個 repo 可能會對你有幫助。

## 路線圖

### v0

- 公開最小 RICO 結構
- 公開一個 synthetic example
- 公開適合 method-first open source 的發佈指引

### v1

- 加入更多 continuity、recall、handoff 範例
- 加入 cue registry 草稿
- 加入 session bootstrap 與 task handoff 模板

## License

MIT
