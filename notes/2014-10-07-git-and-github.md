# 目的
  1. 學會使用 git 版本控制器來管控"文字類型"的創作歷程。
  2. 在腦中建立使用 git 指令時的圖像。

# 內容
  0. 何謂 git ? 為什麼要用 git ?
    - 版本控制器, 相似的還有 CVS, SVN, Mercurial…
    - 寫程式也是一種創作, 就像是寫文章、寫小說、寫論文, 定稿之前會不停地修改。
    - 寫程式與一般文字創作, 差別在於, 程式是一種相當動態的創作行為, 不只結果重要, 過程也很重要; 而且, 程式的作品沒有定稿的一天, 永遠可以有新的方法可以做的更好, 永遠有 bug 要修。
  1. git 安裝好之後, 預設是在 CLI 界面下操作 (可安裝使用 GUI client, 但仍然要有觀念)
    - 使用 CLI 操作好像在跟一個無形的東西做互動
    - 然而 git 有提供讓你查詢這個無形的東西現在狀態的方法 (status, log, show, diff)
  2. 大部分 git 指令要有作用的前提, 是目前使用者所在的目錄位置, 或目錄位置上層有一個 git repo; 而只有兩種方式能產生 git repo
    - git init
    - git clone
  3. 在 local 使用 git
    - 三個重要的概念 repo, staging area, 還有 working directory
    - 被 commit 進去 repo 的檔案才會成為版本追蹤的對向
    - 何謂 git commit? git 的 commit 是漸進式的, 每次只把變動過的檔案加進來
    - repo 攤開來看就像是一棵樹, commit 就是一個節點, 每個節點都有 ID, 即是 hash 值
    - commit 的 hash 值不夠 human friendly, 所以加入了幾個指標的概念如 HEAD, branch, tag
  4. 使用 remote 的 git repo
    - remote 會有一個名字, 一個專案能有多個 remote
    - origin 是一個特別的 remote
    - remote repo 的東西會歸納到 remote branch 底下
    - 何謂 github

# 參考資料
  - https://www.atlassian.com/git/tutorials
  - http://dylandy.github.io/Easy-Git-Tutorial/
  - http://www.vogella.com/tutorials/Git/article.html
  - http://rogerdudler.github.io/git-guide/