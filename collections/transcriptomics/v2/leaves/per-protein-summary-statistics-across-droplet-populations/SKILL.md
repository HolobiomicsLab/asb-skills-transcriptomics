---
name: per-protein-summary-statistics-across-droplet-populations
description: Use when you have a CITE-seq ADT count matrix split into empty (background)
  and cell-containing droplets and need to quantify or validate that per-protein ambient
  background levels are shared between the two populations.
when_to_use_negative:
- You only have a filtered cell matrix with no empty-droplet/background barcodes (dsb
  instead offers ModelNegativeADTnorm / no-empty-droplets mode)
- The goal is per-cell normalization or denoising (use DSBNormalizeProtein, not this
  diagnostic summary)
- Input is already log-transformed or normalized ADT values — recomputing log10(count+1)
  means would be invalid
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: dsb
  role: Supplies the example raw empty-droplet and cell-containing ADT matrices for
    the 87-antibody PBMC CITE-seq experiment
  repo: https://cran.r-project.org/package=dsb
- name: R
  role: Language used to compute per-protein mean log10(count+1) and Pearson correlation
    via cor()
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-29356-8
    title: Normalizing and denoising protein expression data from droplet-based single
      cell profiling
  source_content_hash: sha256:ffa0bf88d68c57105bf7cce2bd7da3e2eb2a5b5ac9c9ef0d34b0c80dc556ff1a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - dsb
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# per-protein summary statistics across droplet populations

## Summary

Computes per-protein (per-ADT) mean log10(count+1) within empty droplets and cell-containing droplets, then correlates the two vectors to show that ambient antibody levels in empty droplets track background protein captured by cells. This is the empirical basis for dsb's ambient-noise estimation in step I.

## When to use

Use when you have a CITE-seq ADT count matrix split into empty (background) and cell-containing droplets and need to quantify or validate that per-protein ambient background levels are shared between the two populations.

## When NOT to use

- You only have a filtered cell matrix with no empty-droplet/background barcodes (dsb instead offers ModelNegativeADTnorm / no-empty-droplets mode)
- The goal is per-cell normalization or denoising (use DSBNormalizeProtein, not this diagnostic summary)
- Input is already log-transformed or normalized ADT values — recomputing log10(count+1) means would be invalid

## Inputs

- Raw ADT/protein count matrix for cell-containing droplets (e.g. dsb::raw_pbmc_mtx_example_87ab_citeseq_Tsang, proteins x barcodes)
- Raw ADT/protein count matrix for empty droplets (e.g. dsb::raw_emptydrop_mtx_example_87ab_citeseq_Tsang)

## Outputs

- CSV table (ambient_correlation_results.csv) with columns: protein name, empty-droplet mean log10(count+1), cell mean log10(count+1), overall Pearson correlation coefficient
- Scalar Pearson correlation between per-protein empty-droplet and cell background mean vectors

## How to apply

Load the raw cell-containing droplet matrix (e.g. dsb::raw_pbmc_mtx_example_87ab_citeseq_Tsang) and the empty-droplet matrix (dsb::raw_emptydrop_mtx_example_87ab_citeseq_Tsang). For each of the 87 ADT proteins, compute the mean of log10(count+1) across all barcodes separately within each matrix. Compute the Pearson correlation between the two per-protein mean vectors with R's cor(), and write a table of protein name, empty-droplet mean, cell mean, and the overall correlation. The rationale is that protein-specific noise originates from unbound antibody encapsulated in droplets and can be estimated from ambient ADT counts in empty droplets, which the paper shows are highly correlated with background levels in cells.

## Related tools

- **dsb** (Supplies the example raw empty-droplet and cell-containing ADT matrices for the 87-antibody PBMC CITE-seq experiment) — https://cran.r-project.org/package=dsb
- **R** (Language used to compute per-protein mean log10(count+1) and Pearson correlation via cor())

## Examples

```
cor(colMeans(log10(raw_emptydrop_mtx + 1)), colMeans(log10(raw_pbmc_mtx + 1)))
```

## Evaluation signals

- Pearson correlation between per-protein empty-droplet and cell background mean log10(count+1) vectors should be strong and positive, consistent with Fig. 1b
- Output table contains exactly one row per protein (87 rows for the 87-antibody dataset)
- Positive protein counts should be detectable in the unstained control cells, matching the paper's finding
- Each per-protein mean is finite and non-negative after log10(count+1) transformation; no NA/NaN rows in the output table

## Limitations

- Defining a reasonable subset of barcodes as background droplets still requires care — dsb normalized values are sensitive to how background droplets are defined
- Very low-signal proteins (maximum raw UMI count < 5 across all cells) were removed prior to normalization in the paper (e.g. CD34 in two datasets); such proteins may give unstable summary statistics
- This is a diagnostic of ambient correlation; it does not itself correct cell-level technical noise (dsb step II / Gaussian mixture modeling is separate)

## Evidence

- [abstract] protein-specific noise originates from unbound antibodies encapsulated during droplet generation: "protein-specific noise originates from unbound antibodies encapsulated during droplet generation"
- [intro] ambient counts estimable from empty droplets: [evidence span withheld — anchor status: not_found]
- [discussion] In step I, protein counts in empty droplets are used to estimate the expected ambient background noise for each antibody: "In step I, protein counts in empty droplets are used to estimate the expected ambient background noise for each antibody"
- [readme] We observed that ADT reads from empty droplets—often more than tenfold the number of cell-containing droplets—closely match levels in: [evidence span withheld — anchor status: external_doc]
