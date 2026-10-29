---
name: rna-seq-tpm-expression-retrieval
description: Use when building a soybean–fungal infection expression dataset (C. gregata,
  F. graminearum, F. virguliforme, M. phaseolina, P. pachyrhizi) from public TPM estimates
  to feed coexpression or differential analyses.
when_to_use_negative:
- Input is raw sequencing data (FASTQ) requiring alignment or quantification rather
  than pre-computed TPM estimates
- The analysis targets non-infection soybean tissues or pathogens outside the five
  fungi (C. gregata, F. graminearum, F. virguliforme, M. phaseolina, P. pachyrhizi),
  so the 150-sample infection subset and its noise threshold do not apply
- A different expression unit (e.g., raw counts for DESeq2-style modeling) is required,
  since this skill relies on TPM estimates from the Soybean Expression Atlas
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: SoyFungiGCN
  role: R/Shiny package that hosts the coexpression network built from the retrieved
    and filtered TPM expression matrix, for exploring soybean-pathogenic fungi interactions
  repo: https://github.com/almeidasilvaf/SoyFungiGCN
- name: R
  role: Language used to download/subset the Soybean Expression Atlas TPM data, apply
    the median TPM ≥ 5 filter, and assemble the final matrix
- name: remotes
  role: Used to install the SoyFungiGCN package from GitHub so the downstream network
    resource can consume the filtered expression data
  repo: https://github.com/almeidasilvaf/SoyFungiGCN
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-021-03864-x
    title: Integration of genome-wide association studies and gene coexpression networks
      unveils promising soybean resistance genes against five common fungal pathogens
  source_content_hash: sha256:2eadf0da1c682acf61f196f048ff71c66b31232452b9ea0e516f5260dc747b0c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/almeidasilvaf/SoyFungiGCN
  tools:
  - R
  - SoyFungiGCN
  - remotes
  - utils
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# rna-seq-tpm-expression-retrieval

## Summary

Retrieves TPM expression estimates from the Soybean Expression Atlas plus supplementary infection RNA-seq sample data, subsets them to soybean–fungal infection samples, and applies a median-TPM noise filter to assemble a filtered gene expression matrix for coexpression/network analyses.

## When to use

Use when building a soybean–fungal infection expression dataset (C. gregata, F. graminearum, F. virguliforme, M. phaseolina, P. pachyrhizi) from public TPM estimates to feed coexpression or differential analyses.

## When NOT to use

- Input is raw sequencing data (FASTQ) requiring alignment or quantification rather than pre-computed TPM estimates
- The analysis targets non-infection soybean tissues or pathogens outside the five fungi (C. gregata, F. graminearum, F. virguliforme, M. phaseolina, P. pachyrhizi), so the 150-sample infection subset and its noise threshold do not apply
- A different expression unit (e.g., raw counts for DESeq2-style modeling) is required, since this skill relies on TPM estimates from the Soybean Expression Atlas

## Inputs

- TPM expression estimates from the Soybean Expression Atlas
- Supplementary infection RNA-seq sample data referenced by the paper
- Sample metadata mapping the 150 samples to soybean–fungal infection conditions (five phytopathogenic fungi)

## Outputs

- Filtered gene expression matrix of 18,748 genes × 150 samples written as expression_matrix_filtered.tsv
- Every retained gene has a median TPM ≥ 5 across the 150 infection samples

## How to apply

Download the TPM expression estimates from the Soybean Expression Atlas together with the supplementary infection RNA-seq sample data referenced by the paper. Subset to the 150 samples corresponding to soybean–fungal infection conditions spanning the five phytopathogenic fungi. Compute the per-gene median TPM across the 150 infection samples and discard genes whose median expression is below 5 TPM to attenuate noise. Assemble the retained genes into the final filtered matrix (expected to be 18,748 genes × 150 samples) and write it as expression_matrix_filtered.tsv for downstream coexpression network construction. The resulting network is distributed via the SoyFungiGCN R/Shiny package and web application, so the filtered matrix must be compatible with that R ecosystem.

## Related tools

- **SoyFungiGCN** (R/Shiny package that hosts the coexpression network built from the retrieved and filtered TPM expression matrix, for exploring soybean-pathogenic fungi interactions) — https://github.com/almeidasilvaf/SoyFungiGCN
- **R** (Language used to download/subset the Soybean Expression Atlas TPM data, apply the median TPM ≥ 5 filter, and assemble the final matrix)
- **remotes** (Used to install the SoyFungiGCN package from GitHub so the downstream network resource can consume the filtered expression data) — https://github.com/almeidasilvaf/SoyFungiGCN

## Examples

```
remotes::install_github("almeidasilvaf/SoyFungiGCN")  # then run_app() to explore the network built from expression_matrix_filtered.tsv
```

## Evaluation signals

- Output matrix dimensions are exactly 18,748 genes × 150 samples
- Every retained gene has a median TPM ≥ 5 when computed across the 150 infection samples
- Sample set matches the 150 soybean–fungal infection conditions spanning all five phytopathogenic fungi (C. gregata, F. graminearum, F. virguliforme, M. phaseolina, P. pachyrhizi)
- Output file is a valid tab-separated file named expression_matrix_filtered.tsv with gene IDs and TPM values parseable in R
- Filtered matrix can be loaded into the SoyFungiGCN R package / Shiny app workflow without schema errors

## Limitations

- Genes with median expression lower than 5 TPM are deliberately excluded to attenuate noise, so lowly expressed but biologically relevant genes are absent from the final matrix
- The dataset depends on availability of the supplementary infection RNA-seq sample data referenced by the paper; specific accessions are not named in the article sections provided
- The matrix is restricted to the 150 soybean–fungal infection samples and the five target fungi; it cannot be reused for other soybean conditions without resampling

## Evidence

- [results] median TPM filter threshold: [evidence span withheld — anchor status: not_found]
- [methods] data retrieval steps: [evidence span withheld — anchor status: not_found]
- [methods] sample subsetting scope: [evidence span withheld — anchor status: not_found]
- [readme] downstream R package availability: "You can install the released version of SoyFungiGCN from GitHub with:"
