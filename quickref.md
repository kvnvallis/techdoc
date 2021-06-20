# Quick reference

[TOC]


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


## Bash

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

* `git ls-tree -r HEAD --name-only` : Show all committed files on current branch
* `git restore --staged .` : Unstage changes / Undo a git add
* `git checkout .` : Revert unstaged changes to most recent commit
* `git reset --hard HEAD~1` : Revert to the commit before the most recent commit


## Vim

* `/\C` : case sensitive search (lowercase c to ignore case)
* `:%y+` : copy current file to clipboard
* `ctrl-w q` : close split window
* `ctrl-w o` or `:only` : close all other split windows
* `:source ~/.vimrc` : to reload your vimrc file
* `1 ctrl-g` : Show the name of the current file with full path
* `:bd` or `:bw` : close file without quitting vim

## Windows

* `super-x u s` : sleep the computer without a mouse

### Chocolatey

* `choco list -l` : list all locally installed packages
* `choco outdated` : list outdated packages

### Notepad++

- `CTRL + L`    delete line and copy to clipboard
- `CTRL + SHIFT + L`    delete line but don't copy to clipboard
- `SHIFT + TAB`     Un-indent (works for highlighted blocks too)
- `CTRL + D`    duplicate line on next line
