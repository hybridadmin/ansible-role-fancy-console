## Fancy-Console Role

[![Release](https://github.com/hybridadmin/ansible-role-fancy-console/actions/workflows/release.yml/badge.svg)](https://github.com/hybridadmin/ansible-role-fancy-console/actions/workflows/release.yml)
![Ansible Role](https://img.shields.io/ansible/role/d/12641)
![Ansible Quality Score](https://img.shields.io/ansible/quality/12641)

> [!NOTE] Tested on Debian 11, Debian 12, Ubuntu 18.04, Ubuntu 20.04, Ubuntu 22.04, Ubuntu 24.04, macOS Ventura,  macOS Sonoma, CentOS 8, Fedora 39, Fedora 40, Amazon Linux 2.


## Includes:

- [`zsh`](http://zsh.sourceforge.net)
- [`antigen`](https://github.com/zsh-users/antigen) or [`antidote`](https://antidote.sh/)
- [`oh-my-zsh`](https://github.com/robbyrussell/oh-my-zsh)
- [`oh-my-posh`](https://ohmyposh.dev/) or [`powerline-go`](https://github.com/justjanne/powerline-go)
- [`zsh-autosuggestions`](https://github.com/zsh-users/zsh-autosuggestions)
- [`zsh-syntax-highlighting`](https://github.com/zsh-users/zsh-syntax-highlighting)
- [`unixorn/autoupdate-antigen.zsh plugin`](https://github.com/unixorn/autoupdate-antigen.zshplugin)
- [`urbainvaes/fzf-marks`](https://github.com/popstas/urbainvaes/fzf-marks)

## Features

- default colors tested with solarized dark and default grey terminal in putty
- add custom prompt elements from yml
- custom zsh config with `~/.zshrc.local` or `/etc/zshrc.local`
- load `/etc/profile.d` scripts
- install only plugins that useful for your machine. For example, plugin `docker` will not install if you have not Docker

## screen capture

![screen capture](./console.png?raw=true)

## Known bugs

>N/A

## Install for real machine

### Zero-knowledge install:

If you using Ubuntu or Debian and not familiar with Ansible, you can just execute [`install.sh`](install.sh) on target machine:

```
curl https://raw.githubusercontent.com/hybridadmin/ansible-role-zsh/master/install.sh | bash
```

It will install zsh for root and current user.
Then [`configure terminal application`](#configure-terminal-application).

### Manual install

[`Install Ansible`](https://docs.ansible.com/ansible/latest/installation_guide/).

For Ubuntu:

```bash
sudo apt update
sudo apt install python3-pip -y
sudo pip3 install ansible
```

For CentOS:

```bash
yum install epel-release
yum install ansible
```

1. Download role:

```bash
sudo ansible-galaxy install hybridadmin.fancy_console --force
```

2. Write playbook or use [`playbook.yml`](playbook.yml):

```yaml
- hosts: all
  vars:
    prompt_theme_engine: powerline
    zsh_antigen_bundles_extras:
      - nvm
      - joel-porquet/zsh-dircolors-solarized
      - MichaelAquilina/zsh-you-should-use
      - oldratlee/hacker-quotes
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

4. Install fzf **without shell extensions**, [`download binary`](https://github.com/junegunn/fzf/releases)
   or `brew install fzf` for macOS.

Note: I don't use `tmux-fzf` and don't tested work of it.

## Multiuser shared install

If you have 10+ users on host, probably you don't want manage tens of configurations and thousands of files.

In this case you can deploy single zsh config and include it to all users.

It causes some limitations:

- Users have read only access to zsh config
- Users cannot disable global enabled bundles
- Possible bugs such cache write permission denied
- Possible bugs with oh-my-zsh themes

For install shared configuration you should set `zsh_shared: yes`.
Configuration will install to `/usr/share/zsh-config`, then you just can include to user config:

```bash
source /usr/share/zsh-config/.zshrc
```

You can still provision custom configs for several users.

## Configure

You should not edit `~/.zshrc`!
Add your custom config to `~/.zshrc.local` (per user) or `/etc/zshrc.local` (global).
`.zshrc.local` will never touched by ansible.

### Configure terminal application

1. Download [`powerline fonts`](https://github.com/powerline/fonts), install font that you prefer.
   You can see screenshots [`here`](https://github.com/powerline/fonts/blob/master/samples/All.md).

2. Set color scheme.

Personaly, I prefer Solarized Dark color sceme, Droid Sans Mono for Powerline in iTerm and DejaVu Sans Mono in Putty.

#### iTerm

Profiles - Text - Change Font - select font "for Powerline"

Profiles - Colors - Color Presets... - select Solarized Dark

#### Putty

Settings - Window - Appearance - Font settings

You can download [`Solarized Dark for Putty`](https://github.com/altercation/solarized/tree/master/putty-colors-solarized).

#### Gnome Terminal

gnome-terminal have built-in Solarized Dark, note that you should select both background color scheme and palette scheme.

### Hotkeys

> N/A

### Aliases

You can use aliases for your command with easy deploy.
Aliases config mostly same as hotkeys config:

```yaml
zsh_aliases:
  - { alias: 'dfh', action: 'df -h | grep -v docker' }
# with dependency of bundle and without replace default asiases
- zsh_aliases_extra
  - { alias: 'dfh', action: 'df -h | grep -v docker', bundle: }
```

## Configure bundles

You can check default bundles in [`defaults/main.yml`](defaults/main.yml#L37).
If you like default bundles, but you want to add your bundles, use `zsh_antigen_bundles_extras` variable for `antigen` or `zsh_antidote_bundles_extras` variable for `antidote` (see example playbook above).
If you want to remove some default bundles, you should use `zsh_antigen_bundles` variable for `antigen` or `zsh_antidote_bundles` for antidote.

Format of list matches [`antigen`](https://github.com/zsh-users/antigen#antigen-bundle) or [`antidote`](https://antidote.sh/usage). All bellow variants valid:

```yaml
- docker # oh-my-zsh plugin
- zsh-users/zsh-autosuggestions # plugin from github
- zsh-users/zsh-autosuggestions@v0.3.3 # plugin from github with fixed version
- ~/projects/zsh/my-plugin --no-local-clone # plugin from local directory
```

Note that bundles can use conditions for load. There are two types of conditions:

1. Command conditions. Just add `command` to bundle:

```yaml
- { name: docker, command: docker }
- name: docker-compose
  command: docker-compose
```

Bundles `docker` and `docker-compose` will be added to config only if commands exists on target system.

2. When conditions. You can define any ansible conditions as you define in `when` in tasks:

```yaml
# load only for zsh >= 4.3.17
- name: zsh-users/zsh-syntax-highlighting
  when: "{{ zsh_version is version('4.3.17', '>=') }}"
# load only for macOS
- { name: brew, when: "{{ ansible_os_family != 'Darwin' }}" }
```

Note: you should wrap condition in `"{{ }}"`
