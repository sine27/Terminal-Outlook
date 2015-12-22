# Terminal-Outlook

> vi ~/.bash_profile

```
export PS1='😶 \[\033[1;36m\]\A \[\033[1;35m\]@\u: \[\033[1;33;4m\]\w\[\033[m\] \$\n\[\033[1;m\]'
export CLICOLOR=1
export LSCOLORS=CxExCxDxCxegedabagaced
alias ls='ls -GFh'
```

Some PS1 Codes
These are generally called Bash prompt "escape sequences", as they consist of a backslash (the standard escape character) followed by a regular character. Although there are many available, I will just list the one's I've used:

u - the current user's username
h - the current machine's hostname
j - the number of jobs managed by this shell
@ - the current time (12 hour format)
d - the current date
w - path of the current working directory
W - just the current working directory
e - an ASCII escape character (033)
n - adds a newline character

Summary
We can now clearly see how '[u@h W]$ ' becomes [bdjnk@PowerMAP icons]$ because it makes use of some basic PS1 codes plus some plain text. No problem.

Checking State
There are two states I want to track. The exit status of the previously run command, and whether or not I'm logged in as root. You can see from the examples that the PS1 is aware of these circumstances. Here's how:

Exit Status
In Linux, an exit code of anything other than 0 generally indicates some kind of error. To get the exit code, we need to take a look at the $? variable. Try running various commands, then running echo $? right after from the same prompt. This should be 0 if the command completed successfully, and something other than 0 if it didn't.

For our purposes, we need to test the exit status. We can use the form if [ $? -eq 0 ]; then echo 'good'; else echo 'bad'; fi replacing the echo commands with whatever we want to do in each case.

Root Prompt
In Linux, the root user receives the user id 0. The current user id for a given prompt is stored in the variable $EUID. Thus we test for root with if [ ${EUID} -eq 0 ]; then echo 'root'; else echo 'user'; fi.

Gathering Data
Linux is blessed with a plethora of fast little command line utilities, perfect for gathering data to use in your Bash prompt. You'll notice I use the full path of each command. This allows the system to use the utility without knowing anything about the PATH variable.

Number of Files
I first use /bin/ls -1 to generate a list of all the files (and folders, which are also called files in Linux) in the current directory. I then use | to pipe (pass) that data to /usr/bin/wc -l which counts the number of newlines. This provides the number of files in the current directory.

The complete command is /bin/ls -1 | /usr/bin/wc -l

Size of Contents
I first use /bin/ls -lah which lists, in long form, all the files in the current directory, using human readable size information. I then use | to pipe that data to /usr/bin/grep -m 1 total which outputs the first (a maximum of 1) line containing the word "total". Finally, I use /bin/sed 's/total //' to replace the text "total " with nothing, thus discarding it and retaining just the size of the current directory's contents.

The complete command is /bin/ls -lah | /usr/bin/grep -m 1 total | /bin/sed 's/total //'


ANSI Colors
There are 16 ANSI Colors, which are actually 8 colors, each having "normal" and "bright" intensity variants. The colors are black, red, green, yellow, blue, magenta, cyan, and white. In certain circumstances bright intensity could be actually brighter, or, in the case of an xterm, it could be bold.

Usage Syntax
To set a font to the color and style desired, we use the syntax \033[#m where # can be a valid set of semicolon separated numbers. The \033[ is a control sequence initiator which begins an escape sequence. In our case the sequence is a set of "m" terminated select graphic rendition parameters for rendering the text as desired. More details.

If that makes no sense, just concentrate on which numbers to use. Font color makes use of 30 to 37. Background color makes use of 40 to 47. A 0 (or no value given) for the font color resets it to the default, which depends on the setup. For additional rendering options, 1 sets the font to bright / bold, 4 sets it to underlined, 7 sets it to negative colors.

This might help. Run the command: echo -e "testing \033[0;37;41mCOLOR1\033[1;35;44mCOLOR2\033[m"

This should output testing COLOR1COLOR2 with COLOR1 being white (light gray) text on a red background, and COLOR2 being bold / bright magenta text on a blue background. All subsequent text should be normally colored.

Try playing around with that echo command until you feel a bit more comfortable.

Reference Table
Obviously you don't want to come back to this page every time you want to change your Bash colors. Here is the shell script (origin) used to generate the lovely color table:

```
#!/bin/bash
#
# This file echoes a bunch of color codes to the terminal to demonstrate
# what's available. Each line is the color code of one forground color,
# out of 17 (default + 16 escapes), followed by a test use of that color
# on all nine background colors (default + 8 escapes).
#
T='gYw'   # The test text
echo -e "\n                 40m     41m     42m     43m     44m     45m     46m     47m";
for FGs in '    m' '   1m' '  30m' '1;30m' '  31m' '1;31m' '  32m' '1;32m' '  33m' '1;33m' '  34m' '1;34m' '  35m' '1;35m' '  36m' '1;36m' '  37m' '1;37m';
  do FG=${FGs// /}
  echo -en " $FGs \033[$FG  $T  "
  for BG in 40m 41m 42m 43m 44m 45m 46m 47m;
    do echo -en "$EINS \033[$FG\033[$BG  $T \033[0m\033[$BG \033[0m";
  done
  echo;
done
echo
```
###Finally, the Goal
Here is the code shared by both:
```
c="\[\033["

This is the PS1 used to create the slightly steam-punk looking Bash prompt, using box-drawing characters:

PS1="\n${p}╔ǁ ${c}1;3\$(if [ \$? -eq 0 ]; then echo '2'; else echo '1'; fi)m\]*$p ǁ ${c}36m\]\@ \d$p ǁ ${c}35m\]\j$p ǁ ${c}30m\]\u${c}33m\]@\h$p ǁ ${c}1;34m\]\w$p ǁ ${c}32m\]\$(/bin/ls -1 | /usr/bin/wc -l) files, \$(/bin/ls -lah | /usr/bin/grep -m 1 total | /bin/sed 's/total //')${c}m\]$p ǁ═╗\n╚═${c}1;3\$(if [ ${EUID} -eq 0 ]; then echo '1'; else echo '4'; fi)m\]»${c}m\] "
And this is the PS1 used to create the more compact and somber looking Bash prompt:

n="${c}m\]"
PS1="$n\n${c}1;3\$(if [ \$? -eq 0 ]; then echo '2'; else echo '1'; fi);40m\]*$n ${c}36;40m\]\@ \d$n ${c}35;40m\]\j$n ${c}37;40m\]\u${c}33m\]@\h$n ${c}1;34;40m\]\w$n ${c}32;40m\]\$(/bin/ls -1 | /usr/bin/wc -l) files, \$(/bin/ls -lah | /usr/bin/grep -m 1 total | /bin/sed 's/total //')${c}m\]\n${c}1;3\$(if [ ${EUID} -eq 0 ]; then echo '1'; else echo '4'; fi);47m\]>$n "
```
Hopefully, at this point you are reading these lines of seeming gibberish like plain English. If not, play more. You'll get it.

###256 (8-bit) Colors
It is also possible (in these newfangled modern times) to use a full 256 colors in most consoles. The color table below can be used to determine the code for each color by adding the column and row number. In order to make use of these codes, we use the syntax 33[38;5;#m for the foreground (text) and 33[48;5;#m for the background, or in a single statement like 33[38;5;#;48;5;#m to set both at once, where # is an 8-bit (0-255) color code.

The best way to understand is to try it out. Run the command: echo -e "testing \033[38;5;196;48;5;21mCOLOR1\033[38;5;208;48;5;159mCOLOR2\033[m"


