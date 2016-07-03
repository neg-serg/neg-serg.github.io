---
layout: post
title: UNIX (s)hell, pt5
categories: personal
tags: 
  - zsh
comments: true
mathjax: null
featured: true
published: true
date: 2016-07-04 00:46:00 +03:00
---

В этой части я покажу как у меня реализован впаппер для vim, расскажу про
некоторые плагины. О плагинах уже было в общем-то рассказано в первой части.
Так что похоже эта часть последняя, которая говорит преимущественно
о конфиге. Ещё я хочу рассказать об extended globbing. Многие знают что он
есть, но не все знают насколько это может быть полезно на практике.


<!--excerpt-->

# Vim wrapper

Все знают про vim. Также все знают, что у него есть GUI морда gvim, которая
во многом неплохо работает. К сожалению нет возможности комбинирвоать gvim
и tmux. Так что вместо этого я сдвинулся в сторону обычного vim. Но вот
незадача: нет удобного способа отделить vim в отдельный tmux. Для этого была
придумана вандервафля, который держит tmux на отдельном сокете. Также я хотел
чтобы была возможность переключаться на окно из командной строки, для этого
используется интеграция с notion с помощью notionflux(это его IPC через
сокет). В результате моим разумом было порождено следующее чудовище:

{% highlight sh %}
readonly to_normal="<C-\><C-N>:call<SPACE>foreground()<CR>"

function wim_run(){
    local proc="process_list"
    [[ $1 == "__wim_embed" ]] && {proc="eprocess_list"; shift}
    local wid=$(xdotool search --classname wim)
    if [[ -z "${wid}" ]]; then
        st -f "${wim_font}:pixelsize=${wim_font_size}" -c 'wim' -e bash -c "tmux -S ${wim_sock_path} new -s vim -n vim \"vim --servername ${vim_server_name}\" && \
            tmux -S ${wim_sock_path} switch-client -t vim" 2>/dev/null &!
        eval ${proc} ${wim_timer} "$@"
    else  
        eval ${proc} ${wim_timer} "$@"
    fi
}

function wim_goto() {
    if [[ $(pidof notion) && -x $(which notionflux) ]]; then
        notionflux -e "app.byclass('', 'wim')" > /dev/null
    else
        if [[ -x $(which wmctrl) ]]; then
            wmctrl -i -a $(wmctrl -l -x|awk '/wim.wim/{print $1}')
        elif [[ -x $(which xdotool) ]]; then
            xdotool windowfocus $(xdotool search --class wim)
        fi
    fi
}

function vim_file_open() (
    local file_name="$(resolve_file ${line})"
    file_name=$(bash -c "printf %q '${file_name}'")
    { vim --servername ${vim_server_name} --remote-send "${to_normal}:silent edit ${file_name}<CR>" 2>/dev/null \
        || { while [[ $(vim --servername ${vim_server_name} --remote-expr "g:vim_is_started" 2>/dev/null) != "on" ]]; do
            sleep ${wim_timer}
        done \
        && vim --servername ${vim_server_name} --remote-send "${to_normal}:silent edit ${file_name}<CR>" 2>/dev/null } } && {
        local file_size=$(stat -c%s "${file_name}" 2>/dev/null| numfmt --to=iec-i --suffix=B|sed "s/\([KMGT]iB\|B\)/$fg[green]&/")
        local file_length="$(wc -l ${file_name} 2>/dev/null|grep -owE '[0-9]* '|tr -d ' ')"
        local sz_msg=$(_zwrap "sz$(_zfg 237)~$fg[white]${file_size}")
        local len_msg=$(_zwrap "len$(_zfg 237)=$fg[white]${file_length}")
        local new_file_msg=$(_zwrap new_file)
        local dir_msg=$(_zwrap directory)
        local pref=$(_zwrap ">>")
        if [[ ! -e "${file_name}" ]]; then
            <<< "${pref} $(_zfwrap ${file_name}) $(_zdelim) ${new_file_msg}"
        elif [[ -f "${file_name}" ]] && [[ ! -d "${file_name}" ]]; then
            <<< "${pref} $(_zfwrap ${file_name}) $(_zdelim) ${sz_msg} $(_zdelim) ${len_msg}${syn_msg}"
        else
            if [[ -d "${file_name}" ]]; then
                if [[ $(readlink -f ${file_name}) == $(readlink -f $(pwd)) ]]; then
                    <<< "${pref} $(_zfwrap "current dir") $(_zdelim) ${dir_msg}"
                else
                    <<< "${pref} ${fancy_name} $(_zdelim) ${dir_msg}"
                fi
            fi
        fi
    }
    unset file_name
)

function process_list() {
    wim_goto
    sleep "$1"; shift
    while getopts ":b:a:c:" opt; do
        case ${opt} in
            a|c) after="${OPTARG}" ;;
            b) before="${before}${OPTARG}" ;;
            --) shift ; break ;;
        esac
    done
    shift $((OPTIND-1))
    [[ ${after#:} != ${after} && ${after%<CR>} == ${after} ]] && after="${after}<CR>"
    [[ ${before#:} != ${before} && ${before%<CR>} == ${before} ]] && before="${before}<CR>"
    local cmd="${to_normal}${before}${after}"
    if [[ ${cmd} == ${to_normal} ]]; then
        for line; do vim_file_open; done
    else
        vim --servername ${vim_server_name} --remote-send "${cmd}"
        for line; do vim_file_open; done
    fi
    unset before; unset after
}

function eprocess_list() {
    wim_goto
    sleep "$1"; shift   
    for line; do vim --servername ${vim_server_name} --remote-wait "$@"; done
}

function v { 
    while read -r arg; do
        wim_run ${arg[@]}
    done <<< "$(printf '%q\n' "$@")"
}

function wim_embed { wim_run "__wim_embed" "$@" }

function wdiff {
    local prev_ 
    local arg2_
    # or it's maybe better to use :windo diffthis
    if [[ $# == 2 ]]; then
        prev_="$1" 
        wim_run "" && v -b":tabnew" && \
        { wim_run $1; shift } && v -b":diffthis" && \
        { v -b":vs" && {
            if [[ -d $1 ]]; then  
                arg2_="$1/$(basename ${prev_})"
            else
                arg2_="$1"
            fi
            wim_run ${arg2_};
            shift
        } && v -b":diffthis" }
    fi
}
{% endhighlight %}

Чрезмерное углубление в логику этого скрипта опасно для моска. Тем не менее
грех его не привести, потому что ~~моя основная цель заховать ваш мозг~~
пользуюсь этим практически каждый день. Представляет собой набор эвристик,
которые просто выполняют свою работу, с корректным экранированием файлов,
поддержкой локалей, также можно выполнять diff с помощью wdiff и передавать
другие команды, не запуская новый инстанс вима. В практически всех случаях
корректно работает, корректно создает новый инстанс tmux если это нужно,
корректно выбирает искомый, если это не нужно, в общем работает хорошо.
Иногда случаются накладки, видимо это какие-то race conditions, но это
случается так редко, что я не знаю в чем дело, к тому же они не фатальные.
Создан был когда я ещё не знал про zsh-functional достаточно хорошо, так что
возможно его код может быть упрощен. Вы можете доработать его и взять себе,
если оставите ссылку на автора(я хочу чтобы все знали про моего
ребенка-гомункула). Отклик на события vim формируется с помощью "адаптивного"
таймера, что вроде тоже уже работает довольно стабильно, по крайней мере
у меня. Так же есть враппер для того чтобы использовать эту штуку в качестве
$VISUAL в mutt, yaourt, pacman и тд и тп:

{% highlight sh %}
#!/bin/zsh
source ~/.zshrc
wim_embed "$@"
{% endhighlight %}

К счастью его код тривиален и состоит всего из одной функции, которая есть
в коде, приведенном выше. На теоретически возможные потери производительности
из-за загрузки всего zshrc целиком я забил, потому что вроде и так быстро.

# Vim cat

cat с подсветкой синтаксиса, выглядит вот так:

![vimcat](https://i.imgur.com/HTdqFDU.png)

Можно найти, например, у меня:
<a href="https://github.com/neg-serg/dotfiles/blob/master/bin/_v">https://github.com/neg-serg/dotfiles/blob/master/bin/_v</a>

# Extended globbing

Теперь я хочу показать вам такую особенность zsh как extended globbing. Это
улучшенный globbging(которые звездочка), который позволяет делать выборку по
множеству разных критериев, простой рекурсивный обход и так далее.

Сразу скажу что почти все эти параметры можно комбинировать. Акцент сделан на
возможностях, которые удобно применять в интерактивном режиме. Такой
улучшенный globbing, равно как и обычный, может использоваться в практически
всех выражениях.

Вы всегда можете сходить <a
href="http://www.zzapper.co.uk/zshtips.html">http://www.zzapper.co.uk/zshtips.html</a>
где примеров такого рода побольше. Собственно я просто выбрал из этого списка
такие, которые кажутся мне наиболее полезными и которые сам постоянно(или
часто) использую на практике.

{% highlight sh %}
ls -d *(/)           # list just directories *C*
{% endhighlight %}

Как вы поняли (/) задает поиск только по директориям. Равно так же работает
и (.) -- это поиск только по обычным файлам, (@) -- только по симлинкам. Это
очень удобно. Кто хочет большего может обратиться например к хэлперам из
моего автокомплита, это самый простой и быстрый способ. Ну или документацию
почитать.

Также для этих аргументов внутри скобок мы имеем массив, по которому можно
итерироваться и делать перечисления, например:

{% highlight sh %}
vi *(.om[1])      # vi newest file
{% endhighlight %}

Здесь мы редактируем самый новый файл в текущей директории. Можно
использовать отрицательные индексы для создания обратного эффекта. Также Om
взаимообратно om, за исключением того что точку для него писать не
обязательно.

Полезным оператором также является крышечка(^), она меняет смысл выражения на
противоположный, например нам нужно исключить из множества файлы на mp4, вот
как это сделать:

{% highlight sh %}
ls ^*.mp4
{% endhighlight %}

Это примерно то же самое что и:
{% highlight sh %}
ls *.^mp4
{% endhighlight %}

Крышечка также работает и внутри скобок и может находиться в любом месте. То
есть можно сделать половину globbing'а нормальным, а вторую половину, которая
после ^ исключающим.

Частоиспользуемой мной фичей является !$, возвращает последний аргумент
предыдущей команды, удобно, например:

{% highlight sh %}
vi !$
{% endhighlight %}

Отредактировать файл, который был последним аргументом у предыдущей
команды. Представьте что последняя команда это mv и юзкейс станет понятен.

Также возможен рекурсивный глоббинг, например 

{% highlight sh %}
ls **/*.zsh
{% endhighlight %}

Выведет все *.zsh файлы в текущей директории и её потомках.

Также частоиспользуемой возможностью является оператор фигурных скобок.
Например представим что мы хотим создать временный файл с суффиксом .bak, вот
как это сделать:

{% highlight sh %}
mv notionerr913{,.bck}
# Это выражение превращается в
mv notionerr913 notionerr913.bck
# Скобки могут быть в любом месте, например в середине
# Так можно отменить предыдущую команду:
mv notionerr913{.bck,}
# Превращется в
mv notionerr913.bck notionerr913
{% endhighlight %}

Есть возможность игнорирования регистра при globbing'е:
{% highlight sh %}
[~/doc/new] >> ls (#i)oca*
OCaml.Practice.pdf
{% endhighlight %}

Ещё одной частоиспользуемой возможностью, которой, кстати, нет в списке
являются знаки процента и решетки(%,#,%%,##). Вообще-то они являются частью
parameter expansion, но мне как-то всё равно. Итак, если i это имя файла, то:

{% highlight sh %}
${i%.*}  # Получить имя файла без расширения
${i##*.} # Получить расширение файла

# Создать директорию которая как *.zip, только без суффикса .zip
# Потом перенести файл в полученный результат.
for i in *.zip ;do mkdir ${i%%.zip}; mv $i ${i%%.zip}; done
{% endhighlight %}
