---
layout: post
title:  "Some problem in callback"
date:   2014-10-27 21:03:51
categories: rails
---

Rails的回调方法callback的小问题
========

我们来个小例子:

```ruby

ActiveRecord::Schema.define(:version => 20141027143100) do

  create_table "posts", :force => true do |t|
  end

  create_table "user_posts", :force => true do |t|
    t.integer  "user_id"
    t.integer  "post_id"
  end

  create_table "users", :force => true do |t|
  end
end

class User < ActiveRecord::Base
  has_many :user_posts
  has_many :posts, :through => :user_posts
  after_create :add_posts

  def add_posts
    puts 'start'
    posts<<Post.first
    puts 'end'
  end
end

class UserPost < ActiveRecord::Base
  belongs_to :user
  belongs_to :post
end

class Post < ActiveRecord::Base
end
```

在rails c里:

```ruby

Post.create
User.create
```
得到

```

2.1.5 :001 > Post.create
   (0.2ms)  BEGIN
  SQL (0.4ms)  INSERT INTO `posts` (`created_at`, `updated_at`) VALUES ('2015-04-06 15:10:31', '2015-04-06 15:10:31')
Post - after create
   (47.0ms)  COMMIT
 => #<Post id: 2, created_at: "2015-04-06 15:10:31", updated_at: "2015-04-06 15:10:31">
2.1.5 :002 > User.create
   (0.2ms)  BEGIN
  SQL (0.4ms)  INSERT INTO `users` (`created_at`, `updated_at`) VALUES ('2015-04-06 15:10:33', '2015-04-06 15:10:33')
start
  Post Load (0.5ms)  SELECT `posts`.* FROM `posts` LIMIT 1
  SQL (0.4ms)  INSERT INTO `user_posts` (`created_at`, `post_id`, `updated_at`, `user_id`) VALUES ('2015-04-06 15:10:33', 1, '2015-04-06 15:10:33', 4)
end
   (26.4ms)  COMMIT
 => #<User id: 4, created_at: "2015-04-06 15:10:33", updated_at: "2015-04-06 15:10:33">
```

都很正常,我们将上面User里的代码调整一下，把after_create放到has_many前面:

```ruby

after_create :add_posts
has_many :user_posts
has_many :posts, :through => :user_posts
```

reload!然后再User.create，结果：

```ruby

Reloading...
 => true
2.1.5 :005 > User.create
   (0.1ms)  BEGIN
  SQL (0.4ms)  INSERT INTO `users` (`created_at`, `updated_at`) VALUES ('2015-04-06 15:18:04', '2015-04-06 15:18:04')
start
  Post Load (0.3ms)  SELECT `posts`.* FROM `posts` LIMIT 1
  SQL (0.4ms)  INSERT INTO `user_posts` (`created_at`, `post_id`, `updated_at`, `user_id`) VALUES ('2015-04-06 15:18:04', 1, '2015-04-06 15:18:04', 5)
end
  SQL (7.7ms)  INSERT INTO `user_posts` (`created_at`, `post_id`, `updated_at`, `user_id`) VALUES ('2015-04-06 15:18:04', 1, '2015-04-06 15:18:04', 5)
   (68.1ms)  COMMIT
 => #<User id: 5, created_at: "2015-04-06 15:18:04", updated_at: "2015-04-06 15:18:04">

```
**这时发生了诡异的行为，关联表user_posts插入了两条重复的数据**。

这个原因是`:autosave`这个选项，`:autosave`为`true`时`associated objects`也会被save，对新记录这个选项默认为`true`，这个选项是通过`after_create`实现的，`after_create`会按照声明的顺序来调用，就产生了这个问题。

**在使用中要多注意callback的顺序**，在rails文档中已指出，callback不要放到associations前面定义，如果使用rails 4的`ActiveSupport::Concern`也要注意`include`的顺序如下：

```ruby

module AutoIncrement
  extend ActiveSupport::Concern

  included do
    after_create :add_posts
  end

  def add_posts
    puts 'start'
    self.posts << Post.first
    puts 'end'
  end
end

class User < ActiveRecord::Base
  has_many :user_posts
  has_many :posts, :through => :user_posts
  include AutoIncrement
end

```

rails开发组关于这点也在征求好的改进意见，[issue#16823](https://github.com/rails/rails/issues/16823)。
