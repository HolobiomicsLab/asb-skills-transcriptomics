---
name: single-cell-differential-expression-analysis
description: Use when scRNA-seq cells have been partitioned by a binary event (e.g.,
  presence/absence of a fusion such as TMPRSS2-ERG) and you must test association
  with treatment or differential gene expression to quantify the fusion's transcriptional
  impact per cell.
when_to_use_negative:
- Data are bulk RNA-seq or droplet 10X data — scFusion and its per-cell grouping are
  built for Smart-Seq single-cell FASTQ (*_1.fastq, *_2.fastq), not 10X or *_1.fq
  files
- No per-cell binary grouping exists (no fusion calls or treatment labels), so a 2x2
  contingency/Fisher test framework cannot be defined
- Fusion is rare and the tumor is highly heterogeneous — detection power is limited
  and group sizes are too small for reliable testing
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scFusion
  role: Detects per-cell gene fusions (e.g., TMPRSS2-ERG in 27 cells) that define
    the fusion-positive/fusion-negative cell groups being compared
  repo: ZijieJin/scFusion
- name: R
  role: Statistical environment (>= 3.5) for building the 2x2 contingency table, running
    Fisher's exact test, and ERG differential expression testing
- name: stringr
  role: R package used in the scFusion workflow for string manipulation of fusion
    labels and cell annotations during the association/DE analysis
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-28661-6
    title: Single-cell gene fusion detection by scFusion
  source_content_hash: sha256:6b17383b71119a5f3371888acfa068300aedded5947b564ce843d0e1e740472e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZijieJin/scFusion
  tools:
  - Python
  - R
  - bedtools
  - pyensembl
  - samtools
  - scFusion
  - stringr
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 21
---

# single-cell-differential-expression-analysis

## Summary

Statistical comparison of gene expression between subpopulations of single cells — here, ERG expression in TMPRSS2-ERG fusion-positive vs fusion-negative prostate epithelial cells from a 922-cell scRNA-seq patient dataset — to quantify the transcriptional impact of a fusion at single-cell resolution.

## When to use

Use when scRNA-seq cells have been partitioned by a binary event (e.g., presence/absence of a fusion such as TMPRSS2-ERG) and you must test association with treatment or differential gene expression to quantify the fusion's transcriptional impact per cell.

## When NOT to use

- Data are bulk RNA-seq or droplet 10X data — scFusion and its per-cell grouping are built for Smart-Seq single-cell FASTQ (*_1.fastq, *_2.fastq), not 10X or *_1.fq files
- No per-cell binary grouping exists (no fusion calls or treatment labels), so a 2x2 contingency/Fisher test framework cannot be defined
- Fusion is rare and the tumor is highly heterogeneous — detection power is limited and group sizes are too small for reliable testing

## Inputs

- Per-cell scFusion fusion calls from Smart-Seq scRNA-seq data (e.g., TMPRSS2-ERG positivity per cell)
- Cell-level metadata table (treatment status: enzalutamide vs untreated; n=257 vs n=665)
- Gene expression matrix from the 922 prostate epithelial single cells (hg19-aligned Smart-Seq data)
- GTF annotation (Ensembl/NCBI/UCSC) for gene identity mapping

## Outputs

- 2x2 contingency table of fusion positivity vs treatment (16.7% vs 3.7%)
- Fisher's exact test p-value (expected p = 0.0047, Table 2)
- Differential expression result for ERG between fusion-positive and fusion-negative cells
- scfusion_results.csv consolidating fusion calls and downstream test outputs

## How to apply

After scFusion assigns per-cell fusion calls (e.g., TMPRSS2-ERG detected in 27 of 922 prostate epithelial cells), build a 2x2 contingency table crossing fusion positivity with a cell-level covariate such as enzalutamide treatment (fusion in 16.7% of 257 treated vs 3.7% of 665 untreated cells). Run Fisher's exact test in R (>= 3.5, using the stringr package for string handling of fusion labels) and expect p = 0.0047 as reported in Table 2. Then perform differential expression testing of the 5' fusion partner gene (ERG) between fusion-positive and fusion-negative cells to link the fusion to its oncogenic transcriptional output. The rationale is that jointly analyzing all single cells exploits the fact that multiple cells from a sample should contain the same fusion, making per-cell association and expression tests well-powered.

## Related tools

- **scFusion** (Detects per-cell gene fusions (e.g., TMPRSS2-ERG in 27 cells) that define the fusion-positive/fusion-negative cell groups being compared) — ZijieJin/scFusion
- **R** (Statistical environment (>= 3.5) for building the 2x2 contingency table, running Fisher's exact test, and ERG differential expression testing)
- **stringr** (R package used in the scFusion workflow for string manipulation of fusion labels and cell annotations during the association/DE analysis)

## Examples

```
Rscript -e 'm <- matrix(c(43, 214, 25, 640), nrow=2); fisher.test(m)'
```

## Evaluation signals

- Reproduced 2x2 contingency percentages match Table 2: 16.7% fusion positivity among 257 enzalutamide-treated cells vs 3.7% among 665 untreated cells
- Fisher's exact test returns p = 0.0047
- Underlying fusion counts match the scFusion calls: TMPRSS2-ERG in 27 cells and 10 total fusions in the patient data (4 in LNCaP, all in CCLE)
- ERG differential expression is computed between exactly the 27 fusion-positive and remaining fusion-negative epithelial cells
- Analysis runs under R >= 3.5 with the stringr package available

## Limitations

- scFusion has reduced sensitivity for fusions with low expression, as is the case for all RNA-based methods, which can distort the fusion-positive group
- Detection power is limited for rare fusions in highly heterogeneous tumor samples, shrinking group sizes for testing
- Choosing appropriate thresholds to define distinct cell populations based on marker gene expression or transcriptional profile similarity is difficult

## Evidence

- [abstract] transcriptional impact: "Our results demonstrate that scFusion can be used to investigate cellular heterogeneity of gene fusions and their transcriptional impact at the single-cell level."
- [intro] per-cell grouping rationale: "Such analysis could help to identify cell subtypes or subclones in which a fusion plays a role"
- [readme] input format: "Single cell RNA sequencing files from Smart-Seq protocol. File names should be *_1.fastq, *_2.fastq"
