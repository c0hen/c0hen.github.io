---
layout: default
title: Useful commands and scripts
description: Useful command examples and scripts to modify to your liking.
tags: command shell script rust vim tools fast lightweight
---

**Some here are scripts, those begin with a shebang**
```sh
#!
```

* Table of contents
{:toc}

### Main

#### Control characters

- EOT - End Of Transmission, ^D (Ctrl+d)
- ETX - End Of Text, ^C
- LF - Line Feed, ^J
- CR - Carriage Return, ^R

#### skeleton template path

```sh
/etc/skel
```

#### Special variables and parameters of the shell

`man bash` does not mention special parameters in full form, they are listed under the Special Parameters section. This is due to all variables being prefixed with `$` on use. For example, `$$` is `$` in the manual. Same with dash.

Other shells may have different conventions.

- fish - `man fish-doc` provides translations to bash, search for `Special variables`
- zsh - `man zshparams`, search for `PARAMETERS SET BY THE SHELL`
- nu
```
wget --content-disposition \
https://github.com/nushell/nushell.github.io/blob/main/book/special_variables.md?raw=true
```

#### Shell operators, add to variable

For help on operators, search for `test` expression.

```sh
man $(basename $SHELL) # bash, dash
help test # fish
```

Helper function for environmental variables.

```sh
push_path_to_var() {
  if [ $# -ne 2 ] || [ ! -x "$2" ] || [ ! -d "$2" ]; then
    printf "Usage: push_path_to_var VAR executable_directory\n"
    printf 'Current $1: %s $2: %s\n' "$1" "$2"
    return 1
  fi
  # Appends the content of VAR to the argument array.
  # 'set --' to change the positional parameters without changing any options
  eval set -- "$1 $2 \$$1"

  # If the variable is empty, set it
  if [ -z "$3" ]; then
    eval "export $1=$2"
  else
    case ":$3:" in
      # If path already exists in VAR do nothing
      *":$2:"*) ;;
      # prepend path to VAR
      *) eval "$1=$2:$3" ;;
    esac
  fi
}

push_path_to_var PATH "$HOME/bin"
```

#### print all environment variables

```sh
printenv
jq -n env
jq -nr 'env|.HOME' # -r = raw
set # list values, including functions
```

#### man pages and paths

`manpath` is installed as part of [man-db](https://gitlab.com/man-db/man-db).

```sh
manpath --debug
```

User man pages should be installable to `$XDG_DATA_HOME/man` if XDG spec is followed and `XDG_DATA_HOME` is the default `$HOME/.local/share`. If it is a custom path, the relationship with `$HOME/.local/bin` can break.

```sh
manpath --debug

... snip ...
path directory /home/user/.local/bin is not in the config file
  adding /home/user/.local/share/man to manpath
... snip ...
```
`manp.c` from [man-db repository](https://gitlab.com/man-db/man-db.git) contains
```c
/* The directory we're working on isn't in the config file.
  See if it has ../man, man, ../share/man, or share/man
  subdirectories.  If so, and they haven't been added to
  the list, do. */
```

#### replace in place with sed
```sh
sed -i -e 's/^/#/' filename
sed -i '1s/^/added line before 1st line of file\n/' filename
```
Empty files don't have that first line and are not affected.

#### Find, grep, sed find files with pattern and replace

```sh
find group_vars/ -type f -name '*.yml' ! -exec grep -q -- '---' '{}' \; -print

find group_vars/ -type f -name '*.yml' \
! -exec grep -q -- '---' '{}' ';' \
-exec sed -i '1s/^/---\n/' '{}' '+'
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

#### Vim {#vim}

- Get enabled features info.
```
:help feature-list
```
- Delete empty lines.
```
:g/^$/d
:help :g
```
- Show shell executable.
```
:set shell?
```
- Execute a child shell process. Exiting returns to vi.
```
:sh
```
- Execute string that results from the evaluation of expression as an Ex command. Insert variable at the beginning of the file.
```
:execute append(0, &shiftwidth)
```
- Run given shell commands and exit after pressing enter.
```
:!printf ":help :!"
```
- Execute current line ('.') in sh and replace it in the vi buffer with the output.
```
:.!sh
```
- Insert the output of the shell command in the buffer.
```
:r!date
```
- Get the output of a shell command with system ( :help system ).
```
:call system('touch /tmp/$(date +%Y%m%d)')
```
```
:echo system('ls -lt /tmp/$(date +%Y)* | tail')
```
- Start a job that doesn't wait to finish ( :help job\_status ), execute with no shell.
```
:call job_start(['/bin/bash', '-c', '{ sleep 60 && printf "DONE"; }'])
```
- Registers (+ * ~ / : % . -)
  - c - characterwise text
  - l - linewise text
  - b - blockwise text
  ```
  :registers
  "+p # paste in visual mode
  ```
    1. unnamed register `""` fills when using delete or yank
    1. numbered registers `"0`-`"9` (deletion history stack)
    1. small delete register `"-` (one line or less deleted)
    1. named registers `"a`-`"z`,`"A`-`"Z` (write to F gets appended, f replaced)
    1. read only registers:
         - `":` most recently executed command;
         - `"%` current file name;
         - `".` last inserted text.
    1. alternate file name register `"#` (in case of multiple open files)
    1. expression register `"=`
    1. last search result register `"/`
    1. GUI primary `"+` and secondary `"*` , middle click clipboard
    1. GUI drop register `"~` (drag and drop)
    <br />
    <br />

- Keyboard combination to insert register content while in insert mode.
```
(Ctrl-R)+
```
- Redirect output.
```
:redir @a
:set textwidth?
:redir END
"ap
```
- Read and set registers as variables.
```
:let @a = "hello!"
"ap
```
- Options as variables.
```
:echo &textwidth
```
- Set diff algorithm
```
:help diffopt
:set diffopt+=algorithm:patience
```

#### Bash history expansion

Run last command. `man bash` `Event Designators`

```sh
sudo !!
```

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

#### Check if process is running, wait for response

`pgrep` full process path, run web server if not running, wait until port responds.
```sh
if ! pgrep --full jekyll >/dev/null ; then
  bundle exec jekyll serve >/dev/null &
fi

while ! nc -z localhost 4000; do
  sleep 0.1
done
```

#### Wait for changes to files

```sh
inotifywait --quiet --event close_write /tmp/
```

#### grep

Search directory recursively for lines starting 0 or more whitespace and $.
```sh
grep -ER '^(\w*)\$'
```

#### find delete empty directories

Starts from the deepest, deletes recursively up.
```sh
find . -depth -type d -empty -delete
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

Display help.

```
?
```

List messages (print message headers).

```
h
f *
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

Touch all messages, they'll be acted on as if they were read.

```
tou *
```

Quit without removing system mailbox.

```
x
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

#### url encode

```sh
printf https://example.com | jq -Rr @uri # --raw-input, --raw-output (treat as strings)
```

#### [linkchecker](https://github.com/wummel/linkchecker)

Check HTML documents and websites for broken links.

```
linkchecker --check-extern --ignore-url crates.io http://localhost:4000
```
Supports clamav integration.

#### ncdu equivalent

For when [ncdu](https://dev.yorhel.nl/ncdu) is not installed.
```sh
du -d1 -h | sort -h
```

### Rust tools

#### Bacon {#bacon}

[bacon](https://docs.rs/crate/bacon/latest) - background code checker designed for minimal interaction. It has [analyzers](https://dystroy.org/bacon/analyzers/) for Rust, python, javascript, typescript, C++, Go.

[Kondo](https://github.com/tbillington/kondo) recursively cleans project directories. Interface to `rm -rf` build files.
```sh
kondo --dry-run
```

#### Faster alternatives written in Rust

fd - alternative to find.
debian package name fd-find, executable name on debian is
```sh
fdfind
```
ripgrep - alternative to grep.
Debian package name ripgrep.
```sh
rg --pcre2 '(?!\\)_'
```
[runiq](https://github.com/whitfin/runiq) filters duplicate lines. Alternative to uniq.

[Uv](https://docs.astral.sh/uv/guides/projects/) claims to replace `virtualenv`, `pip`, `pip-tools`, `pipx`, `poetry`, `pyenv`, `twine`.
```sh
uv
```
`pre-commit`, `husky` alternative
```sh
prek
```
[watchexec](https://github.com/watchexec/watchexec) watches a path and runs a command whenever it detects modifications.

#### Other alternatives written in Rust

eza - ls

bat - cat

xh - httpie. Friendlier interface than curl, can print out an equivalent curl command.

[zellij](https://github.com/zellij-org/zellij) - tmux, screen

du-dust - du like

dua-cli - ncdu

hyperfine - benchmarking, default repeated runs

helix, evil-helix - vim

just, mask - make

### Extra

#### pandoc to convert between text formats {#pandoc-converter}

General markup coverter, supports lots of formats including epub, html, odt, csv, json.

Markdown to plain text.
```sh
pandoc -t plain README.md
```
Render markdown as html and view.
```sh
pandoc -t html README.md | w3m -T text/html
```

#### [pass](/secret/#create-password-store)

Unix style password management, encrypts with gpg.

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

#### nginx

Dump config (along with testing it)
```sh
nginx -T
```
#### Systemd get specific service properties
```sh
systemctl show --property ActiveState nginx.service
```
#### Systemd test if service is failed, get exit code
```sh
ecode=$(systemctl is-failed --quiet nginx.service)
```
