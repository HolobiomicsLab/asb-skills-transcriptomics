---
name: kegg-pathway-enrichment-analysis
description: Use when a BUSseq fit has yielded a ranked intrinsic-gene list (e.g.,
  1,419 genes from hematopoietic scRNA-seq data at Bayesian FDR < 0.05) and biological
  interpretability must be assessed via KEGG pathway enrichment.
when_to_use_negative:
- The gene set is not a differentially expressed / intrinsic gene list from a batch-corrected
  model but an arbitrary or already pathway-filtered list
- Cell-type membership is completely confounded with batch (e.g., batch 1 measures
  cell types 1 and 2, batch 2 measures types 3 and 4), so no valid BUSseq fit — and
  hence no trustworthy intrinsic genes — can be obtained first
- The organism under study lacks usable KEGG pathway annotation
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3308
tools:
- name: BUSseq
  role: Fits the Bayesian hierarchical model and supplies the intrinsic-gene selection
    (via intrinsic_genes_BUSseq at fdr = 0.05) that serves as the query gene set for
    KEGG enrichment
  repo: github.com/songfd2018/BUSseq
- name: SingleCellExperiment
  role: Container for the fitted BUSseq results passed to intrinsic_genes_BUSseq(sce_BUSseqfit)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-16905-2
    title: Flexible experimental designs for valid single-cell RNA-sequencing experiments
      allowing batch effects correction
  source_content_hash: sha256:fd85a21be17fdd257d8d6960c74344627fb5b05bc8dc036a80c28c3175dac345
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BUSseq
  - SingleCellExperiment
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# kegg-pathway-enrichment-analysis

## Summary

Performs KEGG over-representation analysis on intrinsic (differentially expressed) genes selected by BUSseq's spike-and-slab Bayesian-FDR procedure, to confirm that the selected gene set is biologically meaningful rather than statistically arbitrary. In the BUSseq manuscript, this is applied to the 1,419 intrinsic genes identified in the mouse hematopoietic study, where 51 KEGG pathways were enriched.

## When to use

Use when a BUSseq fit has yielded a ranked intrinsic-gene list (e.g., 1,419 genes from hematopoietic scRNA-seq data at Bayesian FDR < 0.05) and biological interpretability must be assessed via KEGG pathway enrichment.

## When NOT to use

- The gene set is not a differentially expressed / intrinsic gene list from a batch-corrected model but an arbitrary or already pathway-filtered list
- Cell-type membership is completely confounded with batch (e.g., batch 1 measures cell types 1 and 2, batch 2 measures types 3 and 4), so no valid BUSseq fit — and hence no trustworthy intrinsic genes — can be obtained first
- The organism under study lacks usable KEGG pathway annotation

## Inputs

- Fitted BUSseq model object stored as a SingleCellExperiment container (e.g., BUSseqfits_example)
- Intrinsic gene list from intrinsic_genes_BUSseq() selected at Bayesian FDR cutoff 0.05 (e.g., 1,419 genes from the mouse hematopoietic study)
- KEGG pathway annotation (KEGG organism database for the species under study, e.g., mouse)

## Outputs

- KEGG pathways enriched among the intrinsic genes (target: 51 enriched KEGG pathways for the hematopoietic dataset)
- Final intrinsic-gene table written to intrinsic_genes.csv
- Enrichment statistics (pathway IDs, gene counts, significance) supporting biological interpretability of the selection

## How to apply

First obtain the intrinsic gene set by applying intrinsic_genes_BUSseq() to the fitted BUSseq SingleCellExperiment object; this function extracts posterior indicator probabilities xi_gk from the normal-mixture spike-and-slab prior on cell-type effects beta_gk, ranks genes by cumulative indicator mass kappa0, and selects genes whose estimated Bayesian FDR stays below alpha = 0.05. Take the resulting gene list (in the hematopoietic case, 1,419 genes) as the query set for KEGG pathway over-representation analysis. Enrichment is a validation step: the manuscript considers the selection biologically meaningful only if 51 KEGG pathways are enriched among the hematopoietic intrinsic genes. Write the final gene table (e.g., intrinsic_genes.csv) alongside the enrichment results so the statistical and functional evidence remain linked.

## Related tools

- **BUSseq** (Fits the Bayesian hierarchical model and supplies the intrinsic-gene selection (via intrinsic_genes_BUSseq at fdr = 0.05) that serves as the query gene set for KEGG enrichment) — github.com/songfd2018/BUSseq
- **SingleCellExperiment** (Container for the fitted BUSseq results passed to intrinsic_genes_BUSseq(sce_BUSseqfit))

## Examples

```
intrinsic_genes <- intrinsic_genes_BUSseq(sce_BUSseqfit)
```

## Evaluation signals

- The intrinsic-gene selection at alpha = 0.05 returns exactly 1,419 intrinsic genes on the mouse hematopoietic fit
- KEGG enrichment of the 1,419 hematopoietic intrinsic genes yields 51 enriched pathways
- On simulated data, the same FDR procedure recovers all 500 planted intrinsic genes with a true FDR of approximately 2% (below the nominal 5%)
- The selected gene table is written to intrinsic_genes.csv with gene identifiers consistent with the input count matrix
- Enriched pathways are consistent with hematopoietic biology (biologically meaningful interpretation rather than generic housekeeping terms)

## Limitations

- MCMC inference for BUSseq carries heavy computational load, and the prior clustering step introduces uncertainty that multi-stage pipelines ignore, potentially causing false positives or false negatives that propagate into the enrichment gene set
- C++ source support is Linux-only (tested on Ubuntu 18.04 and CentOS 7); raw count data for the hematopoietic study must be obtained from the authors' OneDrive link to reproduce the 1,419-gene result
- The 51-pathway enrichment result is dataset-specific; other datasets will yield different pathway counts, so it serves as a reproduction benchmark rather than a universal expectation

## Evidence

- [methods] intrinsic_genes_BUSseq on fitted BUSseq SingleCellExperiment object: "- `intrinsic_genes_BUSseq(sce_BUSseqfit)`"
- [discussion] BUSseq identifies intrinsic genes that differentiate cell types: "BUSseq simultaneously corrects batch effects, clusters cell types, imputes missing data, and identifies intrinsic genes that differentiate cell types."
- [methods] Intrinsic gene selection is downstream of batch-corrected fitting: "the corrected value can be used for downstream analysis as if all cells were sequenced in a single batch"
- [readme] Hematopoietic raw count data required for reproduction of the enrichment analysis: [evidence span withheld — anchor status: external_doc]
- [abstract] BUSseq detects differentially expressed genes without preliminary normalization: [evidence span withheld — anchor status: not_found]
