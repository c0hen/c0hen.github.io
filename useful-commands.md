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

+ chronic: runs a command quietly unless it fails
+ combine: combine the lines in two files using boolean operations
+ errno: look up errno names and descriptions
+ ifdata: get network interface info without parsing ifconfig output
+ isutf8: check if a file or standard input is utf-8
+ ifne: run a command if the standard input is not empty
+ lckdo: execute a program with a lock held (deprecated)
+ mispipe: pipe two commands, returning the exit status of the first
+ parallel: run multiple jobs at once
+ pee: tee standard input to pipes
+ sponge: soak up standard input and write to a file
+ ts: timestamp standard input
+ vidir: edit a directory in your text editor
+ vipe: insert a text editor into a pipe
+ zrun: automatically uncompress arguments to command

#### data structure or configuration processing tools

+ jq
+ yq
+ tomlq
+ rq (see: open policy agent, Rego, kube-mgmt)

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
