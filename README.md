# gitwatch

A bash script to watch a file or folder and commit changes to a git repo

## What to use it for?
That's really up to you, but here are some examples:
* **config files**: some programs auto-write their config files, without waiting for you to click an 'Apply' button; or even if there is such a button, most programs offer you no way of going  back to an earlier version of your settings. If you commit your config file(s) to a git repo, you can track changes and go back to older versions. This script makes it convenient, to have all changes recorded automatically.
* **document files**: if you use an editor that does not have built-in git support (or maybe if you don't like the git support it has), you can use gitwatch to automatically commit your files when you save them, or combine it with the editor's auto-save feature to fully automatically and regularly track your changes
* *more stuff!* If you have any other uses, or can think of ones, please let us know, and we can add them to this list!

## Installation
`gitwatch` can be installed in various ways.

### From Source
`gitwatch` can be installed from source by simply cloning the repository
and putting the shell script into your `$PATH`. The commands below will
do that for you if `/usr/local/bin` is in your `$PATH`. You may need to
invoke `install` with `sudo`.

```sh
$ git clone https://github.com/gitwatch/gitwatch.git
$ cd gitwatch
$ [sudo] install -b gitwatch.sh /usr/local/bin/gitwatch
```
#### Update

If you installed `gitwatch` from source, you can update it by following the exact same steps (or `git pull` rather than clone if you kept the repository around).

### bpkg
`gitwatch` can be installed with [bpkg](https://github.com/bpkg/bpkg). Make
sure you have [bpkg](https://github.com/bpkg/bpkg) installed before
running the command below. You may need to invoke `bpkg` with `sudo`
when using the `-g` flag.

```sh
$ [sudo] bpkg install -g gitwatch/gitwatch
```

## Requirements
To run this script, you must have installed and globally available:
* `git` ( [git/git](https://github.com/git/git) | http://www.git-scm.com )
* `inotifywait` (part of **inotify-tools**: [rvoicilas/inotify-tools](https://github.com/rvoicilas/inotify-tools) )

### Notes for Mac
If running on OS X, you'll need to install the following Homebrew tools:

```sh
$ brew install fswatch
$ brew install coreutils
```


## What it does
When you start the script, it prepares some variables and checks if the file [a] or directory [b] given as input really exists.<br />
Then it goes into the main loop (which will run forever, until the script is forcefully stopped/killed), which will:
* watch for changes to the file/directory using `inotifywait` (`inotifywait` will block until something happens)
* wait 2 seconds
* `cd` into the directory [b] / the directory containing the file [a] \(because `git` likes to operate locally)
* `git add <file>`[a] / `git add .`[b]
* `git commit -m "Scripted auto-commit on change (<date>)"`[a] / `git commit -a -m"Scripted auto-commit on change (<date>)"`[b]
* if a remote is defined (with `-r`) do a push after the commit (a specific branch can be selected with `-b`)

Notes:
* the waiting period of 2 sec is added to allow for several changes to be written out completely before committing; depending on how fast the script is executed, this might otherwise cause race conditions when watching a folder
* currently, folders are always watched recursively

## Usage
`gitwatch.sh <file or directory to watch> [-r <remote> [-b <branch>]]`<br />
It is expected that the watched file/directory are already in a git repository (the script will not create a repository). If a folder is being watched, this will be watched fully recursively; this also means that all files and sub-folders added and removed from the directory will always be added and removed in the next commit. The `.git` folder will be excluded from the `inotifywait` call so changes to it will not cause unnecessary triggering of the script.

### Starting on Boot

If you want to have the script auto-started upon boot, the method to do this depends on your operating system and distribution. If you have a GUI dialog to set up startup launches, you might want to use that, so you can more easily find and change the startup script calls later on.

Please also note that if either of the paths involved (script or target) contains spaces or special characters, you need to escape them accordingly; if you don't know how to do that, the internet will help you, or feel free to ask here or contact me directly.

#### SysVInit

A central place to put startup scripts on Linux is generally `/etc/rc.local` (to my knowledge; only tested and confirmed on Ubuntu). This file, if it has the +x bit, will be executed upon startup, **by the root user account**. If you want to start `gitwatch` from `rc.local`, the recommended way to call it is:

`su -c "/absolute/path/to/script/gitwatch.sh /absolute/path/to/watched/file/or/folder" -l <username> &`

The `<username>` bit should be replaced with your username or that of any other (non-root) user account; it only needs write-access to the git repository of the file/folder you want to watch. The ampersand (`&`) at the end sends the launched process into the background (this is important if you have other calls in `rc.local` after the mentioned line, because the `gitwatch` call does not usually return).

#### systemd

- If installed to a path other than `/usr/bin/gitwatch`, modify `gitwatch@.service` to suit
- Create dir if it does not exist and copy systemd service file with `mkdir -p "$HOME/.config/systemd/user" && cp gitwatch@.service $HOME/.config/systemd/user`
- Start and enable the service for a given path by running `systemctl --user --now enable gitwatch@$(systemd-escape "'-r url/to/repository' /path/to/folder").service`

## Other Articles
### On the Gitwatch Wiki
* [How to install `gitwatch` as a Debian service with `supervisord`](https://github.com/gitwatch/gitwatch/wiki/gitwatch-as-a-service-on-Debian-with-supervisord)
