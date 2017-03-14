---
layout: post
title:  "React Loadable简介[译]"
date:   2017-03-14 12:01:53 +0800
categories: jekyll update
---
> * 原文地址：[Introducing React Loadable](https://medium.com/@thejameskyle/react-loadable-2674c59de178)
* 作者：james kyle
* 翻译人：李绍懿

当你的项目足够大时，把所有代码打包到一个bundle中的启动时间就会成为问题。这时就需要把app拆分为若干个bundle，然后根据需求动态加载它们。

![图片](/assets/image1.png)

Jekyll also offers powerful support for code snippets:

{% highlight ruby %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: http://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
