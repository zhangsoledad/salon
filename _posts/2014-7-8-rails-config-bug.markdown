---
layout: post
title:  "Bug in rails config"
date:   2014-7-8 18:55:03
categories: rails
---

Rails 3.2.18及之前的版本有个小bug
========
在rende json的时候有可能需要过滤掉html标签防止恶意脚本,需要设置

```ruby

config.active_support.escape_html_entities_in_json = true
```
这个设置在Rails 3.2.18及之前的版本是无效的

```ruby

Loading development environment (Rails 3.2.18)
>> Rails.application.config.active_support.escape_html_entities_in_json
=> true
>> ActiveSupport.escape_html_entities_in_json
=> false
>> ActiveSupport::JSON::Encoding.escape_html_entities_in_json
=> false
```

开发组开始还不打算修这个bug
![fix](http://7fvk4m.com1.z0.glb.clouddn.com/Fs3rxj6do--XnBzlmTk9dH1Ib_Hc)

最后他们还是默默的修了
![not fix](http://7fvk4m.com1.z0.glb.clouddn.com/Fsyp4gUIwCWsGZEVbvVXuy18OvLm)