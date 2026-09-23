# Learning Session #001 — Organizing Folders Taught Me More Than I Expected

**Date:** 2026-07-28

## Summary
I originally just wanted to clean up my laptop's folder structure. The process ended up taking about two days, and honestly, I still do not consider it fully finished. What surprised me was that the real result was not just a tidier set of folders — I picked up several lessons I did not expect going in.

## How it started
I first asked Claude for help organizing my folders. Claude gave me a `.sh` script that could automate moving files around. Once I hit Claude's usage limit, I continued the discussion with ChatGPT, which suggested a simpler approach using the `mv` command directly.

After checking my command history (`history | tail`) and comparing both approaches, I noticed a real difference in how each one "thought" about the problem — one leaned toward a full script with more structure, the other toward a simpler direct command.

## What I learned
Along the way, I picked up several things that were not directly about coding:
- The basics of how Bash thinking works.
- The structure and purpose of `.sh` scripts.
- Why understanding directory structure (`tree`) matters.
- Becoming more aware of the risk of losing files during bulk operations.
- Realizing that a simple command can sometimes be riskier than a more complex script, because it lacks built-in safety checks.

## Reflection
The most valuable part of this was not the script itself. It was learning how to **audit** the AI's output — not just accept it.

I got into the habit of:
- Re-checking each command before running it,
- Reviewing what actually changed afterward,
- Comparing different proposed solutions against each other,
- Making sure no files were lost in the process.

It felt like training my memory and observation skills at the same time. The more I practiced this, the easier it became to notice when something felt "off."

## Outcome
What started as a simple folder-cleanup session turned into the first real lesson in treating AI output as a draft to verify, not a finished answer to trust blindly. This habit carried directly into my later, more complex projects (see [Home Directory Reorganization](./home-directory-reorganization.md)), where catching a subtle bug early made the difference between a safe recovery and real data loss.
