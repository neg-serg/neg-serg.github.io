---
layout: post
title: UNIX (s)hell, pt3
categories: personal
tags: zsh
comments: true
mathjax: null
featured: true
published: true
---

Продолжение статей про zsh, здесь будет приведен обзор автодополнения, одной
из основных фич, которая существенно выделяет zsh на фоне bash и других
оболочек. Кто-то может сказать что fish не лучше, но он совершенно не
настраиваемый и далеко не такой гибкий. Причем эту ситуацию разработчики
изменять не планируют.

<!--excerpt-->

Оригинал автокомплита можно посмотреть здесь:
<a href="https://github.com/neg-serg/dotfiles/blob/master/.zsh/12-completion.zsh">https://github.com/neg-serg/dotfiles/blob/master/.zsh/12-completion.zsh</a>

Автокомплит основан, большей частью на конфиге zsh из GRML.

Почти весь представляет собой большую функцию mycompletion. Просто потому что
я почему-то люблю представлять скрипты в виде функций.

{% highlight shell %}
    zstyle ':acceptline:*' rehash true
{% endhighlight %}

Указывает на то, что при выполнении команды будет обновлены PATH и тому
подобные вещи до актуального состояния. Можно рассматривать это также как
ребилд индексов автокомплита.

{% highlight shell %}
    # allow one error for every three characters typed in approximate completer
    zstyle ':completion:*:approximate:' \
                                                max-errors 'reply=( $((($#PREFIX+$#SUFFIX)/3 )) numeric )'
{% endhighlight %}

Задает толерантность к ошибкам, которая может быть исправлена по [tab]-[tab].

{% highlight shell %}
    # don't complete backup files as executables
    zstyle ':completion:*:complete:-command-::commands' \
                                                ignored-patterns '(aptitude-*|*\~)'
{% endhighlight %}

Убирает из автокомплита файлы с атрибутом +x, которые выглядят как
временные(у меня заканчиваются на тильду, можно и что-то emacs-специфичное
добавить).

{% highlight shell %}
    # start menu completion only if it could find no unambiguous initial string
    zstyle ':completion:*:correct:*'            insert-unambiguous true
{% endhighlight %}

Не выдавать меню в том случае, когда выбирать не из чего.

{% highlight shell %}
    zstyle ':completion:*:corrections'          format "%{${fg[blue]}%}--%{${reset_color}%} %d%{${reset_color}%} - (%{${fg[cyan]}%}errors %e%{${reset_color}%})"
    zstyle ':completion:*:descriptions'         format "%{${fg[blue]}%}--%{${reset_color}%} %d%{${reset_color}%}%{${reset_color}%}"
{% endhighlight %}

Подсветка возможных исправлений либо меню автокомплита.


{% highlight shell %}
    zstyle ':completion:*:correct:*'            original true
{% endhighlight %}

То, что уже было написано вне зависимости от того видит это zsh как валидное,
будет также отражено в меню автокомплита.

{% highlight shell %}
    zstyle ':completion:*:-tilde-:*'            group-order 'named-directories'
{% endhighlight %}

Первыми в автокомплите для тильды будут hash-директории. Hash это такие
глобальные алиасы для директорий. Например hash -d v="${HOME}/vid" задает
соответствие ~v -> ~/vid/


{% highlight shell %}
    # insert all expansions for expand completer
    zstyle ':completion:*:expand:*'             tag-order all-expansions
{% endhighlight %}

Эффект, который похож на glob_complete, только для expand completer. expand
completer это, например /u/b/z -> /usr/bin/z и далее после второго tab будет
меню. Must have.

{% highlight shell %}
    zstyle ':completion:*:history-words'        list false
    # activate menu                            
    zstyle ':completion:*:history-words'        menu yes
    # ignore duplicate entries                 
    zstyle ':completion:*:history-words'        remove-all-dups yes
    zstyle ':completion:*:history-words'        stop yes
{% endhighlight %}

Меню, а не список для истории. Без дубликатов.

{% highlight shell %}
    # match uppercase from lowercase           
    zstyle ':completion:*'                      matcher-list 'm:{a-z}={A-Z}' 'r:|[._-]=* r:|=*' 'l:|=* r:|=*'
{% endhighlight %}

Задает соответствие символов, которые могут быть автоматически преобразованы
при автокомлите, в частности это делает автокомплит нечувствительным
к регистру. Must have.

{% highlight shell %}
    # separate matches into groups          
    zstyle ':completion:*:matches'              group 'yes'
    zstyle ':completion:*'                      group-name ''
{% endhighlight %}

Группировка автокомлита по группам. Часто бывает что для одного варианта есть
несколько возможных источников, которые выдают разные варианты комплита,
в том случае когда семантику предугадать нельзя.

{% highlight shell %}
    zstyle ':completion:*'                      menu select=5
{% endhighlight %}

Выводить меню только в случае, если число подходящих вариантов в списке 5 или
больше, в противном случае прыгать по этому списку в цикле по tab по аналогии
с tcsh или zsh по-умолчанию.

{% highlight shell %}
    zstyle ':completion:*:messages'             format "- %{${fg[cyan]}%}%d%{${reset_color}%} -"
    zstyle ':completion:*:options'              auto-description '%d'
    zstyle ':completion:*:options'              description 'yes'
{% endhighlight %}

Украшательства описаний для автокомлита.

{% highlight shell %}
    zstyle ':completion:*'                      verbose true
    zstyle ':completion:*:-command-:*:'         verbose false
{% endhighlight %}

Убирает описания для команд при автокомплите, потмоу что они медленно
работают.


{% highlight shell %}
    zstyle ':completion:*:warnings'             format $'%{\e[0;31m%}No matches for:%{\e[0m%} %d'
    zstyle ':completion:correct:'               prompt 'correct to: %e'
{% endhighlight %}

Украшательства warning'ов и предложений для коррекции.


{% highlight shell %}
    zstyle ':completion:*:*:zcompile:*'         ignored-patterns '(*~|*.zwc)'
{% endhighlight %}

Временные файлы не учавствуют в автодополнении.


{% highlight shell %}
    zstyle ':completion::(^approximate*):*:functions' \
                                                ignored-patterns '_*'
{% endhighlight %}

Также в автодополнении не учавствуют сами файлы автокомплита, которые
начинаются с \_.

{% highlight shell %}
    zstyle ':completion:*:manuals'              separate-sections true
    zstyle ':completion:*:manuals.*'            insert-sections   true
    zstyle ':completion:*:man:*'                menu yes select
{% endhighlight %}

Разделение секций для автокомплита man-страниц. 

{% highlight shell %}
    zstyle ':completion:*'                      special-dirs ..
{% endhighlight %}

Автокомплит для предыдущей директории, must have.

{% highlight shell %}
    zstyle ':completion:*:(mv|cp|file|m|mplayer|mp|mpv):*' \
                                                ignored-patterns '(#i)*.(url|mht)'
{% endhighlight %}

Игнорировать url и mht для плеера, копирования, перемещения и др.

{% highlight shell %}
    zstyle ':completion:*:*:(mplayer|mp|mpv):*' tag-order files
    zstyle ':completion:*:*:(mplayer|mp|mpv):*' file-sort name
    zstyle ':completion:*:*:(mplayer|mp|mpv):*' menu select auto
    zstyle ':completion:*:*:(mplayer*|mp):*'    file-patterns '(#i)*.(rmvb|mkv|vob|ts|mp4|m4a|iso|wmv|webm|flv|ogv|avi|mpg|mpeg|iso|nrg|mp3|flac|rm|wv|m4v):files:mplayer\ play *(-/):directories:directories'
{% endhighlight %}

Сортировка по отсортированному списку файлов, из списка, который представляет
собой расширение файлов аудио или видео, без учета регистра, для медиаплеера.
Нужно чтобы автодополнение не выдывало всякую ненужную ерунду.

{% highlight shell %}
    zstyle ':completion:*:default'      \
        select-prompt \
        "%{${fg[cyan]}%}Match %{${fg_bold[cyan]}%}%m%{${fg_no_bold[cyan]}%}  Line %{${fg_bold[cyan]}%}%l%{${fg_no_bold[blue]}%}  %p%{${reset_color}%}"
        zstyle ':completion:*:default'      \
        list-prompt   \
        "%{${fg[cyan]}%}Line %{${fg_bold[cyan]}%}%l%{${fg_no_bold[cyan]}%}  Continue?%{${reset_color}%}"
        zstyle ':completion:*:warnings'     \
        format        \
        "- %{${fg_no_bold[blue]}%}no match%{${reset_color}%} - %{${fg_no_bold[cyan]}%}%d%{${reset_color}%}"
{% endhighlight %}

Украшательства для длинного меню. Это нужно в том случае, когда есть очень
много вариантов, которые не влезают на экран.

{% highlight shell %}
    zstyle ':completion:*:default'  list-colors ${${(s.:.)LS_COLORS}%ec=*}
{% endhighlight %}

Использовать при автокомплите файлов подсветку наиболее быстрым известным мне
способом. Некоторые другие будут тормозить, особенно на больших списках.
Возможно можно реализовать подстановку LS_COLORS и быстрее.

{% highlight shell %}
    zstyle ':completion:*' squeeze-slashes true # e.g. ls foo//bar -> ls foo/bar
{% endhighlight %}

Впихивает два слеша в один.

{% highlight shell %}
    [[ -r ~/.ssh/known_hosts ]] && _ssh_hosts=(${${${${${${${(f)"$(<$HOME/.ssh/known_hosts)"}:#[\|]*}%%\ *}%%,*}%%:*}#\[}%\]}) ||
    _ssh_hosts=()
    [[ -r ~/.ssh/config ]] && _ssh_config_hosts=($(sed -rn 's/host\s+(.+)/\1/ip' "$HOME/.ssh/config" | grep -v "\*" )) ||
    _ssh_config_hosts=()
    hosts=($HOST "$_ssh_hosts[@]" $_ssh_config_hosts[@] localhost)
    zstyle ':completion:*:hosts' hosts ${hosts}
        # highlight the original input.
        zstyle ':completion:*:original' list-colors "=*=$color[blue];$color[bold]"
        # colorize username completion
        zstyle ':completion:*:*:*:*:users' list-colors "=*=$color[blue];$color[bg-black]"
{% endhighlight %}

Быстрый и красивый, хотя страшно выглядещий, способ автокомплита ssh хостов.

{% highlight shell %}
    zstyle ':completion:*:wine:*'             file-patterns '*.(exe|EXE):exe'
{% endhighlight %}

wine автодополняет только exe файлы.

{% highlight shell %}
    # highlight parameters with uncommon names
    zstyle ':completion:*:parameters'         list-colors "=[^a-zA-Z]*=$color[cyan]"
{% endhighlight %}

Раскраска параметров в меню автодополнения циановым цветом

{% highlight shell %}
    # highlight aliases                      
    zstyle ':completion:*:aliases'            list-colors "=*=$color[green]"
{% endhighlight %}

А алиасов зеленым.

{% highlight shell %}
    ### highlight the original input.
    zstyle ':completion:*:original'           list-colors "=*=$color[blue];$color[bold]"
{% endhighlight %}

Комплит того, что было введено, без исправления ошибок, голубым цветом. Почти
невероятная ситуация, на самом деле. будет в самом конце списка.

{% highlight shell %}
    ### highlight words like 'esac' or 'end'
    zstyle ':completion:*:reserved-words'     list-colors "=*=$color[blue]"
{% endhighlight %}

Зарезервированные слова подсвечиваются голубым цветом.


{% highlight shell %}
    ### colorize processlist for 'kill'
    zstyle ':completion:*:*:kill:*:processes' list-colors "=(#b) #([0-9]#) #([^ ]#)*=$color[cyan]=$color[yellow]=$color[green]"
    zstyle ':completion:*:kill:*' command 'ps -u $USER -o pid,%cpu,tty,cputime,cmd'
{% endhighlight %}

Умная и красивая автодополнялка для kill. Дополняет вывод по имени процесса
и подсвечивает это дело голубеньким :D

{% highlight shell %}
    zstyle ':completion:*:*:zathura:*'        tag-order files
    zstyle ':completion:*:*:zathura:*'        file-patterns '*(/)|*.{pdf,djvu}'
{% endhighlight %}

Zathura это минималистичная читалка для pdf/djvu и всякого. Дополнять только
по файлам, типа pdf,djvu и директории(для навигации).

{% highlight shell %}
    # make them a little less short, after all (mostly adds -l option to the whatis calll)
    zstyle ':completion:*:command-descriptions' command '_call_whatis -l -s 1 -r .\*; _call_whatis -l -s 6 -r .\* 2>/dev/null'
{% endhighlight %}

Задает то, что будет показано в описании команд


{% highlight shell %}
    zstyle ':completion:*:*:task:*'                verbose yes         # taskwarrior
    zstyle ':completion:*:*:task:*:descriptions'   format '%U%B%d%b%u' # taskwarrior
    zstyle ':completion:*:*:task:*'                group-name ''       # taskwarrior
{% endhighlight %}

Украшательства автокомплита для taskwarrior. Я его использую в качестве
легкого псевдо-аналога emacs org-mode для управления своими тасками.

{% highlight shell %}
    # command completion: highlight matching part of command, and 
    zstyle -e ':completion:*:-command-:*:commands' list-colors 'reply=( '\''=(#b)('\''$words[CURRENT]'\''|)*-- #(*)=0=38;5;45=38;5;136'\'' '\''=(#b)('\''$words[CURRENT]'\''|)*=0=38;5;248'\'' )'
{% endhighlight %}

Подсвечивает белым цветом ту часть команды, которая уже была введена.
Практического смысла большого нету, но мне нравится как это выглядит.

{% highlight shell %}
    # This is needed to workaround a bug in _setup:12, causing almost 2 seconds delay for bigger LS_COLORS
    # UPDATE: not sure if this is required anymore, with the -command- style above.. keeping it here just to be sure
    zstyle ':completion:*:*:-command-:*' list-colors ''
    # run rehash on completion so new installed program are found automatically:
    _force_rehash() {
        (( CURRENT == 1 )) && rehash
        return 1
    }
    ## correction
    setopt correct
    zstyle -e ':completion:*' completer '
        if [[ $_last_try != "$HISTNO$BUFFER$CURSOR" ]] ; then
            _last_try="$HISTNO$BUFFER$CURSOR"
            reply=(_complete _match _ignored _prefix _files)
        else
            if [[ $words[1] == (rm|mv) ]] ; then
                reply=(_complete _files)
            else
                reply=(_oldlist _expand _force_rehash _complete _ignored _correct _approximate _files)
            fi
        fi'

    [[ -d $ZSHDIR/cache ]] && zstyle ':completion:*' use-cache yes && \
                              zstyle ':completion::complete:*' cache-path $ZSHDIR/cache/
    # use generic completion system for programs not yet defined; (_gnu_generic works
    # with commands that provide a --help option with "standard" gnu-like output.)
    for compcom in cp deborphan df feh fetchipac head hnb ipacsum mv \
                   pal stow tail uname ; do
        [[ -z ${_comps[$compcom]} ]] && compdef _gnu_generic ${compcom}
    done; unset compcom
    # see upgrade function in this file
    # compdef _hosts upgrade

() {
    local -a coreutils
    coreutils=(
        # /bin
        cat chgrp chmod chown cp date dd df dir ln ls mkdir mknod mv readlink
        rm rmdir vdir sleep stty sync touch uname mktemp
        # /usr/bin
        install hostid nice who users pinky stdbuf base64 basename chcon cksum
        comm csplit cut dircolors dirname du env expand factor fmt fold groups
        head id join link logname md5sum mkfifo nl nproc nohup od paste pathchk
        pr printenv ptx runcon seq sha1sum sha224sum sha256sum sha384sum
        sha512sum shred shuf sort split stat sum tac tail tee timeout tr
        truncate tsort tty unexpand uniq unlink wc whoami yes arch touch
    )

    for i in $coreutils; do
        # all which don't already have one
        # at time of this writing, those are:
        # /bin
        #   chgrp chmod chown cp date dd df ln ls mkdir rm rmdir stty sync
        #   touch uname
        # /usr/bin
        #   nice comm cut du env groups id join logname md5sum nohup printenv
        #   sort stat unexpand uniq whoami
        (( $+_comps[$i] )) || compdef _gnu_generic $i 
    done

}

{% endhighlight %}

Какие-то опции, связанные с кешированием, rehash и тп, смысла которых я не
помню. Также задается автоматическое выдирание автокомплита опций для команд
gnu по списку cp deborphan df feh ...

{% highlight shell %}
# Load completion from bash, which isn't available in zsh yet.
bash_completions=()
if [ -n "$commands[vzctl]" ] ; then
  bash_completions+=(/etc/bash_completion.d/vzctl.sh)
fi
if (( $#bash_completions )); then
  if ! which complete &>/dev/null; then
    autoload -Uz bashcompinit
    if which bashcompinit &>/dev/null; then
      bashcompinit
    fi
  fi
  bash_source /etc/bash_completion.d/vzctl.sh
fi
{% endhighlight %}

Загрузка автокомплита из bash-completions.

{% highlight shell %}
expand-or-complete-with-dots() {
    echo -n "\e[31m......\e[0m"
    zle expand-or-complete
    zle redisplay
}
zle -N expand-or-complete-with-dots

function expand-or-complete-with-dots() {
    echo -n "\e[36m-=--...--=-\e[0m"
    zle expand-or-complete
    zle redisplay
}
zle -N expand-or-complete-with-dots
bindkey "^I" expand-or-complete-with-dots

{% endhighlight %}

При нажатии на tab показывает три точки. Это просто украшательство, которое
будет работать в случае долгого ответа системы автокомплита, например это
возможно при обращении к удаленному ресурсу. Почему-то работает лично у меня
только в linux-консоли, я особо не заморачивался почему, возможно из-за
высокой скорости st/urxvt.
