# netbsd-archive

Re-hosted **NetBSD archived release images** for the
[anyvm-org](https://github.com/anyvm-org) image builders.

## Why this repo exists

Older NetBSD releases leave the primary mirrors once they are superseded:
`ftp.netbsd.org` and `cdn.netbsd.org` keep only *current* releases and answer
`404` for the rest, so the single upstream source left is
[`archive.netbsd.org`](https://archive.netbsd.org/pub/NetBSD-archive/). That
has happened to **9.0 / 9.1 / 9.2** (2026-06) and to **10.0 / 10.1** (found
2026-10-20, when every 10.0/10.1 build of netbsd-builder failed).

`archive.netbsd.org` **does not serve automated downloads**. As observed on
2026-10-20, from a GitHub runner and from a Hong Kong VPS alike:

- every request, including a plain non-Range `GET`, delivers at most the first
  1 MiB of the file and then drops the connection;
- any Range request past that point is answered `HTTP 402 Payment Required`
  with an HTML page titled "trivial botcatcher" asking "what are you here
  for?";
- `HEAD` is answered normally (`200` and the real `Content-Length`), so a link
  check passes while the download itself cannot succeed.

(In 2026-06 the cap was about 9 MiB and resuming from a home IP got past it.
On 2026-10-20 resuming did not get past it from either host above; a home IP
was not tried.) The `*-builder` workflows therefore cannot download
these images during CI -- the build fails fast and loud with
`FATAL: download failed ... download stalled at 1048576/...` rather than
silently shipping a truncated image.

The botcatcher is deliberate, and nothing here tries to get past it. The bytes
come from official NetBSD mirrors instead (see below), and this repo keeps them
as **GitHub release assets**, which CI can fetch quickly, so the builders stay
green.

## What is hosted here

Byte-for-byte, **unmodified** copies of the upstream files under
`archive.netbsd.org/pub/NetBSD-archive/`. Nothing is repackaged or patched --
this is a download-reliability mirror, not a fork. See the
[releases](../../releases) page for the authoritative list of assets.

Suggested asset naming (keep the original compression suffix, e.g. `.img.gz`, so
the builder picks the right decompress path):

| Release | Asset | Upstream path | Verified against |
|---------|-------|---------------|------------------|
| 9.0 evbarm-aarch64 | `NetBSD-9.0-evbarm-aarch64-arm64.img.gz` | `NetBSD-9.0/evbarm-aarch64/binary/gzimg/arm64.img.gz` | uploaded by hand |
| 9.1 evbarm-aarch64 | `NetBSD-9.1-evbarm-aarch64-arm64.img.gz` | `NetBSD-9.1/evbarm-aarch64/binary/gzimg/arm64.img.gz` | uploaded by hand |
| 9.2 evbarm-aarch64 | `NetBSD-9.2-evbarm-aarch64-arm64.img.gz` | `NetBSD-9.2/evbarm-aarch64/binary/gzimg/arm64.img.gz` | uploaded by hand |
| 10.0 amd64 | `NetBSD-10.0-amd64.iso` | `NetBSD-10.0/images/NetBSD-10.0-amd64.iso` | official `SHA512` |
| 10.0 sparc64 | `NetBSD-10.0-sparc64.iso` | `NetBSD-10.0/images/NetBSD-10.0-sparc64.iso` | official `SHA512` |
| 10.0 evbarm-aarch64 | `NetBSD-10.0-evbarm-aarch64-arm64.img.gz` | `NetBSD-10.0/evbarm-aarch64/binary/gzimg/arm64.img.gz` | size + `gzip -t` |
| 10.1 amd64 | `NetBSD-10.1-amd64.iso` | `NetBSD-10.1/images/NetBSD-10.1-amd64.iso` | official `SHA512` |
| 10.1 sparc64 | `NetBSD-10.1-sparc64.iso` | `NetBSD-10.1/images/NetBSD-10.1-sparc64.iso` | official `SHA512` |
| 10.1 evbarm-aarch64 | `NetBSD-10.1-evbarm-aarch64-arm64.img.gz` | `NetBSD-10.1/evbarm-aarch64/binary/gzimg/arm64.img.gz` | size + `gzip -t` |

Every image uploaded by the sync workflow has a `<asset>.sha512` sidecar in
`sha512sum -c` format, holding the digest that was verified before upload.

**9.3 and 9.4 are not here.** They are archived too, but on 2026-10-20 no
mirror on NetBSD's mirror list still carried them, so the only copy is behind
the botcatcher above.

## How the builders use it

Point the builder conf's image link at a release asset here instead of
`archive.netbsd.org`, e.g.:

```sh
VM_VHD_LINK="https://github.com/anyvm-org/netbsd-archive/releases/download/<tag>/NetBSD-9.0-evbarm-aarch64-arm64.img.gz"
```

## Adding a new image

An archived release never changes, so an image only has to be synced once.

1. Add a `{ path, asset, checksum }` entry to the matrix in
   `.github/workflows/sync.yml`. Keep the upstream filename for `images/`
   files (they already carry release and port); prefix gzimg files with
   `NetBSD-<rel>-<port>-`.
2. Run the **Sync archived NetBSD images** workflow (`workflow_dispatch`) with
   the release tag to publish under (`v0.0.1`). For each entry it:
   - takes the authoritative size from `archive.netbsd.org` (`HEAD` works);
   - downloads the bytes in one resumable stream from an official mirror that
     still carries the release in its main tree (`ftp.uni-erlangen.de`, then
     `ftp.fau.de`), skipping any mirror whose size differs;
   - verifies the file: `checksum: official` must match the `SHA512` file
     NetBSD publishes next to it (`images/SHA512`); `checksum: gzip` is for
     evbarm gzimg files, for which NetBSD publishes no checksum at all, and
     requires the exact size plus `gzip -t`;
   - uploads the image and its `.sha512` sidecar.

   Images already on the release are skipped, so only new entries download
   anything.
3. Point the builder conf's image link at the new asset.

Mirrors drop a superseded release on their own schedule, so sync a release
**as soon as** it moves to the archive. Once no mirror carries it any more,
it can no longer be added here (that is what happened to 9.3 / 9.4).

## License / provenance

These are unmodified NetBSD distribution images. NetBSD is distributed under the
[2-clause BSD license](https://www.netbsd.org/about/redistribution.html); the
original copyright and distribution terms apply unchanged. This repository only
re-hosts the bytes for download reliability on CI.
