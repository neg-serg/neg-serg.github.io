---
layout: post
title: Cyclesort&#x3a; unusual O(n+m) sorting algorithm
categories: personal
tags: 
  - algorithm
  - sorting
comments: true
mathjax: null
featured: true
published: true
---

One day I've discovered some interesting algorithm: cyclesort. There is
interesting <a
href="http://comjnl.oxfordjournals.org/content/33/4/365.full.pdf">paper</a>
about it. And also nice explanation on  <a
href="https://corte.si/posts/code/cyclesort/index.html">https://corte.si/posts/code/cyclesort/index.html</a>.

I think that main idea is pretty clear: you can decompose any permutation
into a unique set of disjoint cycles and then apply them to get
lexicographical order.

Also look at this explaination:

{% highlight python %}
l=[0, 5, 6, 8, 7, 4, 9, 1, 3, 2]

cls=[], i=1 cycle=[]
cls=[], i=1 cycle=[]
cls=[], i=1 cycle=[1]
cls=[], i=1 cycle=[1, 5]
cls=[], i=1 cycle=[1, 5, 4]
cls=[], i=1 cycle=[1, 5, 4, 7]
cls=[[7, 4, 5, 1]], i=2 cycle=[]
cls=[[7, 4, 5, 1]], i=2 cycle=[]
cls=[[7, 4, 5, 1]], i=2 cycle=[2]
cls=[[7, 4, 5, 1]], i=2 cycle=[2, 6]
cls=[[7, 4, 5, 1]], i=2 cycle=[2, 6, 9]
cls=[[7, 4, 5, 1], [9, 6, 2]], i=3 cycle=[]
cls=[[7, 4, 5, 1], [9, 6, 2]], i=3 cycle=[]
cls=[[7, 4, 5, 1], [9, 6, 2]], i=3 cycle=[3]
cls=[[7, 4, 5, 1], [9, 6, 2]], i=3 cycle=[3, 8]

cycles=[[7, 4, 5, 1], [9, 6, 2], [8, 3]]
{% endhighlight %}

<!--excerpt-->
