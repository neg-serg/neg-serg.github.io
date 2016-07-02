---
layout: post
title: UNIX (s)hell, pt2
categories: personal
tags: 
  - zsh
comments: true
mathjax: null
featured: true
published: true
---

Это продолжение нарциссической серии статей про мой конфиг zsh. Здесь будут
рассмотрены некоторые полезные глобальные переменные окружения(env),
различные мелочи, helper'ы, приглашение командной строки и основные функции.
Это не всё что есть, лишь то что я считаю важным, за полным конфигом
в актуальном состоянии всегда можно слазить на мой гитхаб. В конфиге
биндинги, функции и тому подобное строго разделены. Тут это будет не так,
чтобы было более понятно. По сути это представляет собой просто обзор того
что может быть, а не руководство к действию.

<!--excerpt-->

* TOC
{:toc}

# Exports

#### PATH

Многим часто приходится задавать кастомный $PATH(список директорий для поиска
исполняемых файлов), который может разрастаться до больших размеров и его
относительно сложно редактировать, возможным решением может быть что-то вроде
этого:

{% highlight shell %}
path_dirs=(
    /usr/{s,}bin
    /usr/lib64/notion/bin
    {/usr/local,}/{s,}bin
	/usr/bin/{site,vendor,core}_perl
	${HOME}/.rvm/bin
	${BIN_HOME}/{,go/bin}
    /opt/android-sdk/platform-tools
    /mnt/home/.local/bin
    /opt/cuda/bin
)

whence ruby >/dev/null && \
    path_dirs+=$($(whence ruby) -e 'puts Gem.user_dir')/bin

export PATH=${(j_:_)path_dirs}
{% endhighlight %}

Здесь совмещены возможности globbing'а(перечисления) и возможность не
заморачиваться по поводу разделителя в виде двоеточия(:). Для этого
использован zsh массив и одна из возможностей extended globbing'а zsh(join
строк).

Многие современные программы используют по-умолчанию XDG-пути для тех же
конфигов, вместо того чтобы засорять $HOME, это хорошая практика, но мне
удобно задавать такого рода директории вручную, чтобы другие программы не
преподносили мне никаких неприятных сюрпризов типа создания в HOME
~/Downloads или их локализованных версий и тд и тп.

{% highlight shell %}
export XDG_CONFIG_HOME="${HOME}/.config"
export XDG_DATA_HOME="${HOME}/.local/share"
export XDG_CACHE_HOME="${HOME}/.cache"
export XDG_DOWNLOAD_DIR="${HOME}/dw"
export XDG_MUSIC_DIR="${HOME}/music"
export XDG_DESKTOP_DIR="${HOME}/.local/desktop"
export XDG_TEMPLATES_DIR="${HOME}/1st_level/templates"
export XDG_DOCUMENTS_DIR="${HOME}/doc/"
export XDG_PICTURES_DIR="${HOME}/pic"
export XDG_VIDEOS_DIR="${HOME}/vid"
export XDG_PUBLICSHARE_DIR="${HOME}/1st_level/upload/share"
{% endhighlight %}

{% highlight shell %}
fpath=(${HOME}/.zsh/zsh-completions/src $fpath)
{% endhighlight %}

fpath это как path, только для директорий. Если честно не самая используемая
мной возможность.

{% highlight shell %}
export VIDIR_EDITOR_ARGS='-c :set nolist | :set ft=vidir-ls'
{% endhighlight %}

Настройки, которые используются для vimv -- консольного переименователя
файлов, так чтобы использовался vim и подсветка синтаксиса.

{% highlight shell %}
export GREP_COLOR='37;45'
export GREP_COLORS='ms=0;32:mc=1;33:sl=:cx=:fn=1;32:ln=1;36:bn=36:se=1;30'
{% endhighlight %}

Я не люблю красный цвет и вместо него для подсветки того, что нашел греп
использую циановый.

{% highlight shell %}
for q in vim nvim vi; 
    { [[ -n ${commands}[(I)${q}] ]] \
    && export EDITOR=${q}; break }
export VISUAL="${SCRIPT_HOME}/v_standalone"
{% endhighlight %}

По-умолчанию в качестве графического редактора используется мой враппер для
vim, в котором используется st + vim + tmux(с отдельным сокетом), рассказано о нем будет чуть
позже.

{% highlight shell %}
export INPUTRC=${HOME}/.config/inputrc
{% endhighlight %}

Расположения конфига для readline-использующих программ и rlwrap в ~/.config.

{% highlight shell %}
export BROWSER="firefox"
{% endhighlight %}

Firefox мой браузер по-умолчанию благодаря продвинутому плагину
vimperator.

{% highlight shell %}
if [[ -x "$(which "vimpager" 2>/dev/null)" ]]; then
    export PAGER="vimpager -u ~/.vim/vimpagerrc" SDCV_PAGER=${PAGER}
    alias less=${PAGER}
    alias zless=${PAGER}
else
    export PAGER="less"
fi
{% endhighlight %}

В качестве PAGER по-умолчанию используется vimpager. Фактически это vim для
man страниц. И быстро и удобно.

{% highlight shell %}
export LESSCHARSET=UTF-8
# support colors in less
export LESS_TERMCAP_mb=$'\e[1;31m'     # begin blinking
export LESS_TERMCAP_md=$'\e[1;35m'     # begin bold
export LESS_TERMCAP_me=$'\e[0m'        # end mode
export LESS_TERMCAP_so=$'\e[1;40;36m'  # begin standout - info box
export LESS_TERMCAP_se=$'\e[0m'        # end standout
export LESS_TERMCAP_us=$'\e[1;32m'     # begin underline
export LESS_TERMCAP_ue=$'\e[0m'        # end underline
{% endhighlight %}

Настройки цветов для LESS и man(по-умолчанию)

{% highlight shell %}
export HISTFILE=${HOME}/.zsh_history
export HISTSIZE=50000
export SAVEHIST=100000 # useful for setopt append_history
export HISTIGNORE="&:ls:[bf]g:exit:reset:clear:cd*"
export HISTCONTROL=ignoreboth:erasedups # ignoreboth (= ignoredups + ignorespace)
{% endhighlight %}

Я люблю историю больших размеров, без дубликатов.

{% highlight shell %}
export ACK_COLOR_MATCH="cyan bold"
export ACK_COLOR_FILENAME="cyan bold on_black"
export ACK_COLOR_LINENO="bold green"
{% endhighlight %}

Перенастройка цветов для ack, более медленного, но и более функционального
аналога grep. 

{% highlight shell %}
export WORDCHARS='*?_-.[]~&;!#$%^(){}<>~` '
{% endhighlight %}

Эта переменная задает список разделителей, которые учавствуют, в частности
в определении того что считать словом. Например если вы нажимаете
C-w(удаление слова) и у вас только / в списке WORDCHARS, то удаление будет
идти до него, независимо от пробелов.

#### dirstack

{% highlight shell %}
DIRSTACKSIZE=${DIRSTACKSIZE:-20}
DIRSTACKFILE=${HOME}/.zsh/.99-zdirs

if [[ -f ${DIRSTACKFILE} ]] && [[ ${#dirstack[*]} -eq 0 ]] ; then
    dirstack=( ${(f)"$(< $DIRSTACKFILE)"} )
    # "cd -" won't work after login by just setting $OLDPWD, so
    [[ -d $dirstack[1] ]] && cd $dirstack[1] && cd $OLDPWD
fi

{% endhighlight %}

#### keybindings

Я использую прыжки по стеку директорий назад с помощью C-=, очень удобно.
Биндинги задаются в отдельной секции.

{% highlight shell %}
export KEYTIMEOUT=5 # allow to use ,<key> more fast
export ESCDELAY=1
{% endhighlight %}

Важные и полезные в плане интерактивной работы опции. Задают клавиатурные
задержки для комбинаций клавиш в zsh поменьше. В часности используется для
abbr. Тогда при работе нажатие запятой, которая у меня используется как
префикс для их разворота не придется испытывать долгой задержки.

#### java

{% highlight shell %}
export JAVA_FONTS=/usr/share/fonts/TTF
export _JAVA_AWT_WM_NONREPARENTING=1
export _JAVA_OPTIONS='-Dawt.useSystemAAFontSettings=on -Dswing.aatext=true -Dswing.defaultlaf=com.sun.java.swing.plaf.gtk.GTKLookAndFeel -Dswing.crossplatformlaf=com.sun.java.swing.plaf.gtk.GTKLookAndFeel'
_SILENT_JAVA_OPTIONS="${_JAVA_OPTIONS}"
unset _JAVA_OPTIONS
{% endhighlight %}

Добавляет в java приложения чуть лучшие с точки зрения отображения опции,
а также улучшает совместимость с некоторыми оконными менеджерами. Сама java
используется следующим образом:

{% highlight shell %}
alias java='java "$_SILENT_JAVA_OPTIONS"'
{% endhighlight %}

Это нужно чтобы не писать всякий лишний мусор в stdout

{% highlight shell %}
if which drip > /dev/null 2>&1; then
  export DRIP_SHUTDOWN=30
  export JAVACMD=$(which drip)
fi
{% endhighlight %}

drip это как java, только быстрее. Позволяет быстро запускать
java-приложения. Особенно это заметно на clojure, lein.

{% highlight shell %}
export FZF_DEFAULT_COMMAND='ag -l -g ""'
export FZF_CTRL_T_COMMAND="$FZF_DEFAULT_COMMAND"
export FZF_DEFAULT_OPTS="${FZF_DEFAULT_OPTS} --color=16"
{% endhighlight %}

fzf это классный standalone fuzzy-finder, тут собраны небольшие настройки для
него, в частности по-умолчанию поиск выполняется с помощью ag(это такой ещё
один аналог grep, на этот раз более быстрый).

{% highlight shell %}
export PULSE_LATENCY_MSEC=60

export QEMU_AUDIO_DRV=pa
export QEMU_PA_SOURCE=input
export QEMU_PA_SINK=alsa_output.usb-E-MU_Systems__Inc._E-MU_0204___USB_E-MU-A8-3F19-07DC0212-089A1-8740AT2A-00.analog-stereo
{% endhighlight %}

Задается другая латентность для pulseaudio приложений. В общем может
оказаться полезным(избавить от скрежета в приложениях, которые пускаются под
wine, например), а может и нет. Похоже на историзм. Также добавлена поддержка
моей звуковой карточки pulseaudio в эмулятор QEMU.

{% highlight shell %}
export SXHKD_FIFO="/tmp/sxhkd_fifo"
export SXHKD_SHELL="zsh"

export vim_server_name="VIM"
export wim_font="PragmataPro for Powerline"
export wim_font_s="Mensch:size=14"
export wim_font_size=20
export wim_sock_path="${HOME}/1st_level/vim.socket"
export wim_timer=".1s"

export nvim_server_addr=/tmp/nvimsocket
export NVIM_LISTEN_ADDRESS=/tmp/nvimsocket
export nwim_font="${wim_font}"
export nwim_font_s="${wim_font_s}"
export nwim_font_size=${wim_font_size}
export nwim_timer=".1s"

(){
    local _home="/mnt/home"
    local _dev="/one/dev"
    hash -d dev=${_dev}
    hash -d doc="${_home}/doc"
    hash -d nv="${_dev}/src/1st_level/neovim"
    hash -d torrent="${_home}/torrent"
    hash -d v="${_home}/vid"
    hash -d z="${_dev}/src"
    hash -d p='/home/neg/pic'
}
{% endhighlight %}

Далее идут параметры для моего vim-wrapper'а и хеши путей. Я считаю такую
раскладку удобной и вам рекомендую.)


# Helpers

Располагаются у меня в 03-helpers.zsh, нужны для украшательства строк,
использутся, выглядит это, например, вот так:

![fancyHi](https://i.imgur.com/N27HFsR.png)

Это соответственно мой лаунчер для vim и mpv. Эти функии представляют собой
просто набор утилитарных хаков, но если кому интересно могу объяснить.

# Prompt

Мой prompt довольно сложен, потому что сам раскрашивает пути, по сути
представляет собой переделку из fizsh(попытка сделать zsh похожим на fish,
в целом успешная, как мне кажется). Также справа выполняется индикация
текущего режима, если используется vi-cmd-mode. Здесь, как и везде,
я использую смесь из биндингов vim и emacs. Также выводится
осмысленное(насколько это возможно) сообщение по поводу кода ошибки последней
команды. Вот он, код:

{% highlight shell %}
export _zsh_oldprompt_is_sourced="true"

local _lambda_ok="" # "%{$fg_bold[green]%}=λ\\"
local _lambda_err="%{$fg_bold[red]%}=λ\\"
local _lambda="%(?,${_lambda_ok},${_lambda_err})%{${reset_color}%}"

function nice_exit_code() {
    local exit_status="${1:-$(print -P %?)}";
    # nothing to do here
    [[ -z $exit_status || $exit_status == 0 ]] && return;

    local sig_name;
    
    # is this a signal name (error code = signal + 128) ?
    case $exit_status in
        129)  sig_name=HUP ;;
        130)  sig_name=INT ;;
        131)  sig_name=QUIT ;;
        132)  sig_name=ILL ;;
        134)  sig_name=ABRT ;;
        136)  sig_name=FPE ;;
        137)  sig_name=KILL ;;
        139)  sig_name=SEGV ;;
        141)  sig_name=PIPE ;;
        143)  sig_name=TERM ;;
    esac

    # usual exit codes
    case $exit_status in
        -1)   sig_name=FATAL ;;
        1)    sig_name=WARN ;; # Miscellaneous errors, such as "divide by zero"
        2)    sig_name=BUILTINMISUSE ;; # misuse of shell builtins (pretty rare)
        126)  sig_name=CCANNOTINVOKE ;; # cannot invoke requested command (ex : source script_with_syntax_error)
        127)  sig_name=CNOTFOUND ;; # command not found (ex : source script_not_existing)
    esac

    # assuming we are on an x86 system here
    # this MIGHT get annoying since those are in a range of exit codes
    # programs sometimes use.... we'll see.
    case $exit_status in
        19)  sig_name=STOP ;;
        20)  sig_name=TSTP ;;
        21)  sig_name=TTIN ;;
        22)  sig_name=TTOU ;;
    esac

    echo "%{$fg[blue]%}[%{$fg[white]%}$ZSH_PROMPT_EXIT_SIGNAL_PREFIX${exit_status}:${sig_name:-$exit_status}%{$fg[blue]%}]%{$reset_color%}"
}

case ${UID} in
0)
    PROMPT="%{$fg[red]%}<%T>%{$fg[red]%}[root@%m] %(!.#.$) %{${reset_color}%}%{${fg[red]}%}[%~]%{${reset_color}%} "
    PS1="${PROMPT}"
    PROMPT2="%{${fg[red]}%}%_> %{${reset_color}%}"
    PS2="${PROMPT2}"
    SPROMPT="%{${fg[red]}%}correct: %R -> %r [nyae]? %{${reset_color}%}"
    RPROMPT="%{${fg[cyan]}%}[%~]%{${reset_color}%} "
    ;;
*)
    DARK_BLUE="%{"$'\033[00;38;5;4m'"%}"
    _neg_user_pretok="${DARK_BLUE}[${NOCOLOR}"
    function precmd(){ 
        # _neg_hashes="$(hash -d|tr -d \')"
        # while read i; do 
        #     what_change=$(echo ${_neg_tilda_path}|grep -o "$(awk -F '=' '{print $2}' <<< ${i})")
        #     if [[ ${what_change} != "" ]]; then
        #         fst_arg=$(awk -F '=' '{print $1}' <<< ${i})
        #         _neg_tilda_path=${fst_arg}
        #     fi
        # done <<< ${_neg_hashes[@]}
        export PS1="${_neg_user_pretok}%40<..<$(${ZSH}/neg-prompt)" 
    }
    PS2="%{$fg[magenta]%}» %{$reset_color%}"
    # selection prompt used within a select loop.
    PS3='?# '
    # the execution trace prompt (setopt xtrace). default: '+%N:%i
    PS4='+%N:%i:%_> '
    function vi_mode_prompt_info() { [[ "${RPS1}" == "" && "${RPROMPT}" == "" ]] && RPS1='${_lambda}$(vi_mode_prompt_info)' }

    # if mode indicator wasn't setup by theme, define default
    [[ "${mode_ind}" == "" ]] && mode_ind="%{$fg[blue]%}[%{$fg[white]%}<<%{$fg[blue]%}]%{$reset_color%}"

    function vi_mode_prompt_info() {
        echo "${${KEYMAP/vicmd/${mode_ind}}/(main|viins)/}"
    }
    # define right prompt, if it wasn't defined by a theme
    [[ "${RPS1}" == "" && "${RPROMPT}" == "" ]] && RPS1='$(nice_exit_code)${_lambda}$(vi_mode_prompt_info)'
    ;;
esac
{% endhighlight %}


Основная же работа выполняется в этом скрипте:


{% highlight shell %}
#!/usr/bin/zsh
# -*- mode: zsh; sh-indentation: 2; indent-tabs-mode: nil; sh-basic-offset: 2; -*-
# vim: ft=zsh sw=2 ts=2 et

local red_color="%{"$'\033[03;31m'"%}"
local green_color="%{"$'\033[03;32m'"%}"
local white_color="%{"$'\033[00;38;5;7m'"%}"
local dark_blue_color="%{"$'\033[00;38;5;4m'"%}"
local blue_color="%{"$'\033[00;38;5;61m'"%}"
local purple_color="%{"$'\033[00;38;5;91m'"%}"
local black_color="%{"$'\033[00m'"%}"

[[ $UID -ne 0 ]] && _neg_promptcolor=${green_color} && _neg_user_pretoken="${dark_blue_color}[${NOCOLOR}"
[[ $UID -ne 0 ]] && _neg_promptcolor=${green_color} && _neg_user_token="${dark_blue_color}] ${purple_color}>>${NOCOLOR} "
[[ $UID -eq 0 ]] && _neg_promptcolor=${red_color} && _neg_user_token="# "

setopt sh_word_split
_neg_dyn_pwd=""
_neg_full_path="$(pwd)"
_neg_tilda_path=${_neg_full_path/${HOME}/\~}
# write the home directory as a tilda
[[ ${_neg_tilda_path[2,-1]} == "/" ]] && _neg_tilda_path=${_neg_tilda_path[2,-1]}
# otherwise the first element of split_path would be empty.
_neg_forwards_in_tilda_path=${_neg_tilda_path//[^["\/"]/}
# remove everything that is not a "/"
_neg_number_of_elements_in_tilda_path=$(( $#_neg_forwards_in_tilda_path + 1 ))
# we removed the first forward slash, so we need one more element than the number of slashes
_neg_saveIFS="${IFS}"
IFS="/"
_neg_split_path=(${_neg_tilda_path})
_neg_start_of_loop=1
_neg_end_of_loop=${_neg_number_of_elements_in_tilda_path}
for i in {$_neg_start_of_loop..$_neg_end_of_loop}; do
    if [[ $i == $_neg_end_of_loop ]]; then
        _neg_to_be_added=$_neg_split_path[i]'/'
        _neg_dyn_pwd=${_neg_dyn_pwd}${_neg_to_be_added}
    else
        _neg_to_be_added=${_neg_split_path[i]}
        _neg_to_be_added=${_neg_to_be_added}"${blue_color}/${white_color}"
        # _neg_to_be_added=$_neg_to_be_added[1,1]'/'
        _neg_dyn_pwd=${_neg_dyn_pwd}${_neg_to_be_added}
    fi
done
unsetopt sh_word_split
IFS=${_neg_saveIFS}
[[ ${_neg_full_path/${HOME}/\~} != ${_neg_full_path} ]] && _neg_dyn_pwd=${_neg_dyn_pwd/\/~/~}
# remove the slash in front of ${HOME}
_neg_prompt="%F${_neg_promptcolor}${_neg_dyn_pwd[0,-2]}${black_color}$_neg_user_token%b%k%f"
# _neg_prompt="%n@%m%F${_neg_promptcolor} $_neg_dyn_pwd[0,-2]${black_color}$_neg_user_token%b%k%f"
echo ${_neg_prompt}
{% endhighlight %}

# Functions

Сейчас будет обзор функций, которые у меня используются в zsh, одной из самых
полезных частей конфига ,скорее всего пространный. Ворнинг! Полный текст всех
функций занимает где-то 700 loc, просьба убрать беременных детей-наркоманов!


{% highlight shell %}
# Dir reload
.() { [[ $# = 0 ]] && cd . || builtin . "$@"; }
{% endhighlight %}

Мелкая приблуда для того чтобы точка не считалась чем-то плохим.

{% highlight shell %}
function chpwd() {
    if [ -x ${BIN_HOME}/Z ]; then
        [ "${PWD}" -ef "${HOME}" ] || Z -a "${PWD}"
    fi
    if [[ ${_zsh_oldprompt_is_sourced} == "true" ]]; then
        export PS1="${_neg_user_pretok}%40<..<$(${ZSH}/neg-prompt)"
    fi
}
{% endhighlight %}

Функция, которая выполняется при смене директории, нужна при обновлении
prompt. Также используется в Z, программе, которая накапливает стек
использованных файлов и директорий для дальшейших прыжков по директориям
и файлам с помощью fasd. Например если вы помните что где-то в истории у вас
был mathSpace.cc, то вы можете набрать mathS C-x C-a и получить его полный
путь, это очень удобно.

{% highlight shell %}
function magic-abbrev-expand() {
    local MATCH
    LBUFFER=${LBUFFER%%(#m)[_a-zA-Z0-9]#}
    LBUFFER+=${abbreviations[$MATCH]:-$MATCH}
    zle self-insert
}

function no-magic-abbrev-expand() { LBUFFER+=' ' }
{% endhighlight %}

Функция для разворота сокращений. Вы можете нажать ,G и получить |& grep -i
и тп.

{% highlight shell %}
function zc(){
    local cache="${ZSH}/cache"
    autoload -U compinit zrecompile
    compinit -d "${cache}/zcomp-${HOST}"
    for z in ${ZSH}/*.zsh ${HOME}/.zshrc; do 
        zcompile ${z}; 
        echo $(_zpref) $(_zfwrap "${z}"); 
    done
    for f in ${HOME}/.zsh/.zshrc "${cache}/zcomp-${HOST}"; do
        zrecompile -p ${f} && command rm -f ${f}.zwc.old
    done
    source ${HOME}/.zshrc
}

# completion system
if zrcautoload compinit ; then
    compinit || print 'Notice: no compinit available :('
else
    print 'Notice: no compinit available :('
    function zstyle { }
    function compdef { }
fi
{% endhighlight %}

Функция для компиляции файлов zsh. Вроде как они грузятся быстрее. Что уж
точно, так это то что выкидывается информация о форматировании кода
в функциях, так что which zc покажет нам не то что здесь написано, если файл
был скомпилирован.

{% highlight shell %}
function pk () {
    if [ $1 ] ; then
        case $1 in
            tbz)    tar cjvf $2.tar.bz2 $2                ;;
            tgz)    tar czvf $2.tar.gz  $2                ;;
            tar)    tar cpvf $2.tar  $2                   ;;
            bz2)    bzip $2                               ;;
            gz)     gzip -c -9 -n $2 > $2.gz              ;;
            zip)    zip -r $2.zip $2                      ;;
            7z)     7z a $2.7z $2                         ;;
            *)      echo "'$1' cannot be packed via pk()" ;;
        esac
    else
        echo "'$1' is not a valid file"
    fi
}
{% endhighlight %}

Функция для запаковки, можно просто написать pk zip music2 чтобы получить
music2.zip. Это просто и вызывает привыкание.

{% highlight shell %}
function extract() {
  local remove_archive
  local success
  local file_name
  local extract_dir

  if (( $# == 0 )); then
    echo "Usage: extract [-option] [file ...]"
    echo
    echo Options:
    echo "    -r, --remove    Remove archive."
    echo
    echo "Report bugs to <sorin.ionescu@gmail.com>."
  fi

  remove_archive=1
  if [[ "$1" == "-r" ]] || [[ "$1" == "--remove" ]]; then
    remove_archive=0 
    shift
  fi

  while (( $# > 0 )); do
    if [[ ! -f "$1" ]]; then
      echo "extract: '$1' is not a valid file" 1>&2
      shift
      continue
    fi

    success=0
    file_name="$( basename "$1" )"
    extract_dir="$( echo "$file_name" | sed "s/\.${1##*.}//g" )"
    case "$1" in
      (*.tar.gz|*.tgz) [ -z $commands[pigz] ] && tar zxvf "$1" || pigz -dc "$1" | tar xv ;;
      (*.tar.bz2|*.tbz|*.tbz2) tar xvjf "$1" ;;
      (*.tar.xz|*.txz) tar --xz --help &> /dev/null \
        && tar --xz -xvf "$1" \
        || xzcat "$1" | tar xvf - ;;
      (*.tar.zma|*.tlz) tar --lzma --help &> /dev/null \
        && tar --lzma -xvf "$1" \
        || lzcat "$1" | tar xvf - ;;
      (*.tar) tar xvf "$1" ;;
      (*.gz) [ -z $commands[pigz] ] && gunzip "$1" || pigz -d "$1" ;;
      (*.bz2) bunzip2 "$1" ;;
      (*.xz) unxz "$1" ;;
      (*.lzma) unlzma "$1" ;;
      (*.Z) uncompress "$1" ;;
      (*.zip|*.war|*.jar|*.sublime-package) unzip "$1" -d $extract_dir ;;
      (*.rar) unrar x -ad "$1" ;;
      (*.7z) 7za x "$1" ;;
      (*.deb)
        mkdir -p "$extract_dir/control"
        mkdir -p "$extract_dir/data"
        cd "$extract_dir"; ar vx "../${1}" > /dev/null
        cd control; tar xzvf ../control.tar.gz
        cd ../data; tar xzvf ../data.tar.gz
        cd ..; rm *.tar.gz debian-binary
        cd ..
      ;;
      (*) 
        echo "extract: '$1' cannot be extracted" 1>&2
        success=1 
      ;; 
    esac

    (( success = $success > 0 ? $success : $? ))
    (( $success == 0 )) && (( $remove_archive == 0 )) && rm "$1"
    shift
  done
}
{% endhighlight %}

Функция распаковки архивов, была взята, насколько я помню из GRML

{% highlight shell %}
function up-one-dir   { pushd .. > /dev/null; zle redisplay; zle -M $(pwd);  }
function back-one-dir { popd     > /dev/null; zle redisplay; zle -M $(pwd);  }
zle -N up-one-dir
zle -N back-one-dir
{% endhighlight %}

Функции для того чтобы прыгать назад-вперед

{% highlight shell %}
function xkcdrandom(){ wget -qO- dynamic.xkcd.com/comic/random|tee >(feh $(grep -Po '(?<=")http://imgs[^/]+/comics/[^"]+\.\w{3}'))|grep -Po '(?<=(\w{3})" title=").*(?=" alt)';}
function xkcd(){ wget -qO- http://xkcd.com/|tee >(feh $(grep -Po '(?<=")http://imgs[^/]+/comics/[^"]+\.\w{3}'))|grep -Po '(?<=(\w{3})" title=").*(?=" alt)';}
{% endhighlight %}

Функция, которая позволяет скачивать xkcd комиксы. Добавлена, потому что
выглядит устрашающе :D.

{% highlight shell %}
# just type '...' to get '../..'
function rationalise-dot() {
    local MATCH
    if [[ $LBUFFER =~ '(^|/| |  |'$'\n''|\||;|&)\.\.$' ]]; then
        LBUFFER+=/
        zle self-insert
        zle self-insert
    else
        zle self-insert
    fi
}
zle -N rationalise-dot
{% endhighlight %}

Функция, которая конвертирует ... в ../.. прямо на ходу с помощью ZLE.
Удобно.

{% highlight shell %}
# jump behind the first word on the cmdline.
# useful to add options.
function jump_after_first_word() {
    local words
    words=(${(z)BUFFER})

    if (( ${#words} <= 1 )) ; then
        CURSOR=${#BUFFER}
    else
        CURSOR=${#${words[1]}}
    fi
}
zle -N jump_after_first_word
{% endhighlight %}

ZLE-функция, которая позволяет прыгнуть после первого слова команды. В теории
это удобно, но вообще историзм, надо бы убрать.

{% highlight shell %}
# grep for running process, like: 'any vime
function any() {
    emulate -L zsh
    unsetopt KSH_ARRAYS
    if [[ -z "$1" ]] ; then
        if [ ! -x $(which peco) ]; then
            echo "any - grep for process(es) by keyword" >&2
            echo "Usage: any <keyword>" >&2 ; return 1
        else
            ps xauwww | peco --layout=bottom-up
        fi
    else
        ps xauwww | grep  --color=auto -i "[${1[1]}]${1[2,-1]}"
    fi
}
{% endhighlight %}

Удобная функция для grep или интерактивного поиска по запущенным процессам,
использую практически каждый день.

{% highlight shell %}
function inplace_mk_dirs() {
    # Press ctrl-xM to create the directory under the cursor or the selected area.
    # To select an area press ctrl-@ or ctrl-space and use the cursor.
    # Use case: you type "mv abc ~/testa/testb/testc/" and remember that the
    # directory does not exist yet -> press ctrl-XM and problem solved
    local PATHTOMKDIR
    if ((REGION_ACTIVE==1)); then
        local F=$MARK T=$CURSOR
        if [[ $F -gt $T ]]; then
            F=${CURSOR}
            T=${MARK}
        fi
        # get marked area from buffer and eliminate whitespace
        PATHTOMKDIR=${BUFFER[F+1,T]%%[[:space:]]##}
        PATHTOMKDIR=${PATHTOMKDIR##[[:space:]]##}
    else
        local bufwords iword
        bufwords=(${(z)LBUFFER})
        iword=${#bufwords}
        bufwords=(${(z)BUFFER})
        PATHTOMKDIR="${(Q)bufwords[iword]}"
    fi
    [[ -z "${PATHTOMKDIR}" ]] && return 1
    PATHTOMKDIR=${~PATHTOMKDIR}
    if [[ -e "${PATHTOMKDIR}" ]]; then
        zle -M " path already exists, doing nothing"
    else
        zle -M "$(mkdir -p -v "${PATHTOMKDIR}")"
        zle end-of-line
    fi
}
{% endhighlight %}

Тоже очень частоиспользуемая функция, которая позволяет по C-x Shift-M
создать директорию прямо на ходу без mkdir, если вы её ещё не создали,
работает рекурсивно.

{% highlight shell %}
# If I am using vi keys, I want to know what mode I'm currently using.
# zle-keymap-select is executed every time KEYMAP changes.
# From http://zshwiki.org/home/examples/zlewidgets
function zle-keymap-select {
    VIMODE="${${KEYMAP/vicmd/ M:command}/(main|viins)/}"
    zle reset-prompt
}
{% endhighlight %}

Обертка вокруг смены режима vi-mode.

{% highlight shell %}
function imv() {
    local src dst
    for src; do
        [[ -e ${src} ]] || { print -u2 "${src} does not exist"; continue }
        dst=${src}
        vared dst
        [[ ${src} != ${dst} ]] && mkdir -p ${dst:h} && mv -n ${src} ${dst}
    done
}
{% endhighlight %}

Тоже полезная функция, которая позволяет отредактирвоать имя файла
интерактивно с помощью zsh. Это такое индивидуальное переименование без vimv,
используется мной тоже достаточно часто.

{% highlight shell %}
function pstop() {
    ps -eo pid,user,pri,ni,vsz,rsz,stat,pcpu,pmem,time,comm --sort -pcpu |
    head "${@:--n 20}" | 
    column -t
}
{% endhighlight %}

Что-то вроде неинтерактивного top. Мне нравится его использовать.

{% highlight shell %}
function finfo() {
while [ $# -gt 0 ] ; do
    mime_type=$(file -L -b --mime-type "$1")
    case $mime_type in
        image/svg+xml) inkscape -S "$1" ;;
        video/* | application/vnd.rn-realmedia | audio/* | image/*) mediainfo "$1" ;;
        application/pdf) pdfinfo "$1" ;;
        application/zip) unzip -l "$1" ;;
        application/x-lha) lha -l "$1" ;;
        application/x-rar) unrar lb "$1" ;;
        application/x-bittorrent) aria2c -S "$1" | grep '\./' ;;
        application/x-iso9660-image) isoinfo -J -l -i "$1" ;;
        *)
            case "$1" in
                *.torrent) aria2c -S "$1" | grep '\./' ;;
                *.mkv) mediainfo "$1" ;;
                *.ace) unace l "$1" ;;
                *.icns) icns2png -l "$1" ;;
                *) file -b "$1" ;;
            esac
            ;;
    esac
    shift
done
}
{% endhighlight %}

Выдать информацию о файле в зависимости от его типа.

{% highlight shell %}
function rfc(){
    uri_tmpl='http://www.rfc-editor.org/rfc/rfc%d.txt';
    rfcn=$( printf $1 | sed 's/[^0-9]//g' );
    [[ -z "${rfcn}" ]] && echo  "Usage: rfc <RFC>"; exit;
    uri=$( printf ${uri_tmpl} ${rfcn} )
    curl --silent ${uri} | ${PAGER}
}
{% endhighlight %}

Посмотреть rfc из интернета по данному номеру

{% highlight shell %}
function myip(){
    [[ -n "$1" ]] && sleep $1;
    [[ -x $(which dig) ]] && DNSQUERY=$(dig +short myip.opendns.com @resolver1.opendns.com)

    if [[ -n "$DNSQUERY" ]]; then
        echo "$DNSQUERY"
    else
        ( wget -O- -T2 -q http://noone.org/cgi-bin/whatsmyip.cgi || \
        wget -O- -T2 -q http://v4.showip.spamt.net/ || \
        wget -O- -T2 -q http://showipv6.de/shortv4only || \
        wget -O- -T2 -q http://checkip.dyndns.org/ | sed -e 's|^.*<body>Current IP Address: ||;s|</body>.*$||;s|^50 0 .*|N/A|;' || \
        wget -O- -T2 -q http://ente.limmat.ch/myip | fgrep 'Your IP' | sed -e 's|^.*(||;s|).*$||;s|^500 .*|N/A|;' | \
        echo N/A ) | \
        sed -e 's:^$:N/A:'
    fi
}
{% endhighlight %}

Показывает текущий ip.

{% highlight shell %}
function fg-widget() {
    stty icanon echo -inlcr < /dev/tty
    stty lnext '^V' quit '^\' susp '^Z' < /dev/tty
    zle reset-prompt
    if jobs %- >/dev/null 2>&1; then
        fg %-
    else
        fg
    fi
}
zle -N fg-widget
{% endhighlight %}

{% highlight shell %}
#pcp - copy files matching pattern $1 to $2
function pcp(){ find . -regextype awk -iregex ".*$1.*" -print0 | xargs -0 cp -vR -t "$2" }
{% endhighlight %}

{% highlight shell %}
fasd_cache="$HOME/bin/.fasd-init-cache"
if [ "$(command -v fasd)" -nt "$fasd_cache" -o ! -s "$fasd_cache" ]; then
    fasd --init auto >| "$fasd_cache"
fi
source "$fasd_cache"
unset fasd_cache
{% endhighlight %}

Инициализация fasd, который прыгает по файлам и директориям, о котором я уже
писал выше в этой статье

{% highlight shell %}
function copy-to-clipboard() {
    (( $+commands[xclip] )) || return
    echo -E -n - "$BUFFER" | xclip -i
}
zle -N copy-to-clipboard
{% endhighlight %}

ZLE-функция для копирования в буфер обмена

{% highlight shell %}
function ff() { find . -iregex ".*$@.*" -printf '%P\0' | xargs -r0 ls --color=auto -1d }
{% endhighlight %}

Поиск по regex c помощью find с подсветкой как в прописано в dircolors(для
этого используется ls).

{% highlight shell %}
function magnet_to_torrent() {
    [[ "$1" =~ xt=urn:btih:([^\&/]+) ]] || return 1
    hashh=${match[1]}
    if [[ "$1" =~ dn=([^\&/]+) ]];then
      filename=${match[1]}
    else
      filename=${hashh}
    fi
    echo "d10:magnet-uri${#1}:${1}e" > "${filename}.torrent"
}
{% endhighlight %}

Преобрвазование magnet-ссылки в torrent-файл.

{% highlight shell %}
function hi2() {
    if [ ! -x $(which pygmentize) ]; then
        echo package \'pygmentize\' is not installed!
        exit -1
    fi

    if [ $# -eq 0 ]; then
        pygmentize -g $@
    fi

    for file_name in "$@"; do
        filename=$(basename "${file_name}")
        lexer=$(pygmentize -N \"$filename\")
        if [ "Z$lexer" != "Ztext" ]; then
            pygmentize -l $lexer "${file_name}"
        else
            pygmentize -g "${file_name}"
        fi
    done
}
{% endhighlight %}

Цветной cat, поторый подсвечивает синтаксис файла  с помощью pygmentize.
В директории ~/bin ещё лежит такой, который работает с помощью vim.

{% highlight shell %}
function doc2pdf () { curl -\# -F inputDocument=@"$1" http://www.doc2pdf.net/convert/document.pdf > "${1%.*}.pdf" }
{% endhighlight %}

Конвертация из doc в pdf. Я не люблю {libre,open}office, а использовать
google docs на каждый чих не всегда удобно.

{% highlight shell %}
function sprunge() {
    if [[ -t 0 ]]; then
      echo Running interactively, checking for arguments... >&2
      if [[ "$*" ]]; then
        echo Arguments present... >&2
        if [[ -f "$*" ]]; then
          echo Uploading the contents of "$*"... >&2
          cat "$*"
        else
          echo Uploading the text: \""$*"\"... >&2
          echo "$*"
        fi | curl -F 'sprunge=<-' http://sprunge.us
      else
        echo No arguments found, printing USAGE and exiting. >&2
        usage
      fi
    else
      echo Using input from a pipe or STDIN redirection... >&2
      curl -F 'sprunge=<-' http://sprunge.us
    fi
}
{% endhighlight %}

Функция для загрузки текста на sprunge. Это аналог pastebin, только более
минималистичный и мне нравится намного больше.

{% highlight shell %}
function clock(){
    while sleep 1; do 
        tput sc
        tput cup 0 $(($(tput cols)-29))
        date
        tput rc
    done &
}
{% endhighlight %}

Функция-хак, которая вводит часы вверху экрана. Можно подивить ей знакомых,
хотя работает она не слишком стабильно. Не особо нужна когда есть tmux.

{% highlight shell %}
function pid2xid(){ wmctrl -lp | awk "\$3 == $(pgrep $1) {print \$1}" }
{% endhighlight %}

Выдрать pid по идентификатору окна.

{% highlight shell %}
# Change to repository root (starting in parent directory), using the first
# entry of a recursive globbing.
function RR() {
    setopt localoptions extendedglob
    local a
    # note: removed extraneous / ?!
    a=( (../)#.(git|hg|svn|bzr)(:h) )
    if (( $#a )); then
        cd $a[1]
    fi
}
{% endhighlight %}

Перейти в ближайший репозиторий проекта из командной строки.

{% highlight shell %}
function sp() {
    setopt extendedglob bareglobqual
    if [[ $# == 0 ]]; then
        output=$(du -smc *)
    else
        output=$(du -smc "$@")
    fi
    total=$(tail -1 <<< "${output}")
    distribution.pl -g -s=l --char=em <<< $(sed -e '$ d' <<< "${output}")
    _zwrap "Total: $(cut -f1 <<< ${total})"
}
{% endhighlight %}

distribution.pl программа, которая рисует красивые гистограммы, как у меня на
скриншоте к dotfiles. В данном случае я ей смотрю сколько места что занимает
в текущей директории. И красиво и удобно.

{% highlight shell %}
# Delete 0 byte file
d0() { find "$(retval $1)" -type f -size 0 -exec rm -rf {} \; }
{% endhighlight %}

Рекурсивное удаление пустых файлов.

{% highlight shell %}
function sls(){
    steamcmd '+apps_installed +quit' |\
        awk '/AppID/ {
                id = $2;
                name = substr($0, index($0, " : ") + 3);
                sub(" : .*", "", name);
                print id ": " name;
            }'
}
{% endhighlight %}

Запуск steam-игр из командной строки

{% highlight shell %}
# shameless stolen from http://ft.bewatermyfriend.org/comp/data/zsh/zfunct.html
# MISC: zurl() create small urls via tinyurl.com needs wget, grep and sed. yes, it's a hack ;)
function zurl() {
    [[ -z ${1} ]] && print "please give an url to shrink." && return 1
    local url=${1}
    local tiny="http://tinyurl.com/create.php?url="
    wget -O- -o /dev/null "${tiny}${url}"|grep -Eio "copy\('http://tinyurl.com/.*'"|grep -o "http://.*"|sed s/\'//
}
{% endhighlight %}

Сокращалка URL. Можно прикалывать ей, давая ссылки на всякую ерунду.

{% highlight shell %}
function img(){
    if [[ $# == 0 ]]; then
        imgur -h
    elif [[ $# == 1 ]]; then
        imgur $1 | tee -a ~/tmp/imgur_output_
    else
        imgur_command="$1"; shift
        imgur "$imgur_command" "$@" | tee -a ~/tmp/imgur_output_
        unset imgur_command
    fi
}
{% endhighlight %}

Враппер для загружалки на imgur, которая представляет собой чуть
подправленный imgur-screenshot.

{% highlight shell %}
function mp(){
    for i; do
        vid_fancy_print "${i}"
        mpv --input-unix-socket=/tmp/mpvsocket "${i}"
    done
}
{% endhighlight %}

Это то что выводит красивую информацию о файле, который сейчас загружен
в mpv.

{% highlight shell %}
function clojure(){
    drip -cp /usr/share/clojure/clojure.jar clojure.main
}
{% endhighlight %}

Простой враппер для clojure.

# Aliases

Мне лень комментировать это, просто пожалуйте посмотреть вот этот файл:

<a href="https://github.com/neg-serg/dotfiles/blob/master/.zsh/06-alias.zsh">https://github.com/neg-serg/dotfiles/blob/master/.zsh/06-alias.zsh</a>

Но если будут вопросы я могу ответить. Единственное о чем стоит сказать, как
мне кажется, так это что ты там используется тот же подход что и для
формирования path чтобы создать различные списки алиасов, например алиасы,
которые не требует sudo, программы, для которых не нужно использовать
globbing(например чтобы не экранировать звездочку в find). Также используется
cope в качестве подсветки для таких программ как du, df, mpc и других.
