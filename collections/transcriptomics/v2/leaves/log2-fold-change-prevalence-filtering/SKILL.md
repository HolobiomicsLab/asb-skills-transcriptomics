---
name: log2-fold-change-prevalence-filtering
description: Use when given a gene cluster absence/presence matrix and per-genome
  lifestyle labels, you must identify lifestyle-enriched gene clusters using prevalence
  and log2 fold-change thresholds plus Fisher's exact test.
when_to_use_negative:
- You only need core-genome identification — bacLIFE uses a separate >90% of genomes
  threshold for core genes, not the pLAG rule
- Your input is already a ranked feature-importance list from the Chi2 machine learning
  step (top 1000 features) rather than a presence/absence matrix
- The dataset lacks lifestyle labels for genomes (bacLIFE lifestyle prediction requires
  a >0.8 random forest confidence value before labels can be used)
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: bacLIFE
  role: Source workflow whose analytical module defines the pLAG detection rule (70%
    prevalence, >2 log2 fold change) applied here
  repo: https://github.com/Carrion-lab/bacLIFE
- name: R fisher.test
  role: Base R function used to compute the association p-value for each gene cluster's
    2x2 contingency table per lifestyle
- name: gtools
  role: Provides foldchange() and foldchange2logratio() to compute the log2 fold change
    between lifestyle-group mean presence and remaining bacteria
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-46302-y
    title: 'bacLIFE: a user-friendly computational workflow for genome analysis and
      prediction of lifestyle-associated genes in bacteria'
  source_content_hash: sha256:002c73ee7d8e0d569db02d1a9c873402bb74d15d1de2fbc178073fe8c20ee582
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Carrion-lab/bacLIFE
  tools:
  - R fisher.test
  - bacLIFE
  - gtools
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# log2-fold-change-prevalence-filtering

## Summary

bacLIFE's pLAG detection rule: classify gene clusters as predicted lifestyle-associated genes (pLAGs) when present in >70% of genomes of a lifestyle group AND showing >2 log2 fold change in presence versus remaining bacteria, ranked by Fisher's exact test p-value with Benjamini-Hochberg correction.

## When to use

Use when, given a gene cluster absence/presence matrix and per-genome lifestyle labels, you must identify lifestyle-enriched gene clusters using prevalence and log2 fold-change thresholds plus Fisher's exact test.

## When NOT to use

- You only need core-genome identification — bacLIFE uses a separate >90% of genomes threshold for core genes, not the pLAG rule
- Your input is already a ranked feature-importance list from the Chi2 machine learning step (top 1000 features) rather than a presence/absence matrix
- The dataset lacks lifestyle labels for genomes (bacLIFE lifestyle prediction requires a >0.8 random forest confidence value before labels can be used)

## Inputs

- Gene cluster absence/presence binary matrix (rows = genomes, columns = gene clusters from linclust/MCL clustering)
- Per-genome lifestyle labels from metadata and bacLIFE lifestyle predictions
- Lifestyle grouping definition (e.g., environmental, opportunistic/animal pathogen, plant pathogen)

## Outputs

- plags_table.csv listing per pLAG: gene cluster ID, lifestyle, group prevalence, log2 fold change, Fisher test p-value, BH-adjusted p-value
- Per-lifestyle pLAG counts (expected e.g. 382 environmental / 478 opportunistic / 786 plant pathogen for Burkholderia/Paraburkholderia; 48/74/377 for Pseudomonas)

## How to apply

Load the gene cluster absence/presence matrix (e.g., from the Burkholderia/Paraburkholderia or Pseudomonas datasets) with per-genome lifestyle labels from metadata and bacLIFE lifestyle predictions. For each lifestyle group, compute group relative presence per gene cluster, then compute the log2 fold change between the group mean presence and the mean presence in remaining bacteria using gtools' foldchange() and foldchange2logratio(). For each gene cluster-lifestyle pair, build a 2x2 contingency table (group vs. remainder, present vs. absent) and run base R fisher.test(), correcting p-values with p.adjust(method="BH"). Call a gene cluster a pLAG when group prevalence exceeds 70% and the fold change exceeds 2 log2; export the ranked table with gene cluster ID, lifestyle, prevalence, log2 fold change, and p-values.

## Related tools

- **bacLIFE** (Source workflow whose analytical module defines the pLAG detection rule (70% prevalence, >2 log2 fold change) applied here) — https://github.com/Carrion-lab/bacLIFE
- **R fisher.test** (Base R function used to compute the association p-value for each gene cluster's 2x2 contingency table per lifestyle)
- **gtools** (Provides foldchange() and foldchange2logratio() to compute the log2 fold change between lifestyle-group mean presence and remaining bacteria)

## Examples

```
l2 <- foldchange2logratio(foldchange(mean_group, mean_rest)); p <- apply(tables, 1, function(t) fisher.test(matrix(t,2))$p.value); padj <- p.adjust(p, method="BH")
```

## Evaluation signals

- Row count check: per-lifestyle pLAG counts must match reported values (382/478/786 environmental/opportunistic/plant pathogen for Burkholderia/Paraburkholderia; 48/74/377 for Pseudomonas)
- Every reported pLAG satisfies both criteria: group prevalence > 70% and log2 fold change > 2
- Adjusted p-values (BH) in plags_table.csv are monotonically consistent with uncorrected Fisher test p-value ranks
- plags_table.csv contains all required columns: gene cluster ID, lifestyle, group prevalence, log2 fold change, p-value, p-adjusted

## Limitations

- The 70% prevalence and >2 log2 fold change thresholds are dataset-specific choices from the paper; the counts (e.g., 786 phytopathogenic pLAGs) will not transfer to other genera without re-tuning
- pLAGs are computational predictions (hypothesis-generation); only 6 of 14 experimentally tested pLAGs of unknown function were confirmed in plant bioassays, so many candidates may be false positives

## Evidence

- [methods] pLAG definition thresholds: "we used the threshold of 70% of the genomes belonging to a lifestyle/group and more than a 2 log2 fold change"
- [intro] reported pLAG counts: "we obtained a total of 786 and 377 predicted lifestyle-associated genes (pLAGs) for a phytopathogenic lifestyle, respectively"
- [abstract] validation of 6/14 candidates: "Site-directed mutagenesis of 14 of these predicted LAGs of unknown function, followed by plant bioassays, showed that 6 predicted LAGs are indeed involved"
- [readme] bacLIFE scope statement: "annotates bacterial genomes and performs large-scale comparative genomics to predict bacterial lifestyles and to pinpoint candidate genes"
