# OpenTron Injest — feature-rich plan

Conforms to opentron-design-system SPEC.md rev 50.
Plan written 2026-10-30, from the owner's workflow description and a sourced research
pass over ShotPut Pro / ShotPut Studio and the open-source ingest field (sources at the
end). This plan sits on top of INJEST-SPEC.md: the spec says what Injest is; this plan
says which features it gets, where each one comes from, and in what order.

An inspired feature set, in honour of the hard work of the developers who worked on
projects like: ShotPut Pro, Bitmatch, Ingesto, Dalet Flex, Axle AI, iconik, and
Bynder — with similar features and other homages drawn from each of them.

## 1. Two corrections the research forced

These change the earlier conversation, so they go first.

- **ShotPut Pro has no S3 and no transcoding.** Verified on Imagine Products' own
  pages and manuals: Pro ($169 perpetual) is local verified copy only. S3 upload,
  transcoding and Automation Pipelines are **ShotPut Studio** ($299 perpetual,
  Mac-only in the vendor's listings). Any plan that says "Pro, plus S3 later" is
  wrong — the cloud leg was always a Studio feature or a build of our own.
- **No tool — commercial or open source — does hash-based duplicate detection
  against an existing archive.** ShotPut Pro skips a file as a "duplicate" when the
  same name exists at the destination with the same modification date and byte size;
  it never hashes the archive. Bitmatch and Ingesto notice a card or file they have
  copied before; rclone skips new-or-changed files on a sync. The owner's central
  requirement — "check it for duplicates of already existing work" across the whole
  archive, by content — **has to be built**. It is Injest's headline feature, not a
  feature it inherits.

## 2. What ShotPut actually does (the feature bar)

From Imagine Products' 2026 manual and product pages, read 2026-10-30.

| Area | ShotPut Pro | ShotPut Studio adds |
| --- | --- | --- |
| Copy | Reads the source once, copies to several destinations simultaneously; cascaded copies (destination 1 → destination 2) are sequential | Same |
| Checksums | XXHash-64, XXHash3-64, XXHash-128, MD5, SHA-1, SHA-2 256, SHA-2 512, C4, CRC-32 (2026 builds add CRC-32C); xxHash3-64 is the default; verification types are File Size–Source Checksum, Full Checksum read-back, File Size Only; optional second source read (Source Integrity Verification) | Same |
| Seals | ASC MHL (default) and legacy MHL v1, saved inside each destination | Same, plus a standalone MHL tool |
| Reports | PDF, TXT, CSV; standalone, wrap (several jobs, one report) and project reports; the PDF is video-aware — up to 10 thumbnails per clip plus metadata from 20+ formats, 4 layouts, thumbnail LUTs | Same, standalone reporting tool |
| Presets | Simple and Preset modes; multi-destination presets; several presets can run as one job; importable, duplicable, colour-coded | Copy and Transcode presets |
| Naming | Token-based output naming with nested folders, custom tokens prompted at job start, global tokens; file renaming with an Original File Name token and extension filtering | Same |
| Automation | Queue automation: auto-add drives, begin the job on entry, auto-eject on completion — "just insert a card, and it will appear in the queue and begin copying automatically"; job sequencing, pause/resume; notifications (desktop, email/SMS on an active update plan, Slack webhooks) | **Automation Pipelines**: chained Copy → Transcode → Report presets, triggered on card insertion |
| Cloud | **None as a destination** (2026 builds add upload *monitoring* for Google Drive/iCloud/Dropbox — that is not S3) | **S3 upload** (built-in provider list, several buckets; cloud is a destination only, never a source); Frame.io, MASV, LucidLink, Axle AI integrations |
| Transcoding | None | Transcode tool: LUTs, burned-in timecode, watermarks; certified ProRes RAW engine |
| Other | Drive Health (SMART), speed benchmarking, drive indexing, projects workspaces | Same class of tooling, plus job history |
| Platform / price | macOS 13–27 and Windows 11/10; **$169 perpetual**, updates $59/yr auto-renew or $70/yr pay-as-you-go (first 12 months included), 30-day rental $60 | macOS only in vendor listings; **$299 perpetual**, updates $99/yr, 30-day rental $99 |

Two limits that matter for us: **no public CLI, API or watch-folder was found** for
either product — automation is the GUI queue and Studio's GUI pipeline builder. So
ShotPut cannot be driven by Quantum. It can be a reference, a fallback, or an interim
purchase; it cannot be Injest's engine. That is the practical reason the fork exists.

What ShotPut Pro is used for in the wild (the owner's question): on-set camera-card
offloads by a DIT / data wrangler, event and wedding photography backup before cards
are reused, documentary fieldwork where footage cannot be re-shot, editor hand-offs
with a report as proof of the offload, and recorder-card offloads for audio — always
the same shape: verified copies on two drives, a seal and a report, then the card is
released.

## 3. The open-source field

| Tool | Licence | Platforms | Verified copy | Simultaneous | S3 | Notes for Injest |
| --- | --- | --- | --- | --- | --- | --- |
| **Bitmatch** | MIT | macOS 15.5+ (iPhone/iPad build from source) | SHA-256 read-back; Thorough adds MD5 + source re-read; Paranoid compares bytes; Quick is size-only and never earns a safe verdict | Yes — card read once, written to all drives | No — optional SFTP only | **Our fork's base.** ASC MHL, PDF/CSV/JSON reports, transfer queue, camera detection (Sony, Canon, ARRI, RED, Blackmagic, Panasonic, Fujifilm, GoPro, DJI, Insta360). Beta, one-person project — its own README says test with disposable cards first |
| **Ingesto** | GPL v3 (per its site; its repository was not directly verified) | macOS, Windows, Linux | xxHash64/128 and MD5 read-back; a second source read in its Pro mode | Yes | No — a post-ingest hook can call external tools such as rclone | The cross-platform reference: folder-name token templates, kiosk mode, per-format filters, ASC MHL v2.0 validated against the ASC reference tool, HTML/CSV/JSON log on every destination. **GPL: study its behaviour, do not copy its code into the MIT fork** |
| **rclone** | MIT | Cross-platform CLI | MD5/SHA-1 on transfers; `check` for hash equality; `hashsum` incl. SHA-256/512, BLAKE3, xxHash | No — one run, one destination; runs can be chained | **Yes — its core purpose** (Amazon S3 and many S3-compatible stores) | **The S3 leg.** By far the most mature project here (60k+ stars, 133 releases). No card awareness, no renaming tokens, no MHL, no reports — it is a leg, not the station |
| **Rapid Photo Downloader** | GPL-3.0 | Linux | **None — no checksum verification at all** (an open feature request) | Parallel sources, backup during download | No | Borrow its *ideas*: auto-download on insertion, strong renaming from metadata, job codes. Single developer, documentation last updated Feb 2024. Never the verified-copy leg |
| **digiKam** (import) | GPL-2.0 | Linux, Windows, macOS | **None for copies** | No | No | A photo manager, not an offload tool; duplicates by perceptual fingerprint, not hashes. Its GitHub repo is an archived mirror — the live project is KDE's. Reference for photo organisation only |
| **ascmhl** (ASC reference) | MIT | Cross-platform, Python | Creates/verifies manifests: xxHash64, XXH3-64/128, MD5, SHA-1, C4 | Does not copy files | No | Use to *validate* our MHL output, not as a production engine: the implementation is explicitly incomplete (status dated 2022-09) and looks inactive |

**Not open source — do not let these into the plan mislabelled:** Hedge / OffShoot
(proprietary, about $169 / $249 Pro, S3 and API are Pro tier), Silverstack
(proprietary subscription), YoYotta (proprietary Mac subscription, €95/yr on its
pricing page), mmMover (free but "source available, all rights reserved"), Copsy
(free, its own site says it is not open source), DaVinci Resolve's Clone Tool (free,
closed).

**Building blocks (all open source):** ExifTool — read/write metadata across many
formats (EXIF, GPS, IPTC, XMP, MakerNotes). ffprobe — scriptable stream/format
inspection; FFmpeg — the transcode/proxy step. MediaInfo — unified technical data
(container, codec, frame rate, colour space, bit depth), exportable. xxHash —
extremely fast non-cryptographic hashes (XXH64, XXH3, XXH128), the practical
checksum for ASC MHL.

## 4. The stack Injest is built from

- **Copy engine: the Bitmatch fork** (`opentron-broadcast/injest`, MIT). It already
  does the hard, boring, dangerous part — simultaneous verified copy, seals,
  reports, queue — and its engine is a separate Swift package.
- **S3 leg: rclone**, driven by Injest as a separate process, against the Hub's
  named S3 connections. Credentials are vault references only (SPEC UI-XAPP-02);
  Injest's settings show vault status, never a key field.
- **MHL validation: ascmhl** in the test suite only.
- **Inspection: ffprobe + MediaInfo + ExifTool** — capture date, camera, format,
  dimensions, duration, audio — feeding both the assessment screen and the MAM
  record.
- **Hashes: SHA-256 is the archive hash of record** (it is what Bitmatch verifies
  with, and what rclone can check). xxHash3 is the fast first-pass hash where speed
  matters and a collision would only cost a re-check, never a wrong "duplicate".
- **Ingesto is the behaviour reference** for the two things Bitmatch lacks and we
  need on a rack box: kiosk mode and token-based folder naming. Reference, not code.
- **Orchestration: Quantum.** Injest emits events; Quantum runs the flows;
  Workhorse executes heavy jobs (proxies, transcodes, transcription). Injest does
  not grow a second workflow engine, and the MAM does not either.

## 5. The feature plan, in phases

### Phase 0 — Fork hygiene and family registration
Rename and re-icon the fork (upstream's BRANDING.md asks forks for their own name
and icon — Injest already has the name), keep the MIT licence and Bitmatch credit,
set an upstream-sync policy, add the conformance line citing SPEC rev 50, claim
**port 4850** for Injest in the SPEC's second block (UI-XAPP-01: after Presenter
4840, the next product takes 4850; its extra listeners take 4851 onwards) — the
SPEC records it before anything ships on it. UPPER-CASE core docs, the "Works with
the other OpenTron apps" cross-link table, the `opentron` topic (UI-DOC-01–04).

### Phase 1 — The station core (ShotPut parity where it counts)
- Card and drive detection on insertion for the cameras the studio owns, with a
  queue: several cards waiting, each a row with its own progress and verdict.
- **Assess before copying**: inventory, total bytes, per-destination free-space
  check, format/camera/duration inspection, unreadable or suspicious files flagged
  by name — never silently skipped.
- **The one question**: project / show / client / event, auto-suggested from date
  and camera, one tap to confirm. Everything downstream hangs off it.
- **Presets**, in ShotPut's sense: named, colour-coded, duplicable sets of
  destinations + verification mode + naming scheme; several can run as one job;
  a preset is chosen per project and remembered.
- **Token naming**: output folders and file names built from tokens (project, card
  name, camera, date, counter, custom tokens prompted at job start) — original
  names kept by default; renaming is opt-in per preset, and camera packages and
  sidecars always stay whole.
- Pause/resume, including after a disconnect; a job interrupted is a job resumed,
  never restarted from zero without saying so.

### Phase 2 — Verified copy (the Bitmatch core, kept honest)
- Simultaneous copy to the server landing area and the second physical drive,
  read back and verified. Two folders on one drive count as one backup — the UI
  says so when it sees it.
- Verification modes named plainly, following Bitmatch's ladder: **Quick**
  (sizes only — and the card can never be released on a Quick job), **Standard**
  (SHA-256 read-back, the default), **Thorough** (adds MD5 and a fresh source
  read), **Paranoid** (byte-for-byte). Irreplaceable field cards default to
  Thorough.
- ASC MHL seal written into every destination, plus a report (PDF/CSV/JSON) with
  thumbnails, in the studio's brand from the Hub brand kit.
- Every stage is a visible, resumable state per card: Detected → Assessed →
  Copied → Verified → Archived → Registered → In Lightroom → Released. A failed
  stage names the stage and what to do next; nothing shows green on a guess
  (UI-COPY-03, UI-CMP-04 — grey means no data, never a claim).

### Phase 3 — The archive hash index and true duplicate detection (the headline build)
- The MAM keeps a **hash index**: every archived file's SHA-256, its asset, and the
  verified state of each of its instances (server, second drive, S3), kept current
  by re-verification on a schedule and whenever a volume mounts.
- At assessment, each incoming file is looked up by hash **before** copying:
  - **New** — copy it.
  - **Already archived and fully protected** (two verified local copies and one
    verified S3 copy, checked now, not remembered) — skip the copy, and link the
    existing asset to this card's project/gallery as well. A duplicate can belong
    to two projects without a second original.
  - **Archived but under-protected** (a copy has since failed or a drive is gone)
    — it is not a duplicate, it is the only copy: re-copy it and say why.
- Filename matches are shown as a hint and never used as the decision.

### Phase 4 — Second copy off the box, and S3
- The second-drive copy is swept to the studio server / NAS over 10GbE, so both
  local copies do not stay on the ingest box.
- S3 upload via rclone to the designated bucket (Hub connection), originals plus
  the MHL manifest; integrity confirmed with rclone's hash check; object locations
  recorded on the MAM asset. Originals and proxies may go to different buckets,
  as Studio does.
- Resumable end to end: internet drops slow the archive leg, never the local one.
  The status screen distinguishes "local copies verified", "cloud archive
  verified" and "Lightroom/gallery ready" as three separate facts.

### Phase 5 — Into the MAM
- Registration through the MAM's scope-gated ingest endpoint, machine-scoped token
  from the vault. An **asset is the logical thing**; its instances are the copies,
  each with location, hash and verified status.
- Metadata from ffprobe/MediaInfo/ExifTool lands on the asset at registration —
  not re-typed later by a person.
- Thumbnails and playback proxies (H.264, ≤720p class) made by Workhorse;
  transcription by Workhorse's whisper path where the material needs it; Quantum
  flows subscribe to `ASSET_REGISTERED` and run those jobs.
- The gallery is built from proxies and renditions only. Originals are never
  public files.

### Phase 6 — Lightroom Cloud (photos)
- A new Lightroom album per ingest (named from the project answer), photos added,
  and the MAM asset records its Lightroom album/asset links both ways.
- Honest route, in order: (a) partner API if Adobe ever grants it; (b) v1 working
  route — Lightroom on the ingest machine auto-importing the verified photo
  folder, Adobe's own sync making the cloud album; (c) if Lightroom ends up being
  only a pretty album link, the MAM gallery may replace it — the owner's call,
  and the plan does not depend on it.
- Videos go to the MAM only. Lightroom is the photo editing/selection client;
  the MAM stays the truth about where originals live, and per-field ownership
  applies: Lightroom owns rating/edit state, the MAM owns archive/verification
  state, Workhorse/AI owns transcripts and tags, people own title and rights.

### Phase 7 — Gallery and visibility
- Every ingest creates a **private** project gallery draft automatically.
- Publishing is a separate, deliberate step with four choices, in plain words:
  **Private** (only you and authorised staff), **Client** (invited people or a
  protected link), **Unlisted** (anyone with the link — the link *is* the key,
  and the screen says so), **Public** (everyone).
- Per-gallery download rights: originals, proxies, or nothing. Publishing exposes
  viewing copies; the archive stays private either way.
- Brand separation applies: a government-affairs client's gallery lives in that
  client's world, with no media-world modules around it.

### Phase 8 — Review, release, reports
- The release screen is the product: verification results and previews side by
  side — server copy ✓, second drive ✓, S3 ✓, MAM proxies play ✓, Lightroom ✓
  or explicitly waived, gallery draft ✓. Checksums prove the copies match; eyes
  prove it is the right material.
- The final button reads **"Release card for reuse"**, and its confirmation states
  the consequence first (UI-COPY-04): this card's material is verified in three
  places; formatting happens in-camera. Injest never formats or deletes a card,
  and an incomplete required stage keeps the card un-released.
- Reports: the offload report (PDF with thumbnails, CSV, JSON) in the studio
  brand, the MHL seal, and a diagnostics export for bug reports that carries no
  footage names or paths (Bitmatch's own practice, kept).
- Notifications when a job finishes or fails — in the Hub first; email/Slack
  are later, and never carry credential material.

### Phase 9 — Automation and the event log
- Insert-to-start is the default posture (Queue automation parity): a known
  preset + a recognised card starts without a click; an unrecognised card asks
  the one question and waits.
- Events are facts, past tense, in an audit log: `CARD_DETECTED`,
  `INGEST_STARTED`, `ASSET_VERIFIED`, `ASSET_REGISTERED`,
  `LIGHTROOM_LINKED`, `CARD_RELEASED`. Quantum subscribes; chains are guarded
  by the per-asset state machine, so a metadata change cannot trigger itself
  into a loop.
- Two auth planes, kept apart: machine-scoped tokens for Injest/Workhorse, and
  human identity for any assistant/MCP search — search runs as the person
  asking, inside the assets they may see, never as a super-user.
- Later, if wanted: drive-health (SMART) warnings and speed benchmarking
  (ShotPut has both), and a Kiosk module (per UI-MOD) showing the queue on the
  studio dashboard.

### Phase 10 — The rack hardware
- Rackmount host with front-panel SD (UHS-II) and the other card readers the
  studio's cameras use, front SSD bays / USB-C for drives, a fast NVMe landing
  cache, a physically separate second drive, 10GbE to the server, a UPS, and a
  status display at the rack.
- **The open decision that gates the purchase**: a rack Mac running Injest as
  the station app (Bitmatch's native ground), or a headless service built
  around the engine package with a small status UI. Ingesto proves a
  cross-platform kiosk-shaped ingest is possible, but its GPL licence keeps it
  a reference, not a base. Decide this before buying the box; the front panel
  differs between the two.

## 6. How this fits OpenTron design standards

Checked against the live SPEC, rev 50 (2026-10-25) — not a cached copy.

- **Conformance**: the repo carries "Conforms to opentron-design-system SPEC.md
  rev 50" (UI-CONF-01), and build briefs cite rule IDs (UI-CONF-02).
- **Port**: 4850, second block, ten per product (UI-XAPP-01). Recorded in the
  SPEC before it is shipped on.
- **Tokens, no hard-coded colours**: navy `#1d2133` → `#2a2540`, accent
  `#2e9bf0`, success `--ot-ok`, warning `--ot-warn`, error `--ot-error`
  (UI-TOK-01–04). `--ot-onair` red is reserved for on-air/recording and is
  **never** an ingest error colour.
- **Honest status**: dots per UI-CMP-04; grey is no data, green is a probe that
  answered, and the word "verified" appears only when a read-back actually ran
  (UI-COPY-03). A Quick job can never display as verified.
- **Plain copy, British spelling** in docs and UI copy (colours, grey, licence),
  engineer terms introduced once in plain words (UI-COPY-02); spoken/script
  copy in complete sentences (UI-COPY-01); confirmations state the consequence
  before the action (UI-COPY-04) — the card-release confirmation is the worked
  example.
- **Vault only**: S3, MAM and any other credentials are vault references; the
  settings screen uses the shared vault field and shows status, never a key
  (UI-XAPP-02).
- **Brand from the Hub**: name, logo and colours read from the Hub brand
  (`GET /api/brand`), falling back to the tokens; reports carry the studio's
  brand, and after a custom brand is applied no stray OpenTron chrome remains
  (UI-XAPP-04/05).
- **Studio Bar** on every operator window, with the connection sentence and
  location path naming the instance (UI-XAPP-08, UI-BAR-01 onwards); never on
  an output.
- **Updater behaviour** matches the family: verified releases only, with
  confirmation, progress, success and failure states (UI-XAPP-03).
- **Direct links** resolve through the workspace, never hard-coded IPs
  (UI-XAPP-07); the MAM, Hub, Quantum and Workhorse links on an asset resolve
  the same way in every app.
- **AI features carry the AI indicator** (UI-AI-01–05): transcript and tag
  suggestions are labelled proposals a person accepts, never silent facts.
- **No dead ends** (UI-CMP-10): every control acts or explains — a stage that
  cannot run says which stage, why, and what to do next.
- **Docs**: UPPER-CASE core docs, cross-link table, `opentron` topic, both
  changelogs kept (UI-DOC-01–04); manuals in English, Spanish and Arabic on
  the family's cadence once there is something to manual.

## 7. Risks, named plainly

- Bitmatch is beta software from a one-person project (created Dec 2025). The
  fork inherits that: test with disposable cards, keep source cards until every
  destination finishes cleanly, and expect to harden the engine ourselves.
- Ingesto's repository and GPL v3 licence file were not directly verified in
  this research — its claims rest on its site. Treat it as unproven until
  someone reads the repo.
- ascmhl is an incomplete, inactive-looking reference implementation: a
  validator, never an engine.
- Rapid Photo Downloader and digiKam have **no** checksum-verified copy; either
  one as the copy leg would quietly delete the product's core promise.
- Lightroom's partner gate is the least controllable dependency in the plan;
  Phase 6 is ordered so nothing else waits on it.
- ShotPut itself cannot be automated (no public CLI/API found), so "just drive
  ShotPut from Quantum" is not a fallback architecture. Buying Studio ($299)
  remains a sane *interim* station while Injest is built — a purchase decision
  for the owner, not assumed in this plan.

## 8. Decisions for the owner

1. Rack Mac running the app, or headless engine service? (Gates the hardware.)
2. Which S3 service/bucket is "the designated S3" for originals, and do
   proxies get their own bucket?
3. Default verification: Standard for everything, Thorough for field cards —
   or Thorough for everything, at the cost of ingest time?
4. Buy ShotPut Studio ($299 perpetual, vendor's site) as an interim station,
   or go straight to building Injest?
5. Lightroom: pursue the auto-import v1 now, or park Phase 6 until the MAM
   gallery proves whether it is needed at all?

## Sources

- Imagine Products — ShotPut Pro Mac 2026.3.1 manual; ShotPut Pro for Mac and
  for Windows product pages (pricing read on the vendor's pages, 2026-10-30);
  ShotPut Studio for Mac product page; checksum support list (2023-05-09);
  S3 FAQ; Studio 2.0 press release (2025-04-02); automation and S3 blog posts
  (2025-07-11, 2025-11-03, 2025-11-19). imagineproducts.com.
- Bitmatch — github.com/BitmatchApp/Bitmatch (README, releases), read 2026-10-30.
- Ingesto — just-edit.fr/tools/ingesto.html (site crawled 2026-09-26; repository
  not directly verified).
- Rapid Photo Downloader — github.com/damonlynch/rapid-photo-downloader and
  damonlynch.net/rapid/documentation.
- digiKam — github.com/KDE/digikam (archived mirror) and docs.digikam.org
  import documentation.
- ascmhl — github.com/ascmitc/mhl.
- rclone — github.com/rclone/rclone, rclone.org `check` and `hashsum` docs.
- YoYotta pricing — yoyotta.com/yoyotta/pricingEUR.html; mmMover —
  mmmover.video; Copsy — copsy.app.
- ExifTool — exiftool.sourceforge.net; FFmpeg — ffmpeg.org; MediaInfo —
  mediaarea.net; xxHash — github.com/Cyan4973/xxHash.
- OpenTron design system — SPEC.md rev 50 (2026-10-25), read live 2026-10-30.
- Full research notes: the Injest research report saved with this plan's
  working files (2026-10-30).
