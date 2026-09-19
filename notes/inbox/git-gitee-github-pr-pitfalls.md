# Git / Gitee / GitHub 实战中容易踩的坑总结

> 场景来源：空仓库创建、远程仓库绑定、Fork、Pull Request、远程分支、账号切换、HTTPS 凭据、手动合并 PR、跨仓库拉取等。

---

## 1. 新建远程仓库时，三个初始化选项不要乱勾

如果目的是完整练习 Git 流程，建议创建**真正的空仓库**：

- 不勾“初始化仓库”
- 不勾“设置模板”
- 不勾“选择分支模型”

原因：

如果远程仓库先自动生成了 README、`.gitignore` 等，就会先产生一个远程 commit；而本地又自己创建了 root commit，两边历史可能不同，第一次 push 会更复杂。

最干净的学习流程：

```text
本地 git init
→ git add
→ git commit
→ git remote add origin ...
→ git push
```

---

## 2. `git remote add origin` 只是给远程仓库地址起别名

例如：

```bash
git remote add origin https://gitee.com/rainkoli/demo.git
```

这里：

```text
origin
```

只是一个本地别名，通常表示“默认远程仓库”。

查看：

```bash
git remote -v
```

重要结论：

```text
origin ≠ GitHub/Gitee 账号
origin ≠ 分支
origin = 某个远程仓库地址的本地名称
```

---

## 3. `git clone` 会自动配置 `origin`

例如：

```bash
git clone https://gitee.com/SemiTris/demo.git
```

clone 完成后，进入仓库：

```bash
cd demo
git remote -v
```

通常自动存在：

```text
origin -> https://gitee.com/SemiTris/demo.git
```

所以 clone 后一般不需要再次：

```bash
git remote add origin ...
```

---

## 4. `git remote -v` 报 `not a git repository`

典型错误：

```text
fatal: not a git repository (or any of the parent directories): .git
```

常见原因：

你还停留在 clone 后的**父目录**，没有进入真正的仓库目录。

例如：

```text
git-test-dxa/
└── demo/
    └── .git/
```

应该：

```bash
cd demo
git remote -v
```

---

## 5. `origin`、`master`、`origin/master` 是三个不同概念

```text
origin
```

是远程仓库别名。

```text
master
```

是本地分支。

```text
origin/master
```

是“远程跟踪分支”，代表你上次 fetch 后了解到的远程 `master` 状态。

不要混淆。

---

## 6. `git push -u origin master` 中 `-u` 的作用

```bash
git push -u origin master
```

等价于：

```bash
git push --set-upstream origin master
```

作用有两个：

1. 把本地 `master` 推到 `origin/master`
2. 建立本地 `master` 与 `origin/master` 的 tracking 关系

之后通常可以直接：

```bash
git push
git pull
```

---

## 7. `--set-upstream` 容易拼错

错误：

```bash
git push --set-upstrem
```

正确：

```bash
git push --set-upstream origin master
```

或者：

```bash
git push -u origin master
```

---

## 8. `git checkout SemiTris01` 不是“切换账号”

这是一个非常典型的坑。

```bash
git checkout SemiTris01
```

Git 会把 `SemiTris01` 当成：

```text
分支名 / 路径
```

而不是：

```text
Gitee 用户名
```

如果该分支不存在，就会报：

```text
error: pathspec 'SemiTris01' did not match any file(s) known to git
```

切换账号和切换分支完全是两回事。

---

## 9. `git config user.name / user.email` 不是 GitHub/Gitee 登录账号

这是整组问题中最重要的一点。

例如：

```bash
git config --global user.name "rainkoli"
git config --global user.email "xxx@example.com"
```

这两个值主要用于：

```text
commit 作者署名
```

例如：

```text
Author: rainkoli <xxx@example.com>
```

它们不决定：

```text
谁有权限 push
```

---

## 10. Git 的“提交作者身份”和“远程认证身份”是两套系统

可以记成：

```text
git commit
    ↓
user.name + user.email
    ↓
谁写了这个 commit

git push
    ↓
HTTPS Token / SSH Key / Credential Manager
    ↓
谁正在访问 GitHub/Gitee，是否有权限
```

所以完全可能出现：

```text
commit 作者：rainkoli
push 认证用户：SemiTris
```

这是合法的。

---

## 11. GitHub 通过 commit 邮箱“关联账号”，不等于“用邮箱授权 push”

容易产生误解：

```text
GitHub 看到 commit 的 email
→ 可能把 commit 关联到某个 GitHub 账号
```

但这不等于：

```text
email 可以直接作为 push 权限认证
```

真正的 push 身份认证通常依赖：

```text
HTTPS Token
SSH Key
Credential Manager 中保存的凭据
```

---

## 12. 网站登录邮箱 / 用户名 和 `git config user.email` 不是同一回事

这两个“邮箱”要区分。

### 网站登录时的邮箱 / 用户名

用于：

```text
我是哪个账号
```

再配合密码、Token、2FA 等完成认证。

### `git config user.email`

用于：

```text
这个 commit 的作者邮箱是什么
```

不负责远程登录。

---

## 13. Gitee / GitHub 真正等价的是“认证机制”，不是 `user.email`

更准确的对应：

```text
Gitee 用户身份 + Token
≈
GitHub 用户身份 + Token
```

或：

```text
Gitee SSH Key
≈
GitHub SSH Key
```

而不是：

```text
Gitee 登录用户名
≈
git config user.email
```

---

## 14. 为什么换了 Gitee 账号/Token，却不需要改全局 `user.name` 和 `user.email`

因为你切换的是：

```text
远程认证身份
```

不是：

```text
commit 作者身份
```

所以只要清除旧凭据，再使用另一个 Gitee 账号 + Token，push 就会按新的远程身份认证。

而：

```bash
git config --global user.name
git config --global user.email
```

可以完全不变。

---

## 15. `403 Access denied` 通常是远程认证身份不对

例如：

```text
origin -> https://gitee.com/rainkoli/demo.git
```

但 Windows Credential Manager 里缓存的是：

```text
SemiTris 的 Gitee 凭据
```

那么：

```bash
git push origin master
```

可能得到：

```text
403 Access denied
```

因为：

```text
目标仓库：rainkoli/demo
认证用户：SemiTris
```

服务器发现该用户没有写权限。

---

## 16. 403 时不要去改 `git config user.name`

错误思路：

```bash
git config --global user.name ...
```

这通常解决不了 403。

因为：

```text
user.name
→ commit 作者

Token / SSH Key
→ push 权限
```

403 更应该排查：

```bash
git remote -v
```

以及：

```text
当前 HTTPS / SSH 认证使用的是哪个账号
```

---

## 17. Windows 会缓存 GitHub/Gitee HTTPS 凭据

所以即使你切换了 remote，Git 可能仍然自动使用之前缓存的账号。

常见表现：

```text
不弹登录框
直接 403
```

原因可能是 Credential Manager 已经保存了旧凭据。

---

## 18. 使用 `git credential reject` 清除某个站点的凭据

执行：

```bash
git credential reject
```

然后输入：

```text
protocol=https
host=gitee.com

```

最后再按一次 Enter，输入空行结束。

然后重新：

```bash
git push
```

Git 才可能重新要求认证。

注意：这不会修改：

```bash
git config --global user.name
git config --global user.email
```

---

## 19. Token / 密码不要和别人共享

练习双账号时可以模拟身份切换，但真实使用中不要共享：

```text
密码
Personal Access Token
SSH 私钥
```

更推荐：

```text
每个人使用自己的账号
每个人使用自己的 Token / SSH Key
```

如果需要多人协作，就通过：

```text
Fork
Collaborator / Member 权限
Pull Request
```

来完成。

---

## 20. Fork 后，`origin` 通常指向自己的 Fork

经典 Fork PR 模式：

```text
原仓库：
rainkoli/demo

Fork：
SemiTris/demo
```

如果 clone：

```bash
git clone https://gitee.com/SemiTris/demo.git
```

那么默认：

```text
origin -> SemiTris/demo
```

这是正常的。

---

## 21. 可以再加一个 `upstream` 指向原仓库

例如：

```bash
git remote add upstream https://gitee.com/rainkoli/demo.git
```

此时：

```text
origin   -> 自己的 Fork
upstream -> 原仓库
```

这是开源项目中非常常见的命名约定。

---

## 22. “upstream” 有两种常见含义，容易混淆

### 含义 1：remote 名称

例如：

```text
upstream -> 原仓库
```

这是人为起的 remote 名字。

### 含义 2：branch upstream / tracking branch

例如：

```text
本地 master
tracks
origin/master
```

这是分支跟踪关系。

两者都叫 upstream，但不是一个层面的概念。

---

## 23. `git pull` 后看到远程新分支，不代表已经创建本地分支

例如：

```text
[new branch] SemiTris01 -> origin/SemiTris01
```

说明 Git 已经获取到：

```text
origin/SemiTris01
```

这是远程跟踪分支。

但：

```bash
git branch
```

可能仍然只有：

```text
* master
```

因为本地 `SemiTris01` 还没创建。

---

## 24. 查看本地分支、远程分支、全部分支

本地：

```bash
git branch
```

远程：

```bash
git branch -r
```

全部：

```bash
git branch -a
```

带 tracking 信息：

```bash
git branch -vv
```

---

## 25. 从远程分支创建本地 tracking 分支

如果已经存在：

```text
origin/SemiTris01
```

可以：

```bash
git switch -c SemiTris01 --track origin/SemiTris01
```

之后关系：

```text
本地 SemiTris01
    ↕
origin/SemiTris01
```

---

## 26. `git pull` 本质上不是简单“下载”

经典理解：

```text
git pull
≈
git fetch
+
git merge
```

或根据配置：

```text
git fetch
+
git rebase
```

所以 `pull` 会影响当前分支。

---

## 27. `git fetch` 更安全：先把对方历史拿回来，但不改当前代码

例如：

```bash
git fetch origin
```

主要更新：

```text
origin/master
origin/feature
...
```

不会直接修改当前工作分支。

所以在不确定对方仓库内容时：

```text
先 fetch
再观察
再决定 merge / rebase
```

通常更稳。

---

## 28. Pull Request 手动合并的本质

Gitee 可能给出：

```bash
git checkout master
git pull https://gitee.com/SemiTris/demo.git SemiTris01
git push origin master
```

含义：

1. 切换到自己的本地 `master`
2. 从贡献者仓库的 `SemiTris01` 分支拉取并合并
3. 把合并后的本地 `master` 推回自己的 `origin/master`

这很好地暴露了 PR 的底层本质。

---

## 29. 临时 `git pull <URL> <branch>` 不会修改 `origin`

例如：

```bash
git pull https://gitee.com/SemiTris/demo.git SemiTris01
```

只是这一次临时从指定 URL 拉取。

不会自动把：

```text
origin
```

改成 SemiTris 的仓库。

之后：

```bash
git remote -v
```

仍然可以保持：

```text
origin -> rainkoli/demo
```

---

## 30. “配置不冲突”不等于“代码不会冲突”

从同学仓库临时 pull：

```bash
git pull https://gitee.com/SemiTris/demo.git SemiTris01
```

不会破坏你原有的：

```text
master -> origin/master
```

tracking 配置。

但是代码仍然可能冲突：

```text
CONFLICT (content)
```

所以要区分：

```text
远程/分支配置冲突
```

和：

```text
代码 merge conflict
```

---

## 31. Git 并不要求“只有 Fork 关系才能拉取”

任何 Git 仓库理论上都可以访问任何其他 Git 仓库。

例如当前是 C++ 项目仓库 B：

```bash
git remote add repoA https://gitee.com/xxx/repo-a.git
git fetch repoA
```

即使仓库 A 是 Java 项目，也完全可以 fetch。

Git 不理解：

```text
Java
C++
Python
```

Git 只关心：

```text
commit
tree
blob
ref
```

---

## 32. 完全无关的仓库也可以 `fetch`

假设：

```text
仓库 A：
A1 -- A2 -- A3

仓库 B：
B1 -- B2 -- B3
```

没有共同祖先。

你仍然可以：

```bash
git fetch repoA
```

此时只是把 A 的 Git 对象和 ref 拿到本地，不会强制合并。

---

## 33. 完全无关的历史直接 merge 可能被拒绝

典型错误：

```text
fatal: refusing to merge unrelated histories
```

因为双方：

```text
没有共同祖先 commit
```

如果确实要硬合，可以：

```bash
git merge repoA/master --allow-unrelated-histories
```

但要非常谨慎。

---

## 34. `--allow-unrelated-histories` 只是允许“强行建立共同历史”

原本：

```text
A1 -- A2 -- A3

B1 -- B2 -- B3
```

强行 merge 后可能：

```text
A1 -- A2 -- A3 ---\
                   M
B1 -- B2 -- B3 ---/
```

从 merge commit `M` 开始，两套历史被连接起来。

---

## 35. 两个无关仓库的同名文件容易产生冲突

例如：

```text
仓库 A 有 README.md
仓库 B 也有 README.md
```

强行合并可能出现：

```text
CONFLICT (add/add): Merge conflict in README.md
```

所以技术上能合，不代表项目结构上合理。

---

## 36. Git remote 本质只是“另一个 Git 仓库地址”

一个本地仓库完全可以配置很多 remote：

```text
origin      -> 自己的 Gitee
github      -> 自己的 GitHub
upstream    -> 原作者仓库
contributor -> 某个贡献者 Fork
repoA       -> 完全无关的另一个项目
```

Git 本身不会规定：

```text
谁才是“主仓库”
```

这是一种工作流约定。

---

## 37. PR / Fork 之所以合并自然，是因为它们通常共享提交历史

原仓库：

```text
A -- B -- C
```

Fork：

```text
A -- B -- C -- D
```

双方共享：

```text
A、B、C
```

所以合并 D 很自然。

这和两个完全无关仓库：

```text
A1 -- A2

B1 -- B2
```

是本质不同的。

---

## 38. `git status` 中 “ahead of origin/master by 1 commit” 的含义

例如：

```text
Your branch is ahead of 'origin/master' by 1 commit.
```

表示：

```text
本地 master 比 origin/master 多 1 个 commit
```

并不表示远程已经有这个 commit。

需要：

```bash
git push
```

才会发布到远程。

---

## 39. 如果误在 `master` 上提交，也可以把 commit 保留到新分支

例如本地 master 已经多了一个 commit，可以：

```bash
git switch -c feature/pr-demo
```

新分支会直接指向当前 commit。

不需要重新提交。

这是修复“本来应该在 feature 分支开发，却提交到了 master”时很常用的思路。

---

## 40. 本地作者身份可以用仓库级配置覆盖全局配置

全局：

```bash
git config --global user.name "rainkoli"
```

某个实验仓库里：

```bash
git config --local user.name "SemiTris"
git config --local user.email "semi@example.com"
```

当前仓库会优先使用 local 配置。

查看来源：

```bash
git config --show-origin user.name
git config --show-origin user.email
```

---

## 41. `--global` 的真正含义不是“绑定 GitHub/Gitee 全局账号”

`--global` 只是：

```text
当前操作系统用户下，所有 Git 仓库的默认配置
```

不是：

```text
GitHub/Gitee 的全局登录账号绑定
```

---

## 42. 建议用 `git config --show-origin --list` 排查配置来源

命令：

```bash
git config --show-origin --list
```

可以看到配置来自：

```text
system
global
local
worktree
```

很适合排查：

```text
为什么这个仓库和别的仓库行为不一样
```

---

## 43. `credential.https://gitee.com.provider=generic` 不是“当前 Gitee 用户”

例如配置里出现：

```text
credential.https://gitee.com.provider=generic
```

这只是 credential 相关配置。

它不表示：

```text
当前 Gitee 登录用户是谁
```

实际认证身份仍可能保存在 Credential Manager 或其他凭据存储中。

---

## 44. 两个 Gitee / GitHub 账号长期共存，HTTPS 反复切换比较麻烦

如果一台电脑长期使用多个账号，更推荐：

```text
SSH + 多套 SSH Key + ~/.ssh/config
```

这样可以给不同账号定义不同 Host 别名，避免频繁删除 HTTPS 凭据。

---

## 45. 一个完整的 Fork + PR 底层流程

可以记成：

```text
原仓库
rainkoli/demo
    |
    | Fork
    v
SemiTris/demo
    |
    | clone
    v
SemiTris 本地仓库
    |
    | 新建 feature 分支
    | commit
    | push origin feature
    v
SemiTris/demo:feature
    |
    | Pull Request
    v
rainkoli 审核
    |
    | merge
    v
rainkoli/demo:master
```

如果手动合并：

```text
SemiTris/demo:feature
    |
    | fetch / pull
    v
rainkoli 本地 master
    |
    | push origin master
    v
rainkoli/demo:master
```

---

# 最后建议记住的 8 句话

1. **`user.name / user.email` 是 commit 署名，不是远程登录凭据。**
2. **Token / SSH Key 才决定你有没有权限 push。**
3. **`origin` 只是一个远程仓库别名。**
4. **`master` 是本地分支，`origin/master` 是远程跟踪分支。**
5. **`git clone` 会自动配置 `origin`。**
6. **`git pull` 通常 = `fetch + merge/rebase`，而 `fetch` 更安全。**
7. **PR 的本质是把另一条 Git 历史中的 commits 合并到目标分支。**
8. **Git 不要求仓库必须有 Fork 关系；任何 Git 仓库理论上都可以 fetch 另一个 Git 仓库。**

---

# 推荐排错命令清单

```bash
# 查看当前状态
git status

# 查看 remote
git remote -v

# 查看本地分支
git branch

# 查看远程分支
git branch -r

# 查看全部分支
git branch -a

# 查看 tracking 关系
git branch -vv

# 查看 Git 配置
git config --list

# 查看配置来源
git config --show-origin --list

# 查看当前 commit 作者信息
git config user.name
git config user.email

# 查看最近一次 commit 作者/提交者
git log -1 --pretty=fuller

# 清除某站点 HTTPS 凭据
git credential reject
# 然后输入：
# protocol=https
# host=gitee.com
# 最后输入一个空行

# 从远程获取但不合并
git fetch origin

# 创建本地分支并跟踪远程分支
git switch -c <branch> --track origin/<branch>
```
