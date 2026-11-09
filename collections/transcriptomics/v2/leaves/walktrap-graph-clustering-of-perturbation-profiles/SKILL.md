---
name: walktrap-graph-clustering-of-perturbation-profiles
description: Use when you have a perturbation-by-gene matrix of zdWUI z-scores (or
  analogous effect-size profiles) and need to group perturbations that cause similar
  patterns of 3'UTR shortening or lengthening across response genes.
when_to_use_negative:
- Input is raw single-cell UMI counts or unnormalized gene expression — clustering
  requires perturbation-level zdWUI z-scores of 3'UTR-length responses, not abundance
  data
- Perturbation profiles lack a matched non-targeting control set (the zdWUI normalization
  is defined against the 97 non-targeting guides)
- The goal is clustering cells rather than perturbations — this skill operates on
  pseudobulk perturbation response profiles only
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scUTRboot
  role: Computes the zdWUI z-scores per perturbation relative to non-targeting controls
    that form the clustering input matrix
  repo: https://github.com/mfansler/scutrboot
- name: scUTRquant
  role: Produces the gene-level and 3'UTR isoform SingleCellExperiment objects from
    the 48 K562 Perturb-seq BAM inputs that are pseudobulked per perturbation
  repo: https://github.com/Mayrlab/scUTRquant
- name: kallisto-bustools
  role: Pseudoaligns reads with kallisto bus and corrects barcodes with bustools correct
    inside the scUTRquant pipeline
- name: gwps-sq
  role: Repository holding the initial filtering, pseudobulk summarization, and Perturb-seq
    analysis stages on K562/RPE1 data reprocessed with scUTRquant
  repo: https://github.com/Mayrlab/gwps-sq
- name: Snakemake
  role: Workflow engine for the scUTRquant pipeline run on the Perturb-seq BAM inputs
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-48254-9
    title: Quantifying 3′UTR length from scRNA-seq data reveals changes independent
      of gene expression
  source_content_hash: sha256:da277de9f259f3a423e49671bb154e8fb03c1085513a495ed64d201f5aa2b65a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mfansler/scutrboot
  tools:
  - Conda
  - Snakemake
  - gwps-sq
  - kallisto-bustools
  - scUTRboot
  - scUTRquant
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# Walktrap graph-based clustering of perturbation profiles

## Summary

Groups genetic perturbations into clusters sharing similar 3'UTR-length response signatures by running walktrap graph-based clustering on a perturbation-by-gene matrix of z-scaled dWUI (zdWUI) values, yielding 18 perturbation clusters in the K562 Perturb-seq analysis (836 perturbations x 883 response genes).

## When to use

Use when you have a perturbation-by-gene matrix of zdWUI z-scores (or analogous effect-size profiles) and need to group perturbations that cause similar patterns of 3'UTR shortening or lengthening across response genes.

## When NOT to use

- Input is raw single-cell UMI counts or unnormalized gene expression — clustering requires perturbation-level zdWUI z-scores of 3'UTR-length responses, not abundance data
- Perturbation profiles lack a matched non-targeting control set (the zdWUI normalization is defined against the 97 non-targeting guides)
- The goal is clustering cells rather than perturbations — this skill operates on pseudobulk perturbation response profiles only

## Inputs

- Perturbation-by-gene zdWUI z-score matrix (836 perturbations x 883 response genes) from scUTRboot, computed relative to 97 non-targeting control guides
- Pseudobulk 3'UTR isoform counts per perturbation (SingleCellExperiment) from scUTRquant on the 48 K562 Perturb-seq BAM inputs with 'utrome_hg38_v1' targets
- Perturbation effect-size/quality filter criteria for retaining response genes and perturbations

## Outputs

- perturbation_clusters.csv — perturbation-to-cluster assignment table with 836 perturbation rows and cluster labels spanning exactly 18 distinct clusters
- Groupings of knockdown factors whose knockdown in K562 cells caused similar patterns of 3'UTR shortening or lengthening in specific groups of response genes

## How to apply

First compute perturbation-level zdWUI z-scores with scUTRboot, testing each perturbation's 3'UTR-length response against the 97 non-targeting control guides in the K562 Perturb-seq data, and filter to the response genes passing effect-size/quality criteria (836 perturbations x 883 response genes). Build a graph over perturbation response profiles and run walktrap graph-based clustering in three successive rounds with decreasing numbers of clusters (k=5, then k=4, then k=3) to progressively merge similar perturbations. The rationale is that perturbations knocking down factors with related APA-regulatory roles produce correlated shortening/lengthening patterns, so multi-round merging yields robust, biologically interpretable groups. The iterative decrease in k is a deliberate coarse-to-fine merge strategy rather than a single-shot cut. Record the final perturbation-to-cluster assignment table and verify it spans exactly 18 distinct clusters.

## Related tools

- **scUTRboot** (Computes the zdWUI z-scores per perturbation relative to non-targeting controls that form the clustering input matrix) — https://github.com/mfansler/scutrboot
- **scUTRquant** (Produces the gene-level and 3'UTR isoform SingleCellExperiment objects from the 48 K562 Perturb-seq BAM inputs that are pseudobulked per perturbation) — https://github.com/Mayrlab/scUTRquant
- **kallisto-bustools** (Pseudoaligns reads with kallisto bus and corrects barcodes with bustools correct inside the scUTRquant pipeline)
- **gwps-sq** (Repository holding the initial filtering, pseudobulk summarization, and Perturb-seq analysis stages on K562/RPE1 data reprocessed with scUTRquant) — https://github.com/Mayrlab/gwps-sq
- **Snakemake** (Workflow engine for the scUTRquant pipeline run on the Perturb-seq BAM inputs)

## Examples

```
library(igraph); comm <- cluster_walktrap(graph_from_adjacency_matrix(sim, mode='undirected', weighted=TRUE)); clusters <- cut_at(comm, 18)
```

## Evaluation signals

- perturbation_clusters.csv exists and contains exactly 836 perturbation rows
- Cluster labels in perturbation_clusters.csv span exactly 18 distinct clusters
- The input zdWUI matrix has 883 response-gene columns matching the filtered response-gene set
- Successive walktrap rounds show monotonic merge behavior: k=5 rounds followed by k=4 and k=3 rounds converge on 18 final clusters
- Cluster members share coherent zdWUI patterns — perturbations within a cluster show similar 3'UTR shortening/lengthening directions against the same response-gene groups

## Limitations

- Clustering depends on quality filtering of the zdWUI matrix (effect-size/quality criteria); perturbations failing these criteria are excluded, so results are sensitive to the filter stringency
- The 18-cluster outcome is specific to the three-round k=5/k=4/k=3 scheme and the 836 x 883 matrix; other granularity choices will produce different cluster counts
- The perturbation set is restricted to K562 essential-gene knockdowns; most 10x Genomics reads do not span mRNA 3' end cleavage sites, so 3'UTR isoform resolution relies on the curated CS annotations rather than de novo mapping

## Evidence

- [discussion] Coupled with scUTRboot, significant differences in 3′UTRs across samples are identified, which facilitates the integration of 3′UTR quantification into standard scRNA-seq data analysis: "Coupled with scUTRboot, significant differences in 3′UTRs across samples are identified, which facilitates the integration of 3′UTR quantification into standard"
- [discussion] Perturb-seq analysis to build a catalog of 3′UTR regulators: "we analyzed the global 3′UTR changes in a Perturb-seq data set that contains over 2000 knockdown experiments"
- [readme] Perturb-seq reprocessed with scUTRquant for 3'UTR isoform analysis: [evidence span withheld — anchor status: external_doc]
- [intro] Statistical testing via scUTRboot: "and a statistical testing package, called scUTRboot"
