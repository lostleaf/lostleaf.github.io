---
title: "Notes on Vim basics"
date: 2016-04-02 16:13:44
tags:
- Vim

categories:
- Vim笔记
---

Vim basic usages

<!--- more --->

**Basics** - Character-wise movements with the home keys: `h`, `j`, `k` and `l`.

**Line terminus** - Beginning of line: `^` or `0`(non-blank). End of line: `$`.

**Forward word movement** - word and WORD: `w` and `W`, `e` and `E`.

**Backward word movement** - word and WORD: `b` and `B`, `ge` and `gE`.

**Move to character** - `f`, `F`, `t`, `T`

**Paging** - Full page: `CTRL-f` and `CTRL-b`. Half page: `CTRL-u` and `CTRL-d`.

**Scroll** - One line: `CTRL-y` and `CTRL-e`.

**Cursor jumping** - Head, middle and last line of a screen: `H`, `M` and `L`.

**Top and Bottom** - `gg` and `G`.

**Jumping to a particular line** - `(number)G` or `:(number)`.

**Seach current word** - `*` and `#`, or `g*` and `g#`(no word bounds).

**Regular expression searching** - `/` and `?`. Next and pervious: `n` and `N`.

**Start of Function or Class Jumping** - Beginning of *previous* and *next* functions or classes: `[[` and `]]`

**End of Function or Class Jumping** - *Forwards* and *backwords* to the end of a function or class: `][` and `[]`.

**Jumping to Matching Braces** - The fantastic `%` characters.

**Marks** - Basic mark functionality and how it works with `m`, `'` and <code>\`</code>.

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