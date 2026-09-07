---
name: doc-distill
description: Condense a project's knowledge into ONE self-contained Markdown file, every sentence still sourced.
disable-model-invocation: true
---

# doc-distill

把一個專案的相關知識濃縮成**一份**自包含的 Markdown。讀者手上只有這一份,讀完就懂。每一句仍然**有據**。

## 輸入來源(依序找,用最先找到的)

1. 這段對話裡已經整理好的內容(剛跑完 doc-mine 就直接用)
2. 專案既有的 `docs/`
3. 專案原始碼——這時才派 subagent,**上限 2 個**,依區域分工,自己讀完直接回報

使用者沒說寫給誰,預設寫給**沒看過這專案的工程師**。

## 自包含約束

- 每個術語第一次出現時,同一句解釋。
- 讀者需要的內容直接寫進來;原本要引用其他文件的地方,把那段內容搬進來取代引用。
- 證據標記保留——它指向原始碼和官網,不是其他文件。
- 只有一份檔案。使用者指定路徑就用它,否則寫 `docs/distill.md`。

## 核心原則:有據

敘述的依據只有三種——專案裡的檔案、官方文件、實際執行結果。命名、慣例、「通常是這樣」只是線索;找不到證據就寫 `[UNVERIFIED]` 並說明缺什麼。**寧可留白,不補猜。**

證據標記放在句尾,一句一個:

| 標記 | 來源 | 例 |
|---|---|---|
| `[src: <path>:<line>]` | 專案內的檔案(原始碼、README、config) | `[src: src/main.rs:42]` |
| `[doc: <URL>]` | 官方網站、官方文件 | `[doc: https://example.com/docs/cli]` |
| `[run: <指令>]` | 實際執行過,指令和輸出一起貼進文件 | `[run: rtk --version]` |
| `[UNVERIFIED]` | 三種都沒有,括號寫缺什麼 | `[UNVERIFIED](找不到寫入邏輯)` |

## 結構

依序:概述(它是什麼、解決什麼)→ 核心概念 → 怎麼用(含 `[run:]` 的實際輸出)→ 注意事項與取捨 → `[UNVERIFIED]` 清單。「濃縮」是去掉重複和鋪陳;內容該多長就多長。

## 收尾

1. 通篇搜「詳見」「參考」「另見」「見 docs/」,每一處改成內容本身。
2. 每個 `[src:]` 實際開檔看那一行,對不上就改。
3. 回報檔案路徑和 `[UNVERIFIED]` 數量。

完成條件:全文找不到指向其他文件的引用;`[src:]` 全部對得上。
