---
title:  "Vim Cheat Sheet"
date:   2014-04-07 19:07:00 +0100
---

## Movement

- `h` — Move cursor left  
- `j` — Move cursor down  
- `k` — Move cursor up  
- `l` — Move cursor right  
- `H` — Move to top of screen  
- `M` — Move to middle of screen  
- `L` — Move to bottom of screen  
- `w` — Jump forwards to the start of a word  
- `W` — Jump forwards to the start of a word (words can contain punctuation)  
- `e` — Jump forwards to the end of a word  
- `E` — Jump forwards to the end of a word (words can contain punctuation)  
- `b` — Jump backwards to the start of a word  
- `B` — Jump backwards to the start of a word (words can contain punctuation)  
- `ge` — Jump backwards to the end of a word  
- `gE` — Jump backwards to the end of a word (words can contain punctuation)  
- `%` — Move cursor to matching character (`()`, `{}`, `[]`)  
- `0` — Jump to the start of the line  
- `^` — Jump to the first non-blank character of the line  
- `$` — Jump to the end of the line  
- `g_` — Jump to the last non-blank character of the line  
- `gg` — Go to the first line of the document  
- `G` — Go to the last line of the document  
- `gd` — Move to local declaration  
- `gD` — Move to global declaration  
- `fx` — Jump to next occurrence of character x  
- `tx` — Jump to before next occurrence of character x  
- `Fx` — Jump to the previous occurrence of character x  
- `Tx` — Jump to after previous occurrence of character x  
- `zz` — Center cursor on screen
- `Ctrl+d` — Move cursor and screen down 1/2 page  
- `Ctrl+u` — Move cursor and screen up 1/2 page  

## Insert Mode

- `i` — Insert before the cursor  
- `I` — Insert at the beginning of the line  
- `a` — Insert (append) after the cursor  
- `A` — Insert (append) at the end of the line  
- `o` — Open a new line below the current line  
- `O` — Open a new line above the current line  
- `ea` — Insert (append) at the end of the word  
- `Ctrl+t` — Indent (move right) line one shiftwidth  
- `Ctrl+d` — De-indent (move left) line one shiftwidth  
- `Ctrl+n` — Auto-complete next match  
- `Ctrl+p` — Auto-complete previous match  

## Edit

- `r` — Replace a single character  
- `R` — Replace more than one character, until ESC is pressed  
- `J` — Join line below to the current one with one space in between  
- `gJ` — Join line below to the current one without space in between  
- `gwip` — Reflow paragraph  
- `g~` — Switch case up to motion  
- `gu` — Change to lowercase up to motion  
- `gU` — Change to uppercase up to motion  
- `cc` — Change (replace) entire line  
- `c$` or `C` — Change (replace) to the end of the line  
- `ciw` — Change (replace) entire word  
- `cw` or `ce` — Change (replace) to the end of the word  
- `s` — Delete character and substitute text (same as `cl`)  
- `S` — Delete line and substitute text (same as `cc`)  
- `xp` — Transpose two letters (delete and paste)  
- `u` — Undo  
- `U` — Restore (undo) last changed line  
- `Ctrl+r` — Redo  
- `.` — Repeat last command  

## Indent

- `>>` — Indent (move right) line one shiftwidth  
- `<<` — De-indent (move left) line one shiftwidth  
- `>%` — Indent a block with `()` or `{}` (cursor on brace)  
- `<%` — De-indent a block with `()` or `{}` (cursor on brace)  
- `>ib` — Indent inner block with `()`  
- `>at` — Indent a block with `<>` tags  
- `3==` — Re-indent 3 lines  
- `=%` — Re-indent a block with `()` or `{}` (cursor on brace)  
- `=iB` — Re-indent inner block with `{}`  
- `gg=G` — Re-indent entire buffer  
- `]p` — Paste and adjust indent to current line  

## Visual Mode

- `v` — Start visual mode, mark lines, then do a command (like yank)  
- `V` — Start linewise visual mode  
- `o` — Move to other end of marked area  
- `Ctrl+v` — Start visual block mode  
- `O` — Move to other corner of block  
- `aw` — Mark a word  
- `ab` — A block with `()`  
- `aB` — A block with `{}`  
- `at` — A block with `<>` tags  
- `ib` — Inner block with `()`  
- `iB` — Inner block with `{}`  
- `it` — Inner block with `<>` tags  

## Visual Commands

- `>` — Shift text right  
- `<` — Shift text left  
- `y` — Yank (copy) marked text  
- `d` — Delete marked text  
- `~` — Switch case  
- `u` — Change marked text to lowercase  
- `U` — Change marked text to uppercase  

## Cut and Paste

- `yy` — Yank (copy) a line  
- `2yy` — Yank (copy) 2 lines  
- `yw` — Yank (copy) the characters of the word from the cursor position to the start of the next word  
- `yiw` — Yank (copy) word under the cursor  
- `yaw` — Yank (copy) word under the cursor and the space after or before it  
- `y$` or `Y` — Yank (copy) to end of line  
- `p` — Paste after cursor  
- `P` — Paste before cursor  
- `gp` — Paste after cursor and leave cursor after the new text  
- `gP` — Paste before cursor and leave cursor after the new text  
- `dd` — Delete (cut) a line  
- `2dd` — Delete (cut) 2 lines  
- `dw` — Delete (cut) the characters of the word from the cursor position to the start of the next word  
- `diw` — Delete (cut) word under the cursor  
- `daw` — Delete (cut) word under the cursor and the space after or before it  
- `:3,5d` — Delete lines starting from 3 to 5  
- `:g/{pattern}/d` — Delete all lines containing pattern  
- `:g!/{pattern}/d` — Delete all lines not containing pattern  
- `d$` or `D` — Delete (cut) to the end of the line  
- `x` — Delete (cut) character  

## Registers

- `:reg[isters]` — Show registers content  
- `"xy` — Yank into register x  
- `"xp` — Paste contents of register x  
- `"+y` — Yank into the system clipboard register  
- `"+p` — Paste from the system clipboard register  

**Special registers:**  
- `0` — Last yank  
- `"` — Unnamed register, last delete or yank  
- `%` — Current file name  
- `#` — Alternate file name  
- `*` — Clipboard contents (X11 primary)  
- `+` — Clipboard contents (X11 clipboard)  
- `/` — Last search pattern  
- `:` — Last command-line  
- `.` — Last inserted text  
- `-` — Last small (less than a line) delete  
- `=` — Expression register  
- `_` — Black hole register  

## Search and Replace

- `/pattern` — Search for pattern  
- `?pattern` — Search backward for pattern  
- `\vpattern` — 'Very magic' pattern: non-alphanumeric characters are interpreted as special regex symbols (no escaping needed)  
- `n` — Repeat search in same direction  
- `N` — Repeat search in opposite direction  
- `:%s/old/new/g` — Replace all old with new throughout file  
- `:%s/old/new/gc` — Replace all old with new throughout file with confirmations  
- `:noh[lsearch]` — Remove highlighting of search matches  

## Search in Multiple Files

- `:vim[grep] /pattern/ {file}` — Search for pattern in multiple files  
  - Example: `:vim[grep] /foo/ **/*`  
- `:cn[ext]` — Jump to the next match  
- `:cp[revious]` — Jump to the previous match  
- `:cope[n]` — Open a window containing the list of matches  
- `:ccl[ose]` — Close the quickfix window  

## Working with Multiple Files

- `:e[dit] file` — Edit a file in a new buffer  
- `:bn[ext]` — Go to the next buffer  
- `:bp[revious]` — Go to the previous buffer  
- `:bd[elete]` — Delete a buffer (close a file)  
- `:b[uffer]#` — Go to a buffer by index #  
- `:b[uffer] file` — Go to a buffer by file  
- `:ls` or `:buffers` — List all open buffers  
- `:sp[lit] file` — Open a file in a new buffer and split window  
- `:vs[plit] file` — Open a file in a new buffer and vertically split window  
- `:vert[ical] ba[ll]` — Edit all buffers as vertical windows  
- `:tab ba[ll]` — Edit all buffers as tabs  
- `Ctrl+ws` — Split window  
- `Ctrl+wv` — Split window vertically  
- `Ctrl+ww` — Switch windows  
- `Ctrl+wq` — Quit a window  
- `Ctrl+wx` — Exchange current window with next one  
- `Ctrl+w=` — Make all windows equal height & width  
- `Ctrl+wh` — Move cursor to the left window (vertical split)  
- `Ctrl+wl` — Move cursor to the right window (vertical split)  
- `Ctrl+wj` — Move cursor to the window below (horizontal split)  
- `Ctrl+wk` — Move cursor to the window above (horizontal split)  
- `Ctrl+wH` — Make current window full height at far left  
- `Ctrl+wL` — Make current window full height at far right  
- `Ctrl+wJ` — Make current window full width at the very bottom  
- `Ctrl+wK` — Make current window full width at the very top  

## Exit

- `:w` — Write (save) the file, but don't exit  
- `:w !sudo tee %` — Write out the current file using sudo  
- `:wq` or `:x` or `ZZ` — Write (save) and quit  
- `:q` — Quit (fails if there are unsaved changes)  
- `:q!` or `ZQ` — Quit and throw away unsaved changes  
- `:wqa` — Write (save) and quit on all tabs  

## Misc

- `:h[elp] keyword` — Open help for keyword  
- `:sav[eas] file` — Save file as  
- `:clo[se]` — Close current pane  
- `:ter[minal]` — Open a terminal window  
- `K` — Open man page for word under the cursor  
