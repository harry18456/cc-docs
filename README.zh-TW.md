[English](README.md) | 繁體中文

# docs — Doc Mine & Distill

把你正在研究的專案(repo、CLI、library)整理成 Markdown 知識,**每一句都帶依據**——`path:line`、官方 URL、或你實際跑過的指令——追不到來源的就標 `[UNVERIFIED]`,不用猜的補。

內含兩個 skill:

| Skill | 產出 | 什麼時候用 |
|---|---|---|
| **`doc-mine`** | `docs/` 下依編號分類的文件:`00-answers.md`(專門回答*你*問的)、`1x` 技術原理、`2x` 應用場景、`3x` 操作參考、`4x` 設計取捨、`90-unverified.md`,加一份 `README.md` 索引 | 你在研究一個專案,想把它的知識攤開來看 |
| **`doc-distill`** | **一份**自包含的檔案,10 分鐘讀完——沒有「詳見」、讀者不懂的術語就地解釋——可以直接貼、直接給人、直接餵給另一個 agent | 你想要一份可以直接交給別人的檔案 |

兩個都是**只能手動呼叫**(`disable-model-invocation: true`):Claude 不會自己觸發,你不打指令就**不佔任何 context**。

## 前提

無——**零依賴**、跨平台,全部跑在 Claude Code 內。

## 安裝

```
/plugin marketplace add harry18456/cc
/plugin install docs@harry18456
```

重開 Claude Code 生效。

### 本機立即使用(免 push、免 marketplace)

把 repo 以 junction 連到 skills 目錄,下個 session 就自動以 `docs@skills-dir` 載入:

```powershell
New-Item -ItemType Junction -Path "$env:USERPROFILE\.claude\skills\docs" -Target "D:\side_project\cc-docs"
```

> ⚠️ 之後改從 marketplace 正式安裝前,先移除這個 junction(`Remove-Item $env:USERPROFILE\.claude\skills\docs`),避免同名 skill 重複載入。

只想試一個 session:`claude --plugin-dir D:\side_project\cc-docs`。

## 怎麼執行

在要整理的專案目錄開 Claude Code,直接打指令:

```
/docs:doc-mine
/docs:doc-distill
```

補充指示接在指令後面——`/docs:doc-mine 技術和應用分開寫`。想跳過訪談,在同一則訊息說「直接開始」。`doc-distill` 可以說寫給誰、寫到哪——`/docs:doc-distill 寫給新同事,放 docs/onboarding.md`。slash 選單會顯示這些提示。

> 若是用下面的 junction 安裝而非 marketplace,指令名稱是 `/docs@skills-dir:doc-mine` 和 `/docs@skills-dir:doc-distill`。

## `doc-mine` 怎麼運作

```
0. 掃一眼 ──── README + 頂層目錄,讓訪談問得出具體模組名;
               確定輸出位置(目標檔已存在就先問你)
1. 訪談 ────── 一輪最多兩題,問到你說「開始」
2. 讀專案 ──── 小專案主線自己讀;大專案派 2–4 個 subagent 依程式碼區域分工,
               硬上限 4 個,每個自己讀完回報(不巢狀)
3. 寫文件 ──── docs/10-how-it-works.md … 40-design-tradeoffs.md,十位數是類別;
               重點多的類拆成 11-、12-…;每句敘述句尾帶標記
4. 回應 ────── docs/00-answers.md:你問的、你以為的 → 逐條驗證對或錯
5. 收尾 ────── 00-answers.md 每個 path:line 開檔重看,其他文件每份抽 5 條
               (錯 1 條就整份驗);每個 [doc:] URL 重開一次;
               寫 90-unverified.md 和 README.md 索引
```

訪談問五件事:你目前怎麼理解、你用它做什麼、最想搞清楚哪部分、**你覺得應該是這樣但沒確認過的**、哪些不用整理。第四題是重點——每一條都會變成有出處的*對*或*錯*。

## 證據標記

| 標記 | 來源 | 例 |
|---|---|---|
| `[src: <path>:<line>]` | 專案內的檔案(原始碼、README、config) | `[src: src/main.rs:42]` |
| `[doc: <URL>]` | 官方網站、官方文件,**限本次實際開過的頁面**,不憑記憶 | `[doc: https://example.com/docs/cli]` |
| `[run: <指令>]` | 實際執行過的**唯讀**指令(`--version`、`--help`、dry-run);會寫檔、安裝、連網的指令先問你再跑。指令和輸出一起貼進文件;指令一行寫不下時,貼成編號的程式碼區塊(`R1`、`R2`…),標記寫 `[run: R1]` | `[run: rtk --version]` |
| `[UNVERIFIED]` | 三種都沒有,括號寫缺什麼 | `[UNVERIFIED](找不到寫入邏輯)` |

命名、慣例、「通常是這樣」只是*找*證據的線索,本身不是證據。寫到專案的句子都要標,使用建議也算,標記裡寫的就是證據本身;只有導覽句(介紹文件、指向其他檔案)不加標記。

## `doc-distill` 怎麼運作

拿最近的輸入——對話裡已整理的內容 → 既有的 `docs/` → 原始碼(最多 2 個 subagent)——寫成一份檔案(預設 `docs/distill.md`)。篇幅以 10 分鐘讀完為目標:一段話刪掉後讀者不會誤解、用錯或改錯這個專案,就刪;語言和工具的通用知識不寫。收尾時通篇搜中英文的引用(「詳見」「參考」「見 docs/」/ see、refer to、see docs/),每一處換成內容本身,再把每個 `[src:]` 開檔重看、每個 `[doc:]` 重開一次。

## ⚠️ 注意事項

- **subagent 上限**:`doc-mine` 最多派 4 個、`doc-distill` 最多 2 個,且每個都被交代自己讀完回報、不再往下派。Claude Code 本身預設允許 subagent 巢狀 3 層;想要硬性擋住,在 settings 設 `CLAUDE_CODE_MAX_SUBAGENT_SPAWN_DEPTH` 為 `1`(全域生效,會影響其他 skill)。
- **`[UNVERIFIED]` 是設計,不是缺陷**:skill 寧可留一個看得見的洞,也不用一個聽起來合理的猜測填掉。收尾會把每一條列給你;`doc-mine` 另外集中寫進 `90-unverified.md`。
- **輸出**:語言跟你對 Claude 說話的語言,你指定就依你。放在 `docs/`(repo 用別的目錄名就跟它)。那個目錄已經有專案自己的文件,就改放到底下的 `mined/`。目標檔已存在不會直接覆蓋,skill 開始讀專案前就會先問你。

## 這個 plugin 的組件

目前只用了 **skills**(`skills/doc-mine/`、`skills/doc-distill/`,Claude Code 自動發現、不必在 plugin.json 宣告)。`commands/`、`agents/`、`hooks/`、`output-styles/`、`bin/` 為預留骨架,日後擴充直接放檔案即可。

## 相關檔案

- **skill 本體**:[`skills/doc-mine/SKILL.md`](skills/doc-mine/SKILL.md)、[`skills/doc-distill/SKILL.md`](skills/doc-distill/SKILL.md)
