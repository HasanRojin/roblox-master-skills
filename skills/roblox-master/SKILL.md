---
name: roblox-master
description: >
  Start a Roblox or game-development session with the full master-skills setup. Use when the user types /roblox-master,
  or says "roblox master", "start roblox session", "load master skills", "গেম শুরু করো", "মাস্টার স্কিল লোড করো", at the
  beginning of any new game or Roblox project, before any design, scripting or map work. Loads the repository rules and
  the session flow; it does not copy or install anything.
---

# Roblox Master: session starter

This skill is the single entry point for the Roblox master-skills setup. It loads the repository rules and the session
flow. The individual skills stay installed in the personal skills folder and are picked up automatically when a task
matches their descriptions, so this skill does not load every skill's full text.

## Steps

1. **Read the repository rules.** Open these two files:
   - `D:\Game Development Master Skills\CLAUDE.md`
   - `D:\Game Development Master Skills\UNIVERSAL-BOOTSTRAP.md`
   If a file cannot be read, say so and stop. Do not guess its contents.

2. **Check what is installed.** Follow the steps in the `project-setup` skill (list the personal skills folder, compare
   against `project-setup/references/manifest.md`). Report only the summary: how many recommended skills are installed,
   and the names of any that are missing. Do not copy anything here; ask first if something is missing.

3. **Confirm the session flow.** Tell the user in two or three lines:
   - the phases: discussion, then plan, then work;
   - that skills load on demand when a task matches them;
   - that no code or map work starts until a plan is agreed.

4. **Ask for the first task.** Ask what the user wants to build or fix. Do not start coding in this turn.

## Rules

- Do not load the full text of every skill. Load a skill only when the agreed plan needs it.
- Do not change files or install anything in this skill. Installation belongs to `project-setup`, and only after a yes.
- Keep the summary short. The user wants to start working, not read a report.
