---
layout: post
title: "Ubuntu 10.04.3 kernel bug"
date: 2012-05-17 10:29
comments: true
categories: Ops
---

运行了200天的 Ubuntu 10.04.3 在没有任何征兆的情况下突然系统负载(Load)狂高，Java 进程完全不能启动，但是有些服务 MySQL / Redis 等却不受影响，因为害怕影响线上的服务，所以没有全部重启，看着系统负载从10一直飙升到500，后来干脆连 SSH 也进不去了。最后只能悲剧的重启，重启后一切恢复正常。

查看日志发现一些内核的报错

    May 16 10:07:16 w2 kernel: [ 8985.860328] Call Trace:
    May 16 10:07:16 w2 kernel: [ 8985.860339]  [<ffffffff8121b4ea>] ? jbd2_journal_dirty_metadata+0x10a/0x140
    May 16 10:07:16 w2 kernel: [ 8985.860345]  [<ffffffff8153fc5d>] schedule_timeout+0x22d/0x300
    May 16 10:07:16 w2 kernel: [ 8985.860350]  [<ffffffff8104b3c9>] ? sched_slice+0x59/0xa0
    May 16 10:07:16 w2 kernel: [ 8985.860355]  [<ffffffff8115dea3>] ? dup_fd+0x33/0x340
    May 16 10:07:16 w2 kernel: [ 8985.860360]  [<ffffffff812b43d3>] ? cpumask_next_and+0x23/0x40
    May 16 10:07:16 w2 kernel: [ 8985.860363]  [<ffffffff8153f87b>] wait_for_common+0xdb/0x180
    May 16 10:07:16 w2 kernel: [ 8985.860367]  [<ffffffff8105ddcb>] ? enqueue_task_fair+0x5b/0xa0
    May 16 10:07:16 w2 kernel: [ 8985.860371]  [<ffffffff8105ccb0>] ? default_wake_function+0x0/0x20
    May 16 10:07:16 w2 kernel: [ 8985.860374]  [<ffffffff8153f9dd>] wait_for_completion+0x1d/0x20
    May 16 10:07:16 w2 kernel: [ 8985.860378]  [<ffffffff81065870>] do_fork+0x150/0x440
    May 16 10:07:16 w2 kernel: [ 8985.860381]  [<ffffffff8115ff40>] ? mntput_no_expire+0x30/0x110
    May 16 10:07:16 w2 kernel: [ 8985.860387]  [<ffffffff8107e085>] ? set_one_prio+0x75/0xd0
    May 16 10:07:16 w2 kernel: [ 8985.860391]  [<ffffffff8101a085>] sys_vfork+0x25/0x30
    May 16 10:07:16 w2 kernel: [ 8985.860396]  [<ffffffff81012513>] stub_vfork+0x13/0x20
    May 16 10:07:16 w2 kernel: [ 8985.860399]  [<ffffffff810121b2>] ? system_call_fastpath+0x16/0x1b 

Google 了一下，发现果然是个悲剧，装服务器时候使用了 Ubuntu-10.04.3-amd64 版本，装出来的 Kernel 版本是 2.6.32-33-generic，至少应该是 2.6.32-33-server 啊，这两者的差别可以参考下边链接的描述，总之服务器上用上了 Desktop 的 Kernel 肯定是有问题的。

[http://jaseywang.me/2011/09/19/generic-kernel-%E5%92%8C-server-kernel/](http://jaseywang.me/2011/09/19/generic-kernel-%E5%92%8C-server-kernel/)

后来又发现就算是 2.6.32-33-server 也还是有 bug 的，所有的特征都符合

[http://jaseywang.me/2012/05/07/2-6-32-33-%E5%86%85%E6%A0%B8%E8%B7%91%E5%88%B0-208-%E5%A4%A9%E5%AE%95%E6%9C%BA](http://jaseywang.me/2012/05/07/2-6-32-33-%E5%86%85%E6%A0%B8%E8%B7%91%E5%88%B0-208-%E5%A4%A9%E5%AE%95%E6%9C%BA)

解决办法就是升级内核，对于 10.04，选择用最新的 Ubuntu-10.04.4-amd6 吧，顺便把内核升级到 2.6.32-41-server 吧，以前的某些服务器用的是这个内核一直很稳定。在安装服务器时候还是要小心谨慎!
