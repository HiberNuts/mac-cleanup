---
name: mac-cleanup
description: Read-only Mac storage audit that identifies app leftovers and drills into large apps, browser profiles, and developer caches, then presents user-selectable cleanup candidates ranked by safety. Use for Mac cleanup advice or investigating large System Data; never performs cleanup.
---

# Mac cleanup audit

Inspect storage and produce a simple cleanup menu. The user chooses and performs deletions. This skill is read-only: never delete, move, rename, edit, uninstall, prune, unload, stop, quit, or restart anything. Do not perform cleanup even after the user selects a row; explain the manual steps instead. If the user says ‘delete these,’ provide manual steps for the selected items; do not execute them. Do not create reports, backups, or other files unless requested.

## 1. Measure the actual Mac

Confirm the target is macOS and discover its home directory. If tools run in a container or remote environment, establish access to the intended Mac first. Never copy another person's paths, dates, installed versions, or measurements.

Record available disk space and inspection time with `df -k`. Start with shallow `du -x -k -d 1` scans of the home directory, `~/Library`, `/Library`, `/private/var`, and installed developer-tool locations. Drill into large children instead of repeatedly scanning overlapping trees. Keep independent scans modest: package stores can contain hundreds of thousands of files.

Retain permission errors and name inaccessible areas. Privacy-protected or unreadable storage is unknown, not empty. Do not request broader access merely to make an audit appear complete; explain what remains unmeasured.

Use allocated disk usage rather than a sparse file's advertised capacity. Avoid double-counting parent/child directories, mounted simulator images, aliases, and shared package files. APFS clones, hard links, snapshots, and purgeable space mean folder sizes are estimates of potential savings, not guaranteed reclaimed space. Separate GB from GiB.

“System Data” is macOS's changing classification, not a directory to delete. Do not claim the folder scan exactly reconciles that category. Check snapshots using native read-only tools; distinguish Time Machine snapshots from macOS update snapshots.

## 2. Drill down until the choice is meaningful

A large app folder is a starting point, not a cleanup recommendation. For each major accessible consumer, inspect its largest children and split categories with different deletion consequences. Stop drilling when the user can make an informed choice from a named item, size, and concrete consequence. Do not dump every tiny file or produce hundreds of rows.

For a browser such as Arc, Chrome, or Brave:

- Map profile display names from browser metadata and measure profiles separately.
- Separate ordinary page, code, GPU, and favicon caches from service-worker/offline caches, IndexedDB, local file storage, extensions, history, cookies, saved credentials, sessions, and personal captures.
- Within large website stores, identify the owning website or extension from metadata. Numeric and hashed directory names do not identify their purpose. Report the biggest owners separately when safely attributable.
- Distinguish extension code from extension-owned databases and file stores. A download extension may hold partial transfers; a wallet may hold access-critical data. Unused-looking folders are not automatically cache.
- Ordinary caches are usually rebuildable. Offline caches may support offline documents. Website databases and local file stores may contain unsynced work. Captures are personal content.
- Inspect only names, sizes, and necessary ownership metadata. Do not print browsing history, credentials, wallet material, document contents, or private URL paths. If a database is locked, report the limitation; do not close the browser or modify it to gain access. Stale metadata is tentative evidence.

Apply the same breakdown elsewhere: separate updater downloads from app state, package caches from installed dependencies, generated builds from release archives, and runtime binaries from persistent data. Check both app support directories and separate cache locations; do not count the same bytes twice.

## 3. Classify using evidence

Use these tiers in this order:

1. **Safest candidates:** verified regenerable caches, currently inactive, with no identified unique content. Explain rebuild or download cost; never call them risk-free.
2. **Check first:** update downloads, old runtimes, optional models, unused images, and suspected app leftovers. Name the required check and what functionality is lost.
3. **Personal or offline data:** website databases, profiles, recordings, captures, histories, simulator device data, databases, and release archives. List only for explicit user review; do not recommend blanket deletion.
4. **Keep / currently in use:** active installers, required runtimes, credentials, live databases, system-managed files, or anything with unresolved ownership. Explain the reason briefly.

Safety takes priority over size. Within each tier, put larger potential savings first. Clearly mark estimates, stale measurements, and inaccessible items.

Before calling an app's files leftovers, compare bundle identifiers, installed apps in system and user locations, Spotlight results where available, project-built apps, command-line tools, and running processes. Missing from `/Applications` is insufficient. Old app data may be the user's only copy even after uninstall.

Use read-only process and open-file inspection to check active use. Match executable, parent, working directory, and listening ports when relevant. A process named `node` or `server.mjs` may belong to the editor or agent. No open handles does not prove data is expendable. Avoid printing full command lines that may contain secrets. Do not stop processes or automatically propose disabling unfamiliar services.

### Developer tools and Docker

- Package stores: identify the manager, versions, and actual store. Distinguish a supported unused-package prune from deleting the whole store. Installed runtimes and project dependencies are separate decisions. Check local help without running mutating cleanup commands.
- Build outputs: identify the project and whether builds are running. Release archives and debug symbols may be needed to debug shipped software; keep by default.
- Docker: use `docker system df` and read-only inventories. Separate build cache, images, containers, and volumes. An unreferenced local-only image may not be reproducible. Unused volumes may hold valuable databases. Keep `Docker.raw` intact; its host allocation differs from Docker's internal accounting. Do not run prune commands.
- Simulators: list installed runtimes and booted devices with `xcrun simctl`. Separate runtime downloads, generated caches, and device/test-app data. Ask which OS versions the user tests before suggesting versions for removal. Re-measure after prior cleanup: caches may already be gone. Do not remove runtimes or assume “newest only” fits every project.

### Updaters, services, and system data

- Compare installed and downloaded app versions, updater logs, processes, and open handles. A pending newer version is useful; active staging belongs in “keep.” Cached bundles can supply Dock plugins even when the main app is closed. Successful installation and inactive handles support listing old download files as candidates.
- Startup definitions: inspect labels and targets, and query applicable user, GUI, and system launchd domains. Unloaded means inactive in the queried domain, not unnecessary. Separate removing a startup definition from uninstalling software or deleting its data.
- Keep OS update snapshots, swap/sleep files, system databases, and managed assets out of manual cleanup recommendations. Time Machine backup deletion loses recovery history and requires a separate informed decision.
- Photo Booth and Photos libraries are different collections, not duplicates. Cloud-folder deletion may sync to other devices. Preserve source projects, databases, keys, and user-created media.
- Agent tools: separate conversation transcripts, history databases, generated artifacts, credentials, plugins, and runtime packages. Folder names such as `cache` and `.tmp` may contain installed code. Never suggest deleting live SQLite WAL/SHM files. For versioned binaries, identify current symlink targets and running releases; old releases may provide rollback.

## 4. Give a short, ranked cleanup menu

Lead with measured available disk space and the main finding in one or two sentences. Then show a table ordered by the safety tiers above. Use short labels and everyday language:

| # | Item | Potential space | Safety | What happens if you remove it |
|---|---|---:|---|---|
| 1 | App page cache | Measured size | Safest | Pages download again |
| 2 | Old updater download | Measured size | Check first | Confirm the update finished |
| 3 | Website offline files | Measured size | Personal/offline | May lose unsynced work |

These rows illustrate the format, not real findings. Include only measured items from the target Mac; never reuse example values. Use distinct rows for important subcategories inside an app. For browsers, include the profile/site/extension name when attribution is established. Put the best few choices first, combine tiny similar caches, and offer further detail rather than overwhelming the user. If the user asks for a full map, include all measured major consumers while keeping recommended candidates distinct from items to retain.

Below the table:

- Recommend the safest starting choices using row numbers. Sum only non-overlapping estimates and label the total approximate; do not add whole-parent sizes to child categories.
- State important exclusions and access limitations briefly. Name active items that should be left alone.
- Say explicitly: “Nothing changed. Choose the rows you want to remove, and I can give you the steps.”

When the user chooses rows, give short manual instructions using Finder, app settings, or supported tool commands. Verify the command's scope and current local help before suggesting it. Quote paths, explain whether an action is reversible, and keep each command specific to the chosen target. For profiles, sites, or extensions, prefer the app's normal removal UI; avoid editing protected preferences or mixed databases. Warn that moving items to Trash does not reclaim space until emptied, and irreversible emptying deserves a separate decision. Never run these steps within this skill.

Refresh measurements and active-use checks when the user asks again. Report cleanup as completed only if the user confirms it or read-only verification establishes the outcome; never claim the agent deleted anything. Once the easy caches are exhausted, say so instead of recommending increasingly risky data loss.
