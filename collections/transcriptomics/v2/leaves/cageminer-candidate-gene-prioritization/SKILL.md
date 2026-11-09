---
name: cageminer-candidate-gene-prioritization
description: Use when you need to rank candidate genes underlying GWAS resistance
  loci by integrating gene coexpression modules and defense-related guide genes, e.g.
  genes within 2 Mb of resistance SNPs in soybean.
when_to_use_negative:
- Your organism lacks GWAS SNP results or a derived gene coexpression network — prioritization
  requires both evidence sources.
- You are exploring the published network interactively rather than reproducing candidate-gene
  counts; the SoyFungiGCN Shiny app (run_app()) is the better entry point for exploration.
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0797
tools:
- name: SoyFungiGCN
  role: R package providing the integrated coexpression data (genes_modules) and the
    cageminer-based candidate prioritization for the five pathogens
  repo: https://github.com/almeidasilvaf/SoyFungiGCN
- name: R
  role: Runtime environment in which the SoyFungiGCN package and cageminer prioritization
    are executed
- name: remotes
  role: Used to install SoyFungiGCN from GitHub before running the prioritization
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-021-03864-x
    title: Integration of genome-wide association studies and gene coexpression networks
      unveils promising soybean resistance genes against five common fungal pathogens
  source_content_hash: sha256:af03ea7dbaed4b6df54dd170552fa1b29ddca22f30d0a20fe1d516b3482830e5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/almeidasilvaf/SoyFungiGCN
  tools:
  - R
  - SoyFungiGCN
  - remotes
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# cageminer-candidate-gene-prioritization

## Summary

Prioritize high-confidence candidate resistance genes by combining defense-related guide genes, GWAS SNPs, and RNA-seq-derived coexpression data. Applied here with the SoyFungiGCN R package's cageminer-based workflow to rank genes near resistance SNPs for five soybean fungal pathogens.

## When to use

Use when you need to rank candidate genes underlying GWAS resistance loci by integrating gene coexpression modules and defense-related guide genes, e.g. genes within 2 Mb of resistance SNPs in soybean.

## When NOT to use

- Your organism lacks GWAS SNP results or a derived gene coexpression network — prioritization requires both evidence sources.
- You are exploring the published network interactively rather than reproducing candidate-gene counts; the SoyFungiGCN Shiny app (run_app()) is the better entry point for exploration.

## Inputs

- Publicly available GWAS SNP results for soybean resistance to five fungal pathogens
- RNA-seq-derived gene coexpression network data integrated in SoyFungiGCN (e.g. genes_modules dataset)
- MapMan defense-gene annotations from PLAZA 3.0 Dicots (guide genes)
- Genomic SNP coordinates and soybean gene annotation to define the 2 Mb candidate window

## Outputs

- Ranked list of high-confidence candidate resistance genes per pathogen
- Per-pathogen candidate counts (candidate_counts.csv) totaling 188/56/11/8/3 for F. virguliforme/F. graminearum/C. gregata/M. phaseolina/P. pachyrhizi

## How to apply

Load the pre-integrated GWAS SNP results and RNA-seq-derived coexpression data shipped with the SoyFungiGCN package (e.g. the genes_modules dataset) in R. Assign defense-related guide genes using MapMan annotations from PLAZA 3.0 Dicots so candidates are weighted by defense relevance. Restrict the search space to genes lying within a 2 Mb window around each significant resistance SNP, then run the cageminer prioritization separately for each pathogen with an rpb (point-biserial correlation) threshold of 0.2. Tabulate high-confidence candidate counts per pathogen and compare against the reported 188, 56, 11, 8, and 3 genes for F. virguliforme, F. graminearum, C. gregata, M. phaseolina, and P. pachyrhizi respectively.

## Related tools

- **SoyFungiGCN** (R package providing the integrated coexpression data (genes_modules) and the cageminer-based candidate prioritization for the five pathogens) — https://github.com/almeidasilvaf/SoyFungiGCN
- **R** (Runtime environment in which the SoyFungiGCN package and cageminer prioritization are executed)
- **remotes** (Used to install SoyFungiGCN from GitHub before running the prioritization)

## Examples

```
remotes::install_github("almeidasilvaf/SoyFungiGCN"); library(SoyFungiGCN); data(genes_modules); # then run cageminer per pathogen with rpb threshold 0.2
```

## Evaluation signals

- Reproduced high-confidence candidate counts equal the reported 188, 56, 11, 8, and 3 for F. virguliforme, F. graminearum, C. gregata, M. phaseolina, and P. pachyrhizi, respectively
- Every output candidate lies within the 2 Mb window around at least one significant resistance SNP
- All candidates carry a point-biserial correlation (rpb) score of at least 0.2
- Candidate lists are generated by running cageminer independently per pathogen, so each pathogen yields its own ranked set rather than a merged list
- Guide genes used for weighting correspond to defense-related MapMan annotations from PLAZA 3.0 Dicots

## Limitations

- The SoyFungiGCN package is labeled experimental (lifecycle: experimental) and no changelog or package version is stated in the article
- Specific GWAS and RNA-seq source accessions are not named in the discussion section, so exact input provenance may require consulting the full paper or package data
- Candidate genes are heavily biased towards fungal species-specific defense responses, so counts are not transferable across pathogens or species

## Evidence

- [abstract] Reported candidate counts: "We identified 188, 56, 11, 8, and 3 high-confidence candidates for resistance to F. virguliforme, F. graminearum, C. gregata, M. phaseolina and P. pachyrhizi, respectively"
- [intro] Data integration strategy: [evidence span withheld — anchor status: not_found]
- [methods] Package installation: [evidence span withheld — anchor status: not_found]
- [methods] Package datasets: "- `genes_modules (SoyFungiGCN)`"
- [readme] Package installation (README): "SoyFungiGCN is an R package that contains a Shiny app to let users explore a gene coexpression network of soybean tissues infected with phytopathogenic fungi."
