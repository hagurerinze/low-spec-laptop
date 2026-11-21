# New GRUB Theme With Less AI Help

Date: 2026-11-21

## Goal

I opened my laptop and wanted to change my GRUB theme. I decided to challenge myself:
use my own brain and experience as much as I can, and use AI only for small questions.

AI used: **Meta AI on WhatsApp Business** (only when I was stuck).

## Steps

1. I copied my old GRUB style and edited the image myself in **Pixellab**. It is my old favorite editing app on Android.
2. After editing, I sent the theme files to my laptop by **Bluetooth**. I also used Bluetooth before to send images between devices.
3. I wanted to copy the old GRUB folder to a new folder, but I got a problem. I read `cp --help`, but it did not help. So I asked Meta AI.
4. Copying and editing were done. I asked Meta AI which second text file I need to edit. After checking, there was none, so I rebooted.
5. GRUB rejected my `.png` file.

## Problems and Fixes

### Problem 1: Copy command

- **What happened:** I did not know how to copy the old GRUB folder. `cp --help` did not help me.
- **Fix:** I asked Meta AI and the copy worked.

### Problem 2: GRUB rejected the .png

- **What happened:** After reboot, GRUB did not accept my image.
- **My guess:** I did not convert the image. I only renamed a `.jpg` file to `.png`.
- **Fix:** My guess was right. I converted the image with **ImageMagick**. After that, the new GRUB worked.

### Problem 3: Terminal name in bspwm

- **What happened:** After GRUB, I wanted to change my bspwm keyboard shortcut. I like XFCE Terminal more than xterm. I forgot the program name. I tried `which` with a name I guessed, and with another name, but both failed.
- **Fix:** I asked Meta AI again and found the real name (`xfce4-terminal`).

### Problem 4: sxhkd did not respond

- **What happened:** I kept xterm on `alt + Return`. After I restarted sxhkd, the shortcut did nothing.
- **Cause:** I looked at the file again. I made a typo: I wrote `return` instead of `Return`.
- **Fix:** I fixed the typo and it worked. (Key names are case sensitive.)

## Result

- New GRUB theme: done
- bspwm shortcuts: done
  - XFCE Terminal: `super + Return`
  - xterm: `alt + Return`

## Lessons

- A `.jpg` file renamed to `.png` is not a real `.png`. Use ImageMagick to convert it.
- Check small typos first when a shortcut does not work (like `Return` and `return`).
- When `which` fails, the program name may be different from what I guess.

## Why I Did This

I wanted to see how far I can go with my own brain and experience in something I like, with less AI help.
I tried a similar experiment before with the conky project (using GPT). For this one, I think I understand the workflow better than in the conky experiment.
