---
name: tf-affinity-summarisation-per-gene
description: Use when you have a non-overlapping candidate region bed-file with per-cell-type
  activity columns and need gene-level TF affinity features for downstream modelling
  such as INVOKE expression prediction.
when_to_use_negative:
- You only need the enhancer-gene interaction scores themselves — run STARE's separate
  ABC-scoring part independently instead of the gene-level affinity summarisation.
- Your regions lack an activity signal and only promoter-proximal regions matter —
  use promoter-mode (no bed-file scaling) rather than the activity-scaled ABC-mode
  summarisation.
- You want TF-gene scores from raw peaks with TEPIC's own window-based assignment,
  length normalisation and peak-count features instead of STARE's mapping-dependent
  scalers.
edam_topics:
- http://edamontology.org/topic_0654
tools:
- name: STARE
  role: Core framework that performs the region-gene mapping, mode-specific scaling
    and per-gene TF affinity summarisation
  repo: https://github.com/schulzlab/STARE
- name: TRAP
  role: Computes the region-level TF affinities that are subsequently summarised per
    gene
  repo: https://github.com/SchulzLab/TEPIC
- name: TEPIC 2
  role: Alternative extended framework for TF affinity computation and TF-gene scoring
    with window-based assignment; its INVOKE module consumes the resulting gene-TF
    matrices
  repo: https://github.com/SchulzLab/TEPIC
- name: GAZE
  role: Downstream single-cell regulatory analysis consuming STARE's optional binary
    output (written with -z)
  repo: https://github.com/schulzlab/gaze
- name: bedtools
  role: Genome arithmetic helper for preparing non-overlapping candidate region bed-files
  repo: https://github.com/arq5x/bedtools2
provenance:
  source_task_ids:
  - task_007
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad062
    title: The adapted Activity-By-Contact model for enhancer–gene assignment and
      its application to single-cell data
  source_content_hash: sha256:54d8cdb25ecff0cda3f2a7b8b978a1d9fa14d8313131f57b99d05726935309f9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/schulzlab/STARE
  tools:
  - Cicero
  - GAZE
  - STARE
  - TEPIC 2
  - TRAP
  - bedtools
  - stare-abc
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# tf-affinity-summarisation-per-gene

## Summary

Summarises region-level TRAP TF affinities (computed over 818 non-redundant PFMs from JASPAR 2022, HOCOMOCO v11 and Kheradpour & Kellis) into a per-gene per-TF affinity matrix, with the summation scaling determined by the chosen region-gene mapping (gene window, ABC, gABC or custom existing_abc mapping).

## When to use

Use when you have a non-overlapping candidate region bed-file with per-cell-type activity columns and need gene-level TF affinity features for downstream modelling such as INVOKE expression prediction.

## When NOT to use

- You only need the enhancer-gene interaction scores themselves — run STARE's separate ABC-scoring part independently instead of the gene-level affinity summarisation.
- Your regions lack an activity signal and only promoter-proximal regions matter — use promoter-mode (no bed-file scaling) rather than the activity-scaled ABC-mode summarisation.
- You want TF-gene scores from raw peaks with TEPIC's own window-based assignment, length normalisation and peak-count features instead of STARE's mapping-dependent scalers.

## Inputs

- Non-overlapping candidate region bed-file (e.g. heart single-cell ATAC-seq peaks) with one activity column per cell type, passed via -n
- Gene annotation in gtf-format (e.g. Gencode)
- PSEM collection of 818 non-redundant PFMs (JASPAR 2022, HOCOMOCO v11, Kheradpour & Kellis) from STARE's PWMs folder
- Normalized chromatin contact files (bin|bin|contact format, e.g. KR-normalized) when using ABC-mode region-gene mapping
- Custom region-gene mapping (existing_abc via -r / --existing_abc with Ensembl ID, PeakID and intergenicScore columns) for external mappings such as Cicero co-accessibility

## Outputs

- Gene-TF affinity matrix per activity column, one *_TF_Gene_Affinities.txt.gz with genes as rows and TFs as columns plus three extra columns: NumPeaks, AvgPeakSize, AvgPeakDistance
- Discarded-genes file listing genes with no assignable regions

## How to apply

Load the non-overlapping region bed-file with the activity columns specified via -n together with a Gencode gtf, and compute TRAP affinities for each region from the 818 non-redundant PSEMs (STARE's PWMs collection). Assign regions to genes within a window centred at the 5' TSS — 50 kb for gene-window mode, 5 MB (±2.5 MB around the TSS) for ABC-mode — then scale each region's affinity by the mode-appropriate scaler: 1 for promoter-mode; activity × exponential distance decay for the gene window (disable decay with -e False); for ABC-based mappings, activity × contact / maximum contact, treating TSS-proximal regions (≤ 2.5 kb) separately from more distant ones, with adapted activity used only when -q True. Sum the scaled affinities per gene per TF and write one *_TF_Gene_Affinities.txt.gz matrix per activity column, using the default ABC cutoff -t 0.02 for ABC-mode mappings. Genes with no assignable regions are reported to a discarded-genes file rather than silently dropped.

## Related tools

- **STARE** (Core framework that performs the region-gene mapping, mode-specific scaling and per-gene TF affinity summarisation) — https://github.com/schulzlab/STARE
- **TRAP** (Computes the region-level TF affinities that are subsequently summarised per gene) — https://github.com/SchulzLab/TEPIC
- **TEPIC 2** (Alternative extended framework for TF affinity computation and TF-gene scoring with window-based assignment; its INVOKE module consumes the resulting gene-TF matrices) — https://github.com/SchulzLab/TEPIC
- **GAZE** (Downstream single-cell regulatory analysis consuming STARE's optional binary output (written with -z)) — https://github.com/schulzlab/gaze
- **bedtools** (Genome arithmetic helper for preparing non-overlapping candidate region bed-files) — https://github.com/arq5x/bedtools2

## Examples

```
conda install stare-abc
```

## Evaluation signals

- Run Code/runTestCases.sh and confirm the output matrix matches the precomputed controls in /Test/Test_Controls/ in content and quantity with no ERROR messages.
- Each *_TF_Gene_Affinities.txt.gz has genes as rows, TFs as columns, and exactly three extra columns: NumPeaks, AvgPeakSize, AvgPeakDistance.
- Genes with no assignable regions appear in the discarded-genes file rather than in the affinity matrix.
- For ABC-mode runs, the affinity matrix dimensionality equals the number of TFs (818 non-redundant PSEMs) and the number of genes receiving at least one mapped region.
- In the heart comparison, the summarised matrices feed INVOKE models whose Pearson correlation between predicted and observed expression should reach μ ≈ 0.563 for gABC-based mapping (vs ~0.538 regular ABC, ~0.505 Cicero).

## Limitations

- The summarisation is mapping-dependent: window-based mode sums all open regions around the TSS, whereas gABC-based mapping always includes regions within 2500 bp of the TSS (with exponential decay) and scales more distant regions by their gene-specific adapted activity — choosing the wrong mode changes the scaling and hence the matrix.
- The exponential distance decay in gene-window mode is only applied when activity columns are given; -e False disables the decay.
- Genes with no regions assignable within the mapping window are discarded from the output matrix, so downstream models exclude them.
- Under the non-adapted ABC score, the adapted activity cannot be used for scaling, so the resulting affinities underperform (μ ≈ 0.538 vs ~0.563 for gABC in the heart comparison).

## Evidence

- [methods] STARE summarises the TF-affinities from regions that were assigned to that gene.: "STARE summarises the TF-affinities from regions that were assigned to that gene."
- [methods] Window size centred at the 5' TSS in which regions from the --bed_file will be considered for a gene: "Window size centred at the 5' TSS in which regions from the --bed_file will be considered for a gene"
- [methods] If you did the region-gene mapping with the ABC-score, we change the scaling and also distinguish between regions close to the promoter and more distant ones.: "If you did the region-gene mapping with the ABC-score, we change the scaling and also distinguish between regions close to the promoter and more distant ones."
- [readme] STARE is a framework that combines two tasks, which can also work independently:: "STARE is a framework that combines two tasks, which can also work independently:"
- [readme] The core functionality is the fast and efficient annotation of user provided candidate regions with TF affinities using TRAP: [evidence span withheld — anchor status: external_doc]
