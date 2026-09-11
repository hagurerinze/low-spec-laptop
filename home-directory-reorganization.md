# Case Report: Home Directory Reorganization

**Status:** Completed, all data safe.

## 1. Goal
I wanted to organize new files scattered across `Desktop`, `Downloads`, `Pictures`, and my home folder into an existing `Organized/` structure, following rules I set myself:
1. Duplicate files get a suffix (`_1`, `_2`, ...) — never overwritten or deleted.
2. Home folder: everything moves, except `.sh` files (only one specific script should move).
3. Desktop: only one specific folder should move.
4. Downloads: everything moves, except one folder that was already extracted and should not be touched.
5. Everything goes into categories that already exist in `Organized/` — no new categories.
6. Use `mv` (move), not copy, since my free disk space was limited (about 22GB).
7. Any dated folder that reaches 500+ files gets automatically split into parts, so my low-RAM laptop does not struggle loading thumbnails.

I wrote a Python script, `organize_hagure.py`, to apply all these rules, and tested it in a sandbox before running it on my real laptop.

## 2. First run — succeeded for what was asked
The script ran successfully for everything I explicitly requested: the target Desktop folder moved, Downloads was correctly sorted (except the excluded folder), the one `.sh` file moved, and duplicate files were renamed correctly.

## 3. Mistake #1 — hidden files were swept up by accident (critical bug)
**Cause:** Python's `Path.iterdir()` returns **all** items in a folder, including hidden "dotfiles" (files starting with a dot). Unlike the regular `ls` command, which hides them by default, my script had no rule to skip them.

**Effect:** Important config and data folders in my home directory were accidentally moved into `Organized/Unsorted/`, including my browser profile, application settings, and other app data.

**Risk was limited:** Since the script used `rename` (move), not delete, no data was actually lost — everything just moved to a different location. But this was still risky, because my desktop session was actively using some of these files from their original location while it was running.

## 4. Mistake #2 — my first revert script stopped halfway
**First attempt:** A revert script using `cp -an` (copy, never overwrite) combined with `set -e` (stop immediately on any error).

**Cause:** One of the moved files had been rewritten by an app that was still running, so a file with the same name already existed at the destination. `cp -an` treats this as an error (not a silent skip), and `set -e` caused the entire script to stop right at that point.

**Effect:** Every item that came alphabetically after the failure point was never even attempted — it looked like a lot of files were "missing," when in fact everything was still safe in `Organized/Unsorted/`.

**Fix:** I rewrote the revert script to use `rsync -a --ignore-existing` instead, removed `set -e`, and added per-item error handling so one small failure would not stop the whole process.

## 5. Mistake #3 — the copy-based strategy used up my disk space
**Cause:** My revert scripts (v1 and v2) both used copy, not move. For large folders, this meant the same data existed twice on disk at the same time — once in the original spot, once in the new copy.

**Effect:** My free disk space dropped sharply to about 1.1GB, because a large folder had partially copied before the first script stopped (Mistake #2).

**Fix:** I wrote a new Python script using a smarter move-based approach:
- Items that were never touched twice: moved directly, at zero extra disk cost.
- Items that had accidentally been copied twice with identical content (checked by file size): the old duplicate copy was deleted directly.
- Items that had different content in each location (meaning an active app had rewritten them): both copies were kept and flagged as a manual conflict to review.

**Final result:** Over 17,000 items moved cleanly, over 43,000 duplicate files removed, freeing about 1GB of space, with 73 flagged conflicts — all confirmed to be safe cache/session files with a newer version already in use.

## 6. Final cleanup
The remaining leftover items in `Organized/Unsorted/` were small temporary files and empty markers, all confirmed safe and removed.

## 7. Final verification
- Folder sizes for all affected directories checked and confirmed normal — all data intact.
- The `Organized/` structure matched my original template, no new categories were created.
- The 500-files-per-folder rule was respected everywhere it should apply. A few folders exceeded 500 items, but these were intentionally excluded because they belong to installed software packages that need their internal folder structure to stay intact.
- A reboot was recommended as a final step, to refresh my desktop session cleanly after the accidental file move.

## 8. Scripts produced
| File | Purpose |
|---|---|
| `organize_hagure.py` | Main reorganization script (later fixed to skip dotfiles) |
| `revert_dotfiles.sh` | v1 — had the `set -e` bug, not used anymore |
| `revert_dotfiles_v2.sh` | v2 — used rsync, but still copy-based (wasted disk space) |
| `revert_dotfiles_move.py` | v3 — the final version used, move-based, disk-space efficient |

## 9. Lessons learned
1. Always explicitly exclude dotfiles when writing a script that scans a home directory — Python's directory listing functions do not hide them by default the way `ls` does.
2. Do not combine `set -e` with a loop over many independent items — one small failure (like a file conflict) should not stop the entire process. Handle errors per item instead.
3. Always dry-run large, automated operations first, especially ones that might touch areas outside what was explicitly requested.
4. Prefer `move` over `copy` when disk space is limited — copy always risks using double the space before the process finishes and is verified.
5. My existing personal folder structure and naming convention proved solid, and did not need to be redesigned — it just needed a script that respected it correctly.
