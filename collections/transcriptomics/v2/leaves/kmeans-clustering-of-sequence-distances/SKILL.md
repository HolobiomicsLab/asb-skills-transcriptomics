---
name: kmeans-clustering-of-sequence-distances
description: Use when an ortholog group (e.g., vitellogenin group s2f_0005567) aggregates
  too many genes across species for gene-level insights and must be split into sequence-level
  clusters.
when_to_use_negative:
- The input is raw RNA-seq reads rather than an already-defined ortholog group of
  protein sequences — use Seq2Fun read-to-ortholog mapping instead
- The ortholog group is already small enough for gene-level interpretation, so splitting
  would only fragment functionally coherent proteins across clusters
- 'Sequences lack reliable species assignments, since k depends on #sequences / #species
  and cluster interpretation relies on species hit plots'
edam_operation: http://edamontology.org/operation_3432
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
  role: 'Runs k-means on the cophenetic distance matrix with k = 2 × (#sequences /
    #species)'
- name: ExpressAnalyst
  role: Platform whose EcoOmicsDB ortholog groups (e.g., s2f_0005567) are the input
    to and context for the splitting procedure
  repo: github.com/xia-lab/ExpressAnalystR
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-38785-y
    title: 'ExpressAnalyst: A unified platform for RNA-sequencing analysis in non-model
      species'
  source_content_hash: sha256:a91a56fd1948cfd9777a1a8dc65184f5a510389a2b46ba1525fe0309f9ec9ad5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ExpressAnalyst
  - FastTree
  - MAFFT
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# k-means clustering of sequence distances

## Summary

Resolves oversized ortholog groups in EcoOmicsDB by clustering protein sequences on cophenetic distances from a FastTree dendrogram (built from a MAFFT alignment) with k-means at k = 2 × (#sequences / #species), yielding gene-level-resolution subclusters.

## When to use

Use when an ortholog group (e.g., vitellogenin group s2f_0005567) aggregates too many genes across species for gene-level insights and must be split into sequence-level clusters.

## When NOT to use

- The input is raw RNA-seq reads rather than an already-defined ortholog group of protein sequences — use Seq2Fun read-to-ortholog mapping instead
- The ortholog group is already small enough for gene-level interpretation, so splitting would only fragment functionally coherent proteins across clusters
- Sequences lack reliable species assignments, since k depends on #sequences / #species and cluster interpretation relies on species hit plots

## Inputs

- Protein sequences of an EcoOmicsDB ortholog group (e.g., s2f_0005567, vitellogenin)
- Species assignment for each sequence in the group

## Outputs

- Sequence-to-cluster assignments table (e.g., s2f_0005567_cluster_assignments.tsv)
- Figure of the clustered dendrogram/distance space annotated with the cluster IDs (e.g., s2f_0005567_clusters.png)
- Phylogenetic tree annotated with species hit plots per cluster

## How to apply

Retrieve the protein sequences of the ortholog group together with each sequence's species assignment. Build a multiple sequence alignment of all group sequences with MAFFT, infer a dendrogram with FastTree, and extract the cophenetic distance matrix from the tree. Run k-means (scikit-learn) on that distance matrix with k = 2 × (#sequences / #species) — for the vitellogenin group s2f_0005567 this yields the reported 8 clusters. The choice of k embodies a resolution tradeoff: higher k minimizes grouping of distinct functional proteins but makes grouping across species harder, so k should scale with the sequences-to-species ratio rather than be fixed arbitrarily. Write sequence-to-cluster assignments to a TSV and render the clustered dendrogram/distance space annotated with cluster IDs and species hit plots per cluster on a phylogenetic tree.

## Related tools

- **MAFFT** (Builds the multiple sequence alignment of all protein sequences in the ortholog group)
- **FastTree** (Infers the dendrogram from the MAFFT alignment from which the cophenetic distance matrix is extracted)
- **scikit-learn** (Runs k-means on the cophenetic distance matrix with k = 2 × (#sequences / #species))
- **ExpressAnalyst** (Platform whose EcoOmicsDB ortholog groups (e.g., s2f_0005567) are the input to and context for the splitting procedure) — github.com/xia-lab/ExpressAnalystR

## Examples

```
from sklearn.cluster import KMeans; labels = KMeans(n_clusters=2*(n_seqs//n_species), n_init=10).fit_predict(cophenetic_dist_matrix)
```

## Evaluation signals

- The clustering must reproduce the reported result of 8 clusters for ortholog group s2f_0005567
- Every sequence in the group is assigned to exactly one cluster (assignments form a partition)
- k equals 2 × (#sequences / #species) computed from the retrieved group's composition
- Each resulting cluster can be annotated on the phylogenetic tree with a species hit plot, i.e., clusters are interpretable in species space
- Distance matrix is the cophenetic matrix derived from the FastTree dendrogram of the MAFFT alignment, not raw pairwise identity scores

## Limitations

- Choosing k involves an inherent tradeoff: higher k (increased resolution) makes it easier to avoid grouping distinct functional proteins together but harder to group across species
- Reads/sequences from the same gene may map to similar but different orthologs across species, requiring care when interpreting taxonomically diverse comparisons
- Improper selection of the EcoOmicsDB taxonomic sub-group database may miss key functional groups, degrading the ortholog group being split

## Evidence

- [other] k formula from task card: [evidence span withheld — anchor status: not_found]
- [other] alignment and tree steps from task card: [evidence span withheld — anchor status: not_found]
- [other] validation criterion from task card: [evidence span withheld — anchor status: not_found]
- [discussion] k tradeoff from article discussion: "There is an inherent tradeoff between these two objectives because the higher the resolution (increased value of k), the easier to minimize grouping of functional proteins but the harder to group"
- [discussion] cross-species ortholog caveat from article discussion: "it’s possible for reads from the same gene to be mapped to similar but different orthologs for different species, and care must be taken while interpreting the results"
