---
name: deepseek-manual
description: DeepSeek harness(dsh)網頁 UI(http://127.0.0.1:3080/)操作手冊——側邊欄、工作區/session 管理、斜線指令、權限檔位、模型選擇、goal 機制、設定對話框,以及「对话/轨迹」session 視圖。當使用者問 dsh 網頁 UI 本身怎麼操作、某個按鈕/選單/斜線指令是幹嘛的、或想了解 dsh 介面機制時使用——像是「dsh 這個按鈕幹嘛的」「dsh 網頁怎麼用」「/goal 是什麼」「dsh 的權限怎麼設」。這是純查詢/參考型 skill,查的是 UI 本身——不是拿來把 coding 任務委派給 DeepSeek(那是另一個獨立的 `deepseek-outsource` skill,管 worktree/dispatch/驗收/merge 整套工作流程)。
---

# DeepSeek Harness(dsh)網頁 UI 操作手冊

實測於 `http://127.0.0.1:3080/`(本機 `/home/crazy/deepseek-game` 建置)。

## 這份手冊的存在前提

這份文件的知識,建立在「CC 直接操作 browserclaw 控制 dsh 網頁」這個前提上——不是給人類讀的操作說明,是給下一次 CC 用 browserclaw 碰 dsh 時當記憶用的。目的是讓每一輪操作(不管是繼續補完這份手冊、還是執行 `deepseek-outsource` 的委派流程)都不用從頭重新摸索按鈕在哪、有什麼陷阱、哪條路徑最快,直接加速。

## ⚠️ 涵蓋率與未驗證清單

核心操作流程(開新會話→設權限→設模型→下指令→監控完工→匯出/歸檔)已經實測跑過,可以放心照這份手冊操作。但**不是每個按鍵都跑過**,下面這幾項是真正的缺口,不是保守估計——遇到要用到的時候要有心理準備可能跟這份文件描述的不一樣,先自己點開確認:

- `/compact` 實際觸發後的畫面(沒點過,不確定會不會消耗 API 額度)
- `/feedback` 實際觸發後的互動介面長怎樣(沒點過)
- 插件列表分頁的完整插件清單(只看過分類:終端/Agent 循環/網頁搜索,沒有逐一展開列出品項)
- 「上下文已用 X%」按鈕點下去有沒有額外資訊(只確認按鈕存在)
- 「後台任務」面板在**還在執行中**的 session 上長什麼樣子(只看過已完工 session 的空清單)
- 網頁 UI 有沒有 CLI 打不到的專屬能力,或反過來(沒驗證過)

之後補完一項,就把這份清單對應那條刪掉,搬到下面對應章節。

## ⚠️ 版本漂移(2026-08-29,dsh 更新到 0.1.2-alpha.1,來源:同日另一 session 實測記錄於 memory——本 session 未複驗,下次實跑時順手確認)

- **裸開 `http://127.0.0.1:3080/` 現在會 401**——要用啟動 log(`~/deepseek-game/dsh-web.log`)裡印出的帶 `?token=` 的完整網址開,browserclaw `navigate` 一次到那個網址就會直接是登入狀態,不用另外處理登入畫面。
- **「命令」按鈕改名成「指令」**——下面文件裡寫到「命令」按鈕的地方,實際畫面上可能已經顯示「指令」,功能沒變,先以畫面實際文字為準。
- **輸入框 placeholder 文字變了**:從「描述你想要构建的内容」變成類似「描述你想要构建的内容… / 调用指令 @ 文件或对话」——多了 `@ 文件或对话` 這種行內引用提示,別把側欄的「搜索会话」輸入框誤認成這個 composer。

## 啟動與基本狀態

- 網址:`http://127.0.0.1:3080/`(server 要先手動啟動:`cd /home/crazy/deepseek-game && export PATH="$HOME/.nvm/versions/node/v22.23.2/bin:$PATH" && corepack pnpm dsh web`)
- 首頁(未選任何 session):側邊欄 session 樹 + 底部空白輸入框(工作區未選時 placeholder 是「選擇工作區」,命令按鈕是 disabled)

### ⚠️ 開新會話的陷阱

點側邊欄「新建会话」按鈕後,如果畫面跳出一個已經在「進行中」的 session(treeitem 顯示「進行中 <workspace> 剛剛」),代表這不是空白新對話,是繼承了某個之前排隊/暫存的任務描述,且可能已經在真的執行(跑 Bash、Read 檔案等)。**這種情況下不要送出任何訊息、不要碰暫停/編輯/清除目標按鈕。**

**這是純前端 SPA**:導航回根網址 `http://127.0.0.1:3080/` **不會**清空畫面,只會恢復上一個瀏覽過的 session(連停在哪個分頁都原封不動恢復)。要開新會話,唯一正確做法是**直接點側邊欄的「新建会话」按鈕**。

## 側邊欄功能

| 按鈕 | 位置 | 功能 |
|---|---|---|
| 新建会话 | 頂部(收起/展開狀態各一個) | 開新對話(見上方陷阱) |
| 收起侧边栏 | 頂部 | 摺疊側邊欄 |
| 搜索会话 | 頂部 | 展開成搜尋框,過濾 session 樹 |
| 视图选项 | 頂部 | 下拉選單:分組方式「按工作区」/「单列表」;排序方式「手动排序」/「最近更新」 |
| 添加工作区 | 頂部 | 跟新建會話裡「選擇工作區→添加工作區」是同一套目錄瀏覽對話框 |
| 设置 | 底部 | 開啟設定對話框(見下方) |
| session 樹 | 中間 | 依工作區分組列出所有歷史對話,hover 出現操作按鈕 |

**工作區操作**(hover 工作區節點):重命名 / 刪除工作區。

**Session 操作**(hover 具體 session 項目):重命名 / 分叉会话 / 归档会话。hover session 項目時,畫面下方會浮出一個「複製:<工作區絕對路徑>」按鈕,一鍵複製該 session 綁定的工作區路徑。

## 新建會話畫面(選對應 PJ + 下指令)

1. **選擇工作區**(輸入框上方按鈕):點開是下拉選單,列出最近用過的工作區,最後一項「添加工作区…」跳出目錄瀏覽對話框。**關鍵功能**:對話框有「編輯路徑」按鈕,點下去導覽列變成文字輸入框,可以直接打完整絕對路徑,清單即時 fuzzy filter,不用逐層點資料夾——這是最快最穩定的操作路徑。底部還有:新建文件夹 / 显示隐藏文件 / 取消 / 打开。
   - ⚠️ 用工具清空文字框時,`fill` 類動作預設是「附加」不是「取代」,要先手動全選(Ctrl+A)+刪除再輸入。

2. **Agent 模式**(「標準模式」按鈕,工作區選擇器右邊):對應設定對話框的「Agent 预设」分頁(見下方),可切換 PTC 模式/極簡模式/創造模式,或自訂 preset。⚠️ **只能在這個新建會話畫面選,一旦發送訊息開始跑,這顆按鈕就從畫面上消失**(2026-08-29 實測:對比進行中 session 跟新會話畫面的 snapshot,前者完全沒有模式切換按鈕)——選錯要換模式,只能整個開新會話重來,沒有「跑到一半切換」這回事。怎麼判斷該選標準還是 PTC,見 `deepseek-outsource` 的「模式判定」那節。

3. **輸入框**(placeholder「描述你想要构建的内容」):直接打字輸入任務描述。打 `/` 或點旁邊「命令」按鈕跳出斜線指令候選清單。

4. **访问模式**按鈕(輸入框下方,預設「当前:Workspace Write」):見下方 `/permission`。

5. **選擇模型**按鈕(顯示「当前 DeepSeek-V4-Flash,推理等级 Max」):見下方 `/model`。

6. 填完內容(或用 `/goal <目標>` 設定長任務目標)後,「發送消息」按鈕從 disabled 變成可點,按下去正式開始執行。

## 斜線指令總表

打 `/` 跳出的候選清單,共 7 個:

| 指令 | 介面原文 | 行為 |
|---|---|---|
| `/compact` | Compact older conversation history | 壓縮舊對話歷史(未實測畫面) |
| `/export` | Download this Session log as a ZIP archive | 下載這個 session 的完整 log 成 ZIP(=「Session log」按鈕,同一功能) |
| `/feedback` | record feedback about this session | 記錄關於這個 session 的回饋(未實測畫面) |
| `/goal` | set or view the goal for a long-running task | 設定或查看長任務目標(見下方) |
| `/permission` | Switch the permission preset (sandbox mode + approval policy) | 切換權限預設(見下方) |
| `/plan` | Enter or leave plan mode | 進入/離開計劃模式 |
| `/model` | 选择本会话使用的模型 | 選擇模型/推理等級 |

### `/permission`(等同「访问模式」按鈕)

三個檔位,一路開放:**Read Only**(唯讀)/ **Workspace Write**(預設,可寫入所選工作區內檔案)/ **Full access**(完整存取)。

⚠️ **這是解決 headless CLI 升權問題的關鍵**:`dsh --profile headless "任務"` 模式下,需要升權的操作(例如寫入被鎖定的 `.git/index.lock`)會因「沒有互動批准管道」直接被拒絕。但網頁 UI 一次真實 session 記錄裡(`github-star-radar-simplify2` 任務),確實成功執行了 `git add`(一開始遇到 `Unable to create '.../index.lock': Read-only file system`,之後 log 顯示「Approved and staged」)——升權請求跑通了,可能跟目前是 `Workspace Write`(甚至更高的 `Full access`)這種較寬鬆的權限有關。

### `/model`(等同「選擇模型」按鈕)

兩層選單:**模型**(`DeepSeek-V4-Flash` 預設 / `DeepSeek-V4-Pro`)、**推理等級**(`Off` / `High` / `Max`,預設 Max)。

## Goal 機制(`/goal`)

長任務(long-running task)的目標追蹤器,不只是一句普通指令:

- 打 `/goal <目標描述>` 送出後建立一個 goal 物件:
  ```
  Goal created
  Status: active
  Objective: 「<目標描述>」
  Rounds: 0/256
  Activation: armed
  Commands: /goal edit <objective>, /goal pause, /goal clear
  ```
- Session 頂部出現三顆按鈕:**暫停目標** / **編輯目標** / **清除目標**
- Agent 持續執行直到自己判斷目標達成,呼叫內部工具 `get_goal` → `update_goal`(狀態改成 complete),自動產生結構化收尾訊息(做了什麼、驗收線、建議事項)——跟人類寫的任務簡報格式一致,代表 agent 被訓練成看到「brief 檔案 + 驗收線」格式就會照做並在完成時自動總結。
- 實測用法:目標描述可以只寫「讀這個資料夾裡的 `<TASK_FILE>.md`,照裡面寫的做」——`/goal` 不需要把完整規格塞進單一字串,可以只引用一個檔案路徑,讓 agent 自己去讀。

## 設定對話框(側邊欄「设置」按鈕)

四個分頁:

**通用設置**:語言(目前中文)、主題(淺色/深色/跟隨系統)、排隊發送(開關)。

**模型**:列出已配置的 LLM 提供方(目前 `DeepSeek (deepseek-official)`,API 金鑰已配置)。有「添加提供方」/「添加自定義提供方」——理論上可以接其他 LLM 供應商,不是綁死 DeepSeek 自家 API。

**插件**:兩個子分頁——插件配置(3 項可展開設定:終端/Agent 循環/網頁搜索)、插件列表(⚠️ 未逐一展開列出完整品項)。

**Agent 预设**:內建 4 個(標準模式/PTC 模式/極簡模式/創造模式)+ 自訂 1 個(`standard-claude`,讓 DeepSeek 可透過 `subagent_claude_code` 工具委派任務給本機 Claude Code)。每個 preset 有「設為預設」/「查看」/「複製」;自訂的多一個「打開目錄」/「刪除」。底部有「用「創造模式」創作自定義預設」的入口。

## 已完成 Session 的畫面

頂部 banner:
- 會話層級導覽(顯示目前 session 名稱,disabled,純顯示用)
- **背景任務**按鈕(例如「1 个后台任务」):點開展開「後台任務」清單面板。已完工 session 這個清單常常是空的(背景 job 完工後被收進正式軌跡記錄,不再留在面板顯示)——按鈕上的數字是執行期間曾丟過幾個背景 job,不是「現在還有幾個在跑」。⚠️ 執行中 session 上這個面板長什麼樣沒驗證過。
- **Session log** 按鈕:實測結果就是 `/export` 的捷徑,點下去跳「正在导出 Session」對話框,瀏覽器直接下載 ZIP,沒有另外的 log 檢視器畫面。
- 兩個分頁:**对话** / **轨迹**。

### 「对话」分頁

給人看的呈現方式:整個執行過程摺成一串可展開/摺疊的動作氣泡(「Think ...」「Code ...」「Read <檔名>」「Write <檔名>」),預設摺疊,點開才看完整內容;agent 最終正式回覆用 markdown 渲染(標題、清單、粗體、code block 都排版過)。

每則助手訊息下方有 **4 顆互動按鈕**:**复制**(複製訊息文字)、**好的回答**(標記品質好)、**有问题的回答**(標記有問題)、**在新对话中分支**(以這則訊息為分岔點開新 session,原 session 不受影響)。

### 「轨迹」分頁

給人看原始執行記錄,不是排版過的閱讀體驗:
- 工具列:**Use actual duration**(真實耗時而非壓縮顯示)、**Collapse turns**(摺疊整輪對話)、**Collapse calls**(摺疊個別工具呼叫)、**搜索轨迹**(關鍵字搜尋)
- 上方「Trajectory timeline」時間軸(可左右拖曳快速定位,有「Load earlier history」載入更早記錄)
- 主體是表格,每列一個原始事件:
  - `Request #N, ASSISTANT, <內容>`——每輪助手回應,編號遞增
  - `TOOL, run_code {...}`——助手發起的工具呼叫,帶完整原始 JSON 參數
  - `SUBTOOL, <實際工具名> {...}`——`run_code` 底下實際觸發的子工具(`todo_write`、`write`、`read`、`bash`、`get_goal`、`update_goal`、`job_output` 等),一樣帶完整原始參數
  - `CONTEXT, <系統注入的上下文>`——例如任務目標達成時系統自動注入 `<goal_complete>` 指示文字,直接命令 agent 寫收尾訊息
  - `USER, <使用者輸入>`——使用者自己打的訊息,原文原樣顯示

**判斷準則**:「对话」是給人看懂「發生了什麼、結果是什麼」的排版視圖;「轨迹」是「精確發生了哪些工具呼叫、原始參數是什麼、系統背後注入了什麼隱藏指令」的除錯視圖。**要驗收 DeepSeek 有沒有照 brief 做,看「轨迹」比「对话」更可靠**——「对话」的摺疊氣泡可能省略細節,「轨迹」的 SUBTOOL 那一列是最接近「它到底執行了什麼」的原始記錄。

### 輸入框工具列(進入任一 session 後)

除了「命令」按鈕,還有:**访问模式**按鈕(顯示目前檔位,等價 `/permission`)、**選擇模型**按鈕(顯示目前模型+推理等級,等價 `/model`)、**上下文已用**按鈕(例如「上下文已用 16%」,⚠️ 點下去有沒有額外資訊未驗證)。

### 其他觀察(來自一次真實 simplify 任務的軌跡回放)

- Agent 完工後除了打印總結訊息,被使用者用一句話追問(例如「跑完了?」)時只單純回答摘要,不會重新觸發任何動作
- Agent 執行過程中會自己抓到並修正環境坑(例如發現 `git` 指令被 `rtk` wrapper 攔截、bracket 路徑 `[locale]` 撞到 git pathspec glob 語法),過程透明可查

## 跟另一個 skill 的分工

這份手冊只管「dsh 網頁 UI 本身怎麼操作」。真的要把一個 coding 任務委派給 DeepSeek 執行(worktree 隔離、寫任務書、監控、驗收、merge)是另一個獨立 skill `deepseek-outsource` 的範圍,兩者分開維護,不要混淆。
