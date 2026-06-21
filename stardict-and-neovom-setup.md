---
id: stardict-and-neovom-setup
aliases:
  - stardict and neovom setup
tags: []
---

# Stardict in neovim setup Good

## StarDict + Neovim Spell: Two Separate Systems

It's important to understand the distinction up front:

- **Neovim's built-in spell checker** (`set spell`) highlights misspelled words
  using `.spl` files. It does _not_ use StarDict format.
- **StarDict / `sdcv`** is a separate dictionary lookup tool (definitions,
  translations). It doesn't drive spell highlighting on its own.

You can use both together: spell-check highlights errors, and sdcv lets you look
up definitions or translations on demand.

---

## Part 1 — Neovim Built-in Spell Checking (French + English)

### Enable spell checking

By default, spell checking is not enabled in Neovim. You can turn it on with
`:set spell spelllang=en`, which enables spell checking and sets the language to
English. Only English works out-of-the-box; for other languages you need to
download the `.spl` files.

For French, either run `:set spelllang=fr` and Neovim will prompt you to
download the French spell file automatically, or download `fr.utf-8.spl` and
`fr.utf-8.sug` manually and place them in `~/.config/nvim/spell/`.

### Both languages simultaneously

Add this to your config to enable both at once:

```lua
vim.opt.spell = true
vim.opt.spelllang = { "en", "fr" }
```

Use `]s` and `[s` to jump to the next or previous spelling mistake.

### Toggle per language (recommended)

A useful pattern is a toggle function that switches between languages or turns
spell off:

```lua
local function toggle_spell(lang)
  local spell_on = vim.opt_local.spell:get()
  local current_langs = vim.opt_local.spelllang:get()
  local current_lang = spell_on and current_langs[1] or nil
  if current_lang == lang then
    vim.opt_local.spell = false
    return
  end
  vim.opt_local.spell = true
  vim.opt_local.spelllang = { lang }
end

vim.keymap.set("n", "<leader>ne", function() toggle_spell("en") end, { desc = "Toggle English spell" })
vim.keymap.set("n", "<leader>nf", function() toggle_spell("fr") end, { desc = "Toggle French spell" })
```

### Key spell commands

| Key         | Action                                 |
| ----------- | -------------------------------------- |
| `]s` / `[s` | Next / previous misspelling            |
| `z=`        | Show suggestions for word under cursor |
| `zg`        | Add word to your personal dictionary   |
| `zw`        | Mark word as bad                       |
| `1z=`       | Accept first suggestion without prompt |

### Custom word list

You can set a custom spell file location, for example:

```lua
vim.opt.spellfile = vim.fn.stdpath("config") .. "/spell/en.utf-8.add"
```

This stores your personal word list at `~/.config/nvim/spell/en.utf-8.add`.
Adding a word with `zg` appends it there, and Neovim automatically compiles it
into a binary `.spl` file.

For French, add a second file:

````lua
vim.opt.spellfile = { "~/.config/nvim/spell/en.utf-8.add", "~/.config/nvim/spell/fr.utf-8.add" }`.`

---

## Part 2 — StarDict (SDCV) Integration for Word Lookup

StarDict gives you rich dictionary definitions from inside Neovim. Install
`sdcv` first:

```bash
# Debian/Ubuntu
sudo apt install sdcv

# Arch
sudo pacman -S sdcv

# macOS
brew install sdcv
````

Then download StarDict dictionaries (`.ifo`/`.idx`/`.dict.dz` triplets) and
place them in `~/.stardict/dic/` or `/usr/share/stardict/dic/`. Good sources:
[freedict.org](https://freedict.org),
[huzheng.org/stardict](http://huzheng.org/stardict/).

### Plugin: vim-stardict

The `vim-stardict` plugin integrates SDCV into Vim/Neovim. It opens a split and
uses syntax highlighting to present word definitions in an organized way.
Install it with your plugin manager, then configure it:

```vim
" Horizontal split
let g:stardict_split_horizontal = 1
let g:stardict_split_size = 20

" Key maps
nnoremap <leader>sw :StarDict<Space>
nnoremap <leader>sc :StarDictCursor<CR>
```

`:StarDictCursor` looks up the word under your cursor immediately — very handy
when spell check flags something.

In Lua (lazy.nvim):

```lua
{
  "phongvcao/vim-stardict",
  config = function()
    vim.g.stardict_split_horizontal = 1
    vim.g.stardict_split_size = 20
    vim.keymap.set("n", "<leader>sw", ":StarDict<Space>", {})
    vim.keymap.set("n", "<leader>sc", ":StarDictCursor<CR>", {})
  end
}
```

### Raw sdcv without a plugin

You can call sdcv directly from Neovim without any plugin:

```lua
vim.keymap.set("n", "<leader>sd", function()
  local word = vim.fn.expand("<cword>")
  vim.cmd("split | terminal sdcv " .. word)
end, { desc = "Look up word in sdcv" })
```

---

## Tying It Together

A good workflow for bilingual writing:

1. Use `<leader>nf` / `<leader>ne` to switch active spell language per buffer.
2. Navigate errors with `]s`, then press `z=` for Neovim's built-in suggestions.
3. Press `<leader>sc` to look up a flagged word's definition or translation in
   StarDict.
4. Press `zg` to silently add correctly spelled proper nouns to your personal
   word list.

This gives you on-the-fly spell highlighting in both languages plus rich
dictionary lookup without leaving Neovim.
