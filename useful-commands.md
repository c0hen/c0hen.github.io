---
layout: default
title: Useful commands and scripts
description: Useful command examples and scripts to modify to your liking.
tags: command shell script
---

**Some here are scripts, those begin with a shebang**
```sh
#!
```

* Table of contents
{:toc}

#### replace in place with sed
```sh
sed -i -e 's/^/#/' filename
```
#### -u to turn off buffering for sed

```sh
dmesg -T --follow | sed -u -e "s/^/$HOSTNAME/"
```

#### pz (pythonize shell)

operate on s (the line variable).
```sh
echo -e "example\nwikipedia" | pz 's += ".com"'
```

#### Vi(m)

- Show shell executable.
 ```sh
 :set shell?
 ```
- Execute a child shell process. Exiting returns to vi.
 ```sh
 :sh
 ```
- Run given shell commands and exit after pressing enter.
 ```sh
 :!printf ":help :!"
 ```
- Execute current line ('.') in sh and replace it in the vi buffer with the output.
 ```sh
 :.!sh
 ```
- Insert the output of the shell command in the buffer.
 ```sh
 :r!date
 ```
- Get the output of a shell command with system ( :help system ).
 ```sh
 :call system('touch /tmp/$(date +%Y%m%d)')
 ```
 ```sh
 :echo system('ls -lt /tmp/$(date +%Y)* | tail')
 ```
- Start a job that doesn't wait to finish ( :help job\_status ), execute with no shell.
 ```sh
 :call job_start(['/bin/bash', '-c', '{ sleep 60 && printf "DONE"; }'])
 ```
- Registers (+ * ~ / : % . -)
  - c - characterwise text
  - l - linewise text
  - b - blockwise text
  ```sh
  :registers
  "+p # paste in visual mode
  ```
  1. unnamed register "" fills when using delete or yank
  1. numbered registers "0-"9 (deletion history stack)
  1. small delete register "- (one line or less deleted)
  1. named registers "a-"z"A-"Z (write to F gets appended, f replaced)
  1. read only registers
     ": most recently executed command
     "% current file name
     ". last inserted text
  1. alternate file name register "# (in case of multiple open files)
  1. expression register "=
  1. last search result register "/
  1. GUI primary "+ and secondary "* , middle click clipboard
  1. GUI drop register "~ (drag and drop)

#### Bash job control

Ctrl+Z to suspend a program in bash. Sends SIGTSTP (Keyboard stop).

Start a job in the background.
```sh
sleep 4500 &
```
Bash shell builtins for jobs. Applies to current shell process, not all user's shells.
####
```sh
jobs --help
```
```sh
fg # Bring to foreground. No job spec, targets shells notion of the current job
bg %1 # Sends the first job to execute in the background
wait 20110 # Process ID or job spec
```
Remove the second job from current shell's job list, keeping it in the process table. Not reliable for running a program in the background (in case the parent shell is destroyed).
```sh
disown %2
```
Run a program in a new session. Allows bypassing potential signals from initial parents.
```sh
setsid --fork sleep 3600 # always create a new process
```
Format text in a way that is safe to use as shell input.
```sh
printf '%q\n' "It's magic!"
```

#### grep

Search directory recursively for lines starting 0 or more whitespace and $.
```sh
grep -ER '^(\w*)\$'
```

#### mail

[mailutils.txt](https://mailutils.org/manual/mailutils.txt)

3.5.2.1 Syntax of mail internal commands

Open the default inbox (mbox) instead of the spool file.

```sh
mail -f
```

With mail open, match string from the body.

```
h :/\\[SPAM\\]
```

Match string from the header From.

```
h From:/root
```

Header is Subject if omitted.
Escaped to prevent an attempt to POSIX regex match.

```
h /\\[SPAM\\]
```

Delete matching messages.

```
d /\\[SPAM\\]
```

#### moreutils

- chronic: runs a command quietly unless it fails
- combine: combine the lines in two files using boolean operations
- errno: look up errno names and descriptions
- ifdata: get network interface info without parsing ifconfig output
- isutf8: check if a file or standard input is utf-8
- ifne: run a command if the standard input is not empty
- lckdo: execute a program with a lock held (deprecated)
- mispipe: pipe two commands, returning the exit status of the first
- parallel: run multiple jobs at once
- pee: tee standard input to pipes
- sponge: soak up standard input and write to a file
- ts: timestamp standard input
- vidir: edit a directory in your text editor
- vipe: insert a text editor into a pipe
- zrun: automatically uncompress arguments to command

#### data structure or configuration processing tools

- jq
- yq
- tomlq
- rq (see: open policy agent, Rego, kube-mgmt)

#### [linkchecker](https://github.com/wummel/linkchecker)

Check HTML documents and websites for broken links. Return value is 1 when
- invalid links were found or
- link warnings were found and warnings are enabled
Supports clamav integration.

#### ncdu equivalent

For when [ncdu](https://dev.yorhel.nl/ncdu) is not installed.
```sh
du -d1 -h | sort -h
```

#### Faster alternatives written in Rust

fd - alternative to find.
debian package name fd-find, executable name on debian is
```sh
fdfind
```
ripgrep - alternative to grep.
debian package name ripgrep.
```sh
rg
```

#### pass

Unix style password management script, encrypts with gpg.

#### find and play videos flexibly

Using [mpv](https://mpv.io/), my preferred video player.

To use as is, create a shell script with below contents, arguments:

1st argument - how long ago was the video file last changed.

The rest of the arguments: path(s) to look for videos (glob, like 
```sh
/media/videos/201*
```
can be used as argument).

```sh
#!/bin/sh
find -L "${@:2}" \( -iname '*.mp4' -o -iname '*.webm' -o -iname '*.avi' \) -a -ctime -"$1" -print0 | xargs -0 mpv 
```

#### Rude wget robot to recursively download a site

```sh
#!/bin/sh
# shows the correct script path even if called from another script
if [ -z "$1" -o -z "$2" ]; then
  echo "Usage: $(basename $(readlink -nf $0)) site_to_download_recursively destination_directory" 
  exit 1
else
  wget --continue --recursive --execute robots=off --wait 1 --directory-prefix="$2" "$1"
  # Below line for offline mirroring, test on each site.
  # wget  --mirror --convert-links --adjust-extension --page-requisites --no-parent --continue --recursive --execute robots=off --wait 1 --directory-prefix="$2" "$1"
  exit 0
fi
```

#### ImageMagick
Requires [ImageMagick](https://www.imagemagick.org/)

Resize .jpg pictures in current directory level to roughly 1500x2000 pixels in size.
```sh
mogrify -resize 1500x2000 *.{jpg,JPG}
```

Create a black background in required size (for example, to use in a video).
```sh
convert -size 1920x1080 xc:black bg.png
```

#### [FFMpeg](https://ffmpeg.org/)

Dump audio using [parallel](https://www.gnu.org/software/parallel/). Parallel uses 1 thread for each core by default.

Make sure to change audio format according to source.
```sh
parallel ffmpeg -i '{}' -map 0:1 -c:a copy '{.}.m4a' ::: /media/video/source_video_file.mkv
```
