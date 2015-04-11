---
layout: post
title:  "react-rails"
date:   2015-03-16 17:41:02
categories: react
---
react-rails
========

要使用 React 的jsx语法，就需要Transform来编译，在 Rails 里使用还需要在asset pipeline里面集成，facebook的开发者写了react-rails 这个 Gem 来提供这些功能，但是这个 Gem 的 1.0版本还在开发，对应 React 0.13的分支缺少一个功能，就是没有提供 jsx_transform_options 这个选项，React 0.13 最大的特性就是支持 ES6 语法，但是使用 ES6 语法需要在Transform编译时指定`jsx --harmony`。

这个功能在master分支上有，这个分支上没有

```ruby
config.react.jsx_transform_options = {
  harmony: true,
  strip_types: true, # for removing Flow type annotations
}
```
今天 React 发布了0.13.1，修复了一些小bug，react-rails也没能及时更新。

我提交了[pull request](https://github.com/reactjs/react-rails/pull/202)，不过看起来他们是不想浪费时间在这个分支上了，但是1.0不知道要等到什么时候才能release，所以现在先fork出来，使用自己的分支吧。可以参考[我的](https://github.com/zhangsoledad/react-rails)
