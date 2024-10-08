---
title: Installation ZSH & OhMyZSH
author :
    name : Mathieu BESSON
    linkedin : mathieubesson
---

# Installation de ZSH et OhMyZSH sous Linux

## Prérequis

- Une distribution linux

## Nécéssaires avant installation  

- Mise à jour des repositories apt
```shell
sudo apt update && sudo apt upgrade
```

## Installation

- Installation de ZSH
```shell
sudo apt install zsh
```

- Changement de shell par défaut
```shell
sudo chsh -s /bin/zsh
```

- Installation de OhMyZSH
```shell
sh -c "$(wget https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)"
```

- Installation des plugins
```shell
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
git clone https://github.com/zsh-users/zsh-completions ${ZSH_CUSTOM:-${ZSH:-~/.oh-my-zsh}/custom}/plugins/zsh-completions
```

- Modification des configurations .zshrc : thème [agnoster](https://github.com/agnoster/agnoster-zsh-theme) et ajout des plugins

=== "/home/user/.zshrc"
```shell
ZSH_THEME="agnoster"

plugins=(last-working-dir git zsh-autosuggestions zsh-syntax-highlighting zsh-completions)
```

- Installer la font du thème
```shell
sudo apt-get install fonts-powerline
```

- Activation de la modification des configurations
```shell
source ~/.zshrc
```