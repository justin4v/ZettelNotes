## rebase
git rebase [base-branch] 不进行分支合并，而是将 base-branch 分支的更改（最近的共同祖先之后的提交）在当前分支的最后依次应用一遍。
- **不合并**，没有多余的合并历史记录；
- commit 作为 patch保存，更新后再应用 patch；
- **打乱历史提交记录**。

## merge
git merge [brach] 对两个分支最新提交和公共祖先进行三方合并，并形成合并记录。
- **三方合并，有合并记录**；
- 历史记录按时间顺序。

### merge的三种参数
-   -ff Fast-forward 模式：**当合并的分支为当前分支的后代，没有分叉**，实际没有合并直接移动 HEAD 指针到，那么会自动执行 `--ff (Fast-forward)` 模式，如果不匹配则执行 `--no-ff（non-Fast-forward）` 合并模式；
-   --no-ff : 非 Fast-forward 模式, 在任何情况下都会创建新的 commit 进行多方合并（及时被合并的分支为自己的直接后代）
-   --ff-only ：只会 `Fast-forward` 模式进行合并，如果不符合条件（并非当前分支的直接后代），则会拒绝合并请求并且退出。


## 应用原则
rebase 和 merge 都可以合并分支，有以下应用原则：
1. rebase 适用于从**公共分支合并最新代码到个人开发分支**；
2. merge 适用于将**其他分支合并到公共分支**。

公共分支是指与他人的合作分支，如 mater 、release、feature等。在 **公共分支上使用rebase 会导致公共分支的历史记录混乱，难以追踪**。
