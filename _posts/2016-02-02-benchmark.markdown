---
layout: post
title:  "一次Benchmark"
date:   2016-02-02 20:56:22
categories: blog
---

### 一次Benchmark
闲暇时间无聊和同事搞了一次`Benchmark`，没什么大的意义，就是玩玩，[脚本](https://github.com/souche-hero/ruby-daily/issues/4)在这里，用的是阿里云一台8核8G的机器，ruby用了`EventMachine`来实现并发IO，
这是同事写的node和go，我写了个`Phoenix`的，都是现学现写，go的代码应该是有问题，开始我都还不知道 `elixir` 怎么弄 `redis` 链接池，后面用了 `poolboy` + `redix`。

结果：

1. ruby
![ruby-500](https://cloud.githubusercontent.com/assets/3198439/13052924/b115c882-d43c-11e5-99f3-6f56ceda8f03.png)

2. node
![node-500](https://cloud.githubusercontent.com/assets/3198439/13052982/fbdf2eee-d43c-11e5-9cec-49b8d1766b0f.png)

3. phoenix
![phoenix-500](https://cloud.githubusercontent.com/assets/3198439/13053007/1f423caa-d43d-11e5-90d2-9163e010b3c7.png)

这里有更正经的测试[mroth/phoenix-showdown](https://github.com/mroth/phoenix-showdown)
