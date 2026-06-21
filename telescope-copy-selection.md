---
id: telescope-copy-selection
aliases:
  - telescope copy selection
tags: []
---
# Telescop copy selection

Here are the main approaches to copy a Telescope selection to the clipboard:

---

## 1. Custom action in Telescope mappings (most common)

Add a mapping in your `telescope.setup()` that yanks the selected entry's value to the `+` (system) clipboard:

```lua
local actions = require("telescope.actions")
local action_state = require("telescope.actions.state")

require("telescope").setup({
  defaults = {
    mappings = {
      i = {
        ["<C-y>"] = function(prompt_bufnr)
          local entry = action_state.get_selected_entry()
          actions.close(prompt_bufnr)
          vim.fn.setreg("+", entry.value)
          vim.notify("Copied: " .. entry.value)
        end,
      },
      n = {
        ["yy"] = function(prompt_bufnr)
          local entry = action_state.get_selected_entry()
          actions.close(prompt_bufnr)
          vim.fn.setreg("+", entry.value)
          vim.notify("Copied: " .. entry.value)
        end,
      },
    },
  },
})
```

`entry.value` is the full path for file pickers. Use `entry.ordinal` for the display name only.

---

## 2. Copy just the filename vs full path

Different fields on the entry object:

| Field | Content |
|---|---|
| `entry.value` | Full path (e.g. `/home/user/foo/bar.lua`) |
| `entry.ordinal` | Display string (often just filename) |
| `entry.path` | Explicit path field (file pickers) |
| `entry.filename` | Filename (grep pickers) |

---

## 3. Per-picker overrides

If you only want this in `find_files` but not everywhere:

```lua
require("telescope").setup({
  pickers = {
    find_files = {
      mappings = {
        i = {
          ["<C-y>"] = function(prompt_bufnr)
            local entry = action_state.get_selected_entry()
            actions.close(prompt_bufnr)
            vim.fn.setreg("+", entry.value)
          end,
        },
      },
    },
  },
})
```

---

## 4. Clipboard register note (Neovim 0.10+)

Neovim 0.10+ has native OSC52 clipboard support, so `vim.fn.setreg("+", ...)` should work over SSH too. Make sure your config has:

```lua
vim.opt.clipboard = "unnamedplus"  -- optional: makes + the default yank register
```

Or set it only via `setreg` in the action (as above) to keep yanks local by default.

---

**TL;DR:** The key is `action_state.get_selected_entry()` to grab what's highlighted, then `vim.fn.setreg("+", entry.value)` to push it to the system clipboard. Wire that to any key inside a Telescope `mappings` block.
