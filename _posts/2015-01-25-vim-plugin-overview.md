---
layout: post
title: vim as IDE
categories: personal
tags: 
  - vim
  - ide
  - ycm
  - clang
comments: true
mathjax: null
featured: true
published: true
---

Vim - обзор плагинов
===================

В предыдущей статье я рассмотрел парочку плагинов, но не давал всему этому
зоопарку полноценного обзора, пора бы это исправить. Не знаю насколько это
удобно, но я для простоты, чтобы ничего не пропустить, пойду более-менее по
порядку.

{% highlight vim %}
NeoBundle 'Valloric/YouCompleteMe'
{% endhighlight %}

Первый у нас это youcompleteme, про который я уже рассказывал. Это быстрый
и удобный автокомплитер для vim. Также есть биндинги для emacs, atom и других
редакторов c помощью ycm, но я не проверял как они работают.

Достоинства:

* Самая высокая скорость работы
* Fuzzy-finding
* Поддержка довольно большого количества языков

Недостатки:

* Требуется поддержка python
* Требуется компиляция

Чтобы собрать его нужно воспользоваться примерно следующей командой(за
подробностями можно обратиться к документации):

{% highlight sh %}
python2 install.py --clang-completer --omnisharp-completer --gocode-completer --tern-completer --racer-completer
{% endhighlight %}

Возможно есть комплитеры быстрее, но я их не знаю.

Тем не менее грех о них не упомянуть:

* neocomplete. Требует lua, что делает его невозможным для использования
  в neovim на настоящий момент. Из достоинств, как мне кажется, большая
  информативность некоторых видов автокомплита по сравнению с youcompleteme.
  Когда я в последний раз его пробовал скорость была так себе, возможно
  в настоящий момент это изменилось.
* clang_compete. Не универсальный, работает медленней чем youcompleteme,
  следовательно не нужен.
* AutoComplPop. Довольно старый, использование его не имеет смысла как по мне.
* supertab. Довольно неплохой, активно поддерживается, автокомплит не совсем
  auto, а с помощью tab. Возможно неплохое решение для слабых компьютеров, но
  я не использую.
* Shougo/deoplete.nvim. Выглядит круто, честно говоря пока не пробовал.
  Возможно в будущем это будет классный проект, но в настоящий момент он  ещё
  не вылез из детских яслей.

{% highlight vim %}
" python powerline autodetection
if !(&runtimepath =~ 'site-packages/powerline/bindings/vim')
    NeoBundle 'itchyny/lightline.vim.git' "lightline is more fancy than default
endif
{% endhighlight %}

Статусная строка. Использую в neovim из-за слабой поддержки python, которой
в настоящий момент не достаточно для использования powerline. Также можно
воспользоваться airline, которая вроде бы симпотичнее, но я не заморачиваюсь,
потому что в данный момент я не являюсь активным пользователем neovim, а для
чего-то ещё оно мне и не нужно.

{% highlight vim %}
NeoBundle 'junegunn/fzf'     "to work with fzf-vim
NeoBundle 'junegunn/fzf.vim' "use fzf plug for vim
{% endhighlight %}

Сладкая парочка плагинов, которая добавляет поддержку fzf в vim. Штука эта
очень хорошая, но как по мне недостаточно фичастая. Итак:

Достоинства:

* Самая высокая скорость работы
* standalone, можно использовать как плагин для bash/zsh и прочих программ
* Не имеет прямого аналога
* Универсальность конфигурации(с оговорками)
* Поддержка tmux

Недостатки:

* Не всегда удобно в использовании, например по сравнению с lusty для vim
  нельзя просто так взять и откатиться на директорию назад(если можно, то
  скажите как), лично для меня особенно не хватает многострочного отображения.

Хотя у этой штуки нет прямого аналога есть своего рода конкуренты, рассмотрим их:

* unite. Довольно неплохой fuzzy-finding, но уступает по удобству использования
  lusty. В целом нужен из-за большего количества функций, лучшей интеграции
  с vim, а также более богатыми возможностями вывода и кастомизации по
  сравнению с конкурентами.
* ctrlp. В принципе ничего так, активно развивается, быстро работает. Но не
  вижу существенных преимуществ по сравнению с fzf. И вывод не позволяет
  настроить так тонко как в unite.
* Всякое разное. Не больно-то нужно когда есть unite. Однако...:
  
{% highlight vim %}
if !has("nvim")
    NeoBundle 'sjbach/lusty.git' "file/buffer explorer
else
    "temp disable because of segfault
    if has("ololo")
        NeoBundle 'https://bitbucket.org/mikehart/lycosaexplorer' "python lusty analog
    endif
endif
{% endhighlight %}

Также существует вот эта компания плагинов, из которых реально используется только
один. lusty это мой любимый способ для навигации по файлам. Требует поддержки
ruby, в отличие от всего остального немногословен и есть поддержка
многострочности, которой мне не хватает в fzf, а также удобен. Есть так же
что-то вроде форка lycosaexplorer на python, который я пытался использовать для
интеграции в neovim, но когда я в последний раз проверял это, то(как минимум)
в сочетании с unite это приводило к сегфолту nvim.

{% highlight vim %}
NeoBundle 'luochen1990/rainbow'
{% endhighlight %}

Это раскраска скобочек всеми цветами радуги для vim. Достоинства в том что она
есть и вроде как худо-бедно поддерживается. Недостатки как по мне в том что
нельзя ограничить уровень вложенности(мне не по душе когда цвета) начинают
повторяться, также мне не нравится что не всегда можно управлять семантикой, то
есть он просто считает уровень скобок(любых) вне зависимости от их
характеристик. На фоне того что есть для emacs смотрится как-то слабо. Также не
раскрашивает скобки в отдельности, без пар, не позволяет раскрашивать операторы
не так как скобки и проч.

{% highlight vim %}
NeoBundle 'chrisbra/colorizer'
    \, { 'autoload': { 'commands': ['ColorToggle'] } } "ascii to colors
{% endhighlight %}

![Terminal Coloring](https://raw.githubusercontent.com/chrisbra/Colorizer/master/Colorizer.gif)

Плагин для перевода esc-последовательностей в раскраску vim. Иногда это нужно,
а иногда приятно. Больше добавить особо нечего.

{% highlight vim %}
NeoBundle 'Shougo/neomru.vim.git' "mru for unite
{% endhighlight %}

Для того чтобы получать список последних использованных файлов. Это очень удобно.
Из достоинств: standalone, так что теоретически можно настроить интеграцию с fzf.
Используется в интеграции с unite.

{% highlight vim %}
NeoBundle 'Shougo/unite.vim.git' "unite for creating ui
{% endhighlight %}

Комбайн для построения классного TUI для vim, чего там только нет. Прямого аналога
не имеет. Разумеется очень рекомендуется.

{% highlight vim %}
NeoBundle 'mopp/autodirmake.vim.git' "automake dir which didnt exists
{% endhighlight %}

Небольшой плагин для автосоздания директорий. Удобно.

{% highlight vim %}
NeoBundle 'Shougo/vimfiler.vim', {
            \ 'depends' : 'Shougo/unite.vim', 'commands' : [
            \ { 'name' : 'VimFiler', 'complete' : 'customlist,vimfiler#complete' },
            \ { 'name' : 'VimFilerTab', 'complete' : 'customlist,vimfiler#complete' },
            \ { 'name' : 'VimFilerExplorer', 'complete' : 'customlist,vimfiler#complete' },
            \ { 'name' : 'Edit', 'complete' : 'customlist,vimfiler#complete' },
            \ { 'name' : 'Write', 'complete' : 'customlist,vimfiler#complete' },
            \ 'Read', 'Source'],
            \ 'mappings' : '<Plug>(vimfiler_', 'explorer' : 1, }
{% endhighlight %}

{% highlight vim %}
if !has("nvim")
    NeoBundle 'Shougo/vimshell.vim' "shell inside a vim for unite and vimfiler integration
endif
{% endhighlight %}

Shell внутри vim. Честно говоря особо и не нужно. В neovim и так встроено. Пожалуй от его
использования я откажусь в будущем.

{% highlight vim %}
NeoBundleLazy 'Shougo/unite-outline', { 'unite_sources' : 'outline' }
{% endhighlight %}

Приблуда к unite для быстрой навигации по файлу. В чем-то удобнее чем tagbar
и иже с ним, но нужно не часто.

{% highlight vim %}
NeoBundle 'junkblocker/unite-codesearch' "junkblocker google codesearch wrapper
{% endhighlight %}

Интеграция google code-search. Это форк, так что нужно использовать именно его,
с кастомным codesearch от того же автора. В принципе можно использовать для
поиска по большим базам кода, но это скорее актуально для Google Inc у которой
счет количества строк кода идет на миллиарды.

{% highlight vim %}
NeoBundle 'Shougo/junkfile.vim.git' "junkfile for unite
{% endhighlight %}
Удобное создание временных файлов. Прелесть этого плагина 
мне ещё предстоит на себе испытать, но в целом нужно.

{% highlight vim %}
NeoBundle 'rhysd/vim-clang-format.git' "format code by clang, better than astyle -A14
{% endhighlight %}
Простой плагин для семантического выравнивания кода. По сути не делает ничего,
просто вызывает clang-format. А вот сам clang-format штука очень хорошая.
Отличается от аналогов тем что учитывает AST.

{% highlight vim %}
NeoBundle 'SirVer/ultisnips.git' "Snippets with ycm compatibility
{% endhighlight %}

Самый крутой движок для сниппетов в vim. Nuff-said, в общем-то.
Безальтернативен из-за поддержки youcompleteme

{% highlight vim %}
NeoBundle 'godlygeek/tabular.git' "for tabularizing
{% endhighlight %}

{% highlight vim %}
if executable(resolve(expand("ack")))
    NeoBundle 'mileszs/ack.vim' "ack wrapper
endif
if executable(resolve(expand("ag")))
    NeoBundle 'rking/ag.vim.git' "ag (ack replacement) wrapper
endif
{% endhighlight %}

Интеграция поисковиков ag и ack. Это такие аналоги grep. Честно говоря если
unite работает без них, то не нужно, надо проверить.

{% highlight vim %}
NeoBundleLazy 'tpope/vim-repeat', { 'mappings' : '.' } "dot for everything
{% endhighlight %}

Повторение через точку для большего количества команд, чем по умолчанию
предусмотрено в vim.

{% highlight vim %}
NeoBundle 'tpope/vim-eunuch.git' "for SudoWrite, Locate, Find etc
{% endhighlight %}

Дополнительные встроенные в vim команды. Мой любимчик это SudoWrite, потому что
я всё предпочитаю редактировать через один и тот же instance vim'а.

{% highlight vim %}
NeoBundleLazy 'sjl/gundo.vim', { 'commands' : 'GundoToggle' }
{% endhighlight %}

Дерево отмены. Нужно, используется частенько, когда нужно отменить что-то после
redo.

{% highlight vim %}
NeoBundleLazy 'Raimondi/delimitMate', { 'insert' : 1 } "autopair ()[]{}
{% endhighlight %}

Автоподстановка скобок для vim. Сначала кажется что не нужно, потом без этого жить
не можешь. Безальтернативен из-за поддержки в youcompleteme.

{% highlight vim %}
NeoBundleLazy 'scrooloose/syntastic', { 'insert' : 1 } "syntax checker
{% endhighlight %}

Проверка синтаксиса в vim. Одна из тех вещей, которая нужна чтобы сделать vim
похожим на IDE

{% highlight vim %}
NeoBundle 'nathanaelkane/vim-indent-guides' "indent tabs visually with |-es too slow
{% endhighlight %}
Подветка отступов с помощью изменения цвета background. Аналоги не подходят из-за
конфликта с переопределением цвета conceal.

{% highlight vim %}
NeoBundle 'thinca/vim-qfreplace.git' "visual replace for multiple files
{% endhighlight %}

Довольно необычный способ взаимодействия с Quickfix. для того чтобы понять что
это можно посмотреть вот это видео: https://vimeo.com/24700977

{% highlight vim %}
NeoBundle 'c9s/vimomni.vim' "autocompletion for VimL
{% endhighlight %}
omni-completion для vim. Иногда работает, иногда не очень. Не нужно если использовать
neocomplete например.

{% highlight vim %}
if executable(resolve(expand("git")))
    NeoBundle 'tpope/vim-fugitive.git' "Git stuff. Needed for powerline etc
    NeoBundle 'junegunn/vim-github-dashboard.git' "Git dashboard in vim
    NeoBundle 'jaxbot/github-issues.vim.git' "github issues autocomp
    NeoBundle 'idanarye/vim-merginal.git' "to handle branches/merge conflicts
    NeoBundle 'junegunn/gv.vim' "yet another git commit browser
    NeoBundle 'vim-scripts/DirDiff.vim.git' "diff directories easyer with vim
    NeoBundle 'airblade/vim-gitgutter.git' "last changes
endif
{% endhighlight %}
Пачка плагинов для поддержки git в vim. Здесь самый главный это fugitive,
также мне очень нравится смотреть последние коммиты с помощью GV. Думаю добавить
это в интеграцию одной своей приблуды, про которую я расскажу чуть позже, чтобы
использовать вместо tig. Mergial это простой плагин для того чтобы разбираться
с конфликтами слияний. В общем-то он так прост, что и рассказать особо нечего,
остальные плагины по желанию.

{% highlight vim %}
if executable(resolve(expand("tmux")))
    NeoBundle 'tpope/vim-tbone.git' "tmux basics
    NeoBundle 'benmills/vimux.git' "exec commands in tmux
    NeoBundle 'christoomey/vim-tmux-navigator' "easy jump between windows
    NeoBundle 'epeli/slimux' "better interaction with tmux
endif
{% endhighlight %}
Различные плагины для взаимодействия с tmux. Из них выделю
christoomey/vim-tmux-navigator который позволяет прыгать между буферами tmux
с помощью тех же хоткеев, что и между vim. И прозрачно. Словно это одна и та же
сущность. Slimux это что-то вроде реинкарнации slime. Можно использовать,
удобно. Хотя в общем-то уже есть dispatch.

{% highlight vim %}
NeoBundleLazy 'chrisbra/unicode.vim', { 'commands' : ['UnicodeComplete','UnicodeGA', 'UnicodeTable'] } "better digraphs
{% endhighlight %}
Улучшенная поддержка диграфов в vim. На любителя.

{% highlight vim %}
NeoBundle 'Shougo/neossh.vim.git' "work with ssh easier
{% endhighlight %}
Для удобной работы с буферами через ssh. Помнится netrw мне в этом плане
показался более сложным и неудобным, а благодаря этому плагину всё ок.

{% highlight vim %}
NeoBundle 'vim-scripts/ViewOutput.git' "VO commandline output
{% endhighlight %}
Более простой и удобный способ перенаправления команд vim, чем стандартный redir.

{% highlight vim %}
NeoBundle 'kana/vim-gf-user.git' "framework open file by context
NeoBundle 'kana/vim-gf-diff.git' "go to the changed block under the cursor from diff output
NeoBundle 'mattn/gf-user-vimfn.git' "vim-gf-user extension: jump Vim script function
NeoBundle 'mkomitee/vim-gf-python.git' "gf for python
{% endhighlight %}
Различные плагины, которые направлены на улучшение поддержки команды gf для
навигации по всяким файлам и прочим сущностям.

{% highlight vim %}
" There is no need in fixkey for nvim because of it's default behaviour
if !has("nvim")
    NeoBundle 'drmikehenry/vim-fixkey' "fixes key codes for console Vim
endif
{% endhighlight %}

Очень полезный плагин, для добавления поддерки mod1(alt) хоткеев в консольный вим.
В neovim это уже не нужно. Используется прежде всего для поддержки моих хоткеев,
которые переносят часть из emacs(вроде c-a в начало строки).

{% highlight vim %}
NeoBundle 'ReekenX/vim-rename2.git' "rename for files even with spaces in filename
{% endhighlight %}
Переименование текущего файла прямо из vim. Иногда нужно, но можно обойтись и без него.

{% highlight vim %}
NeoBundle 'thinca/vim-ref.git' "integrated reference viewer man/perldoc etc
{% endhighlight %}
Расширение функциональности просмотра документации.

{% highlight vim %}
NeoBundle 'othree/eregex.vim' "Perl-like extended regex for vim
{% endhighlight %}
Улучшенный поисковик с поддержкой pcre регулярных выражений. Лично я использую
редко, потому что как правило когда попадаю в такую ситуацию, то я в командной
строке, а не виме.

{% highlight vim %}
NeoBundle 'chrisbra/Join.git' "Extended and fast Join for vim
{% endhighlight %}
Плагин, который ускоряет команду join. Также можно немного кастомизировать.
Нужно проверить так ли это актуально в настоящий момент.

{% highlight vim %}
NeoBundle 'lyokha/vim-xkbswitch.git' "Autoswitch on <esc> with libxkb needs xkb-switch-git to run
{% endhighlight %}
Плагин, который вкупе с определенными настройками дает более удобную работу с русской раскладкой в vim.
Идеал не достигнут. Чтобы его получить можно например написать кастомный переключатель, который работает
без определений вроде

{% highlight shell %}
setxkbmap -option keypad:pointerkeys -layout 'us,ru' -variant altgr-intl \
    -option 'grp:alt_shift_toggle' -option ctrl:nocaps
{% endhighlight %}

И определяет текущее окно, для которого делать триггер раскладки vim вместо того чтобы дергать системную
раскладку. Тем не менее подход с control+s, который используется у меня позволяет использовать все нужные
хоткеи без проблем и есть не просит.

Также есть плагин 
{% highlight vim %}
NeoBundle 'kana/vim-arpeggio.git' "mappings for simultaneously pressed keys
{% endhighlight %}

Который позволяет отрабатывать одновременные нажатия клавиш. У меня это
используется для сочетания jk, которое заменяет мне esc. Это нужно для того
чтобы буква j не создавала задержку ожидания. Есть некоторая несовместимость по
отношению к русской раскладке. К счастью в большинстве случаев можно
воспользоваться control+c или control+] в зависимости от того что нужно и не
тянуться к esc.

{% highlight vim %}
"is all about surroundings: parentheses, brackets, quotes, XML tags, and more
NeoBundleLazy 'tpope/vim-surround', { 'mappings' : [ ['n', 'cs', 'ds', 'ys', 'yS'], ['x', 'S']] }
{% endhighlight %}
Довольно известный плагин для vim, который позволяет с помощью одной команды
добавлять и убирать скобки. Также обладает расширяемостью под разные паттерны.
Иногда полезно.

{% highlight vim %}
NeoBundle 'jamessan/vim-gnupg.git' "Transparent work with gpg-encrypted files
{% endhighlight %}
Улучшенная поддержка gpg в vim. Нужно в некоторых специфических случаях.

{% highlight vim %}
NeoBundle 'Shougo/echodoc.vim' "prints doc in echo area
{% endhighlight %}
Выводит всякую информацию вроде прототипов функции или чего-то ещё в область
echo(это та, которая под статусной строкой)

{% highlight vim %}
if executable(resolve(expand("task")))
    NeoBundle 'blindFS/vim-taskwarrior' "add taskwarrior vim plug wrapper
endif
{% endhighlight %}
Очень классная интеграция taskwarrior в vim. Представляет собой упрощенный аналог 
org-mode emacs. Лично мне очень по душе. Просто и полезно. Как по мне главный плюс
это более узкая направленность, простота и то что сама утилита standalone.

{% highlight vim %}
NeoBundle 'kopischke/vim-fetch' "vim path/to/file.ext:12:3
{% endhighlight %}
Удобный прыжок на определенное место в файле, иногда бывает нужно. Достоинство 
в том что работает везде.

{% highlight vim %}
NeoBundle 'FooSoft/vim-argwrap' "vim arg wrapper
{% endhighlight %}
Команда для "схлопывания" и "захлопывания" содержимого скобок,
лично я использую достаточно часто.

{% highlight vim %}
NeoBundleLazy 'majutsushi/tagbar', { 'commands' : 'TagbarToggle' }
{% endhighlight %}
Список тегов, используются довольно часто. Лучше всех прямых аналогов.

{% highlight vim %}
NeoBundle 'chrisbra/vim-diff-enhanced.git' "patience diff
{% endhighlight %}
Улучшенная реализация diff с более грамотным алгоритмом. Используется редко, но
метко, когда нужно обновить программу, в которой было сделано много изменений
это помогает.

{% highlight vim %}
NeoBundle 'sombr/vim-scala-worksheet.git' "tiny Vim plugin that turns your file into interactive worksheet
{% endhighlight %}
Интересный плагин, который позволяет интерактивно использовать scala c помощью
создания любого файла с расширением \*.scalaws код рассматривается
и выполняется worksheet. Очень удобно в плане работы, подходит даже для
взаимодействия с большими проектами, которые используют akka

{% highlight vim %}
NeoBundle 'ensime/ensime-vim' "scala vim autocompletion
{% endhighlight %}
Система автодополнения для scala для разных редакторов, в том числе vim.
Установка и настройка требует некоторых телодвижений, но в целом удобно.

{% highlight vim %}
NeoBundle 'derekwyatt/vim-scala' "various initial scala support for vim
NeoBundle 'derekwyatt/vim-sbt' "basic SBT support for vim
{% endhighlight %}
Добавляет некоторую простую поддержку scala и sbt в vim без автодополнения
и прочих фич IDE.

{% highlight vim %}
NeoBundle 'tpope/vim-commentary.git' "try it instead of tcomment
{% endhighlight %}
Простой авторасстановщик комментариев. Выделели область или textobj,
скомандовали gc и готово! Очень просто и удобно.

{% highlight vim %}
NeoBundle 'tpope/vim-endwise' "to insert endif for if, end for begin and so on
{% endhighlight %}
Автоматическое добавление парных слов для разных языков. Достаточно удобно, 
хотя и не немного дебильное, не помешала бы поддержка семантики, интересно
узнать есть ли что-то лучше.

{% highlight vim %}
NeoBundle 'tpope/vim-unimpaired.git' "good mappings and toggles
{% endhighlight %}
Всякие разнообразные кастомизации от Tim Pope. Довольно неплохая совместимость с существующими
плагинами, так что всё ок, лучше обратиться к документации за подробностями.

{% highlight vim %}
NeoBundle 'dbakker/vim-projectroot' "better rooter
{% endhighlight %}
Удобный детект текущей директории проекта. Нужно, поскольку autochdir не всегда
корректно работает для автокоманд vim, а ручной переход медленно и лениво,
а так нажали cd в режиме редактирования и готово!

{% highlight vim %}
NeoBundle 'Valloric/ListToggle.git' "toggle quickfix and location list <leader>l by def
{% endhighlight %}
Простой плагин для вкл/выкл quickfix window. У меня забинджено на qq.
Используется частенько.

{% highlight vim %}
NeoBundle 'derekwyatt/vim-fswitch.git' "switching between companion source files (e.g. .h and .cpp)
{% endhighlight %}
Простой плагин для переключения между сорцом и хедером. У меня забинджено на
control+a. Используется достаточно часто.

{% highlight vim %}
NeoBundle 'vim-scripts/IndentConsistencyCop.git' "autochecks for indent
NeoBundle 'hynek/vim-python-pep8-indent.git' "python autoindent pep8 compatible
NeoBundle 'fs111/pydoc.vim' , {'autoload': {'filetypes': ['python']} } "pydoc integration
{% endhighlight %}
Всякие мелкие плагины для поддержки python, которые не связаны с автодополнением.

{% highlight vim %}
if executable("mono")
    NeoBundleLazy 'nosami/Omnisharp.git', { 'filetypes' : 'cs' } "omnisharp completion
endif
{% endhighlight %}
Автодополнение для c#

{% highlight vim %}
if executable(resolve(expand("gotags")))
    NeoBundle 'jstemmer/gotags.git' "tags for go
endif
if executable(resolve(expand("go")))
    NeoBundle 'Blackrush/vim-gocode.git' "omnicomplete for go
    NeoBundle 'fatih/vim-go.git' "golang support
endif
{% endhighlight %}
Теги для go и некоторая поддержка для этого языка, в частности автодополнение.
{% highlight vim %}
if executable(resolve(expand("rustc")))
    NeoBundle 'rust-lang/rust.vim' "detection of rust files
    NeoBundle 'jtdowney/vimux-cargo' "rust-cargo bindings
endif
{% endhighlight %}
Поддержка детекта rust, улучшенный(на момент написания поста) syntax highlighting для rust,
а также поддержка асинхронной работы cargo с через tmux.

{% highlight vim %}
NeoBundleLazy 'vim-perl/vim-perl', { 'filetypes' : 'perl' }
NeoBundleLazy 'wannesm/wmgraphviz.vim', { 'filetypes' : 'dot' }
NeoBundle 'sbl/scvim.git' "vim plugin for supercollider
NeoBundle 'xolox/vim-lua-ftplugin.git' "test lua bindings
NeoBundle 'oscarh/vimerl' "vim erlang support
{% endhighlight %}
Простая поддержка для языков perl, dot, supercollider, erlang и lua. Из
последнего поддержка lua получше, но конечно не такая хорошая как хотелось бы.

{% highlight vim %}
NeoBundle 'janko-m/vim-test.git' "easy testing for various langs
{% endhighlight %}
Универсальный плагин для тестирования на разных языках.

{% highlight vim %}
NeoBundle 'tpope/vim-dispatch.git' "provide async build via tmux
if executable(resolve(expand("rc")))
    NeoBundle 'lyuts/vim-rtags.git' "rtags plugin for vim
endif
if executable(resolve(expand("ghci")))
    NeoBundle 'ujihisa/neco-ghc' "autocomplete for hs using ghc-mod
    NeoBundle 'eagletmt/ghcmod-vim.git'
    NeoBundle 'bitc/vim-hdevtools' "type-related features
    NeoBundle 'neg-serg/vim2hs' "better haskell syntax hi with better indenting
endif            
if executable(resolve(expand("ruby")))
    NeoBundle 'vim-ruby/vim-ruby' "ruby autocompletion
    NeoBundle 'tpope/vim-rails.git' "rails plugin from Tim Pope
    NeoBundle 'tpope/vim-rake.git' "ruby rake support
    NeoBundle 'tpope/vim-rbenv.git' "ruby rbenv support
    NeoBundle 'tpope/vim-bundler' "ruby bundler support
    NeoBundle 'vim-scripts/dbext.vim' "provides database access to many dbms
    NeoBundle 'skalnik/vim-vroom' "plugin to run ruby tests
endif
NeoBundle 'shawncplus/phpcomplete.vim.git' "better than default phpcomplete.vim
" Multi-language DBGP debugger client for Vim (PHP, Python, Perl, Ruby, etc.)
NeoBundleLazy 'joonty/vdebug', { 'autoload': { 'commands': 'VdebugStart' }}
" html5 autocomplete and syntax
NeoBundleLazy 'othree/html5.vim' , {'autoload': {'filetypes': ['html', 'htmldjango']} }
NeoBundle 'fatih/vim-nginx' "nginx runtime files
"----------------[  Tags  ]--------------------------------------------------------------
NeoBundle 'szw/vim-tags' "autogen ctags
if executable(resolve(expand("gtags")))
    NeoBundle 'yuki777/gtags.vim.git' "Gtags v0.64
    NeoBundle 'bbchung/gasynctags.git' "autogenerate gtags to cscope db
    NeoBundle 'tranngocthachs/gtags-cscope-vim-plugin.git' "gtags-cscope navigation
endif
"--[ latex ]-----------------------------------------------------------------------------
NeoBundle 'lervag/vimtex' "LaTeX-Box replacement
"--[ web ]-------------------------------------------------------------------------------
NeoBundle 'rstacruz/sparkup.git' "write html code faster
NeoBundle 'Valloric/vim-instant-markdown' "realtime markdown preview
NeoBundleLazy 'marijnh/tern_for_vim', { 'autoload': { 'filetypes': ['javascript'] } }
"---------------[ syntax ]---------------------------------------------------------------
NeoBundle 'vimez/vim-tmux' "syntax hi for tmux
NeoBundle 'elzr/vim-json' "syntax hi for json format
NeoBundle 'cespare/vim-toml' "syntax hi for toml format
NeoBundle 'rsmenon/vim-mathematica.git' "Mathematica syntax and omnicomp
NeoBundle 'jelera/vim-javascript-syntax.git' "js highlighting
NeoBundle 'tpope/vim-git' "syntax, indent, and filetype plugin files for git
NeoBundle 'ekalinin/Dockerfile.vim' "dockerfile hi
NeoBundle 'jnwhiteh/vim-golang.git' "golang syntax highlight
NeoBundleLazy 'farseer90718/vim-regionsyntax', { 'filetypes' : ['vimwiki', 'markdown', 'tex', 'html'] }
NeoBundle 'JulesWang/css.vim' "better css syntax hi
NeoBundle 'leafo/moonscript-vim' "basic moonscript support
NeoBundle 'rodjek/vim-puppet' "basic puppet support
if !has("nvim") && has("ololo")
    NeoBundle 'bbchung/clighter.git' "hi with clang
elseif has("nvim")
    NeoBundle 'bbchung/Clamp' "clighterr for neovim
endif
if has("gui_running")
    NeoBundle 'drmikehenry/vim-fontsize.git' "set fontsize on the fly
    NeoBundle 'tyru/restart.vim.git' "add restart support
    NeoBundle 'vim-scripts/utl.vim.git' "Open urls in files
    NeoBundle 'bling/vim-airline.git' "statusline for gvim only
endif
NeoBundle 'ryanoasis/vim-devicons.git' "fancy icons for fonts
{% endhighlight %}
