---
layout: post
title:  "I’m still human and still make mistakes"
date:   2016-02-16 18:56:22
categories: blog
---

### I’m still human and still make mistakes
在我刚进这家公司的时候，我现在这个项目就已经在失控的边缘了，刚接手就修了两个星期的bug，代码的质量也是可想而知，重构势在必行，但是业务驱动又比较紧，需求又不能停，每次都是夹着需求上线把逻辑牵扯相关的部分做些重构，又遇到一个困境，项目里遗留的测试也写得很不到位，测试都是集成测试，相当于是自动的冒烟测试，每次逻辑改动需要维护多个个测试用例，有些逻辑稍稍改动一下测试用例就挂了几十个，没办法维护，就把测试舍弃了，当时为了快速迭代，没有重视，没有去重建测试，想着测试可以后补，功能先上线，经过QA至少能保证功能正常，然后我们就走错了棋，违背了重构的第一原则。

后面项目在一段时期就像着了火的飞机，漏了水的轮船，每次重构没有测试保证，本意是为了更好的维护性却引入了新的bug。虽然一再强调测试很重要，但是别人说是一回事，自己感受又是另一回事了，也算是去年的经验成长吧。

最近重读《refactoring》ruby版，也是另有感受。

>Whenever I do refactoring, the first step is always the same. I need to build a solid set of tests for that section of code.The tests are essential because even though I follow refactorings structured to avoid most of the opportunities for introducing bugs, I’m still human and still make mistakes. Thus I need solid tests.

书里也是再三强调测试，重构的关键是测试，每做一次小的修改，就测试一次，步步为营，小心谨慎。

>Refactoring changes the programs in small steps. If you make a mistake, it is easy to find the bug.

>Whenever I make a change like this, I test.

>The key to testing is running tests after every small change so when you mess up you don’t have to look in many places to find the problem. Comparing the failing version of code to a previous working version (which I call Diff Debugging) is a useful technique, particularly so when the diffs are small. Because each change is so small, any errors are easy to find. You don’t spend a long time debugging, even if you are as careless as I am.

看到 Martin Fowler 的 I’m still human and still make mistakes，真是感触颇深。

《黑客与画家》有一小节叫“安全带还是手铐”，讲述语言设计者之间的最大分歧，有些人认为编程语言应该防止程序员干蠢事，另一些人则认为程序员应该可以用编程语言干一切他们想干的事情。受到paul graham的影响，最开始我是自由派，这也是后面选择ruby的原因之一，不过现在我算是不那么坚持这种观点了吧。

一开始当然是理想的，lisp的宏，ruby的动态元编程，真是引人入胜，美轮美奂，在刚逃离java“名词王国”地狱的时候，我是坚定的认为编程语言应该是这样的，不应该在语言上就形成壁垒，使得开发者受到限制，再之java这些没有类型推导的语言，还要添加烦复的类型标注，真是恶心。

在工作快两年了，现在也不是那么理想了，I’m still human and still make mistakes，在实际项目工程中，正确性才是第一优先的，静态类型检查确实很有用，回想起来我们有些bug也是可以通过静态类型检查避免的。

[想谈谈我们如何用 Go 取代 Ruby 重写了我们的 Qor](https://ruby-china.org/topics/27089)在和the plant交流的时候，他们的给出的解释是其中之一就是静态类型检查，Go经过编译部署之后，可以规避很多bug。

同时说下新出的语言`Kotlin`，作为一门新的jvm语言，比起java的一大亮点就是null safety，在java里面

    String a  = null;
    System.out.println(a.length());

这段代码能通过编译，但是会在运行时报错，而且在java的你是不知道一个变量是否会为空的，所以要做空检查。

    val a:String = null

在`Kotlin`中类型明确分开了 可空类型、非空类型，上面这段代码就通不过编译，你需要显示在声明后面加一个 ? 号，表示这个是可空类型，

    val a: String? = null
    println(a.length())

上面这段代码还是不能编译通过 `println(a.length())`这个地方编译器会要求你考虑为空的情况。

另外昨天一个bug，同事在一个model里改动了`validate`，看测试通过就发布了，结果引入了bug，测试通过主要是因为我写的时候少加一个断言，漏测一种情况，因为测试是后补的，不是很全面，让我想到那么多人推崇TDD测试驱动，测试先行还是有道理的，测试本身就要先 self-check，测试先行测试的质量肯定会高一些。
