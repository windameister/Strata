# mac-stable

This branch is a personal maintenance branch: an upstream release plus the unmerged upstream PRs that a Mac needs.
It is not an official Strata release. All code changes here are upstream PRs, kept as one commit each with the original author.

| Base | `v0.1.44.1` (296b59d2) |
|---|---|

| PR | Why it is needed on a Mac | Status upstream (2026-10-19) |
|---|---|---|
| [#2338](https://github.com/Niko1221/Strata/pull/2338) | wrap a model larger than `maxBufferLength` as overlapping buffers (UD-Q4_K_XL on macOS 15, 64 GB Macs) | open (said to be in 0.1.45 as opt-in) |
| [#2503](https://github.com/Niko1221/Strata/pull/2503) | the conversation memory guard reads available RAM on macOS (it only read `/proc/meminfo`) | open |
| [#2631](https://github.com/Niko1221/Strata/pull/2631) | Metal: conversation cache keeps a conversation across a side request | open |

Together #2503 + #2631 make `--conversation-cache-mib` work on Metal, so a coding agent and its permission classifier
(or a title request) can alternate without re-reading the whole prompt each time (follow-up TTFT 57 s -> 0.5 s at 30K tokens on an M4 Max).

## Build (macOS 26, CLT 26.x)

If more than one Command Line Tools version is installed, pin the SDK the linker matches:

    SDKROOT=/Library/Developer/CommandLineTools/SDKs/MacOSX26.sdk ./setup.sh ...

## Syncing upstream

    git fetch upstream --tags
    git push origin upstream/main:main            # keep main a mirror of upstream
    git rebase --onto <new-tag> v0.1.44.1 mac-stable   # replay our commits on a new release
    # drop any commit whose PR upstream has merged into <new-tag>; update the table above
