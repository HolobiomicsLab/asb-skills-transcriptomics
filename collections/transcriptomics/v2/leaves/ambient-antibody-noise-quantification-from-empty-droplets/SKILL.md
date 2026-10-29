---
name: ambient-antibody-noise-quantification-from-empty-droplets
description: Use when you have raw ADT/UMI count matrices from both empty droplets
  and cell-containing droplets (e.g. raw vs filtered Cell Ranger output from a CITE-seq
  run) and need to characterize or validate ambient, unbound antibody background per
  protein.
when_to_use_negative:
- The dataset has no empty droplets available (use the dsb 'no empty drops' strategy
  / ModelNegativeADTnorm or define background from the modeled negative population
  instead)
- The ADT matrix is already dsb-normalized or CLR-normalized — the correlation must
  be computed on raw counts with log10(count+1) transformation
- Proteins with very low signal (maximum raw UMI count < 5 across all cells) — the
  paper removes such proteins (e.g. CD34) prior to normalization
edam_operation: http://edamontology.org/operation_3463
tools:
- name: dsb
  role: Provides the raw empty-droplet and cell-containing ADT count matrices used
    as inputs, and implements the downstream ambient-background correction (DSBNormalizeProtein,
    step I) that this quantification validates
  repo: https://cran.r-project.org/package=dsb
- name: R
  role: Base R stats used for the log10(count+1) transformation and cor() Pearson
    correlation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-29356-8
    title: Normalizing and denoising protein expression data from droplet-based single
      cell profiling
  source_content_hash: sha256:db2e14e04e9aedfd56fbd989082ef18f5e520f838e29e05819c363953f67da87
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - dsb
  edam_operation: http://edamontology.org/operation_3463
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# ambient-antibody-noise-quantification-from-empty-droplets

## Summary

Quantifies protein-specific ambient antibody noise in CITE-seq by computing per-antibody mean log10(count+1) in empty droplets and cell-containing droplets, then correlating the two vectors to demonstrate that background ADT levels in empty droplets closely match levels in cells.

## When to use

Use when you have raw ADT/UMI count matrices from both empty droplets and cell-containing droplets (e.g. raw vs filtered Cell Ranger output from a CITE-seq run) and need to characterize or validate ambient, unbound antibody background per protein.

## When NOT to use

- The dataset has no empty droplets available (use the dsb 'no empty drops' strategy / ModelNegativeADTnorm or define background from the modeled negative population instead)
- The ADT matrix is already dsb-normalized or CLR-normalized — the correlation must be computed on raw counts with log10(count+1) transformation
- Proteins with very low signal (maximum raw UMI count < 5 across all cells) — the paper removes such proteins (e.g. CD34) prior to normalization

## Inputs

- Raw ADT/protein count matrix for cell-containing droplets (e.g. dsb::raw_pbmc_mtx_example_87ab_citeseq_Tsang)
- Raw ADT/protein count matrix for empty droplets (e.g. dsb::raw_emptydrop_mtx_example_87ab_citeseq_Tsang)
- Protein/antibody names (matrix row names, 87 antibodies)

## Outputs

- Per-protein table (ambient_correlation_results.csv) with columns: protein name, empty-droplet mean log10(count+1), cell mean log10(count+1), overall Pearson correlation coefficient
- Pearson correlation coefficient (expected strongly positive, per Fig. 1b)

## How to apply

Load the raw cell-containing ADT matrix (e.g. dsb::raw_pbmc_mtx_example_87ab_citeseq_Tsang) and the empty-droplet ADT matrix (dsb::raw_emptydrop_mtx_example_87ab_citeseq_Tsang). For each of the 87 ADT proteins, compute the mean of log10(count + 1) across all barcodes separately within each matrix, since the dsb workflow log-transforms ADT counts before background estimation. Compute the Pearson correlation between the two 87-length vectors with R's cor(). Rationale: the paper identifies protein-specific noise as originating from unbound antibodies encapsulated during droplet generation, estimable via ambient ADT counts in empty droplets; this correlation establishes that empty-droplet background is a valid estimate of the ambient component captured in cells (Fig. 1b). Write results as a table of protein name, empty-droplet mean, cell mean, and the overall correlation coefficient.

## Related tools

- **dsb** (Provides the raw empty-droplet and cell-containing ADT count matrices used as inputs, and implements the downstream ambient-background correction (DSBNormalizeProtein, step I) that this quantification validates) — https://cran.r-project.org/package=dsb
- **R** (Base R stats used for the log10(count+1) transformation and cor() Pearson correlation)

## Examples

```
Rscript -e 'library(dsb); c<-colMeans(log10(raw_pbmc_mtx_example_87ab_citeseq_Tsang+1)); e<-colMeans(log10(raw_emptydrop_mtx_example_87ab_citeseq_Tsang+1)); cor(e,c)'
```

## Evaluation signals

- Pearson correlation between the 87 per-protein empty-droplet and cell mean log10(count+1) vectors is strong and positive, matching Fig. 1b of the paper
- Output table contains exactly one row per protein (87 rows for the PBMC 87-antibody dataset)
- Per-protein means are finite and non-negative, computed as log10(count+1) over all barcodes in each matrix
- Positive protein counts are observed in unstained control cells, consistent with the paper's finding of ambient capture

## Limitations

- dsb normalized values are sensitive to how background droplets are defined; 'defining a reasonable subset of barcodes as background droplets still requires care'
- Empty droplets can vastly outnumber cell-containing droplets (often more than tenfold), so summary statistics on the empty matrix may dominate computation
- Assessment details such as robustness of noise estimation and empty-droplet definition sensitivity are deferred to the Supplementary Note, not the main text

## Evidence

- [intro] ambient noise estimable from empty droplets: [evidence span withheld — anchor status: not_found]
- [discussion] step I definition: "In step I, protein counts in empty droplets are used to estimate the expected ambient background noise for each antibody"
- [methods] cell/background drop definition: "Cells were defined as barcodes in the filtered Cell Ranger output, and background drops were defined as after removing the cells from the from the raw Cell Ranger output"
- [readme] readme empirical support: [evidence span withheld — anchor status: external_doc]
- [abstract] noise source identification: "protein-specific noise originates from unbound antibodies encapsulated during droplet generation"
