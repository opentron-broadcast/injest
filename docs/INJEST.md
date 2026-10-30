# OpenTron Injest — rack ingest server spec

Owner's note, 2026-10-30. Recorded in the MAM roadmap (`opentron-broadcast/opentron-mam`,
ROADMAP.md, "Injest — the rack ingest station", commit 7f3c600).

## The idea in one line

A dedicated machine in the rack with card and drive ports on the front. Stick an SD card
or an SSD in, and it backs up the new files by itself — verified, in more than one place —
then puts videos and photos in the MAM and photos in Lightroom Cloud. Nobody runs a copy
by hand, and no card is ever cleared on trust alone.

## The engine: Bitmatch, forked as Injest

- Instead of ShotPut Pro, use its open-source alternative,
  [Bitmatch](https://github.com/BitmatchApp/Bitmatch): free, MIT-licensed camera-card
  offloading — copy to several drives at once, SHA-256 read-back verification
  (Standard / Thorough / Paranoid modes; Quick is size-only and never earns a safe
  verdict), ASC MHL inventories, a transfer queue, camera detection for Sony, Canon,
  ARRI, RED, Blackmagic, Panasonic, Fujifilm, GoPro, DJI and Insta360, and PDF / CSV /
  JSON reports. Upstream commit at fork time: 1828575.
- Forked 2026-10-30: **[opentron-broadcast/injest](https://github.com/opentron-broadcast/injest)**
  — its own name and, in time, its own icon, as the upstream's BRANDING.md asks of forks.
  Internal studio tool; it is not sold or listed as Bitmatch.
- An inspired feature set, in honor of the hard work of the developers who worked on
  projects like: ShotPut Pro, Bitmatch, Dalet Flex, Axle AI, iconik, and Bynder — with
  similar features and other homages drawn from each of them.

## The rack box

- Rackmount machine whose whole job is ingest — nothing else runs on it.
- Front panel, reachable without pulling the rack: SD slots (UHS-II), the other card
  readers the studio's cameras use, and front SSD / USB-C ports for drives.
- Inside: a fast NVMe landing cache, and a second physically separate drive for copy B
  (two folders on one drive count as one backup, not two — Bitmatch already knows this).
- 10GbE to the studio server / NAS, so copy B does not live on the ingest box for long.
- A UPS, and a status screen: the big review display is where a person confirms a card
  is done (below).

## What happens when a card goes in

1. **Detect and assess.** Identify the card, inventory it, total the bytes, check there
   is room at every destination before copying starts. Flag unreadable or suspicious files.
2. **One question: whose card is this?** Project / show / client / event — auto-suggested
   from date and camera where it can be, one tap to confirm. Everything downstream
   (MAM project, Lightroom album name, gallery title, S3 prefix) hangs off this answer.
   Without it, media lands in a limbo folder someone has to sort by hand.
3. **Duplicate check.** Hash every file and compare against the MAM's archive. A file
   counts as already-backed-up only when the archived asset still has, right now, two
   verified local copies and one verified S3 copy. A matching filename never counts.
   Camera packages stay whole: sidecars, XML, spanned clips, BRAW / XAVC folders.
4. **Copy and verify (the Bitmatch part).** New files copy to the server and to the
   second drive, read back and checked with SHA-256. ASC MHL inventory and a report
   are kept with the copies as the proof of the offload.
5. **Archive to S3.** Originals plus the manifest upload to the designated S3 service
   (through the Hub's named S3 connections), integrity confirmed, object locations
   recorded on the asset. Resumable: if the internet drops, local copies finish and
   the upload picks up later.
6. **Register in the MAM.** Videos and photos become assets. An asset is the logical
   thing; under it sit its instances — server copy, second-drive copy, S3 object,
   proxy, thumbnail — each with its own location, hash and verified status. The MAM
   makes thumbnails and playback proxies (Workhorse does the heavy transcode work).
7. **Photos to Lightroom Cloud.** A new Lightroom album for the ingest; photos go in,
   and the MAM asset records its Lightroom album / asset links. Videos go to the MAM
   only. Editing and selection happen in Lightroom; the MAM stays the truth about
   where the originals live.
8. **Gallery.** The ingest appears as a project gallery of proxies and renditions —
   never public originals. Everything is private at ingest; Private / Client /
   Unlisted / Public is a separate, deliberate publish step afterwards, including
   whether anyone may download originals, proxies, or nothing.
9. **Release the card.** A checklist screen, verified with your own eyes: server copy
   ✓ verified, second drive ✓ verified, S3 ✓ verified, MAM proxies play ✓, Lightroom
   ✓ or explicitly waived, gallery draft ✓. Only then is the card marked
   **"Release for reuse"**. Injest never formats or deletes a card — formatting
   happens in-camera. A stage that has not finished keeps the card un-released.

Every stage is a visible state per card and per asset (Detected → Assessed →
Copied → Verified → Archived → Registered → In Lightroom → Released), and resumable.
Events are facts in a log — CARD_DETECTED, ASSET_VERIFIED, ASSET_REGISTERED — and
Quantum is the workflow engine that reacts to them; Injest does not grow a second one.

## Honest limits and open choices

- **Bitmatch is an app, not a server — yet.** Upstream is Swift / SwiftUI for macOS
  15.5+, iPhone and iPad, with the copy engine in a separate Swift package
  (`Packages/BitMatchEngine`). Two real routes for the rack box, still to choose:
  (a) a rack Mac running Injest as the station app, or (b) a headless service built
  around the engine package, with a small status UI. The front-panel hardware differs
  between the two; decide this before buying the box.
- **Beta, one-person upstream.** Bitmatch says so itself: test with disposable cards
  first, keep the source card until every destination finishes cleanly. Injest keeps
  that rule — the release checklist is the product, not a courtesy.
- **No S3, MAM or Lightroom upstream.** Bitmatch has optional SFTP for an off-site
  copy; S3, MAM registration and Lightroom are Injest's additions, built on what the
  suite already has (the MAM's scope-gated ingest endpoint, Hub S3 connections,
  Quantum flows, Workhorse jobs).
- **Lightroom Cloud is the fragile leg.** Adobe's Lightroom APIs are partner-gated —
  there is no self-serve key. Pragmatic v1: Lightroom on the ingest machine itself,
  auto-importing from the verified folder, so Adobe's own sync creates the cloud
  album. Album creation / linking by API waits on partner access, if it ever comes.
  If Lightroom turns out to be only a pretty album link, the MAM gallery may make
  it redundant — that call stays open.

## Not in v1

- No recording or capture of live SDI / HDMI feeds — that is Movie Recorder /
  HyperDeck work; those files then flow through this same pipeline.
- No automatic card formatting, deletion, or publishing.
- No second workflow engine, and no AI search permissions of its own: media search
  through an assistant runs as the human asking, inside the assets they may see.
