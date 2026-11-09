---
name: coexpression-evidence-scoring
description: Use when you have GWAS resistance SNPs plus a gene coexpression network
  and need to rank genes near SNPs by defense-relevance to produce high-confidence
  candidate resistance genes.
when_to_use_negative:
- Your organism or pathogen system lacks a precomputed coexpression network like SoyFungiGCN's
  soybean-fungal infection modules, since the skill depends on those packaged data
  objects
- You need de novo SNP discovery or GWAS itself — this skill starts from already-significant
  resistance SNPs and only prioritizes nearby genes
- You want to explore the network interactively via the Shiny app rather than run
  scripted candidate prioritization
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3517
tools:
- name: SoyFungiGCN
  role: R package providing the coexpression network data objects (genes_modules)
    used as input to the cageminer prioritization
  repo: https://github.com/almeidasilvaf/SoyFungiGCN
- name: remotes
  role: Used to install SoyFungiGCN from GitHub before running the prioritization
  repo: https://github.com/almeidasilvaf/SoyFungiGCN
- name: R
  role: Execution environment for the cageminer scoring of candidate genes
  repo: https://github.com/almeidasilvaf/SoyFungiGCN
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-021-03864-x
    title: Integration of genome-wide association studies and gene coexpression networks
      unveils promising soybean resistance genes against five common fungal pathogens
  source_content_hash: sha256:e6ebf7668f97d69345a78e11a1e78a715de747ec2d147f0e4c2a9ac25b37bc9b
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
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# coexpression-evidence-scoring

## Summary

Prioritizes soybean resistance candidate genes by scoring genes near significant resistance SNPs for coexpression-based defense evidence, using defense-related guide genes and a point-biserial correlation (rpb) threshold of 0.2. It integrates GWAS SNP positions with RNA-seq-derived coexpression modules from the SoyFungiGCN package to yield high-confidence candidates per pathogen.

## When to use

Use when you have GWAS resistance SNPs plus a gene coexpression network and need to rank genes near SNPs by defense-relevance to produce high-confidence candidate resistance genes.

## When NOT to use

- Your organism or pathogen system lacks a precomputed coexpression network like SoyFungiGCN's soybean-fungal infection modules, since the skill depends on those packaged data objects
- You need de novo SNP discovery or GWAS itself — this skill starts from already-significant resistance SNPs and only prioritizes nearby genes
- You want to explore the network interactively via the Shiny app rather than run scripted candidate prioritization

## Inputs

- GWAS SNP results for soybean resistance to the five fungal pathogens
- RNA-seq-derived coexpression network data (SoyFungiGCN genes_modules object)
- MapMan defense gene annotations from PLAZA 3.0 Dicots (guide genes)
- R environment with the SoyFungiGCN package installed from GitHub

## Outputs

- Ranked, scored list of high-confidence candidate resistance genes per pathogen
- candidate_counts.csv tabulating candidate gene counts per pathogen (expected: 188, 56, 11, 8, 3)

## How to apply

Load the SoyFungiGCN data objects (e.g., genes_modules) in R after installing the package from GitHub with remotes. Assign defense-related guide genes from MapMan annotations (PLAZA 3.0 Dicots) to weight candidates by defense relevance, and define the search space as genes within a 2 Mb window around each significant resistance SNP. Run the cageminer prioritization separately for each pathogen (F. virguliforme, F. graminearum, C. gregata, M. phaseolina, P. pachyrhizi) with an rpb threshold of 0.2 to score and rank candidates. Rationale: genes near a significant SNP that are coexpressed with defense guide genes are more likely to underlie resistance than SNPs' physical proximity alone. Tabulate high-confidence candidate counts per pathogen and compare against the reported 188/56/11/8/3 counts to confirm reproducibility.

## Related tools

- **SoyFungiGCN** (R package providing the coexpression network data objects (genes_modules) used as input to the cageminer prioritization) — https://github.com/almeidasilvaf/SoyFungiGCN
- **remotes** (Used to install SoyFungiGCN from GitHub before running the prioritization) — https://github.com/almeidasilvaf/SoyFungiGCN
- **R** (Execution environment for the cageminer scoring of candidate genes) — https://github.com/almeidasilvaf/SoyFungiGCN

## Examples

```
remotes::install_github("almeidasilvaf/SoyFungiGCN"); library(SoyFungiGCN); # run cageminer per pathogen with rpb threshold 0.2 on genes within 2 Mb of resistance SNPs
```

## Evaluation signals

- Reproduced high-confidence candidate counts per pathogen must equal 188 (F. virguliforme), 56 (F. graminearum), 11 (C. gregata), 8 (M. phaseolina), and 3 (P. pachyrhizi)
- Every output candidate must lie within the 2 Mb window around a significant resistance SNP
- Candidates must pass the rpb threshold of 0.2 and be weighted by defense-related guide gene association
- Prioritized genes should be highly conserved in the pangenome of cultivated soybeans, as reported in the article
- candidate_counts.csv must contain one count row per pathogen covering all five fungal species

## Limitations

- The SoyFungiGCN package is marked experimental and no changelog or version is stated, so exact reproducibility may depend on the package snapshot
- Supplementary figures and tables referenced by the article are not included in the provided text, limiting verification of intermediate steps
- Counts were validated only for these five soybean pathogens with these specific GWAS and RNA-seq datasets; other inputs are unvalidated

## Evidence

- [abstract] Candidate counts match abstract: "We identified 188, 56, 11, 8, and 3 high-confidence candidates for resistance to F. virguliforme, F. graminearum, C. gregata, M. phaseolina and P. pachyrhizi"
- [intro] Strategy basis: "This strategy consists in investigating the transcriptional patterns of all the genes near a significant SNP"
- [methods] Tool installation for running prioritization: "You can install the released version of SoyFungiGCN from GitHub with"
- [readme] Package purpose from README: "SoyFungiGCN is an R package that contains a Shiny app to let users explore a gene coexpression network of soybean tissues infected with phytopathogenic fungi."
