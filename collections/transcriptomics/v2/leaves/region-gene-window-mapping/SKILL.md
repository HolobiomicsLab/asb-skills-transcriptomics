---
name: region-gene-window-mapping
description: 'Use when building a gene-level TF affinity matrix from region-level
  TRAP affinities: to pick gene-window (default 50 kb around 5'' TSS) vs ABC-based
  (5 Mb, i.e. ±2.5 Mb) mapping, or to decide scaling of promoter-proximal vs distant
  regions.'
when_to_use_negative:
- You only need enhancer-gene interaction scores and not gene-level TF affinities
  — STARE exposes the ABC-scoring part independently.
- Regions are already assigned to genes (e.g. precomputed enhancer-gene links); re-windowing
  would duplicate regions across genes and change the summation.
- No Gencode-style GTF gene annotation is available, so 5' TSS coordinates for the
  window cannot be defined.
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0203
tools:
- name: STARE
  role: Performs the region-gene mapping within the TSS-centred window (gene-window
    or ABC modes) and scales/sums TRAP affinities into the gene-TF matrix
  repo: https://github.com/schulzlab/STARE
- name: TEPIC 2
  role: Antecedent framework providing the window-based TF-gene score aggregation
    with exponential decay and activity scaling that STARE's mapping builds on
  repo: https://github.com/SchulzLab/TEPIC/tree/master/MachineLearningPipelines/INVOKE
- name: bedtools
  role: Genome-arithmetic operations (interval intersection/assignment) underlying
    region-gene assignment
  repo: https://github.com/arq5x/bedtools2
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad062
    title: The adapted Activity-By-Contact model for enhancer–gene assignment and
      its application to single-cell data
  source_content_hash: sha256:2198c7d4e8d337b4e611d78b5a697c513213dee8eddeceaf806a0032cbb7d00c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/schulzlab/STARE
  tools:
  - STARE
  - TEPIC 2
  - TRAP
  - bedtools
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# region-gene-window-mapping

## Summary

The STARE region-gene mapping step that assigns open-chromatin candidate regions to genes within a window centred at the 5' TSS, defining which TRAP TF affinities (from 818 non-redundant PSEMs) are summed into the per-gene per-TF affinity matrix. The mapping choice (gene-window vs ABC-based) also determines the per-region scaling applied before summation.

## When to use

Use when building a gene-level TF affinity matrix from region-level TRAP affinities: to pick gene-window (default 50 kb around 5' TSS) vs ABC-based (5 Mb, i.e. ±2.5 Mb) mapping, or to decide scaling of promoter-proximal vs distant regions.

## When NOT to use

- You only need enhancer-gene interaction scores and not gene-level TF affinities — STARE exposes the ABC-scoring part independently.
- Regions are already assigned to genes (e.g. precomputed enhancer-gene links); re-windowing would duplicate regions across genes and change the summation.
- No Gencode-style GTF gene annotation is available, so 5' TSS coordinates for the window cannot be defined.

## Inputs

- Non-overlapping candidate region bed-file with chromatin accessibility/activity column(s) (activity columns selected via -n)
- Gene annotation file in Gencode gtf-format (defines 5' TSS coordinates)
- PSEM files converted from PSCMs using GC-content of the bed-file (or fixed GC-content via the -y flag)
- Normalized chromatin contact files from .hic (Juicer KR normalization) for ABC-based mapping mode

## Outputs

- Per-gene per-TF affinity matrix (genes as rows, TFs as columns) with three extra columns: NumPeaks, AvgPeakSize, AvgPeakDistance
- Discarded-genes file listing genes with no assignable regions
- Optionally a binary IEEE-754 32-bit float matrix with meta file (for GAZE) via the -z flag

## How to apply

Load the non-overlapping candidate region bed-file (with activity column(s) given via -n) and a Gencode GTF gene annotation, and compute TRAP affinities for each region from the 818 non-redundant PSEMs (JASPAR 2022, HOCOMOCO v11, Kheradpour & Kellis collections; PSCMs are auto-converted to PSEMs using the bed-file's GC-content, or a fixed GC-content via -y). Assign regions to genes within a window centred at the 5' TSS — default 50 kb in gene-window mode, 5 MB (±2.5 MB) in ABC-mode — then scale each region's affinity by the mode-appropriate scaler: 1 in promoter-mode (no bed-file); activity × exponential distance decay in gene-window mode (-e False disables the decay); in ABC-based mapping, regions ≤ 2.5 kb from the TSS are always included with exponential distance decay, while more distant regions are scaled by activity × contact / maximum contact (the adapted activity). Finally sum the scaled affinities per gene per TF, so the mapping method directly controls both which regions contribute to a gene and how strongly they weigh.

## Related tools

- **STARE** (Performs the region-gene mapping within the TSS-centred window (gene-window or ABC modes) and scales/sums TRAP affinities into the gene-TF matrix) — https://github.com/schulzlab/STARE
- **TEPIC 2** (Antecedent framework providing the window-based TF-gene score aggregation with exponential decay and activity scaling that STARE's mapping builds on) — https://github.com/SchulzLab/TEPIC/tree/master/MachineLearningPipelines/INVOKE
- **bedtools** (Genome-arithmetic operations (interval intersection/assignment) underlying region-gene assignment) — https://github.com/arq5x/bedtools2

## Examples

```
bash Code/runTestCases.sh
```

## Evaluation signals

- Run STARE's Code/runTestCases.sh against the precomputed controls in /Test/Test_Controls/ and confirm the output matrix matches in content and quantity with no ERROR messages
- Every region falling within the chosen window (50 kb gene-window or ±2.5 MB ABC window) around a gene's 5' TSS appears in that gene's assignment set
- Genes with no assignable regions appear in the discarded-genes file rather than as all-zero rows
- Matrix schema check: genes as rows, one column per TF plus NumPeaks, AvgPeakSize, AvgPeakDistance columns
- Switching -e (exponential decay) on/off changes only the per-region scaling of gene-window scores, not the region-gene assignment itself

## Limitations

- Summarization is mapping-dependent: window-based and ABC-based mappings yield different per-gene per-TF matrices for the same regions and affinities
- The exponential distance decay can be disabled with -e False, changing scores substantially; the appropriate choice depends on the intended downstream analysis
- ABC-based mapping requires normalized Hi-C contacts (e.g. Juicer KR normalization of .hic files), which the gene-window and promoter modes do not need
- Genes with no regions in their window produce no scores and are written to the discarded-genes file

## Evidence

- [methods] Window size centred at the 5' TSS in which regions from the --bed_file will be considered for a gene: "Window size centred at the 5' TSS in which regions from the --bed_file will be considered for a gene"
- [methods] If you did the region-gene mapping with the ABC-score, we change the scaling and also distinguish between regions close to the promoter and more distant ones.: "If you did the region-gene mapping with the ABC-score, we change the scaling and also distinguish between regions close to the promoter and more distant ones."
- [intro] Those two parts are combined to summarise TF affinities on gene level, which allows for a variety of downstream analyses: "Those two parts are combined to summarise TF affinities on gene level, which allows for a variety of downstream analyses"
- [readme] These predictions are aggregated to TF-gene scores using a window-based assignment strategy.: [evidence span withheld — anchor status: external_doc]
