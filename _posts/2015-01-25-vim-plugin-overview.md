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
порядку. Заодно покажу как они у меня настроены, с некоторыми рекомендациями.

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
не так как скобки и проч. В настоящее время не использую, потому что версия от
otommod/rainbow у меня ломает подсветку, а оригинал неправильно
инициализируется(не очень понял почему). Суть того как это выглядит изображена
вот на этой картинке:
![Rainbow](https://i.ytimg.com/vi/3ozPypdOUcM/maxresdefault.jpg)

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
Используется в интеграции с Unite.

{% highlight vim %}
NeoBundle 'Shougo/unite.vim.git' "unite for creating ui
{% endhighlight %}

Комбайн для построения классного TUI для vim, чего там только нет. Прямого
аналога не имеет. Разумеется очень рекомендуется. Как мне сказали
функциональность этого плагина достаточно похожа на helm из emacs. Чтобы
посмотреть на workflow этого плагина можно обратиться к 
<a href="https://www.youtube.com/watch?v=fwqhBSxhGU0&hd=1">следующему  видео</a>

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

Интеграция unite с google code-search. Это форк, так что нужно использовать
именно его, с кастомным codesearch от того же автора. В принципе можно
использовать для поиска по большим базам кода, но это скорее актуально для
Google Inc у которой счет количества строк кода идет на миллиарды.

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
Отличается от аналогов тем что учитывает AST. В результате это получается
как по мне куда лучше аналогов вроде astyle и других.

{% highlight vim %}
NeoBundle 'SirVer/ultisnips.git' "Snippets with ycm compatibility
{% endhighlight %}

Самый крутой движок для сниппетов в vim. Nuff-said, в общем-то.
Безальтернативен из-за поддержки youcompleteme. Обладает довольно
богатыми возможностями по кастомизации, за которыми лучше обратиться
к документации на его github-странице.

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
unite работает без них, то не нужно, надо проверить. Для тех кто не слышал
ack отличается более приятным выводом в stdout и нацеленностью именно на
поиск кода. Недостаток в том что более медленный. Также есть ag, который,
насколько мне известно, как правило быстрее чем grep. Когда-то существовал
также silver_search, но в настоящее время не развивается, потому что
все его преимущества уже включены в ag.

{% highlight vim %}
NeoBundleLazy 'tpope/vim-repeat', { 'mappings' : '.' } "dot for everything
{% endhighlight %}

Повторение через точку для большего количества команд, чем по умолчанию
предусмотрено в vim. Собственно имеется в виду поддержка некоторых 
плагинов vim, в частности surround, speeddating, abolish, unimpaired,
commentary и vim-easyclip, большая часть из которых от того же автора,
то есть Tim Pope. Используется не очень-то активно.

{% highlight vim %}
NeoBundle 'tpope/vim-eunuch.git' "for SudoWrite, Locate, Find etc
{% endhighlight %}

Дополнительные встроенные в vim команды. Мой любимчик это SudoWrite, потому что
я всё предпочитаю редактировать через один и тот же instance vim'а.

{% highlight vim %}
NeoBundleLazy 'sjl/gundo.vim', { 'commands' : 'GundoToggle' }
{% endhighlight %}

Дерево отмены. Нужно, используется частенько, когда нужно отменить что-то после
redo. Использование простое и наглядное, не вижу смысла что-то писать, очень
полезный хороший плагин.

{% highlight vim %}
NeoBundleLazy 'Raimondi/delimitMate', { 'insert' : 1 } "autopair ()[]{}
{% endhighlight %}

Автоподстановка скобок для vim. Сначала кажется что не нужно, потом без этого жить
не можешь. Безальтернативен из-за поддержки в youcompleteme.

{% highlight vim %}
NeoBundleLazy 'scrooloose/syntastic', { 'insert' : 1 } "syntax checker
{% endhighlight %}

Проверка синтаксиса в vim. Одна из тех вещей, которая нужна чтобы сделать vim
похожим на IDE. Также имеет поддержку youcompleteme.

{% highlight vim %}
NeoBundle 'nathanaelkane/vim-indent-guides' "indent tabs visually with |-es too slow
{% endhighlight %}
Подветка отступов с помощью изменения цвета background.
Аналоги(https://github.com/Yggdroot/indentLine) не подходят из-за конфликта
с переопределением цвета conceal. Дело в том что по крайней мере для темной
темы вы вряд ли захотите делать отступы яркими цветами. А если сделаете
темными, то пострадает сцет conceal символов, которые могут использоваться
для операторов. По крайней мере я люблю использовать полноценные лямбды
вместо *\* в haskell.

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
остальные плагины по желанию. Про vim-fugitive я уже говорил, с ним наоборот
лучше обратиться к официальной документации.

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
![Better unicode support](https://camo.githubusercontent.com/1cf1562c8ebf06decd05537b5c51d7081b307224/68747470733a2f2f63687269736272612e6769746875622e696f2f76696d2d73637265656e63617374732f756e69636f64652d73637265656e636173742e676966)

{% highlight vim %}
NeoBundle 'Shougo/neossh.vim.git' "work with ssh easier
{% endhighlight %}
Для удобной работы с буферами через ssh. Помнится встроенный netrw мне в этом
плане показался более сложным и неудобным, а благодаря этому плагину всё ок.

{% highlight vim %}
NeoBundle 'vim-scripts/ViewOutput.git' "VO commandline output
{% endhighlight %}
Более простой и удобный способ перенаправления команд vim, чем стандартный
redir. Никем давно не поддерживается, но это и не нужно. Использование очень
простое достаточно вбить что-то вроде
{% highlight vim %}
:VO au
:VO let
{% endhighlight %}

{% highlight vim %}
NeoBundle 'kana/vim-gf-user.git' "framework open file by context
NeoBundle 'kana/vim-gf-diff.git' "go to the changed block under the cursor from diff output
NeoBundle 'mattn/gf-user-vimfn.git' "vim-gf-user extension: jump Vim script function
NeoBundle 'mkomitee/vim-gf-python.git' "gf for python
{% endhighlight %}
Различные плагины, которые направлены на улучшение поддержки команды gf для
навигации по всяким файлам и прочим сущностям. Для тех кто в танке эта
команда по умолчанию позволяет прыгнуть на файл под курсором.

{% highlight vim %}
" There is no need in fixkey for nvim because of it's default behaviour
if !has("nvim")
    NeoBundle 'drmikehenry/vim-fixkey' "fixes key codes for console Vim
endif
{% endhighlight %}

Очень полезный плагин, для добавления поддерки mod1(alt) хоткеев в консольный вим.
В neovim это уже не нужно. Используется прежде всего для поддержки моих хоткеев,
которые переносят часть из emacs(вроде c-a в начало строки), о которых стоит
рассказать отдельно.

{% highlight vim %}
NeoBundle 'ReekenX/vim-rename2.git' "rename for files even with spaces in filename
{% endhighlight %}
Переименование текущего файла прямо из vim. Иногда нужно, но можно обойтись и без него,
если для вас критично количество плагинов, которые вы используете.

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
{% highlight vim %}
NeoBundle 'kana/vim-arpeggio.git' "mappings for simultaneously pressed keys
{% endhighlight %}

Который позволяет отрабатывать одновременные нажатия клавиш. У меня это
используется для сочетания jk, которое заменяет мне esc. Это нужно для того
чтобы буква j не создавала задержку ожидания следующей, выглядит это очень
неприятно, особенно если как у меня на эту операцию отсутствует таймаут. Есть
некоторая несовместимость по отношению к русской раскладке. К счастью
в большинстве случаев можно воспользоваться control+c или control+]
в зависимости от того что нужно и не тянуться к esc.

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
echo(это та, которая под статусной строкой). Кого-то это может раздражать, так
что ставить по-желанию.

{% highlight vim %}
if executable(resolve(expand("task")))
    NeoBundle 'blindFS/vim-taskwarrior' "add taskwarrior vim plug wrapper
endif
{% endhighlight %}
Очень классная интеграция taskwarrior в vim. Представляет собой упрощенный аналог 
org-mode emacs. Лично мне очень по душе. Просто и полезно. Как по мне главный плюс
это более узкая направленность, простота и то что сама утилита standalone.
Выглядит это чудо вот так:
![taskwarriorgif](https://camo.githubusercontent.com/fdae507403d80262534930152a555fc66f1f4c32/687474703a2f2f7461736b6578747261732e6f72672f6174746163686d656e74732f646f776e6c6f61642f3635352f32303133313131305f3030323735332e676966)

{% highlight vim %}
NeoBundle 'kopischke/vim-fetch' "vim path/to/file.ext:12:3
{% endhighlight %}
Удобный прыжок на определенное место в файле, по тому же принципу как это
делается из командной строки, иногда бывает нужно. Основное достоинство в том
что работает везде.

{% highlight vim %}
NeoBundle 'FooSoft/vim-argwrap' "vim arg wrapper
{% endhighlight %}
Команда для "схлопывания" и "захлопывания" содержимого скобок,
лично я использую достаточно часто.

{% highlight vim %}
NeoBundleLazy 'majutsushi/tagbar', { 'commands' : 'TagbarToggle' }
{% endhighlight %}
Список тегов, используются довольно часто. Лучше всех прямых аналогов,
но для непосредственного поиска по тегам лучше использовать что-то ещё.

{% highlight vim %}
NeoBundle 'chrisbra/vim-diff-enhanced.git' "patience diff
{% endhighlight %}
Улучшенная реализация diff с более грамотным алгоритмом. Используется редко, но
метко, когда нужно обновить программу, в которой было сделано много изменений
это помогает, особенно если там отличаются отступы. Ещё один плюс в том что
позволяет выбрать алгоритм для diff.

{% highlight vim %}
NeoBundle 'sombr/vim-scala-worksheet.git' "tiny Vim plugin that turns your file into interactive worksheet
{% endhighlight %}
Интересный плагин, который позволяет интерактивно использовать scala c помощью
создания любого файла с расширением \*.scalaws код рассматривается
и выполняется worksheet. Очень удобно в плане работы, подходит даже для
взаимодействия с большими проектами, которые используют akka. Кстати не совместима со
многими плагинами и поддерживается плохо, но по крайней мере очень приятна в плане
использования. Для этого плагина лучше всего использовать отдельный instance vim, с
отдельным упрощенным конфигом и набором плагинов для него.

{% highlight vim %}
NeoBundle 'ensime/ensime-vim' "scala vim autocompletion
{% endhighlight %}
Система для поддержки scala для разных редакторов, в том числе vim. Установка
и настройка требует некоторых телодвижений, но в целом удобно. Поддерживает
отладку, навигацию, семантическую подсветку, автодополнение, проверку типов
и семантическое форматирование, в общем мастхэв для скалиста.

{% highlight vim %}
NeoBundle 'derekwyatt/vim-scala' "various initial scala support for vim
NeoBundle 'derekwyatt/vim-sbt' "basic SBT support for vim
{% endhighlight %}
Добавляет некоторую простую поддержку scala и sbt в vim без автодополнения
и прочих фич IDE. Если использовать ensime, то возможно большого смысла
в этих плагинах и нет, надо подумать.

{% highlight vim %}
NeoBundle 'tpope/vim-commentary.git' "try it instead of tcomment
{% endhighlight %}
Простой авторасстановщик комментариев. Выделели область или textobj,
скомандовали gc и готово! Очень просто и удобно. От аналогов, насколько
я помню, отличается большей простотой. Можно легко добавлять поддержку
собственных типов через автокоманды.

{% highlight vim %}
NeoBundle 'tpope/vim-endwise' "to insert endif for if, end for begin and so on
{% endhighlight %}
Автоматическое добавление парных слов для разных языков. Достаточно удобно, 
хотя и не немного дебильное, не помешала бы поддержка семантики, интересно
узнать есть ли что-то лучше. Активно поддерживается.

{% highlight vim %}
NeoBundle 'tpope/vim-unimpaired.git' "good mappings and toggles
{% endhighlight %}
Всякие разнообразные кастомизации от Tim Pope. Довольно неплохая совместимость
с существующими плагинами, так что всё ок, лучше обратиться к документации за
подробностями.

{% highlight vim %}
NeoBundle 'dbakker/vim-projectroot' "better rooter
{% endhighlight %}
Удобный детект текущей директории проекта. Нужно, поскольку autochdir не всегда
корректно работает для автокоманд vim, а ручной переход медленно и лениво,
а так нажали cd в режиме редактирования и готово! Детект происходит благодаря
директориям вроде *.git* и прочим, также это поведение можно легко
кастомизировать или даже переписать, потому что плагин не сложный.

{% highlight vim %}
NeoBundle 'Valloric/ListToggle.git' "toggle quickfix and location list <leader>l by def
{% endhighlight %}
Простой плагин для вкл/выкл quickfix window. У меня забинджено на qq.
Используется частенько.

{% highlight vim %}
NeoBundle 'derekwyatt/vim-fswitch.git' "switching between companion source files (e.g. .h and .cpp)
{% endhighlight %}
Простой плагин для переключения между сорцом и хедером. У меня забинджено на
control+a в режиме редактирования. Используется достаточно часто. Из
недостатков: по-видимому не хватает guard'а на тот случай, если данный тип
файлов не поддерживается, надо допилить.

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
Автодополнение для c#, есть поддержка в youcompleteme. Требует создания или
файла проекта.

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
Поддержка детекта rust, улучшенный(на момент написания поста) syntax
highlighting для rust, а также поддержка асинхронной работы cargo с через tmux.
Также поддерживается семантическое форматирование.

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
{% endhighlight %}
Очень полезный плагин, который я воспринимаю прежде всего как 
middleware для интеграции различных программ, tmux и vim. Как результат
это позволяет получить удобную и красивую асинхронную компиляцию, которая
не затрагивает текущий инстанс вима и не мешает работе в нем. При этом по
завершении работы мы можем видеть скажем результаты ошибок и вывода make и
других команд в quickfix с удобной навигацией по всему этому. Очень удобно,
нужно и замечательно, наряду с youcompleteme.

{% highlight vim %}
if executable(resolve(expand("rc")))
    NeoBundle 'lyuts/vim-rtags.git' "rtags plugin for vim
endif
{% endhighlight %}
Простая поддержка rtags. До emacs'овской пока не дотягивает, но в перспективе
должно расширить возможности рефакторинга. Несколько сложновато в настройке
и использовании и требует перекомпиляции.

{% highlight vim %}
if executable(resolve(expand("ghci")))
    NeoBundle 'ujihisa/neco-ghc' "autocomplete for hs using ghc-mod
    NeoBundle 'eagletmt/ghcmod-vim.git'
    NeoBundle 'bitc/vim-hdevtools' "type-related features
    NeoBundle 'neg-serg/vim2hs' "better haskell syntax hi with better indenting
endif            
{% endhighlight %}
Пачка плагинов для поддержки haskell в vim. neco-ghc используется для автодополения,
ghcmod и hdevtools добавляют всякие удобные фичи связанные с типами и т.д., также я
использую собственный небольшой форк vim2hs с улучшенным indent'ом, потому что стандартный,
то есть тот который входит в vim2hs, мне не нравится.

{% highlight vim %}
if executable(resolve(expand("ruby")))
    NeoBundle 'vim-ruby/vim-ruby' "ruby autocompletion
    NeoBundle 'tpope/vim-rails.git' "rails plugin from Tim Pope
    NeoBundle 'tpope/vim-rake.git' "ruby rake support
    NeoBundle 'tpope/vim-rbenv.git' "ruby rbenv support
    NeoBundle 'tpope/vim-bundler' "ruby bundler support
    NeoBundle 'vim-scripts/dbext.vim' "provides database access to many dbms
    NeoBundle 'skalnik/vim-vroom' "plugin to run ruby tests
endif
{% endhighlight %}
Набор плагинов для ruby. Для автодополнения я использую vim-ruby, некоторые
используют vim-monster. Мне если честно не очень нравятся результаты, которые
он выдает, так что я его не использую, но, вероятно пока что его использование
может иметь смысл, потому что vim-ruby в neovim точно не работает, а вот
vim-monster я ещё не проверял.

{% highlight vim %}
NeoBundle 'shawncplus/phpcomplete.vim.git' "better than default phpcomplete.vim
{% endhighlight %}
Автодополнение есть и для php.

{% highlight vim %}
" Multi-language DBGP debugger client for Vim (PHP, Python, Perl, Ruby, etc.)
NeoBundleLazy 'joonty/vdebug', { 'autoload': { 'commands': 'VdebugStart' }}
{% endhighlight %}
Интерейс дебаггера для множества языков, который тестировался как минимум для
PHP, Python, Ruby, Perl, Tcl и NodeJS. Как утверждает автор возможно
взаимодействие с любым, который использует DBGP протокол, в частности Xdebug
для PHP. В очереди на проверку, честно говоря мне это было как-то особо не
нужно, потому я что я в этих случаях всегда пользовался командной строкой
и gdb, а для других языков мне это было не нужно. Тем не менее штука интересная
и на неё стоит обратить внимание.

{% highlight vim %}
NeoBundleLazy 'othree/html5.vim' , {'autoload': {'filetypes': ['html', 'htmldjango']} }
{% endhighlight %}
Плагин для поддержки html5. Толком не помню что он там делает :D

{% highlight vim %}
NeoBundle 'szw/vim-tags' "autogen ctags
if executable(resolve(expand("gtags")))
    NeoBundle 'yuki777/gtags.vim.git' "Gtags v0.64
    NeoBundle 'bbchung/gasynctags.git' "autogenerate gtags to cscope db
    NeoBundle 'tranngocthachs/gtags-cscope-vim-plugin.git' "gtags-cscope navigation
endif
{% endhighlight %}
Vim-tags это автогенерация тегов для ctags. Не использую, видимо добавил для
порядка, потому что easytags люто, бешено тормозит весь вим, а для всего
остального я всё равно использую gtags, благо он поддерживает большое
количество языков и нравится мне намного больше. gtags это просто стандартный
плагин, который идет вместе с дистрибутивом gnu global. В целом всю эту кухню
я вроде как довольно подробно описывал в своей статье "vim как ide".

{% highlight vim %}
NeoBundle 'lervag/vimtex' "LaTeX-Box replacement
{% endhighlight %}
Простая, но работающая latex в vim. Не auctex для emacs, конечно, но пойдет. По
крайней мере диплом на около 100 страниц без воды был написан с помощью этого
плагина.

{% highlight vim %}
NeoBundle 'rstacruz/sparkup.git' "write html code faster
{% endhighlight %}
Позволяет быстро набивать html-код с помощью чего-то вроде сниппетов. Возможно я откажусь
от него, потому что есть подозрения, что он немного криво работает с markdown, короче говоря
надо подумать, но в любом случае плагин интересный.

{% highlight vim %}
NeoBundle 'Valloric/vim-instant-markdown' "realtime markdown preview
{% endhighlight %}
Автоматически позволяет видеть изменения в файле по мере его редактирования(сохранения).
Я где-то это использовал, но сейчас оно мне особо не надо, в частности в этом блоге используется
jekyll, который тоже использует markdown, но смысла во всем этом нет, потому что содержимое браузера
и так автообновляется средствами самого jekyll, а для гитхаба я сложные README пока не делал.

{% highlight vim %}
NeoBundleLazy 'marijnh/tern_for_vim', { 'autoload': { 'filetypes': ['javascript'] } }
{% endhighlight %}
Очень хорошая поддержка js, когда я тестировал прекрасно работало. Семантическое автодополнение.
Возможно сейчас это не особо нужно из-за поддержки в youcompleteme, надо потестировать.

{% highlight vim %}
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
{% endhighlight %}
Компания плагинов для синтаксиса разных типов файлов. В общем-то рассказать
особо нечего, кроме последнего. Эти два плагина используют clang для
семантической раскраски. Получается в общем-то даже лучше, чем, в настоящий
момент в том же Clion от Jetbrains. Но с оговорками. В общем оно(подсветка) как
бы пляшет туда сюда и постоянно видны "следы" инкрементальной компиляции. Clamp
возможно работает лучше, но я его пока не тестировал из-за того что neovim
у меня в целом в процессе вялой настройки.

{% highlight vim %}
if has("gui_running")
    NeoBundle 'drmikehenry/vim-fontsize.git' "set fontsize on the fly
    NeoBundle 'tyru/restart.vim.git' "add restart support
    NeoBundle 'vim-scripts/utl.vim.git' "Open urls in files
    NeoBundle 'bling/vim-airline.git' "statusline for gvim only
endif
{% endhighlight %}
Компания плагинов, которые относились к gvim, который я больше не использую, главным
образом из-за отсутствия интеграции с tmux. Тем не менее расскажу что это для порядка.
vim-fontsize позволяет динамически изменять размер шрифта. Насколько я помню там эта
фича реализована была только для gui, но в любом случае оно для консоли не очень-то и
нужно, потому что это можно сделать например с помощью esc-последовательностей и
perl скриптов для случая rxvt-unicode или дополнительных патчей в случае suckless terminal

{% highlight vim %}
NeoBundle 'ryanoasis/vim-devicons.git' "fancy icons for fonts
{% endhighlight %}

Удивительный плагин, который добавляет что-то вроде поддержки иконок в vim.
Весь этот процесс целая история, которую надо рассказывать, но результат
выглядит приятно. Должен быть инициализован последним.
