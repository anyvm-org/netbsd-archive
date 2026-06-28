# netbsd-archive

Re-hosted **NetBSD archived release images** for the
[anyvm-org](https://github.com/anyvm-org) image builders.

## Why this repo exists

A few older NetBSD releases (currently **9.0 / 9.1 / 9.2**) are no longer on the
primary mirrors: `ftp.netbsd.org` and `cdn.netbsd.org` only keep *current*
releases and return `404` for these, so the single upstream source left is
[`archive.netbsd.org`](https://archive.netbsd.org/pub/NetBSD-archive/).

`archive.netbsd.org` **rate-limits data-center / CI IP ranges**: it serves only
about the first 9 MiB of a file and then returns `HTTP 402 Payment Required` for
every following (and resumed) request. GitHub Actions runners hit this almost
immediately, so the `*-builder` workflows cannot download these images during CI
-- the build now fails fast and loud with `FATAL: download stalled ...` rather
than silently shipping a truncated image.

This repo mirrors those images as **GitHub release assets**, which CI can fetch
quickly and without throttling, so the builders stay green.

## What is hosted here

Byte-for-byte, **unmodified** copies of the upstream files under
`archive.netbsd.org/pub/NetBSD-archive/`. Nothing is repackaged or patched --
this is a download-reliability mirror, not a fork. See the
[releases](../../releases) page for the authoritative list of assets.

Suggested asset naming (keep the original compression suffix, e.g. `.img.gz`, so
the builder picks the right decompress path):

| Release | Asset | Upstream path |
|---------|-------|---------------|
| 9.0 evbarm-aarch64 | `NetBSD-9.0-evbarm-aarch64-arm64.img.gz` | `NetBSD-9.0/evbarm-aarch64/binary/gzimg/arm64.img.gz` |
| 9.1 evbarm-aarch64 | `NetBSD-9.1-evbarm-aarch64-arm64.img.gz` | `NetBSD-9.1/evbarm-aarch64/binary/gzimg/arm64.img.gz` |
| 9.2 evbarm-aarch64 | `NetBSD-9.2-evbarm-aarch64-arm64.img.gz` | `NetBSD-9.2/evbarm-aarch64/binary/gzimg/arm64.img.gz` |

## How the builders use it

Point the builder conf's image link at a release asset here instead of
`archive.netbsd.org`, e.g.:

```sh
VM_VHD_LINK="https://github.com/anyvm-org/netbsd-archive/releases/download/<tag>/NetBSD-9.0-evbarm-aarch64-arm64.img.gz"
```

## Adding a new image

1. Download the file from `archive.netbsd.org` on a **non-throttled host** (a
   home / non-cloud IP). Use resume to get past the ~9 MiB cap:
   `curl -L -C - --retry 10 --retry-all-errors -o <file> <url>`
2. Verify it against NetBSD's published `SHA512` for that file before uploading.
3. Upload it as an asset on a release in this repo, keeping a descriptive name
   (release + arch + original filename) so its source is obvious.
4. Point the builder conf's image link at the new asset.

## License / provenance

These are unmodified NetBSD distribution images. NetBSD is distributed under the
[2-clause BSD license](https://www.netbsd.org/about/redistribution.html); the
original copyright and distribution terms apply unchanged. This repository only
re-hosts the bytes for download reliability on CI.
