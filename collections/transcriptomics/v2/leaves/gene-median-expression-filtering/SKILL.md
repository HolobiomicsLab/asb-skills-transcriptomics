---
name: gene-median-expression-filtering
description: Use when a raw TPM expression matrix (e.g. from the Soybean Expression
  Atlas plus supplementary infection RNA-seq data) must be filtered to remove lowly
  expressed genes before building a coexpression network.
when_to_use_negative:
- The input is already a filtered matrix or contains only a small number of genes
  where the median-TPM cutoff would discard biologically relevant low-abundance transcripts
- Expression values are not TPM-normalized (e.g. raw counts or FPKM), since the 5
  TPM threshold is only meaningful for TPM-scaled estimates
- The analysis targets single-sample or per-accession differential expression rather
  than building a multi-sample coexpression network
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3523
- http://edamontology.org/topic_0203
tools:
- name: R
  role: Used to compute per-gene median TPM across the 150 infection samples and apply
    the ≥5 TPM filtering threshold
  repo: https://github.com/almeidasilvaf/SoyFungiGCN
- name: SoyFungiGCN
  role: R/Shiny package that ships the resulting filtered expression data as part
    of the soybean–fungal coexpression network resource
  repo: https://github.com/almeidasilvaf/SoyFungiGCN
- name: utils
  role: Used for basic R file reading/writing of the TSV expression matrices
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-021-03864-x
    title: Integration of genome-wide association studies and gene coexpression networks
      unveils promising soybean resistance genes against five common fungal pathogens
  source_content_hash: sha256:e180ff791653a4ebd69145a54dd14d16407735434277fc7d6948d5637d5e92f4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/almeidasilvaf/SoyFungiGCN
  tools:
  - R
  - SoyFungiGCN
  - utils
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3523
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# gene-median-expression-filtering

## Summary

A noise-attenuation step for bulk RNA-seq expression matrices in which genes whose median expression across all samples falls below a fixed TPM cutoff (here 5 TPM) are discarded, yielding a compact matrix for downstream coexpression network construction.

## When to use

Use when a raw TPM expression matrix (e.g. from the Soybean Expression Atlas plus supplementary infection RNA-seq data) must be filtered to remove lowly expressed genes before building a coexpression network.

## When NOT to use

- The input is already a filtered matrix or contains only a small number of genes where the median-TPM cutoff would discard biologically relevant low-abundance transcripts
- Expression values are not TPM-normalized (e.g. raw counts or FPKM), since the 5 TPM threshold is only meaningful for TPM-scaled estimates
- The analysis targets single-sample or per-accession differential expression rather than building a multi-sample coexpression network

## Inputs

- TPM expression estimates from the Soybean Expression Atlas
- Supplementary infection RNA-seq sample data referenced by the paper
- Sample metadata identifying the 150 soybean–fungal infection samples spanning five phytopathogenic fungi

## Outputs

- expression_matrix_filtered.tsv — an 18,748 genes × 150 samples filtered TPM expression matrix where every retained gene has median TPM ≥ 5

## How to apply

Download TPM expression estimates (from the Soybean Expression Atlas plus the supplementary infection RNA-seq samples) and subset them to the 150 samples corresponding to soybean–fungal infection conditions covering the five phytopathogenic fungi (C. gregata, F. graminearum, F. virguliforme, M. phaseolina, P. pachyrhizi). Compute the per-gene median TPM across all 150 samples in R (e.g. using matrixStats::rowMedians or apply(..., 1, median)). Discard every gene whose median TPM is below the threshold of 5, which the authors applied 'to attenuate noise'. Assemble the retained genes into the final matrix and write it as expression_matrix_filtered.tsv, verifying it has exactly 18,748 genes × 150 samples.

## Related tools

- **R** (Used to compute per-gene median TPM across the 150 infection samples and apply the ≥5 TPM filtering threshold) — https://github.com/almeidasilvaf/SoyFungiGCN
- **SoyFungiGCN** (R/Shiny package that ships the resulting filtered expression data as part of the soybean–fungal coexpression network resource) — https://github.com/almeidasilvaf/SoyFungiGCN
- **utils** (Used for basic R file reading/writing of the TSV expression matrices)

## Examples

```
Rscript -e 'expr <- read.delim("expression_matrix.tsv", row.names=1); med <- apply(expr, 1, median); filt <- expr[med >= 5, ]; write.table(filt, "expression_matrix_filtered.tsv", sep="\t", quote=FALSE)'
```

## Evaluation signals

- The output expression_matrix_filtered.tsv has exactly 18,748 rows (genes) and 150 columns (samples).
- Every retained gene has a median TPM ≥ 5 across the 150 infection samples.
- Row names correspond to soybean gene identifiers consistent with the Soybean Expression Atlas TPM table.
- Column names span exactly the 150 samples covering the five phytopathogenic fungi (C. gregata, F. graminearum, F. virguliforme, M. phaseolina, P. pachyrhizi).
- The file is a valid tab-separated values table readable with read.delim() in R without header or delimiter errors.

## Limitations

- The 5 TPM median cutoff was tuned for this specific soybean-fungal dataset; applying it unchanged to other organisms or tissue panels may be too lenient or too stringent.
- Filtering based on median TPM across all 150 samples will remove genes that are lowly expressed on average but strongly infection- or timepoint-specific, potentially discarding some conditionally expressed defense genes.
- The paper notes no changelog or version for the R package, so the exact TPM table version from the Soybean Expression Atlas should be recorded separately for reproducibility.

## Evidence

- [results] After excluding genes with median expression values lower than 5 to attenuate noise, the final filtered dataset comprised an 18,748 x 150 gene expression matrix: [evidence span withheld — anchor status: not_found]
- [readme] SoyFungiGCN is an R package that contains a Shiny app to let users explore a gene coexpression network of soybean tissues infected with phytopathogenic fungi.: "SoyFungiGCN is an R package that contains a Shiny app to let users explore a gene coexpression network of soybean tissues infected with phytopathogenic fungi."
- [methods] You can install the released version of SoyFungiGCN from GitHub with: "You can install the released version of SoyFungiGCN from GitHub with"
- [intro] we integrated multiple publicly available RNA-seq and GWAS datasets to identify high-confidence candidate genes for resistance to five phytopathogenic fungi in soybean: [evidence span withheld — anchor status: not_found]
