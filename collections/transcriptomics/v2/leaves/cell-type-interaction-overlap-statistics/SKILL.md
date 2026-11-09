---
name: cell-type-interaction-overlap-statistics
description: Use when gABC/ABC-scored enhancer-gene interactions have been predicted
  per cell type from snATAC-seq candidate enhancers and you need per-cell-type interaction
  counts plus cross-cell-type overlap/shared fractions.
when_to_use_negative:
- You only have a single bulk enhancer activity profile with no per-cell-type activity
  columns — there are no cell-type-specific interaction sets to compare.
- Your enhancer coordinates and Hi-C contacts are not on the same genome build/window
  scheme (e.g., hg38 enhancers against the hg19 average Hi-C matrix) — the 5 Mb window
  and 5000 bp binning would silently mismatch.
- Your goal is TF affinity summarisation on gene level (GAZE) rather than counting
  ABC-scored enhancer-gene interactions — the separate ABC-scoring mode and overlap
  statistics are then the wrong endpoint.
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: STARE
  role: Framework providing the separate ABC-scoring mode (STARE_ABCpp) used to predict
    cell-type-specific enhancer-gene interactions and the Juicebox_normalization.sh
    script used to prepare contact files
  repo: https://github.com/schulzlab/STARE
- name: stare-abc
  role: Bioconda package that installs the STARE ABC-scoring executable used with
    parameters -w, -k, -i, -q, -t
  repo: http://bioconda.github.io/recipes/stare-abc/README.html
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad062
    title: The adapted Activity-By-Contact model for enhancer–gene assignment and
      its application to single-cell data
  source_content_hash: sha256:3e783fba2fa86f0b55b351c01d6893b7f06bc1f61a2cad56385e9288f82585b6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/schulzlab/STARE
  tools:
  - STARE
  - stare-abc
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# cell-type-interaction-overlap-statistics

## Summary

Quantify and compare gABC-scored enhancer-gene interactions predicted per cell type from single-cell ATAC-seq data, measuring how many interactions each cell type shares with all others. This reveals the extent of cell-type specificity in the predicted regulatory landscape (e.g., of the human heart snATAC-seq dataset).

## When to use

Use when gABC/ABC-scored enhancer-gene interactions have been predicted per cell type from snATAC-seq candidate enhancers and you need per-cell-type interaction counts plus cross-cell-type overlap/shared fractions.

## When NOT to use

- You only have a single bulk enhancer activity profile with no per-cell-type activity columns — there are no cell-type-specific interaction sets to compare.
- Your enhancer coordinates and Hi-C contacts are not on the same genome build/window scheme (e.g., hg38 enhancers against the hg19 average Hi-C matrix) — the 5 Mb window and 5000 bp binning would silently mismatch.
- Your goal is TF affinity summarisation on gene level (GAZE) rather than counting ABC-scored enhancer-gene interactions — the separate ABC-scoring mode and overlap statistics are then the wrong endpoint.

## Inputs

- Per-chromosome gzipped normalized Hi-C contact files in bin|bin|contact format (5000 bp bins, KR-normalized, from average_hic.v2.191020 Fulco et al. 2019 average hg19 matrix)
- Pooled candidate enhancer BED file (286,777 enhancers) with per-cell-type enhancer activity columns (Hocker 2021 human heart snATAC-seq)
- Full Gencode gene annotation file in gtf format

## Outputs

- Per-cell-type gABC-scored interaction files: *_ABCpp_scoredInteractions_*.txt.gz (interactions surpassing the 0.02 cut-off)
- Count of gABC-scored interactions per cell type (expected mean ~408,846, SD ~12,200 across eight heart cell types)
- Fraction of each cell type's interactions shared with all other cell types (expected mean ~23.6%)
- Pairwise cell-type intersection counts, e.g., aCM-vCM intersection of 39,707 interactions

## How to apply

Prepare per-chromosome gzipped normalized contact files (bin|bin|contact format) from the Fulco et al. (2019) average hg19 Hi-C matrix at 5000 bp resolution using STARE's Juicebox_normalization.sh with KR normalization. For each cell type, run STARE_ABCpp in separate ABC-scoring mode on the pooled enhancer bed file (-b) with that cell type's activity column (-n), full Gencode gtf (-a), 5,000,000 bp window (-w), 5000 bp bin size (-k), all_tss mode (-i), adapted activity enabled (-q True), and score cut-off 0.02 (-t 0.02). Collect the *_ABCpp_scoredInteractions_*.txt.gz outputs containing interactions passing the 0.02 cut-off, count interactions per cell type, then compute the fraction of each cell type's interactions shared with all other cell types and pairwise intersections (notably aCM-vCM). The rationale is that the generalized ABC model adapts enhancer activity to the number of contacts and considers all annotated TSS, yielding cell-type-specific interaction sets whose overlap quantifies regulatory sharing.

## Related tools

- **STARE** (Framework providing the separate ABC-scoring mode (STARE_ABCpp) used to predict cell-type-specific enhancer-gene interactions and the Juicebox_normalization.sh script used to prepare contact files) — https://github.com/schulzlab/STARE
- **stare-abc** (Bioconda package that installs the STARE ABC-scoring executable used with parameters -w, -k, -i, -q, -t) — http://bioconda.github.io/recipes/stare-abc/README.html

## Examples

```
STARE_ABCpp -b pooled_enhancers.bed -n celltype_activity.txt -a gencode.gtf -w 5000000 -f contacts_5kb/ -k 5000 -i all_tss -q True -t 0.02
```

## Evaluation signals

- Mean number of gABC-scored interactions per cell type is ~408,846 (SD ~12,200) across the eight heart cell types.
- Mean fraction of a cell type's interactions shared with all other cell types is ~23.6%.
- The aCM-vCM two-cell-type intersection equals exactly 39,707 interactions.
- All reported interactions per cell type come from *_ABCpp_scoredInteractions_*.txt.gz files and satisfy the 0.02 score cut-off within the 5,000,000 bp window.
- Contact files were validated as 5000 bp KR-normalized bin|bin|contact format before scoring (runTestCases.sh passes after install).

## Limitations

- The workflow relies on an averaged Hi-C matrix (Fulco et al. 2019 average hg19) rather than cell-type-specific chromatin contacts, so interaction overlap reflects shared activity patterns over a common contact scaffold.
- Interaction counts are sensitive to the score cut-off (0.02) and window size (5 Mb); different thresholds change both counts and overlap fractions.
- Adapted activity (-q True) and all_tss mode are required settings; running without adaptation changes the scoring and prevents the adapted activity from being used for scaling.

## Evidence

- [methods] If you're only interested in the ABC-scored interactions, you can also call that part independently.: "If you're only interested in the ABC-scored interactions, you can also call that part independently."
- [methods] Window size centred at the 5' TSS in which regions from the --bed_file will be considered for a gene: "Window size centred at the 5' TSS in which regions from the --bed_file will be considered for a gene"
- [abstract] We combined our generalized ABC model with TF binding information and illustrated an analysis of a single-cell ATAC-seq dataset of the human heart, where we were able to characterize cell: "We combined our generalized ABC model with TF binding information and illustrated an analysis of a single-cell ATAC-seq dataset of the human heart, where we were able to characterize cell"
- [abstract] The ABC score can yield higher accuracy by adapting enhancer activity according to the number of contacts the enhancer has to its candidate target genes: [evidence span withheld — anchor status: not_found]
