# Quick reference

[TOC]


## Regex

Notepad++ uses [perl regex](https://www.boost.org/doc/libs/1_55_0/libs/regex/doc/html/boost_regex/syntax/perl_syntax.html) 

Vim uses its own flavor of regex that is similar in functionality to Perl. Enter search and replace strings as a command after typing `:`. End with `g` to apply to every match in a line. Prefix with `%` to apply to the entire file (e.g. `:%s`). Apply pattern to visually selected text by hitting `v`, selecting the text, then `:` to type the command, or hit up for a previous command. 

- (perl) `^<div id=.*\R` : Match single-line divs and any line ending
- (vim) `s/\s\+$//` : delete trailing whitespace for all lines in file
- (vim) `s/\(.\+\)$/\1  /` : Add a double space to end of each non-empty line


## ffmpeg

- `ffmpeg -fflags +genpts -i dvd.iso -map 0 -c copy dvd.mkv` : Copy dvd to mkv and generate missing timestamps
- `ffprobe -analyzeduration 100M -probesize 100M dvd.iso` : Seek ahead on disc to discover subtitle or other streams

### forloops

Extract subtitles to same directory as video, with matching filenames.

    for i in ./*.mkv; do ffmpeg -i "$i" -map 0:s:0 -c:s text "./${i%.*}.en.srt"; done

Mux streams from multiple input files. Remove audio from first file and copy audio from second file.

    one=(../number1/*.mkv)
    two=(../number2/*.mkv)
    for i in "${!h264[@]}"; do ffmpeg -i "${h264[i]}" -i "${h265[i]}" -map 0 -map -0:a:0 -c copy -map 1:a:0 -c:a copy ./"$(basename "${h264[i]}")"; done

### encode recipes

- __8-bit H264 lvl 4__: `ffmpeg -i file.mkv -c:v libx264 -profile:v high -level:v 4.0 -vf format=yuv420p -crf 18 outfile.mkv`
- __2-Channel AC3__: `ffmpeg -i file.mkv -map 0:a:0 -ar 44100 -ac 2 -c:a ac3 -b:a 224k`
- __Extract Subtitles__: `ffmpeg -i file.mkv -map 0:s:0 -c:s text subs.srt`
- __PS3 Compatibility__ : `ffmpeg -i file.mkv -map 0:v -map 0:a -c copy -map_chapters -1 file.mp4` : Change container from mkv to mp4, remove chapters, copy only audio and video streams, do not re-encode
- __Divx Player (USB 2.0)__ : `ffmpeg -i file.mkv -q:a 3 -b:v 2M -maxrate:v 3.5M -vtag xvid -vf scale=640:-1,setsar=1:1 file.avi`: Transcode high def mkv to standard def avi and limit bitrate < 4000k
- __Retroarch Web MP4__ : `ffmpeg -i file.mkv -pix_fmt yuv420p -crf 1 -s 512x480 -sws_flags neighbor -b:a 320k file.mp4` : Start with lossless retroarch recording, encode and scale from yuv444p to yuv420p, double original resolution from 256x240 to compensate for scaling artifacts, use nearest neighbor scaling for sharp pixels


### yt-dlp

- `yt-dlp -f "bv[ext=mp4][height=1080]+ba[ext=m4a]"` : combine best 1080p h264 with best aac


## Godot

Note that code in a GD script takes precedence over what you set in the UI. 

---

__Change viewport resolution and scale:__

    func _ready():
        get_tree().set_screen_stretch(2, 1, Vector2(320, 240)) 

- _Project_ menu > _Project Settings_ > _General_ tab > _Display_ then _Window_ in sidebar, set _Width_ and _Height_, set _Mode: Viewport_, and _Aspect: Keep_

__Fix blurry sprites:__

- Select sprite file in _FileSystem_ tab, then under _Import_ tab, uncheck _Filter: On_, click _Reimport_


## Wine

* `wine cmd` : Load cmd.exe using stdin/out/err
* `wineconsole` : Load cmd.exe CUI in new window


## Pacman

* `pacman -Qo /usr/bin/appname` : Query what package a file belongs to


## Vscode

* `Ctrl-Shift-P` : Open command pallette
* `Ctrl-]` or `[` : Indent or unindent highlighted code
* `Ctrl-K Ctrl-C` : Comment out highlighted code block
* `Ctrl-K Ctrl-U` : Uncomment code block
* `Ctrl-Shift-K` : Delete current line
* `Ctrl-X` : Delete line and copy to clipboard


## Bash / Linux

* `setfacl --remove-all -R [file]` : Recursively remove all ACLs from a file or folder
* `ls -i` ; `find / -inum [inode]` : List inodes then find inode (identify hard links)
* `diff -u oldfile newfile > patchfile` : create a patch file
* `patch -u oldfile -i patchfile` : apply a patch file
* `ssh -N -L 8001:localhost:8000 user@host` : redirect local port 8001 to remote port 8000
* `ssh -N -D 9999 user@host` : start a socks proxy with no shell
* `sudo netstat -nltp` / `ss -nltp` : show addresses and ports for running services
* `sudo -u username -s -E` : Open a shell as another user but preserve your env vars
* `sudo -i -u username` : Open a login shell like with _su - username_
* `useradd -r -s /sbin/nologin username` : Create a system user for running services (do not create home folder)
* `cat -A` : Show tabs, newlines, and carriage returns
* `Alt-B / Alt-F` : Back/forward one word
* `Ctrl-K / Ctrl-U` : Cut from cursor to end/beginning of line
* `if ./exitzero.sh ; then echo success; else echo nonzero; fi` : Return success if script runs __exit 0__
* `2>&1` : Redirect stderr to stdout so error messages get sent to the same place as standard output
* `<(echo 'file contents')` : Pass the output of a command to another command that expects a file
* `set -x` : Print all commands before executing them (does not wait before execution)

### Parameter Expansion

* `${filename##*.}` : Get the file extension from a standard filename
* `${filename%.*}` : Remove the file extension from a standard filename

### Other

- `screen -dmS session-name bash -c 'echo hello world; exec bash'` : Run a command in a detached screen terminal and keep it alive with exec bash
- `pandoc -s file.docx --wrap=none -t markdown -o file.md` : Convert word doc to markdown
- `dpkg -L packagename` : List all files installed for package
- `sudo ufw allow in on eth0 from 192.168.1.0/24 to 192.168.1.2 port 8096 comment 'Jellyfin'`


## Python

- `>>> ''.join(filter(lambda x : x in string.ascii_letters, '!My$Weird&Str#'))` : return string with only ascii letters
- `>>> 'ab '.translate(str.maketrans('ab', 'yz', ' '))` : return string with a->y b->z and no spaces
* `$ pipdeptree -fl` : show dependency tree to identify core packages
* `>>> [n for n in iterable if re.match(".*searchstring", n)]` : Filter a list of strings
* `>>> inspect.signature(the_function)` : Display parameters of a function
* `$ autopep8 -i scriptfile.py` : Format your python code and replace the file

### Django

- `>>> list(Tags.objects.filter(mdfile_id=46).values_list('tag', flat=True))` : Create a list from queryset values
* `python manage.py makemigrations --dry-run` : Check if migrations exist for any change in the models
* `python manage.py shell` : Open a django shell where you can import models
* `>>> from appname.models import ModelName` : Import a model in the shell

### PyInstaller

* `pyinstaller --clean --onefile myproject.py` : compile a single linux binary from a python script

### Virtual Environments

- `python -m venv ~/Pyvenvs/projectname` : Create env the recommended way as of 3.5
- `mkvirtualenv -p $HOME/.pyenv/versions/3.9.4/bin/python projectname` : create env with virtualenvwrapper
* `source ./myenv/Scripts/activate` : Activate a virtualenv in BASH

### Pyenv

* `pyenv install -l` : show all python versions available for install
* `pyenv activate myenv` : activate a virtual environment by name
* `pyenv deactivate` : deactivate a virtual environment
* `pyenv virtualenv 3.6.6 myenv` : create a virtual environment with python version and env name
* `pyenv uninstall myenv` : delete a virtual environment by its name
* `env PYTHON_CONFIGURE_OPTS="--enable-shared" LD_LIBRARY_PATH="$HOME/.pyenv/versions/3.6.6/lib/" pyenv install 3.6.6` : compile a python version for compatibility with pyinstaller


## Git

* `git cherry-pick -n <commit>` : don't commit the cherry-pick, just stage it
* `git reflog` : show refs for hard resets so you can recover from them
* `git tag archive/mybranch mybranch; git push --tags` : archive a branch using tags
* `git branch -vv` : show tracking branches next to local branches
* `git diff branch1 branch2 file.txt` : Compare same file between branches
* `git push origin --delete my-feature-branch` : Delete remote branch
* `git push origin :my-feature-branch` : Delete a remote branch the old way
* `git ls-tree -r HEAD --name-only` : Show all committed files on current branch
* `git restore --staged .` : Unstage changes / Undo a git add
* `git checkout .` : Revert unstaged changes to most recent commit
* `git reset --soft HEAD~1` : Go back one commit and keep changes to files
* `git reset --hard HEAD~1` : Go back one commit and discard changes to files

### git config

- `git config --list --show-origin --global` : show git config (global, local, or system)
- `core.autocrlf true` : checkout as CRLF, commit as LF
    - `core.safecrlf false` : suppress misleading warnings about CRLF conversion
- `core.autocrlf input` : checkout as LF, commit as LF
- `core.autocrlf false` : no conversions on checkout or commit


## Vim

* `:tabm` : Move tab to front of queue
* `:hi Error NONE` : disable error highlighting
* `:set list` / `nolist` : show listchars like tabs and newlines
* `:%retab!` : change file's indentation to tabs when _noexpandtab_ and _tabstop_ set
* INSERT `ctrl-u` : remove all characters to left of cursor
* `/\C` : case sensitive search (lowercase c to ignore case)
* `:%y+` : copy current file to clipboard
* `:source ~/.vimrc` : to reload your vimrc file
* `1 ctrl-g` : Show the name of the current file with full path

### netrw, windows, buffers

- `F1` (in netrw) : show netrw help page
- `:e ./` or `:Ex` / `:Vex` / `:Sex` : Open netrw in full or split modes
* `:bd` or `:bw` : delete buffer (close file or tab) without quitting vim
- `ctrl-6` / `:b#` : switch between previous and current buffer
- `:enew` : Create a new empty buffer
* `ctrl-w q` : close split window
* `ctrl-w o` or `:only` : close all other split windows


## Windows

* `super-x u s` : sleep the computer without a mouse
- `mklink mylink mytarget` : create a symlink in cmd.exe (not powershell)

---

__Stop mouse from waking computer:__

* _Control Panel_ > _Mouse_ > _Hardware_ tab > select _HID-compliant mouse_ > _Properties_ > _Change Settings_ > _Power Management_ tab > uncheck _Allow this device to wake the computer_


### Chocolatey

* `choco list -l` : list all locally installed packages
* `choco outdated` : list outdated packages

### Notepad++

- `CTRL + L`    delete line and copy to clipboard
- `CTRL + SHIFT + L`    delete line but don't copy to clipboard
- `SHIFT + TAB`     Un-indent (works for highlighted blocks too)
- `CTRL + D`    duplicate line on next line
