# Injest — design notes

Conforms to opentron-design-system SPEC.md rev 51.

OpenTron Injest is the family's rack ingest station: a fork of the open-source
Bitmatch camera-card offload app (MIT), renamed and re-iconed as the upstream's
BRANDING.md asks of forks, with Bitmatch credited in the README and licence kept.
The feature plan is docs/PLAN.md; the product spec is docs/INJEST.md.

## The rules this app builds against

- **Port 4850** (UI-XAPP-01, second block, registered in SPEC rev 51). Extra
  listeners take 4851 onwards, never another product's ten. The port is shown
  in mono wherever it appears.
- **Tokens only** (UI-TOK-01–04): navy `#1d2133` → `#2a2540`, accent `#2e9bf0`,
  `--ot-ok` / `--ot-warn` / `--ot-error` for status. `--ot-onair` red is reserved
  for on-air and recording — it is never an ingest error colour.
- **Honest status** (UI-COPY-03, UI-CMP-04): grey means no data, never a claim.
  The word "verified" appears only when a read-back verification actually ran;
  a Quick (size-only) job can never display as verified and can never release
  a card.
- **Plain copy, British spelling** in docs and interface copy (colours, grey,
  licence); engineer terms are introduced once in plain words (UI-COPY-01/02).
  Confirmations state the consequence before the action (UI-COPY-04) — the
  card-release confirmation is the worked example.
- **Vault only** (UI-XAPP-02): S3, MAM and every other credential is a vault
  reference. Settings shows the shared vault field and its status, never a
  credential field.
- **Brand from the Hub** (UI-XAPP-04/05): name, logo and colours come from the
  Hub brand (`GET /api/brand`), falling back to the tokens; offload reports
  carry the studio's brand.
- **Studio Bar** on every operator window, with the connection sentence and
  location path naming the instance (UI-XAPP-08); never on an output.
- **Updater behaviour** matches the family: verified releases only, with
  confirmation, progress, success and failure states (UI-XAPP-03).
- **Direct links** resolve through the workspace, never hard-coded addresses
  (UI-XAPP-07).
- **AI features carry the AI indicator** (UI-AI-01–05): transcript and tag
  suggestions are proposals a person accepts, never silent facts.
- **No dead ends** (UI-CMP-10): a stage that cannot run says which stage, why,
  and what to do next.

## Deviations

None yet. Anything built that departs from a SHOULD-level rule records the
reason here (UI-CONF-02).
