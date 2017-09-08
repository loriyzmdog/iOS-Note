#Git 指令

### 基本指令

- cd *projecct location*
- git init
- git add .  (新增至暫存區)
- git commit -m "`comment xxxxxxx`"
- git status
- git log
- git checkout `branch name` (切換分支)

---

### 搬移檔案、刪除實體檔

- git reset (從暫存區Stage返回工作目錄WD) 

  或 git rm \--cached `xxxfile`，並非將實體檔案刪除

- rm `xxxfile` (此為實體檔案移除)

---

### 新增分支

- git branch (查看分支狀態)
- git branch `branch name` (新增branch)
- git **checkout** **\-b** `feature2` (新增branch並且直接切換過去)
- mkdir `forder name` (新增目錄)

---

### 刪除分支

- git branch \-d  `branch name` (刪除分支，刪除時必須跳出此分支)
- git branch \-D `branch name` (強制刪除分支，因git怕你誤刪，所以沒有merge的無法直接刪除)

---

### 分支合併

- git checkout master (切換至併入的主支)
- git merge `branch name` (若要merge分支，則切到併入的主支(ex:master)，將分支`branch name`合併進去)

---

### 分支回復 （合併後反悔）

- git log \--oneline (先查看上一個版本的ID編號，複製ID編號)

- git reset \--hard [commit ID] (回復至該ID編號版本)

  \--hard表示將該工作目錄的版本強制回復

  或 git reset \--hard ORIG_HEAD (回復至上一個版本)

---

### Git Flow開發流程

- **master**: 正式上線分支

  合併最好要有專人負責

  ​

- **develop**: 開發分支

  commit, merge最好要有專人負責

  ​

- **hotfix**: 緊急修補分支

  由master分支出來，可合併至master/develop，用途是解決正式版上線問題

  ​

- **feature**: 功能分支

  由develop分支出來，可合併至develop，用途是開發新功能

  ​

- **release**: 釋出版本分支

  由develop分支出來，可合併至master/develop，用途是開發下一個版本，也就是開發完成準備釋出時建立．此分支只會針對該版本作bug修補及commit而已，不要在release上繼續開子分支







