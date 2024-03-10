# 如何发布到 jquery plugin 官方网站上

## 1.7\. 如何发布到 jquery plugin 官方网站上

jquery 插件标准化

1 月 16 日，jQuery Foundation 发布了新版插件资源库，以期能够为 jQuery 核心代码库的第三方开发带来更好的支持与促进。

自从一年多以前，早先的 jQuery 插件站点关闭以来，jQuery Foundation 团队就在着手搭建一个能够更智能地抵御垃圾的插件系统。作为 jQuery Foundation 的秘书长，Scott Gonzalez 同时也是新站点在 GitHub 上最大的贡献者。他说到，这个新站点“将通过某个大多数垃圾制造者都不会关注的提交过程 —— 修订控制系统，来减少垃圾的数量。”利用 GitHub 钩子（Hooks），第三方 jQuery 插件的开发者将获得前所未有的丰富工具集。

“托管在 GitHub 或者 Bitbucket 这样的平台上的一大好处是，作为用户，你可以直接获得一系列功能。比方说：你可以联系作者，你可以看到代码是否还在继续维护，你也可以检查 bug 报告或提交 bug，甚至可以提交 bug 的补丁。这在之前的站点上多数都做不到。我们认为，促使用户使用能够免费提供这些功能的服务，并且在已有大量用户每天都在使用的环境中工作，是一种巨大的进步。” Gonzalez 说到。

要发布你的 jQuery 插件，你需要利用 Post-receive 钩子，以及一个 Package 清单（manifest）文件。自动化的流程正在创建中。“David Radcliffe 已经提交了一个 Pull request，为站点新加入了一个 Service 钩子，使得用户不必再手动填写钩子的 URL。我们也计划创建一个能够自动生成清单文件的 Grunt 任务。” Gonzalez 说到。

随着 jQuery 2.0 的即将到来，现有插件的作者们需要将他们的插件重新发布到新的平台。Gonzalez 和其他 jQuery Foundation 的成员希望为整个社区的积极参与搭建好舞台。“新插件站点的一大亮点是其 100%开源，因此整个社区可以建议新特性、讨论特性的优缺点，乃至开发实现新特性。我们非常乐于看到能够为我们的用户提供更好的服务，并把与我们的代码项目同等程度的透明性以及开发性带给我们的这个站点项目，以实现更快的迭代。”

查看英文原文：jQuery's Github-Driven Plugin Repository Launched 上面是 infoq 的文章

* * *

详细步骤：[`plugins.jquery.com/docs/publish/`](http://plugins.jquery.com/docs/publish/)

* * *

我总结一下：

1、Add a Post-Receive Hook 这步必须做

学习一下[`help.github.com/articles/post-receive-hooks 不错哦`](https://help.github.com/articles/post-receive-hooks%E4%B8%8D%E9%94%99%E5%93%A6)

[`plugins.jquery.com/docs/publish/`](http://plugins.jquery.com/docs/publish/)

Settings -> Service Hooks -> WebHook URLs

[`plugins.jquery.com/postreceive-hook`](http://plugins.jquery.com/postreceive-hook)

2、＊jquery.json 可以用 grunt 生成

grunt init:jquery

3、发布的 tag 必须和＊jquery.json 里的版本一样，可以用 v 开头

the tag should be either "0.1.1" or "v0.1.1"

建议说不用 git tag -f 来覆盖老版本的 tag