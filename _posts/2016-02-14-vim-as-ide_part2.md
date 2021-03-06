---
layout: post
title: Vim as IDE часть II
categories: personal
tags: vim ide ycm rtags clang
comments: true
mathjax: null
featured: true
published: true
date: 2016-02-14 14:30:00 +03:00
---

Решил добавить парочку небольших дополнений к той статье, про использование
vim как IDE, которая уже была. Пост будет небольшой, расскажу про современное
состояние поддержки rtags и про генерацию файлов проекта.

<!--excerpt-->


# Генерация файлов проекта

Как всем известно в любой IDE, таких как eclipse или продуктах от JetBrains
почти всегда перед работой над файлами требуется создать проект, в котором
обычно указываются опции для сборки и отладки, репозитории если есть и другая
метаинформация, которая нужна для корректной поддержки фич IDE как сборка,
автокомплит, отладка, теги, навигация по коду, рефакторинг и так далее.

Сейчас я покажу способ как можно быстро и просто получить информацию
о проекте не компилируя его так как это делается с помощью bear
+ clang. 

## <a href="https://github.com/rdnetto/YCM-Generator">Ycm-Generator</a>

Эта небольшая утилита позволяет быстро и просто получить метаинформацию
о коде, которая потом будет использоваться для проверки на warnings,
синтаксиса, навигации, семантического автокомплита, короче всего того что
предоставляет YouCompleteMe. 

В случае простого проекта всё замечательно, вам достаточно корректно
настроенного стандартного .ycm_extra_conf.py, в котором будут настройки для
наиболее типичных случаев, что позволяет просто создать файл и без всяких
телодвижений мгновенно получить для него автокомплит. В более сложном случае
это не получится. Для этого можно воспользоваться или bear или <a
href="https://github.com/rdnetto/YCM-Generator">Ycm-Generator</a>, который
можно скачать по ссылке, установка же как для обычного плагина.

Использование:

Первый способ(standalone): Достаточно просто перейти в директорию с плагином,
скорее всего это будет *.vim/bundle/YCM-Generator* , после чего
воспользоваться 
```
    ./config_gen.py PROJECT_DIRECTORY
```

, где PROJECT_DIRECTORY это директория с файлом проекта где уже лежит
MakeFile или что-то подобное для системы сборки.

Второй способ это перейти в vim в директорию проекта, после чего просто
использовать команду *:YcmGenerateConfig*. Работает эта штука очень быстро.

Однако не всё так гладко как описывает автор. Да, часть библиотек, пути
и переменные окружения будут подхвачены, но в целом "качество" конфига будет
уступать тому, который будет получен с помощью bear. Тогда зачем это нужно?
Как мне кажется ей можно найти хорошее применение для новых проектов, которые
пишутся своими руками. Чтобы не придумывать конфиг с нуля, если вы при этом
хотите обойтись без *compile_commands.json*, который ручному редактированию
особо не поддается, потому что большой и это безумие.

# Rtags и простой рефакторинг

Для вима всегда не хватало хотя бы простого рефакторинга. В тех случаях,
когда для проекта возможно использование clang можно воспользоваться и rtags.
Чтобы добавить поддержку rtags в vim нужно воспользоваться вот этим плагином:
<a href="https://github.com/lyuts/vim-rtags">vim-rtags</a>
Compilation database можно сгенерировать с помощью bear как обычно, например
*bear make -j10*. После чего 

Допустим мы в директории с проектом, тогда:

```
    rdm &
    rc -J compile_commands.json
```

В качестве примера я рассматриваю Notion. Убедимся что семантическое
автодополнение работает, а значит скорее всего конфигурация считана верно:

![notion_cfg](http://i.imgur.com/a7OPeOI.png)

После этого мы можем получить _семантическую_ навигацию по тегам, а также
метаинформацию о типе:

<blockquote class="imgur-embed-pub" lang="en" data-id="a/Abqr5">
<a href="//imgur.com/a/Abqr5">View post on imgur.com</a></blockquote>
<script async src="//s.imgur.com/min/embed.js" charset="utf-8"></script>

Можете воспользоваться простым переименованием символов с помощью
*\<leader\>rw*, чтобы узнать что есть ещё можно обратиться к документации.
Например есть поддержка Unite для вим. Функций довольно много. И сам Rtags
и его поддежка в Vim активно развиваются, если чего-то недостает энтузиасты
могут портировать уже созданную для Emacs функциональность с Elisp на VimL.
