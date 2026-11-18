---
name: project-setup
description: >
  Check and install the recommended skills for Roblox and game-development work from this repository into the user's
  personal skills folder. Use whenever the user asks which skills they need, what to install, whether a skill is
  installed, to set up or refresh their skills, or says things like "setup", "install skills", "কোন স্কিল লাগবে",
  "স্কিল ইনস্টল করো", "স্কিল চেক করো", even if they do not name this skill. Never copies anything without a clear yes.
---

# Project Setup: recommended skills

This skill keeps the user's personal skills folder in line with the list in this repository. It answers "which skills
do I need?", checks what is installed, and copies missing skills only after the user says yes.

## When to load

When the user asks which skills to use, whether something is installed, or asks to set up or refresh skills. Do not
load it for ordinary Roblox or design tasks; those tasks use the skills they match directly.

## Steps

1. **Read the manifest.** Open `references/manifest.md` in this skill folder. It lists every recommended skill, its
   source path in the repository, and what to skip.

2. **Check what is installed.** List the personal skills folder:
   ```bash
   ls ~/.claude/skills
   ```
   A skill counts as installed only when its folder exists there and contains `SKILL.md`. Note that the Roblox plugin
   provides some skills (for example `roblox-security`) under a plugin name, so do not treat a plugin skill as missing.

3. **Report, then ask.** Show the user two short lists: already installed, and missing. Name the source of each
   missing skill and the total count. Ask one question: "এগুলো ইনস্টল করব?" Do not ask about each skill separately
   unless the user wants to choose.

4. **Copy only after a yes.** Copy the whole folder for each approved skill. Use the source paths in the manifest.
   For the UI UX Pro Max skills, copy the whole `ui-ux-pro-max/` folder so the shared `data/` and `scripts/` stay with
   it. Example for one skill:
   ```bash
   cp -r "/d/Game Development Master Skills/skills/third-party/superpowers/brainstorming" ~/.claude/skills/
   ```
   Copy only the approved ones. If the user approves part of the list, copy only that part.

5. **Verify.** For each copied skill, confirm `SKILL.md` exists in the destination. Report the result as a list, and
   say that a new session (or restarting Claude Code) may be needed before the new skills appear.

## Rules

- Never copy, delete or overwrite anything in the personal skills folder without an explicit yes in this conversation.
  Approval given earlier for a different set does not carry over.
- If a destination folder already exists, show the user the difference and ask before replacing it.
- Never install `using-superpowers`. It is in the manifest's skip list on purpose.
- If a source path does not exist in the repository, stop, report the missing path, and do not guess a substitute.
- Report what was copied and what was not. Do not claim a skill works in a session until it has been used there.
