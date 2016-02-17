---
layout: post
title:  "GitHub Pages升级"
date:   2016-02-13 14:56:22
categories: blog
---

### GitHub Pages升级
GitHub Pages最近发布升级，升级到Jekyll 3.0，blog需要做些调整。

Two additional changes:

The Jekyll 3.0 upgrade will introduce two additional changes that may affect a small subset of users:

1. Jekyll no longer supports relative permalinks. This has been the default since Jekyll 2.0, and is only an issue if you explicitly added relative_permalinks: true to your site's configuration. Going forward, regardless of your site's configuration, if you add the permalink directive to a page's YAML front matter, the path should be relative to the site's root directory, not the page's parent.

2. Starting May 1st, 2016, GitHub Pages will no longer support Textile. If you are currently using Textile (Redcloth) to author your Jekyll site, you'll need to convert your site to use Markdown instead.

同时 GitHub Pages 在今年五一开始只支持`kramdown`作为Markdown的引擎，这个影响比较大，常用的代码块句法无法正确解析：

    ```ruby
    def hello
      puts 'Hello'
    end
    ```

会被解析成行内元素，而不是代码块：

ruby def hello puts 'Hello' end

`kramdown`也是可以支持上面句法的，需要把`input`选项设为`GFM`,在`_config.yml`里

    kramdown:
      input: GFM

诶，好像没用啊，因为现在版本有bug、，Jekyll从 `_config.yml`文件中读取的key使用的是`String`，而 `kramdown`的设置使用的是`Symbol`，[issue#4427](https://github.com/jekyll/jekyll/issues/4427)(好low的bug)，3.1.2发布会修复(都几个版本了喂)，在此之前，你可以自己写`plugins`

`_plugins/my_kramdown.rb`

    module Jekyll
      module Converters
        class Markdown
          class MyKramdown < KramdownParser
            def convert(content)
              Kramdown::Document.new(content, Utils.symbolize_hash_keys(@config)).to_html
            end
          end
        end
      end
    end

然后 `_config.yml`

    markdown: MyKramdown
