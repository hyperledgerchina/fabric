# 使用Gerrit
依照下面的步骤，通过Gerrit审阅系统，你可以与其它开发者对Hyperledger Fabric项目进行协作开发。

请确保你订阅了[邮件列表](http://lists.hyperledger.org/mailman/listinfo/hyperledger-fabric)，如果有任何问题，你也可以通过 [Slack](https://hyperledgerproject.slack.com/)寻求帮助。

Gerrit 为用户分配了以下角:

* **提交者**: 可提交修改变更，审查其他代码的变更，并分别投以+1或-1，作出接受或拒绝的建议。
* **维护者**: 基于审阅的反馈， 投以+2或-2来批准或拒绝变更。
* **构建者**: (如Jenkins)使用构建自动化工具来验证更改。

维护者应熟悉[审阅流程](reviewing.md)。也欢迎鼓励任何人来审阅变更，以此来发现该变更文件的价值。

## Git审阅

Gerrit有一个**非常**有用的工具叫[Git审阅](https://www.mediawiki.org/wiki/Gerrit/git-review)。
这个命令行工具可以为你自动完成大部分的工作，建议阅读下面的内容，这样可以让你明白这些工作背后的工作原理。


## 示例项目

我们创建了一个[示例项目](https://gerrit.hyperledger.org/r/#/admin/projects/lf-sandbox)来让开发者熟悉Gerrit和我们的工作流程，你可以通过下面的命令和工具来练习。

## 深入了解Gerrit

全面介绍Gerrit不在此文档的范围之内，互联网上有许多相关的资源，你可以在[这里](https://www.mediawiki.org/wiki/Gerrit/Tutorial)找到一个很好的简介，我们提供的一系列[最佳实践](best-practices.md)可能也会对你有帮助。

##Gerrit存储库的本地克隆

当你要实现新功能或者修复bug时。

1. 打开 Gerrit中的 [项目页面](https://gerrit.hyperledger.org/r/#/admin/projects/)

2. 选择你要变更的项目。

3. 打开一个命令终端，通过`Clone with git hook` URL将项目克隆到本地. 确保ssh也被选中，因为这将使认证更为简单。
```
git clone ssh://LFID@gerrit.hyperledger.org:29418/fabric && scp -p -P 29418 LFID@gerrit.hyperledger.org:hooks/commit-msg fabric/.git/hooks/
```

**注意:** 当你克隆fabric项目的时候，最好将其克隆到`$GOPATH/src/github.com/hyperledger` 目录下，这样你可以按照[文档](../dev-setup/devenv.md)的步骤通过Vagrant来使用它.

4. 为你的本地克隆创建一个具有描述性名称的分支

```
cd fabric
git checkout -b issue-nnnn
```

5. 提交你的代码，如何创建一个已经过深度讨论的高效提交格式，请仔细阅读[文档](changes.md)。
```
git commit -s -a
```
然后输入精确和可读性高的描述并提交。

6. 当代码的变更影响文档和测试案例，那么文档和测试案例的变更需要同代码一起提交，这样确保一旦代码变更被回滚，相应的文档和测试案例也被相应的回滚。

##提交变更

目前，只支持通过Gerrit来为项目提交变更给审阅者审阅，**请按照[指南](changes.md)的格式来修改并提交变更**

### 使用git审阅

**注意:**如果你喜欢，你可以使用 [git审阅](#Git审阅)来代替以下的例子。

复制以下的设置到 `.git/config`，将`<USERNAME>`替换成你的Gerrit帐号。

```
[remote "gerrit"]
    url = ssh://<USERNAME>@gerrit.hyperledger.org:29418/fabric.git
    fetch = +refs/heads/*:refs/remotes/gerrit/*
```

然后通过命令`git review`来提交你的变更。

```
$ cd <your code dir>
$ git review
```
当你更新了你的补丁，你可以通过`git commit --amend`命令再次提交，然后重复 `git review`命令。


### 不使用git审阅

 可以根据[说明](../dev-setup/build.md)来编译构建源码。

当变更可以提交时，要求将其推送到特定的分支上，其分支名包含了变更最终应该提交的目标分支信息

 对Hyperledger Fabric项目来说，这个特定的分支名叫 `refs/for/master`。

当要将本地的开发分支推送到Gerrit服务器时，在克隆项目的根目录下，打开一个终端并执行以下命令。

```
cd <your clone dir>
git push origin HEAD:refs/for/master
```
如果你的命令执行正确，将会有类似以下的输出。

```
Counting objects: 3, done.
Writing objects: 100% (3/3), 306 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
remote: Processing changes: new: 1, refs: 1, done
remote:
remote: New Changes:
remote:   https://gerrit.hyperledger.org/r/6 Test commit
remote:
To ssh://LFID@gerrit.hyperledger.org:29418/fabric
* [new branch]      HEAD -> refs/for/master
```
Gerrit将为你的推送创建一个链接，可以通过这个链接来查看此次推送的具体变更。

## 增加审阅者

你也可以为你的变更增加审阅查。

你可以通过指令`%r=reviewer@project.org`来为你的提交增加审阅者。  

比如：

```
git push origin HEAD:refs/for/master%r=rev1@email.com,r=rev2@notemail.com
```

另外，如果你所有提交的审阅者都是相同的,还可以通过git配置添加一组审阅者。

要添加默认审阅者，打开文件`.git/config`，在`[ branch “master” ]`下添加以下配置：

```
[branch "master"] #.... push =
HEAD:refs/for/master%r=rev1@email.com,r=rev2@notemail.com`
```

将 `@email.com` 和 `@notemail.com`换成审阅者的邮箱地址，别忘了将`origin`替换成你的远程名（基本不用变更）

##使用Gerrit审阅

* **添加（Add）**: 提交者可以通过这个按键来为自己的变更添加审阅者，开始输入一个审阅者名称时，系统将根据系统中已注册的用户信息自动为你补全用户名,系统将会通过邮件通知这些审阅者。

* **丢弃（Abandon）**: 这个按键只有提交者可以使用，它允许提交者丢弃变更，并将变更从合并队列中删除。

* **Change-ID**: 这个ID是Gerrit创建的。如果在审阅过程中发现你的提交需要作出修改时，它就变得特别有用。您可以提交的新版本; 如果提交的Change-ID提交，Gerrit会记住它并将其显示为相同的变更的另一个版本。

* **状态（Status）**:现在这个变更示例就是在审阅状态，在左上角有需要验证（Need Verified）的标志，列表中的审阅者都会发表他们的建议，如果同意合并就投+1，如果不同意的就投-1，而Gerrit的维护者可以分别投以+2或-2来拒绝或接受合并。

系统将会发送通知到提交信息中`Signed-off-by`行中的邮箱地址里。可以通过访问[Gerrit 面板](https://gerrit.hyperledger.org/r/#/dashboard/self)来查看请求的进度。

在Gerrit底部的历史标签下会显示所有审阅者的意见。

## 查看正在进行的变更

通过点击左上角 `All --> Changes` 或者[链接](https://gerrit.hyperledger.org/r/#/q/project:fabric)来查看所有正在进行的变更。

如果您在多个项目中进行协作开发，您可能希望通过左上角的搜索栏搜索添加过滤条件使其只显示特定的分支。

添加过滤器*project:fabric*来使其只有Hyperledger Fabric的项目可视。

通过点击 `My --> Changes` or [链接](https://gerrit.hyperledger.org/r/#/dashboard/self)，来列出你提交的所有变更，或列出那些需要你参与的变更。