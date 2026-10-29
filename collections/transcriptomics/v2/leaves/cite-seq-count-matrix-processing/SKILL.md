---
name: cite-seq-count-matrix-processing
description: Use when raw ADT/protein UMI count matrices from a CITE-seq experiment
  are available (e.g., raw and filtered Cell Ranger outputs) and you need to characterize
  ambient antibody background before normalization, or verify that empty-droplet background
  reflects protein levels in cells.
when_to_use_negative:
- Input is already normalized/denoised (e.g., dsb DSBNormalizeProtein output) rather
  than raw UMI counts — the correlation analysis requires raw counts
- The dataset contains no empty droplets (e.g., sorted-cell experiments) — use the
  dsb 'no empty drops' mode with ModelNegativeADTnorm instead
- The goal is full normalization and denoising rather than ambient background characterization
  — call DSBNormalizeProtein() directly instead of running this diagnostic workflow
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: dsb
  role: R package providing example 87-antibody PBMC CITE-seq count matrices (raw
    cell and empty-droplet matrices) and the downstream DSBNormalizeProtein() function
    that uses empty-droplet background in step I
  repo: https://cran.r-project.org/package=dsb
- name: R
  role: Language used for loading matrices, computing per-protein mean log10(count+1),
    and Pearson correlation via cor()
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-29356-8
    title: Normalizing and denoising protein expression data from droplet-based single
      cell profiling
  source_content_hash: sha256:1cbdfe1cf25875ca604bdf6ecac578be2bade28cf45cc5a94ab598e96880f007
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - dsb
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# cite-seq-count-matrix-processing

## Summary

Process raw CITE-seq ADT count matrices by loading cell-containing and empty-droplet UMI matrices, computing per-protein mean log10(count+1) levels, and quantifying the ambient antibody background shared between empty droplets and cells. This characterizes protein-specific noise from unbound antibodies encapsulated during droplet generation, the motivation for dsb normalization.

## When to use

Use when raw ADT/protein UMI count matrices from a CITE-seq experiment are available (e.g., raw and filtered Cell Ranger outputs) and you need to characterize ambient antibody background before normalization, or verify that empty-droplet background reflects protein levels in cells.

## When NOT to use

- Input is already normalized/denoised (e.g., dsb DSBNormalizeProtein output) rather than raw UMI counts — the correlation analysis requires raw counts
- The dataset contains no empty droplets (e.g., sorted-cell experiments) — use the dsb 'no empty drops' mode with ModelNegativeADTnorm instead
- The goal is full normalization and denoising rather than ambient background characterization — call DSBNormalizeProtein() directly instead of running this diagnostic workflow

## Inputs

- Raw cell-containing droplet ADT count matrix (protein x barcodes, e.g., filtered Cell Ranger output or dsb::raw_pbmc_mtx_example_87ab_citeseq_Tsang)
- Empty-droplet ADT count matrix (e.g., raw Cell Ranger output minus cell barcodes, or dsb::raw_emptydrop_mtx_example_87ab_citeseq_Tsang)
- List of ADT protein names (87 proteins in the PBMC 87-antibody example; antibody list in supplementary Data 1)

## Outputs

- Per-protein mean log10(count+1) values for empty droplets and for cells (one value per protein per matrix)
- Pearson correlation coefficient between empty-droplet and cell background protein levels
- Results table (e.g., ambient_correlation_results.csv) with columns: protein name, empty-droplet mean log10(count+1), cell mean log10(count+1), overall correlation

## How to apply

Load the raw cell-containing droplet matrix and the empty-droplet matrix as standard R matrices (e.g., dsb::raw_pbmc_mtx_example_87ab_citeseq_Tsang and dsb::raw_emptydrop_mtx_example_87ab_citeseq_Tsang for the 87-antibody PBMC CITE-seq dataset). For each ADT protein, compute the mean of log10(count + 1) across all barcodes separately within each matrix, mirroring the dsb log transformation step (dlog = log(cell + 10)). Compute the Pearson correlation between the two per-protein mean log10(count+1) vectors with R's cor(). Optionally define cells as barcodes in the filtered Cell Ranger output and background drops as the remaining barcodes with protein reads from the raw output. Write per-protein means and the overall correlation to a results table (e.g., ambient_correlation_results.csv). The rationale: ambient, unbound antibody encapsulated during droplet generation produces protein-specific noise estimable from empty-droplet counts, and strong positive correlation between empty-droplet and cell background levels validates its use as the background estimate in step I of dsb.

## Related tools

- **dsb** (R package providing example 87-antibody PBMC CITE-seq count matrices (raw cell and empty-droplet matrices) and the downstream DSBNormalizeProtein() function that uses empty-droplet background in step I) — https://cran.r-project.org/package=dsb
- **R** (Language used for loading matrices, computing per-protein mean log10(count+1), and Pearson correlation via cor())

## Examples

```
cell <- dsb::raw_pbmc_mtx_example_87ab_citeseq_Tsang; empty <- dsb::raw_emptydrop_mtx_example_87ab_citeseq_Tsang; cor(rowMeans(log10(empty+1)), rowMeans(log10(cell+1)))
```

## Evaluation signals

- Pearson correlation between per-protein empty-droplet and cell mean log10(count+1) vectors should be strong and positive, consistent with Fig. 1b and the paper's finding that 'Background levels in the empty droplets and cells were highly correlated'
- Output table contains exactly one row per ADT protein (87 rows for the 87-antibody PBMC dataset)
- Unstained control cells show positive protein counts, confirming ambient capture is detectable in cell-containing droplets
- Empty-droplet barcodes are defined as raw Cell Ranger barcodes minus the filtered (cell) barcodes; total raw droplets are often more than tenfold the number of cell-containing droplets

## Limitations

- Defining a reasonable subset of barcodes as background droplets still requires care; dsb normalized values are sensitive to how background droplets are defined
- Proteins with very low raw signal (maximum UMI count < 5 across all cells) should be removed prior to normalization (CD34 was removed from two datasets in the paper)
- The antibody list for the 87-antibody panel is only available in supplementary Data 1, not in the main text

## Evidence

- [intro] Ambient background estimable from empty droplets: [evidence span withheld — anchor status: not_found]
- [discussion] Empty-droplet definition: "Cells were defined as barcodes in the filtered Cell Ranger output, and background drops were defined as after removing the cells from the from the raw Cell Ranger output"
- [readme] Background-closeness observation: [evidence span withheld — anchor status: external_doc]
- [methods] Log transformation step: "# log transformation
dlog = log(cell + 10)"
- [discussion] Step I uses empty droplets: "In step I, protein counts in empty droplets are used to estimate the expected ambient background noise for each antibody"
