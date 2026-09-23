# Boot Chain Theming Project — Session Recap (with mistakes included)

Target: Debian XFCE laptop, 2GB RAM, Celeron N3xxx.

## GRUB

**What we tried first (custom theme from scratch):**
- Built a custom `theme.txt` at `/boot/grub/themes/hagure/`, using the user's
  own photo (originally 4:3, 1600x1200) resized to 1366x768, and a Terminus
  TTF font rasterized via `grub-mkfont`.
- Multiple wrong turns finding the right font file: first tried a `.ttf` path
  that didn't exist, then discovered the installed fonts were PSF console
  bitmap fonts (wrong format for grub-mkfont), then found the real
  `fonts-terminus` package only ships a `.ttf` under a different version
  number (4.46.0) than initially assumed (4.49.3).
- Misdiagnosed the actual background file in use — spent time investigating
  `/usr/share/images/desktop-base/desktop-grub.png` (a symlink into
  `update-alternatives`) before discovering the real active file was
  `/boot/grub/background.png`, auto-detected by Debian's `grub-mkconfig`
  purely by filename.
- **Bug: "polaroid" duplicate-image effect** — after selecting a boot entry,
  the screen showed a shifted/offset duplicate of the background layered
  under the boot-progress text. Chased this for a long time:
  - First theory: GRUB_GFXMODE/GRUB_GFXPAYLOAD_LINUX resolution mismatch.
    Added `GRUB_GFXMODE=1366x768` and `GRUB_GFXPAYLOAD_LINUX=keep` — bug
    persisted, theory was wrong.
  - Second theory: leftover top-level `/boot/grub/background.png` being
    redundantly drawn by `05_debian_theme` alongside the theme's own
    background — plausible but never conclusively proven before pivoting
    away from the custom theme entirely.
  - Root cause was never fully confirmed. Best guess: something specific to
    how the custom theme was built (likely tied to the original 4:3 source
    image being resized into a theme built loosely around it).

**Decision: abandoned the custom theme, switched to a premade one** —
voidlhf/StarRailGrubThemes, "Evernight" character theme, downloaded from
GitHub Releases (not the repo itself, easy mistake to nearly make since the
themes aren't in the git tree). Evernight installed cleanly and the
duplicate-image bug did NOT appear with it, using the exact same
gfxmode/gfxpayload settings — this confirmed the bug was specific to the
custom theme, not the gfx settings, but the real root cause inside the old
theme was never identified. The old `hagure` theme folder was kept on disk
as a fallback rather than deleted.

**Mistake during background swap:** accidentally overwrote
`/boot/grub/themes/Evernight/background.png` with the user's own photo when
the intent was just to inspect it, with no backup made first. Recovered
by copying from the still-intact extracted tarball at `~/Evernight/`.

**Also tested:** `quiet splash` + Plymouth `text` theme, to get a clean
transition — user disliked the splash dots, reverted to
`GRUB_CMDLINE_LINUX_DEFAULT=""`. GRUB_GFXMODE/GRUB_GFXPAYLOAD_LINUX kept
regardless, since they're a separate concern (locked resolution across the
handoff) unrelated to the splash dots issue.

**Plymouth:** deliberately skipped enhancing it, to focus time elsewhere.

## bspwm (lightweight DE/WM)

Chosen over Wayland/Hyprland (bad fit for this GPU/RAM) and over sticking
with XFCE alone. Installed alongside XFCE via LightDM's session picker.

**Live wallpaper — this took several failed attempts:**
- First tried reusing the existing xwinwrap+mpv setup from XFCE.
  Failed immediately with `X Error: BadMatch (invalid parameter attributes)`
  on window creation.
- Diagnosed (via `bash -x` and `xdpyinfo`) that bspwm's bare root window is
  depth 24, while xwinwrap's override-redirect window creation assumes
  32-bit ARGB — a real, confirmed incompatibility, not a flag/config issue.
  Tried stripping xwinwrap down to its most minimal invocation — still
  failed identically, confirming it wasn't fixable via flags.
- Decided to switch to `mpvpaper` instead (never actually implemented —
  see below).
- Also found a stray leftover mpv process from the failed xwinwrap attempts
  still running, needed to be manually `pkill`ed since it hadn't died on
  its own.
- **Final decision, after going back and forth on it multiple times
  throughout the session: skip live wallpaper for bspwm entirely.**
  Went with a static image via `feh --bg-fill` instead. This flip-flopped
  several times before landing here — draft → reconsidered → draft again →
  finally rejected for good.
- Mistake along the way: the first static-wallpaper path given had a typo/
  wrong location (file didn't actually exist there) and had to be re-found.

**Copy-paste in xterm:** took a couple of attempts — first pass didn't
account for the touchpad having no middle-click, then Ctrl+Shift+V didn't
work until `selectToClipboard` + explicit VT100 translations were added to
`.Xresources`. Physical middle-click emulation was considered but explicitly
deferred (not done).

**tint2 panel:** several edits that didn't apply on the first try —
`panel_items` sed either didn't get run or landed with the wrong ordering
more than once, requiring re-checks via `grep` before it actually stuck.
A second separator was missed initially (each `:` in `panel_items` needs
its own matching `separator = new` block — only one was defined at first
for two separators needed).

**rofi:** multiple failed styling attempts before it actually looked right —
plain default theme → background/border added → element selectors didn't
match rofi's actual internal state names (`normal.normal` etc.) so unselected
items stayed white despite edits → forcing `-theme` explicitly on the CLI
made it worse (bypassed rofi's base style merge) → had to revert that and
fix the underlying selector names → still had alternating white/black rows
because zebra-striping uses a separate `alternate.normal` selector that
hadn't been styled → fixed by adding that too. Icon theme also went back
and forth (Papirus-Dark added, then explicitly removed in favor of keeping
default Tango) with a leftover duplicate `icon-theme` line causing confusion
about which was actually active. Some white icon-background artifacts remain
unresolved — traced to the same depth-24/no-true-transparency limitation
seen elsewhere, accepted as a minor cosmetic imperfection rather than
chased further.

**GTK dark mode:** bspwm has no session daemon applying GTK theme/dark-mode
preference the way XFCE's `xfsettingsd` does, so this had to be set manually
via `lxappearance` / a hand-written `gtk-3.0/settings.ini` — not automatic.

**RAM comparison:** first measurement (bspwm ~220MB vs XFCE ~944MB total
across top processes) was flagged by the user as potentially skewed, and
they were right — zram was confirmed to be compressing hundreds of MB of
data at the time of measurement, meaning idle processes could look
artificially small in the `ps` snapshot depending on what had already been
swapped out. The comparison is directionally correct (bspwm is genuinely
lighter, largely because XFCE auto-launches several tray/indicator applets
bspwm never starts) but the exact numbers were not fully clean. A proper
fresh-reboot, immediate-`free -h` retest was planned but not yet done.

## Still open / drafted, not done

- Touchpad middle-click emulation (libinput `MiddleEmulation`)
- Per-icon panel separators (only section-level separators exist currently)
- `setsid`-detaching terminal-launched GUI apps so closing the terminal
  doesn't kill them
- General Debian command-line cheat sheet (separate, unstarted idea)
- Clean RAM re-test after a full reboot for each session
- Plymouth enhancement (deliberately deferred)
- Xmessage guest-login screen modernization (mentioned early on, never
  actually done)
