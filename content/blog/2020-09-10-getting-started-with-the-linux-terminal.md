---
title: Getting Started with the Linux Terminal
date: 2020-09-10T23:07:11.783Z
image: /images/bash-161382_960_720.png
tags:
  - Linux
  - Terminal
  - Command line
draft: false
---
*So you just installed Linux and you need to get comfortable with the terminal right? Here's the right place to get you started.*

You're probably familiar with the Windows operating system and you could do a whole bunch by moving around using the GUI yeah right but you are here because you have installed a Linux operating system and you want to get your feet wet and enjoy all the goodies the OS has to offer. Similar to the Windows operating system, most Linux distributions now have a GUI too with which you can navigate around but the real fun and power is in knowing how to use the Linux terminal, or command line.

<!-- excerpt -->

In this write up, we will cover the basic commands you need to get started. One thing you need to always keep in mind is that in Linux everything is a [file](https://en.wikipedia.org/wiki/Everything_is_a_file).

First off, open up a terminal window. You can do this by either opening up your application menu and select Terminal or press `Ctrl+Alt+T`on your keyboard (default terminal shortcut) and you should have a terminal window staring right at you.

`whoami`

The whoami command returns the username of the current user under which the command is being run. This command is available in both the Unix/Linux operating systems as well as the Windows operating system.

Other commands similar to whoami you should know about are `w, who`.

`id`

The id command prints real and effective user and group IDs.

`sudo`

Think of this command as 'Run as Administrator' in Windows (that's close enough). It grants you the privilege to run commands as the super user - `root`. Note that for this to work, the user must be part of the `sudo` group.

Now to some system navigation.

`pwd`

pwd returns the name of the present working directory. So, for instance if you're in your home folder, pwd will return `/home/<username>`. If you decide to navigate to the /opt folder then pwd will be `/opt`.

`cd`

cd allows you to change directory from one part of the file system to another. Think of this as the way you open File Explorer in Windows and you use your cursor to click around, from one folder to another. cd gives the options to change into various folders right from the terminal. Assuming there is a folder called `secret` in your home directory, you can navigate to it using `/home/<username>/secret` and you'll be right in that folder. As a shortcut, instead of using `/home/<username/secret` you can do `~/secret`, the ~ represents your home directory. 

`ls`

With the ls command, you can see all the files contained within the present working directory (including hidden files/directories).

`mkdir`

mkdir stands for make directories. It create directory(-ies) in the present working directory by default or the specified path if one is provided.

`rmdir`

Opposite to mkdir, this command removes directory(-ies).

`cat`

The cat command concatenate files and prints it on the standaard output. It can be used to view files (ASCII text) right from the terminal.

`nano`

This is a terminal-based text editor and it is easy to use for beginners. You can create and edit files with this editor.

You will also find yourself needing to update, upgrade, install or purge packages from time to time.

```
sudo apt update
sudo apt upgrade
sudo apt full-upgrade
sudo apt install <package>
sudo apt purge <package>
```

Finally, if you need to know more about a particular command use the manual: `man <command>` or `<command> --help`.

Thanks for reading.