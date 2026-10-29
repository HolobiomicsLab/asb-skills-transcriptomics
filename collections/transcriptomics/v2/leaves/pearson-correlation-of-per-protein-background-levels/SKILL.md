---
name: pearson-correlation-of-per-protein-background-levels
description: Use when a CITE-seq (or ASAP-seq/TEA-seq) ADT dataset has both a raw
  empty-droplet matrix and a cell-containing matrix, and you need to verify that empty-droplet
  counts faithfully capture the ambient antibody background before using them for
  dsb step I correction.
when_to_use_negative:
- When the dataset provides no empty-droplet (background) matrix — dsb documentation
  notes defining a reasonable subset of barcodes as background droplets still requires
  care, and the 'no empty drops' vignette should be used instead
- For RNA expression matrices — this skill applies to antibody-derived tag (ADT) protein
  counts, not mRNA counts
- To validate the cell-level denoising (dsb step II) — this skill only checks the
  ambient/background correspondence, not the Gaussian-mixture or technical-component
  correction
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3520
tools:
- name: dsb
  role: Supplies the example raw empty-droplet and cell-containing 87-antibody CITE-seq
    matrices used as correlation inputs; dsb step I uses the same empty-droplet levels
    as the ambient background estimate
  repo: https://cran.r-project.org/package=dsb
- name: R
  role: 'Computation environment: per-protein mean of log10(count+1) and cor() for
    the Pearson correlation'
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-29356-8
    title: Normalizing and denoising protein expression data from droplet-based single
      cell profiling
  source_content_hash: sha256:4537a664a37922d3158c5ab13c93516cb287ed15d6268c4bbcdf23f690313ec0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - dsb
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# pearson-correlation-of-per-protein-background-levels

## Summary

Quantifies the correspondence between ambient antibody levels in cell-free (empty) droplets and unstained/cell-containing droplets by correlating per-protein mean log10(count+1) values across the 87-antibody CITE-seq panel, validating that empty-droplet ADT counts estimate the ambient noise each antibody contributes to cells.

## When to use

Use when a CITE-seq (or ASAP-seq/TEA-seq) ADT dataset has both a raw empty-droplet matrix and a cell-containing matrix, and you need to verify that empty-droplet counts faithfully capture the ambient antibody background before using them for dsb step I correction.

## When NOT to use

- When the dataset provides no empty-droplet (background) matrix — dsb documentation notes defining a reasonable subset of barcodes as background droplets still requires care, and the 'no empty drops' vignette should be used instead
- For RNA expression matrices — this skill applies to antibody-derived tag (ADT) protein counts, not mRNA counts
- To validate the cell-level denoising (dsb step II) — this skill only checks the ambient/background correspondence, not the Gaussian-mixture or technical-component correction

## Inputs

- Raw empty-droplet ADT UMI count matrix (proteins x barcodes), e.g., dsb::raw_emptydrop_mtx_example_87ab_citeseq_Tsang
- Raw cell-containing droplet ADT UMI count matrix (proteins x barcodes), e.g., dsb::raw_pbmc_mtx_example_87ab_citeseq_Tsang

## Outputs

- Per-protein table (one row per antibody, e.g., 87 rows) with empty-droplet mean log10(count+1), cell mean log10(count+1), and the overall Pearson correlation coefficient (e.g., ambient_correlation_results.csv)
- Single Pearson correlation coefficient quantifying agreement between ambient and cellular background levels

## How to apply

Load the raw cell-containing droplet matrix (e.g., dsb::raw_pbmc_mtx_example_87ab_citeseq_Tsang) and the empty-droplet matrix (dsb::raw_emptydrop_mtx_example_87ab_citeseq_Tsang) from the dsb R package. For each of the panel's ADT proteins (87 in the example), compute the mean of log10(count + 1) across all barcodes separately within the empty-droplet matrix and the cell-containing matrix. Compute the Pearson correlation coefficient between the two per-protein mean vectors with R's cor(); the paper's Fig. 1b shows this is strong and positive, supporting use of empty-droplet levels as the ambient background estimate. Write one row per protein with both means and the overall correlation (e.g., ambient_correlation_results.csv) and check the table has exactly one row per antibody and a strong positive coefficient.

## Related tools

- **dsb** (Supplies the example raw empty-droplet and cell-containing 87-antibody CITE-seq matrices used as correlation inputs; dsb step I uses the same empty-droplet levels as the ambient background estimate) — https://cran.r-project.org/package=dsb
- **R** (Computation environment: per-protein mean of log10(count+1) and cor() for the Pearson correlation)

## Examples

```
R: e <- log10(dsb::raw_emptydrop_mtx_example_87ab_citeseq_Tsang + 1); c <- log10(dsb::raw_pbmc_mtx_example_87ab_citeseq_Tsang + 1); cor(rowMeans(e), rowMeans(c))
```

## Evaluation signals

- Output table contains exactly one row per antibody (87 rows for the example panel)
- Pearson correlation between per-protein empty-droplet and cell mean log10(count+1) vectors is strong and positive, consistent with the paper's Fig. 1b finding
- Positive protein counts are detectable in unstained/control (empty) droplets, confirming ambient signal exists to correlate
- Means are computed on the log10(count+1) scale consistently in both matrices before correlating

## Limitations

- Sensitivity of downstream dsb normalized values to how background droplets are defined — defining a reasonable subset of barcodes as background droplets still requires care
- The correlation establishes ambient correspondence but does not by itself validate cell-to-cell technical noise correction, which requires isotype controls or the dsb technical component
- dsb caveats for datasets without isotype control measurements are deferred to the Supplementary Note, so the ambient correlation alone cannot substitute for full validation

## Evidence

- [abstract] protein-specific noise originates from unbound antibodies encapsulated during droplet generation: "protein-specific noise originates from unbound antibodies encapsulated during droplet generation"
- [intro] ambient estimable from empty droplets: [evidence span withheld — anchor status: not_found]
- [discussion] empty droplets used as ambient estimate: "In step I, protein counts in empty droplets are used to estimate the expected ambient background noise for each antibody"
- [discussion] background definition care: "defining a reasonable subset of barcodes as background droplets still requires care"
- [readme] empty droplets match cell levels: [evidence span withheld — anchor status: external_doc]
