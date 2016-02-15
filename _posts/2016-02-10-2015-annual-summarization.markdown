---
layout: post
title:  "2015年终终结"
date:   2016-01-10 12:56:22
categories: rails
---

### 数据库

* 编码格式 UTF8mb4
采用 utf8 编码的 MySQL 无法保存占位是4个字节的 Emoji 表情。备注：MySQL 5.5.3 版本之后，引入了 utf8mb4 字符集。

* migration comment

* 并发场景创建唯一索引，Mysql不支持索引加条件，可采用增加一个字段作为索引的triger。

```sql
  select (null = null); \\null
  select (1 = 2); \\ 0
  select (1 = 1); \\ 1

  unique_trigger :boolean
  update(unique_trigger: true) 开启唯一索引
  update(unique_trigger: nil) 关闭唯一索引

```

* 需要索引的列如果是string，加上limit限制长度，mysql的索引列长度不能超过767bytes(utf8-255, utf8mb4-191)

* Default collation_ci 后缀是大小写不敏感的

* 预加载(preload,includes,eager)  

[3 ways to do eager loading ](http://blog.arkency.com/2013/12/rails4-preloading/)
   * preload 总是拆分sql
   * eager_load 总是合并sql（join）
   * includes 如果使用了where 并且where制定的列是来自于预载的表会代理给eager_load

```ruby
  User.includes(:addresses).where("addresses.country = ?", "Poland")
  User.eager_load(:addresses).where("addresses.country = ?", "Poland")

```

根据场景选择判断

```ruby
scope = Auction.includes(:auction_session, :shop, :bids, :users_auctions, :order, car: [:images, :configuration])

```
![](http://git.souche.com/cheniu/cheniu_auction/uploads/cd353cbf14975bd41e414fd6a8db0494/007AD190-680C-412F-8BB2-C3BDA7720A50.png)

上面这段代码，后面的查询条件和预载的表相关，生成的sql会`join`8张表，造成性能问题，这个时候应该使用preload

```ruby
scope = Auction.preload(:auction_session, :shop, :bids, :users_auctions, :order, car: [:images, :configuration])

```

<High Performance MySQL> Chapter 6, Ways to Restructure Queries, Join Decomposition

It looks wasteful at first glance, because you’ve increased the number of queries without getting anything in return. However, such restructuring can actually give significant performance advantages:

* 简单查询的缓存更加高效
* 降低锁竞争
* 减少扫描的行数

```ruby
  #be careful, association with argument scope can't preload
has_one :amount_record, ->(order){ where(auction_id: order.auction_id) }, foreign_key: :user_id, primary_key: :user_id
has_one :users_auction, ->(order){ where(auction_id: order.auction_id) }, foreign_key: :user_id, primary_key: :user_id

```

rails提供的预加载适用场景有限,更复杂的场景需要自己做hash匹配.

### 细节

* 和金钱相关的适用`money` Gem 或者 BigDecimel，不要使用 Float
* 外部接口时间最好已时间戳为格式
* to_d, to_datetime
* 方法参数尽量用关键字参数
* [Why You Should Never Rescue Exception in Ruby](http://daniel.fone.net.nz/blog/2013/05/28/why-you-should-never-rescue-exception-in-ruby/)

```ruby
  SystemStackError
NoMemoryError
  SecurityError
  ScriptError
    NotImplementedError
    LoadError
      Gem::LoadError
    SyntaxError
  SignalException
    Interrupt
  SystemExit
    Gem::SystemExitException

```

rescue => e 是 rescue StandardError => e 的缩写

把所有的rescue Exception => e 替换成 rescue => e

in Rails 3.2.13 , there are 375 StandardErrors defined. 最好是明确需要捕获的异常.


### vagrant

* vagrant是个非常好的东西,最近才发现(相见恨晚),现在公司的架构是前后端分离,但有时候还是需要前端介入一下,帮配环境的时候很蛋疼,有时候觉得这东西简直是玄学(正确的配置只有一种,单配错的情况各种花样,还有就是GFW),用虚拟机管理起来方便很多,先封装个BOX,其他人拷贝就能直接用了,非常方便。
* 和docker的区别,看这篇文章,[should-i-use-vagrant-or-docker-for-creating-an-isolated-environment
](http://stackoverflow.com/questions/16647069/should-i-use-vagrant-or-docker-for-creating-an-isolated-environment
),docker和vagrant的作者都各自做了解答,主要是应用场景不同。

### 并发查询

最近ruby-china迁移Postgresql,也去凑了下热闹,看他们代码的时候发现很多用Thread来并发IO的写法,李华顺开贴也说这样写效果不错,ruby-china原先用mongo,有几个查询是不能索引命中的，查询大概要100ms,如果不并发查询,那个请求要查询四次,大概就要400+ms.

记得上次一个做过node的同事在学rails的时候问我,rails的无顺序关系查询不能并发吗？当时确实把我问住了,平时不会这么去考虑的.但是这样写也是有问题的,也仅仅做参考吧.

* 是在大访问量的情况下,会申请很多线程,ruby的线程现在是native的,并不轻量,也受系统限制,比如在mac上跑很容易就出现线程限制.

* 这种写法太原始低级了,没法控制异常,ruby的并发是个蛋疼问题.

```ruby
Benchmark.ms {
  threads = []
  20.times do
    threads << Thread.new do
      ActiveRecord::Base.connection_pool.with_connection do |conn|
        conn.execute("select sleep(1)")
      end
    end
  end
  threads.each(&:join)
}

```


```ruby
Benchmark.ms {
  20.times do
    ActiveRecord::Base.connection_pool.with_connection do |conn|
      conn.execute("select sleep(1)")
    end
  end
}

```
