---
layout: post
title:  "ActiveJob使用心得"
date:   2015-12-11 12:56:22
categories: rails
---

### ActiveJob使用心得
最近把项目升级到了4.2，也顺便用 `ActiveJob` 把 `sidekiq` 的接口全部包装了一遍。

我自己感觉主要是两点：

1.sidekiq 要求参数要small and simple，而且 Don't save state to Sidekiq, save simple identifiers，因为 sidekiq 的参数要序列化到 redis 里面，所以只能是能转化成JSON datatypes的类型: string, integer, float, boolean, null, array and hash，而 ActiveJob 实现了一套 GlobalID 机制，让你可以直接用 AR 对象做参数，不用自做反序列化：


    class TrashableCleanupJob
      include Sidekiq::Worker
      def perform(trashable_class, trashable_id, depth)
        trashable = trashable_class.constantize.find(trashable_id)
        trashable.cleanup(depth)
      end
    end

可以写成：

    class TrashableCleanupJob < ActiveJob::Base
      def perform(trashable, depth)
        trashable.cleanup(depth)
      end
    end

2.还是因为参数要序列化的问题，sidekiq 如果用 Symbol 传参数的话，在perform方法里面经过反序列化，会变成 String，算得上是个坑，ActiveJob直接就不让你传 Symbol 参数，会直接报错，让你规避这个问题，还有就是 sidekiq 是不支持ruby `keyword argument` 的传参方式的。


但是 `ActiveJob` 也有让人蛋疼的点，`ActiveJob` 作为一个抽象层次更高的统一包装接口，就不能支持一些特定后端的特定选项，比如：

    sidekiq_options retry: 2
    sidekiq_retry_in { |count, _| 3 * count }

但是这些功能都是实际要用到的，怎么办？在ruby的世界里，调教第三方代码永远是so easy，一个猴子补丁

    module ActiveJob
      module QueueAdapters
        class SidekiqAdapter
          def enqueue(job)
            JobWrapper.sidekiq_options job.sidekiq_options_hash if job.sidekiq_options_hash
            JobWrapper.sidekiq_retry_in job.sidekiq_retry_in_block if job.sidekiq_retry_in_block
            Sidekiq::Client.push(
              'class' => JobWrapper,
              'wrapped' => job.class.to_s,
              'queue' => job.queue_name,
              'args'  => [ job.serialize ]
            )
          end

          def enqueue_at(job, timestamp)
            JobWrapper.sidekiq_options job.sidekiq_options_hash if job.sidekiq_options_hash
            JobWrapper.sidekiq_retry_in job.sidekiq_retry_in_block if job.sidekiq_retry_in_block
            Sidekiq::Client.push(
              'class' => JobWrapper,
              'wrapped' => job.class.to_s,
              'queue' => job.queue_name,
              'args'  => [ job.serialize ],
              'at'    => timestamp
            )
          end
        end
      end

      class Base
        class_attribute :sidekiq_options_hash
        class_attribute :sidekiq_retry_in_block

        def self.sidekiq_options(opts={})
          self.sidekiq_options_hash = opts
        end

        def self.sidekiq_retry_in(&block)
          self.sidekiq_retry_in_block = block
        end
      end
    end

就能直接这样写

    class BaseJob < ActiveJob::Base
      sidekiq_options retry: 2
      sidekiq_retry_in { |count, _| 3 * count }

      def perform; end
    end
