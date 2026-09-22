# Recital Under The Stars — handoff

A single-file web app for Baros Piano Studio. It randomly picks which of 13 piano
students performs next, until everyone has played. A 3D drum spins through the
names and lands on the pick; the programme on the right fills in as students go.

**Live:** https://spinner.barosfamily.org
**Repo:** https://github.com/mattbaros1/recital-spinner

## Source of truth

**Claude Design is the source of truth.** The interface is edited there, not here.
This repo holds only the exported build.

The old local build chain (`*.dc.html` source + `support.js` + `_ds/`) is gone and
is not coming back. Don't try to reconstruct it — export from Claude Design instead.

## Deploy workflow

1. Edit the interface in Claude Design.
2. Export `index.html` (a single self-contained file, ~244 KB, zero external requests).
3. Copy it over `index.html` in this repo, commit, push to `main`.
4. GitHub Pages rebuilds automatically, ~30–60s. Verify the live page, then you're done.

No build step, no dependencies, no bundler to run locally.

Verify a deploy landed by comparing hashes rather than eyeballing it:

```bash
shasum -a 256 index.html
curl -s https://spinner.barosfamily.org/ | shasum -a 256
```

## Files

| File | Purpose |
|---|---|
| `index.html` | the whole app — this is what gets deployed |
| `CNAME` | `spinner.barosfamily.org`; GitHub Pages needs this at repo root |
| `robots.txt` | `Disallow: /` — keeps the students' names out of search engines |

## Making changes

Interface changes go through Claude Design. These knobs live in the exported JS if
you need to edit the bundle directly:

- **Student list** — since the Sept 2026 export there's an "Edit students" dialog
  in the app itself (one name per line or comma-separated; saving starts a new
  recital). The edited list is stored per-browser under `localStorage` key
  `recital-under-the-stars-students` and overrides the bundled default — so it only
  applies on the device where it was entered. The bundled default still ships in
  the export (search `Levi Hansen`, comma-separated string of 13 names); change it
  in Claude Design when the new list should apply everywhere.
- **Spin duration** — `spinSeconds`, default `3.5` (range 1–8).
- **Accent colour** — `--color-accent` in a `:root` block.
- **Saved progress** — `localStorage` key `recital-under-the-stars-v2`, shape
  `{ played, current, phase }`. "Start over" clears it after a native browser
  confirm dialog (which embedded/automated browsers may suppress).

## Gotchas

**The page title is fixed at source — don't re-patch it.** The bundler runs
`document.documentElement.replaceWith(...)` at boot, which throws away the outer
shell's `<title>`. Early exports had no title in the inner template, so the browser
tab fell back to showing the bare hostname. Claude Design now writes
`<title>Recital Spinner</title>` into the `<helmet>` block, which survives the swap.
If a future export loses it again, fix it in Claude Design's helmet — patching the
exported bundle works but gets overwritten on the next export.

**Progress format is stable across redesigns.** Both the pre- and post-redesign
versions persist `{ played, current, phase }` under the same key, so a recital in
progress survives a mid-event deploy. The drum angle is derived from `current` on
load rather than stored.

**A redesign can look identical at rest.** The tumbler only renders while spinning.
To confirm a new design actually deployed, click the spin button and look — an idle
screenshot cannot distinguish the flat-strip version from the 3D drum.

**HTTPS certificate, if it ever breaks.** Custom domain is `spinner.barosfamily.org`
via CNAME to `mattbaros1.github.io`. If Let's Encrypt provisioning hangs (it once sat
for 75+ minutes with correct DNS and CAA), remove and re-add the custom domain:

```bash
echo '{"cname":null}' | gh api -X PUT repos/mattbaros1/recital-spinner/pages --input -
echo '{"cname":"spinner.barosfamily.org"}' | gh api -X PUT repos/mattbaros1/recital-spinner/pages --input -
gh api -X PUT repos/mattbaros1/recital-spinner/pages -F https_enforced=true
```

That issued a cert in about two minutes. No outage — Pages keeps serving the previous
build throughout. Don't repeat it in a loop: Let's Encrypt rate-limits at 5 failed
authorizations per hostname per hour.

**DNS looking broken is usually a cache.** Consumer routers cache NXDOMAIN for up to
an hour (GoDaddy's SOA sets a 3600s negative TTL). If the domain resolves on
`1.1.1.1` but not on this machine, it's the router, not the DNS record. Check with
`dig spinner.barosfamily.org @1.1.1.1` before touching anything.

## This working copy

Lives in OneDrive (`Claude AI Folder`), so it syncs across machines. Two caveats:

- Git inside a cloud-synced folder can corrupt `.git`. GitHub is authoritative — if
  that happens, delete the folder and re-clone.
- The repo is public and contains 13 students' full names. `robots.txt` blocks
  crawlers, but the names are readable by anyone with the URL.

Recovery is always:

```bash
git clone https://github.com/mattbaros1/recital-spinner.git
```
