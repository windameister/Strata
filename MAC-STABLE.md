# mac-stable

This is a personal maintenance branch: an upstream release plus the unmerged upstream PRs that a Mac needs.
It is not an official Strata release. Each code change is kept with its original author.

| Base | `v0.2.4` (`96ca4383c7b05a8a3e6064c65d9e9977d4b3bb59`) |
|---|---|

| PR | Why it is needed on a Mac | State at this base (2026-10-24) |
|---|---|---|
| [#2338](https://github.com/Niko1221/Strata/pull/2338) | wrap a model larger than `maxBufferLength` as overlapping buffers | dropped locally: upstream's independent implementation `8b3ec44b` is in the base |
| [#2503](https://github.com/Niko1221/Strata/pull/2503) | the conversation memory guard reads available RAM on macOS | retained; rebased from `3758766` |
| [#2631](https://github.com/Niko1221/Strata/pull/2631) | Metal: keep a conversation's K/V and running state across a side request | retained; rebased from `a3f603e`, including Metal support for the base's borrowed-prefix restore path |

Together #2503 and #2631 make `--conversation-cache-mib` work on Metal. The cache copies the Metal window's own
buffers, checks their sizes and keeps the donor parked when another conversation borrows a prefix. The caller
restores that prefix's checkpoint separately. CUDA/HIP/SYCL keep the upstream path.

The previous branch measured follow-up TTFT of about 0.5 s rather than 57 s at 30K tokens on an M4 Max when a coding
agent alternated with a side request. That is a historical measurement, not a result for this new base. Repeat the
main/side/main check after restarting the updated engine before claiming the same performance here.

## Validation of this base

Measured on 2026-10-24: M4 Max, 128 GB, macOS 26.7.1, Metal engine `0.1.48-metal`. Four synthetic OpenAI-compatible
streaming requests ran in sequence, with thinking off and no project data. TTFT below is client wall time,
including the tailnet round trip.

| Request | Input tokens | Cached tokens | TTFT |
|---|---:|---:|---:|
| Main, cold | 29,462 | 0 | 49.168 s |
| Unrelated short side request | 57 | 0 | 0.743 s |
| Side request sharing the main system prefix | 29,466 | 29,443 | 0.434 s |
| Main follow-up after both side requests | 29,488 | 29,466 | 0.463 s |

The engine logged a borrowed 29,443-token prefix in 17.6 ms, keeping its donor parked, then restored the main
conversation's 29,466 tokens in 10.2 ms. This exercised the Metal borrowed-prefix adapter as well as a full restore.
An existing Claude-harness worker also returned its expected ping through the updated API.

The Metal build, 4,772 standalone cache checks, 31 memory-guard checks and a live macOS memory reading passed.
The API reported a loaded IQ3_S model and a 262,144 context limit. This was one short cache-validation sequence,
not a quality comparison or a full-window stress test. CUDA, HIP and SYCL builds were not run on this Mac.

## Build (macOS 26, CLT 26.x)

If more than one Command Line Tools version is installed, pin the SDK the linker matches:

    SDKROOT=/Library/Developer/CommandLineTools/SDKs/MacOSX26.sdk ./update.sh --yes --model IQ3_S

`update.sh` first runs `git pull --ff-only`; use it only after the branch and its tracking target are correct.
For an upgrade pinned to a tag plus these patches, prevent that pull from replacing the prepared history. The
Mac update path installs dependencies and rebuilds the engine, then returns before downloading/preparing models
or starting the server. It does not rewrite the existing run config or draft subset in this mode; check the
installed config's arguments against the new engine's help. Restart the serving process separately in an idle
window. Building a new binary does not upgrade a process already running the old one.

## Syncing upstream

    git fetch upstream --tags
    git rebase --onto <new-tag> v0.2.4 mac-stable
    # drop a local patch once the target release has the same fix; update the table above

Keep `main` an upstream mirror. Review the rewritten `mac-stable` commits and validate the running engine before
pushing; do not merge the maintenance patches into `main`.
