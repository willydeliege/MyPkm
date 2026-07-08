---
id: keyd-navigation
aliases:
  - Keyd navigation
tags: []
---

```nix
 # configuration.nix (or a dedicated module)
{ pkgs, ... }:

{
  services.keyd = {
    enable = true;
    keyboards.default = {
      ids = [ "*" ];
      settings = {
        main = {
          # Tap = Escape (great for nvim), hold = Ctrl (great for tmux prefix / emacs-style binds)
          capslock = "overload(control, esc)";

          # Dedicated layer key. Left alt is usually free once you get used to
          # right alt for AltGr-ish stuff, and it sits right under your thumb.
          leftalt = "layer(nav)";
        };

        nav = {
          # Arrow keys on home row, vim-style
          h = "left";
          j = "down";
          k = "up";
          l = "right";

          # word / line jumps, mirroring vim motions
          u = "C-left";      # word back
          i = "C-right";     # word forward
          n = "home";
          m = "end";

          # page/scroll, mirroring tmux copy-mode & vim <C-u>/<C-d>
          y = "pageup";
          o = "pagedown";

          # Extra: delete/backspace without leaving home row
          d = "delete";
          backspace = "backspace";
        };
      };
    };
  };
}
```
