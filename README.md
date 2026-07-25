# KEE — 本地知識庫系統

這是你的本地知識庫。因為公司電腦不能裝 Obsidian，改用「純 Markdown + AI agent」做同一件事：
Obsidian 負責的雙向連結、圖譜、索引，這裡全部交給 agent 手動維護；你不用點圖譜，直接問。

**支援 Claude Code 和 Codex CLI 兩種工具**——同一份 `CLAUDE.md` 是唯一規則來源，
`AGENTS.md` 是給 Codex 用的指標檔（內容就是叫它去讀 `CLAUDE.md`），兩邊看到的是同一套規則，
不會分裂成兩份要分別維護。指令實作各自放：Claude Code 用 `.claude/commands/`，
Codex 用 `.codex/prompts/`，內容互為鏡像。

> ⚠️ Codex 的 custom prompts 資料夾位置本地沒有實機驗證過（官方文件在這次查詢時沒挖到完整
> 規格），`.codex/prompts/` 是專案層級的最佳猜測。如果 `/ingest` 這類指令在你的 Codex 版本上
> 打不出來，把 `.codex/prompts/` 底下三個檔案複製一份到全域路徑 `~/.codex/prompts/` 再試一次；
> 就算 slash command 完全不吃，直接跟 Codex 打自然語言「幫我 ingest 這個」也會生效，
> 因為觸發規則本來就寫在 `CLAUDE.md`／`AGENTS.md` 裡，不綁定特定語法。

## 下載安裝

不用裝任何額外軟體，不用資料庫，不用 Obsidian。**唯一前提**：電腦上已經裝好
[Claude Code](https://claude.com/claude-code) 或 [Codex CLI](https://developers.openai.com/codex)
其中一個——這個 repo 本身只是一堆規則檔案，是給這兩個工具讀的，不是獨立跑的程式。

```
方法一：會用 git
  git clone https://github.com/Jason5330/kee.git
  cd kee

方法二：不會用 git
  打開 https://github.com/Jason5330/kee
  → 綠色「Code」按鈕 → Download ZIP → 解壓縮 → 打開那個資料夾
```

## 開始使用

在剛剛那個資料夾（`kee`）裡開終端機：

```
用 Claude Code：cd kee → 執行 claude
用 Codex CLI： cd kee → 執行 codex
```

**位置一定要在 `kee` 資料夾裡面**，工具才找得到 `CLAUDE.md`／`AGENTS.md` 和指令檔。
第一次用，wiki 是空的：直接丟資料進去說「/ingest」，或先問問題看它怎麼回應都可以——
第一次動作時 agent 會自動把 `wiki/index.example.md`、`wiki/log.example.md` 複製成正式在用的
`wiki/index.md`、`wiki/log.md`，不用手動處理。

## 三個指令，日常只用這三個

```
新資料進來        你有問題想問
    ↓                  ↓
 /ingest            /ask
    ↓                  ↓
agent 讀資料 →       agent 查 wiki →
拆成筆記、建/更新    整理相關頁面 →
wiki 頁面、互相連結   附引用回答你
    ↓                  ↓
 記一筆 log.md        (可選)存成新頁面
```

```
偶爾（累積一段時間後）
    ↓
  /lint
    ↓
agent 健檢整個 wiki：
斷link / 孤兒頁 / 反向連結漏補 / 矛盾
    ↓
能自動修的直接修，修不了的列給你決定
```

## 資料夾在幹嘛

| 資料夾/檔案 | 這是什麼 | 你要手動碰嗎 |
|---|---|---|
| `sources/` | 原始資料（PDF、文章、貼上的文字） | 丟進去就好，不要編輯 |
| `wiki/` | agent 整理好的筆記本體，會一直長大 | 不用，agent 維護 |
| `CLAUDE.md` | 這套系統唯一的規則說明書 | 規則要改才動 |
| `AGENTS.md` | 給 Codex 看的指標檔 | 不用動，改規則一律改 `CLAUDE.md` |
| `.claude/commands/` `.codex/prompts/` | `/ingest` `/ask` `/lint` 的指令實作 | 不用動 |

## 第一步要做什麼

1. 把你手上現成的資料（PDF、筆記、文章連結、隨便一段文字）丟給 agent，說「/ingest 這個」。
2. 之後想到什麼問題，直接問，不用特別打指令，agent 也會自動判斷是不是該查 wiki。
3. 資料越餵越多，`wiki/index.md` 會自己長成一份完整目錄，你隨時可以打開看目前知識庫有什麼。

## 資料是 Excel 的話

Excel/CSV 這種表格資料跟一般文章不一樣，`CLAUDE.md` 裡有專門一段規則（「1a. Ingest 表格類
資料」）：整份 `.xlsx` 存進 `sources/`（檔名帶日期，同一份表格更新就存新檔、不要覆蓋舊檔），
但 wiki 頁面只放**摘要層級的結論**（欄位有哪些、關鍵數字、趨勢），不會把整張表逐列搬進筆記。
之後你問到摘要沒涵蓋的明細數字，agent 會照頁面「來源」區塊的指引回頭開對應的 Excel 檔案重新讀，
不會憑摘要瞎猜。

## 開源分享給別人用

這個資料夾本身就是可以直接 `git clone` 出去的模板：`sources/` 和 `wiki/` 裡你自己的真實內容
已經被 `.gitignore` 排掉（只留 `.gitkeep` 保住空資料夾結構、`*.example.md` 當起始模板），
別人 clone 下來後 agent 第一次動作時會自動把 `wiki/index.example.md` → `wiki/index.md`
（`log.example.md` 同理），不會拿到你的個人資料。

分享前記得：
- `LICENSE` 裡的 `[YOUR NAME OR GITHUB HANDLE HERE]` 換成你自己的名字/帳號。
- 檢查 `wiki/`、`sources/` 底下有沒有已經被你 ingest 進去、但其實不想公開的真實內容
  （這些檔案不會被 git 追蹤，但如果你之前用別的方式手動 `git add -f` 過就要注意）。

## 為什麼這樣設計

- **不用 Obsidian 也能有 Obsidian 的效果**：雙向連結、原子筆記、MOC 索引頁，這些都是「約定 +
  agent 手動維護」做出來的，純 Markdown 檔案本身就可攜帶，之後想換到 Obsidian 也是直接打開就能用。
- **知識會「複利」而不是每次重算**：這是 Karpathy 那篇 gist 的核心——傳統 RAG 每次現場從原始
  資料生答案，這裡是 agent 先把資料整理成一份會持續累積、互相連結的 wiki，之後回答問題只要
  讀 wiki（快很多，也更準），原始資料只在第一次 ingest 時被讀過。
- **維護成本轉嫁給 agent**：知識庫失敗的常見原因是「維護比它的價值成長得快」，人會放棄更新。
  這裡反向連結、索引更新、健檢，全部規則寫進 `CLAUDE.md`，由 agent 每次自動做，你只需要
  負責「餵資料」和「問問題」。
