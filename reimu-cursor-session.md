# Session Notes: Hakurei Reimu Mouse Cursor on Debian XFCE

## How the session started

- Started out wanting to continue "organizing files part 3", but the files turned out to be on a **different laptop**, so that was dropped.
- Switched topic to **finding a Hakurei Reimu mouse cursor** for the Debian XFCE laptop.

## First recommendation: Tubs' set on itch.io

- Found a free **Reimu Hakurei cursor set by Tubs** on itch.io (`tubulartoasts.itch.io/reimu-cursor`).
- Includes: idle, link select, diagonal / vertical / horizontal resize, and animated "working in background" and "busy" cursors, plus sprites.
- It is a **Windows-style pack** (`.cur` / `.ani`), so on Debian it would need converting to an X11 cursor theme with `win2xcur`.

## Found a better one on GNOME-Look

- Found a ready-made **X11 cursor theme** of Reimu on gnome-look.org, which made the conversion unnecessary.
- Source page: https://www.gnome-look.org/p/1914275
- Folder contents: `cursors/`, `index.theme`, `preview.png`, `thumb.png`.
- Theme name in `index.theme`: **Reimu cursors** (animated, ported to X11 cursors).
- Credit: the original art is by a pixiv artist (https://www.pixiv.net/en/users/345405); the X11 port on GNOME-Look was made by someone else. Keep this credit if the theme gets reused or shared.

## Detour: installed win2xcur, then undid it

Before switching to the GNOME-Look theme, `pipx` and `win2xcur` had already been installed.

```bash
sudo apt install pipx
pipx install win2xcur
pipx ensurepath
```

Undo steps:

```bash
pipx uninstall win2xcur
sudo apt purge pipx
sudo apt autoremove        # answer y; only removes python3-argcomplete, python3-click, python3-userpath
```

- `autoremove` was aborted the first time because `n` was typed at the prompt. It has to be re-run with `y`.
- **Python itself must not be removed.** Debian depends on it. Only those three packages came with pipx.
- `pipx ensurepath` added two lines to `~/.bashrc` (lines 117-118):

  ```
  # Created by `pipx` on 2026-10-01 08:19:57
  export PATH="$PATH:/home/hagure/.local/bin"
  ```

  Remove them with:

  ```bash
  sed -i '117,118d' ~/.bashrc
  ```

  Keep the older `export PATH=$PATH:/usr/sbin` line.
- `grep -n "pipx" ~/.bashrc` only found the comment line, because the `export` line below it does not contain the word "pipx".
- Leftover files cleaned with:

  ```bash
  rm -rf ~/.local/share/pipx ~/.local/bin/win2xcur ~/.local/bin/inspectcur ~/.local/bin/win2xcurtheme ~/.local/bin/x2wincur ~/.local/bin/x2wincurtheme
  ```

- `/etc` is tracked by etckeeper, so apt commands produced auto-commit messages. That is normal.

## Final install

```bash
mkdir -p ~/.icons
cp -r ~/Downloads/Reimu ~/.icons/
cat ~/.icons/Reimu/index.theme
ls ~/.icons/Reimu/cursors | head -30
```

- The `cursors/` folder already has the standard X11 names (`left_ptr`, `hand2`, `ibeam`, `ew-resize`, `default`, and so on), so **no renaming or symlinks were needed**.

## Applying the theme

- GUI: **Settings → Mouse and Touchpad → Theme tab → Reimu cursors**
- Terminal:

  ```bash
  xfconf-query -c xsettings -p /Gtk/CursorThemeName -s "Reimu cursors"
  ```

## Troubleshooting

- Theme missing from the list: check that the folder is `~/.icons/Reimu` and that `index.theme` has a `Name=` line.
- Old cursor still showing in some apps (Firefox, GTK apps): log out and back in.
- Apps running as root use a separate cursor setting and may keep the default.
- Cursor too small or large: with the Windows-style route, `win2xcur` has a `--scale` option. For this X11 theme, check the cursor size setting in the same Settings menu.
- To undo: pick the previous theme in the same menu and delete `~/.icons/Reimu`.

## Result

Reimu cursor theme installed and working on the Debian XFCE laptop.
