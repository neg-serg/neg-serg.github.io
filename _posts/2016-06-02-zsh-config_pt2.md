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
рассмотрены некоторые полезные глобальные переменные окружения(env). Это не
всё что есть, лишь то что я считаю важным, за полным конфигом в актуальном
состоянии всегда можно слазить на мой гитхаб.

<!--excerpt-->

* TOC
{:toc}

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

if which drip > /dev/null 2>&1; then
  export DRIP_SHUTDOWN=30
  export JAVACMD=$(which drip)
fi

export FZF_DEFAULT_COMMAND='ag -l -g ""'
export FZF_CTRL_T_COMMAND="$FZF_DEFAULT_COMMAND"
export FZF_DEFAULT_OPTS="${FZF_DEFAULT_OPTS} --color=16"

export PULSE_LATENCY_MSEC=60
export NVIM_TUI_ENABLE_TRUE_COLOR=0

export STEAM_RUNTIME=1

export QEMU_AUDIO_DRV=pa
export QEMU_PA_SOURCE=input
export QEMU_PA_SINK=alsa_output.usb-E-MU_Systems__Inc._E-MU_0204___USB_E-MU-A8-3F19-07DC0212-089A1-8740AT2A-00.analog-stereo

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
