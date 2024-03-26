---
tags:
  - 技术
  - 工具
---

# Tmux

由于工作需要经常连接远程服务器，故采用`tmux`作为终端分屏器来提高效率。本文的目的在于理解`tmux`软件背后的逻辑，而不是仅仅将其作为一个工具使用。

## 核心理念

`tmux`的最基本组成单元是session。其中，每个session可以由多个window组成。每个window可以被划分为多个pane。session拥有持久化的功能，适合用于运行长时间的任务。

实际上`tmux`的逻辑和所有的[[窗口管理器]]的逻辑是类似的。

## 工作流

`tmux`的命令相当地复杂且繁琐，难以掌握其所有的命令，主要是形成自己的工作流。

### Session

对于每一个独立的任务采取一个独立的session，也就是一个tmux session对应了一个独立的任务，所以需要合理地对session命名，使其更加的语义化。这样的好处在于我们合理地将任务和session进行了一个映射。

```sh
tmux new-session -s <name>
```

### Window

对于window来说，每个拥有明确功能的window都应该拥有语义化的命名以提高效率。对于的临时的window需要及时处理以保证工作区的合理。

### Pane

对于pane来讲，一般情况并不推荐使用，我始终认为应该尽可能地多使用window。

## 配置文件

```conf
### Some useful tips

## tmux sessions conclusion

# + Create a session in tmux with `tmux new -s MY_SESSION`
# + Detach from a session with `Prefix + d` or `tmux detach`
# + List sessions with `tmux ls`
# + Attach to a session with `tmux attach -t MY_SESSION`
# + List sessions and switch to a different session with `Prefix + s`
# + Kill a session with `tmux kill-session -t MY_SESSION`
# + Close a pane/ window with either `Ctrl + d` or `Prefix + x`

## tmux window conclusion

# + Create a window with `tmux neww -n MY_WINDOW` or `Prefix + c`
# + Switch to a different window with `Prefix + N`, N is the windows number
# + Kill a window with `tmux kill-window -t MY_WINDOW`
# + Rename the window: `Ctrl + d` or `Prefix` + ','

## pane conclusion
# + Switch pane: `Ctrl + d` or `Prefix` + Arrow


set-option -g prefix C-b
set-option -g prefix2 C-Space

# Increase history
set-option -g history-limit 10000

# Jump to a marked pane
# We can use Prefix + m to mark the current you on
# And use Prefix + \` to jump to the mark
# To remove a mark, press Prefix + M anywhere
bind \` switch-client -t'{marked}'

# tmux windows and panes are 0-based. Change them
# so start with 1
set -g base-index 1
setw -g pane-base-index 1

# Enable auto renumber-windows
set -g renumber-windows on

# Split pane like i3wm, although I rarely use it
bind h split-window -hc "#{pane_current_path}"
bind v split-window -vc "#{pane_current_path}"

# Keeping current path
bind c new-window -c "#{pane_current_path}"
```
