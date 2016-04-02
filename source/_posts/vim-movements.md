---
title: "Note on Vim Movements"
date: 2016-04-02 16:13:44
tags:
- Vim
---

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
