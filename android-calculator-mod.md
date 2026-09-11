# Learning Session #002 — Modding a Calculator App for Android 5.1 Was Not as Simple as I Thought

**Date:** 2026-07-28

## Summary
My original goal sounded simple: make Android 5.1.1 look more modern. I decided to start with the Calculator app, since I assumed it would be the easiest one to modify. I was wrong.

## Expectation
Since AI tools became common, I got used to thinking that almost anything could be done quickly. I remember thinking: *"It's just a Calculator. I'll probably just need to change the appearance."* Reality turned out very different.

## Process
I started using an AI assistant alongside development. A lot of my work looked like copy-and-paste on the surface, but I never treated it as just that — every step raised a new question I had to understand.

Step by step, I got introduced to things I had never worked with before:
- JDK
- Java
- Android Studio
- The Android build process
- The structure of an older Android project
- The limitations of Android 5.1.1
- The limitations of a 2GB RAM laptop

This was when I realized that even a simple-looking Android app has many interconnected parts.

## Result
Honestly, the result was bad. The Calculator did not look modern at all — some buttons rendered with a strange shape, closer to an old phone UI than the clean design I wanted. On top of that, the app technically ran, but crashed every time a math operation button (like `+`) was pressed.

## What I learned
I started this project thinking it was about building a Calculator. By the end, I realized the real goal had shifted — it was no longer "how do I build a Calculator," but "how do I understand things I previously knew nothing about."

Some specific lessons:
- A simple-looking app is not necessarily simple to build.
- AI can speed up the work, but it does not replace the need to understand the underlying system.
- Old Android versions come with many limitations that have to be understood, not guessed at.
- A successful build does not mean the app is actually correct.
- A crash is often the starting point for understanding an app's real architecture.

## Reflection
This project fails if the only goal was producing a good Calculator app. It succeeds if the goal was learning. I now see AI less as a tool that makes everything easy, and more like a guide that opens doors to things I did not even know existed.

## Next step
The next target is not just fixing the Calculator. It is understanding *why* each change affects the whole app — especially on an Android 5.1.1 project with so many constraints.
