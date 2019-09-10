---
title: Set Solarized Color Scheme in Ubuntu
date: 2016-06-27 09:24:59
tags:
- Linux
---

I always want to find a fancy color scheme for my terminal. Then I settled down with Solarized.

<!--- more --->

## Install Solarized Colorscheme
To install solarized in Ubuntu, first make sure you have git installed.

```bash
sudo apt-get install git
```

Then clone the Solarized repository from github:

```bash
git clone https://github.com/sigurdga/gnome-terminal-colors-solarized.git
```

And install it:

```bash
cd gnome-terminal-colors-solarized
./install
```

## Fix `ls` dircolor
The output of `ls` set by this colorscheme looks terrible. We need to fix this issue:

```bash
wget --no-check-certificate https://raw.github.com/seebi/dircolors-solarized/master/dircolors.ansi-dark
mv dircolors.ansi-dark ~/.dircolors
eval dircolors ~/.dircolors
```

`eval dircolors ~/.dircolors` may need  to be added to your bashrc or zshrc file.