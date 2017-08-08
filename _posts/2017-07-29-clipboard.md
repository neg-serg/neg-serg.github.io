---
layout: post
title: Clipboard management
categories: personal
tags: clipboard
comments: true
mathjax: null
featured: true
published: true
---

Думаю, что многие сталкивались с тем что скопировав что-то в буфер обмена,
они потом вспоминали что это не то что им нужно. Особенно это актуально если
используется не подсистема windows или mac os, а X11, где по-умолчанию во
многих DE могут синхронизироваться побочный(`SECONDARY`) буфер, который
используется для `Ctrl-C/Ctrl-V`. Для борьбы с этой напастью существуют
менеджеры буфера обмена, такие как `clipit/parcellite`, `gpaste`, `klipper`,
`clipster` и другие. Я рассмотрю два самых "современных" и удобных для себя на
данный момент: `Gpaste` и `clipster`. Они интересны тем что на мой взгляд
хорошо соответствуют концепции UNIX-way, где их вывод можно перенаправлять
куда-то ещё: `dmenu/rofi`, различные фильтры и тд и тп. Ещё существует
достаточно похожий на `clipster` `greenclip`. Он интересен тем что сделан на
`haskell`, что для многих усложнило бы его разработк, но по моим личным
ощущениями работает он не быстрее чем `greenclip`, несмотря на то, что вроде
как код, который генерирует `ghc` может быть быстрее.

Тем, кто хочет знать, как это всё настроено, добро пожаловать под кат.

<!--excerpt-->

# Gpaste

Gpaste это один из менеджеров буфера обмена, который дружит с проектом `GNOME`.
Несмотря на это у него есть cli-interface и нет никакой проблемы использовать
его через пайпы, например получать содержимое истории копирования или
добавлять новую. Настраивается он с помощью команды `gpaste-client settings`.
У меня все его опции в "General Behaviour" включены, из них особо важными
я считаю синхронизацию между всеми типами буфера обмена. Как известно в X11
их существуют три вида: `PRIMARY`, который обычно используется в эмуляторах
терминала и соответствует хоткеям `Ctrl-Insert/Shift-Insert`, `CLIPBOARD`
используется для операций в стиле ms windows вроде Control+C/Control-V
в Firefox, `SECONDARY` обычно, насколько я знаю, автоматически заполняется при
выделении какого-либо текста. 

Одной из наиболее удобных фич эти менеджеров буфера обмена я как раз считаю
автоматическую синхронизацию между различными версиями буфера обмена. 

Второй важной возможностью является возможность
поиска и взаимодействия по истории. Вот как она выглядит:

![rofi](http://i.imgur.com/V2PVrOI.png)

Большим достоинством использования rofi в данном случае является то, что он
обновляется отдельно от собственно менеджеров буфера обмена, да и вообще
любых приложений, которые служат для него источником входных строк, обладает
хорошей запуска и поиска, возможностью кастомизации поиска, таких как
например сортировка по расстоянию Левенштейна, fuzzy-finding и другие.

По-умолчанию у gpaste нет binding'ов к rofi, но это не проблема. Можно очень
просто реализовать взаимодействие с помощью такого скрипта:

{% highlight shell %}
#!/bin/zsh
export DBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/$(id -u)/bus

function gpaste_run() {
    if [[ -x $(which gpaste-client) ]]; then
        echo $(which gpaste-client) --oneline
    elif [[ -x $(which gpaste) ]]; then
        echo $(which gpaste) -client --oneline
    else
        echo There is no gpaste in \$PATH>/dev/stderr
    fi
}

function clip_main(){
    local rofi_pid=" -pid /run/user/$(id -u)/rofi_clip.pid"
    local rofi_font="PragmataPro for Powerline bold 12"
    declare -a rofi_opts_
    local x11_width="$(xrandr -q |awk '/Screen/{print $8}')"
    rofi_opts_=(-font ${rofi_font}  -yoffset -22 -location 6 -width $[x11_width-70] -lines 10 "${rofi_pid}" -i -levenshtein-sort -matching glob -dmenu -p "[>>]")
    local menu=$($(gpaste_run)|rofi ${rofi_opts_[@]})
    local selection=$(echo "${menu}" | cut -d ':' -f1)
    $(gpaste_run) select ${selection}
}

clip_main
{% endhighlight %}

# Clipster

К сожалению не всех может устроить то что Gpaste тянет за собой довольно
солидный пакет зависимостей, которые не нужны тем, кто не пользуется GNOME,
также он довольно тяжеловесен, хотя и работает быстро, избыточен по своей
функциональности, да и вообще какой-то не UNIX-way'ный. В общем мне он
кажется более "правильным" и близким к идеалу. К сожалению он написан на
python, так что работает медленнее, чем gpaste, возможно если его переписать
на c, то ситуация улучшится, благо объем кода у него меньше 1000 строк
и проблем с портированием быть не должно.

Скрипт для взаимодействия с ним примерно такой же как и в случае с gpaste:

{% highlight shell %}
function clipster_main(){
    local rofi_pid=" -pid /run/user/$(id -u)/rofi_clip.pid"
    local rofi_font="PragmataPro for Powerline bold 12"
    declare -a rofi_opts_
    local x11_width="$(xrandr -q |awk '/Screen/{print $8}')"
    rofi_opts_=(-font ${rofi_font}  -yoffset -22 -location 6 -width $[x11_width-70] -lines 10 "${rofi_pid}" -i -levenshtein-sort -matching glob -dmenu -p "[>>]" -sep '\x00')
    local menu=$(clipster -o -n 250 -0|rofi ${rofi_opts_[@]})
    local selection=$(echo "${menu}" | cut -d ':' -f1)
    [[ ${selection} != "" ]] && clipster <<< ${selection}
}
{% endhighlight %}

