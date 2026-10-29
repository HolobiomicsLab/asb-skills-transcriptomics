---
name: replicate-pearson-correlation-analysis
description: Use when a normalized barcode counts table (e.g. counts/all_counts_combined.tsv)
  contains biological replicates per treatment (e.g. DMSO, IBET-151) and replicate
  concordance must be quantified before downstream clonal analyses.
when_to_use_negative:
- The input is raw, unnormalized barcode counts with differing library sizes — normalize
  first, otherwise correlation is driven by sequencing depth rather than clonal composition
- Samples lack biological replicates per condition (e.g. single AraC IC90 samples)
  — pairwise replicate correlation cannot be computed
- The goal is barcode read pre-processing quality control (quality, UMI, chimerism
  filtering) — that is handled upstream in the BARtab pipeline, not by replicate correlation
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: bartools
  role: R package used to load, normalize, and analyze the dose-escalation barcode
    counts table and support replicate concordance assessment
  repo: https://github.com/DaneVass/bartools
- name: R
  role: Environment in which cor(method='pearson') is computed on log-normalized barcode
    counts
- name: tidyverse
  role: Used to load and wrangle the counts/all_counts_combined.tsv table prior to
    correlation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100763
    title: Analysis of synthetic cellular barcodes in the genome and transcriptome
      with BARtab and bartools
  source_content_hash: sha256:6dc0ac1a969d460af6dcfd4d348f808e5ee3473b207a6f5b045183534bf8ecdc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DaneVass/bartools
  tools:
  - BARtab
  - R
  - bartools
  - tidyverse
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# replicate-pearson-correlation-analysis

## Summary

Quantifies concordance of biological replicates in a cellular barcoding experiment by computing pairwise Pearson correlations between replicate samples over log-normalized barcode counts, validating that replicate clonal composition is highly similar (correlations > 0.95).

## When to use

Use when a normalized barcode counts table (e.g. counts/all_counts_combined.tsv) contains biological replicates per treatment (e.g. DMSO, IBET-151) and replicate concordance must be quantified before downstream clonal analyses.

## When NOT to use

- The input is raw, unnormalized barcode counts with differing library sizes — normalize first, otherwise correlation is driven by sequencing depth rather than clonal composition
- Samples lack biological replicates per condition (e.g. single AraC IC90 samples) — pairwise replicate correlation cannot be computed
- The goal is barcode read pre-processing quality control (quality, UMI, chimerism filtering) — that is handled upstream in the BARtab pipeline, not by replicate correlation

## Inputs

- Normalized combined barcode counts table (counts/all_counts_combined.tsv) with sample metadata (treatment, time point, replicate)
- Log-normalized barcode abundance matrix (barcodes x samples) in R

## Outputs

- Pairwise Pearson correlation matrix between replicate samples per treatment
- Per-treatment replicate correlation values with a > 0.95 pass/fail assessment
- Optionally, a correlation dissimilarity matrix (1 - cor) for hierarchical sample clustering

## How to apply

Load the normalized combined barcode counts table with tidyverse in R and log-normalize barcode counts across samples to stabilize variance before computing associations. For each treatment group with replicates (DMSO and IBET-151 in the dose-escalation dataset), compute all pairwise Pearson correlations across barcode abundance vectors using R's cor(method='pearson') or bartools helpers. Judge success by the article's stated validation criterion: all DMSO and IBET-151 replicate correlations must exceed 0.95. The same correlation matrix can be reused as a dissimilarity (1 - cor) input for hierarchical clustering of samples by barcode composition. Report per-treatment minimum correlation as the headline QC metric alongside PCA treatment separation.

## Related tools

- **bartools** (R package used to load, normalize, and analyze the dose-escalation barcode counts table and support replicate concordance assessment) — https://github.com/DaneVass/bartools
- **R** (Environment in which cor(method='pearson') is computed on log-normalized barcode counts)
- **tidyverse** (Used to load and wrangle the counts/all_counts_combined.tsv table prior to correlation)

## Examples

```
cor(log_norm_counts[, paste0("DMSO_rep", 1:2)], method = "pearson")
```

## Evaluation signals

- All pairwise Pearson correlations between DMSO and IBET-151 biological replicates exceed 0.95
- Correlations are computed on log-normalized barcode counts, not raw counts
- Replicate pairs cluster together when samples are hierarchically clustered on correlation dissimilarity (1 - cor)
- Treatment groups separate along PC1 in the accompanying PCA, consistent with replicate similarity within groups
- The reported minimum replicate correlation is explicitly stated as a validation metric in the analysis output

## Limitations

- The article notes benchmarking was hindered because 'there was no gold-standard dataset with known barcode abundance', so a universal correlation threshold other than the reported > 0.95 is not externally validated
- Pearson correlation is sensitive to low-count barcodes and sequencing depth; thresholds may not transfer to datasets with different barcode library designs or fewer supporting reads

## Evidence

- [other] Compute pairwise Pearson correlations between biological replicates for DMSO and IBET-151 samples and verify correlations exceed 0.95.: [evidence span withheld — anchor status: not_found]
- [results] replicate samples displayed similar overall clonal composition with DMSO and IBET-151 correlations all > 0.95.: [evidence span withheld — anchor status: not_found]
- [intro] we developed bartools, a flexible open-source R package that incorporates workflows for population-level cellular barcoding data analysis: "we developed bartools, a flexible open-source R package that incorporates workflows for population-level cellular barcoding data analysis"
- [readme] Tools for the analysis of cellular barcoding datasets: [evidence span withheld — anchor status: external_doc]
