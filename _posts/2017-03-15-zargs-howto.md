---
layout: post
title: zsh-builtin xargs
categories: personal
tags: zsh xargs
comments: true
mathjax: null
featured: true
published: true
---

Of course all of you already knows about xargs: nice cli-mode tool to execute
command lines from standard input. As you know some tools can get input as
a parameter like awk or grep, but some like cp or 7z can't. Here I shall talk
about xargs and one of zsh-builtin replacement of it. If you are interested
please read further.

<!--excerpt-->

Also you cannot to bypass it directly by value as
{% highlight shell %}
rm $(find -name *.cpp) 
{% endhighlight %}
because of it's error prone and also can lead to "argument list too long" error.
It's the point where xargs appears, for example you can use something like
that:

{% highlight shell %}
find -name *.cpp -print | xargs rm -v
{% endhighlight %}

Also you can handle filenames which are not well
defined(multiline, tricky symbols, etc) with that:

{% highlight shell %}
find -name *.cpp -print0 | xargs -0n1 rm -v
{% endhighlight %}

Also xargs can be used as easy-to-use "replacement", as minimum in some
cases, to gnu parallel with -P flag use it like that:

{% highlight shell %}
echo {0..255} | xargs -n1 -P8
{% endhighlight %}

8 threads are used here.

All of these features are very nice. But you cannot bypass functions from zsh or another
shell to xargs, only commands. Also it's not perfect because of poor performance when
amount of input is large.

Instead zsh has nice zargs addon:

To use is you need to autoload:

{% highlight shell %}
autoload -U zargs
{% endhighlight %}

and then use it in way like that:

{% highlight shell %}
zargs **/*~ -- rm
{% endhighlight %}

to add arguments to zargs you need to use -- from both sides like here:

{% highlight shell %}
zargs -n1 -- ~/.*rc -- wc
{% endhighlight %}

To prove that it's better consider following task:

you need(for some reason) to count all usings of word "mutex" in the linux kernel headers:

{% highlight shell %}
[~/dev/src/1st_level/linux] >> time find . -name \*.h -exec grep mutex>/dev/null {} \; | wc -l
3255
[‒ noglob find . -name \*.h -exec grep mutex {} \; > /dev/null ‒] [0.17s] [user 1.00s] [system 4%] [cpu 27.352 total] [-||-] [Mem: 12 kb max]
[‒ wc -l ‒] [0.01s] [user 0.01s] [system 0%] [cpu 27.352 total] [-||-] [Mem: 12 kb max]
{% endhighlight %}

and for zargs you have:

{% highlight shell %}
[~/dev/src/1st_level/linux] >> time zargs ./**/*.h -- grep mutex>/dev/null|wc -l
3255
[‒ zargs ./**/*.h -- grep mutex > /dev/null ‒] [0.29s] [user 0.11s] [system 79%] [cpu 0.492 total] [-||-] [Mem: 10 kb max]
[‒ wc -l ‒] [0.00s] [user 0.00s] [system 0%] [cpu 0.492 total] [-||-] [Mem: 3 kb max]
{% endhighlight %}

As you can see zargs also works faster than find + xargs.

Note that if you are using fasd or stuff like that then true performance(time
which you need to wait) will be increased significantly and you will notice
that, but "time" util is not care about true performance.
