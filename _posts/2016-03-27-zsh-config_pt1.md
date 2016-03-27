---
layout: post
title: UNIX (s)hell, pt1
categories: personal
tags: 
  - zsh
comments: true
mathjax: null
featured: true
published: true
date: 2016-03-27 18:00:00 +03:00
---

Думаю для большинства нормальных UNIX'оидов не секрет, что 99,9%
рутинной работы проще и удобнее выполнять из командной строки. Здесь играет
на руку и простота и единообразие. Всё с чем вы работаете это текст. Текст
принимаете и его же и возвращаете. На практике никто не достиг пайпов для
GUI, например. Да и использовать шелл можно только с помощью клавиатуры без
участия мыши, из-за чего нет переключения внимания. Как нет его из-за того
что большая часть приложений всегда везет себя предсказуемым образом. Даже,
как правило, в одной и той же палитре цветов. Я начну рассматривать
конфигурацию для zsh, самого продвинутого шелла под UNIX. Эта первая статья
из запланированных такого рода, рассчитана скорее на "начинающих", чем на
"заканчивающих".

<!--excerpt-->

* TOC
{:toc}

Для начала кратко о zsh. Это мощная и современная оболочка, которую можно
использовать как в интерактивном режиме, так и как интерпретатора сценариев.
Большей частью совместим с bash. А также с ksh и csh c помощью команд
*emulate sh/ksh/csh* соответственно.

# Минутка рекламы или зачем использовать zsh вместо другого ${SHELL}

Начать, надо, конечно, с того что использовать нужно то что вам нравится.
"Башизмы" в unix-сообществе почему-то не поощряются. А уже тем более
zsh-измы, но слушать таких людей не надо, потому что это в лучшем случае
неосиляторы, а в большинстве случаев попугаи. Zsh это не ghc, он не займет
много места, зато упростит вам жизнь.

Zsh это потрясающая вещь, которая облегчит вам жизнь и повысит
производительность работы, я гарантирую это.

* Лучшее автодополнение. Никакой bash-completions и рядом не валялся. Есть
  встроенная поддрежка нескольких сотен команд, можно программировать
  собственное. Есть интеративное меню с навигацией и многое другое, чему нет
  прямых аналогов в других шеллах.
* Небольшой размер и высокая производительность.
* Исправление опечаток.
* Можно настроить почти всё.
* Zle вместо readline позволяет делать с командной строкой почти что угодно
  в плане навигации. В качестве доказательства можно посмотреть как раньше
  делали vim(не vi)-подобные keybindings: <a
  href="https://github.com/hchbaw/opp.zsh">github.com/hchbaw/opp.zsh</a>.
  В настоящее время этот плагин немного устарел, большая часть этих фич есть
  из коробки. Также есть поддержка многострочных команд.
  На горячие клавиши можно повесить что угодно включая запуск ncurses
  приложений.
* Подсветка синтаксиса как в fish(с помощью плагинов)
* Extended glob. Например cp -r \*(/) ~/tmp скопирует все директории(и только
  их) в ~/tmp. И другие удобные штуки того же типа, например =vim -> /usr/sbin/vim.
  Также возможен рекурсивный globbing, представляет собой некое подобие
  вывода find(ls \*/\*\*, например).
* Inline переменных и других вещей в командную строку, например 
    {% highlight shell %}
    echo $[2*32] <Tab>
    # Превратится в 
    echo 64
    {% endhighlight %}
* Продвинутое приглашение командной строки.
* Много возможностей для операций с массивами, строками и переменными.

А вообще чтобы понять всю его прелесть нужно просто его использовать. Вы вряд
ли после него станете использовать другой шелл в интерактивном режиме. Правда
нужно будет немного потрудиться над его настрокой, потому что по-умолчанию
мало что из этого списка работает так хорошо как может.

# С чего начать

Чтобы просто начать пользоваться я бы рекомендовал вот эту доку: <a
href="http://www.bash2zsh.com/zsh_refcard/refcard.pdf">bash2zsh.com/zsh_refcard/refcard.pdf</a>
Чуть позже можно ознакомиться с ресурсом zsh-lovers: <a
href="https://grml.org/zsh/zsh-lovers.html">grml.org/zsh/zsh-lovers</a>
и вики: <a href="http://zshwiki.org/home/">zshwiki.org/home/</a>

Но, конечно, если вы хотите познать всю мощь zsh, то  лучше всего начать
с чтения полной документации, её можно найти вот здесь: <a
href="http://zsh.sourceforge.net/Doc/">zsh.sourceforge.net/Doc/</a>, впрочем
она также часто идет вместе с дистрибутивом zsh тогда вам поможет man zsh
и далее по секциям. Хотя как по мне в man'е удобнее искать что-то конкретное,
а читать приятнее в pdf, но это дело вкуса. Если всё внимательно изучить, то
вы сами для себя выработаете самые лучшие best-practices и сделаете самый
лучший и наиболее подходящий конфиг. Но для начала так же можно посмотреть на
готовое. Существует ещё несколько устаревший, но всё ещё годный user guide:
<a
href="http://zsh.sourceforge.net/Guide/zshguide.html">zsh.sourceforge.net/Guide/zshguide.html</a>

Большие списки этого например вот здесь:
<a href="https://github.com/unixorn/awesome-zsh-plugins">unixorn/awesome-zsh-plugins</a>
В нем перечислено довольно большое количество плагинов, фреймворков, тем для
командной строки и прочего, достаточно чтобы залипнуть(а для новичка чтобы
утонуть в информации). Тем не менее грех с таким большим списком не
познакомить.

Помимо прочего этом списке приведено большое количество фреймворков, из них я бы для
ознакомления рекомендовал <a
href="https://github.com/sorin-ionescu/prezto">prezto</a> и <a
href="https://github.com/robbyrussell/oh-my-zsh">oh-my-zsh</a>. Также стоит
обратить внимание на <a
href="https://github.com/zsh-users/antigen">antigen</a>, это что-то вроде
менеджера для дополнений bundle/vundle/neobundle для vim, только для zsh.
Выглядит красиво и инкапсуляция это хорошо, но сам я ничего из этого не
использую, хотя оно хорошее и многим нравится. Вы можете использовать их не
целиком, а те части, которые вам нравятся, собственно это нормальная
ситуация, я часто вместо заимствования кода переписываю его на свой лад.

Сам я за основу когда-то конфиг из <a href="https://grml.org/zsh/">grml</a>.
Он достаточно прост и удобен, хотя для повседневного использования мне больше
нравится конфиг, в котором больше "targeting'а", без кучи хуков на разные
дремучие версии zsh и тп.

# Мой конфиг

Мой конфиг zsh можно найти там же где и все dotfiles по адресу: <a
href="https://github.com/neg-serg/dotfiles/tree/master/.zsh">dotfiles/tree/master/.zsh</a>

Он у меня как и конфиг vim'а сделан модульным, из нескольких файлов:

* 01-init -- начальная инициализация, в основном опции и настройки
* 03-exports -- настройка переменных окружения(например $PATH)
* 03-helpers -- различные "вспомогательные" функции, например врапперы для
  строк, которые я решил выделить чтобы потом использовать извне.
* 04-oldprompt -- приглашение командной строки, представляет собой
  кастомизированный вариант fish.
* 05-functions -- большая часть самописных функций, почти все, например для
  извлечения файлов из архивов, взаимодействия с vim и так далее.
* 06-alias -- алиасы
* 11-open -- xdg-open подобная открывашка для командной строки
* 12-completion -- всё автодополнение
* 12-vi_additions -- улучшения vi-mode в zsh, в последнее время не очень-то
  актуально в новых версиях
* 13-bindkeys -- остальные горяцие клавиши и настройки системы ввода zle
* 20-autopair -- плагин для автоматической вставки парных скобок по аналогии
  с vim unimpaired
* 60-functional -- функции высшего порядка и фп в zsh. Работает.
* 81-completion_gen -- генератор автодополнения для неизвестных программ.
  Работает большей частью на основе *--help*
* 89-vim-interation.plugin -- самопильное взаиомдействие с vim, используется
  в связке с <a href="">моим форком Notion</a>.
* 92-history-substring-search -- плагин для поиска в истории по подстроке,
  которая уже введена в zle.
* 96-fzf -- взаимодействие с fzf в zsh
* 98-syntax -- подсветка синтаксиса. Кастомизированная, добавлена подсветка
  директорий, типов файлов и проч.

Обо всем этом я подробнее напишу позже.

Для начала рассмотрим инициализацию, первый из этих файлов. Сверху код, снизу
описание. Часть настроек, которая связана с stty подойдет, разумеется, не
только для zsh, а для любого шелла вообще. Также приведен поверхностный обзор
некоторых используемых модулей.

{% highlight shell %}
inpath() { [[ -x "$(which "$1" 2>/dev/null)" ]]; }
nexec() { [[ -z $(pidof "$1") ]]; }
{% endhighlight %}

Вспомогатетельные функции. Первая проверяет есть ли такой exe-шник в $PATH,
вторая проверяет запущено ли приложение.

{% highlight shell %}
nexec ssh-agent && eval "$(ssh-agent -s)"
{% endhighlight %}

Незатейливая настройка ssh-агента.

{% highlight shell %}
# Execute code that does not affect the current session in the background.
{
  # Compile the completion dump to increase startup speed.
  zcompdump="${ZDOTDIR:-$HOME}/.zcompdump"
  if [[ "$zcompdump" -nt "${zcompdump}.zwc" || ! -s "${zcompdump}.zwc" ]]; then
    zcompile "$zcompdump"
  fi

  # Set environment variables for launchd processes.
  if [[ "$OSTYPE" == darwin* ]]; then
    for env_var in PATH MANPATH; do
      launchctl setenv "$env_var" "${(P)env_var}"
    done
  fi
} &!
{% endhighlight %}

Компиляция кэша zcompdump

{% highlight shell %}
# autoload wrapper - use this one instead of autoload directly
# We need to define this function as early as this, because autoloading
# 'is-at-least()' needs it.
function zrcautoload() {
    emulate -L zsh
    setopt extended_glob
    local fdir ffile
    local -i ffound

    ffile=$1
    (( found = 0 ))
    for fdir in ${fpath} ; do
        [[ -e ${fdir}/${ffile} ]] && (( ffound = 1 ))
    done

    (( ffound == 0 )) && return 1
    autoload -U ${ffile} || return 1
    return 0
}
{% endhighlight %}

Враппер для autoload. Был в своё время взят из grml, в настоящее время от
него хорошо было бы и отказаться, но пусть будет, он не особо-то и мешает.

{% highlight shell %}
zrcautoload colors && colors
{% endhighlight %}

Загрузка ассоциативных массивов для работы с цветом, например $fg[white] дает
белый цвет. За подробностями можно полезть в документацию, но в целом всё
очень наглядно.

{% highlight shell %}
zle_highlight+=(suffix:fg=blue)
{% endhighlight %}

ZLE будет печатать суффикс голубым цветом при автодополнении и подобных
операциях где вызываются её виджеты.

{% highlight shell %}
zle -N zle-keymap-select
{% endhighlight %}

Для vi-mode переход в режим редактирования принудительно, по-умолчанию.

{% highlight shell %}
unset MAILCHECK
{% endhighlight %}

Убрать "уведомления о новой почте", которые раньше были особенно характерны
для debian-based дистрибутивов, надоедливые сообщения, ну их.

{% highlight shell %}
stty eof  2> /dev/null  # stty eof ''
stty ixany
stty ixoff -ixon # Disable XON/XOFF flow control; this is required to make C-s work in Vim.
{% endhighlight %}

Control+D служит для закрытия шелла/файла/потока. C-s и C-q не
используются. По-умолчанию они работают как блокировщики и разблокировщики
ввода-вывода в консоль соотвественно. Я не люблю это поведение, к тому же
того же самого можно добиться с помощью tmux, например переведя буфер в режим
редактирования

{% highlight shell %}
function stty_setup(){
    #stty intr "^C" 2> /dev/null
    #stty erase "^?" 2> /dev/null
    #stty start "undef" 2> /dev/null
    #stty stop "undef" 2> /dev/null
    #stty susp "^Z" 2> /dev/null
    #stty rprnt "^R" 2> /dev/null
    #stty werase "^W" 2> /dev/null
    #stty lnext "^B" 2> /dev/null
    #stty flush "undef" 2> /dev/null
    ##stty eol "undef" 2> /dev/null
    ##stty eol2 "undef" 2> /dev/null
    ##stty swtch "undef" 2> /dev/null
    ##stty kill "undef" 2> /dev/null
    ##stty quit "undef" 2> /dev/null
    stty time 0 2> /dev/null
    stty min 0 2> /dev/null
    stty line 6 2> /dev/null
    stty speed 4000000 &> /dev/null
}

[[ $- =~ i ]] && stty_setup
{% endhighlight %}

Экспериментальные настройки stty связанные с отзывчивостью, присуствуют
в основном ради интереса, хотя при работе со всякими модемами, minicom(или
GNU Screen) и прочими вещами подобные настроки могут оказаться актуальными.

{% highlight shell %}
[[ -f ~/.config/dircolors/.dircolors ]] && eval $(dircolors ~/.config/dircolors/.dircolors)
{% endhighlight %}

Загрузка dircolors. Они используются во многих программых из
coreutils(например ls) и не только, своего рода аналог иконок из GUI,
позволяет быстро визуально определить что за тип файла перед вами. Особенно
это хорошо работает если в терминале 256-цветов(24-bit esc-последовательности
пока используются не слишком широко).

{% highlight shell %}
# No core dumps for now
ulimit -c 0

setopt append_history # this is default, but set for share_history
setopt share_history # import new commands from the history file also in other zsh-session
setopt extended_history # save each command's beginning timestamp and the duration to the history file
setopt histignorealldups # remove command lines from the history list when the first character on the line is a space
{% endhighlight %}

Первая группа настроек для истории команд. 

append_history: Добавлять команды в историю, а не писать её заново с каждым
запуском, стоит по дефолту.

share_history: синхронизировать разные истории из разных zsh сессий между
собой

extended_history: сохранение в файл с историей временных меток

histignorealldups: удаление дубликатов

{% highlight shell %}
setopt hist_expire_dups_first # when trimming history, lose oldest duplicates first
setopt hist_ignore_dups # ignore duplication command history list
setopt hist_verify # don't execute, just expand history
setopt hist_ignore_space # reduce whitespace in history
setopt inc_append_history # add comamnds as they are typed, don't wait until shell exit

# remove command lines from the history list when the first character on the
# line is a space
setopt histignorespace 
{% endhighlight %}

hist_expire_dups_first: при удалении дубликатов из истории удалять самые
старые первыми.

hist_ignore_dups: игнорировать дубликаты в истории команд

hist_verify: обработка случаев вроде !$\<enter\>, сначала осуществляется
переход на следующую строку и раскрытие команды.

histignorespace: не добавлять в историю команду, которая начинается
с пробела.

{% highlight shell %}
# if a command is issued that can't be executed as a normal command, and the
# command is the name of a directory, perform the cd command to that directory.
setopt auto_cd 
{% endhighlight %}

С помощью auto_cd можно не печатать команду cd. Когда есть директория с таким
именем и в ${PATH} нет исполняемого файла с таким именем, то осуществляется
переход. То есть ./vim перейдет в директорию vim, а vim запустит исполняемый
файл. Всё логично. Очень удобно, одна из моих любимых особенностей zsh.

{% highlight shell %}
# in order to use #, ~ and ^ for filename generation grep word
# *~(*.gz|*.bz|*.bz2|*.zip|*.Z) -> searches for word not in compressed files
# don't forget to quote '^', '~' and '#'!
setopt extended_glob # ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^ ^^ ^ ^ ^ ^
{% endhighlight %}

Улучшенный globbing, то есть то что идет после звездочки. Добавляются
различные операторы, например ls -d ^dev выведет все директории кроме dev.
За подробностями можно обратиться к документации, но, что намного проще можно
воспользоваться моим конфигом в автодополнением, он покажет какие опции чему
соответствуют.

{% highlight shell %}
setopt longlistjobs # display PID when suspending processes as well
setopt nonomatch # try to avoid the 'zsh: no matches found...'
setopt notify  # report the status of backgrounds jobs immediately
setopt hash_list_all  # whenever a command completion is attempted, make sure the entire command path is hashed first.
setopt completeinword # not just at the end
setopt nohup  # don't send SIGHUP to background processes when the shell exits.
setopt auto_pushd # make cd push the old directory onto the directory stack.
setopt pushdminus # pushd -N goes to Nth dir in stack
setopt pushdsilent # do not print dirstack after each cd/pushd
setopt pushdtohome #pushd with no args pushes to home
setopt pushd_ignore_dups # don't push the same dir twice.
setopt nobeep # get rid of beeps
setopt noglobdots  # * shouldn't match dotfiles. ever.
setopt noshwordsplit  # use zsh style word splitting
setopt noflowcontrol # no c-s/c-q output freezing
{% endhighlight %}

longlistjobs: показывать jobs в "длинном" формате, вместе с pid.

nonomatch: избавляется от ненужного шума что zsh что-то там не может найти.

notify: выдает статус программ, которые были отправлены в бэкграунд(в
смысле с помощью оператора & или команды bg). Или, иными словами просто
сообщать об изменении статуса фонового задания. Тем, кому эта информация
кажется шумом, стоит отключить такое поведение.

hash_list_all: хэшировать всё перед автодополнением. В общем-то мастхэв.

completeinword: автодополнение не только в конце, но и в середине. Как по мне
мастхэв, хотя иногда может работать странненько, но это лучше чем ничего.

nohup: не откреплять процессы от шелла, который вышел. В большинстве случаев
это то поведение, которое вы и ожидаете от интерактивного шелла, но могут
быть исключения.

auto_pushd: автоматически заполнять стек посещенных директорий. Для моего
конфига это мастхэв, потому что есть хоткеи для навигации по истории
директорий назад, что я считаю удобным, но в целом, когда этого нет, то это
не нужная опция, мне кажется этот список лучше заполнять вручную.

pushdminus: отрицательные элементы в стеке, думаю очевидно для чего. cd -1
предыдущая в истории(или точнее в стеке) директория.

pushdsilent: не писать никаких сообщений при добавлении директории в стек.
Меньше ненужного информационного шума.

pushdtohome: pushd без аргументов добавляет в стек ${HOME}

pushd_ignore_dups: не добавлять одно и то же по два раза подряд

nobeep: и не пикать

noglobdots: в globbing'е не учавствуют скрытые файлы.

noshwordsplit: делить строку на слова в стиле zsh. Так лучше и больше
возможностей для настройки.

noflowcontrol: выключает "контроль потока", то есть то что уже по сути и так
было выключено с помощью stty, c-s и c-q больше не используются.

{% highlight shell %}
setopt c_bases  # print $(( [#16] 0xff ))
setopt prompt_subst # set the prompt
# make sure to use right prompt only when not running a command
setopt transient_rprompt # only show the rprompt on the current prompt
setopt interactivecomments # allow interactive comments
{% endhighlight %}

c_bases: добавляет возможность для печати hex чисел встроенными средствами.
Чтобы понять как это работает попробуйте print $(( [#16] 0xff ))

prompt_subst: раскрывать переменные в prompt. Очень полезно, без этой опции
возможности приглашения коммандной строки будут довольно бедными.

transient_rprompt: скрывать правую часть приглашения командной
строки(zsh может показывать приглашение и слева и справа одновременно) когда
заполнено достаточно много места. Это полезно например для длинных команд или
использовании copy-paste.

interactivecomments: интерактивные комменты, которые начинаются с символа
\# прямо в командной строке.

{% highlight shell %}
# ~ substitution and tab completion after a = (for --x=filename args)
setopt magicequalsubst
# setopt glob_star_short # */** -> **
{% endhighlight %}

glob_star_short: более короткий варинт для рекурсивного globbing'а, можно
писать просто две звездочки. Надо включить, нужно. Но работает только
в последних версиях.

magicequalsubst: автодополнять после знака =, довольно полезно, например для
команды dd if=<...>

{% highlight shell %}
# watch for everyone but me and root
watch=(notme root)
{% endhighlight %}

Следить за всеми кроме меня и рута.

{% highlight shell %}
# automatically remove duplicates from these arrays
typeset -U path cdpath fpath manpath
{% endhighlight %}

Убрать дубликаты из путей. Немного поднимет производительность.

{% highlight shell %}
zrcautoload zmv # who needs mmv or rename?
{% endhighlight %}

Включить встроенный в zsh модуль для переименования файлов группами. Немного
мозголомный, но могу рассказать про него, если интересно. Лично я предпочитаю
в большинстве случаев использовать vimv -e vim или vimv -e sed в зависимости
от того что проще. Может служить заменой perl rename или mmv.


{% highlight shell %}
zrcautoload history-search-end
{% endhighlight %}

Для задействования команд

zle -N history-beginning-search-backward-end history-search-end
zle -N history-beginning-search-forward-end history-search-end

Они нужны для более удобного поиска в командной строке. То что уже введено
служит преффиксом для поиска вверх по истории команд. Например ls<up>
покажет, скажем, lsof, который был 10 команд назад, а не echo lol, которое
было одну команду назад, очень удобно, одна из киллер-фич zsh.

{% highlight shell %}
zrcautoload zargs
{% endhighlight %}

Узкоспециализированный плагин, который реализует xargs средствами zsh.
В теории обладает большими возможностями. В частности вы не сможете передать
xargs в качестве аргумента функцию, а в zargs сможете. Но, если честно, к тех
сложных случаях, когда мне было необходимо такое поведение находились
дополнительные проблемы, которые сводили пользу от этого плагина на нет. Тем
не менее он довольно стоящий, особенно в том случае, если в системе вдруг не
окажется xargs. Из минусов также отмечу куда менее удобный интерфейс
пользователя. Но всё равно штука интересная.

{% highlight shell %}
fpath=(~/.zsh/compdef $fpath) 
{% endhighlight %}

Добавить compdef к директориям, для поиска по которым не нужен никакой
преффикс.

{% highlight shell %}
# completion system
if zrcautoload compinit ; then
    compinit || print 'Notice: no compinit available :('
else
    print 'Notice: no compinit available :('
    function zstyle { }
    function compdef { }
fi
{% endhighlight %}

Инициализация системы автодополнения

{% highlight shell %}
zrcautoload zed # use ZLE editor to edit a file or function
{% endhighlight %}

Встроенный в zsh редактор на основе zle. Честно говоря уже забыл зачем он
мне, возможно не нужно.

{% highlight shell %}
for mod in complist deltochar mathfunc ; do
    zmodload -i zsh/${mod} 2>/dev/null || print "Notice: no ${mod} available :("
done
{% endhighlight %}

complist: работа с длинными меню и другие важные улушения для автокомплита,
мастхэв. За подробностями пожалуйте, например, <a
href=http://www.fifi.org/cgi-bin/info2www?(zsh)The+zsh/complist+Module"">сюда</a>.

deltochar: модуль, который повторяет zap-to-char из emacs. Уже устарело, для
этого мне кажется удобнее воспользоваться vi-mode, после чего dt[символ], но
дело вкуса.

mathfunc: разные не сложные математические функции, например квадратные корни.

{% highlight shell %}
# autoload zsh modules when they are referenced
zmodload -a  zsh/stat    zstat
zmodload -a  zsh/zpty    zpty
zmodload -ap zsh/mapfile mapfile
{% endhighlight %}

zstat: встроенный stat, отличается от оригинала более простым выводом.

zpty: запуск команды в невидимом виде в псевдотерминале.

mapfile: доступ к файлам через ассоциативные массивы. Было нужно для
какого-то специального скрипта, в обычной конфигурации как по мне особого
смысла не имеет.

{% highlight shell %}
# Use hard limits, except for a smaller stack and no core dumps
unlimit
limit stack 8192
limit core 0 # important for a live-cd-system
limit -s
{% endhighlight %}

Уменьшить размер стека, остальные лимиты снять, не сбрасывать coredump(для
systemd скорее всего не сработает), остальные лимиты убрать.

{% highlight shell %}
# Keeps track of the last used working directory and automatically jumps
# into it for new shells.
export ZSH=~/.zsh
{% endhighlight %}

$ZSH это просто альтернативное имя для директории с zsh.

Это первая часть такого рода, в остальных я рассмотрю... Всё остальное.
