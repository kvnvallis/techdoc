# Quick reference

[TOC]


## ffmpeg

- `ffprobe -fflags +genpts -i dvd.iso -map 0 -c copy dvd.mkv` : Copy dvd to mkv and generate missing timestamps
- `ffprobe -analyzeduration 100M -probesize 100M dvd.iso` : Seek ahead on disc to discover subtitle or other streams

### recipes

- __8-bit H264 lvl 4__: `ffmpeg -i file.mkv -c:v libx264 -profile:v high -level:v 4.0 -vf format=yuv420p -crf 18 outfile.mkv`
- __2-Channel AC3__: `ffmpeg -i file.mkv -map 0:a:0 -ar 44100 -ac 2 -c:a ac3 -b:a 224k`
- __Extract Subtitles__: `ffmpeg -i file.mkv -map 0:s:0 -c:s text subs.srt`
- __PS3 Compatibility__ : `ffmpeg -i file.mkv -map 0:v -map 0:a -c copy -map_chapters -1 file.mp4` : Change container from mkv to mp4, remove chapters, copy only audio and video streams, do not re-encode
- __Divx Player (USB 2.0)__ : `ffmpeg -i file.mkv -q:a 3 -b:v 2M -maxrate:v 3.5M -vtag xvid -vf scale=640:-1,setsar=1:1 file.avi`: Transcode high def mkv to standard def avi and limit bitrate < 4000k
- __Retroarch Web MP4__ : `ffmpeg -i file.mkv -pix_fmt yuv420p -crf 1 -s 512x480 -sws_flags neighbor -b:a 320k file.mp4` : Start with lossless retroarch recording, encode and scale from yuv444p to yuv420p, double original resolution from 256x240 to compensate for scaling artifacts, use nearest neighbor scaling for sharp pixels


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


## Django

* `python manage.py makemigrations --dry-run` : Check if migrations exist for any change in the models
* `python manage.py shell` : Open a django shell where you can import models
* `from appname.models import ModelName` : Import a model in the shell


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

* `diff -u oldfile newfile > patchfile` : create a patch file
* `patch -u oldfile -i patchfile` : apply a patch file
* `ssh -N -D 9999 user@host` : start a socks proxy with no shell
* `sudo netstat -nltp` / `ss -nltp` : show addresses and ports for running services
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

* `${filename#*.}` : Get the file extension from a standard filename
* `${filename%.*}` : Remove the file extension from a standard filename

### Screen

- `screen -dmS session-name bash -c 'echo hello world; exec bash'` : Run a command in a detached screen terminal and keep it alive with exec bash


## Python

* `>>> [n for n in iterable if re.match(".*searchstring", n)]` : Filter a list of strings
* `>>> inspect.signature(the_function)` : Display parameters of a function
* `$ autopep8 -i scriptfile.py` : Format your python code and replace the file
* `$ python -m virtualenv ./myvenv` : Create a virtualenv
* `$ source ./myenv/Scripts/activate` : Activate a virtualenv in BASH


## PyInstaller

* `pyinstaller --clean --onefile myproject.py` : compile a single linux binary from a python script


## Virtualenvwrapper

- `mkvirtualenv -p $HOME/.pyenv/versions/3.9.4/bin/python projectname` : create a virtualenv


## Pyenv

* `pyenv install -l` : show all python versions available for install
* `pyenv activate myenv` : activate a virtual environment by name
* `pyenv deactivate` : deactivate a virtual environment
* `pyenv virtualenv 3.6.6 myenv` : create a virtual environment with python version and env name
* `pyenv uninstall myenv` : delete a virtual environment by its name
* `env PYTHON_CONFIGURE_OPTS="--enable-shared" LD_LIBRARY_PATH="$HOME/.pyenv/versions/3.6.6/lib/" pyenv install 3.6.6` : compile a python version for compatibility with pyinstaller


## Git

* `git branch -vv` : show tracking branches next to local branches
* `git diff branch1 branch2 file.txt` : Compare same file between branches
* `git push origin --delete my-feature-branch` : Delete remote branch
* `git ls-tree -r HEAD --name-only` : Show all committed files on current branch
* `git restore --staged .` : Unstage changes / Undo a git add
* `git checkout .` : Revert unstaged changes to most recent commit
* `git reset --hard HEAD~1` : Revert to the commit before the most recent commit

### git config

- `core.autocrlf true` : checkout as CRLF, commit as LF
    - `core.safecrlf false` : suppress misleading warnings about CRLF conversion
- `core.autocrlf input` : checkout as LF, commit as LF
- `core.autocrlf false` : no conversions on checkout or commit


## Vim

* `:hi Error NONE` : disable error highlighting
* `:set list` / `nolist` : show listchars like tabs and newlines
* `:%retab!` : change file's indentation to tabs when _noexpandtab_ and _tabstop_ set
* INSERT `ctrl-u` : remove all characters to left of cursor
* `/\C` : case sensitive search (lowercase c to ignore case)
* `:%y+` : copy current file to clipboard
* `ctrl-w q` : close split window
* `ctrl-w o` or `:only` : close all other split windows
* `:source ~/.vimrc` : to reload your vimrc file
* `1 ctrl-g` : Show the name of the current file with full path
* `:bd` or `:bw` : close file without quitting vim


## Windows

* `super-x u s` : sleep the computer without a mouse

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
