---
name: multiple-sequence-alignment-of-ortholog-protein-sequences
description: Use when an EcoOmicsDB ortholog group (like Seq2Fun 's2f_' groups) aggregates
  too many genes across species to allow gene-level insights, and the protein sequences
  plus species labels need alignment before clustering.
when_to_use_negative:
- Input is raw RNA-seq reads rather than protein sequences of a defined ortholog group
  — use Seq2Fun read-to-KO mapping instead of ortholog splitting.
- The goal is KO-based functional annotation; note vitellogenin is not found in the
  KO system, which is precisely why ortholog-level splitting is needed for such biomarkers.
- The ortholog group is already small enough to permit gene-level insights without
  splitting.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0194
tools:
- name: ExpressAnalyst
  role: Platform integrated with EcoOmicsDB from which the ortholog group sequences
    and species assignments are retrieved
  repo: github.com/xia-lab/ExpressAnalystR
- name: MAFFT
  role: Builds the multiple sequence alignment of all protein sequences in the ortholog
    group
- name: FastTree
  role: Infers the dendrogram from the alignment from which the cophenetic distance
    matrix is extracted
- name: scikit-learn
  role: 'Runs k-means on the cophenetic distance matrix with k = 2 x (#sequences /
    #species)'
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-38785-y
    title: 'ExpressAnalyst: A unified platform for RNA-sequencing analysis in non-model
      species'
  source_content_hash: sha256:1a0fb48e4279260fb2acdfd7f04842b21a65bbd7df275d33a2427199c6beb48c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ExpressAnalyst
  - FastTree
  - MAFFT
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0194
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# Multiple sequence alignment of ortholog protein sequences

## Summary

Builds a MAFFT multiple sequence alignment of all protein sequences in an EcoOmicsDB ortholog group (e.g. vitellogenin group s2f_0005567), preserving each sequence's species assignment, so that phylogenetic structure within oversized ortholog groups can be quantified for ortholog splitting.

## When to use

Use when an EcoOmicsDB ortholog group (like Seq2Fun 's2f_' groups) aggregates too many genes across species to allow gene-level insights, and the protein sequences plus species labels need alignment before clustering.

## When NOT to use

- Input is raw RNA-seq reads rather than protein sequences of a defined ortholog group — use Seq2Fun read-to-KO mapping instead of ortholog splitting.
- The goal is KO-based functional annotation; note vitellogenin is not found in the KO system, which is precisely why ortholog-level splitting is needed for such biomarkers.
- The ortholog group is already small enough to permit gene-level insights without splitting.

## Inputs

- EcoOmicsDB ortholog group protein sequences (e.g. s2f_0005567, vitellogenin), in FASTA
- Species assignment for each sequence in the ortholog group

## Outputs

- Multiple sequence alignment of the ortholog group proteins (MAFFT output)
- Sequence-to-cluster assignments table (s2f_0005567_cluster_assignments.tsv)
- Figure of the clustered dendrogram/distance space annotated with the 8 cluster IDs (s2f_0005567_clusters.png)

## How to apply

Retrieve the protein sequences of the target ortholog group (e.g. s2f_0005567, vitellogenin) from EcoOmicsDB together with the species assignment of each sequence; vitellogenin is a relevant example because it is a biomarker of interest absent from the KO system. Build a multiple sequence alignment of all sequences in the group with MAFFT. Infer a dendrogram from the alignment with FastTree and extract the cophenetic distance matrix from the tree. Run k-means (scikit-learn) on the cophenetic distance matrix with k = 2 x (#sequences / #species), which yields 8 clusters for s2f_0005567. Write sequence-to-cluster assignments to s2f_0005567_cluster_assignments.tsv and render the clustered dendrogram/distance space annotated with the 8 cluster IDs (s2f_0005567_clusters.png).

## Related tools

- **ExpressAnalyst** (Platform integrated with EcoOmicsDB from which the ortholog group sequences and species assignments are retrieved) — github.com/xia-lab/ExpressAnalystR
- **MAFFT** (Builds the multiple sequence alignment of all protein sequences in the ortholog group)
- **FastTree** (Infers the dendrogram from the alignment from which the cophenetic distance matrix is extracted)
- **scikit-learn** (Runs k-means on the cophenetic distance matrix with k = 2 x (#sequences / #species))

## Examples

```
mafft --auto s2f_0005567.fasta > s2f_0005567.aln.fasta
```

## Evaluation signals

- The clustering must reproduce the reported result of 8 clusters for ortholog group s2f_0005567.
- Every sequence in the ortholog group is assigned to exactly one cluster.
- k equals 2 x (#sequences / #species) for the given group.
- The MSA contains every sequence retrieved from the ortholog group, each retaining its species assignment for cluster-level species-hit plots.
- Outputs include s2f_0005567_cluster_assignments.tsv and s2f_0005567_clusters.png with the 8 cluster IDs annotated on the phylogenetic tree.

## Limitations

- Choosing k for clustering involves an inherent tradeoff: higher k minimizes grouping of distinct functional proteins but makes grouping across species harder.
- Reads from the same gene may map to similar but different orthologs across species, requiring care in interpretation for taxonomically diverse comparisons.
- ExpressAnalystR is still under development; full functionality is not guaranteed, and R functions must be executed in sequence without skipping commands.

## Evidence

- [other] Retrieve the protein sequences of EcoOmicsDB ortholog group s2f_0005567 (vitellogenin) together with the species assignment of each sequence.: [evidence span withheld — anchor status: not_found]
- [other] Run k-means on the cophenetic distance matrix with k = 2 x (#sequences / #species), which corresponds to the reported 8 clusters for this group.: [evidence span withheld — anchor status: not_found]
- [intro] Vitellogenin absent from KO system motivates ortholog-level analysis.: [evidence span withheld — anchor status: not_found]
- [discussion] Tradeoff in choosing k for ortholog clustering.: "There is an inherent tradeoff between these two objectives because the higher the resolution (increased value of k), the easier to minimize grouping of functional proteins but the harder to group"
- [readme] ExpressAnalystR install command grounding the local reproducibility path.: "devtools::install_github("xia-lab/ExpressAnalystR", build = TRUE, build_opts = c("--no-resave-data", "--no-manual"))"
