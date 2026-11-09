---
name: candidate-gene-conservation-assessment
description: Use when you have a binary presence/absence (PAV) matrix over 204 cultivated
  soybean accessions and a list of prioritized candidate resistance genes, and need
  per-gene conservation frequencies and a >99% core-gene fraction.
when_to_use_negative:
- The goal is to discover or prioritize new candidate resistance genes from GWAS and
  RNA-seq data rather than to assess conservation of an already-defined candidate
  list
- The input is coexpression or expression data (e.g., the SoyFungiGCN network) rather
  than a binary PAV matrix over soybean pangenome accessions
- The genotype set is not the 204-accession cultivated soybean pangenome, so the /204
  denominator and the 202-of-204 threshold no longer apply without recalibration
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0625
tools:
- name: R
  role: Language used to load the PAV matrix and candidate list, compute per-gene
    presence frequencies and the >99% conservation fraction, and write pangenome_conservation.csv
- name: SoyFungiGCN
  role: R/Shiny package accompanying the study whose prioritized candidate gene lists
    and pangenome conservation analysis this skill reproduces; installed from GitHub
    with remotes
  repo: https://github.com/almeidasilvaf/SoyFungiGCN
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-021-03864-x
    title: Integration of genome-wide association studies and gene coexpression networks
      unveils promising soybean resistance genes against five common fungal pathogens
  source_content_hash: sha256:19a1ccaf379e605a194f507c140a8c7e317158630f3b6208c04448290a58f3ff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/almeidasilvaf/SoyFungiGCN
  tools:
  - R
  - SoyFungiGCN
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# candidate-gene-conservation-assessment

## Summary

Quantifies how conserved prioritized soybean candidate resistance genes are across the cultivated soybean pangenome by computing per-gene presence frequencies from a binary PAV matrix of 204 accessions and flagging genes present in >99% of accessions as highly conserved core genes.

## When to use

Use when you have a binary presence/absence (PAV) matrix over 204 cultivated soybean accessions and a list of prioritized candidate resistance genes, and need per-gene conservation frequencies and a >99% core-gene fraction.

## When NOT to use

- The goal is to discover or prioritize new candidate resistance genes from GWAS and RNA-seq data rather than to assess conservation of an already-defined candidate list
- The input is coexpression or expression data (e.g., the SoyFungiGCN network) rather than a binary PAV matrix over soybean pangenome accessions
- The genotype set is not the 204-accession cultivated soybean pangenome, so the /204 denominator and the 202-of-204 threshold no longer apply without recalibration

## Inputs

- Presence/absence variation (PAV) matrix: genes as rows, 204 cultivated soybean accessions as columns, binary presence/absence calls
- Prioritized candidate gene list (266 genes total: 188 F. virguliforme, 56 F. graminearum, 11 C. gregata, 8 M. phaseolina, 3 P. pachyrhizi)

## Outputs

- pangenome_conservation.csv with per-gene presence frequency (count/204), per-gene conserved status, and the overall summary fraction of conserved candidates

## How to apply

Load the PAV matrix with one row per gene and one binary column per accession for the 204 cultivated soybean genomes, then load the prioritized candidate gene list (188 F. virguliforme, 56 F. graminearum, 11 C. gregata, 8 M. phaseolina, 3 P. pachyrhizi candidates). For each candidate gene, compute presence frequency as the count of accessions in which the gene is present divided by 204. Apply the >99% conservation threshold, i.e., present in at least 202 of 204 accessions, to assign a binary conserved status per gene. Compute the overall fraction of candidate genes meeting the threshold as the summary statistic. Write per-gene presence frequency, conserved status, and the summary fraction to pangenome_conservation.csv; this step reproduces the paper's rationale that conservation in the pangenome supports the biological plausibility of resistance candidates.

## Related tools

- **R** (Language used to load the PAV matrix and candidate list, compute per-gene presence frequencies and the >99% conservation fraction, and write pangenome_conservation.csv)
- **SoyFungiGCN** (R/Shiny package accompanying the study whose prioritized candidate gene lists and pangenome conservation analysis this skill reproduces; installed from GitHub with remotes) — https://github.com/almeidasilvaf/SoyFungiGCN

## Examples

```
Rscript -e 'pav <- read.csv("pav_matrix.csv"); freq <- rowSums(pav[, -1] == 1) / 204; conserved <- freq >= 202/204; write.csv(data.frame(gene = pav$gene, freq, conserved), "pangenome_conservation.csv")'
```

## Evaluation signals

- Every gene in the input candidate list has a computed presence frequency in the output table (266 genes: 188 + 56 + 11 + 8 + 3)
- Conserved status is assigned with the exact threshold of presence in at least 202 of 204 accessions (>99%)
- The summary fraction of conserved candidate genes is consistent with the paper's report that prioritized candidates are highly conserved in the cultivated soybean pangenome
- Output file pangenome_conservation.csv contains the three required columns: per-gene presence frequency, conserved status, and summary fraction
- Per-gene presence frequencies all fall in [0, 1] with denominator exactly 204

## Limitations

- Conservation is only assessed against the 204 cultivated soybean accessions in the pangenome; wild accessions or other panels are not covered
- The analysis depends on binary PAV calls and does not distinguish copy-number differences or partial gene models among accessions
- No version is stated for the SoyFungiGCN R package, and supplementary data referenced in the paper are needed to obtain the PAV matrix and candidate lists

## Evidence

- [abstract] The prioritized candidate genes are highly conserved in the pangenome of cultivated soybeans: "The prioritized candidate genes are highly conserved in the pangenome of cultivated soybeans"
- [intro] The prioritized resistance genes are species-specific and highly conserved in the pangenome of cultivated soybeans: "The prioritized resistance genes are species-specific and highly conserved in the pangenome of cultivated soybeans"
- [methods] remotes::install_github("almeidasilvaf/SoyFungiGCN"): "remotes::install_github("almeidasilvaf/SoyFungiGCN")"
- [readme] SoyFungiGCN is an R package that contains a Shiny app to let users explore a gene coexpression network of soybean tissues infected with phytopathogenic fungi.: "SoyFungiGCN is an R package that contains a Shiny app to let users explore a gene coexpression network of soybean tissues infected with phytopathogenic fungi."
