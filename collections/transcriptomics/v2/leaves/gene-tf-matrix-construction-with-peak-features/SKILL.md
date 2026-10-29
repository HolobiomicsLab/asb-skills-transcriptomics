---
name: gene-tf-matrix-construction-with-peak-features
description: Use when you have open-chromatin candidate regions (e.g. ATAC/DNase peaks)
  with activity scores and need a gene-level TF affinity matrix (genes as rows, TFs
  as columns) whose scaling follows the chosen region–gene mapping mode.
when_to_use_negative:
- You only need per-region TF affinities without gene-level aggregation — use the
  region-level TRAP affinity step alone
- You are only interested in the ABC-scored enhancer–gene interactions themselves
  — STARE can call that part independently without TF summarisation
- Your input is already a gene-level feature table (e.g. a precomputed TF-gene score
  matrix), where further summarisation would double-count signal
edam_topics:
- http://edamontology.org/topic_0797
tools:
- name: STARE
  role: Assigns regions to genes around the 5' TSS (50 KB gene-window or 5 MB ABC
    window), applies the mode-specific affinity scaler, and writes the gene-TF matrix
    with NumPeaks/AvgPeakSize/AvgPeakDistance plus a discarded-genes file
  repo: https://github.com/schulzlab/STARE
- name: TRAP
  role: Computes the per-region TF affinities (from 818 non-redundant PSEMs) that
    are subsequently scaled and summed per gene; the C++ implementation is bundled
    in TEPIC
  repo: https://github.com/SchulzLab/TEPIC
- name: TEPIC 2
  role: Extended framework providing the TRAP-based affinity computation and the window-based
    gene-assignment aggregation logic that STARE's gene-level summarisation builds
    on
  repo: https://github.com/SchulzLab/TEPIC
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad062
    title: The adapted Activity-By-Contact model for enhancer–gene assignment and
      its application to single-cell data
  source_content_hash: sha256:5b32937377e627a7e213f4a4f0a1092b9d8af57d0c99d47472ddbe6b5fada355
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/schulzlab/STARE
  tools:
  - STARE
  - TEPIC 2
  - TRAP
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# Gene-TF matrix construction with peak features

## Summary

Constructs a per-gene per-TF transcription factor affinity matrix by summing TRAP region-level affinities over regions assigned to each gene, with scaling that depends on the region–gene mapping mode (gene-window or adapted ABC) and output of peak-count features (NumPeaks, AvgPeakSize, AvgPeakDistance) per gene.

## When to use

Use when you have open-chromatin candidate regions (e.g. ATAC/DNase peaks) with activity scores and need a gene-level TF affinity matrix (genes as rows, TFs as columns) whose scaling follows the chosen region–gene mapping mode.

## When NOT to use

- You only need per-region TF affinities without gene-level aggregation — use the region-level TRAP affinity step alone
- You are only interested in the ABC-scored enhancer–gene interactions themselves — STARE can call that part independently without TF summarisation
- Your input is already a gene-level feature table (e.g. a precomputed TF-gene score matrix), where further summarisation would double-count signal

## Inputs

- Non-overlapping candidate region bed-file with chromatin activity column(s) specified via -n (e.g. ATAC-seq accessibility signal)
- Gencode gene annotation file in gtf-format
- 818 non-redundant position specific energy matrices (PSEMs) from JASPAR 2022, HOCOMOCO v11, and the Kheradpour & Kellis collection (PSCMs converted to PSEMs using bed_file GC-content or -y)
- Normalized Hi-C contact data (e.g. KR-normalized .hic processed via Juicer normalization script) for ABC-mode contact scaling

## Outputs

- Per-gene per-TF affinity matrix (genes as rows, TFs as columns) with three extra columns: NumPeaks, AvgPeakSize, AvgPeakDistance
- Discarded-genes file listing genes with no assignable regions
- Optional binary (IEEE-754 32-bit float) output with meta file for downstream GAZE use (via -z flag)

## How to apply

Load a non-overlapping candidate region bed-file with activity column(s) specified via -n and a Gencode gtf gene annotation, then compute TRAP affinities for every region using the 818 non-redundant PSEMs (JASPAR 2022, HOCOMOCO v11, Kheradpour & Kellis); PSCMs are converted to PSEMs using the GC-content of the --bed_file as background, or a fixed GC-content via -y. Assign regions to genes within a window centred at the 5' TSS (default 50 KB for gene-window mode, 5 MB for ABC-mode, i.e. ±2.5 MB around the TSS). Scale each region's affinity by the mode-appropriate scaler: 1 for promoter-mode (no bed-file); activity × exponential distance decay for the gene window (pass -e False to disable decay); for ABC-based mappings, activity × contact / maximum contact, treating TSS-proximal regions (≤ 2.5 kb) separately from more distant ones. Sum the scaled affinities per gene per TF, write the gene-TF matrix with three extra columns (NumPeaks, AvgPeakSize, AvgPeakDistance), and write genes with no assignable regions to a discarded-genes file. Validate the installation and outputs by running STARE's runTestCases.sh against the precomputed controls in /Test/Test_Controls/ and confirming content and quantity match with no ERROR messages.

## Related tools

- **STARE** (Assigns regions to genes around the 5' TSS (50 KB gene-window or 5 MB ABC window), applies the mode-specific affinity scaler, and writes the gene-TF matrix with NumPeaks/AvgPeakSize/AvgPeakDistance plus a discarded-genes file) — https://github.com/schulzlab/STARE
- **TRAP** (Computes the per-region TF affinities (from 818 non-redundant PSEMs) that are subsequently scaled and summed per gene; the C++ implementation is bundled in TEPIC) — https://github.com/SchulzLab/TEPIC
- **TEPIC 2** (Extended framework providing the TRAP-based affinity computation and the window-based gene-assignment aggregation logic that STARE's gene-level summarisation builds on) — https://github.com/SchulzLab/TEPIC

## Examples

```
bash Code/runTestCases.sh  # validate installation against /Test/Test_Controls/ before running the gene-TF summarisation
```

## Evaluation signals

- Run STARE's runTestCases.sh and confirm the output matrix matches the precomputed controls in /Test/Test_Controls/ in both content and quantity with no ERROR messages
- Matrix schema check: genes as rows, TFs as columns, plus exactly three extra columns — NumPeaks, AvgPeakSize, AvgPeakDistance
- Column count check: one column per TF from the 818 non-redundant PSEM collection (JASPAR 2022, HOCOMOCO v11, Kheradpour & Kellis)
- Gene completeness check: every input gene appears either in the matrix or in the discarded-genes file (genes with no assignable regions)
- Scaling-mode invariant: in promoter-mode (no bed-file) the region scaler is 1, so gene scores are plain sums of TRAP affinities; in ABC-mode TSS-proximal regions (≤ 2.5 kb) are scaled separately from distant regions (activity × contact / max contact)

## Limitations

- Summarisation is mapping-dependent: gene-window mode sums all open regions around the TSS (default 50 KB) whereas ABC-mode scales by adapted activity with contact normalization, so matrices from the two modes are not directly comparable
- The exponential distance decay in gene-window mode is a default behaviour that must be explicitly disabled with -e False; forgetting this changes scores
- Genes with no assignable regions are dropped from the matrix and reported only in the discarded-genes file, which can bias downstream gene-level analyses
- ABC-mode requires normalized chromatin contact data (e.g. KR-normalized .hic prepared via Juicer); without contacts the ABC scaling path is not available

## Evidence

- [methods] To get TF scores for a gene, STARE summarises the TF-affinities from regions that were assigned to that gene.: "To get TF scores for a gene, STARE summarises the TF-affinities from regions that were assigned to that gene."
- [methods] Window centred at the 5' TSS defines candidate regions per gene: "Window size centred at the 5' TSS in which regions from the --bed_file will be considered for a gene"
- [methods] ABC-mode scaling distinguishes promoter-proximal and distant regions: "If you did the region-gene mapping with the ABC-score, we change the scaling and also distinguish between regions close to the promoter and more distant ones."
- [methods] PSEM background GC-content comes from the bed-file or -y flag: "The background GC-content to construct the PSEMs will be taken from your provided --bed_file. You can also specify a fixed GC-content with the -y flag."
- [readme] TEPIC window-based aggregation rationale for TF-gene scores: [evidence span withheld — anchor status: external_doc]
