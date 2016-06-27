---
title: "Note on Vim Editing"
date: 2016-04-02 16:22:14
tags:
- Vim
---

**Insert** - `i` and `I` can bring you into the insert mode. Insert **before** the current character by hitting `i`. Insert at the beginning of the line by hitting `I`, which equals to `^i`.

**Insert with a New Line** - Use `o`(`O`) to insert with a **new line** after(before) the current.

**Insert with Append** - Insert **after** the current character by hitting `a`. Insert at the end of the line by hitting `A`, which equals to `$a`.

**Replacing Characters** - Replace single character under cursor by hitting `r`. Switch to **replace mode** by hitting `R`.

**Changing Things** - Change things by `c + <text object>` or `c + <motion>`. `cc` to change the whole line and `C` change from the current character to the end of the line, which equals to `^c$` and `c$`.

**Deleting Characters** - Delete a single character under the cursor with `x` and before the cursor with `X`.

**Deleting Things** - Delete things with `d + <text object>` or `d + <motion>`. Delete a single line with `dd` and delete to the end of the line with `D`.

**Repeat** - Repeat the last command by hitting `.`.

**Yanking** - Yanking means "copying". Yank(copy) with `y + <text object>`. Yank the whole line with `yy`.

**Putting** - Putting means "pasting". Once you've yanked, you can put with the `p` (put after) or `P` (put before).

**Joining** - Join lines with the `J`. But if you don't want the extra space, you need to use `gJ`.

**Visual Mode** - Use the `v` key for character-wise visual selection, `V` for line-wise selection and `ctrl-v` for block-mode selection. Use `gv` to help you re-select an area you just selected.
