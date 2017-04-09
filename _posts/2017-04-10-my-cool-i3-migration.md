---
layout: post
title: A cool story about my migration from notion to i3
categories: personal
tags: 
  - i3
  - notion
comments: true
mathjax: null
featured: true
published: true
---

This a post about my migration from <a href="https://github.com/neg-serg/notion">my own fork of notion</a> to
a i3-gaps and porting ion3-specific features like multiwindow scratchpads,
etc. In this article you will probably see a lot of semi-stupid pythonish
hacks because of I am new to python.

<!--excerpt-->

# Reasons for migration:

* i3 is more modern and still under pretty active developement.

* i3 codebase is more easy to change.

* Almost no compatibility problems, it is pretty popular.

* Better support of EWMH.

* Because of large community it has nice tools support(like <a href="https://github.com/DaveDavenport/rofi">rofi</a>) out of the
  box. Almost no one using ion3/notion wm for now.

* More dynamic tiling system, no ugly placeholders.

* No problems with aRGB stuff.

* No problems with games and another fullscreen stuff out of the box.

* No problems with jetbrains stuff out of the box.

* It is more easy to interact with python bindings, better and more modern API.
  A lot of scripts made by others.

* It is more easy to use beautiful and modern panels like <a
  href="https://github.com/jaagr/polybar">polybar</a> or <a href="https://github.com/geommer/yabar">yabar</a>.

* You can use it with xfce and probably another DEs if you want.

* Nicer theming / visual appearance.

# Features which i3 by default have not:

But anyway there are no some features which I have in notion as:

* run-or-raise(app.lua) features by default

* Multiwindow scratchpads, presented on UX level as "tabs for windows" and
  imagined as solid object. By the way no wm expect notion/ion3 supported
  that... by that time

* "Tags" support. It works like cyclic run or rise, more sophisticated
  version of app.lua/run-or-raise invented by me.

* No alt-tab to previous window by default

* And there are a lot of things which you need to google as move workspace to
  current, goto to application, etc.

So let's start:

## Fetch it

First of all I recommend you the latest i3-gaps fork, because of I think that it looks
nicer:

<a href="https://github.com/Airblader/i3">github.com/Airblader/i3</a>

[i3-gaps-screen](https://camo.githubusercontent.com/7da062cb011d81a3551cd34ee2d33886ee6b2e06/687474703a2f2f692e696d6775722e636f6d2f7938735a45366f2e6a7067)

Just get the README.md to read and git clone it to install or use AUR repo if you are
Arch Linux user as me.

To make changes in my config you are also need ppi3 preprocessor. You can get
it <a href="https://github.com/KeyboardFire/ppi3">here</a>

Also look for my config at <a href="https://github.com/neg-serg/dotfiles/tree/master/.config/i3">neg-serg/dotfiles/tree/master/.config/i3</a>

It is pretty easy to understand, except of python scripts which we will
discuss later.

## Scripts stuff

First of all you can find this scripts and other stuff on my <a href="dotfiles repo">https://github.com/neg-serg/dotfiles</a>.

To port nice ion3/notion UX to i3 I had to use some tricky python stuff to
interact with i3 `python-i3-git`. I've also reworked it as FIFO based client-server for the better performance and input lag.

My scripts have following declarations:


{% highlight python %}
from singleton_mixin import *
from script_i3_general import *
{% endhighlight %}

Here the "thread-safe" singleton stuff:

{% highlight python %}
from threading import Lock

# Based on tornado.ioloop.IOLoop.instance() approach.
# See https://github.com/facebook/tornado
class SingletonMixin(object):
    __singleton_lock = Lock()
    __singleton_instance = None

    @classmethod
    def instance(class_):
        if not class_.__singleton_instance:
            with class_.__singleton_lock:
                if not class_.__singleton_instance:
                    class_.__singleton_instance = class_()
    return class_.__singleton_instance
{% endhighlight %}

And here some generic stuff, which can be used from the various scripts(not
only mine, I hope):

{% highlight python %}
import i3ipc
import os

from sys import exit
from subprocess import check_output
from singleton_mixin import *
from threading import Thread

i3 = i3ipc.Connection()

def find_visible_windows(windows_on_workspace):
    visible_windows = []
    for w in windows_on_workspace:
        try:
            xprop = check_output(['xprop', '-id', str(w.window)]).decode()
        except FileNotFoundError:
            raise SystemExit("The `xprop` utility is not found!"
                            " Please install it and retry.")
        if '_NET_WM_STATE_HIDDEN' not in xprop:
            visible_windows.append(w)

    return visible_windows

def get_windows_on_ws():
    return filter(
        lambda x: x.window,
        i3.get_tree()
        .find_focused()
        .workspace()
        .descendents()
    )

from queue import Queue

class daemon_manager(SingletonMixin):
    def __init__(self):
        self.daemons={}

    def add_daemon(self, name):
        daemon_=daemon_i3.instance()
        if daemon_ not in self.daemons.keys():
            self.daemons[name]=daemon_
            self.daemons[name].bind_fifo(name)

class daemon_i3(SingletonMixin):
    def __init__(self):
        self.q = Queue()
        self.fifo_=""

    def bind_fifo(self, name):
        self.fifo_=os.path.realpath(os.path.expandvars('$HOME/tmp/'+name+'.fifo'))
        if os.path.exists(self.fifo_):
            os.remove(self.fifo_)

        try:
            os.mkfifo(self.fifo_)
        except OSError as oe:
            if oe.errno != errno.EEXIST:
                raise

    def fifo_listner(self, singleton):
        with open(self.fifo_) as fifo:
            while True:
                data = fifo.read()
                if len(data) == 0:
                    break
                eval_str=data.split('\n', 1)[0]
                args=list(filter(lambda x: x != '', eval_str.split(' ')))
                singleton.switch(args)

    def worker(self):
        while True:
            if self.q.empty():
                exit()
            i = self.q.get()
            self.q.task_done()

    def mainloop(self, singleton):
        while True:
            self.q.put(self.fifo_listner(singleton))
            Thread(target=self.worker).start()
{% endhighlight %}

It is far more easy to understand with following example:


### Alt-tab stuff

i3 have no stuff like jump to the previous window is it on this workspace or
not. So, let's fix it.

I have stoled original idea from <a href="https://github.com/metakirby5/scripts/blob/master/i3-focus-last">https://github.com/metakirby5/scripts/blob/master/i3-focus-last</a>.
The difference with my stuff mostly about FIFO-based client-server. Also I've
cleaned it up a bit for my taste and "fixed" too fast alt-tab with a little
like 50ms sleep for the alt_tab function, because of my keybindings latency
too low and also probably desktop is too fast. Anyway without this stupid
waiting stuff you are most likely get dead-lock of i3(seems some kind of
race-condition depending on workspace switching).

The interface for it looks pretty simple:

{% highlight python %}
#!/usr/bin/env python3

""" i3 focus last client
Usage:
  flast.py switch
  flast.py (-h | --help)
  flast.py --version
Options:
  -h --help     Show this screen.
  --version     Show version.
"""

from docopt import docopt
import os.path

name='flastd-i3'
fifo_=os.path.realpath(os.path.expandvars('$HOME/tmp/'+name+'.fifo'))

if __name__ == '__main__':
    argv = docopt(__doc__, version='i3 alt-tabber')
    possible_commands=["switch"]

    for i in argv:
        if argv[i] and i in set(possible_commands):
            with open(fifo_,"w") as fp:
fp.write(i+"\n")
{% endhighlight %}

and use only one function. Let's look current implementation:

{% highlight python %}
#!/usr/bin/env python3

"""
Focus last focused window.
Usage:
    flastd.py daemon
Options:
    -h, --help  show this help message and exit
"""

import i3ipc
import os

from threading import Thread
from docopt import docopt
from queue import Queue
import time

from singleton_mixin import *
from script_i3_general import *

max_win_history_ = 10

class FocusWatcher(SingletonMixin):
    def __init__(self):
        self.window_list = i3.get_tree().leaves()
        self.prev_time = 0
        self.curr_time = 0

    def switch(self, args):
        switch_ = {
            "switch": self.alt_tab,
        }
        if len(args) == 2:
            switch_[args[0]](args[1])
        elif len(args) == 1:
            switch_[args[0]]()

    def alt_tab(self, timer=0.05):
        self.curr_time = time.time()
        windows = set(w.id for w in i3.get_tree().leaves())
        for wid in self.window_list[1:]:
            if wid not in windows:
                self.window_list.remove(wid)
            else:
                if self.curr_time - self.prev_time > timer:
                    i3.command('[con_id=%s] focus' % wid)
                    self.prev_time = self.curr_time
                    self.curr_time = time.time()
                break

def on_window_focus(self, event):
    wid = event.container.id
    fw=FocusWatcher.instance()

    if wid in fw.window_list:
        fw.window_list.remove(wid)

    fw.window_list.insert(0, wid)
    if len(fw.window_list) > max_win_history_:
        del fw.window_list[max_win_history_:]

def go_back_if_nothing(self, event):
    con=event.container
    fw=FocusWatcher.instance()
    if len(find_visible_windows(get_windows_on_ws())) == 0:
        fw.alt_tab(0)

if __name__ == '__main__':
    argv = docopt(__doc__, version='i3 nice alt-tab 1.0')
    i3 = i3ipc.Connection()

    if argv["daemon"]:
        name='flastd-i3'

        fw=FocusWatcher.instance()

        mng=daemon_manager.instance()
        mng.add_daemon(name)

        def cleanup_all():
            daemon_=mng.daemons[name]
            if os.path.exists(daemon_.fifo_):
                os.remove(daemon_.fifo_)

        import atexit
        atexit.register(cleanup_all)

        i3.on('window::focus', on_window_focus)
        i3.on('window::close', go_back_if_nothing)

        mainloop=Thread(target=mng.daemons[name].mainloop, args=(fw,)).start()

i3.main()
{% endhighlight %}

It is almost self-desctiptive. To interact with generic script stuff you need
to create instance of FocusWatcher, which tracks last selected windows:

{% highlight python %}
fw=FocusWatcher.instance()
{% endhighlight %}

Then create an instance of daemon manager, bind fifo and bind fifo to it:

{% highlight python %}
    mng=daemon_manager.instance()
    mng.add_daemon(name)

    def cleanup_all():
        daemon_=mng.daemons[name]
        if os.path.exists(daemon_.fifo_):
            os.remove(daemon_.fifo_)

    import atexit
    atexit.register(cleanup_all)
{% endhighlight %}

Also you need an `switch(self, args)` implementation in your main class to
bind strings from the fifo client to the daemon:

{% highlight python %}
    def switch(self, args):
        switch_ = {
            "switch": self.alt_tab,
        }
        if len(args) == 2:
            switch_[args[0]](args[1])
        elif len(args) == 1:
            switch_[args[0]]()
{% endhighlight %}

This stuff looks appears on i3 events:

{% highlight python %}
    i3.on('window::focus', on_window_focus)
    i3.on('window::close', go_back_if_nothing)
{% endhighlight %}

### Notion-like named scratchpads stuff

This module is pretty sophisticated. The main idea is to separate all windows
by the equivalence classes determined by X11 Windows attributes and then
work with it as with notion scratchpad with show/hide, next window stuff and
another. For now matching by window_class and window_instance is supported.

Also you can to work with it from fullscreen with auto switching it on/off
when appropriate. By default you cannot to see scratchpads in some cases,
because of the policy of i3 with this things are far more strict than in
ion3/notion wm.

{% highlight python %}
#!/usr/bin/env python3

""" i3 Named Scratchpads
Usage:
  ns.py daemon
Options:
  -h --help     Show this screen.
  --version     Show version.
Created by :: Neg
email :: <serg.zorg@gmail.com>
github :: https://github.com/neg-serg?tab=repositories
year :: 2017
"""

import i3ipc

from docopt import docopt
from ns_config import *

from threading import Thread
from singleton_mixin import *
from script_i3_general import *

import uuid
import re
import errno
import os

settings_=ns_settings().settings
marked={i:[] for i in settings_}

class named_scratchpad(SingletonMixin):
    def __init__(self):
        self.group_list=[]
        self.fullscreen_list=[]
        [self.group_list.append(gr) for gr in settings_]

    def parse_geom(self, group):
        geom=re.split(r'[x+]', settings_[group]["geom"])
        return "move absolute position {2} {3}, resize set {0} {1}".format(*geom)

    def make_mark(self, group):
        output=(group) + str(str(uuid.uuid4().fields[-1]))
        return 'mark {}'.format(output)

    def focus(self, gr):
        for j,i in zip(
                range(len(marked[gr])),
                marked[gr]
            ):
            marked[gr][j].command('move container to workspace current')

    def toggle(self, gr):
        if marked[gr] == [] and "prog" in settings_[gr]:
            i3.command("exec {}".format(settings_[gr]["prog"]))

        # We need to hide scratchpad it is visible, regardless it focused or not
        focused = i3.get_tree().find_focused()

        if self.visible(gr) > 0:
            self.unfocus(gr); return

        for i in marked[gr]:
            if focused.id == i.id:
                self.unfocus(gr); return

        if focused.fullscreen_mode:
            focused.command('fullscreen toggle')
            self.fullscreen_list.append(focused)

        self.focus(gr)

    def unfocus(self, gr):
        def restore_fullscreens():
            [i.command('fullscreen toggle') for i in self.fullscreen_list]
            self.fullscreen_list=[]

        for j,i in zip(
                range(len(marked[gr])),
                marked[gr]
            ):
            marked[gr][j].command('move scratchpad')
        restore_fullscreens()

    def visible(self, gr):
        visible_windows = find_visible_windows(get_windows_on_ws())
        vmarked = 0
        for w in visible_windows:
            for i in marked[gr]:
                if w.id == i.id:
                    vmarked+=1
        return vmarked

    def apply_to_current_group(self, func):
        def get_current_group(self,focused):
            for group in settings_:
                for i in marked[group]:
                    if focused.id == i.id:
                        return group

        curr_group=get_current_group(self,i3.get_tree().find_focused())
        if curr_group  != None:
            func(curr_group)

    def next_win(self):
        focused_=i3.get_tree().find_focused()

        def next_win_(group):
            self.focus(group)
            for number,win in zip(
                    range(len(marked[group])),
                    marked[group]
                ):
                if focused_.id != win.id:
                    marked[group][number].command('move container to workspace current')
                    marked[group].insert(len(marked[group]), marked[group].pop(number))
                    win.command('move scratchpad')
            self.focus(group)

        self.apply_to_current_group(next_win_)

    def hide_current(self):
        self.apply_to_current_group(self.unfocus)

    def switch(self, args):
        switch_ = {
            "show": self.focus,
            "hide": self.unfocus,
            "next": self.next_win,
            "toggle": self.toggle,
            "hide_current": self.hide_current,
        }

        if len(args) == 2:
            switch_[args[0]](args[1])
        elif len(args) == 1:
            switch_[args[0]]()

def mark_group(self, event):
    def scratch_move(by):
        con.command(ns.make_mark(group)+', move scratchpad,'+ns.parse_geom(group))
        marked[group].append(con)

    def check_class():
        return bool(con.window_class in settings_[group]["classes"])

    def check_instance():
        return bool(con.window_instance in settings_[group]["instances"])

    for group in settings_:
        ns=named_scratchpad.instance()
        con=event.container
        try:
            if check_class():    scratch_move(by="class")
            if check_instance(): scratch_move(by="instance")
        except KeyError:
            pass

def mark_all(hide=True):
    def scratch_move(by):
        if hide:
            hide_cmd=', [con_id=__focused__] scratchpad show'
        else:
            hide_cmd=''
        con.command(ns.make_mark(group)+', move scratchpad,'+ns.parse_geom(group)+hide_cmd)
        marked[group].append(con)

    def check_class():
        return bool(con.window_class in settings_[group]["classes"])

    def check_instance():
        return bool(con.window_instance in settings_[group]["instances"])

    window_list = i3.get_tree().leaves()
    for group in settings_:
        ns=named_scratchpad.instance()
        for con in window_list:
            try:
                if check_class():    scratch_move(by="class")
                if check_instance(): scratch_move(by="instance")
            except KeyError:
                pass

def cleanup_mark(self, event):
    for tag in settings_:
        for j,win in zip(range(len(marked[tag])),marked[tag]):
            if win.id == event.container.id:
                del marked[tag][j]

if __name__ == '__main__':
    argv = docopt(__doc__, version='i3 Named Scratchpads 0.3')
    if argv["daemon"]:
        i3 = i3ipc.Connection()
        name='ns_scratchd'

        mng=daemon_manager.instance()
        mng.add_daemon(name)

        def cleanup_all():
            daemon_=mng.daemons[name]
            if os.path.exists(daemon_.fifo_):
                os.remove(daemon_.fifo_)

        import atexit
        atexit.register(cleanup_all)

        ns=named_scratchpad.instance()
        mark_all(hide=True)

        i3.on('window::new', mark_group)
        i3.on('window::close', cleanup_mark)
        mainloop=Thread(target=mng.daemons[name].mainloop, args=(ns,)).start()
i3.main()
{% endhighlight %}

### Tag-based / circle run-or-raise stuff

Run-or-raise is a well known technique to increase your productivity. Instead
of finding window by hand or with rofi you can jump to it using window_class,
window_title or other matching stuff or run it if it does not exists
very easily with single shortcut.

To understand it better look at pretty popular EWMH-based implementation of
this idea <a href="https://github.com/mkropat/jumpapp">here</a>.

Anyway my implementation is better than others because of it works like
circle above the some matching criteria. For example here is my settings for
this script:

{% highlight python %}
#!/usr/bin/env python3
class cycle_settings(object):
    settings={}

    def __init__(self):
        self.settings = {
            'web' : {
                'classes' : {
                    'Firefox',
                    'Tor Browser',
                    'Chromium',
                },
                'prog':"firefox",
                'priority':'Firefox',
            },
            'vid':{
                'classes': {'mpv'},
            },
            'steam':{
                'classes': {
                    'wine',
                    'dota2',
                    'darkest.bin.x86_64',
                    'Steam',
                    'steam',
                },
                'prog':"steam",
            },
            'doc':{
                'classes': {
                    'Zathura',
                },
            },
            'vm':{
                'classes': {
                    'VirtualBox',
                    'vmware'
                },
            },
            'term':{
                'classes': { 'MainTerminal' },
                'instances': { 'MainTerminal' },
                'prog':"~/bin/term",
            },
            'wim':{
                'classes': { '' },
                'instances': { 'nwim', 'wim' },
                'prog':"~/bin/nwim",
            },
            'jetbrains-idea':{
                'classes': {
                    'jetbrains-idea',
                    'clion',
                    'andrond-studio',
                },
                'prog':"~/bin/scripts/jetbrains.sh idea",
            },
            'jetbrains-clion':{
                'classes': {
                    '^jetbrains-jetbrains-idea.*',
                    '^jetbrains-clion.*',
                    '^jetbrains-andrond-studio.*',
                },
                'prog':"~/bin/scripts/jetbrains.sh clion",
            },
        }
{% endhighlight %}

Consider you want to jump to firefox at first and chromium if it's exists at
second. With priority stuff you can firstly make `M-w` to get firefox and
take `M-w` again after this to get chromium, it is very nice and convinient,
as I think. And here the code:

{% highlight python %}
#!/usr/bin/env python3

""" i3 window tag circle
Usage:
    circle.py daemon

Created by :: Neg
email :: <serg.zorg@gmail.com>
github :: https://github.com/neg-serg?tab=repositories
year :: 2017

"""

import i3ipc
import re
import errno
import os
import time

from queue import Queue
from sys import exit
from docopt import docopt
from cycle_settings import *
from singleton_mixin import *
from script_i3_general import *
from threading import Thread, enumerate

glob_settings=cycle_settings().settings

class cycle_window(SingletonMixin):
    def __init__(self):
        self.tagged={}
        self.counters={}
        self.restorable=[]
        self.interact=1

        for i in glob_settings:
            self.tagged[i]=list({})
            self.counters[i]=0

    def go_next(self, tag):
        def tag_conf():
            return glob_settings[tag]

        def cur_win():
            return self.current_win

        def cur_win_in_current_class_set():
            tag_classes_set=set(glob_settings[tag]["classes"])
            return cur_win().window_class in tag_classes_set

        def current_class_in_priority():
            if not cur_win_in_current_class_set():
                return cur_win() == tag_conf()["priority"]
            else:
                return True

        def is_priority_attr():
            return "priority" in tag_conf()

        def class_eq_priority():
            return item['win'].window_class == tag_conf()["priority"]

        def inc_c():
            self.counters[tag]+=1

        def target_i():
            return self.tagged[tag][target_]

        def run_prog():
            prog=tag_conf()["prog"]
            i3.command('exec {}'.format(prog))

        def go_next_(inc_counter=True,fullscreen_handler=True):
            if fullscreen_handler:
                fullscreened=i3.get_tree().find_fullscreen()
                for win in fullscreened:
                    if cur_win_in_current_class_set() and cur_win().id == win.id:
                        self.interact=0
                        win.command('fullscreen disable')

            target_i()['win'].command('focus')
            target_i()['focused']=True
            if inc_counter:
                inc_c()

            if fullscreen_handler:
                now_focused=target_i()['win'].id
                for id in self.restorable:
                    if id == now_focused:
                        self.interact=0
                        i3.command('[con_id=%s] fullscreen enable' % now_focused)

            self.interact=1

        def go_to_not_repeat():
            inc_c()
            self.go_next(tag)

        try:
            if len(self.tagged[tag]) == 0:
                if "prog" in tag_conf():
                    run_prog()
                else:
                    return
            elif len(self.tagged[tag]) <= 1:
                target_=0
                go_next_(fullscreen_handler=False)
            else:
                target_=self.counters[tag] % len(self.tagged[tag])

                if is_priority_attr() and not current_class_in_priority():
                    if len([ i for i in self.tagged[tag] if i['win'].window_class == tag_conf()["priority"] ]) == 0:
                        run_prog()
                        return

                    for target_,item in zip(range(len(self.tagged[tag])),self.tagged[tag]):
                        if class_eq_priority():
                            fullscreened=i3.get_tree().find_fullscreen()
                            for win in fullscreened:
                                tag_classes_set=set(glob_settings[tag]["classes"])
                                if win.window_class in tag_classes_set and win.window_class != glob_settings[tag]["priority"]:
                                    self.interact=0
                                    win.command('fullscreen disable')
                            go_next_(inc_counter=False)
                            return
                elif self.current_win.id == target_i()['win'].id:
                    go_to_not_repeat()
                else:
                    go_next_()
        except KeyError:
            invalidate_tags_info()
            self.go_next(tag)

    def switch(self, args):
        switch_ = {
            "next": self.go_next,
        }
        if len(args) == 2:
            switch_[args[0]](args[1])
        elif len(args) == 1:
            switch_[args[0]]()

def find_acceptable_windows_by_class(tag, wlist):
    cw=cycle_window.instance()
    for con in wlist:
        if ("classes" in glob_settings[tag]) and (con.window_class in glob_settings[tag]["classes"]):
            cw.tagged[tag].append({ 'win':con, 'focused':False })
        elif ("instances" in glob_settings[tag]) and (con.window_instance in glob_settings[tag]["instances"]):
            cw.tagged[tag].append({ 'win':con, 'focused':False })

def invalidate_tags_info():
    cw=cycle_window.instance()
    wlist = i3.get_tree().leaves()
    cw.tagged={}

    for i in glob_settings:
        cw.tagged[i]=list({})

    for tag in glob_settings:
        find_acceptable_windows_by_class(tag, wlist)

def add_acceptable(self, event):
    cw=cycle_window.instance()

    def add_tagged_win():
        cw.tagged[tag].append({'win':con,'focused':con.focused})

    con = event.container
    for tag in glob_settings:
        try:
            if ("classes" in glob_settings[tag]) and (con.window_class in glob_settings[tag]["classes"]):
                add_tagged_win()
            elif ("instances" in glob_settings[tag]) and (con.window_instance in glob_settings[tag]["instances"]):
                add_tagged_win()
        except KeyError:
            invalidate_tags_info()
            add_acceptable(self, event)

def del_acceptable(self, event):
    def del_tagged_win():
        if 'win' in cw.tagged[tag]:
            if cw.tagged[tag]['win'].id in cw.restorable:
                cw.restorable.remove(cw.tagged[tag]['win'].id)
        del cw.tagged[tag]

    cw=cycle_window.instance()
    con = event.container
    for tag in glob_settings:
        try:
            if ("classes" in glob_settings[tag]) and (con.window_class in glob_settings[tag]["classes"]):
                del_tagged_win()
            elif ("instances" in glob_settings[tag]) and (con.window_instance in glob_settings[tag]["instances"]):
                del_tagged_win()
        except KeyError:
            invalidate_tags_info()
            del_acceptable(self, event)

def save_current_win(self,event):
    con=event.container
    cw=cycle_window.instance()
    cw.current_win=con

def handle_fullscreen(self,event):
    cw=cycle_window.instance()
    con=event.container
    if cw.interact == 1:
        if con.fullscreen_mode:
            if con.id not in cw.restorable:
                cw.restorable.append(con.id)
        if not con.fullscreen_mode:
            if con.id in cw.restorable:
                cw.restorable.remove(con.id)


if __name__ == '__main__':
    argv = docopt(__doc__, version='i3 window tag circle 0.5')

    if argv["daemon"]:
        i3 = i3ipc.Connection()
        name = 'circled'

        cw=cycle_window.instance()
        cw.current_win=i3.get_tree().find_focused()

        mng=daemon_manager.instance()
        mng.add_daemon(name)

        def cleanup_all():
            daemon_=mng.daemons[name]
            if os.path.exists(daemon_.fifo_):
                os.remove(daemon_.fifo_)

        import atexit
        atexit.register(cleanup_all)

        invalidate_tags_info()

        i3.on('window::new', add_acceptable)
        i3.on('window::close', del_acceptable)
        i3.on("window::focus", save_current_win)
        i3.on("window::fullscreen_mode", handle_fullscreen)

        mainloop=Thread(target=mng.daemons[name].mainloop, args=(cw,)).start()

        i3.main()
{% endhighlight %}

A complex point here is about fullscreen. i3 has pretty strict fullscreen
behaviour. For example you cannot jump from it if the target window is on the
same workspace, etc. To hack it I've used some tricky stuff to enable/disable
and restore(!) fullscreen.

If you have some questions you are welcome to ask me in english or in
russian. I hope that you will consider this stuff nice and handy as I done.
