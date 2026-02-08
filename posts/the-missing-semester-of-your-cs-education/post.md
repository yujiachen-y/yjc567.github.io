

> [!note] 💡 学习完这门课程后，我也有了自己的 dotfile repo：[https://github.com/yujiachen-y/dotfiles](https://github.com/yujiachen-y/dotfiles)

知道的前提是知道自己不知道什么。

[The Missing Semester of Your CS Education](https://missing.csail.mit.edu/)

# Shell

- Connecting programs
    - `< file` and `> file` rewire the input and output streams of a program to a file.
    - `>>` to append to a file.
    - `|` chain programs by the output of one is the input of another.
- `sysfs` exposes a number of kernel parameters as files.
- Conditionally execute commands, `&&` and `||`
- Separate commands within the same line `;`
- Get the output of a command as a variable.
    - `$( CMD )` , directly replace the output in the command.
    - `<( CMD )` , execute `CMD` and place the output in a temporary file and substitute the `<()` with that file’s name.
- Shell globbing
    - `?` and `*` to match one or any amount of characters.
    - `{}` expand a series of commands
        - e.g. `convert image.{png,jpg}`

[Special Characters](https://tldp.org/LDP/abs/html/special-chars.html)

[GitHub - koalaman/shellcheck: ShellCheck, a static analysis tool for shell scripts](https://github.com/koalaman/shellcheck)

## Tools

[GNU coreutils](https://en.wikipedia.org/wiki/List_of_GNU_Core_Utilities_commands) via [Homebrew](https://formulae.brew.sh/formula/coreutils)

- `xargs`
- `man` replacement: [`TLDR`](https://tldr.sh/)
- Finding files [`fd`](https://github.com/sharkdp/fd)
- Finding code `grep` [`ack`](https://github.com/beyondgrep/ack3) [`ag`](https://github.com/ggreer/the_silver_searcher) [`rg`](https://github.com/BurntSushi/ripgrep)
- Shell finding commands [`fzf`](https://github.com/junegunn/fzf/wiki/Configuring-shell-key-bindings#ctrl-r)
- Directory Navigation [`fasd`](https://github.com/clvv/fasd) [`autojump`](https://github.com/wting/autojump) [`frecency`](https://web.archive.org/web/20210421120120/https://developer.mozilla.org/en-US/docs/Mozilla/Tech/Places/Frecency_algorithm)
- View directory structure [`tree`](https://linux.die.net/man/1/tree) [`broot`](https://github.com/Canop/broot) [`nnn`](https://github.com/jarun/nnn) [`ranger`](https://github.com/ranger/ranger)
- Stream editor `sed` , a useful programming language `awk` (人生苦短，我用Python
- [`rsync`](https://www.man7.org/linux/man-pages/man1/rsync.1.html)

## Job Control

- `kill`
- Stop a process `Ctrl-Z`
- Continue the paused job in the foreground or in the background using `fg` or `bg`
- `jobs`
- `tmux`

## Aliases

Note that there is no space around the equal sign `=`, because [`alias`](https://www.man7.org/linux/man-pages/man1/alias.1p.html)  is a shell command that takes a single argument.

## Dotfiles

[GitHub does dotfiles - dotfiles.github.io](https://dotfiles.github.io/)

[Use Git to manage secret content](https://abdullah.today/encrypted-dotfiles/) via creating bash functions to encrypt / decrypt secret content into / from Git.

### Portability

```bash
if [[ "$(uname)" == "Linux" ]]; then {do_something}; fi

# Check before using shell-specific features
if [[ "$SHELL" == "zsh" ]]; then {do_something}; fi

# You can also make it machine-specific
if [[ "$(hostname)" == "myServer" ]]; then {do_something}; fi

# Test if ~/.aliases exists and source it
if [ -f ~/.aliases ]; then
    source ~/.aliases
fi
```

gitconfig supports including configurations from local files.

```
[include]
    path = ~/.gitconfig_local
```

## SSH Configuration

- generate a key pair → `ssh-keygen`
- manage passphrase → `ssh-agent` `gpg-agent`

[Connecting to GitHub with SSH - GitHub Docs](https://help.github.com/articles/connecting-to-github-with-ssh/)

[What's ssh port forwarding and what's the difference between ssh local and remote port forwarding](https://unix.stackexchange.com/questions/115897/whats-ssh-port-forwarding-and-whats-the-difference-between-ssh-local-and-remot)

`~/.ssh/config`

```
Host vm
    User foobar
    HostName 172.16.174.141
    Port 2222
    IdentityFile ~/.ssh/id_ed25519
    LocalForward 9999 localhost:8888

# Configs can also take wildcards
Host *.mit.edu
    User foobaz
```

### Key based authentication

```bash
cat .ssh/id_ed25519.pub | ssh foobar@remote 'cat >> ~/.ssh/authorized_keys'

# Or
ssh-copy-id -i .ssh/id_ed25519 foobar@remote
```

### Copying files over SSH

`rsync`

## ZSH

为什么不使用fish？

因为我对shell不够了解，fish不支持Posix标准，我觉得有很多我不知道的坑。

# Editors (Vim)

Know keyboard shortcuts about your most recently used editor!d

[](https://missing.csail.mit.edu/2020/files/vimrc)

[](https://vimawesome.com/)

[The big list of Vim-like software](https://reversed.top/2016-08-13/big-list-of-vim-like-software)

# Keyboard remapping

[Karabiner-Elements](https://pqrs.org/osx/karabiner/)

[GitHub - koekeishiya/skhd: Simple hotkey daemon for macOS](https://github.com/koekeishiya/skhd)

[folivora.ai - Great Tools for your Mac!](https://folivora.ai/)

# Daemon & Automation

`systemd`

```bash
# /etc/systemd/system/myapp.service
[Unit]
Description=My Custom App
After=network.target

[Service]
User=foo
Group=foo
WorkingDirectory=/home/foo/projects/mydaemon
ExecStart=/usr/bin/local/python3.7 app.py
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

`cron`

[Hammerspoon](https://www.hammerspoon.org/)

# FUSE

filesystem in user space，对文件系统的中间抽象，用来加上远程、分布式、加密等功能。

- [sshfs](https://github.com/libfuse/sshfs) - Open locally remote files/folder through an SSH connection.
- [rclone](https://rclone.org/commands/rclone_mount/) - Mount cloud storage services like Dropbox, GDrive, Amazon S3 or Google Cloud Storage and open data locally.
- [gocryptfs](https://nuetzlich.net/gocryptfs/) - Encrypted overlay system. Files are stored encrypted but once the FS is mounted they appear as plaintext in the mountpoint.
- [kbfs](https://keybase.io/docs/kbfs) - Distributed filesystem with end-to-end encryption. You can have private, shared and public folders.
- [borgbackup](https://borgbackup.readthedocs.io/en/stable/usage/mount.html) - Mount your deduplicated, compressed and encrypted backups for ease of browsing.

# Backups

[https://www.cisa.gov/uscert/sites/default/files/publications/data_backup_options.pdf](https://www.cisa.gov/uscert/sites/default/files/publications/data_backup_options.pdf)


# Filesystem Hierarchy Standard

- `/bin` - Essential command binaries
- `/sbin` - Essential system binaries, usually to be run by root
- `/dev` - Device files, special files that often are interfaces to hardware devices
- `/etc` - Host-specific system-wide configuration files
- `/home` - Home directories for users in the system
- `/lib` - Common libraries for system programs
- `/opt` - Optional application software
- `/sys` - Contains information and configuration for the system
- `/tmp` - Temporary files (also `/var/tmp`). Usually deleted between reboots.
- `/usr/` - Read only user data
    - `/usr/bin` - Non-essential command binaries
    - `/usr/sbin` - Non-essential system binaries, usually to be run by root
    - `/usr/local/bin` - Binaries for user compiled programs
- `/var` - Variable files like logs or caches
