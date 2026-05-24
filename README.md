<h1 align="center">
  <img src="https://raw.githubusercontent.com/neovim/neovim.github.io/master/static/logos/neovim-logo-300x87.png" alt="yanvim">
  <br>
  yanvim
  <br>
  <sub>Yet Another Neovim, we should call it not so Neo-vim anymore</sub>
</h1>

<p align="center">
  <a href="#building"><img src="https://img.shields.io/badge/build-cmake-blue" alt="CMake"></a>
  <a href="https://aur.archlinux.org/packages/yanvim-git"><img src="https://img.shields.io/badge/AUR-yanvim--git-1793d1?logo=archlinux&logoColor=white" alt="AUR"></a>
  <a href="./LICENSE.txt"><img src="https://img.shields.io/badge/license-Apache%202.0-green" alt="License"></a>
  <a href="https://github.com/Rodericuss/yet-another-neovim/tree/stable"><img src="https://img.shields.io/badge/base-neovim%200.12.2-57A143?logo=neovim&logoColor=white" alt="Neovim 0.12.2"></a>
</p>

<p align="center">
  A Neovim fork with a built-in <b>Helix-style selection-first editing paradigm</b>.
  <br>
  You select, <i>then</i> you act. Like a civilized person.
</p>

<p align="center">
  <img src="https://github.com/user-attachments/assets/placeholder" alt="yanvim demo" width="600">
  <br>
  <sub>(demo gif goes here — record one with <code>:set paradigm=helix</code> and show off <code>w w w d</code>)</sub>
</p>

---

## What is this?

A hard fork of [Neovim](https://github.com/neovim/neovim) with one meaningful addition: a built-in Helix editing paradigm, switchable at runtime.

```lua
vim.opt.paradigm = 'vim'    -- default. you already know this one.
vim.opt.paradigm = 'helix'  -- the good one.
```

Set it to `'helix'` and Normal mode becomes selection-first. Every motion selects. Every verb acts on what you selected. No more `d3w` and praying you counted the words right. You press `w` three times, *see* exactly what you're about to obliterate, and *then* press `d`. Revolutionary? No. Helix has been doing this for years. But you like your 847 Neovim plugins, and Helix doesn't have those. So here we are.

---

## How it works

| | What happens |
|---|---|
| **Motions select** | Press `w` and the next word lights up. Press `$` and everything to the end of the line lights up. Press `G` and... you get the idea. |
| **`h`/`j`/`k`/`l` navigate** | They move the cursor and select the single character under it. Like arrow keys, but for people with taste. |
| **Verbs operate on the selection** | `d` deletes it. `y` yanks it. `c` changes it. No operator-pending mode. No guessing. |
| **`Esc` cancels** | Collapses the selection back to one character. No verb executed. No harm done. |

**Everything else is untouched.** Insert mode, Visual mode, your plugins, your sanity. `mode()` returns `'n'` so your statusline plugin doesn't have an identity crisis.

---

## Quick reference

```
               Vim paradigm          Helix paradigm
  ─────────────────────────────────────────────────────
  w            move to next word     SELECT to next word
  b            move to prev word     SELECT to prev word
  e            move to end of word   SELECT to end of word
  $            move to end of line   SELECT to end of line
  0            move to start         SELECT to start
  G            move to last line     SELECT to last line
  gg           move to first line    SELECT to first line
  {  }         paragraph motion      SELECT paragraph
  /  ?  n  N   search                SELECT to match
  ─────────────────────────────────────────────────────
  h  l         move left/right       move + 1-char select
  j  k         move up/down          move + 1-char select
  ─────────────────────────────────────────────────────
  d            (needs motion)        delete selection
  y            (needs motion)        yank selection
  c            (needs motion)        change selection
  <  >         (needs motion)        shift selection
  ─────────────────────────────────────────────────────
  Esc          —                     collapse selection
```

---

## Why not just use Helix?

Because you have 200 hours invested in your Neovim config and you're not throwing that away. Because Telescope exists. Because LSP in Neovim is actually good now and you already suffered through configuring it. Because switching editors is for people who don't have deadlines.

This gives you the one thing Helix got right — selection-first editing — without making you abandon your entire setup.

## Why a fork? Why not a plugin?

Selection-first editing needs to intercept every motion handler at the C level, before Neovim's state machine processes them. A Lua plugin can't do that without being a laggy, fragile hack stapled onto the event loop. This is the kind of change that belongs in the core. So here it is, in the core.

---

## Install

### Arch Linux (AUR)

```bash
yay -S yanvim-git
```

### Building from source

Same as Neovim. It's a Neovim fork, not a different species.

```bash
cmake -B build -DCMAKE_BUILD_TYPE=RelWithDebInfo
cmake --build build -j$(nproc)
```

Install system-wide:
```bash
sudo cmake --install build
```

Or to `~/.local` (no sudo):
```bash
cmake --install build --prefix ~/.local
```

> The binary is called **`yanvim`**, not `nvim`. It installs alongside Neovim without conflicting.
> yanvim uses the same config directory as Neovim (`~/.config/nvim/`), so your existing setup just works.

See [BUILD.md](./BUILD.md) for dependencies. If you can build Neovim, you can build this. If you can't build Neovim, that's between you and CMake.

---

## Configuration

One line in your `init.lua`:

```lua
vim.opt.paradigm = 'helix'
```

Switch back anytime. No restart needed:

```lua
vim.opt.paradigm = 'vim'
```

### Highlight groups

| Group | Default | Purpose |
|:---|:---|:---|
| `HelixCursor` | → `Cursor` | 1-char resting selection |
| `HelixSelection` | → `Visual` | Active multi-char selection |

Works with any colorscheme out of the box. Override them if you want your selections to look different from Visual mode:

```lua
vim.api.nvim_set_hl(0, 'HelixSelection', { bg = '#264f78' })
```

---

## Update policy

This fork tracks upstream Neovim. I merge updates when I need them or when something interesting lands. There is no bot. There is no CI that rebases nightly.

**This is intentional.**

In an era where your package manager updates 47 things before breakfast — half of which introduce breaking changes that their own test suite didn't catch — this fork updates when a human (me) decides it's worth updating. I use this daily. If upstream ships something I need, it gets merged. If upstream ships something that breaks things, it doesn't.

You might call this "lazy maintenance." I call it "not letting a cron job ruin my editor on a Monday morning."

**Artisanal software distribution.** Hand-merged. Locally sourced. Certified free of surprise regressions at 3 AM.

---

## FAQ

<details>
<summary><b>Is this stable?</b></summary>
I use it every day. So either it's stable, or I have a very high tolerance for pain. Probably both.
</details>

<details>
<summary><b>Will my plugins break?</b></summary>
No. The helix paradigm reports <code>mode()</code> as <code>'n'</code>. Plugins see Normal mode. They don't know about the selection state and they don't need to.
</details>

<details>
<summary><b>Can I map custom keys in helix mode?</b></summary>
Standard Neovim mappings work. The paradigm only changes what the built-in motions do in Normal mode.
</details>

<details>
<summary><b>How far behind upstream are you?</b></summary>
Check the commit log. If it's more than a few weeks, I'm either on vacation or nothing interesting happened upstream. Either way, your editor still works.
</details>

---

## License

Same as Neovim. Apache 2.0 for new contributions, Vim license for code inherited from Vim. See [LICENSE.txt](./LICENSE.txt).

## Credits

Built on top of [Neovim](https://github.com/neovim/neovim), which is built on top of [Vim](https://www.vim.org/), which is built on top of [vi](https://en.wikipedia.org/wiki/Vi_(text_editor)), which is built on top of [ed](https://en.wikipedia.org/wiki/Ed_(text_editor)). It's turtles all the way down.

<!-- vim: set tw=80: -->
