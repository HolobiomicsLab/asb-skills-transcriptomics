---
name: expression-matrix-construction
description: 'Use when reproducing the paper''s infection expression matrix: you have
  TPM estimates from the Soybean Expression Atlas plus supplementary infection RNA-seq
  samples and need a filtered gene x sample matrix as coexpression network input.'
when_to_use_negative:
- Input is raw RNA-seq reads (FASTQ) requiring alignment and quantification rather
  than ready-made TPM estimates
- The target analysis needs raw counts or a count-based differential expression model
  instead of the TPM matrix used for coexpression network construction
- Samples are not soybean tissues infected with the five phytopathogenic fungi covered
  by the paper's 150-sample subset
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: R
  role: Language used to download, subset, median-filter, and assemble the TPM expression
    matrix
  repo: https://github.com/almeidasilvaf/SoyFungiGCN
- name: SoyFungiGCN
  role: R/Shiny package downstream consumer of the filtered matrix, containing the
    coexpression network of soybean tissues infected with phytopathogenic fungi
  repo: https://github.com/almeidasilvaf/SoyFungiGCN
- name: remotes
  role: Used to install SoyFungiGCN from GitHub for exploring/reproducing the pipeline
    context
  repo: https://github.com/almeidasilvaf/SoyFungiGCN
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-021-03864-x
    title: Integration of genome-wide association studies and gene coexpression networks
      unveils promising soybean resistance genes against five common fungal pathogens
  source_content_hash: sha256:da6b6a8fa3b45f2f0dc6e7bb57dc4c775fe392ca19c72080b93c0c49c1299e47
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/almeidasilvaf/SoyFungiGCN
  tools:
  - R
  - SoyFungiGCN
  - remotes
  - utils
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# expression-matrix-construction

## Summary

Assembles a filtered gene-by-sample TPM expression matrix for soybean tissues infected with five phytopathogenic fungi (C. gregata, F. graminearum, F. virguliforme, M. phaseolina, P. pachyrhizi), yielding the 18,748 x 150 matrix used to build the SoyFungiGCN coexpression network.

## When to use

Use when reproducing the paper's infection expression matrix: you have TPM estimates from the Soybean Expression Atlas plus supplementary infection RNA-seq samples and need a filtered gene x sample matrix as coexpression network input.

## When NOT to use

- Input is raw RNA-seq reads (FASTQ) requiring alignment and quantification rather than ready-made TPM estimates
- The target analysis needs raw counts or a count-based differential expression model instead of the TPM matrix used for coexpression network construction
- Samples are not soybean tissues infected with the five phytopathogenic fungi covered by the paper's 150-sample subset

## Inputs

- TPM expression estimates from the Soybean Expression Atlas
- Supplementary infection RNA-seq sample data referenced by the paper
- Sample metadata mapping the 150 samples to soybean-fungal infection conditions (C. gregata, F. graminearum, F. virguliforme, M. phaseolina, P. pachyrhizi)

## Outputs

- expression_matrix_filtered.tsv: 18,748 genes x 150 samples TPM matrix with all retained genes having median TPM >= 5

## How to apply

Download the TPM expression estimates from the Soybean Expression Atlas together with the supplementary infection RNA-seq sample data referenced by the paper. Subset the samples to the 150 samples corresponding to soybean–fungal infection conditions spanning the five phytopathogenic fungi. Compute the per-gene median TPM across those 150 samples and discard genes whose median expression is below 5 TPM, a cutoff applied to attenuate noise. Assemble the retained genes into the final filtered matrix and write it as expression_matrix_filtered.tsv; validate that it is exactly 18,748 genes x 150 samples with every retained gene having median TPM >= 5.

## Related tools

- **R** (Language used to download, subset, median-filter, and assemble the TPM expression matrix) — https://github.com/almeidasilvaf/SoyFungiGCN
- **SoyFungiGCN** (R/Shiny package downstream consumer of the filtered matrix, containing the coexpression network of soybean tissues infected with phytopathogenic fungi) — https://github.com/almeidasilvaf/SoyFungiGCN
- **remotes** (Used to install SoyFungiGCN from GitHub for exploring/reproducing the pipeline context) — https://github.com/almeidasilvaf/SoyFungiGCN

## Examples

```
remotes::install_github("almeidasilvaf/SoyFungiGCN"); write.table(expr_filtered, "expression_matrix_filtered.tsv", sep="\t", quote=FALSE)  # 18748 genes x 150 samples, median TPM >= 5
```

## Evaluation signals

- Output matrix expression_matrix_filtered.tsv has exactly 18,748 rows (genes) and 150 columns (samples)
- Every retained gene has a per-gene median TPM >= 5 across the 150 infection samples
- Sample subset covers all five phytopathogenic fungi: C. gregata, F. graminearum, F. virguliforme, M. phaseolina, and P. pachyrhizi
- Expression values are TPM estimates (not raw counts), traceable to the Soybean Expression Atlas plus the supplementary infection RNA-seq samples
- All genes with median expression below 5 TPM have been excluded, so no noise-attenuation filter step was skipped

## Limitations

- The 5-TPM median cutoff attenuates noise but may remove weakly expressed genes relevant to resistance
- Specific RNA-seq accessions/data sources are not named in the accessible text; supplementary tables referenced in the paper were not included/accessible
- The SoyFungiGCN package is marked 'experimental' lifecycle and no changelog or package version is stated

## Evidence

- [other] TPM filter and matrix dimensions: [evidence span withheld — anchor status: not_found]
- [readme] Package provides the coexpression network app: "SoyFungiGCN is an R package that contains a Shiny app to let users explore a gene coexpression network of soybean tissues infected with phytopathogenic fungi."
- [intro] Data integration objective: [evidence span withheld — anchor status: not_found]
- [abstract] Fungal species scope: "to prioritize candidate genes associated with resistance to Cadophora gregata, Fusarium graminearum, Fusarium virguliforme, Macrophomina phaseolina, and Phakopsora pachyrhizi"
