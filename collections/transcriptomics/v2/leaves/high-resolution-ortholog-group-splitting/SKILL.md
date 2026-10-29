---
name: high-resolution-ortholog-group-splitting
description: Use when an EcoOmicsDB ortholog group aggregates too many genes across
  species (e.g. vitellogenin s2f_0005567) to permit gene-level insight and needs subdivision
  into phylogenetically coherent clusters.
when_to_use_negative:
- 'The ortholog group already contains roughly one gene per species (the k = 2 x (#sequences
  / #species) rule yields no meaningful splitting)'
- The input is raw RNA-seq reads to be mapped to orthologs (that is Seq2Fun/ExpressAnalyst
  read-mapping, not ortholog-group refinement)
- No species metadata is available, so cross-species grouping cannot be controlled
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0084
tools:
- name: MAFFT
  role: Builds the multiple sequence alignment of all protein sequences in the ortholog
    group
- name: FastTree
  role: Infers the dendrogram from the MAFFT alignment from which the cophenetic distance
    matrix is extracted
- name: scikit-learn
  role: 'Runs k-means on the cophenetic distance matrix with k = 2 x (#sequences /
    #species)'
- name: ExpressAnalyst
  role: Platform whose EcoOmicsDB ortholog database supplies the ortholog group being
    refined
  repo: github.com/xia-lab/ExpressAnalystR
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-38785-y
    title: 'ExpressAnalyst: A unified platform for RNA-sequencing analysis in non-model
      species'
  source_content_hash: sha256:93f5a76e01c4810444b3e9150e0e08e810bbf7f2905cbf90c80d4932455f06f6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ExpressAnalyst
  - FastTree
  - MAFFT
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# high-resolution-ortholog-group-splitting

## Summary

Resolves oversized EcoOmicsDB ortholog groups by aligning their protein sequences with MAFFT, building a FastTree dendrogram, and k-means clustering the cophenetic distance matrix at k = 2 x (#sequences / #species), as demonstrated on the vitellogenin group s2f_0005567 (split into 8 clusters).

## When to use

Use when an EcoOmicsDB ortholog group aggregates too many genes across species (e.g. vitellogenin s2f_0005567) to permit gene-level insight and needs subdivision into phylogenetically coherent clusters.

## When NOT to use

- The ortholog group already contains roughly one gene per species (the k = 2 x (#sequences / #species) rule yields no meaningful splitting)
- The input is raw RNA-seq reads to be mapped to orthologs (that is Seq2Fun/ExpressAnalyst read-mapping, not ortholog-group refinement)
- No species metadata is available, so cross-species grouping cannot be controlled

## Inputs

- Protein sequences of an EcoOmicsDB ortholog group (e.g. s2f_0005567)
- Species assignment for each sequence in the group

## Outputs

- s2f_0005567_cluster_assignments.tsv — sequence-to-cluster assignment table
- s2f_0005567_clusters.png — clustered dendrogram/distance-space figure annotated with the 8 cluster IDs
- Phylogenetic tree with species hit plots per cluster

## How to apply

Retrieve the protein sequences of the target EcoOmicsDB ortholog group (e.g. s2f_0005567, vitellogenin) together with the species assignment of each sequence. Build a multiple sequence alignment of all sequences with MAFFT, then infer a dendrogram from the alignment with FastTree and extract the cophenetic distance matrix from the tree. Run k-means (scikit-learn) on this cophenetic distance matrix with k = 2 x (#sequences / #species), which corresponds to the reported 8 clusters for the vitellogenin group. Write the sequence-to-cluster assignments to s2f_0005567_cluster_assignments.tsv and render the clustered dendrogram/distance space annotated with the cluster IDs (s2f_0005567_clusters.png) plus species hit plots per cluster. The choice of k embodies an inherent tradeoff in the article's resolution scheme: higher k minimizes grouping of distinct functional proteins but makes grouping across species harder.

## Related tools

- **MAFFT** (Builds the multiple sequence alignment of all protein sequences in the ortholog group)
- **FastTree** (Infers the dendrogram from the MAFFT alignment from which the cophenetic distance matrix is extracted)
- **scikit-learn** (Runs k-means on the cophenetic distance matrix with k = 2 x (#sequences / #species))
- **ExpressAnalyst** (Platform whose EcoOmicsDB ortholog database supplies the ortholog group being refined) — github.com/xia-lab/ExpressAnalystR

## Examples

```
from sklearn.cluster import KMeans; labels = KMeans(n_clusters=2 * n_seqs // n_species).fit(cophenetic_matrix).labels_
```

## Evaluation signals

- Clustering must reproduce the reported result of 8 clusters for ortholog group s2f_0005567
- Every sequence must be assigned to exactly one cluster (no unassigned or multi-assigned sequences)
- The number of clusters k must equal 2 x (#sequences / #species) for the group
- The cophenetic distance matrix must be square and symmetric, with dimensions matching the number of sequences in the group
- The output figure must show the dendrogram/distance space annotated with the cluster IDs and species hit plots per cluster

## Limitations

- Choosing k involves an inherent tradeoff: the higher the resolution (increased k), the easier to minimize grouping of distinct functional proteins but the harder to group across species
- Reads from the same gene may map to similar but different orthologs across species, so care must be taken when interpreting results for taxonomically diverse comparisons
- Improper sub-group database selection may miss key functional groups in the underlying EcoOmicsDB ortholog data

## Evidence

- [other] source card workflow: [evidence span withheld — anchor status: not_found]
- [other] source card finding: [evidence span withheld — anchor status: not_found]
- [discussion] resolution tradeoff: "There is an inherent tradeoff between these two objectives because the higher the resolution (increased value of k), the easier to minimize grouping of functional proteins but the harder to group"
- [intro] motivation for high-resolution orthologs: [evidence span withheld — anchor status: not_found]
