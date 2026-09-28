Chat — XFCE Nordic Theme Discussion

User

What are difference about "/usr/share/themes" and "~/.themes"

Assistant

The difference is primarily scope, ownership, and precedence in the Linux filesystem hierarchy.

"/usr/share/themes" — system-wide themes

This directory is part of the global system data tree.

- Scope: Available to all users on the system
- Ownership: Typically owned by "root"
- Write access: Requires "sudo" / root privileges
- Source: Usually installed through the package manager or manually as root
- Purpose: System-wide themes

Themes installed here are available to every user.

"~/.themes" — user-local themes

This directory belongs to the current user.

- Scope: Available only to that user
- Ownership: Owned by the user
- Write access: No root privileges required
- Source: Usually manually downloaded or customized
- Purpose: Personal themes and experimentation

This is generally the safer location for custom themes.

Theme precedence

Desktop environments can search both locations.

A user-local theme can override or take precedence over a system-wide theme with the same name, depending on the desktop environment and theme implementation.

This makes "~/.themes" useful for customizing a theme without modifying the system copy.

Typical structure

Both locations can contain theme directories such as:

gtk-3.0/
gtk-4.0/
xfwm4/
index.theme

The exact directories depend on the theme.

Practical use

Use "/usr/share/themes" when:

- The theme should be available system-wide.
- The theme is managed by the package manager.
- Multiple users need the theme.

Use "~/.themes" when:

- The theme is downloaded manually.
- You are experimenting or modifying it.
- You want to avoid changing system files.

---

User

I borrow nordic themes

I already set the style

But there is another folder like nordic folders and nordic standard buttons

How to use it

Assistant

What you downloaded is likely a theme bundle. Nordic themes can contain several components, each intended for a different part of the desktop.

Since you're using XFCE, the components generally map like this.

Nordic — GTK theme

The main "Nordic" folder controls the GTK appearance of applications:

- Menus
- Buttons
- Dialogs
- Panels
- Application widgets

You can select it through:

Settings → Appearance → Style

If you already selected "Nordic" there, this part is configured.

Nordic standard buttons — XFWM theme

A folder such as:

Nordic-standard-buttons

is likely an XFWM4 window-manager theme.

It controls:

- Window title bar
- Close button
- Minimize button
- Maximize button
- Window borders

Select it through:

Settings → Window Manager → Style

If you only selected "Nordic" under Appearance → Style, the window borders may still be using another theme.

Nordic folders — icon theme

A folder such as:

Nordic-folders

is likely an icon theme, rather than a GTK theme.

It controls things such as:

- Folder icons
- File icons
- Application icons
- Other desktop icons

For user-local installation, icon themes normally belong in:

~/.icons/

or:

~/.local/share/icons/

System-wide icon themes normally belong in:

/usr/share/icons/

Then select the icon theme through:

Settings → Appearance → Icons

Recommended XFCE layout

For example:

~/.themes/
└── Nordic/
    ├── gtk-3.0/
    ├── gtk-4.0/
    ├── xfwm4/
    └── index.theme

~/.themes/
└── Nordic-standard-buttons/
    └── xfwm4/
        └── ...

~/.icons/
└── Nordic-folders/
    └── ...

However, the exact structure depends on how the Nordic package was distributed.

XFCE configuration

The three components are generally applied independently:

Appearance → Style
    Nordic

Appearance → Icons
    Nordic-folders

Window Manager → Style
    Nordic-standard-buttons

So having the folders installed does not automatically mean all three components are being used. Each desktop layer has its own theme selector.
