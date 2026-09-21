# Why these skills live here

They used to live in `~/.claude/skills/` on the development box. That directory
is not a repository: nothing reviews an edit there, and everything in it is
lost with the machine. These three encode how work lands here, how a
measurement is pre-registered, and what a finding must carry before it is
written down — this project's procedures, so they belong in this project.

## ⚠️ A skill here is NOT discovered by a session started above this repo

Skill discovery walks **up** from the working directory, never down — the same
rule `AGENTS.md` files follow. Measured 2026-09-19: 59 of the 61 recorded
sessions on the development box started in `~/src`, not in a repository, and a
session there sees none of this directory.

**The bridge is a symlink into `~/.claude/skills`, and it is NOT made by hand.**
`platform/ansible/plays/dev-workstation/site.yml` already installs the
machine-level skills from `files/skills/` and already symlinks each one into
`~/.qwen/skills`, because Qwen Code reads only its own directory. The same play
makes the pointer for these. So the content is version-controlled **here**, a
change to it is a pull request in this repo, and the pointer is version
controlled **in platform** and reappears on any rebuilt box.

⚠️ **Do not answer a "skill not found" by copying the directory into
`~/.claude/skills`.** A hand-made copy on one disk is exactly what this move
deleted, the play overwrites it on the next pass, and until it does you have
two copies that drift — which had already happened to `bot-token`, where the
stale copy in `~/src/.claude/skills` still described itself as being in no
repository long after the play had adopted it. Fix the play instead.

## What is NOT here

Machine-level skills that serve every repository — `ask-local`, `bot-token`,
`herdr` — live in platform's dev-workstation play, because they configure the
box rather than this project. The deployment's own procedure,
`run-a-team-news-cycle`, lives in `fplarmband.com` beside the config it edits.
