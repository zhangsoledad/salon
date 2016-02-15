---
layout: post
title:  "Ransack sucks真坑"
date:   2015-09-11 20:56:22
categories: rails
---

### Ransack Bug
项目里面用到`Ransack`这个Gem，可以帮你简化一些复杂的查询条件，用起来挺方便的，但近期升级会有个bug，导致严重的性能问题，要注意。
![bug](https://cloud.githubusercontent.com/assets/3198439/9806640/b0512278-5879-11e5-97ac-4974d25c2d89.png)

[ransack issue#553](https://github.com/activerecord-hackery/ransack/issues/553) 这个问题不止在sqlserver，项目用的是Mysql。

可以用猴子补丁来修复：

    #ransack 为了解决多个数据库查询，没有读取缓存的schema，带来严重性能问题
    module Ransack
      module Adapters
        module ActiveRecord
          class Context < ::Ransack::Context
            def type_for(attr)
              return nil unless attr && attr.valid?
              name         = attr.arel_attribute.name.to_s
              table        = attr.arel_attribute.relation.table_name
              schema_cache = @engine.connection.schema_cache
              unless schema_cache.table_exists?(table)
                raise "No table named #{table} exists."
              end
              schema_cache.columns_hash(table)[name].type
            end
          end
        end
      end
    end
