## rebase
git rebase [base-branch] 不进行分支合并，而是将 base-branch 分支的更改（最近的共同祖先之后的提交）在当前分支的最后依次应用一遍。
- 不合并，没有多余的合并历史记录；
- commit 作为 patch保存，更新后应用 patch；
- 打乱了历史提交记录。

## merge
git merge [brach] 对两个分支最新提交和公共祖先进行三方合并，并形成合并记录。
- 三方合并，有合并记录；
- 历史记录按时间顺序。


## 应用原则
rebase 和 merge 都可以合并分支，有以下应用原则：
1. rebase 适用于从公共分支合并最新代码到个人开发分支；
2. 
