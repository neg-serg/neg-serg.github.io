---
layout: post
title: Vim - обзор плагинов
categories: personal
tags: 
  - vim
  - plugin
  - overview
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
порядку. Заодно покажу как они у меня настроены, с некоторыми рекомендациями.

* TOC
{:toc}

## Автодополнение

### Valloric/YouCompleteMe

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

Теперь к настройке. Во-первых, если вы заметили, то все настройки плагинов
у меня заключены в "скобки" из конструкций вида вроде *if
neobundle#tap('YouCompleteMe') ... endif*. Это сделано для того чтобы не
выполнять настройку плагинов тогда, когда они не используются, что с одной
стороны создает что-то вроде дополнительного уровня абстракции(четко видно где
настройки какого плагина), а с другой позволяет потенциально немного ускорить
загрузку. В любом случае ускорение точно будет в тех случаях, когда команды
"тяжелые".

Вот что у меня сделано для ycm:
{% highlight vim %}
if neobundle#tap('YouCompleteMe')
    " Do not display "Pattern not found" messages during YouCompleteMe completion.
    " Patch: https://groups.google.com/forum/#!topic/vim_dev/WeBBjkXE8H8
    if 1 && exists(':try')
        try
            set shortmess+=c
            " Catch "Illegal character" (and its translations).
        catch /E539: /
        endtry
    endif
    let g:ycm_global_ycm_extra_conf = '~/.vim/.ycm_extra_conf.py'
    let g:ycm_rust_src_path = '/home/neg/src/1st_level/rust_src17/rustc-nightly/src'
    let g:ycm_filepath_completion_use_working_dir = 0
    let g:ycm_confirm_extra_conf = 0
    let g:ycm_cache_omnifunc = 1
    let g:ycm_collect_identifiers_from_tags_files = 1
    "--[ syntastic enabling ]-----------------
    let g:ycm_show_diagnostics_ui          = 1 
    let g:ycm_seed_identifiers_with_syntax = 0
    let g:ycm_use_ultisnips_completer      = 1

    let g:ycm_key_list_select_completion = ['<TAB>', '<Down>']
    let g:ycm_key_list_previous_completion = ['<S-TAB>', '<Up>']
    let g:ycm_key_invoke_completion = '<M-x>'
    let g:ycm_autoclose_preview_window_after_insertion = 1
    nnoremap <leader>y :YcmForceCompileAndDiagnostics<cr>

    " let g:ycm_add_preview_to_completeopt = 1
    " let g:ycm_autoclose_preview_window_after_completion = 0
    " let g:ycm_autoclose_preview_window_after_insertion = 0 

    let g:ycm_semantic_triggers =  {
        \   'c' : ['->', '.'],
        \   'objc' : ['->', '.'],
        \   'cpp,objcpp' : ['->', '.', '::'],
        \   'perl' : ['->'],
        \   'php' : ['->', '::'],
        \   'cs,java,javascript,d,vim,ruby,perl6,scala,vb,elixir,go' : ['.'],
        \   'lua' : ['.', ':'],
        \   'erlang' : [':'],
        \   'ocaml' : ['.', '#'],
        \   'ruby' : ['.', '::'],
        \ }

    let g:ycm_collect_identifiers_from_tags_files = 0

    let g:ycm_filetype_blacklist = {
        \ 'notes'      : 1,
        \ 'markdown'   : 1,
        \ 'text'       : 1,
        \ 'unite'      : 1,
        \ 'conqueterm' : 1,
        \ 'asm'        : 1,
        \ 'tagbar'     : 1,
        \ 'qf'         : 1,
        \ 'vimwiki'    : 1,
        \ 'pandoc'     : 1,
        \ 'infolog'    : 1,
        \ 'mail'       : 1
        \}

    let g:ycm_min_num_identifier_candidate_chars = 4
    let g:ycm_filetype_specific_completion_to_disable = {'javascript': 1, 'python': 1}
    let g:ycm_goto_buffer_command = 'vertical-split'

    nnoremap <silent> <F3> :call youcompleteme#DisableCursorMovedAutocommands()<CR>
    nnoremap <silent> <F4> :call youcompleteme#EnableCursorMovedAutocommands()<CR>

    function! s:SetCompleteFunc()
        if !g:neocomplete#force_overwrite_completefunc
            let &completefunc = 'youcompleteme#Complete'
            let &l:completefunc = 'youcompleteme#Complete'
        endif

        if pyeval( 'ycm_state.NativeFiletypeCompletionUsable()' )
            let &omnifunc = 'youcompleteme#OmniComplete'
            let &l:omnifunc = 'youcompleteme#OmniComplete'

        " If we don't have native filetype support but the omnifunc is set to YCM's
        " omnifunc because the previous file the user was editing DID have native
        " support, we remove our omnifunc.
        elseif &omnifunc == 'youcompleteme#OmniComplete'
            let &omnifunc = ''
            let &l:omnifunc = ''
        endif
    endfunction
endif
{% endhighlight %}

Останавливаться на каждой опции это довольно круто, к тому же большая часть
говорит сама за себя. Часть с SetCompleteFunc, возможно, экспериментальная
и работать не будет из-за некорректной работы ultisnips. Но вообще было бы
интересно проверить возможно ли в динамике переключить youcompleteme на что-то
ещё, видимо так оно там и оказалось. Для youcompleteme характерно некоторое
замедление скорости работы, так что для типов файлов из
*g:ycm_filetype_blacklist* он не используется. Большая часть опций более или
менее стандартные. Не рекомендовал бы использовать preview window, потому что
хоть это и кажется удобным на практике это приводит только к дерганьям
интерфейса и тормозам при работе.

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

## Powerline/Lightline/Airline

<a href="https://github.com/itchyny/lightline.vim.git">itchyny/lightline.vim.git</a>

Статусная строка. Использую в neovim из-за слабой поддержки python, которой
в настоящий момент не достаточно для использования powerline. Также можно
воспользоваться airline, которая вроде бы симпотичнее, но я не заморачиваюсь,
потому что в данный момент я не являюсь активным пользователем neovim, а для
чего-то ещё оно мне и не нужно. Тех кого "современный вид" не устраивает могут
воспользоваться <a href="https://github.com/tpope/vim-flagship">flagship</a>,
который предоставляет нечто более "традиционное" и "минималистичное".

## fzf и другой fuzzy-finding

<a href="https://github.com/junegunn/fzf">junegunn/fzf</a> и 
<a href="https://github.com/junegunn/fzf.vim">junegunn/fzf.vim</a>

Сладкая парочка плагинов, которая добавляет поддержку fzf в vim. Штука эта
очень хорошая, но как по мне недостаточно фичастая. Раньше вроде как был
один, эта ситуация изменилась после перехода fzf с ruby на go.

Что до настройки, то там всё довольно просто, потому что оно поддерживает
взаимодействие через pipe. Парочка интересных моментов, удобных лично для
меня:

{% highlight vim %}
    nnoremap qe :Files %:p:h<CR>
    nnoremap qE :Files<CR>
    let g:fzf_commits_log_options = '--graph --color=always --format="%C(auto)%h%d %s %C(black)%C(bold)%cr"'
{% endhighlight %}

Последнее довольно красивый и удобный поиск по коммитам, иногда это удобнее,
чем обычное окно, если вы собираетесь не работать с проектом, а просто что-то
посмотреть.

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

## Раскраски

### luochen1990/rainbow

Это раскраска скобочек всеми цветами радуги для vim. Достоинства в том что она
есть и вроде как худо-бедно поддерживается. Недостатки как по мне в том что
нельзя ограничить уровень вложенности(мне не по душе когда цвета) начинают
повторяться, также мне не нравится что не всегда можно управлять семантикой, то
есть он просто считает уровень скобок(любых) вне зависимости от их
характеристик. На фоне того что есть для emacs смотрится как-то слабо. Также не
раскрашивает скобки в отдельности, без пар, не позволяет раскрашивать операторы
не так как скобки и проч. В настоящее время не использую, потому что версия от
otommod/rainbow у меня ломает подсветку, а оригинал неправильно
инициализируется(не очень понял почему). Суть того как это выглядит изображена
вот на этой картинке:
![Rainbow](https://i.ytimg.com/vi/3ozPypdOUcM/maxresdefault.jpg)

### chrisbra/colorizer

![Terminal Coloring](https://raw.githubusercontent.com/chrisbra/Colorizer/master/Colorizer.gif)

### Файлы синтаксиса.
Я использую следующие дополнительные файлы для подсветки в vim:

* <a href="https://github.com/vimez/vim-tmux">vim-tmux</a>
* <a href="https://github.com/elzr/vim-json">vim-json</a>
* <a href="https://github.com/cespare/vim-toml">vim-toml</a>
* <a href="https://github.com/rsmenon/vim-mathematica.git">vim-mathematica</a>
* <a href="https://github.com/jelera/vim-javascript-syntax.git">vim-javascript</a>
* <a href="https://github.com/tpope/vim-git">vim-git</a>
* <a href="https://github.com/ekalinin/Dockerfile.vim">Dockerfile</a>
* <a href="https://github.com/jnwhiteh/vim-golang.git">vim-golang</a>
* <a href="https://github.com/farseer90718/vim-regionsyntax">vim-regionsyntax</a>
* <a href="https://github.com/JulesWang/css.vim">css</a>
* <a href="https://github.com/leafo/moonscript-vim">moonscript</a>
* <a href="https://github.com/rodjek/vim-puppet">vim-puppet</a>
* <a href="https://github.com/bbchung/clighter.git">clighter</a>
* <a href="https://github.com/bbchung/Clamp">Clamp</a>

Компания плагинов для синтаксиса разных типов файлов. В общем-то рассказать
особо нечего, кроме последнего. Эти два плагина используют clang для
семантической раскраски. Получается в общем-то даже лучше, чем, в настоящий
момент в том же Clion от Jetbrains. Но с оговорками. В общем оно(подсветка) как
бы пляшет туда сюда и постоянно видны "следы" инкрементальной компиляции. Clamp
возможно работает лучше, но я его пока не тестировал из-за того что neovim
у меня в целом в процессе вялой настройки.

## Интерфейс(TUI)

Плагин для перевода esc-последовательностей в раскраску vim. Иногда это нужно,
а иногда приятно. Больше добавить особо нечего.

### Shougo/neomru.vim.git

Для того чтобы получать список последних использованных файлов. Это очень удобно.
Из достоинств: standalone, так что теоретически можно настроить интеграцию с fzf.
Используется в интеграции с Unite.

### Shougo/unite.vim.git

Комбайн для построения классного TUI для vim, чего там только нет. Прямого
аналога не имеет. Разумеется очень рекомендуется. Как мне сказали
функциональность этого плагина достаточно похожа на helm из emacs. Чтобы
посмотреть на workflow этого плагина можно обратиться к 
<a href="https://www.youtube.com/watch?v=fwqhBSxhGU0&hd=1">следующему  видео</a>

### thinca/vim-qfreplace.git

Довольно необычный способ взаимодействия с Quickfix. для того чтобы понять что
это можно посмотреть вот это видео: https://vimeo.com/24700977


## Поиск

### Shougo/unite-outline

Приблуда к unite для быстрой навигации по файлу. В чем-то удобнее чем tagbar
и иже с ним, но нужно не часто.

### junkblocker/unite-codesearch

Интеграция unite с google code-search. Это форк, так что нужно использовать
именно его, с кастомным codesearch от того же автора. В принципе можно
использовать для поиска по большим базам кода, но это скорее актуально для
Google Inc у которой счет количества строк кода идет на миллиарды.

### Ag и Ack

Для интеграции ack и ag можно поспользоваться вот этой парой:

* <a href="https://github.com/mileszs/ack.vim">mileszs/ack.vim</a>
* <a href="https://github.com/rking/ag.vim.git">rking/ag.vim.git</a>

Интеграция поисковиков ag и ack. Это такие аналоги grep. Честно говоря если
unite работает без них, то не нужно, надо проверить. Для тех кто не слышал
ack отличается более приятным выводом в stdout и нацеленностью именно на
поиск кода. Недостаток в том что более медленный. Также есть ag, который,
насколько мне известно, как правило быстрее чем grep. Когда-то существовал
также silver_search, но в настоящее время не развивается, потому что
все его преимущества уже включены в ag.

## Работа с кодом

### rhysd/vim-clang-format.git

Простой плагин для семантического выравнивания кода. По сути не делает ничего,
просто вызывает clang-format. А вот сам clang-format штука очень хорошая.
Отличается от аналогов тем что учитывает AST. В результате это получается
как по мне куда лучше аналогов вроде astyle и других.

### SirVer/ultisnips.git

Самый крутой движок для сниппетов в vim. Nuff-said, в общем-то.
Безальтернативен из-за поддержки youcompleteme. Обладает довольно
богатыми возможностями по кастомизации, за которыми лучше обратиться
к документации на его github-странице.

### godlygeek/tabular.git

Один из плагинов для автовыравнивания, много функций, поэтому и использую.
Из альтернатив также можно рассмотреть 
<a href="https://github.com/junegunn/vim-easy-align">easy-align</a>
как более простой в использовании.

### Raimondi/delimitMate

Автоподстановка скобок для vim. Сначала кажется что не нужно, потом без этого жить
не можешь. Безальтернативен из-за поддержки в youcompleteme.

### scrooloose/syntastic

Проверка синтаксиса в vim. Одна из тех вещей, которая нужна чтобы сделать vim
похожим на IDE. Также имеет поддержку youcompleteme.

### nathanaelkane/vim-indent-guides

Подветка отступов с помощью изменения цвета background.
Аналоги(https://github.com/Yggdroot/indentLine) не подходят из-за конфликта
с переопределением цвета conceal. Дело в том что по крайней мере для темной
темы вы вряд ли захотите делать отступы яркими цветами. А если сделаете
темными, то пострадает сцет conceal символов, которые могут использоваться
для операторов. По крайней мере я люблю использовать полноценные лямбды
вместо *\* в haskell.

### c9s/vimomni.vim
omni-completion для vim. Иногда работает, иногда не очень. Не нужно если использовать
neocomplete например.

### majutsushi/tagbar
Список тегов, используются довольно часто. Лучше всех прямых аналогов,
но для непосредственного поиска по тегам лучше использовать что-то ещё.

### chrisbra/vim-diff-enhanced.git
Улучшенная реализация diff с более грамотным алгоритмом. Используется редко, но
метко, когда нужно обновить программу, в которой было сделано много изменений
это помогает, особенно если там отличаются отступы. Ещё один плюс в том что
позволяет выбрать алгоритм для diff.

### sombr/vim-scala-worksheet.git
Интересный плагин, который позволяет интерактивно использовать scala c помощью
создания любого файла с расширением \*.scalaws код рассматривается
и выполняется worksheet. Очень удобно в плане работы, подходит даже для
взаимодействия с большими проектами, которые используют akka. Кстати не совместима со
многими плагинами и поддерживается плохо, но по крайней мере очень приятна в плане
использования. Для этого плагина лучше всего использовать отдельный instance vim, с
отдельным упрощенным конфигом и набором плагинов для него.

### ensime/ensime-vim
Система для поддержки scala для разных редакторов, в том числе vim. Установка
и настройка требует некоторых телодвижений, но в целом удобно. Поддерживает
отладку, навигацию, семантическую подсветку, автодополнение, проверку типов
и семантическое форматирование, в общем мастхэв для скалиста.

### derekwyatt/vim-scala

### derekwyatt/vim-sbt

Добавляет некоторую простую поддержку scala и sbt в vim без автодополнения
и прочих фич IDE. Если использовать ensime, то возможно большого смысла
в этих плагинах и нет, надо подумать.

### tpope/vim-commentary.git
Простой авторасстановщик комментариев. Выделели область или textobj,
скомандовали gc и готово! Очень просто и удобно. От аналогов, насколько
я помню, отличается большей простотой. Можно легко добавлять поддержку
собственных типов через автокоманды.

Альтернативы:

* <a href="https://github.com/scrooloose/nerdcommenter">scrooloose/nerdcommenter</a>
* <a href="https://github.com/tomtom/tcomment_vim">tomtom/tcomment</a>

### tpope/vim-endwise
Автоматическое добавление парных слов для разных языков. Достаточно удобно, 
хотя и не немного дебильное, не помешала бы поддержка семантики, интересно
узнать есть ли что-то лучше. Активно поддерживается.

### tpope/vim-unimpaired.git
Всякие разнообразные кастомизации от Tim Pope. Довольно неплохая совместимость
с существующими плагинами, так что всё ок, лучше обратиться к документации за
подробностями.

### dbakker/vim-projectroot
Удобный детект текущей директории проекта. Нужно, поскольку autochdir не всегда
корректно работает для автокоманд vim, а ручной переход медленно и лениво,
а так нажали cd в режиме редактирования и готово! Детект происходит благодаря
директориям вроде *.git* и прочим, также это поведение можно легко
кастомизировать или даже переписать, потому что плагин не сложный.

### Valloric/ListToggle.git
Простой плагин для вкл/выкл quickfix window. У меня забинджено на qq.
Используется частенько.

### derekwyatt/vim-fswitch.git
Простой плагин для переключения между сорцом и хедером. У меня забинджено на
control+a в режиме редактирования. Используется достаточно часто. Из
недостатков: по-видимому не хватает guard'а на тот случай, если данный тип
файлов не поддерживается, надо допилить.

* <a href="https://github.com/vim-scripts/IndentConsistencyCop.git">vim-scripts/IndentConsistencyCop.git</a>
* <a href="https://github.com/hynek/vim-python-pep8-indent.git">hynek/vim-python-pep8-indent.git</a>
* <a href="https://github.com/fs111/pydoc.vim">fs111/pydoc.vim</a>

Всякие мелкие плагины для поддержки python, которые не связаны с автодополнением.
Как уже было написано в моей предыдущей статье с этим справится YouCompleteMe

### nosami/Omnisharp.git
Автодополнение для c#, есть поддержка в youcompleteme. Требует создания или
файла проекта.

### Go
Для поддержки go используется:

* <a href="https://github.com/jstemmer/gotags.git">jstemmer/gotags.git</a>
* <a href="https://github.com/Blackrush/vim-gocode.git">Blackrush/vim-gocode.git</a>
* <a href="https://github.com/fatih/vim-go.git">fatih/vim-go.git</a>
Теги для go и некоторая поддержка для этого языка, в частности автодополнение.

### Rust
Для поддерки rust я использую:
* <a href="https://github.com/rust-lang/rust.vim">rust-lang/rust.vim</a>
* <a href="https://github.com/jtdowney/vimux-cargo">jtdowney/vimux-cargo</a>

Поддержка детекта rust, улучшенный(на момент написания поста) syntax
highlighting для rust, а также поддержка асинхронной работы cargo с через tmux.
Также поддерживается семантическое форматирование.

Также используются плагины для простой поддержки языков perl, dot(graphviz),
supercollider, erlang и lua. Из последнего поддержка lua получше, но конечно не
такая хорошая как хотелось бы:

* <a href="https://github.com/vim-perl/vim-perl">vim-perl/vim-perl</a>
* <a href="https://github.com/wannesm/wmgraphviz.vim">wannesm/wmgraphviz.vim</a>
* <a href="https://github.com/sbl/scvim.git">sbl/scvim.git</a>
* <a href="https://github.com/xolox/vim-lua-ftplugin.git">xolox/vim-lua-ftplugin.git</a>
* <a href="https://github.com/oscarh/vimerl">oscarh/vimerl</a>

### janko-m/vim-test.git
Универсальный плагин для тестирования на разных языках.

### tpope/vim-dispatch.git
Очень полезный плагин, который я воспринимаю прежде всего как 
middleware для интеграции различных программ, tmux и vim. Как результат
это позволяет получить удобную и красивую асинхронную компиляцию, которая
не затрагивает текущий инстанс вима и не мешает работе в нем. При этом по
завершении работы мы можем видеть скажем результаты ошибок и вывода make и
других команд в quickfix с удобной навигацией по всему этому. Очень удобно,
нужно и замечательно, наряду с youcompleteme.

### lyuts/vim-rtags.git
Простая поддержка rtags. До emacs'овской пока не дотягивает, но в перспективе
должно расширить возможности рефакторинга. Несколько сложновато в настройке
и использовании и требует перекомпиляции.
### Haskell

Для Haskell я использую следующие плагины:

* <a href="https://github.com/ujihisa/neco-ghc">ujihisa/neco-ghc</a>
* <a href="https://github.com/eagletmt/ghcmod-vim.git">eagletmt/ghcmod-vim</a>
* <a href="https://github.com/bitc/vim-hdevtools">bitc/vim-hdevtools</a>
* <a href="https://github.com/neg-serg/vim2hs">neg-serg/vim2hs</a>

Пачка плагинов для поддержки haskell в vim. neco-ghc используется для автодополения,
ghcmod и hdevtools добавляют всякие удобные фичи связанные с типами и т.д., также я
использую собственный небольшой форк vim2hs с улучшенным indent'ом, потому что стандартный,
то есть тот который входит в vim2hs, мне не нравится.

### Ruby

Для Ruby я использую следующие плагины:

* <a href="https://github.com/vim-ruby/vim-ruby">vim-ruby/vim-ruby</a>
* <a href="https://github.com/tpope/vim-rails.git">tpope/vim-rails</a>
* <a href="https://github.com/tpope/vim-rake.git">tpope/vim-rake</a>
* <a href="https://github.com/tpope/vim-rbenv.git">tpope/vim-rbenv</a>
* <a href="https://github.com/tpope/vim-bundler">tpope/vim-bundler</a>
* <a href="https://github.com/vim-scripts/dbext.vim">vim-scripts/dbext</a>
* <a href="https://github.com/skalnik/vim-vroom">skalnik/vim-vroom</a>

Набор плагинов для ruby. Для автодополнения я использую vim-ruby, некоторые
используют vim-monster. Мне если честно не очень нравятся результаты, которые
он выдает, так что я его не использую, но, вероятно пока что его использование
может иметь смысл, потому что vim-ruby в neovim точно не работает, а вот
vim-monster я ещё не проверял.

### shawncplus/phpcomplete.vim.git
Автодополнение есть и для php.

### joonty/vdebug
Интерейс дебаггера для множества языков, который тестировался как минимум для
PHP, Python, Ruby, Perl, Tcl и NodeJS. Как утверждает автор возможно
взаимодействие с любым, который использует DBGP протокол, в частности Xdebug
для PHP. В очереди на проверку, честно говоря мне это было как-то особо не
нужно, потому я что я в этих случаях всегда пользовался командной строкой
и gdb, а для других языков мне это было не нужно. Тем не менее штука интересная
и на неё стоит обратить внимание.

### othree/html5.vim
Плагин для поддержки html5. Толком не помню что он там делает :D

### Теги
Вот что я исплользую для работы с тегами:

* <a href="https://github.com/szw/vim-tags">szw/vim-tags</a>
* <a href="https://github.com/yuki777/gtags.vim.git">yuki777/gtags.vim</a>
* <a href="https://github.com/bbchung/gasynctags.git">bbchung/gasynctags</a>
* <a href="https://github.com/tranngocthachs/gtags-cscope-vim-plugin.git">tranngocthachs/gtags-cscope-vim-plugin</a>

Vim-tags это автогенерация тегов для ctags. Не использую, видимо добавил для
порядка, потому что easytags люто, бешено тормозит весь вим, а для всего
остального я всё равно использую gtags, благо он поддерживает большое
количество языков и нравится мне намного больше. gtags это просто стандартный
плагин, который идет вместе с дистрибутивом gnu global. В целом всю эту кухню
я вроде как довольно подробно описывал в своей статье "vim как ide".

### lervag/vimtex
Простая, но работающая latex в vim. Не auctex для emacs, конечно, но пойдет. По
крайней мере диплом на около 100 страниц без воды был написан с помощью этого
плагина.

### rstacruz/sparkup.git
Позволяет быстро набивать html-код с помощью чего-то вроде сниппетов. Плагин
интересный, но он конфликтует с моей emacs-like раскладкой vim'а. Для того
чтобы это устранить достаточно добавить в конфиг что-то вроде вот этого:
{% highlight vim %}
if neobundle#tap('rainbow')
    let g:sparkupMapsNormal = 0 "default = 0
    let g:sparkupMaps = 1 "default = 1
    let g:sparkupExecuteMapping = "<m-i>" "default = <C-e>
endif
{% endhighlight %}
Тогда по умолчанию хоткеем для разворачивания сниппетов будет alt+i.

### Valloric/vim-instant-markdown
Автоматически позволяет видеть изменения в файле по мере его редактирования(сохранения).
Я где-то это использовал, но сейчас оно мне особо не надо, в частности в этом блоге используется
jekyll, который тоже использует markdown, но смысла во всем этом нет, потому что содержимое браузера
и так автообновляется средствами самого jekyll, а для гитхаба я сложные README пока не делал.

### marijnh/tern_for_vim
Очень хорошая поддержка js, когда я тестировал прекрасно работало. Семантическое автодополнение.
Возможно сейчас это не особо нужно из-за поддержки в youcompleteme, надо потестировать.

## Поддержка git

Для интеграции git в vim я использую следующие плагины:

<a href="https://github.com/tpope/vim-fugitive.git">vim-fugitive</a>
<a href="https://github.com/junegunn/vim-github-dashboard.git">vim-github-dashboard</a>
<a href="https://github.com/jaxbot/github-issues.vim.git">github-issues</a>
<a href="https://github.com/idanarye/vim-merginal.git">vim-merginal</a>
<a href="https://github.com/junegunn/gv.vim">junegunn/gv</a>
<a href="https://github.com/vim-scripts/DirDiff.vim.git">DirDiff.vim</a>
<a href="https://github.com/airblade/vim-gitgutter.git">vim-gitgutter</a>

Пачка плагинов для поддержки git в vim. Здесь самый главный это fugitive,
также мне очень нравится смотреть последние коммиты с помощью GV. Думаю добавить
это в интеграцию одной своей приблуды, про которую я расскажу чуть позже, чтобы
использовать вместо tig. Mergial это простой плагин для того чтобы разбираться
с конфликтами слияний. В общем-то он так прост, что и рассказать особо нечего,
остальные плагины по желанию. Про vim-fugitive я уже говорил, с ним наоборот
лучше обратиться к официальной документации.

## Tmux

Вот что я использую для интеграции с tmux:

* <a href="https://github.com/tpope/vim-tbone.git">vim-tbone</a>
* <a href="https://github.com/benmills/vimux.git">vimux</a>
* <a href="https://github.com/christoomey/vim-tmux-navigator">vim-tmux-navigator</a>
* <a href="https://github.com/epeli/slimux">slimux</a>

Различные плагины для взаимодействия с tmux. Из них выделю
christoomey/vim-tmux-navigator который позволяет прыгать между буферами tmux
с помощью тех же хоткеев, что и между vim. И прозрачно. Словно это одна и та же
сущность. Slimux это что-то вроде реинкарнации slime. Можно использовать,
удобно. Хотя в общем-то уже есть dispatch.

## Мелочи

### Shougo/junkfile.vim.git

Удобное создание временных файлов. Прелесть этого плагина 
мне ещё предстоит на себе испытать, но в целом нужно.

### Shougo/neossh.vim.git
Для удобной работы с буферами через ssh. Помнится встроенный netrw мне в этом
плане показался более сложным и неудобным, а благодаря этому плагину всё ок.

### vim-scripts/ViewOutput.git
Более простой и удобный способ перенаправления команд vim, чем стандартный
redir. Никем давно не поддерживается, но это и не нужно. Использование очень
простое достаточно вбить что-то вроде
{% highlight vim %}
:VO au
:VO let
{% endhighlight %}

### tpope/vim-repeat

Повторение через точку для большего количества команд, чем по умолчанию
предусмотрено в vim. Собственно имеется в виду поддержка некоторых 
плагинов vim, в частности surround, speeddating, abolish, unimpaired,
commentary и vim-easyclip, большая часть из которых от того же автора,
то есть Tim Pope. Используется не очень-то активно.

### tpope/vim-eunuch.git

Дополнительные встроенные в vim команды. Мой любимчик это SudoWrite, потому что
я всё предпочитаю редактировать через один и тот же instance vim'а.

### sjl/gundo.vim

Дерево отмены. Нужно, используется частенько, когда нужно отменить что-то после
redo. Использование простое и наглядное, не вижу смысла что-то писать, очень
полезный хороший плагин.


### chrisbra/unicode.vim
Улучшенная поддержка диграфов в vim. На любителя.
![Better unicode support](https://camo.githubusercontent.com/1cf1562c8ebf06decd05537b5c51d7081b307224/68747470733a2f2f63687269736272612e6769746875622e696f2f76696d2d73637265656e63617374732f756e69636f64652d73637265656e636173742e676966)

### Улучшенный gf
Различные плагины, которые направлены на улучшение поддержки команды gf для
навигации по всяким файлам и прочим сущностям. Для тех кто в танке эта
команда по умолчанию позволяет прыгнуть на файл под курсором. Вот они:

* <a href="https://github.com/kana/vim-gf-user.git">kana/vim-gf-user.git</a>
* <a href="https://github.com/kana/vim-gf-diff.git">kana/vim-gf-diff.git</a>
* <a href="https://github.com/mattn/gf-user-vimfn.git">mattn/gf-user-vimfn.git</a>
* <a href="https://github.com/mkomitee/vim-gf-python.git">mkomitee/vim-gf-python.git</a>
* <a href="https://github.com/drmikehenry/vim-fixkey">drmikehenry/vim-fixkey</a>

Очень полезный плагин, для добавления поддерки mod1(alt) хоткеев в консольный вим.
В neovim это уже не нужно. Используется прежде всего для поддержки моих хоткеев,
которые переносят часть из emacs(вроде c-a в начало строки), о которых стоит
рассказать отдельно.

### ReekenX/vim-rename2.git
Переименование текущего файла прямо из vim. Иногда нужно, но можно обойтись и без него,
если для вас критично количество плагинов, которые вы используете.

### thinca/vim-ref.git
Расширение функциональности просмотра документации.

### othree/eregex.vim
Улучшенный поисковик с поддержкой pcre регулярных выражений. Лично я использую
редко, потому что как правило когда попадаю в такую ситуацию, то я в командной
строке, а не виме.

### chrisbra/Join.git
Плагин, который ускоряет команду join. Также можно немного кастомизировать.
Нужно проверить так ли это актуально в настоящий момент.

### lyokha/vim-xkbswitch.git
Плагин, который вкупе с определенными настройками дает более удобную работу
с русской раскладкой в vim. Идеал не достигнут. Чтобы его получить можно
например написать кастомный переключатель, который работает без определений
вроде

{% highlight sh %}
setxkbmap -option keypad:pointerkeys -layout 'us,ru' -variant altgr-intl \
    -option 'grp:alt_shift_toggle' -option ctrl:nocaps
{% endhighlight %}

И определяет текущее окно, для которого делать триггер раскладки vim вместо
того чтобы дергать системную раскладку. Тем не менее подход с control+s,
который используется у меня позволяет использовать все нужные хоткеи без
проблем и есть не просит.

Также есть плагин 
### kana/vim-arpeggio.git
Который позволяет отрабатывать одновременные нажатия клавиш. У меня это
используется для сочетания jk, которое заменяет мне esc. Это нужно для того
чтобы буква j не создавала задержку ожидания следующей, выглядит это очень
неприятно, особенно если как у меня на эту операцию отсутствует таймаут. Есть
некоторая несовместимость по отношению к русской раскладке. К счастью
в большинстве случаев можно воспользоваться control+c или control+]
в зависимости от того что нужно и не тянуться к esc.

### tpope/vim-surround
Довольно известный плагин для vim, который позволяет с помощью одной команды
добавлять и убирать скобки. Также обладает расширяемостью под разные паттерны.
Иногда полезно.

### jamessan/vim-gnupg.git
Улучшенная поддержка gpg в vim. Нужно в некоторых специфических случаях.

### Shougo/echodoc.vim
Выводит всякую информацию вроде прототипов функции или чего-то ещё в область
echo(это та, которая под статусной строкой). Кого-то это может раздражать, так
что ставить по-желанию.

### blindFS/vim-taskwarrior
Очень классная интеграция taskwarrior в vim. Представляет собой упрощенный аналог 
org-mode emacs. Лично мне очень по душе. Просто и полезно. Как по мне главный плюс
это более узкая направленность, простота и то что сама утилита standalone.
Выглядит это чудо вот так:
![taskwarriorgif](https://camo.githubusercontent.com/fdae507403d80262534930152a555fc66f1f4c32/687474703a2f2f7461736b6578747261732e6f72672f6174746163686d656e74732f646f776e6c6f61642f3635352f32303133313131305f3030323735332e676966)

### kopischke/vim-fetch
Удобный прыжок на определенное место в файле, по тому же принципу как это
делается из командной строки, иногда бывает нужно. Основное достоинство в том
что работает везде.

### FooSoft/vim-argwrap
Команда для "схлопывания" и "захлопывания" содержимого скобок,
лично я использую достаточно часто.

Специально вместе с gvim исппользуются:

* <a href="drmikehenry/vim-fontsize.git">drmikehenry/vim-fontsize.git</a>
* <a href="dtyru/restart.vim.git">dtyru/restart.vim.git</a>
* <a href="dvim-scripts/utl.vim.git">dvim-scripts/utl.vim.git</a>
* <a href="dbling/vim-airline.git">dbling/vim-airline.git</a>

Компания плагинов, которые относились к gvim, который я больше не использую, главным
образом из-за отсутствия интеграции с tmux. Тем не менее расскажу что это для порядка.
vim-fontsize позволяет динамически изменять размер шрифта. Насколько я помню там эта
фича реализована была только для gui, но в любом случае оно для консоли не очень-то и
нужно, потому что это можно сделать например с помощью esc-последовательностей и
perl скриптов для случая rxvt-unicode или дополнительных патчей в случае suckless terminal

### mopp/autodirmake.vim.git

Небольшой плагин для автосоздания директорий. Удобно.

### Shougo/vimshell.vim

Shell внутри vim. Честно говоря особо и не нужно. В neovim и так встроено. Пожалуй от его
использования я откажусь в будущем. Из плюсов можно выделить интеграцию с самим вимом
и его командами, для кого-то работа с ним может показаться приятнее, чем с vimfiler,
хотя они и призваны дополнять друг друга.

### ryanoasis/vim-devicons.git
Удивительный плагин, который добавляет что-то вроде поддержки иконок в vim.
Весь этот процесс целая история, которую надо рассказывать, но результат
выглядит приятно. Должен быть загружен последним.
