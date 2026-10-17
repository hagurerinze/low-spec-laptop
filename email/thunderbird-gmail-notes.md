# Thunderbird with Gmail (Remember Password Unticked)

Simple notes from my first time using Thunderbird on Debian (2GB RAM laptop).

## 1. Install

```
sudo apt install thunderbird
```

On first start, Thunderbird asks for name, email, and password. For Gmail it sets up IMAP/SMTP by itself and opens a Google sign-in window.

## 2. Remember password

- **Gmail / Outlook:** Thunderbird uses a browser login (OAuth). It saves a login token, not the password.
- **Other email:** the password is saved in Thunderbird's password manager. See it at Settings → Privacy & Security → Passwords → **Saved Passwords**.
- For more safety, turn on **Use a primary password** in the same page.

### My choice: untick "Remember password"

- Thunderbird does not save the login token.
- Each time I open Thunderbird, it shows the Google sign-in window and asks me to allow Thunderbird again.
- The view looks different from a normal browser. This is normal for Gmail.
- To save it again later: sign in again and tick **Remember password**, or check **Saved Passwords**.

## 3. Light settings for a 2GB RAM laptop

- [ ] Turn off **Global Search and Indexer** (Settings → General → Indexing). Not done yet.
- Use **IMAP**, not POP3.
- Account Settings → **Synchronization & Storage**: keep messages for only a short time (for example 1 month), or untick it to download only when I open a mail. This does not delete anything from Gmail.

## 4. Hide the big Welcome panel

- Press **F8** to hide or show the message pane.
- Or: View → Layout → **Message Pane** (press **Alt** to show the menu bar).
- To stop the Welcome page: Settings → General → **Config Editor**, search `mailnews.start_page.enabled`, set it to **false**.

## 5. What "downloading inbox messages" means

- This is IMAP sync. Thunderbird copies Gmail mail to my laptop.
- It works both ways: read, delete, star, or move in Thunderbird, and Gmail shows the same change.
- First sync can be slow. Later syncs are faster because only changes are downloaded.

## 6. Where the mail is on my laptop

```
~/.thunderbird/xxxxxxxx.default-release/ImapMail/imap.gmail.com/
```

- `INBOX` file = inbox messages. `[Gmail].sbd` = Sent, Trash, and other folders.
- Flatpak: `~/.var/app/org.mozilla.Thunderbird/.thunderbird/`
- Snap: `~/snap/thunderbird/common/.thunderbird/`
- Exact place: Help → Troubleshooting Information → **Profile Folder** → Open Folder.
- Check size: `du -sh ~/.thunderbird/`
- Do not edit or delete files there while Thunderbird is open.
- Deleting the local copy does not delete the real mail on Gmail.

## 7. Closing Thunderbird while it downloads

- Closing is safe. The download pauses, what is already downloaded is saved, and Gmail is not touched.
- Next time I open it while online, it continues and does not start from zero.
- Wait a few seconds after closing before shutting down the laptop.
- Check it fully exited: `pgrep thunderbird` (no output = closed).

### Shortcuts

| Shortcut | What happened on my setup |
|---|---|
| **Alt+F4** | Normal close. Safe. |
| **Ctrl+Q** | File → Quit. Normal close. Safe. |
| **Ctrl+W** | I tested it: it closed the whole Thunderbird window. It is also a normal close, so safe. (It is normally meant to close only a tab, but the last window closes the program.) |

### What can break the local index

- Sudden power cut
- `kill -9` or force-killing the program

## 8. Repair Folder

If a folder looks broken after a power cut:

1. Right-click the folder (for example **Inbox**) under the Gmail account.
2. Click **Properties**.
3. In the **General** tab, click **Repair Folder**.

Wait until it finishes. For a big inbox it can take some time.
