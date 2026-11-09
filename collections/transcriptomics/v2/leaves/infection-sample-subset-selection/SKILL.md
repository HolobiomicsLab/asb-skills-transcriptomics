---
name: infection-sample-subset-selection
description: Use when building a soybean–fungal infection expression matrix from Soybean
  Expression Atlas TPM data plus supplementary infection RNA-seq samples and a gene-level
  noise filter is needed before coexpression network construction.
when_to_use_negative:
- Input data are not soybean–fungal infection samples (e.g. non-infection tissues
  or other host–pathogen systems), since the 150-sample subset and five-fungus scope
  are specific to this study
- Expression values are raw counts or already variance-stabilized/normalized measures
  other than TPM, for which the median-TPM ≥ 5 noise filter is not appropriate
- The goal is to explore the pre-built coexpression network interactively rather than
  reconstruct the filtered matrix — the SoyFungiGCN Shiny app serves that purpose
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: R
  role: Scripting environment used to subset the 150 infection samples, compute per-gene
    median TPM, apply the ≥ 5 TPM filter, and assemble the matrix
- name: utils
  role: R package used to write the filtered expression matrix as expression_matrix_filtered.tsv
- name: SoyFungiGCN
  role: R/Shiny package distributing the resulting soybean–fungal infection coexpression
    network and related data
  repo: https://github.com/almeidasilvaf/SoyFungiGCN
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-021-03864-x
    title: Integration of genome-wide association studies and gene coexpression networks
      unveils promising soybean resistance genes against five common fungal pathogens
  source_content_hash: sha256:a9f63326b114f2441285f987391a5027cb0d18a8292a4e87249ed1dfeba6552e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/almeidasilvaf/SoyFungiGCN
  tools:
  - R
  - SoyFungiGCN
  - utils
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# infection-sample-subset-selection

## Summary

Subsets RNA-seq samples to the 150 soybean samples representing infection with five phytopathogenic fungi (C. gregata, F. graminearum, F. virguliforme, M. phaseolina, P. pachyrhizi), then filters genes by a median TPM ≥ 5 cutoff to yield a low-noise 18,748 x 150 expression matrix for downstream coexpression analyses.

## When to use

Use when building a soybean–fungal infection expression matrix from Soybean Expression Atlas TPM data plus supplementary infection RNA-seq samples and a gene-level noise filter is needed before coexpression network construction.

## When NOT to use

- Input data are not soybean–fungal infection samples (e.g. non-infection tissues or other host–pathogen systems), since the 150-sample subset and five-fungus scope are specific to this study
- Expression values are raw counts or already variance-stabilized/normalized measures other than TPM, for which the median-TPM ≥ 5 noise filter is not appropriate
- The goal is to explore the pre-built coexpression network interactively rather than reconstruct the filtered matrix — the SoyFungiGCN Shiny app serves that purpose

## Inputs

- TPM expression estimates from the Soybean Expression Atlas
- Supplementary infection RNA-seq sample data referenced by the paper
- Sample metadata distinguishing the 150 soybean–fungal infection samples from other tissues

## Outputs

- expression_matrix_filtered.tsv — an 18,748 genes x 150 samples filtered gene expression matrix (TPM)

## How to apply

Download TPM expression estimates from the Soybean Expression Atlas together with the supplementary infection RNA-seq sample data referenced by the paper. Subset the sample columns to the 150 samples corresponding to soybean–fungal infection conditions spanning the five phytopathogenic fungi. Compute the per-gene median TPM across the 150 infection samples and discard genes with median expression below 5 TPM, a threshold chosen to attenuate noise. Assemble the retained genes into the filtered matrix and write it as expression_matrix_filtered.tsv (in R, e.g. with write.table from the utils package).

## Related tools

- **R** (Scripting environment used to subset the 150 infection samples, compute per-gene median TPM, apply the ≥ 5 TPM filter, and assemble the matrix)
- **utils** (R package used to write the filtered expression matrix as expression_matrix_filtered.tsv)
- **SoyFungiGCN** (R/Shiny package distributing the resulting soybean–fungal infection coexpression network and related data) — https://github.com/almeidasilvaf/SoyFungiGCN

## Examples

```
expr_filtered <- expr[apply(expr, 1, median) >= 5, ]; write.table(expr_filtered, "expression_matrix_filtered.tsv", sep="\t", quote=FALSE)
```

## Evaluation signals

- Output matrix dimensions are exactly 18,748 genes x 150 samples
- Every retained gene has a median TPM ≥ 5 across the 150 infection samples
- Sample columns correspond only to soybean–fungal infection conditions spanning the five fungi (C. gregata, F. graminearum, F. virguliforme, M. phaseolina, P. pachyrhizi)
- Output file is a tab-separated file named expression_matrix_filtered.tsv with TPM values
- No genes with median expression below 5 TPM remain in the filtered matrix

## Limitations

- The 150-sample count and 18,748-gene dimensions depend on the exact Soybean Expression Atlas release and the supplementary infection RNA-seq samples referenced by the paper; different data versions may not reproduce them
- The median TPM ≥ 5 cutoff is a noise-attenuation heuristic and will discard genuinely low-expressing infection-responsive genes
- No changelog or version is stated for the SoyFungiGCN resource, so data provenance over time is uncertain

## Evidence

- [other] Filtered matrix dimensions and sources: [evidence span withheld — anchor status: not_found]
- [other] Sample subsetting across five fungi: [evidence span withheld — anchor status: not_found]
- [other] Median TPM filter: [evidence span withheld — anchor status: not_found]
- [other] Rationale for filtering: [evidence span withheld — anchor status: not_found]
- [readme] Package scope: "SoyFungiGCN is an R package that contains a Shiny app to let users explore a gene coexpression network of soybean tissues infected with phytopathogenic fungi."
