# XFCE Nordic Theme Setup

## Problem

I downloaded the Nordic theme bundle. It has more than one folder:

- `Nordic`
- `Nordic-standard-buttons`
- `Nordic-folders`

I already set the style, but I did not know how to use the other two folders.

## Two places for themes

| | `/usr/share/themes` | `~/.themes` |
|---|---|---|
| Who can use it | All users | Only me |
| Owner | root | My user |
| Need sudo? | Yes | No |
| Usually comes from | Package manager | Manual download |

If both places have a theme with the same name, the one in `~/.themes` can be used first. This depends on the desktop and the theme.

For themes I download by hand, `~/.themes` is the safer place. I do not change any system files.

## What each Nordic folder does

In most Nordic bundles, the three folders are for three different parts of the desktop.

| Folder | Type | What it changes | Where to select it |
|---|---|---|---|
| `Nordic` | GTK theme | Menus, buttons, dialogs, panels | Settings → Appearance → Style |
| `Nordic-standard-buttons` | XFWM theme | Title bar, close / minimize / maximize buttons, window borders | Settings → Window Manager → Style |
| `Nordic-folders` | Icon theme | Folder, file, and app icons | Settings → Appearance → Icons |

## Where to put the folders

```text
~/.themes/
├── Nordic/
│   ├── gtk-3.0/
│   ├── gtk-4.0/
│   ├── xfwm4/
│   └── index.theme
└── Nordic-standard-buttons/
    └── xfwm4/

~/.icons/
└── Nordic-folders/
```

Icon themes can also go in `~/.local/share/icons/`. The exact folders depend on how the Nordic package was made.

## Important

Each layer of the desktop has its own selector. Having the folders installed does not mean all three are used.

I need to select all three:

```text
Appearance → Style            →  Nordic
Appearance → Icons            →  Nordic-folders
Window Manager → Style        →  Nordic-standard-buttons
```
