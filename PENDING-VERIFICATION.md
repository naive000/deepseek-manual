# 待驗證清單(下次接手先看這份)

這份跨兩個 skill(`deepseek-manual` + `deepseek-outsource`),列出目前還沒實際跑過的動作。目的是讓你或 CC 下次回來這個主題時,不用重新回想「上次做到哪」,直接照這份清單接續。

## deepseek-manual(dsh UI 操作手冊)未驗證的 6 項

1. `/compact` 實際觸發後的畫面(沒點過,不確定會不會消耗 API 額度)
2. `/feedback` 實際觸發後的互動介面長怎樣(沒點過)
3. 插件列表分頁的完整插件清單(只看過分類:終端/Agent 循環/網頁搜索,沒逐一展開列出品項)
4. 「上下文已用 X%」按鈕點下去有沒有額外資訊(只確認按鈕存在)
5. 「後台任務」面板在**還在執行中**的 session 上長什麼樣子(只看過已完工 session 的空清單)
6. 網頁 UI 有沒有 CLI 打不到的專屬能力,或反過來(沒驗證過)

補完一項,就去 `SKILL.md` 的「涵蓋率與未驗證清單」把那條刪掉、內容搬進對應章節,這份清單也同步刪掉那條。

## deepseek-outsource(委派工作流程)未驗證的部分

- ~~整套 14 步 SOP 一次都還沒真的拿任務跑過~~ **2026-08-16 已完成 MVP 2.0**:throwaway repo + 字串反轉小任務,14 步全跑完,merge 成功。撞到的坑已經修進 SKILL.md/brief-template.md/ROADMAP.md,細節看 ROADMAP.md 的 MVP 2.0 段落。
- 尖峰時段檢查邏輯**這次有真的觸發過**(測試當下是尖峰,AskUserQuestion 跳出來問「現在跑還是等離峰」,使用者選了等離峰,離峰後才繼續)——這部分驗證過了。
- ~~「還在執行中的任務怎麼監控」仍然沒有真實案例~~ **2026-08-16 晚間補測完成**:純研究/調查類型的 AI 新聞彙總任務花了約 6 分鐘才寫出報告,期間真的監控了完整過程(從搜集素材→查證關鍵事實→寫報告→自查),看到了「Deep diving...」持續倒數計時、`Bash`/`Search`/`Think` 事件流即時累加的畫面。實測還發現:`read` 工具讀長對話時會從頭截斷在 5000 字元,要看**最新**進度得先 `scroll` 到底部再 `read`(單純 `viewportOnly: true` 沒用,不會自動跳到底部)。
- `/goal` 動作輪數上限對「自查清單」實際吃掉多少輪數,這次沒特別去看軌跡分頁數字,仍未測過具體數值。
- 四種任務類型只有「一般任務」這次用新 skill 本身驗證過,Code Review / Simplify / 翻譯 三種還是只有舊手動流程下的案例佐證,skill 化後的實際操作細節還沒被驗證。
- ~~2026-08-16 第二輪優化新增 3 項未驗證~~ **2026-08-16 同日稍晚,dsh 服務不忙時全部補測完成**,用一個真實 probe 任務(worktree 內改檔+`git commit`)一次驗證:
  1. **dsh 工作區刪除選單**:確認可靠開啟方式是「hover treeitem 讓操作按鈕浮出 → 緊接著立刻 click 那顆操作按鈕(中間不要插入 snapshot/wait)→ 選單裡點『刪除工作區』→ 確認 dialog 再點一次『刪除工作區』」。⚠️ 附帶發現:這個動作只刪掉「工作區」分組,底下的 session 會被移進側欄的「未分組」桶,不會真的消失——這解釋了「未分組」為什麼常年一堆歷史 session。已補進 SKILL.md 第 13 步。
  2. **mid-task 授權升級提示**:真實看到了「審批詳情」卡片(拒絕/允許一次兩顆按鈕),但抓到畫面時已經是 disabled——代表 dsh 後端在 Workspace Write 檔位下對「worktree 內 commit 撞 index.lock」這種情境會自己秒過,不需要人工點擊,DeepSeek 自己的軌跡也記錄了「escalation was approved」的過程。SKILL.md 的自動核准邏輯已經改成「這個模式不用管,dsh 自己會過;只有真的看到按鈕是可點狀態卡住才需要 CC 介入」。~~Full access 檔位下的行為仍未驗證~~ **2026-08-16 同日晚間補測**:Full access 檔位下同任務全程沒有任何升級提示(自動或人工皆無),只在一開始有一則「上下文注入 user-approval / permission preset danger-full-access」宣告進入高權限模式,之後暢通到底,32 秒完工——比 Workspace Write 版本快很多,這是權限檔位的速度/安全取捨。
  3. **worktree 路徑**:同一次 dsh 服務不忙時重測,真實 `/home/crazy/<project>-wt` 路徑的工作區選擇對話框秒開、正常運作——確認之前卡在「加載中…」是當下服務過載的暫時現象,不是路徑本身有問題。唯一真的不行的是 `$TMPDIR` 這類 CC sandbox 暫存目錄(mount namespace 隔離,dsh 看不到)。也順便確認了選資料夾要用「逐層點清單進去」,不是靠編輯路徑文字框打完整路徑按 Enter。
  - 額外意外收穫:實測過程中發現這台機器的 rtk 輸出壓縮工具連 DeepSeek 自己的 bash 環境都會套用,曾經讓它憑空多繞了 11 個步驟去除錯「奇怪的 git diff 格式」——已經補進 `assets/brief-template.md`,以後任務書會先講清楚省下這筆輪數。CC 自己的 bash 也一樣被 rtk 包住(不是只有 DeepSeek 環境),`git log --graph` 這類指令的輸出會被壓縮到看不出真實的 commit 樹狀結構,驗證 merge commit 是不是真的產生兩個 parent 時,一定要用 `/usr/bin/git` 才能看到準確結果。

- **2026-08-16 晚間,使用者要求「全部都要SOP走一輪」,補跑了一次完整 14 步 SOP(含真正的 merge)**,新發現且已解決的問題:
  1. **dsh Full access 有自己的 UI 確認關卡**:選 Full access 後跳出「確認啟用 Full access?」對話框,要勾選「我已了解風險,並願意繼續」核取方塊,「啟用 Full access」按鈕才會從 disabled 變成可點。已補進 SKILL.md 第 5 步。
  2. **dsh 工作區刪除選單第一次點擊偶爾會落空**(點到 treeitem 本身把它收合,而不是點到操作按鈕):徵兆是選單沒開、treeitem 變成 `[collapsed]`。解法是重新 `hover` 拿新鮮的按鈕 ref 再點一次,不要用舊 ref 重試。已補進 SKILL.md 第 13 步。
  3. **`tab_groups list` 回報的 page id 可能跟 `tabs list` 對不上**:實測撞到一次群組回報掛在 page 166,但 `tabs list` 顯示自己真正擁有的是 167,166 反而落在「User's tabs」(不屬於任何 agent)。解法:關閉前先用 `tabs list` 交叉核對,對不上就直接對自己真正擁有的 page id 用 `tabs` 的 `action: "close"`,不要盲信 `tab_groups` 回報的 id——實測關掉自己的分頁後,對應群組通常也會跟著消失。已補進 SKILL.md 第 13 步。
  4. **merge 步驟本身(step 12)首次真實驗證**:`git merge --no-ff` 真的產生了雙親 merge commit,merge 後在 main 重跑測試全綠,整條 SOP 從觸發→worktree→dispatch→驗收→merge 閘門→merge→雙層清理,第一次真正串成一條龍。

現在這份清單裡,deepseek-outsource 委派工作流程本身已經沒有已知未驗證項目;剩下的都是 deepseek-manual UI 手冊那 6 項零散功能(見上方)。

## 建議下次接手的做法

1. 找機會驗證另外三種任務類型(Code Review / Simplify / 翻譯)用新 skill 本身跑一次——目前只有「一般任務」跟「純研究/調查」被真實驗證過。
2. 同一個 browserclaw session 裡,順手點開 `deepseek-manual` 沒探過的 6 項(尤其 `/compact`、`/feedback`——這些在一次真實委派任務進行中最容易自然碰到)。
3. `/goal` 動作輪數上限對「自查清單」實際吃掉多少輪數,還沒測過具體數值,有機會可以順手記一下。
4. 跑完各自更新對應文件的狀態、commit、push。
