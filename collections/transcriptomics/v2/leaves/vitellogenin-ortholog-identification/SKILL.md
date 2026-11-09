---
name: vitellogenin-ortholog-identification
description: Use when an EcoOmicsDB ortholog group (like vitellogenin s2f_0005567)
  aggregates too many genes to yield gene-level insights and must be subdivided into
  clusters before downstream annotation or expression analysis.
when_to_use_negative:
- The ortholog group is already small enough for gene-level insight — k = 2 x (#sequences/#species)
  would produce near-trivial per-sequence clusters.
- The goal is read-level quantification of raw RNA-seq FASTQ data — use Seq2Fun/ExpressAnalyst
  read mapping instead of ortholog-group refinement.
- The gene is already present in the KEGG KO system and does not require EcoOmicsDB
  ortholog-based analysis.
edam_operation: http://edamontology.org/operation_2938
edam_topics:
- http://edamontology.org/topic_0084
tools:
- name: EcoOmicsDB
  role: Source of the ortholog group s2f_0005567 (vitellogenin) and its per-sequence
    species assignments; integrated with ExpressAnalyst modules.
- name: MAFFT
  role: Builds the multiple sequence alignment of all sequences in the ortholog group,
    the input for tree inference.
- name: FastTree
  role: Infers the dendrogram from the MAFFT alignment from which the cophenetic distance
    matrix is extracted.
- name: scikit-learn
  role: 'Performs k-means clustering on the cophenetic distance matrix with k = 2
    x (#sequences / #species).'
- name: ExpressAnalyst
  role: Web platform whose modules are integrated with EcoOmicsDB; provides the surrounding
    ortholog-based analysis for species without a reference transcriptome.
  repo: github.com/xia-lab/ExpressAnalystR
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-38785-y
    title: 'ExpressAnalyst: A unified platform for RNA-sequencing analysis in non-model
      species'
  source_content_hash: sha256:68f8f22960d6860f2a0f9acf06898cc3e78a0fe11c89a6c9eb2cfa237959803e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EcoOmicsDB
  - ExpressAnalyst
  - FastTree
  - MAFFT
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2938
  edam_topics:
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# vitellogenin-ortholog-identification

## Summary

Refines oversized EcoOmicsDB ortholog groups (e.g., the vitellogenin group s2f_0005567, which is absent from the KO system) by splitting them into sequence-level clusters via k-means on a FastTree-derived cophenetic distance matrix, restoring gene-level insight for non-model species.

## When to use

Use when an EcoOmicsDB ortholog group (like vitellogenin s2f_0005567) aggregates too many genes to yield gene-level insights and must be subdivided into clusters before downstream annotation or expression analysis.

## When NOT to use

- The ortholog group is already small enough for gene-level insight — k = 2 x (#sequences/#species) would produce near-trivial per-sequence clusters.
- The goal is read-level quantification of raw RNA-seq FASTQ data — use Seq2Fun/ExpressAnalyst read mapping instead of ortholog-group refinement.
- The gene is already present in the KEGG KO system and does not require EcoOmicsDB ortholog-based analysis.

## Inputs

- Protein sequences of an EcoOmicsDB ortholog group (e.g., s2f_0005567, vitellogenin)
- Species assignment for each sequence in the group

## Outputs

- s2f_0005567_cluster_assignments.tsv (sequence-to-cluster assignments)
- s2f_0005567_clusters.png (clustered dendrogram/distance space annotated with the 8 cluster IDs)
- Per-cluster phylogenetic tree annotations with species hit plots

## How to apply

Retrieve the protein sequences of the EcoOmicsDB ortholog group (e.g., s2f_0005567, vitellogenin) together with the species assignment of each sequence. Build a multiple sequence alignment of all sequences with MAFFT, then infer a dendrogram with FastTree and extract the cophenetic distance matrix from the tree. Run scikit-learn k-means on that cophenetic distance matrix with k = 2 x (#sequences / #species), which resolves to 8 clusters for the reported vitellogenin group. Write the sequence-to-cluster assignments to a TSV (s2f_0005567_cluster_assignments.tsv) and render a figure of the clustered dendrogram/distance space annotated with the cluster IDs (s2f_0005567_clusters.png), with each cluster annotated on a phylogenetic tree and species hit plots per cluster. The rationale is that vitellogenin is absent from the KO system, so EcoOmicsDB orthologs plus this refinement procedure are required to make the biomarker analyzable.

## Related tools

- **EcoOmicsDB** (Source of the ortholog group s2f_0005567 (vitellogenin) and its per-sequence species assignments; integrated with ExpressAnalyst modules.)
- **MAFFT** (Builds the multiple sequence alignment of all sequences in the ortholog group, the input for tree inference.)
- **FastTree** (Infers the dendrogram from the MAFFT alignment from which the cophenetic distance matrix is extracted.)
- **scikit-learn** (Performs k-means clustering on the cophenetic distance matrix with k = 2 x (#sequences / #species).)
- **ExpressAnalyst** (Web platform whose modules are integrated with EcoOmicsDB; provides the surrounding ortholog-based analysis for species without a reference transcriptome.) — github.com/xia-lab/ExpressAnalystR

## Examples

```
from scipy.cluster.hierarchy import cophenet; D = cophenet(Z)[0]  # Z from FastTree Newick of MAFFT alignment
from sklearn.cluster import KMeans; k = 2 * (n_seqs // n_species); labels = KMeans(n_clusters=k).fit_predict(D)
```

## Evaluation signals

- Clustering reproduces the reported result of 8 clusters for ortholog group s2f_0005567.
- Every sequence is assigned to exactly one cluster (no unassigned or double-assigned sequences).
- The chosen k equals 2 x (#sequences / #species), computed from the retrieved group composition.
- The cophenetic distance matrix is derived from a FastTree dendrogram built on the MAFFT alignment, not on raw distances.
- Each cluster is annotated on a phylogenetic tree with species hit plots, and the output figure shows all 8 cluster IDs.

## Limitations

- Choosing k involves an inherent tradeoff: higher k minimizes grouping of distinct functional proteins but makes grouping across species harder.
- Vitellogenin is not found in the KO system, so this pipeline depends entirely on EcoOmicsDB ortholog coverage.
- Improper selection of the ortholog sub-group database can miss key functional groups, distorting the split.

## Evidence

- [intro] vitellogenin KO absence: [evidence span withheld — anchor status: not_found]
- [abstract] EcoOmicsDB integration: "All modules are integrated with EcoOmicsDB, an ortholog database that enables comprehensive analysis for species without a reference transcriptome."
- [discussion] k tradeoff: "There is an inherent tradeoff between these two objectives because the higher the resolution (increased value of k)"
- [discussion] KEGG ortholog DB scale: "All protein-coding genes (n = 12,828,537) from 687 organisms covering all major phyla of eukaryotes were downloaded from KEGG"
- [readme] ExpressAnalystR install: "devtools::install_github("xia-lab/ExpressAnalystR", build = TRUE, build_opts = c("--no-resave-data", "--no-manual"))"
