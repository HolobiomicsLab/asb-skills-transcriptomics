---
name: fasttree-dendrogram-inference
description: 'Use when an oversized EcoOmicsDB ortholog group (too many genes aggregated
  for gene-level insight) must be split by phylogenetic distance: given protein sequences
  plus species assignments, infer the dendrogram needed for cophenetic-distance k-means
  clustering.'
when_to_use_negative:
- Input is raw RNA-seq reads rather than protein sequences of an existing ortholog
  group — read mapping/quantification should go through Seq2Fun/ExpressAnalyst first
- The ortholog group is small enough to allow gene-level insight without splitting
  — k-means/phylogenetic splitting adds no value and risks over-fragmentation
- Sequences lack species assignments — the k formula and per-cluster species hit plots
  cannot be constructed
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_0084
- http://edamontology.org/topic_0080
tools:
- name: MAFFT
  role: Builds the multiple sequence alignment of all ortholog group sequences that
    FastTree requires as input
- name: FastTree
  role: Infers the dendrogram from the MAFFT alignment from which the cophenetic distance
    matrix is extracted
- name: scikit-learn
  role: 'Runs k-means on the FastTree-derived cophenetic distance matrix with k =
    2 x (#sequences / #species) to produce the cluster assignments'
- name: ExpressAnalystR
  role: R package synchronized with the ExpressAnalyst web server; its ortholog framework
    (EcoOmicsDB integration) is the context in which the split groups are analyzed
  repo: github.com/xia-lab/ExpressAnalystR
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-38785-y
    title: 'ExpressAnalyst: A unified platform for RNA-sequencing analysis in non-model
      species'
  source_content_hash: sha256:ca7ee604071570337b2cfec98f47ab56f86fd5c9a4d6b6b0d36196c74a993dba
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ExpressAnalyst
  - ExpressAnalystR
  - FastTree
  - MAFFT
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_0084
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# fasttree-dendrogram-inference

## Summary

Infers a dendrogram with FastTree from a MAFFT multiple sequence alignment of an EcoOmicsDB ortholog group and derives its cophenetic distance matrix, enabling k-means-based ortholog splitting of oversized groups such as vitellogenin s2f_0005567 into 8 clusters.

## When to use

Use when an oversized EcoOmicsDB ortholog group (too many genes aggregated for gene-level insight) must be split by phylogenetic distance: given protein sequences plus species assignments, infer the dendrogram needed for cophenetic-distance k-means clustering.

## When NOT to use

- Input is raw RNA-seq reads rather than protein sequences of an existing ortholog group — read mapping/quantification should go through Seq2Fun/ExpressAnalyst first
- The ortholog group is small enough to allow gene-level insight without splitting — k-means/phylogenetic splitting adds no value and risks over-fragmentation
- Sequences lack species assignments — the k formula and per-cluster species hit plots cannot be constructed

## Inputs

- Protein sequences of an EcoOmicsDB ortholog group (e.g. s2f_0005567) in FASTA format
- Species assignment (taxon label) for each sequence in the group
- Cluster count k = 2 x (#sequences / #species)

## Outputs

- Multiple sequence alignment (MAFFT output) of the ortholog group
- Dendrogram (FastTree tree) and its cophenetic distance matrix
- Sequence-to-cluster assignments table (s2f_0005567_cluster_assignments.tsv)
- Annotated clustered dendrogram/distance-space figure with 8 cluster IDs and species hit plots per cluster (s2f_0005567_clusters.png)

## How to apply

Retrieve all protein sequences of the ortholog group (e.g. s2f_0005567, vitellogenin) together with each sequence's species assignment. Build a multiple sequence alignment of the group's sequences with MAFFT, then infer a dendrogram from that alignment with FastTree and extract the cophenetic distance matrix from the tree. The cophenetic matrix becomes the feature space for k-means (scikit-learn) with k = 2 x (#sequences / #species), which yields the reported 8 clusters for s2f_0005567. Write sequence-to-cluster assignments to s2f_0005567_cluster_assignments.tsv and render an annotated figure of the clustered dendrogram/distance space (s2f_0005567_clusters.png) with species hit plots per cluster. Keep steps strictly sequential — the ExpressAnalystR documentation stresses that skipping steps in such pipelines causes downstream errors.

## Related tools

- **MAFFT** (Builds the multiple sequence alignment of all ortholog group sequences that FastTree requires as input)
- **FastTree** (Infers the dendrogram from the MAFFT alignment from which the cophenetic distance matrix is extracted)
- **scikit-learn** (Runs k-means on the FastTree-derived cophenetic distance matrix with k = 2 x (#sequences / #species) to produce the cluster assignments)
- **ExpressAnalystR** (R package synchronized with the ExpressAnalyst web server; its ortholog framework (EcoOmicsDB integration) is the context in which the split groups are analyzed) — github.com/xia-lab/ExpressAnalystR

## Examples

```
mafft s2f_0005567.faa > s2f_0005567_aln.faa && FastTree s2f_0005567_aln.faa > s2f_0005567_tree.nwk
```

## Evaluation signals

- Clustering reproduces the reported 8 clusters for ortholog group s2f_0005567
- Every sequence in the group is assigned to exactly one cluster (no unassigned/duplicate assignments in s2f_0005567_cluster_assignments.tsv)
- Cluster count matches the k = 2 x (#sequences / #species) formula computed from the group's inputs
- Rendered figure (s2f_0005567_clusters.png) annotates the 8 cluster IDs on the dendrogram/distance space with a species hit plot per cluster
- Pipeline steps executed in sequence without skipping, per ExpressAnalystR guidance

## Limitations

- Choosing k involves an inherent tradeoff: higher k makes it easier to avoid grouping distinct functional proteins but harder to group across species
- Reads/sequences from the same gene may map to similar but different orthologs across species, requiring care in interpretation for taxonomically diverse comparisons
- Improper sub-group database selection may miss key functional groups in EcoOmicsDB
- Many uncharacterized IDs remain in published reference genomes of non-model organisms, which can hamper cluster annotation

## Evidence

- [other] k-means on cophenetic matrix: [evidence span withheld — anchor status: not_found]
- [readme] sequential execution: "Every function must be executed in sequence as it is shown on the R Command history, please do not skip any commands as this can result in errors downstream."
- [discussion] k tradeoff limitation: "There is an inherent tradeoff between these two objectives because the higher the resolution (increased value of k), the easier to minimize grouping of functional proteins"
- [discussion] ortholog database scale: "All protein-coding genes (n = 12,828,537) from 687 organisms covering all major phyla of eukaryotes were downloaded from KEGG"
