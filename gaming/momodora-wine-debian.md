# Momodora: Reverie Under the Moonlight on Debian (Wine)

This note shows how I installed the GOG version of Momodora: Reverie Under the Moonlight on Debian with Wine.

The game is old and light. It runs fine on a low-spec laptop (2GB RAM, Celeron N3xxx).

## What I used

- Debian with Wine 10.0 (from the Debian repo)
- GOG installer: `setup_momodora_reverie_under_the_moonlight_1.07_(51078).exe`
- A new Wine prefix: `~/.wine-momodora`

I used a new prefix so it does not mix with my other games.

## Step 1: Check 32-bit support

The game is 32-bit, so Debian needs i386 support.

```bash
dpkg --print-foreign-architectures
```

If `i386` is not in the list:

```bash
sudo dpkg --add-architecture i386
sudo apt update
sudo apt install wine32:i386
```

## Step 2: Install the game

```bash
cd ~/Downloads/MomodoraReverieUndertheMoonlight
WINEPREFIX=~/.wine-momodora wine 'setup_momodora_reverie_under_the_moonlight_1.07_(51078).exe'
```

Follow the installer. At the end, you can choose to launch the game. It worked for me.

## Step 3: Find the game

```bash
find ~/.wine-momodora/drive_c -maxdepth 4 -iname "*momodora*"
```

The game was here:

```
~/.wine-momodora/drive_c/GOG Games/Momodora Reverie Under the Moonlight/MomodoraRUtM.exe
```

Note: there are spaces in the folder name. Use quotes.

## Step 4: Run the game

```bash
cd "$HOME/.wine-momodora/drive_c/GOG Games/Momodora Reverie Under the Moonlight"
WINEPREFIX=~/.wine-momodora wine MomodoraRUtM.exe
```

The game works with plain Wine. I did not need DXVK.

## Problems I had

### 1. The desktop shortcut did not work

The installer made a shortcut on the desktop, but it did nothing when I clicked it.
Launching from the installer and from the terminal both worked.

### 2. I guessed the wrong folder

At first I thought the game was in a different folder. I used `find` to check, and then I found the real path.

### 3. `find` warning about `-maxdepth`

I got a warning because `-maxdepth` was placed after `-iname`. It still worked, but the correct order is:

```bash
find ~/.wine-momodora/drive_c -maxdepth 4 -iname "*momodora*"
```

## Fix: make my own launcher

Create a `.desktop` file:

```bash
mkdir -p ~/.local/share/applications
cat > ~/.local/share/applications/momodora.desktop << 'EOF'
[Desktop Entry]
Type=Application
Name=Momodora Reverie Under the Moonlight
Path=/home/hagure/.wine-momodora/drive_c/GOG Games/Momodora Reverie Under the Moonlight
Exec=env WINEPREFIX=/home/hagure/.wine-momodora wine MomodoraRUtM.exe
Terminal=false
Categories=Game;
EOF
chmod +x ~/.local/share/applications/momodora.desktop
```

To put it on the desktop:

```bash
cp ~/.local/share/applications/momodora.desktop ~/Desktop/
```

On XFCE, right-click the file and choose **Allow Launching**.

## Summary

- Use a separate Wine prefix for each game.
- Install `wine32:i386` for 32-bit games.
- Use quotes when the path has spaces.
- If the auto shortcut fails, make your own `.desktop` file.
