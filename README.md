# tmux-config
Custom tmux config file to spawn a tmux session already populated

## Table of Content

- [Other's effort](#others-effort)
- [Objectives](#objectives)
- [Process](#process)
  - [1. Create a new session](#create-session)
  - [2. Create different panes](#create-panes)
  - [3. Move around the panes](#move-panes)
  - [4. Show the clock](#show-clock)
  - [5. Miscellaneous information](#miscellaneous-information)
- [How to](#how-to)
- [Conclusions](#conclusions)

## Other's effort

Let me start by saying that this result is not solely my own effort, there is also the effort of other people before me.  
Here a some links to their work (in the order in which I discovered them):
1. https://gist.github.com/Muzietto/325344c2b1b3b723985a85800cafef4f
2. https://gist.github.com/sdondley/b01cc5bb1169c8c83401e438a652b84e

## Objectives

First of all I wanted multiple panes, in a specific order, already present in the tmux session I started.  
Moreover I wanted some commands to be already running when the tmux session was shown to me.  
Finally (and the one that took me the most time to figure out how to do from the configuration file) I wanted one of the panes to have opened the clock (that can be shown using one of the default key bindings of tmux).

## Process

I started by doing a few things: The first one was to open the manual 

```bash
man tmux
```

Then I opened the first of the links you can find above related to the efforts of other people, [@Muzietto](https://gist.github.com/Muzietto)'s link, to get a basic understanding of how to configure the configuration file for tmux.

I performed some tests at first and I learned some things that can be useful for others as well:

1. Not all tmux commands are able to accept shell-commands as arguments;
2. There are multiple ways to perform the same operation;

So let's break down the resulting file I created:

```bash
new -s custom-session 'htop'
split-window -h -l 55% 'watch -d -p sensors'
split-window -h
split-window -v -l 80%
select-pane -t 2
clock-mode
select-pane -t 3
```

### 1. Create a new session

The tmux command ```new``` is what you need to create a new session. You can just use that and nothing else after it.
I decided to give the session I created a name, therefore I used the flag ```-s``` with the name I chose.
Finally the ```new``` tmux command accepts shell-commands as arguments, thus I added ```'htop'``` after it.
If you stop here, and use only this command in your tmux.conf file, the tmux session you create with it will have a single pane the size of the window and htop running in it.

In the CLIENT AND SESSIONS section of the MAN TMUX(1) you can find more details about this tmux command searching for ```new-session```.

### 2. Create different panes

The tmux command ```split-window``` is what you need to split the current window in two different panes.
The content of the current window is moved to the top pane that is created (also pane 0) and the other pane created (also pane 1) will be empty and will be where your focus is landing. This is the default behavior.
Default behavior is also the way in which the window is split, that is vertically, one pane on the top side of the window and the other on the bottom side of the window.
Another default behavior is the size of the 2 panes created. They will have the same size.

The result I wanted was not achievable using the default behavior so:
I used the flag ```-h``` to have an horizontal split of the window in two panes.
I did not want the two panes to have the same size so I used the ```-l``` flag to customize the size of the panes.
The ```-l``` flag allows the usage of both an absolute value in number of lines (in case of vertical splitting) or columns (in case of horizontal splitting) and a percentage of the size of the window to configure the size of the panes.
Important detail, the size you are configuring is for pane 1.
Finally the ```split-window``` tmux command accepts shell-commands as arguments, thus I added ```'watch -d -p sensors'``` after it.

In the WINDOWS AND PANES section of the MAN TMUX(1) you can find more details about this tmux command searching for ```split-window```.

Two panes side by side with information I need to keep a close eye on. We are getting there!

I needed two more panes though, therefore two more ```split-window``` operations were needed, and with them I created pane 2 and pane 3 respectively, on the right of pane 1, one over the other.

### 3. Move around the panes

After the creation of the panes I needed I was missing only one step, showing the clock on the right top pane (also pane 2).
As I mentioned when you create a new pane the focus goes either on the bottom one or the right one. In this case the bottom one so I needed to move to a different pane. Easy enough when you are manually configuring your tmux window, a bit trickier in the configuration file.
Not too difficult though.
The tmux command ```select-pane``` is what you need to move from one pane to the other, you only need to know where to move.
The ```-t``` flag allows you to select the target for the tmux command, in this case it is just to move to said target, but you may remember it if you ever tried to reattach to a tmux session among many others already opened. You may have used the command ```tmux attach -t xxx```.

In the WINDOWS AND PANES section of the MAN TMUX(1) you can find more details about this tmux command searching for ```select-pane```.

### 4. Show the clock

This toook me hours. Mostly because I did not think to search for ```clock``` in the man page, but I searched for ```time```, which is how the clock is named in the section DEFAULT KEY BINDINGS in the man page, where you find the list of all the quick actions you can do manually inside a tmux section.
Not my case though. I have to write the command in the config file.
So it took some troubleshooting and a weird route to find the correct tmux command to add to the config file.

In this case I found help in the work of [@sdondley](https://gist.github.com/sdondley), in the section of his document (linked above) "Introducing key tables".
In there I found that running the command ```C-b :lsk -T prefix``` (C-b means CTRL+b) inside a tmux session will show the list of all the default key bindings (same as the DEFAULT KEY BINDINGS section of the man page) including the related tmux command they are recalling.
And there I found the ```clock-mode``` tmux command, that does exactly what I needed.

In the MISCELLANEOUS section of the MAN TMUX(1) you can find more details about this tmux command searching for ```clock-mode```.

### 5. Miscellaneous information

1. I mentioned there are multiple ways to perform the same operation: there is a tmux command ```resize-pane``` which does just that, resize the target pane, so if you want to create a pane first and then resize it later, it's possible;
2. You do not necessarily need to first move to the correct pane to spawn a clock in it as the ```clock-mode``` tmux command also supports the ```-t``` flag.

## How to

After you create your own custom tmux file you will probably find some mindbending facts.

1. You can run the command ```tmux -f {custom_config_file}```. This will open a tmux session that does not follow the configuration file and a second tmux session that does follow the configuration file. And you will be inside the first (aka wrong) tmux session.
2. If you want to have tmux see your configuration file without the usage of the ```-f``` flag you need to name the configuration file either ```tmux.conf``` and locate it in the ```/etc/``` folder or ```.tmux.conf``` and locate it in the ```/home/{user}/``` folder.
3. Even if you name the configuration file correctly and you locate it in the correct folder running ```tmux``` by itself will not create a new tmux session as you configured it, you need to run ```tmux attach``` without any other tmux session currently running.
This because:

```If no server is started, attach-session will attempt to start it; this will fail unless sessions are created in the  configuration file.```

Quoted from the MAN TMUX(1), section CLIENTS AND SESSIONS, at the command ```attach-session```.

## Conclusions

This was a fun journey for a quick and easy project!
Let me know if you give it a try and feel free to show me your results!

