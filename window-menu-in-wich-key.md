---
id: window-menu-in-wich-key
aliases:
  - window menu in wich-key
tags: []
---

# window menu in wich key

```lua

local wk = require("which-key")

wk.add({
  {
    "<leader>w",
    group = "windows",
    proxy = "<c-w>",
    expand = function()
      return require("which-key.extras").expand.win()
    end
  },
})
```
