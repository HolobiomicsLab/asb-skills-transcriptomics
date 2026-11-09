---
name: distance-decay-weighting-of-regulatory-regions
description: Use when region-level TF affinities (TRAP, 818 PFMs from JASPAR 2022
  / HOCOMOCO v11 / Kheradpour & Kellis) must be summarised to gene level using a TSS-centred
  window (default 50 KB gene-window, 5 MB ABC-mode) with activity × exponential distance
  decay weighting.
when_to_use_negative:
- Promoter-mode runs with no bed-file/activity columns — the scaler is fixed at 1,
  so no distance decay or activity weighting applies
- Only the ABC-scored enhancer–gene interactions are of interest — STARE's ABC-scoring
  part can be called independently and affinity summarisation is unnecessary
- Regions are already assigned to genes by an external method that does not use the
  TSS-window or adapted-ABC mapping — the scaler choice (window decay vs adapted activity
  × contact) would be mismatched
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0654
tools:
- name: STARE
  role: Implements the region-gene window mapping (50 KB / 5 MB around 5' TSS), the
    exponential distance decay scaling (-e flag), the adapted-ABC ≤ 2500 bp promoter-proximal
    vs distant distinction, and the per-gene per-TF affinity summarisation
  repo: https://github.com/schulzlab/STARE
- name: TRAP
  role: Computes the continuous per-region TF affinities that are subsequently scaled
    and summed (C++ implementation shipped with TEPIC 2)
  repo: https://github.com/SchulzLab/TEPIC
- name: TEPIC 2
  role: Provides the TRAP-based affinity computation framework and the exponential
    decay / epigenetic signal scaling concepts reused by STARE
  repo: https://github.com/SchulzLab/TEPIC
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad062
    title: The adapted Activity-By-Contact model for enhancer–gene assignment and
      its application to single-cell data
  source_content_hash: sha256:b9e698c66f8ab9c11e2a230d6c7466c45c13af0a603f50faff524533a80db05d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/schulzlab/STARE
  tools:
  - STARE
  - TEPIC 2
  - TRAP
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# distance-decay-weighting-of-regulatory-regions

## Summary

STARE's method for scaling per-region TRAP TF affinities by chromatin activity and an exponential distance-to-TSS decay (or by ABC contact-based scaling) before summing them into a per-gene, per-TF affinity matrix.

## When to use

Use when region-level TF affinities (TRAP, 818 PFMs from JASPAR 2022 / HOCOMOCO v11 / Kheradpour & Kellis) must be summarised to gene level using a TSS-centred window (default 50 KB gene-window, 5 MB ABC-mode) with activity × exponential distance decay weighting.

## When NOT to use

- Promoter-mode runs with no bed-file/activity columns — the scaler is fixed at 1, so no distance decay or activity weighting applies
- Only the ABC-scored enhancer–gene interactions are of interest — STARE's ABC-scoring part can be called independently and affinity summarisation is unnecessary
- Regions are already assigned to genes by an external method that does not use the TSS-window or adapted-ABC mapping — the scaler choice (window decay vs adapted activity × contact) would be mismatched

## Inputs

- Non-overlapping candidate region bed-file with chromatin activity column(s) (specified via -n, e.g. open-chromatin signal)
- Gencode gene annotation file in gtf-format
- Position specific energy matrices (PSCM/PSEM) — the 818 non-redundant PSEMs from JASPAR 2022, HOCOMOCO v11, and Kheradpour & Kellis collections (PWMs at https://github.com/SchulzLab/STARE/tree/main/PWMs)
- Reference genome (for PSCM→PSEM conversion using bed_file GC-content or fixed GC via -y)
- Chromatin contact file (normalized from .hic via Juicer) when using ABC-based mapping

## Outputs

- Per-gene per-TF affinity matrix (genes as rows, TFs as columns) with three extra columns: NumPeaks, AvgPeakSize, AvgPeakDistance
- Discarded-genes file listing genes with no assignable regions
- Optionally a binary (IEEE-754 32-bit float) output with meta file for GAZE (via -z flag)

## How to apply

Load the non-overlapping candidate region bed-file (activity columns specified via -n) and a Gencode GTF annotation, and compute TRAP affinities for each region from the 818 non-redundant PSEMs (converting PSCMs to PSEMs using the GC-content of the --bed_file, or a fixed GC-content via -y). Assign regions to genes within a window centred at the 5' TSS — 50 KB for gene-window mode, 5 MB (±2.5 MB around the TSS) for ABC-mode. Scale each region's affinity by the mode-appropriate scaler: 1 for promoter-mode (no bed-file), activity × exponential distance decay for the gene window (pass -e False to disable decay), and for ABC-based mappings activity × contact / maximum contact, treating TSS-proximal regions (≤ 2500 bp) separately from more distant ones (the latter additionally scaled by their gene-specific adapted activity). Sum the scaled affinities per gene per TF and write the gene-TF matrix with genes as rows and TFs as columns; genes with no assignable regions go to a discarded-genes file. The rationale is that the summarisation is mapping-dependent: weighting encodes how strongly each open region plausibly regulates the gene's TSS.

## Related tools

- **STARE** (Implements the region-gene window mapping (50 KB / 5 MB around 5' TSS), the exponential distance decay scaling (-e flag), the adapted-ABC ≤ 2500 bp promoter-proximal vs distant distinction, and the per-gene per-TF affinity summarisation) — https://github.com/schulzlab/STARE
- **TRAP** (Computes the continuous per-region TF affinities that are subsequently scaled and summed (C++ implementation shipped with TEPIC 2)) — https://github.com/SchulzLab/TEPIC
- **TEPIC 2** (Provides the TRAP-based affinity computation framework and the exponential decay / epigenetic signal scaling concepts reused by STARE) — https://github.com/SchulzLab/TEPIC

## Examples

```
conda install stare-abc; bash Code/runTestCases.sh
```

## Evaluation signals

- Run STARE's Code/runTestCases.sh against the precomputed controls in /Test/Test_Controls/ and confirm the output matrix matches in content and quantity with no ERROR messages
- Check the output matrix schema: genes as rows, TFs as columns, plus NumPeaks, AvgPeakSize, AvgPeakDistance columns
- Verify genes with no assignable regions appear in the discarded-genes file rather than the matrix
- Toggle -e False and confirm the exponential distance decay is removed from the gene-window scaling while activity scaling remains
- Confirm TSS-proximal regions (≤ 2500 bp) and more distant regions are handled by different scalers under ABC-based mapping

## Limitations

- The summarisation result is mapping-dependent: gene-window mode sums all open regions around the TSS, whereas ABC-based mapping treats ≤ 2500 bp TSS-proximal regions separately from distant ones scaled by gene-specific adapted activity
- Genes with no assignable regions are discarded and only reported in a separate file, so downstream matrices will lack them
- For non-adapted ABC-scoring the adapted activity is unavailable, so distant-region scaling must fall back to a different scheme

## Evidence

- [intro] summarise TF affinities on gene level: "Those two parts are combined to summarise TF affinities on gene level, which allows for a variety of downstream analyses"
- [methods] summarises the TF-affinities from regions that were assigned: "To get TF scores for a gene, STARE summarises the TF-affinities from regions that were assigned to that gene."
- [methods] window centred at the 5' TSS: "Window size centred at the 5' TSS in which regions from the --bed_file will be considered for a gene"
- [methods] ABC-score scaling distinguishes promoter-proximal and distant regions: "If you did the region-gene mapping with the ABC-score, we change the scaling and also distinguish between regions close to the promoter and more distant ones."
- [methods] PSEM conversion from bed_file GC-content: "The background GC-content to construct the PSEMs will be taken from your provided --bed_file. You can also specify a fixed GC-content with the -y flag."
- [readme] test-case validation: "we have the *Code/runTestCases.sh* script for you"
