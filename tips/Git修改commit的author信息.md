# Git修改commit的author信息

对于 git 的用户名与邮箱，我们大多数人都有两套，一套公司账户平时工作用和一套个人账户用于业余项目。而我又常常在 commit 以后才发现用户名与邮箱设置错了。本文总结了如何修改 Commit 的用户名与邮箱。

## 如何设置用户名与邮箱

```bash
$ git config user.name "Author Name"
$ git config user.email email@address.com
```

上面的命令只会修改当前项目的配置。如果需要也可以增加 `--global` 选项来修改当前系统用户的默认配置。

## 修改 Commit 的用户名与邮箱

**注意： 只建议修改未 push 的 commit。**

因为修改 Commit 的用户名或邮箱会生成一个新的 commit 来替换之前的 commit 。如果在修改之前已经 push 到了远端，修改后再次 push 会出现冲突。 只能使用 `push -f`。 如果其他人已经拉取（ pull ）了旧 commit 会出现很多麻烦。

## 只修改最新的 commit

如果你只需要修改最新的 commit ，直接使用：

```bash
git commit --amend --author="Author Name <email@address.com>"
```

如果你已经修改了 git config 中的用户名和邮箱，也可以使用

```bash
git commit --amend --reset-author --no-edit
```

## 如果要修改连续多个 commit

比如，你的 commit 历史为 `A-B-C-D-E-F` ， F 为 `HEAD` ， 你打算修改 C 和 D 的用户名或邮箱，你需要：

1. 运行 

   ```
   git rebase -i B
   ```

    （

   这里有一个运行该命令后的例子（英文）

   ）

   - 如果你需要修改 A ，可以运行 `git rebase -i --root`

2. 把 C 和 D 两个 commit 的那一行的 `pick` 改为 `edit`

3. 当 rebase 开始后，将会暂停在 commit C

4. 运行 `git commit --amend --author="Author Name <email@address.com>"`

5. 然后运行 `git rebase --continue`

6. 将会继续暂停在 commit D

7. 再次运行 `git commit --amend --author="Author Name <email@address.com>"`

8. `git rebase --continue`

9. rebase 结束

10. 如果需要更新到远程仓库， 使用 `git push -f`（请确保修改的commit 不会影响其他人）

转载自：https://juejin.cn/post/6844903806224826375