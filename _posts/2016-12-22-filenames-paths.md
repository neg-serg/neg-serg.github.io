---
layout: post
title: Filenames and pathnames in shell
categories: personal
tags: shell
comments: true
mathjax: null
featured: true
published: true
---

Interesting article about handling paths and filenames in shell:
<a href="http://www.dwheeler.com/essays/filenames-in-shell.html">http://www.dwheeler.com/essays/filenames-in-shell.html</a>

Look at this quote from there:
<p class="message">

Encoding pathnames

It is possible to encode pathnames so that all pathnames can be handled. There is no standard POSIX mechanism for doing this encoding, unfortunately.

encodef is a small utility I wrote that can encode and decode filenames in a few formats. With it, you can do this:

{% highlight shell %}
# This version is POSIX portable; in practice
# you can often use "-print0" instead of "-exec printf '%s\0' {} \;"
for encoded_pathname in $(find . -exec printf '%s\0' {} \; | encodef ) ; do
    file="$(encodef -d -Y -- "$encoded_pathname")" ; file="${file%Y}"
    COMMAND "$file" # Use quoted "$file", not $file, everywhere.
done
{% endhighlight %}
</p>
