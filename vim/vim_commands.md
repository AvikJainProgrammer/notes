# Vim Commands Cheat Sheet

## Modes
| Command | Action |
|---|---|
| `Esc` | Return to Normal mode |
| `i` | Insert before cursor |
| `I` | Insert at start of line |
| `a` | Insert after cursor |
| `A` | Insert at end of line |
| `o` | Open new line below, insert |
| `O` | Open new line above, insert |
| `v` | Visual mode (character) |
| `V` | Visual mode (line) |
| `Ctrl-v` | Visual mode (block) |
| `R` | Replace mode |
| `:` | Command-line mode |

## Movement
| Command | Action |
|---|---|
| `h` `j` `k` `l` | Left, down, up, right |
| `w` | Next word start |
| `b` | Previous word start |
| `e` | Next word end |
| `0` | Start of line |
| `^` | First non-blank char of line |
| `$` | End of line |
| `gg` | Go to first line |
| `G` | Go to last line |
| `:n` | Go to line n |
| `Ctrl-f` | Page down |
| `Ctrl-b` | Page up |
| `Ctrl-d` | Half page down |
| `Ctrl-u` | Half page up |
| `%` | Jump to matching bracket |
| `{` / `}` | Jump to previous/next paragraph |
| `H` / `M` / `L` | Top/middle/bottom of screen |

## Editing
| Command | Action |
|---|---|
| `x` | Delete character under cursor |
| `X` | Delete character before cursor |
| `dd` | Delete (cut) line |
| `dw` | Delete word |
| `d$` | Delete to end of line |
| `d0` | Delete to start of line |
| `D` | Delete to end of line (same as `d$`) |
| `yy` | Yank (copy) line |
| `yw` | Yank word |
| `p` | Paste after cursor/line |
| `P` | Paste before cursor/line |
| `u` | Undo |
| `Ctrl-r` | Redo |
| `.` | Repeat last change |
| `r<char>` | Replace single character |
| `cc` | Change entire line |
| `cw` | Change word |
| `J` | Join line with next line |
| `~` | Toggle case of character |
| `>>` / `<<` | Indent / unindent line |

## Search & Replace
| Command | Action |
|---|---|
| `/pattern` | Search forward |
| `?pattern` | Search backward |
| `n` | Repeat search (same direction) |
| `N` | Repeat search (opposite direction) |
| `*` | Search word under cursor forward |
| `#` | Search word under cursor backward |
| `:%s/old/new/g` | Replace all occurrences in file |
| `:%s/old/new/gc` | Replace all with confirmation |
| `:s/old/new/g` | Replace all occurrences in current line |

## Visual Mode
| Command | Action |
|---|---|
| `v` then move, then `d` | Select and delete |
| `v` then move, then `y` | Select and yank |
| `v` then move, then `>` | Select and indent |
| `gv` | Reselect last visual selection |

## Files & Buffers
| Command | Action |
|---|---|
| `:w` | Save |
| `:w filename` | Save as filename |
| `:q` | Quit |
| `:q!` | Quit without saving |
| `:wq` / `:x` | Save and quit |
| `:e filename` | Open file |
| `:bn` / `:bp` | Next / previous buffer |
| `:ls` | List open buffers |

## Windows & Tabs
| Command | Action |
|---|---|
| `:split` / `:sp` | Horizontal split |
| `:vsplit` / `:vsp` | Vertical split |
| `Ctrl-w w` | Switch between windows |
| `Ctrl-w q` | Close window |
| `:tabnew` | New tab |
| `gt` / `gT` | Next / previous tab |

## Marks & Registers
| Command | Action |
|---|---|
| `m<letter>` | Set mark |
| `` `<letter> `` | Jump to mark |
| `"<letter>y` | Yank into named register |
| `"<letter>p` | Paste from named register |

## Misc
| Command | Action |
|---|---|
| `:set nu` | Show line numbers |
| `ZZ` | Save and quit (like `:wq`) |
| `Ctrl-o` / `Ctrl-i` | Jump to older/newer position |
| `:noh` | Clear search highlight |
