[简体中文](README.md) | English

# Luoke

**A personal AI that runs entirely on your Mac.** Luoke manages the models on your Mac and runs apps on top of them — a mail assistant, an input method. While they work for you, they organize what is yours into a memory that stays on the device, so every app understands you a little better. No cloud service, no account, no upload.

Luoke is made by **Luoke Labs (螺壳实验室)** and is in closed beta (beta1). Downloads and an introduction are at **[luoke.ai](https://luoke.ai)** (Chinese).

## What's in it

| Name | What it is | Repository |
|---|---|---|
| **Luoke** (螺壳) | The app you install: runs all local model inference (choosing models by how much memory your Mac has), installs and updates the other apps; Luoke Memory lives inside it | [luoke-core](projects/luoke-core.md) |
| **Luoke Memory** (螺壳记忆) | One on-device memory shared by all apps: records what happened and distills an understanding of you; every entry shows its evidence and can be viewed, corrected or deleted | [luoke-mem](projects/luoke-mem.md) |
| **Luoke Mail** (螺壳邮件助手) | Sorts your whole mailbox with a local model: triages new mail, cleans up old mail, audits subscriptions, picks what's worth reading, drafts replies | [luoke-mail](projects/luoke-mail.md) |
| **Luoke Input Method** (螺壳输入法) | Chinese input (Xiaohe Shuangpin and Quanpin); a local model picks the top candidate from what you've written so far and suggests what comes next | [luoke-ime](projects/luoke-ime.md) |

In progress and not yet released: proactive reminders and chat (the agent), deep search over your files and photos, and a system checkup that looks for clutter and security issues while your Mac is idle.

The component repositories are still private and will be open-sourced as they are ready.

## Principles

- **The Mac is the boundary.** Models, memory, your mail and what you type stay on this Mac. Luoke only goes online to download models and apps and to check for updates; when each app goes online is written in its own docs and in the [system docs](https://luokelabs.github.io/luoke-atlas/).
- **Your memory is yours.** Every piece of understanding shows its evidence and can be rejected or deleted; local apps can read it, and deletion happens only in Memory's own interface.
- **Only undoable things happen automatically.** Things you can undo, like archiving or marking as read, can run automatically once you've confirmed them, and you're told when they do. Things that can't be undone, like deleting, always wait for your click.
- **Say what's true.** What each app can do on different amounts of memory and different chips is written from evaluation results; if it can't do something, we say so.

## This repository

This is Luoke's top-level documentation: how the components divide the work, how they cooperate, the shared vocabulary, and decisions that span more than one component. Each component's code lives in its own repository, linked here as a submodule. Most documents are in Chinese.

- [System docs](https://luokelabs.github.io/luoke-atlas/): how each component works today — interfaces, tasks and prompts, file locations
- [Architecture](ARCHITECTURE.md): division of work, data flow, contracts between components
- [Vocabulary](CONTEXT.md): shared terms and how we name things publicly
- [Decision records](docs/adr/): decisions that span components
- [Component registry](projects/): what each component provides and depends on
- [Adding a component](docs/new-component.md)

## Download and feedback

- Download: [luoke.ai](https://luoke.ai), or the latest beta in [luokelabs/luoke-releases](https://github.com/luokelabs/luoke-releases/releases/latest). Requires an Apple silicon Mac, macOS 26.2 or later, and at least 16 GB of memory.
- Closed-beta feedback goes through the testers' own channel; for security issues see [SECURITY.md](SECURITY.md).

## For team members

```bash
git clone --recurse-submodules https://github.com/luokelabs/luoke.git
cd luoke
git submodule foreach 'git switch main && git pull'
```

The third line puts each component on the latest `main` (otherwise submodules sit on the commit recorded here, in detached HEAD). Commit and push in the component's own directory. The component commits recorded here are only updated, and tagged, when a build is cut (internal `internal-*`, beta, release), so that each build can be traced back to its code.

---

© 2026 Luoke Labs (螺壳实验室). Each component's license is in its own repository.
