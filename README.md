## Fancy-Console Role

[![Release](https://github.com/hybridadmin/ansible-role-fancy-console/actions/workflows/release.yml/badge.svg)](https://github.com/hybridadmin/ansible-role-fancy-console/actions/workflows/release.yml)
![Build CI](https://img.shields.io/github/actions/workflow/status/hybridadmin/ansible-role-fancy-console/build.yml)
![Ansible Role](https://img.shields.io/ansible/role/d/hybridadmin/fancy_console)

> [!NOTE]
Tested on Debian 11, Debian 12, Ubuntu 18.04, Ubuntu 20.04, Ubuntu 22.04, Ubuntu 24.04, macOS Ventura,  macOS Sonoma, macOS Sequoia, CentOS 8, Fedora 39, Fedora 40, Amazon Linux 2.

## Includes:

- [`zsh`](https://zsh.sourceforge.io)
- [`antidote`](https://antidote.sh/)
- [`oh-my-zsh`](https://github.com/robbyrussell/oh-my-zsh)
- [`oh-my-posh`](https://ohmyposh.dev/)
- [`zsh-autosuggestions`](https://github.com/zsh-users/zsh-autosuggestions)
- [`zsh-syntax-highlighting`](https://github.com/zsh-users/zsh-syntax-highlighting)
- [`urbainvaes/fzf-marks`](https://github.com/urbainvaes/fzf-marks)

## Features

- default colors tested with solarized dark
- add custom prompt elements from yml
- custom zsh config with `~/.zshrc.local` or `/etc/zshrc.local`
- load `/etc/profile.d` and/or `$HOME/.config/zsh` scripts
- install only plugins that useful for your machine. For example, plugin `docker` will not install if you have not Docker

## screen capture

![screen capture](./console.png?raw=true)

## Known bugs

>N/A

## Install for real machine

### Zero-knowledge install:

If you are not familiar with Ansible, you can just execute [`install.sh`](install.sh) on target machine:

```
curl https://raw.githubusercontent.com/hybridadmin/ansible-role-fancy-console/main/install.sh | bash
```

It will install zsh for root and current user, then [`configure terminal application`](#configure-terminal-application).

### Manual install

- <https://docs.ansible.com/ansible/latest/installation_guide/>

```bash
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
python3 get-pip.py --user
python3 -m pip install --user ansible-core
```

1. Download role:

```bash
ansible-galaxy install hybridadmin.fancy_console --force
```

2. Write playbook or use [`playbook.yml`](playbook.yml):

```yaml
- hosts: all
  vars:
    zsh_antidote_bundles_extras:
      - { name: "joel-porquet/zsh-dircolors-solarized" }
      - { name: "MichaelAquilina/zsh-you-should-use" }
      - { name: "oldratlee/hacker-quotes" }
    zsh_autosuggestions_bind_key: "^U"
  roles:
    - hybridadmin.fancy_console
```

3. Provision playbook:

```bash
sudo ansible-playbook -i "localhost," -c local playbook.yml
```

If you want to provision role for root user on macOS, you should install packages manually:

```bash
brew install zsh git wget
```

It will install zsh environment for ansible remote user. If you want to setup zsh for other users,
you should define variable `zsh_user`:

Via playbook:

```yaml
- hosts: all
  roles:
    - { role: hybridadmin.fancy_console, zsh_user: otheruser }
    - { role: hybridadmin.fancy_console, zsh_user: thirduser }
```

Or via command:

```bash
ansible-playbook -i hosts zsh.yml --extra-vars="zsh_user=otheruser"
```

4. Install fzf **without shell extensions**, [`download binary`](https://github.com/junegunn/fzf/releases) or `brew install fzf` for macOS.


## Multiuser shared install

If you have 10+ users on host, managing multiple configurations and thousands of files can become a bit of a hastle in which case
 you can deploy a single shared zsh config and include it to all users. There are some limitations which are listed below:

- Users have read only access to zsh config
- Users cannot disable global enabled bundles
- Possible bugs such cache write permission denied
- Possible bugs with oh-my-zsh themes

For shared configuration you should set `zsh_shared: true` and configuration will be installed to `/usr/share/zsh-config`, then you can include to user config using:

```bash
source /usr/share/zsh-config/.zshrc
```

You can still provision custom configs for several users.

## Configure

You should not edit `~/.zshrc`!
Add your custom config to `~/.zshrc.local` (per user) or `/etc/zshrc.local` (global).
`.zshrc.local` will never touched by ansible.

Any dotfiles can be added to `$HOME/.config/zsh` and these will be automatically detected and loaded when `~/.zshrc` is sourced.

### Configure terminal application

1. Download [`powerline fonts`](https://github.com/powerline/fonts), install font that you prefer.
   You can see screenshots [`here`](https://github.com/powerline/fonts/blob/master/samples/All.md).

2. Set color scheme.
   - The [`Argonaut`](https://github.com/pwaleczek/Argonaut-theme) color scheme is preferred with either of the fonts `FuraMono Nerd Font Mono`, `MesloLGS Nerd Font Mono`, `CaskaydiaCove Nerd Font Mono`, `Roboto Mono for Powerline`
tested in iTerm2.

#### iTerm2

- Profiles - Text - Change Font - select font "for Powerline"

- Profiles - Colors - Color Presets... - select Solarized Dark

#### Gnome Terminal

- gnome-terminal has built-in Solarized Dark color scheme, so ensure that you select both background color scheme and palette scheme.

### Hotkeys

> N/A

### Aliases

You can use aliases for your command(s) with easy deploy. Aliases config is as below:

```yaml
zsh_aliases:
  - { alias: 'dfh', action: 'df -h | grep -v docker' }
# with dependency of bundle and without replace default asiases
- zsh_aliases_extra
  - { alias: 'dfh', action: 'df -h | grep -v docker', bundle: }
```

## Configure bundles

You can check default bundles in [`defaults/main.yml`](defaults/main.yml#L33). If you like default bundles, but you want to add your bundles, use `zsh_antidote_bundles_extras` variable for `antidote` (see example playbook above).
If you want to remove some default bundles, you should use `zsh_antidote_bundles` for antidote.

Format of list matches [`antidote`](https://antidote.sh/usage). All bellow variants valid:

```yaml
- { name: "ohmyzsh/ohmyzsh", path: "plugins/fancy-ctrl-z" } # oh-my-zsh plugin
- { name: "zsh-users/zsh-autosuggestions" } # plugin from github
- { name: "zsh-users/zsh-autosuggestions@v0.3.3" } # plugin from github with fixed version
- { name: "popstas/zsh-command-time" }
```

>NB: that bundles can use conditions for loading. There are two types of conditions:

1. Command conditions. Just add `command` to bundle:

```yaml
- { name: docker, command: docker }
- name: docker-compose
  command: docker-compose
```

Bundles `docker` and `docker-compose` will be added to config only if commands exists on target system.

2. When conditions. You can define any ansible conditions as you define in `when` in tasks:

```yaml
# load only for macOS
- { name: "ohmyzsh/ohmyzsh", path: "plugins/brew", conditional: "is-macos" }
- { name: "ohmyzsh/ohmyzsh", path: "plugins/macos", conditional: "is-macos" }
```
