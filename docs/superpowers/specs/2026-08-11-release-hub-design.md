# nyanspace-build: public release hub — design

Date: 2026-08-11

## Problem

`nyanspace` (Android/Expo, legacy) and `nyanspace-pc` (Electron desktop,
Windows/Linux) are both private repos, each with its own working CI that
builds installers. Neither can be used to distribute downloads publicly
because the repos themselves are private. `nyanspace-build` is a new,
public repo meant to be the place end users download the final installer
files from, without exposing either private repo's source.

## Decisions

- **Scope**: publish both product lines — desktop installers (current) and
  the legacy Android APK (still wanted for existing users), not just one.
- **Build ownership**: the two private repos keep building everything
  themselves (already working, already signs/checksums/SBOMs the desktop
  build). `nyanspace-build` never checks out private source and holds no
  PAT or signing secrets — it is a passive receiver only. This avoids
  duplicating signing secrets into a public-adjacent workflow and avoids
  the classic "secret leak via a PR workflow edit" vector, since
  `nyanspace-build` has no write-path workflow at all.
- **Publish mode**: releases land as **drafts** in `nyanspace-build`
  (`draft: true`), reviewed and published by hand — not auto-public on
  tag push.
- **APK trigger**: `nyanspace`'s `build-apk.yml` moves from its current
  branch-list trigger to `push: tags: ['v*.*.*']`, matching
  `nyanspace-pc`'s existing tag-triggered release workflow.

## Architecture

```
push tag v1.2.3 (nyanspace or nyanspace-pc)
  → existing CI builds/signs/packages (unchanged)
  → new final step: softprops/action-gh-release@v2
      repository: quydang04/nyanspace-build
      token: secrets.BUILD_REPO_PAT
      draft: true
      tag_name: <prefix>-v1.2.3
      files: installer/APK + checksums + SBOM + release notes
  → maintainer reviews the draft on nyanspace-build, clicks Publish
```

### Tag naming

Both apps may ship the same version number independently, so the tag
pushed into `nyanspace-build` is prefixed to avoid collisions:

- Desktop → `desktop-vX.Y.Z`
- Mobile → `mobile-vX.Y.Z`

The source repos keep tagging their own commits `vX.Y.Z` as today; the
prefix is only added when publishing into `nyanspace-build`.

### Credentials

- One fine-grained PAT, scoped to the `nyanspace-build` repo only,
  `Contents: Read and write` permission — enough to create a release and
  upload assets, nothing more.
- Stored as the `BUILD_REPO_PAT` secret in **both** `nyanspace` and
  `nyanspace-pc` (not in `nyanspace-build` — it never needs it).
- Creating the PAT is a manual GitHub UI step (requires the repo owner's
  login); not automatable from here.

## Repo contents (`nyanspace-build`)

- `README.md` — explains this is a downloads-only mirror, links to
  Releases, states the source repos are private.
- `LICENSE` — MIT, matching the source projects.
- No workflows. Nothing here writes anything; it only ever receives
  release assets pushed in from the two private repos' CI.

## Out of scope / deferred

- Auto-publishing (non-draft) releases — explicitly rejected; a human
  reviews and publishes every release.
- A unified changelog/version page across both products — not requested;
  the GitHub Releases list is sufficient for v1.
- macOS desktop builds — out of scope per `nyanspace-pc`'s existing ADR
  0001, unaffected by this change.
