系统管理 101：补丁管理
============================================================ 

就在之前几篇文章，我开始了“系统管理 101”系列文章，用来记录现今许多初级系统管理员，DevOps 工程师或者“全栈”开发者可能不曾接触过的一些系统管理方面的基本知识。按照我原本的设想，该系列文章已经是完结了的。然而后来 WannaCry 恶意软件出现并在补丁管理不善的 Windows 主机网络间爆发。我能想象到那些仍然深陷 2000 年代 Linux 与 Windows 争论的读者听到这个消息可能已经面露优越的微笑。

我之所以这么快就决定再次继续“系统管理 101”文章系列，是因为我意识到在补丁管理方面一些 Linux 系统管理员和 Windows 系统管理员没有差别。实话说，在一些方面甚至做的更差（特别是以运行时间为豪）。所以，这篇文章会涉及 Linux 下补丁管理的基础概念，包括良好的补丁管理该是怎样的，你可能会用到的一些相关工具，以及整个补丁安装过程是如何进行的。

### 什么是补丁管理？

我所说的补丁管理，是指你部署用于升级服务器上软件的系统，不仅仅是把软件更新到最新最好的前沿版本。即使是像 Debian 这样为了“稳定性”持续保持某一特定版本软件的保守派发行版，也会时常发布升级补丁用于修补错误和安全漏洞。

当然，因为开发者对最新最好版本的需求，你需要派生软件源码并做出修改，或者因为你喜欢给自己额外的工作量，你的组织可能会决定自己维护特定软件的版本，这时你就会遇到问题。理想情况下，你应该已经配置好你的系统，让它在自动构建和打包定制版本软件时使用其它软件所用的同一套持续集成系统。然而，许多系统管理员仍旧在自己的本地主机上按照维基上的文档（但愿是最新的文档）使用过时的方法打包软件。不论使用哪种方法，你都需要明确你所使用的版本有没有安全缺陷，如果有，那必须确保新补丁安装到你定制版本的软件上了。

### 良好的补丁管理是怎样的

补丁管理首先要做的是检查软件的升级。首先，对于核心软件，你应该订阅相应 Linux 发行版的安全邮件列表，这样才能第一时间得知软件的安全升级情况。如果你使用的软件有些不是来自发行版的仓库，那么你也必须设法跟踪它们的安全更新。一旦接收到新的安全通知，你必须查阅通知细节，以此明确安全漏洞的严重程度，确定你的系统是否受影响，以及安全补丁的紧急性。

一些组织仍在使用手动方式管理补丁。在这种方式下，当出现一个安全补丁，系统管理员就要凭借记忆，登录到各个服务器上进行检查。在确定了哪些服务器需要升级后，再使用服务器内建的包管理工具从发行版仓库升级这些软件。最后以相同的方式升级剩余的所有服务器。

手动管理补丁的方式存在很多问题。首先，这么做会使补丁安装成为一个苦力活，安装补丁需要越多人力成本，系统管理员就越可能推迟甚至完全忽略它。其次，手动管理方式依赖系统管理员凭借记忆去跟踪他或她所负责的服务器的升级情况。这非常容易导致有些服务器被遗漏而未能及时升级。

补丁管理越快速简便，你就越可能把它做好。你应该构建一个系统，用来快速查询哪些服务器运行着特定的软件，以及这些软件的版本号，而且它最好还能够推送各种升级补丁。就个人而言，我倾向于使用 MCollective 这样的编排工具来完成这个任务，但是红帽提供的 Satellite 以及 Canonical 提供的 Landscape 也可以让你在统一的管理接口查看服务器上软件的版本信息，并且安装补丁。

补丁安装还应该具有容错能力。你应该具备在不下线的情况下为服务安装补丁的能力。这同样适用于需要重启系统的内核补丁。我采用的方法是把我的服务器划分为不同的高可用组，lb1，app1,rabbitmq1 和 db1 在一个组，而lb2，app2,rabbitmq2 和 db2 在另一个组。这样，我就能一次升级一个组，而无须下线服务。

所以，多快才能算快呢？对于少数没有附带服务的软件，你的系统最快应该能够在几分钟到一小时内安装好补丁（例如 bash 的 ShellShock 漏洞）。对于像 OpenSSL 这样需要重启服务的软件，以容错的方式安装补丁并重启服务的过程可能会花费稍多的时间，但这就是编排工具派上用场的时候。我在最近的关于 MCollective 的文章中（查看 2016 年 12 月和 2017 年 1 月的工单）给了几个使用 MCollective 实现补丁管理的例子。你最好能够部署一个系统，以具备容错性的自动化方式简化补丁安装和服务重启的过程。

如果补丁要求重启系统，像内核补丁，那它会花费更多的时间。再次强调，自动化和编排工具能够让这个过程比你想象的还要快。我能够在一到两个小时内在生产环境中以容错方式升级并重启服务器，如果重启之间无须等待集群同步备份，这个过程还能更快。

不幸的是，许多系统管理员仍坚信过时的观点，把运行时间作为一种骄傲的象征——鉴于紧急内核补丁大约每年一次。对于我来说，这只能说明你没有认真对待系统的安全性。

很多组织仍然使用无法暂时下线的单点故障的服务器，也因为这个原因，它无法升级或者重启。如果你想让系统更加安全，你需要去除过时的包袱，搭建一个至少能在深夜维护时段重启的系统。

基本上，快速便捷的补丁管理也是一个成熟专业的系统管理团队所具备的标志。升级软件是所有系统管理员的必要工作之一，花费时间去让这个过程简洁快速，带来的好处远远不止是系统安全性。例如，它能帮助我们找到架构设计中的单点故障。另外，它还帮助鉴定出环境中过时的系统，给我们替换这些部分提供了动机。最后，当补丁管理做得足够好，它会节省系统管理员的时间，让他们把精力放在真正需要专业知识的地方。

______________________

Kyle Rankin 是高级安全与基础设施架构师，其著作包括： Linux Hardening in Hostile Networks，DevOps Troubleshooting 以及 The Official Ubuntu Server Book。同时，他还是 Linux Journal 的专栏作家。

--------------------------------------------------------------------------------

via: https://www.linuxjournal.com/content/sysadmin-101-patch-management

作者：[Kyle Rankin ][a]
译者：[haoqixu](https://github.com/haoqixu)
校对：[校对者ID](https://github.com/校对者ID)

本文由 [LCTT](https://github.com/LCTT/TranslateProject) 原创编译，[Linux中国](https://linux.cn/) 荣誉推出

[a]:https://www.linuxjournal.com/users/kyle-rankin
[1]:https://www.linuxjournal.com/tag/how-tos
[2]:https://www.linuxjournal.com/tag/servers
[3]:https://www.linuxjournal.com/tag/sysadmin
[4]:https://www.linuxjournal.com/users/kyle-rankin