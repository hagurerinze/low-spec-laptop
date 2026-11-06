# Conky Setup Using Free AI Tools (No Account)

Date: 2026-11-06

## Goal

Make a new conky setup on Debian. This time I used a different method:

- ChatGPT as a **guest** (no account, no login)
- **Meta AI** inside WhatsApp

No paid tools. No other AI accounts.

## What I Did

1. Asked ChatGPT (guest) to make a conky config.
2. Tested it.
3. Fixed the problems one by one (see below).
4. Asked Meta AI in WhatsApp when the manual run did not work.
5. Conky works now.

## Problems and Fixes

### Problem 1: Looping information

- **What happened:** ChatGPT put a part in the config that repeats (loops) information.
- **Fix:** I asked ChatGPT for a new config without that part. It gave me a new one.

### Problem 2: Calendar error

- **What happened:** The calendar part in the new config had an error.
- **Fix:** I opened the config in `nano` and deleted the broken calendar lines myself. After that, conky ran fine.

### Problem 3: Manual run did not work

- **What happened:** I tried to start conky by hand and it did not work correctly.
- **Fix:** I asked Meta AI in WhatsApp. The reason was simple: I forgot to give the config file path.
- **Correct command:**

```bash
conky -c /path/to/your/config.conf
```

(`-c` means "use this config file".)

## Status

- Conky config: done
- Manual start: works
- Auto startup: **not done yet**

## Next Step

- Set up auto startup, so conky starts when I log in.

## Lessons

- AI can make mistakes in config files. Always test it.
- Small errors (like the calendar part) can be fixed by hand in `nano`.
- If conky does not start, check the `-c` option first.

## Note

The conky config code is not in this file. I will push the code separately.
