---
name: benjamini-hochberg-fdr-correction
description: Use when a differential expression meta-analysis yields one combined
  P value per gene per cell subtype and you must control multiplicity before ranking
  marker genes (e.g., recovering Sst, Pvalb, Vip, Cck, Htr3a as subtype discriminators).
when_to_use_negative:
- P values are already FDR-adjusted (q-values); re-applying BH would over-correct
  and shrink the marker list
- There is no multiple-testing family — e.g., a single pre-specified marker test rather
  than per-gene P values across the transcriptome per subtype
- Expression is raw counts or normalized values from non-interneuron datasets where
  the per-study Wilcoxon + Fisher's meta-analysis design does not apply
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: R
  role: Environment in which the Wilcoxon rank-sum tests, Fisher's-method combination,
    and Benjamini-Hochberg correction (p.adjust) are executed on the scRNA-seq DE
    results
- name: MetaNeighbor
  role: Provides the replicated subtype labels (MetaNeighborUS reciprocal-top-hits,
    AUROC > 0.95) that define the DE test families the FDR correction is applied to
  repo: github.com/gillislab/MetaNeighbor
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-018-03282-0
    title: Characterizing the replicability of cell types defined by single cell RNA-sequencing
      data using MetaNeighbor
  source_content_hash: sha256:b172250d1d3e2e940f5c8a20a23d2ee35ba994a6c3a48bdb70de5d35b91cc5fe
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Matrix
  - MetaNeighbor
  - R
  - SummarizedExperiment
  - gplots
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# Benjamini-Hochberg FDR correction

## Summary

Applies Benjamini-Hochberg false discovery rate correction to per-gene meta-analysis P values (Fisher's method combined across GSE60361 and GSE71585) from subtype-vs-rest Wilcoxon differential expression tests, retaining genes with FDR < 0.001 and log2FC > 2 to build marker_genes_fdr001.csv.

## When to use

Use when a differential expression meta-analysis yields one combined P value per gene per cell subtype and you must control multiplicity before ranking marker genes (e.g., recovering Sst, Pvalb, Vip, Cck, Htr3a as subtype discriminators).

## When NOT to use

- P values are already FDR-adjusted (q-values); re-applying BH would over-correct and shrink the marker list
- There is no multiple-testing family — e.g., a single pre-specified marker test rather than per-gene P values across the transcriptome per subtype
- Expression is raw counts or normalized values from non-interneuron datasets where the per-study Wilcoxon + Fisher's meta-analysis design does not apply

## Inputs

- Per-gene Fisher-method combined meta-analysis P values for each replicated interneuron subtype
- Per-gene log2 fold changes from one-sided Wilcoxon rank-sum tests (subtype vs. all other interneurons, per study: GSE60361, GSE71585)
- Replicated subtype cell assignments from MetaNeighborUS reciprocal-top-hit analysis
- Gene-by-cell RPKM expression matrices for cortical interneurons

## Outputs

- BH-adjusted FDR q-values per gene per subtype
- Ranked candidate marker lists passing FDR < 0.001 and log2FC > 2, written to marker_genes_fdr001.csv

## How to apply

For each replicated interneuron subtype (from MetaNeighborUS reciprocal-top-hit assignments), first run one-sided Wilcoxon rank-sum tests of subtype cells vs. all other interneurons separately within GSE60361 (Zeisel) and GSE71585 (Tasic), then combine the two per-study P values per gene with Fisher's method. Apply Benjamini-Hochberg FDR correction to the combined P values across all genes within each subtype's test family, using R's p.adjust(..., method='BH'). Retain genes passing FDR < 0.001 and log2FC > 2, rank surviving markers by combined evidence, and write the ranked lists to marker_genes_fdr001.csv. Correctness is judged by recovery of the canonical Cre-driver genes (Sst, Pvalb, Vip, Cck, Htr3a) in their expected subtype lists and Ptn as a top-ranked novel candidate enriched in the Sst Chodl / Som-Nos1 subtypes.

## Related tools

- **R** (Environment in which the Wilcoxon rank-sum tests, Fisher's-method combination, and Benjamini-Hochberg correction (p.adjust) are executed on the scRNA-seq DE results)
- **MetaNeighbor** (Provides the replicated subtype labels (MetaNeighborUS reciprocal-top-hits, AUROC > 0.95) that define the DE test families the FDR correction is applied to) — github.com/gillislab/MetaNeighbor

## Examples

```
p_adj <- p.adjust(meta_p, method = "BH"); markers <- genes[p_adj < 0.001 & log2FC > 2]
```

## Evaluation signals

- BH q-values are monotonically non-decreasing with raw combined P values within each subtype's gene list
- Number of retained markers per subtype exactly equals genes with FDR < 0.001 AND log2FC > 2
- Canonical Cre-driver genes Sst, Pvalb, Vip, Cck, and Htr3a appear in their expected subtype marker lists
- Ptn is recovered as a top-ranked novel candidate enriched in the Sst Chodl / Som-Nos1 subtypes
- marker_genes_fdr001.csv contains one ranked marker list per replicated subtype

## Limitations

- The dual cutoff (FDR < 0.001 and log2FC > 2) is strict; subtypes with few cells may yield very short or empty marker lists
- Correction is applied to Fisher's-method combined P values, which assume the two per-study Wilcoxon P values are combinable per gene; violated assumptions propagate into the adjusted q-values
- Marker genes are prioritized from expression data alone; the article notes orthogonal data will be required to firmly establish cell identity

## Evidence

- [methods] Apply Benjamini-Hochberg FDR correction and retain genes passing FDR < 0.001 and log2FC > 2: [evidence span withheld — anchor status: not_found]
- [methods] Fisher combination of per-study P values: [evidence span withheld — anchor status: not_found]
- [intro] Canonical marker recovery and Ptn prioritization: "Meta-analysis of differential expression across these highly replicable interneuron subtypes correctly identified canonical marker genes, as well as new candidates"
- [readme] R-based analysis environment: [evidence span withheld — anchor status: external_doc]
