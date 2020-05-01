---
description: Eclipse基金会项目的IP/版权要求
---

# Eclipse贡献者

本页说明向eclipse/deeplearning4j github代码库中的项目贡献代码所需的步骤：[https://github.com/eclipse/deeplearning4j](https://GitHub.com/eclipse/deeplearning4j)

贡献者（任何想将代码提交到存储库的人）在合并他们的代码之前需要做两件事：

1. 签署Eclipse贡献者协议（一次）
2. 签署提交（每次）

## 为什么这是必需的？

所有Eclipse基础项目都必须满足这两个要求，而不仅仅是DL4J和ND4J: [https://projects.eclipse.org/](https://projects.eclipse.org/)

通过签署ECA，你实际上是在明确你提交的代码是你编写的，或者你有权为项目做出贡献。这是避免版权问题的必要法律保护。

 通过签署你的提交，你可以明确该特定提交中的代码是你自己的。

## 签署Eclipse贡献者协议

你只需要签署一次Eclipse贡献者协议（ECA）。流程如下：

**步骤1：注册Eclipse帐户**

可以在这里完成 [https://accounts.eclipse.org/user/register](https://accounts.eclipse.org/user/register)

注意：你必须使用与你的github帐户（要提交请求的GitHub帐户）相同的电子邮件进行注册。

**步骤2：签署ECA**

去 [https://accounts.eclipse.org/user/eca](https://accounts.eclipse.org/user/eca) 并按说明操作。

## 签署你的提交

### 签署一个新的提交

有几种方法可以签署提交。请注意，你可以使用这些选项中的任何一个。

**选项1：在命令行提交时使用 -s**

在这里签署提交很简单：

```text
git commit -s -m "My signed commit"
```

注意，-s（小写s）-大写S（即，-S）用于GPG签名（见下文）。

**选项2：设置Bash别名（“或Windows cmd别名”）用于自动签署**

例如，可以在bash中设置以下别名：

```text
alias gcm='git commit -s -m'
```

然后提交将使用以下操作完成：

```text
gcm "My Commit"
```

对于Windows命令行，可以通过一些机制使用类似的选项（请参见[此处](https://stackoverflow.com/questions/20530996/aliases-in-windows-command-prompt)）

一种简单的方法是创建包含以下内容的gcm.bat文件，并将其添加到系统路径中：

```text
@echo off
echo.
git commit -s -m %*
```

然后，你可以使用与上面相同的过程提交（即gcm“My commit”）

**选项3: 使用GPG签署** 

要获取GPG签署详情，查看[此链接](https://harryrschwartz.com/2014/11/01/automatically-signing-your-git-commits) 

 注意，这个选项可以与别名组合使用（见上文），如alias gcm=-git commit-S-m'-注意GPG签署的大写字母-S。

**选项4: 使用带自动签署的IntelliJ提交**

可用于执行git提交，包括通过签署提交。有关详细信息，请参阅[本页](https://www.jetbrains.com/help/idea/commit-and-push-changes.html?section=Windows%20or%20Linux)。

### 检查提交是否已签署

 在执行提交之后，你可以使用几种不同的方法进行签入。一种方法是使用git log--show signature-1来显示最后一次提交的签署（例如，使用-5来显示最后5次提交）

输出如下：

```text
$ git log --show-signature -2
commit 81681455918371e29da1490d3f0ca3deecaf0490 (HEAD -> commit_test_branch)
Author: YourName <you@email.com>
Date:   Fri Jun 21 22:27:50 2019 +1000

    This commit is unsigned

commit 2349c6aa3497bd65866d7d0a18fe82bb691bb868
Author: YourName <you@email.com>
Date:   Fri Jun 21 21:42:38 2019 +1000

    My signed commit

    Signed-off-by: YourName <you@email.com>
```

顶部提交是未签署的，而底部提交是已签署的（请注意存在已签名者）。

### 如果你忘了签署一个提交-修改最后一个提交

如果忘记签署上次提交，可以使用以下命令：

```text
git commit --amend --signoff
```

### 如果你忘记了签署多次提交

假设你的分支有3个新提交，所有提交都是未签署的：

```text
$ git log -4 --oneline
4b164026 (HEAD -> commit_test_branch) Your new commit 3
d7799615 Your new commit 2
6bb6113a Your new commit 1
ef09606c This commit already exists
```

一个简单的方法是压缩并签署这些提交。要在最后3次提交时执行此操作，请使用以下命令：（注意，可能需要先进行备份）

```text
git reset --soft HEAD~3
git commit -s -m "Squashed and signed"
```

结果:

```text
$ git log -2 --oneline
31658e11 (HEAD -> commit_test_branch) Squashed and signed
ef09606c This commit already exists
```

你可以使用如前所示显示`git log -1 --show-signature`来确认提交已签署。 

请注意，一旦将提交合并为master，你的提交将被粉碎，因此丢失提交历史并不重要。

如果你正在更新现有的PR，则可能需要强制使用 -f（如`git push X -f`）。

