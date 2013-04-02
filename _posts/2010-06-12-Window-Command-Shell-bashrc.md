---
layout: post
title: "Windows Command Shell bashrc"
---
Since everyone at my office is currently using Windows 7, I had some issues to overcome with cmd.exe. My main issue being that it is not bash...

I remember that decades ago, when I was checking out debian for the first time I would always type dir instead of ls. Now trying to go back I find myself in the same position, again. Frustrating that is...

Anyway after looking around for a way to define an alias, I found doskey. Doskey lets you define macros, which is another name for alias. The basic syntax is

	doskey macroname=command
	doskey ls=dir

There are a bunch of command flags available, like $* that passes all parameters and $t which is used to separate commands. In any case it's best to read the link given above.

Doskey alone won't be very useful if we have to retype the macros everytime. So we'll put them in a macrofile. The macrofile is a list of macros without the preceding doskey.

	ls=dir
	cp=copy $*
	man = $1 /?

So everytime you open a cmd.exe you only have to issue doskey /macrofile=c:\path\to\macrofile.

But we can make it even more convenient. Open regedit and navigate to

	[HKEY_LOCAL_MACHINE\Software\Microsoft\Command Processor]

Create a new String here called `Autorun`. Everything in `Autorun` is executed at each start of `cmd.exe`. So you can insert the above macrofile call to it. Or you call a bat-file, let's call it bashrc.bat and call the macrofile from there.

The latter also gives you the option to issue other commands, without having to change the registry every time. Instead of polluting your PATH variable you could call

	set HOME=c:\Users\myusername

Or you could display a motivational message (Is there a windows command line fortune program?)

	echo Ah Ah Ah You did not say the magic word
	pause
	exit	

Hope this helped
