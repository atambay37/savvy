# Releasing

The full release checklist is tracked in [#132](https://github.com/uw-ssec/savvy/issues/132). This page covers the citation steps.

## Citation and DOI

Zenodo archives each published GitHub Release and mints a DOI for it, using the metadata in [`CITATION.cff`](../CITATION.cff).

1. **Before the first release only:** confirm `uw-ssec/savvy` is switched on at zenodo.org → Account → GitHub. A release published before this gets no DOI.
2. **Before tagging:** in `CITATION.cff`, set `version` to the tag without the `v` (tag `v26.11.0` → `version: 26.11.0`) and `date-released` to the release date (`YYYY-MM-DD`). Validate and commit:

   ```bash
   pixi exec cffconvert --validate -i CITATION.cff
   ```

   An invalid `CITATION.cff` makes the Zenodo archive fail, so no DOI is minted.
3. **Publish the GitHub Release.** A draft does not trigger Zenodo.
4. **After publishing:** within a few minutes the release should appear at zenodo.org → Account → GitHub. Check the record's title, authors, and version. If the archive failed, the error is shown there.
