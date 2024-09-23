---
title: "Vim (and Neovim)"
tags: [
  "computing",
  "coding",
  "IDE",
  "programming",
  "CLI",
  "linux",
]
---

## Commands

### Edit Remote File Using SSH with Local vim Setup (Settings and Plugins)

```vim scp://user@myserver[:port]//path/to/file.txt```

### Plug

#### Install Plug Plugins

```:PlugInstall```

#### Remove Unlisted Plug Plugins

```:PlugClean```

#### Update Plug Plugins

```:PlugUpdate```

#### Upgrade Plug Itself

```:PlugUpgrade```

### Vundle

#### Install Vundle Plugins

```:PluginInstall``` (in vim) or ```vim +PluginInstall +qall``` (in command line)
