<h1 align="center">
 
  <br>
  yanvim
  <br>
  <sub>Yet Another Neovim, we should call it not so Neo-vim anymore</sub>
</h1>

<p align="center">
  <a href="#building-from-source"><img src="https://img.shields.io/badge/build-cmake-blue" alt="CMake"></a>
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
  <img src="./demo.gif" alt="yanvim demo" width="800">
</p>

---

## What is this?

A hard fork of [Neovim](https://github.com/neovim/neovim) with one meaningful addition: a built-in Helix editing paradigm, switchable at runtime.

```lua
vim.opt.paradigm = 'vim'    -- default. you already know this one.
vim.opt.paradigm = 'helix'  -- the good one.
```

Set it to `'helix'` and Normal mode becomes selection-first. Motions like `w`, `b`, `$`, `G` create visual selections instead of just moving the cursor. You see what you're about to act on before you commit. Then you press `d`, `y`, `c` and it operates on the selection. No more `d3w` and praying you counted right.

Vim-style commands like `di{`, `yi"`, `ca(` still work exactly as you'd expect. You get helix selections on top of Vim, not instead of it.

---

## How it works

**Selection motions** (`w`, `b`, `e`, `$`, `0`, `G`, `gg`, `{`, `}`, `/`, `?`, `n`, `N`) create a highlighted selection from the cursor to the destination. Each motion resets the selection, pressing `w` three times selects only the third word, not all three.

**Navigation** (`h`, `j`, `k`, `l`) moves the cursor and clears any active selection. This keeps operator-pending commands working: after navigating with `hjkl`, pressing `d` waits for a motion or text object just like in regular Vim. So `di{`, `yi"`, `ca(` all work.

**Verbs** (`d`, `y`, `c`, `<`, `>`) act on the active selection if there is one. If there's no selection, they enter operator-pending mode like normal Vim. Best of both worlds.

**`Esc`** collapses the selection without doing anything.

**Everything else is untouched.** Insert mode, Visual mode, your plugins, your sanity. `mode()` returns `'n'` so your statusline plugin doesn't have an identity crisis.

---

## Quick reference

```
  SELECTION MOTIONS — create a highlighted selection
  ─────────────────────────────────────────────────────
  w  b  e        word motions         select to boundary
  $  0  ^        line motions         select to end/start
  G  gg          file motions         select to first/last line
  {  }           paragraph motions    select to paragraph
  /  ?  n  N     search               select to match

  NAVIGATION — move cursor, clear selection
  ─────────────────────────────────────────────────────
  h  l           move left/right      no selection
  j  k           move up/down         no selection

  VERBS — act on selection, or enter operator-pending
  ─────────────────────────────────────────────────────
  d              with selection:      delete it
                 without selection:   operator-pending (di{, dw, etc.)
  y              with selection:      yank it
                 without selection:   operator-pending (yi", yw, etc.)
  c              with selection:      change it
                 without selection:   operator-pending (ci(, cw, etc.)
  <  >           shift selection or operator-pending

  OTHER
  ─────────────────────────────────────────────────────
  Esc            collapse selection
  i  a           enter insert mode (unchanged)
```

---

## Why not just use Helix?

Because you have 200 hours invested in your Neovim config and you're not throwing that away. Because Telescope exists. Because LSP in Neovim is actually good now and you already suffered through configuring it. Because switching editors is for people who don't have deadlines.

This gives you the one thing Helix got right, selection-first editing, without making you abandon your entire setup. And you keep `di{`.

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
| `HelixCursor` | → `Cursor` | Cursor in resting state |
| `HelixSelection` | → `Visual` | Active selection highlight |

Works with any colorscheme out of the box. Override them if you want your selections to look different from Visual mode:

```lua
vim.api.nvim_set_hl(0, 'HelixSelection', { bg = '#264f78' })
```

---

## Update policy

This fork tracks upstream Neovim. I merge updates when I need them or when something interesting lands. There is no bot. There is no CI that rebases nightly.

**This is intentional.**

In an era where your package manager updates 47 things before breakfast, half of which introduce breaking changes that their own test suite didn't catch, this fork updates when a human (me) decides it's worth updating. I use this daily. If upstream ships something I need, it gets merged. If upstream ships something that breaks things, it doesn't.

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
