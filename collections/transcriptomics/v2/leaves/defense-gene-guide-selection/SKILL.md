---
name: defense-gene-guide-selection
description: Use when prioritizing candidate resistance genes around significant SNPs
  and you need to weight or rank those candidates by defense relevance using annotated
  defense-related guide genes.
when_to_use_negative:
- Your goal is de novo functional annotation of genes rather than prioritizing SNP-proximal
  candidates for a specific pathogen.
- Your organism or pathogen system lacks defense-related MapMan/PLAZA annotations
  or a GWAS SNP set, since the guide weighting and SNP-window steps cannot be grounded.
- You need a web-based interactive exploration of the coexpression network only (use
  the SoyFungiGCN Shiny app directly instead of rerunning the prioritization).
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: SoyFungiGCN
  role: R package supplying the coexpression network data (genes_modules) and context
    in which guide genes and candidate prioritization are applied
  repo: https://github.com/almeidasilvaf/SoyFungiGCN
- name: R
  role: Language in which the guide-gene assignment and cageminer prioritization are
    executed
  repo: https://github.com/almeidasilvaf/SoyFungiGCN
- name: remotes
  role: Used to install SoyFungiGCN from GitHub before running the prioritization
  repo: https://github.com/almeidasilvaf/SoyFungiGCN
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-021-03864-x
    title: Integration of genome-wide association studies and gene coexpression networks
      unveils promising soybean resistance genes against five common fungal pathogens
  source_content_hash: sha256:a6cfb733de4d08227b7830e1f324b97ff9568031027e80bcce368a7acdc42d45
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/almeidasilvaf/SoyFungiGCN
  tools:
  - R
  - SoyFungiGCN
  - remotes
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# Defense gene guide selection

## Summary

Selects defense-related guide genes (via MapMan annotations from PLAZA 3.0 Dicots) to weight candidate soybean resistance genes during prioritization of genes near resistance SNPs, so that candidates are ranked by defense relevance rather than genomic proximity alone.

## When to use

Use when prioritizing candidate resistance genes around significant SNPs and you need to weight or rank those candidates by defense relevance using annotated defense-related guide genes.

## When NOT to use

- Your goal is de novo functional annotation of genes rather than prioritizing SNP-proximal candidates for a specific pathogen.
- Your organism or pathogen system lacks defense-related MapMan/PLAZA annotations or a GWAS SNP set, since the guide weighting and SNP-window steps cannot be grounded.
- You need a web-based interactive exploration of the coexpression network only (use the SoyFungiGCN Shiny app directly instead of rerunning the prioritization).

## Inputs

- MapMan functional annotations for soybean genes from PLAZA 3.0 Dicots
- GWAS significant resistance SNP positions per fungal pathogen
- SoyFungiGCN coexpression data (e.g. genes_modules object)
- 2 Mb genomic windows around each resistance SNP defining the candidate search space

## Outputs

- Ranked, high-confidence candidate resistance gene lists per pathogen (with defense-relevance weighting from guide genes)
- Per-pathogen candidate gene counts (expected: 188, 56, 11, 8, 3)

## How to apply

Build the guide-gene set by assigning soybean genes to defense-related MapMan functional classes obtained from PLAZA 3.0 Dicots, so the guides represent defense-related genes rather than all expressed genes. Supply this guide set together with the SoyFungiGCN coexpression data (e.g. the genes_modules object) and a candidate search space defined as all genes within a 2 Mb window of each significant resistance SNP. Run the cageminer-style prioritization separately for each of the five phytopathogenic fungi (C. gregata, F. graminearum, F. virguliforme, M. phaseolina, P. pachyrhizi) using a point-biserial correlation (rpb) threshold of 0.2 to score and rank candidates. The rationale is that genes near a resistance SNP are only high-confidence candidates if they show transcriptional patterns associated with defense, which the guide genes encode. Tabulate the per-pathogen counts of high-confidence candidates and compare them against the reported values before any downstream analysis.

## Related tools

- **SoyFungiGCN** (R package supplying the coexpression network data (genes_modules) and context in which guide genes and candidate prioritization are applied) — https://github.com/almeidasilvaf/SoyFungiGCN
- **R** (Language in which the guide-gene assignment and cageminer prioritization are executed) — https://github.com/almeidasilvaf/SoyFungiGCN
- **remotes** (Used to install SoyFungiGCN from GitHub before running the prioritization) — https://github.com/almeidasilvaf/SoyFungiGCN

## Examples

```
remotes::install_github("almeidasilvaf/SoyFungiGCN"); library(SoyFungiGCN); data(genes_modules); # assign MapMan defense guides, define 2 Mb SNP windows, prioritize with rpb >= 0.2
```

## Evaluation signals

- Reproduced per-pathogen high-confidence candidate counts equal the reported 188, 56, 11, 8, and 3 for F. virguliforme, F. graminearum, C. gregata, M. phaseolina, and P. pachyrhizi, respectively.
- The guide-gene set contains only genes assigned to defense-related MapMan classes from PLAZA 3.0 Dicots (no genome-wide gene set).
- Every prioritized candidate lies within the 2 Mb window of a significant resistance SNP for the corresponding pathogen.
- The rpb threshold used in the prioritization is 0.2, matching the reported setting.
- Prioritized genes are evaluated against the SoyFungiGCN coexpression modules (genes_modules) rather than arbitrary expression data.

## Limitations

- Results depend on the specific publicly available GWAS and RNA-seq datasets integrated in SoyFungiGCN; the exact data source accessions are not fully enumerated in the main text and supplementary tables are not directly accessible.
- The SoyFungiGCN package is marked as experimental (early lifecycle stage) and has no changelog or stated version, so exact reproducibility of counts may be sensitive to package updates.
- Counts are specific to the 2 Mb window and rpb 0.2 settings; changing either threshold will alter the per-pathogen candidate numbers.

## Evidence

- [abstract] Reported candidate counts: "We identified 188, 56, 11, 8, and 3 high-confidence candidates for resistance to F. virguliforme, F. graminearum, C. gregata, M. phaseolina and P. pachyrhizi, respectively"
- [intro] Integrated GWAS/RNA-seq objective: "we integrated multiple publicly available RNA-seq and GWAS datasets to identify high-confidence candidate genes for resistance to five phytopathogenic fungi"
- [readme] Package purpose: "SoyFungiGCN is an R package that contains a Shiny app to let users explore a gene coexpression network of soybean tissues infected with phytopathogenic fungi."
- [readme] Package installation: "You can install the released version of SoyFungiGCN from GitHub with:"
