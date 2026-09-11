# NetPulse — Releases

This repository is where published NetPulse builds live. **There is no source
code here.** NetPulse itself is developed in a private repository; this one is
public for one reason: a shop's computer has to be able to ask "is there a
newer version?" and download it without signing in to anything.

Two products, two version streams. A counter PC and a terminal are updated
independently and their version numbers have nothing to do with each other.

```
client/latest.json     the terminal   (NetPulse.exe)
server/latest.json     the counter    (NetPulse-Server.exe)
```

Each release is a GitHub release in this repository, tagged `client-vX.Y.Z` or
`server-vX.Y.Z`, carrying the built `.exe` as its only asset.

## What a manifest says

```json
{
  "product": "client",
  "version": "1.2.0",
  "released": "2026-09-11",
  "notes": "What changed, written for somebody behind a counter.",
  "url": "https://github.com/Ralph313-creator/NetPulse-Releases/releases/download/client-v1.2.0/NetPulse.exe",
  "size": 14680064,
  "sha256": "b1946ac92492d2347c6235b4d2611184..."
}
```

NetPulse fetches its own product's manifest, compares `version` against the
build it is running, and offers the update if it is newer. Pressing **Install**
downloads `url`, refuses anything whose size or SHA-256 does not match this
file, swaps the new program in beside the old one and restarts.

A manifest at version `0.0.0`, or one with no `url`, means nothing has been
published for that product yet. NetPulse reads that as "you are up to date"
rather than as an error, so a shop never sees a failure because a release has
not been cut.

## What this protects, and what it does not

The manifest and the download both come from GitHub over HTTPS, and NetPulse
refuses to download from any other host. The SHA-256 in the manifest is checked
against the bytes that arrive, so a download that is truncated, cached wrongly,
or tampered with in transit is thrown away rather than installed.

What that does **not** cover: anyone who can write to this repository can
publish a build that every NetPulse in every shop will install. The repository's
write access is the whole of the trust here. Keep it to the accounts that
actually publish, and turn on two-factor authentication for them.

## Publishing a build

From a checkout of the private NetPulse repository:

```sh
scripts/publish-release.sh client 1.2.0 path/to/NetPulse.exe "What changed."
scripts/publish-release.sh server 1.4.1 path/to/NetPulse-Server.exe "What changed."
```

That script builds nothing. It takes an executable you have already built and
tested, computes its SHA-256, creates the GitHub release here, uploads the
executable, and rewrites the matching `latest.json` in this repository. Nothing
is offered to a shop until that last step lands, so an upload that fails halfway
leaves every shop on the version it already has.

The version you pass has to be **higher than the one in `latest.json`**, or no
shop will be offered it — NetPulse only ever moves forwards.
