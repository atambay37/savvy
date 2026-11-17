# Zenodo DOI Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Get `uw-ssec/savvy` ready so that publishing `v26.11.0` mints a correct Zenodo DOI, then add the DOI to the README and `CITATION.cff` afterwards.

**Architecture:** Docs and metadata only. `CITATION.cff` is the single metadata source Zenodo reads (no `.zenodo.json`). PR 1 lands before the release; Zenodo setup is manual; PR 2 lands after the release because the concept DOI does not exist until then.

**Tech Stack:** Citation File Format 1.2.0, `cffconvert` (via `pixi exec`), Zenodo GitHub integration, sandbox.zenodo.org.

**Spec:** `docs/specs/2026-11-17-zenodo-doi-design.md`

## Global Constraints

- No `.zenodo.json` anywhere in the repo — if present, Zenodo ignores `CITATION.cff`.
- Version strings follow the CalVer policy: tag `v26.11.0`, file value `26.11.0`, no zero-padding.
- No test release on real zenodo.org — test releases go to sandbox.zenodo.org only.
- `CITATION.cff` must pass `cffconvert --validate` before every release.
- Do not touch the stale `rse-plugins` URLs elsewhere in the README.

## Review Focus

- **Invalid `CITATION.cff` at release time** — Zenodo fails the archive with no DOI. Pinned by the `cffconvert` step in Task 1 and the release step in Task 2.
- **A release published before the integration is enabled** — gets no DOI and Zenodo will not backfill it. Pinned by the ordering note in `docs/releasing.md` (Task 2) and Task 4's check.
- **Draft releases** — do not trigger Zenodo; only a published release does. Stated in `docs/releasing.md` (Task 2).
- **Version DOI used instead of the concept DOI** — the badge would freeze on `v26.11.0`. Pinned by Task 5's resolve check.
- **ORCIDs or affiliations dropped or mangled by Zenodo's parser** — pinned by the sandbox check in Task 3.

---

### Task 1: Fix `CITATION.cff` version

**Files:**
- Modify: `CITATION.cff:4-5`

- [ ] **Step 1: Set `version: 26.11.0` and delete the `date-released: 2026-11-16` line**

- [ ] **Step 2: Validate**

Run: `pixi exec cffconvert --validate -i CITATION.cff`
Expected: `Citation metadata are valid according to schema version 1.2.0.`

Run: `grep -nE '^(version|date-released):' CITATION.cff`
Expected: exactly one line, `4:version: 26.11.0`

- [ ] **Step 3: Commit**

```bash
git add CITATION.cff
git commit -m "chore(citation): match CITATION.cff version to the CalVer policy (#141)"
```

### Task 2: "How to cite" and the release steps for citation

**Files:**
- Modify: `README.md` — new `## How to cite` section directly before `## License`
- Create: `docs/releasing.md`

**Interfaces:**
- Produces: a `## How to cite` heading in `README.md` and a `## Citation and DOI` heading in `docs/releasing.md`; Task 5 edits both.

- [ ] **Step 1: Add `## How to cite` to `README.md`**

It must contain: a one-line ask to cite savvy when it is used in research; the "Cite this repository" button in the repository sidebar (APA and BibTeX); a relative link to `CITATION.cff`; one sentence saying each GitHub Release is archived on Zenodo with a DOI, which will be listed here after the first release. No DOI, no placeholder link.

- [ ] **Step 2: Create `docs/releasing.md`**

A `# Releasing` title, one line saying the full checklist is tracked in #132, then `## Citation and DOI` with these ordered steps:

1. Before the first release only: confirm `uw-ssec/savvy` is switched on at zenodo.org → Account → GitHub. A release published before this gets no DOI.
2. Before tagging: set `version` (tag without the `v`) and `date-released` (release date, `YYYY-MM-DD`) in `CITATION.cff`, run `pixi exec cffconvert --validate -i CITATION.cff`, and commit.
3. Publish the GitHub Release. A draft does not trigger Zenodo.
4. After publishing: within a few minutes the release should appear at zenodo.org → Account → GitHub. Check the record's title, authors, and version. If it failed, the error is shown there.

- [ ] **Step 3: Verify links**

Run: `pixi run -e ci plugins-check`
Expected: exits 0 (it checks relative links, including the README's link to `CITATION.cff`).

- [ ] **Step 4: Commit**

```bash
git add README.md docs/releasing.md
git commit -m "docs: add How to cite and the citation steps for each release (#141)"
```

### Task 3: Sandbox dry run (manual, done by the user)

Done on the fork `atambay37/savvy` against sandbox.zenodo.org. It produces a test DOI, not a real one.

- [ ] **Step 1:** Push this branch to `fork`, then sign in to sandbox.zenodo.org with GitHub and switch on `atambay37/savvy` under Account → GitHub. If the fork is not listed, use a throwaway public repo containing only this `CITATION.cff` and `LICENSE`.
- [ ] **Step 2:** On the fork, publish a release with tag `v26.11.0` targeting `docs/zenodo-doi`.
- [ ] **Step 3:** Open the sandbox record and check it against `CITATION.cff`:
  - title `savvy`
  - all 13 authors, in file order
  - ORCIDs on the 8 authors that have them
  - affiliation on the 8 UW authors
  - license BSD-3-Clause
  - keywords present

  Record which version the record shows (`26.11.0` from the file or `v26.11.0` from the tag). This settles the open question in the spec.
- [ ] **Step 4:** If anything is wrong, fix `CITATION.cff`, re-run the Task 1 validation, commit, and repeat with a fresh sandbox release.
- [ ] **Step 5:** Delete the fork's test release and tag: `gh release delete v26.11.0 --repo atambay37/savvy --cleanup-tag --yes`

Then open PR 1 from `fork:docs/zenodo-doi` to `uw-ssec/savvy:main`, with the Step 3 findings in the description.

### Task 4: Switch on the integration for `uw-ssec/savvy` (manual, before `v26.11.0`)

- [ ] **Step 1:** Sign in to zenodo.org with GitHub. If `uw-ssec` repos are not listed, an org owner approves the Zenodo app at `https://github.com/organizations/uw-ssec/settings/oauth_application_policy`.
- [ ] **Step 2:** Switch on `uw-ssec/savvy` under Account → GitHub.

Verify: `gh api repos/uw-ssec/savvy/hooks --jq '.[].config.url'` lists a `zenodo.org` URL.

### Task 5: PR 2 — add the DOI (after `v26.11.0` is published)

**Files:**
- Modify: `README.md` — top badge row and `## How to cite`
- Modify: `CITATION.cff`

**Interfaces:**
- Consumes: the `## How to cite` section from Task 2.

- [ ] **Step 1:** On the Zenodo record, copy the concept DOI: the one under "Cite all versions", not the `v26.11.0` one.

Verify: `curl -sIL https://doi.org/<concept DOI> | grep -i '^location'` ends at a Zenodo URL, and the record page shows the latest version.

- [ ] **Step 2:** Add the badge `[![DOI](https://zenodo.org/badge/DOI/<concept DOI>.svg)](https://doi.org/<concept DOI>)` on the same line as the Codespaces badge. Replace the "will be listed here" sentence in `## How to cite` with the DOI link.
- [ ] **Step 3:** Add `doi: <concept DOI>` to `CITATION.cff` after `version`, then run `pixi exec cffconvert --validate -i CITATION.cff` (expect the valid message).
- [ ] **Step 4:** Commit, open the PR, and close #141 with it:

```bash
git add README.md CITATION.cff
git commit -m "docs: add the Zenodo concept DOI badge and citation (#141)"
```
