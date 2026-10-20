# Mac cleanup audit

A read-only agent skill that explains what is using space on a Mac and ranks cleanup candidates from safest to most cautious.

It does not delete files, quit apps, stop services, or run cleanup commands. You choose what to remove and perform the steps yourself.

## Use

Give `SKILL.md` to an agent with local, read-only access to your Mac and ask:

> Audit my Mac storage. Show a simple table ranked by deletion safety, with sizes and consequences. Drill into large apps and browser profiles. Change nothing.

For Codex, put this repository in `~/.codex/skills/mac-cleanup`, then invoke `$mac-cleanup` in a new session.

```sh
git clone https://github.com/HiberNuts/mac-cleanup.git ~/.codex/skills/mac-cleanup
```

The destination must not already exist. Other agents can read `SKILL.md` directly; their skill installation conventions may differ.

## What it covers

- App caches, updater downloads, and possible leftovers from removed apps
- Browser profiles, page caches, offline content, website databases, extensions, and captures
- Developer caches, Docker storage, Xcode runtimes, and simulator data
- Background services, agent history, and system-managed storage

The report separates rebuildable caches from data that may contain drafts, credentials, database records, or personal files. It reports access limitations and avoids counting the same storage twice.

## Limits

Safety labels are recommendations, not guarantees. The skill is an instruction document, not a sandbox or access-control mechanism: the agent and its tools must follow the read-only constraint. macOS privacy permissions, shared files, APFS storage accounting, and active applications can limit accuracy. Review manual commands before running them.
