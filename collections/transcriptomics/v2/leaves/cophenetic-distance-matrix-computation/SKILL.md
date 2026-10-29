---
name: cophenetic-distance-matrix-computation
description: Use when an EcoOmicsDB ortholog group aggregates too many genes across
  species for gene-level insight and must be split into smaller clusters based on
  phylogenetic distance.
when_to_use_negative:
- The ortholog group is small enough (relative to species count) that gene-level insight
  is already attainable without splitting
- The task is standard DE or enrichment analysis on an annotated count table (ExpressAnalystR's
  normalization/DE modules handle that directly)
- No multiple sequence alignment or tree can be inferred for the group, so cophenetic
  distances are undefined
edam_operation: http://edamontology.org/operation_0503
edam_topics:
- http://edamontology.org/topic_0194
tools:
- name: MAFFT
  role: Builds the multiple sequence alignment of all ortholog group sequences that
    feeds dendrogram inference
- name: FastTree
  role: Infers the dendrogram from the MAFFT alignment from which the cophenetic distance
    matrix is extracted
- name: scikit-learn
  role: 'Runs k-means clustering on the cophenetic distance matrix with k = 2 x (#sequences
    / #species)'
- name: ExpressAnalyst
  role: Platform integrated with EcoOmicsDB from which the ortholog group protein
    sequences and species assignments are retrieved
  repo: github.com/xia-lab/ExpressAnalystR
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-38785-y
    title: 'ExpressAnalyst: A unified platform for RNA-sequencing analysis in non-model
      species'
  source_content_hash: sha256:37ab15b64aae1465266438272441aa150db48a7c9092f40d15c2f7002cc92a94
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ExpressAnalyst
  - FastTree
  - MAFFT
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0503
  edam_topics:
  - http://edamontology.org/topic_0194
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# cophenetic-distance-matrix-computation

## Summary

Computes a cophenetic distance matrix from a phylogenetic dendrogram inferred from a multiple sequence alignment (MAFFT + FastTree) so that genes in an oversized EcoOmicsDB ortholog group (e.g., vitellogenin group s2f_0005567) can be subdivided by k-means into biologically meaningful sub-clusters.

## When to use

Use when an EcoOmicsDB ortholog group aggregates too many genes across species for gene-level insight and must be split into smaller clusters based on phylogenetic distance.

## When NOT to use

- The ortholog group is small enough (relative to species count) that gene-level insight is already attainable without splitting
- The task is standard DE or enrichment analysis on an annotated count table (ExpressAnalystR's normalization/DE modules handle that directly)
- No multiple sequence alignment or tree can be inferred for the group, so cophenetic distances are undefined

## Inputs

- Protein sequences of an EcoOmicsDB ortholog group (e.g., s2f_0005567, vitellogenin)
- Species assignment for each sequence in the group
- MAFFT multiple sequence alignment of all sequences in the group
- FastTree-inferred dendrogram from the alignment

## Outputs

- Cophenetic distance matrix extracted from the FastTree dendrogram
- Sequence-to-cluster assignments table (s2f_0005567_cluster_assignments.tsv)
- Annotated figure of the clustered dendrogram/distance space with 8 cluster IDs (s2f_0005567_clusters.png)

## How to apply

Retrieve the protein sequences of the ortholog group (e.g., s2f_0005567) with species assignments from EcoOmicsDB, then build a multiple sequence alignment of all sequences with MAFFT. Infer a dendrogram from the alignment with FastTree and extract the cophenetic distance matrix from the tree, which encodes the tree distance at which each pair of sequences shares a common ancestor. Run k-means (scikit-learn) on this distance matrix using k = 2 x (#sequences / #species) — this formula yields the reported 8 clusters for the vitellogenin group s2f_0005567. Write sequence-to-cluster assignments to s2f_0005567_cluster_assignments.tsv and render a figure of the clustered dendrogram/distance space annotated with the 8 cluster IDs (s2f_0005567_clusters.png).

## Related tools

- **MAFFT** (Builds the multiple sequence alignment of all ortholog group sequences that feeds dendrogram inference)
- **FastTree** (Infers the dendrogram from the MAFFT alignment from which the cophenetic distance matrix is extracted)
- **scikit-learn** (Runs k-means clustering on the cophenetic distance matrix with k = 2 x (#sequences / #species))
- **ExpressAnalyst** (Platform integrated with EcoOmicsDB from which the ortholog group protein sequences and species assignments are retrieved) — github.com/xia-lab/ExpressAnalystR

## Examples

```
from sklearn.cluster import KMeans; labels = KMeans(n_clusters=2*len(seqs)//len(species), random_state=0).fit_predict(cophenetic_matrix)
```

## Evaluation signals

- Clustering must reproduce the reported result of 8 clusters for ortholog group s2f_0005567
- Every sequence in the group must be assigned to exactly one cluster (partition completeness, no unassigned or double-assigned sequences)
- The chosen k equals 2 x (#sequences / #species) for the group
- s2f_0005567_cluster_assignments.tsv contains a sequence-to-cluster row for every sequence in the group
- The rendered PNG annotates the dendrogram/distance space with all 8 cluster IDs and per-cluster species hit plots

## Limitations

- Choosing k for clustering involves an inherent tradeoff: higher k minimizes grouping of distinct functional proteins but makes grouping across species harder
- Improper sub-group database selection may miss key functional groups, affecting which sequences enter the split
- Reads from the same gene may map to similar but different orthologs across species, requiring care in interpretation for taxonomically diverse comparisons

## Evidence

- [other] dendrogram + cophenetic extraction: [evidence span withheld — anchor status: not_found]
- [other] k-means with paper-specific k formula: [evidence span withheld — anchor status: not_found]
- [discussion] k-selection tradeoff: "There is an inherent tradeoff between these two objectives because the higher the resolution (increased value of k), the easier to minimize grouping of functional proteins but the harder to group"
- [abstract] ortholog database source: "All modules are integrated with EcoOmicsDB, an ortholog database that enables comprehensive analysis for species without a reference transcriptome."
