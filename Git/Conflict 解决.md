## ours 和 theirs
git 分支经过 [[Rebase 和 Merge]]合并后，二者可能会产生冲突(conflict)，很多时候希望只使用其中一方的修改。
`--ours`和 `--theirs`选项可用于解决**both added、both deleted、both modified**等类型的冲突，**只保留一方的修改**。


## 步骤
以 git rebase 合并命令为例说明解决冲突的步骤。
```bash
$ git checkout --ours <fileName>
$ git add <fileName>   //标记该冲突已解决
$ git rebase --continue 
$ git status //查看当前冲突状态
```

1. ours或theirs选择要保留一方;
2. `gi`标记冲突已解决;
3. 使用--continue继续执行rebase;
4. 查看冲突状态;
5. 在解决冲突过程中，如果遇到某个提交记录不需要应用，可以用下面命令忽略：

git rebase --skip
使用--abort命令可以取消本次rebase

git rebase --abort