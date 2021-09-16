## rebase
git rebase [base-branch] 不进行分支合并，而是将 base-branch 分支的更改（最近的共同祖先之后的提交）在当前分支的最后依次应用一遍。
- 不合并，没有多余的合并历史记录；
- commit 作为 patch保存，更新后应用 patch；
- 打乱了历史提交记录。

## merge
git merge [brach] 对两个分支最新提交和公共祖先进行三方合并，最