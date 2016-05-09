---
layout: post
title: Простой способ собрать gentoo linux из arch
categories: personal
tags: 
  - gentoo
  - systemd
comments: true
mathjax: null
featured: true
published: true
---

Лично меня Gentoo Linux привлекает из-за его возможностей для безопасности,
а именно его ответвление Hardened Gentoo. Оно существует уже достаточно давно
и в целом стабильно. Когда-то Gentoo получил своё имя от самого быстрого
пингвина. В те далекие времена в ходу была x86 архитектура и процессоры
в массе своей довольно сильно отличались от того что понималось под i686(sse,
новые оптимизации и так далее). Конечно, целевые оптимизации имеют место
и в настоящее время, но былого прироста, как правило, уже не бывает.

ХОТЯ ЕСТЬ ИСКЛЮЧЕНИЯ ДОБАВИТЬ СТАТЬЮ ПРО УБУНДУ

Для начала подготовим место где это будем собирать. Например у меня для этого
сейчас используется ~/src/gentoo

Так что набираем:

mkdir -p ~/src/gentoo ; cd !$

Что ж, начнем с добычи дистрибутива. В данном случае нам пока что понадобится
только stage3. Это подготовленный образ системы. Когда-то ещё поддерживалась
сборка gentoo из stage1 и stage2, но сейчас этот способ оставили для совсем
уж задротов и в хэндбуке он не описан. Связано это скорее всего со сложностью
поддержки, особенностями при сборке компиляторов и тп. В общем идем по ссылке

w3m https://www.gentoo.org/downloads/mirrors/

Я рекомендую для России зеркала yandex, потому что они довольно быстрые(~100
Mbit)

Далее шагаем в /gentoo-distfiles/releases/amd64/autobuilds/current-stage3-amd64-hardened/.
Это Hardened multilib ветка для amd64 архитектуры. Любители странного могут
попробовать варианты с uClibc, но я точно помню, что под этой либой многое не
собирается. Может рассмотрим этот вариант как-нибудь в другой раз, пока что
я просто покажу как делать bootstrap.

Извлекаем архив

{% highlight sh %}
sudo tar xf stage3-amd64-hardened-20160414.tar.bz2
{% endhighlight %}

Сейчас схема немного отличается от той, что написана в хэндбуке. Обычно там
предлагается сделать chroot способом типа такого

{% highlight sh %}
mount /dev/sdX /mnt/gentoo
mkdir -p /mnt/gentoo/boot/efi
mount /dev/sdY /mnt/gentoo/boot/efi
mount --rbind /dev /mnt/gentoo/dev
mount --rbind /sys /mnt/gentoo/sys
mount -t proc none /mnt/gentoo/proc
cp /etc/resolv.conf /mnt/gentoo/etc/
chroot /mnt/gentoo
env-update
source /etc/profile
{% endhighlight %}

Как видите довольно многословно :)

Вместо этого попробуем сделать это с помощью systemd-nspawn:

{% highlight sh %}
sudo systemd-nspawn -M gentoo_hardened -D $(pwd)
{% endhighlight %}

Если всё было сделано правильно, то должно получиться что-то вроде

{% highlight sh %}
Spawning container gentoo_hardened on /home/user/src/gentoo.
Press ^] three times within 1s to kill container.
{% endhighlight %}

Плюс такого подхода в том, что нет никакого захламления основной системы
точками монтирования и проч, а также является более мощным чем chroot.  В нем
есть некоторые ограничения, которые в основном направлены на то чтобы нельзя
было сломать host систему: нельзя менять часы, сетевые интерфейсы, есть
ограничения на запись в /sys, /proc/sys и тп. Короче это упрощенная
реализация контейнеров, которая как раз хорошо подходит для этой цели.

Далее сделаем TERM=linux; export $TERM для корректной обработки
esc-последовательностей. И вообще обновим переменные окружения.
{% highlight sh %}
export TERM=linux 
env-update && source /etc/profile
{% endhighlight %}

{% highlight sh %}
cd /etc/portage
< make.conf cat
{% endhighlight %}

Делаем правки по вкусу.

Далее получаем portage.

{% highlight sh %}
emerge --sync
{% endhighlight %}

Далее выбираем нужный профиль с помощью 
eselect profile list:

{% highlight sh %}
gentoo_hardened portage # eselect profile list
Available profile symlink targets:
  [1]   default/linux/amd64/13.0
  [2]   default/linux/amd64/13.0/selinux
  [3]   default/linux/amd64/13.0/desktop
  [4]   default/linux/amd64/13.0/desktop/gnome
  [5]   default/linux/amd64/13.0/desktop/gnome/systemd
  [6]   default/linux/amd64/13.0/desktop/kde
  [7]   default/linux/amd64/13.0/desktop/kde/systemd
  [8]   default/linux/amd64/13.0/desktop/plasma
  [9]   default/linux/amd64/13.0/desktop/plasma/systemd
  [10]  default/linux/amd64/13.0/developer
  [11]  default/linux/amd64/13.0/no-multilib
  [12]  default/linux/amd64/13.0/systemd
  [13]  default/linux/amd64/13.0/x32
  [14]  hardened/linux/amd64 *
  [15]  hardened/linux/amd64/selinux
  [16]  hardened/linux/amd64/no-multilib
  [17]  hardened/linux/amd64/no-multilib/selinux
  [18]  hardened/linux/amd64/x32
  [19]  hardened/linux/musl/amd64
  [20]  hardened/linux/musl/amd64/x32
  [21]  default/linux/uclibc/amd64
  [22]  hardened/linux/uclibc/amd64
{% endhighlight %}

Поскольку я планирую дальше использовать systemd, то вариант
с hardened/linux/amd64/selinux не подходит, а nomultilib системы я не
использую. У альтернативных libc всё ещё довольно много проблем
с совместимостью. Из необходимых нужно добавить только use-флаг systemd
в make.conf.

В данный момент я рекомендую модифицировать только use-флаги,
а опции оптимизации пока что не трогать. Итак:

{% highlight sh %}
gentoo_hardened portage # cat make.conf
# These settings were set by the catalyst build script that automatically
# built this stage.
# Please consult /usr/share/portage/config/make.conf.example for a more
# detailed example.
CFLAGS="-O2 -pipe"
CXXFLAGS="${CFLAGS}"
# WARNING: Changing your CHOST is not something that should be done lightly.
# Please consult http://www.gentoo.org/doc/en/change-chost.xml before changing.
CHOST="x86_64-pc-linux-gnu"
# These are the USE flags that were used in addition to what is provided by the
# profile used for building.
MAKEOPTS="-j8"
USE="bindist mmx sse sse2 systemd cairo acpi alsa cairo cscope dbus ffmpeg fftw flac gimp iconv imagemagick imap imlib latex lua mp3 mp4 ncurses pcre pdf pulseaudio python ruby udev udisks unicode upnp upnp-av v4l xft xinetd zsh-completion icu policykit ssh threads fontforge png jpeg"
PORTDIR="/usr/portage"
DISTDIR="${PORTDIR}/distfiles"
PKGDIR="${PORTDIR}/packages"
{% endhighlight %}

Собираем систему с новыми флагами. 
{% highlight sh %}
emerge -avuND @world
{% endhighlight %}

Я добавил MAKEOPTS="-j8" для многопоточной сборки в make.conf. Даже на 4790k
это может занять довольно продолжительное время.

Перед сборкой из-за множества новых флагов скорее всего сработает autounmask.
Что ж, расмаскируем нужное, после чего нужно просто обновить конфиг
с помощью dispatch-conf. 

Теперь пересоберем всё с -march=native.

{% highlight sh %}
gentoo_hardened portage # grep CFLAGS make.conf
CFLAGS="-march=native -O2 -pipe"
CXXFLAGS="${CFLAGS}"
{% endhighlight %}

В данном случае binutils-config и gcc-config нам не нужен, потому что система
новая. Поскольку архитектуру мы тоже не трогали, то fix_libtool_files.sh тоже
не нужен.

Поправим locale-gen:

{% highlight sh %}

{% endhighlight %}

{% highlight sh %}
env-update
emerge --ask binutils gcc glibc
{% endhighlight %}

Пересборку binutils gcc glibc рекомендуется выполнять дважды.
Далее пересобираем libtool и мир:

{% highlight sh %}
emerge --ask libtool
emerge -e world
{% endhighlight %}

В теории пересобирать мир не обязательно, но в данном случае желательно,
потому что система пустая. Точно необходимо пересобирать python,
portage-utils и пакеты, которые зависят от perl, которых на самом деле много.
Словом лучше не рисковать и пересобрать всё.

Чтобы попасть в собранную систему достаточно воспользоваться:

{% highlight sh %}
sudo systemd-nspawn  -bD `pwd`
{% endhighlight %}
