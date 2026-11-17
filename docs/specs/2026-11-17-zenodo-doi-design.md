# Zenodo DOI — design

Issue: [#141](https://github.com/uw-ssec/savvy/issues/141). Depends on the first release ([#132](https://github.com/uw-ssec/savvy/issues/132)) and the CalVer policy ([#130](https://github.com/uw-ssec/savvy/issues/130) / PR #258).

## Intent

Zenodo mints a DOI for each published GitHub Release. We want everything in place *before* the first release (`v26.11.0`) so that cutting it produces a correct, citable DOI with no scramble afterwards. Zenodo DOIs are permanent, so the first record has to be right.

## Goals

- The repo is enabled in Zenodo's GitHub integration before the first release.
- The first release produces a Zenodo record whose title, authors, ORCIDs, affiliations, license, and version are correct.
- The README carries the concept DOI badge and a "How to cite" section.
- The release process says what to update for citation on every release.

## Non-goals

- A `.zenodo.json`. **Decided:** `CITATION.cff` is the single metadata source. If both exist Zenodo ignores `CITATION.cff`, which would mean two author lists to keep in sync. Consequence: no Zenodo community or grant metadata on the record.
- Writing the full release checklist — that is #132.
- Fixing the stale `rse-plugins` URLs elsewhere in the README.

## Anti-goals

- A real Zenodo DOI minted by a test release. Test only on sandbox.zenodo.org.
- An invalid `CITATION.cff` at release time — Zenodo fails the archive when the file does not parse.

## Constraints and findings

- The concept DOI does not exist until the first release is published, so the badge and DOI lines land in a follow-up after `v26.11.0`.
- `CITATION.cff` on main says `version: 2026.11.0`, `date-released: 2026-11-16`. The CalVer policy uses `26.11.0` (tag `v26.11.0`), and nothing has been released.
- Zenodo's docs list `version` among the `CITATION.cff` fields it reads; third-party docs say the version comes from the release tag. The sandbox run settles which.
- Enabling the integration for an org repo may need a uw-ssec org owner to approve the Zenodo OAuth app.

## Approaches

1. **`CITATION.cff` only** — one source of truth, already validated, already shown by GitHub's "Cite this repository". No Zenodo-only metadata. **Chosen.**
2. **`.zenodo.json` alongside** — allows communities and grants, but Zenodo then ignores `CITATION.cff` and author lists drift. Rejected.

## Design

**PR 1 — before the release (this branch)**

1. `CITATION.cff`: set `version: 26.11.0`; remove `date-released` (no release has happened — it is set when the release is cut). Validate with `cffconvert --validate`.
2. `README.md`: add a "How to cite" section before "License", pointing to GitHub's "Cite this repository" button and `CITATION.cff`. No DOI yet.
3. `docs/releasing.md` (new, short): a "Citation and DOI" section — before tagging, set `version` and `date-released` in `CITATION.cff` to match the tag and validate it; after publishing, confirm the Zenodo record appeared and is correct. #132 grows this file into the full checklist.

**Setup — outside the repo, before the release**

4. Sandbox dry run: enable the fork (`atambay37/savvy`) on sandbox.zenodo.org, publish a throwaway release there, and check the record's metadata against `CITATION.cff`. Fix anything wrong in PR 1.
5. Enable `uw-ssec/savvy` in Zenodo's GitHub settings (org owner approval if prompted).

**PR 2 — after `v26.11.0` is published**

6. Take the concept DOI from the Zenodo record (the "all versions" DOI, not the version DOI).
7. `README.md`: add the DOI badge next to the Codespaces badge; add the DOI to "How to cite".
8. `CITATION.cff`: add `doi: <concept DOI>`.

## My calls (correct if wrong)

- Remove `date-released` now rather than leave a date for a release that has not happened.
- Put the citation release steps in a new `docs/releasing.md` rather than wait for #132.
- Two PRs, since the DOI cannot exist before the release.
