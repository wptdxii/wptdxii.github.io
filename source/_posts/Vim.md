---
title: Vim
date: 2018-02-10 14:10:57
tags: Keymap
categories: Vim
---

Vim 常用快捷键

<!-- more -->

|Description|Keymap|
|:-|:-|
|Vim tutorial|vimtutor|

# Normal Mode

|Description|Keymap|
|:-|:-|
|Normal mode|Esc / (Ctrl + [) / (Ctrl + c)|
|Left/Up/Down/Right|h / j / k / l|
|Move to the start/end of the line|0 / $|
|Join lines|[count] + J|
|Jump to line|[count] + G|
|Jump to next/previous word|w/b|
|Yank the current line|yy/Y|
|Yank to the end of the current line|y$|
|Yank the current word (excluding surrounding whitespace)|yiw|
|Yank the current word (including surrounding whitespace)|yaw|
|Yank from the current cursor position up to and before the character (til x)|yt[x]|
|Yank from the current cursor position up to and including the character (find x)|yf[x]|
|Paste after/before the cursor|p/P|
|Yank  to system register|"+{motion}|
|Paste from system register|"+p|

> To copy into a register, one can use "{register} immediately before one of the above commands to copy into the register {register}.

# Insert Mode

|Description|Keymap|
|:-|:-|
|Insert normal mode|[Ctrl + o|
|Pasting in insert mode from default register| Ctrl + r , "|
|Pasting in insert mode from system register| Ctrl + r , +|

# Visual Mode

|Description|Keymap|
|:-|:-|
|Select a line|Shfit + v|

# Help

|Description|Keymap|
|:-|:-|
|Getting help|(:help Enter)/(F1)|
|Jump between windows|Ctrl + WW|
|Close help window|:q Enter|

# Ref

* [A vim Tutorial and Primer](https://danielmiessler.com/study/vim/#why)