# mac-stable

This branch is a personal maintenance branch: an upstream release plus the unmerged upstream PRs that a Mac needs.
It is not an official Strata release. All code changes here are upstream PRs, kept as one commit each with the original author.

| Base | `v0.2.10` (`ffa80d73133ea5c548aefe57889a16e8085a938f`) |
|---|---|

| PR | Disposition and reason | Upstream implementation status (2026-11-03) |
|---|---|---|
| [#2338](https://github.com/Niko1221/Strata/pull/2338) | Dropped as requested; do not replay the old buffer wrapping patch. | Included in v0.2.10 history as `38978a6d` (#2338). |
| [#2503](https://github.com/Niko1221/Strata/pull/2503) | Dropped: v0.2.10 already reads macOS available RAM for the conversation memory guard. | Equivalent implementation in `d01b1817` (#4730, #4144); original PR merge status not asserted. `src/core/conversation_memory.cpp:292-305` uses host_statistics64 and hw.memsize, counts free + purgeable + bounded file-backed inactive/speculative pages, and calls combine with the memory limit. It does not use /proc/meminfo on macOS. |
| [#2631](https://github.com/Niko1221/Strata/pull/2631) | Dropped: v0.2.10 already parks and restores Metal conversations across side requests. | Equivalent implementation in `d01b1817` (#4730, #4144); original PR merge status not asserted. `metal/src/engine/window.cpp:2010-2032` binds session QsaStates to the window K/V buffers; `:3311-3323` saves/restores window checkpoints. `src/core/conversation_state.cpp:517-545,696-709` captures snapshots and restores K/V plus running state. |

All source line references above are against the v0.2.10 tag. The upstream `d01b1817` commit reports alternating
11K/6.5K and 24K/3K conversations reusing their prompts with the same answer IDs. This rebase did not rerun that measurement.
No local engine patches remain; all files except this document are byte-identical to v0.2.10.

Rebase conflicts in #2338 (Metal files), #2503 (conversation_memory.cpp), and #2631 (Metal, core, generate, and docs)
were resolved by skipping those commits after the checks above. The pre-rebase branch is preserved locally as
`mac-stable-backup-20261103`; it is not pushed.

## Build (macOS 26, CLT 26.x)

If more than one Command Line Tools version is installed, pin the SDK the linker matches:

    SDKROOT=/Library/Developer/CommandLineTools/SDKs/MacOSX26.sdk ./setup.sh ...

## Syncing upstream

    git fetch upstream --tags
    git push origin upstream/main:main            # keep main a mirror of upstream
    git rebase --onto <new-tag> v0.2.10 mac-stable   # replay our commits on a new release
    # drop any commit whose PR upstream has merged into <new-tag>; update the table above
