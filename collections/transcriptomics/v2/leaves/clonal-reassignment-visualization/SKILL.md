---
name: clonal-reassignment-visualization
description: Use when single-cell RNA-seq data carry linked genotype (GoT) calls for
  somatic mutations (e.g., WT, SF3B1, SF3B1-CALR, SF3B1-CALR-NFE2 clones) and conflicting
  cluster annotations must be reconciled to reassign cells to transcriptionally consistent
  clonal populations.
when_to_use_negative:
- Input lacks linked per-cell genotype calls (GoT); this skill specifically integrates
  clonal-genotype annotations with expression clusters, not generic multimodal data
  alone
- The goal is a consensus clustering rather than stability-based selection — scTriangulate
  picks the most stable solution through coalitional iteration rather than selecting
  the consensus
- Cells cannot be reliably assigned to clonal genotypes because shallow sequencing
  false negatives or ambient RNA false positives dominate; genotype labels would be
  too noisy to serve as conflicting annotation sets
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0625
tools:
- name: scTriangulate
  role: Decision-level integration of cellHarmony expression clusters and clonal-genotype
    subclusters via Shapley value; lazy_run with win_fraction_cutoff=0.25 and abs_thresh=10;
    produces the reassignment table and Sankey visualization
  repo: https://github.com/frankligy/scTriangulate
- name: scanpy
  role: AnnData framework hosting the combined RNA + genotype object consumed by scTriangulate;
    preprocessing/QC pipeline
  repo: https://github.com/frankligy/scTriangulate
- name: cellHarmony
  role: Supervised label transfer assigning cells to Hay et al. CD34+ reference populations,
    used as one conflicting annotation set
- name: ICGS2
  role: Cell barcode filtering and normalization of the RNA matrix with default parameters
    prior to integration
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-36016-y
    title: Decision level integration of unimodal and multimodal single cell data
      with scTriangulate
  source_content_hash: sha256:7c85cd22715b8e7562cbcda9a019c10eff14fb4f2ebe06cf30bb79caf07377d8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/frankligy/scTriangulate
  tools:
  - ICGS2
  - cellHarmony
  - scTriangulate
  - scanpy
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# clonal-reassignment-visualization

## Summary

Resolves clonal architecture within molecularly defined single-cell populations by integrating gene expression clusters with linked gene-mutation (GoT) genotype calls via scTriangulate's Shapley-value decision-level integration, then visualizing clone-by-cell-type reassignments (e.g., SF3B1 single-mutant cells) with a two-column Sankey diagram.

## When to use

Use when single-cell RNA-seq data carry linked genotype (GoT) calls for somatic mutations (e.g., WT, SF3B1, SF3B1-CALR, SF3B1-CALR-NFE2 clones) and conflicting cluster annotations must be reconciled to reassign cells to transcriptionally consistent clonal populations.

## When NOT to use

- Input lacks linked per-cell genotype calls (GoT); this skill specifically integrates clonal-genotype annotations with expression clusters, not generic multimodal data alone
- The goal is a consensus clustering rather than stability-based selection — scTriangulate picks the most stable solution through coalitional iteration rather than selecting the consensus
- Cells cannot be reliably assigned to clonal genotypes because shallow sequencing false negatives or ambient RNA false positives dominate; genotype labels would be too noisy to serve as conflicting annotation sets

## Inputs

- Pre-processed gene expression count matrices and genotyping-of-transcriptomes (GoT) matrices from GEO GSE117825
- Per-cell clonal genotype assignments (WT, SF3B1, SF3B1-CALR, SF3B1-CALR-NFE2) derived from wild-type and mutant allele calls
- cellHarmony cluster labels against the Hay et al. CD34+ reference populations
- Normalized RNA AnnData object (log CPM-style normalization from ICGS2) with adata.obsm['X_umap']

## Outputs

- Cell-level clonal reassignment table (one row per cell with final clone and cell-type columns)
- Clone-by-cell-type two-column Sankey diagram (plot_two_column_sankey) showing SF3B1 single-mutant reassignment pattern
- Final scTriangulate cluster assignment per cell with stability metrics

## How to apply

Download pre-processed gene-expression and genotyping-of-transcriptomes count matrices (GEO GSE117825) and assign each cell to one of four clonal genotypes using both wild-type and mutant allele calls, accounting for false negatives from shallow sequencing and false positives from ambient RNA. Normalize the RNA matrix in ICGS2 with default parameters, transfer Hay et al. CD34+ reference labels with cellHarmony, then combine the normalized matrix, cellHarmony clusters, and clonal-genotype subclusters into one AnnData via scTriangulate.preprocessing.concat_rna_and_other. Run scTriangulate 0.9.0 lazy_run with win_fraction_cutoff = 0.25 and default pruning (abs_thresh = 10), letting the Shapley value select the stable annotation per cell and reassigning cells in unstable clusters (winning fraction < 0.25) to nearest-neighbor cluster labels. Export the cell-level clonal reassignment table and render the clone-by-cell-type mapping with plot_two_column_sankey; the rationale is that SF3B1 single-mutant cells reassigned to transcriptionally more similar clones validate the integration.

## Related tools

- **scTriangulate** (Decision-level integration of cellHarmony expression clusters and clonal-genotype subclusters via Shapley value; lazy_run with win_fraction_cutoff=0.25 and abs_thresh=10; produces the reassignment table and Sankey visualization) — https://github.com/frankligy/scTriangulate
- **scanpy** (AnnData framework hosting the combined RNA + genotype object consumed by scTriangulate; preprocessing/QC pipeline) — https://github.com/frankligy/scTriangulate
- **cellHarmony** (Supervised label transfer assigning cells to Hay et al. CD34+ reference populations, used as one conflicting annotation set)
- **ICGS2** (Cell barcode filtering and normalization of the RNA matrix with default parameters prior to integration)

## Examples

```
sctri = ScTriangulate(dir='./output', adata=adata, query=['cellHarmony_labels','clonal_genotype']); sctri.lazy_run(win_fraction_cutoff=0.25)
```

## Evaluation signals

- Output table exists and contains one row per cell with final clone and cell-type columns
- Reassigned SF3B1 single-mutant cells show transcriptomes more similar to their new clonal population than to their original label, reproducing Fig. 6c,d
- Unstable clusters are correctly identified: clusters with winning fraction < 0.25 (wfc) are relabeled via nearest-neighbor reassignment
- Final clusters retain at least 10 cells (abs_thresh default pruning)
- Sankey diagram renders without error and shows the expected clone-to-cell-type flow patterns reported in the paper

## Limitations

- GoT genotype calls are subject to false negatives from shallow sequencing and false positives from ambient RNA, so clone assignments are probabilistic
- Programmatic default analytical settings (e.g., win_fraction_cutoff, abs_thresh) may not be optimal for all study designs
- Integration of cluster predictions from dozens of sources significantly increases runtime; over 15 annotation sets triggers automatic substitution of Shapley selection with combined rank-based selection
- Analysis limited to select somatic mutations within a single tumor

## Evidence

- [methods] Pre-processed count matrices for gene expression and genotyping of transcriptomes were downloaded from GEO (GSE117825).: "Pre-processed count matrices for gene expression and genotyping of transcriptomes were downloaded from GEO (GSE117825)."
- [methods] By default, wfc < 0.25 will be labeled as unstable clusters.: "By default, wfc < 0.25 will be labeled as unstable clusters."
- [methods] the final annotation assignments are pruned according to the fraction of final versus original cell-cluster assignment: "the final annotation assignments are pruned according to the fraction of final versus original cell-cluster assignment (user-defined) and re-classified using a nearest-neighbor strategy."
- [abstract] scTriangulate picks the most stable solution through coalitional iteration rather than selecting the consensus as ensemble approaches do.: "Rather than ensemble approaches which select the "consensus", scTriangulate picks the most stable solution through coalitional iteration."
- [readme] All the results will be saved in the dir you specified (README lazy_run usage).: "sctri = ScTriangulate(dir='./output',adata=adata,query=['sctri_rna_leiden_1','sctri_rna_leiden_2','sctri_rna_leiden_3'])
sctri.lazy_run()"
