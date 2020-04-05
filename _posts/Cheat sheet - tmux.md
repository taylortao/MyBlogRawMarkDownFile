---
title: Cheatsheet - tmux
date: 2020-04-05 11:37:25
categories:
 - IT Technology
tags:
 - Cheatsheet
---

### Pane
- spilt horizontally: `Ctrl-b %`
- split vertically: `Ctrl-b "`
- navigate between panes: `Ctrl-b <arrow key>`
- close: `Ctrl-d`
- make a pane go full screen: `Ctrl-b z` 
- shrink it back to its previous size: `Ctrl-b z`  again
- resize pane in direction:  `Ctrl-b esc-<arrow key>`

### Window
- create new window: `Ctrl-b c`
- navigate:  `Ctrl-b p` / `Ctrl-b n` / `Ctrl-b <number>`
- close: `Ctrl-d`
- rename: `Ctrl-b ,`

<!-- more -->

### Session 
- list: `tmux ls`
- new session with name: `tmux new -s <session_name>`
- rename session: `tmux rename-session -t <old_name> <new_name>`
- detach: `Ctrl-b d` or `Ctrl-b D` (to have tmux give you a choice which of your sessions you want to detach)
- attach: `tmux attach -t <name>`

### Common
- help: `Ctrl-b ?`

#### Search 
- enter copy mode: `Ctrl-b [`
- press `Ctrl-s` and then type keyword

#### Copy-Paste
- enter copy mode: `Ctrl-b [`
- press `Ctrl-space` to start select
- copy: `Ctrl-w` 
- paste: `Ctrl-b ]`
