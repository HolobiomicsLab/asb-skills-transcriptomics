---
name: fisher-exact-test-enrichment-analysis
description: Use when you have a binary gene-cluster absence/presence matrix plus
  per-genome lifestyle labels and need to find genes significantly enriched in one
  lifestyle group versus the rest.
when_to_use_negative:
- Your input is not a binary absence/presence gene-cluster matrix (e.g., raw reads
  or expression counts) — the enrichment test requires the bacLIFE clustering module's
  binary matrix
- Genome lifestyle labels are unavailable or lack the >0.8 random-forest confidence
  threshold, so group membership cannot be trusted for the 2x2 contingency tables
- You need genomic regions enriched in virulence factors from pLAG positions rather
  than per-gene enrichment statistics — that is a separate downstream analysis
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: bacLIFE
  role: Defines the pLAG detection rule and analytical module whose thresholds (>70%
    group prevalence, >2 log2 fold change) this Fisher's exact test enrichment implements
  repo: https://github.com/Carrion-lab/bacLIFE
- name: R fisher.test
  role: Performs the 2x2 contingency-table Fisher's exact test per gene cluster and
    lifestyle, with p.adjust() for Benjamini-Hochberg correction
- name: gtools
  role: Provides foldchange() and foldchange2logratio() to compute the >2 log2 fold
    change in presence between lifestyle group and remaining bacteria
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-46302-y
    title: 'bacLIFE: a user-friendly computational workflow for genome analysis and
      prediction of lifestyle-associated genes in bacteria'
  source_content_hash: sha256:d711024a318ad563b0256f6c2ff332a7e6772513ace57ff83328c4ce512e29d8
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
n_claims: 6
---

# fisher-exact-test-enrichment-analysis

## Summary

A per-lifestyle enrichment test that identifies predicted lifestyle-associated genes (pLAGs) by testing each gene cluster's absence/presence against lifestyle group membership with Fisher's exact test, applying bacLIFE's thresholds of >70% group prevalence and >2 log2 fold change, as used on Burkholderia/Paraburkholderia and Pseudomonas datasets.

## When to use

Use when you have a binary gene-cluster absence/presence matrix plus per-genome lifestyle labels and need to find genes significantly enriched in one lifestyle group versus the rest.

## When NOT to use

- Your input is not a binary absence/presence gene-cluster matrix (e.g., raw reads or expression counts) — the enrichment test requires the bacLIFE clustering module's binary matrix
- Genome lifestyle labels are unavailable or lack the >0.8 random-forest confidence threshold, so group membership cannot be trusted for the 2x2 contingency tables
- You need genomic regions enriched in virulence factors from pLAG positions rather than per-gene enrichment statistics — that is a separate downstream analysis

## Inputs

- Gene cluster absence/presence binary matrix (genomes x gene clusters) for the Burkholderia/Paraburkholderia or Pseudomonas dataset
- Per-genome lifestyle labels from metadata and bacLIFE's augmented lifestyle predictions (confidence >0.8)
- Lifestyle group assignments (e.g., environmental, opportunistic/animal pathogen, plant pathogen)

## Outputs

- plags_table.csv with gene cluster ID, lifestyle, group prevalence, log2 fold change, Fisher p-value, and Benjamini-Hochberg adjusted p-value
- Per-lifestyle pLAG counts (e.g., 382/478/786 for Burkholderia/Paraburkholderia environmental/opportunistic/plant pathogen; 48/74/377 for Pseudomonas)

## How to apply

Load the gene cluster absence/presence matrix (one row per genome, one column per gene cluster) together with per-genome lifestyle labels from metadata and augmented lifestyle predictions. For each lifestyle group, compute group relative presence per gene cluster and the log2 fold change between group mean presence and mean presence in the remaining bacteria using the foldchange() and foldchange2logratio() functions of the gtools package in R. For each gene cluster and lifestyle, build a 2x2 contingency table and run base R fisher.test(), then rank with p.adjust() using the Benjamini-Hochberg method. Define pLAGs as clusters with >70% presence within the lifestyle group and more than a 2 log2 fold change between group and remaining bacteria, as specified in bacLIFE's analytical module. Export a per-lifestyle pLAG table (gene cluster ID, lifestyle, group prevalence, log2 fold change, p-value, p-adjusted) and validate counts against published values (382/478/786 for environmental/opportunistic/plant pathogen in Burkholderia/Paraburkholderia; 48/74/377 for Pseudomonas).

## Related tools

- **bacLIFE** (Defines the pLAG detection rule and analytical module whose thresholds (>70% group prevalence, >2 log2 fold change) this Fisher's exact test enrichment implements) — https://github.com/Carrion-lab/bacLIFE
- **R fisher.test** (Performs the 2x2 contingency-table Fisher's exact test per gene cluster and lifestyle, with p.adjust() for Benjamini-Hochberg correction)
- **gtools** (Provides foldchange() and foldchange2logratio() to compute the >2 log2 fold change in presence between lifestyle group and remaining bacteria)

## Examples

```
fisher.test(table(cluster_present_in_group, group_membership)); p <- p.adjust(pvals, method='BH'); plags <- subset(res, prevalence > 0.7 & log2fc > 2)
```

## Evaluation signals

- Row-count check on plags_table.csv: per-lifestyle pLAG counts must equal 382/478/786 (environmental/opportunistic/plant pathogen) for Burkholderia/Paraburkholderia and 48/74/377 for Pseudomonas
- Every reported pLAG satisfies both criteria: group prevalence >70% and log2 fold change >2 between group and remaining bacteria
- Adjusted p-values are present and monotonic with Benjamini-Hochberg ranking over the raw Fisher p-values
- Each gene cluster's 2x2 contingency table margins sum to the number of genomes in the lifestyle group plus the remainder
- Group prevalence per gene cluster is reproducible from the absence/presence matrix and lifestyle label assignments

## Limitations

- Results depend on the quality of upstream lifestyle predictions: bacLIFE only selects a lifestyle when the random forest adjudication confidence is >0.8, so mislabeled or low-confidence genomes distort group prevalence
- Only high-quality genomes (>90% completeness, <5% contamination per CheckM) were retained for the reference results; applying the test to lower-quality assemblies may shift pLAG counts
- The 70% prevalence and 2 log2 fold change thresholds are fixed heuristic cutoffs; datasets with different lifestyle group sizes or gene cluster granularity (linclust/MCL parameters) may not reproduce the published counts

## Evidence

- [methods] pLAG definition (>70% group presence, >2 log2 fold change): "we used the threshold of 70% of the genomes belonging to a lifestyle/group and more than a 2 log2 fold change"
- [intro] pLAG counts for phytopathogenic lifestyle: "For Burkholderia/Paraburkholderia and Pseudomonas, we obtained a total of 786 and 377 predicted lifestyle-associated genes (pLAGs) for a phytopathogenic lifestyle, respectively"
- [readme] analytical module description: "Results from the previous modules are embedded in a user-friendly interface for comprehensive and interactive comparative genomics."
- [readme] clustering module upstream dependency: "Predicts, clusters and annotates the genes of every input genome"
