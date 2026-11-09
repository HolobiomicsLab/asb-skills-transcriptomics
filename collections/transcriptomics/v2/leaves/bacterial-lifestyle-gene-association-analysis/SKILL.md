---
name: bacterial-lifestyle-gene-association-analysis
description: Use when you have a binary gene cluster absence/presence matrix plus
  per-genome lifestyle labels and need to enumerate lifestyle-associated genes per
  group using bacLIFE's 70% prevalence / 2 log2FC rule.
when_to_use_negative:
- You only need the lifestyle prediction itself (bacLIFE machine-learning module),
  not the gene-level association analysis that follows it
- You lack the binary gene cluster absence/presence matrix and only have raw genome
  assemblies — the clustering module must run first
- You want core-genome identification (genes in >90% of genomes), which is a different
  filter than the pLAG 70% group-prevalence rule
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: bacLIFE
  role: Analytical module defines the pLAG detection rule (Fisher's exact test, >70%
    group prevalence, >2 log2 fold change) being reproduced
  repo: https://github.com/Carrion-lab/bacLIFE
- name: R fisher.test
  role: Base R function used for the per-cluster 2x2 contingency table enrichment
    test, with Benjamini-Hochberg correction via p.adjust()
- name: gtools
  role: Provides foldchange() and foldchange2logratio() to compute log2 fold change
    between lifestyle group and remaining bacteria
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-46302-y
    title: 'bacLIFE: a user-friendly computational workflow for genome analysis and
      prediction of lifestyle-associated genes in bacteria'
  source_content_hash: sha256:cfbd7beff98ecb165c6b3e5d2bbbe0a243c2c5d5c1a591e5747e9defca4531d0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Carrion-lab/bacLIFE
  tools:
  - R fisher.test
  - bacLIFE
  - gtools
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# bacterial-lifestyle-gene-association-analysis

## Summary

Reproduce bacLIFE's analytical-module detection of predicted lifestyle-associated genes (pLAGs) by running per-lifestyle Fisher's exact tests on a gene cluster absence/presence matrix and filtering with >70% group prevalence and >2 log2 fold change.

## When to use

Use when you have a binary gene cluster absence/presence matrix plus per-genome lifestyle labels and need to enumerate lifestyle-associated genes per group using bacLIFE's 70% prevalence / 2 log2FC rule.

## When NOT to use

- You only need the lifestyle prediction itself (bacLIFE machine-learning module), not the gene-level association analysis that follows it
- You lack the binary gene cluster absence/presence matrix and only have raw genome assemblies — the clustering module must run first
- You want core-genome identification (genes in >90% of genomes), which is a different filter than the pLAG 70% group-prevalence rule

## Inputs

- Gene cluster binary absence/presence matrix (rows = genomes, columns = gene clusters)
- Per-genome lifestyle labels from metadata and bacLIFE augmented lifestyle predictions
- Lifestyle group assignments (e.g., environmental, opportunistic/animal pathogen, plant pathogen)

## Outputs

- plags_table.csv with columns: gene cluster ID, lifestyle, group prevalence, log2 fold change, p-value, p-adjusted
- Per-lifestyle pLAG counts (expected: 382/478/786 for Burkholderia/Paraburkholderia environmental/opportunistic/plant pathogen; 48/74/377 for Pseudomonas)

## How to apply

In R, load the gene cluster absence/presence matrix (Burkholderia/Paraburkholderia and/or Pseudomonas datasets) together with lifestyle labels from metadata and augmented lifestyle predictions. For each lifestyle group, compute group relative presence per gene cluster and the log2 fold change between group mean presence and mean presence in the remaining bacteria using gtools' foldchange() and foldchange2logratio(). For each gene cluster × lifestyle, build a 2x2 contingency table and run base R fisher.test(), correcting p-values with p.adjust() (Benjamini-Hochberg) for ranking. Define pLAGs as clusters with >70% presence within the lifestyle group and >2 log2 fold change versus the remaining bacteria, then export the table (cluster ID, lifestyle, group prevalence, log2FC, p-value, p-adjusted) as plags_table.csv.

## Related tools

- **bacLIFE** (Analytical module defines the pLAG detection rule (Fisher's exact test, >70% group prevalence, >2 log2 fold change) being reproduced) — https://github.com/Carrion-lab/bacLIFE
- **R fisher.test** (Base R function used for the per-cluster 2x2 contingency table enrichment test, with Benjamini-Hochberg correction via p.adjust())
- **gtools** (Provides foldchange() and foldchange2logratio() to compute log2 fold change between lifestyle group and remaining bacteria)

## Examples

```
Rscript -e 'fit <- fisher.test(table(cluster_present, in_group)); padj <- p.adjust(pvals, method="BH")'
```

## Evaluation signals

- row_count_equals check: per-lifestyle pLAG counts equal 382 (environmental), 478 (opportunistic/animal pathogen), and 786 (plant pathogen) for Burkholderia/Paraburkholderia
- row_count_equals check: Pseudomonas pLAG counts equal 48, 74, and 377 per lifestyle
- Every pLAG row satisfies both filters: group prevalence >70% and log2 fold change >2
- plags_table.csv has the expected schema: gene cluster ID, lifestyle, group prevalence, log2 fold change, p-value, p-adjusted (BH-corrected)
- pLAGs are ranked by Fisher test p-value after Benjamini-Hochberg adjustment

## Limitations

- pLAGs are predictions requiring experimental validation: only 6 of 14 mutagenized candidate LAGs of unknown function were confirmed in plant bioassays
- Lifestyle labels depend on bacLIFE's lifestyle prediction confidence (a lifestyle is assigned only when random forest adjudication confidence >0.8), so noisy labels propagate into pLAG calls
- The threshold rule (70% prevalence, >2 log2FC) is specific to bacLIFE's analytical module and may not generalize to other comparative genomics pipelines

## Evidence

- [methods] pLAG definition thresholds: "we used the threshold of 70% of the genomes belonging to a lifestyle/group and more than a 2 log2 fold change"
- [intro] pLAG counts reported: "For Burkholderia/Paraburkholderia and Pseudomonas, we obtained a total of 786 and 377 predicted lifestyle-associated genes (pLAGs) for a phytopathogenic lifestyle"
- [intro] dataset scope: "To test the potential of this bioinformatic tool, designated bacLIFE, we extracted 16,846 genomes of species and strains belonging to the Burkholderiaceae family and Pseudomonas genus"
- [readme] tool purpose: "annotates bacterial genomes and performs large-scale comparative genomics to predict bacterial lifestyles and to pinpoint candidate genes, denominated  **lifestyle-associated genes (LAGs)**"
- [abstract] validation of predictions: "Site-directed mutagenesis of 14 of these predicted LAGs of unknown function, followed by plant bioassays, showed that 6 predicted LAGs are indeed involved in the phytopathogenic lifestyle of"
