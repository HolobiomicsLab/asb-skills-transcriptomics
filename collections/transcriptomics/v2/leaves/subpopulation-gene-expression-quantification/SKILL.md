---
name: subpopulation-gene-expression-quantification
description: Use when scRNA-seq cells have been assigned to subpopulations (e.g.,
  by DC3 coupled clustering) and subpopulation-specific expression levels in FPKM
  are required for candidate transcription factor selection or network construction.
when_to_use_negative:
- When cells have not yet been clustered/assigned to subpopulations — run DC3 coupled
  clustering first; without scRNA-result.txt there is no per-subpopulation cell grouping
  to merge reads over
- When the goal is a single-cell-level expression matrix (e.g., UMI counts per cell)
  rather than subpopulation-level bulk-style FPKM profiles
- When the mixture has no bulk or coupled single-cell counterpart (scRNA-seq, scATAC-seq,
  bulk HiChIP from the same population) and deconvolution-based subpopulation identification
  is not the intended analysis
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: DC3
  role: Performs simultaneous deconvolution and coupled clustering to assign scRNA-seq
    (and scATAC-seq) cells to subpopulations whose merged reads are then quantified
    in FPKM; outputs scRNA-result.txt and cluster-specific peak-gene pairs
  repo: github.com/SUwonglab/DC3
- name: MACS2
  role: Calls open chromatin peaks on scATAC-seq reads merged per subpopulation, providing
    peaks whose Homer motif enrichment is combined with the subpopulation FPKM expression
    to identify key regulators
- name: Homer
  role: Computes motif enrichment scores (geometric mean of -log10(p-value) and fold
    change) on MACS2-called subpopulation peaks, used as the motif component of the
    key-regulator importance score
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-12547-1
    title: DC3 is a method for deconvolution and coupled clustering from bulk and
      single-cell genomics data
  source_content_hash: sha256:2b5635dcb31496f5d86ca99fa99982469707ff29c3267181f24a5c5884611c4f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DC3
  - Homer
  - MACS2
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 4
---

# subpopulation-gene-expression-quantification

## Summary

Quantifies gene expression (FPKM) for each DC3-assigned subpopulation of the RA-day 4 mouse ES cell mixture by merging scRNA-seq reads from cells clustered into each subpopulation, providing the expression profile needed for downstream key-regulator and regulatory-network analyses.

## When to use

Use when scRNA-seq cells have been assigned to subpopulations (e.g., by DC3 coupled clustering) and subpopulation-specific expression levels in FPKM are required for candidate transcription factor selection or network construction.

## When NOT to use

- When cells have not yet been clustered/assigned to subpopulations — run DC3 coupled clustering first; without scRNA-result.txt there is no per-subpopulation cell grouping to merge reads over
- When the goal is a single-cell-level expression matrix (e.g., UMI counts per cell) rather than subpopulation-level bulk-style FPKM profiles
- When the mixture has no bulk or coupled single-cell counterpart (scRNA-seq, scATAC-seq, bulk HiChIP from the same population) and deconvolution-based subpopulation identification is not the intended analysis

## Inputs

- scRNA-seq expression matrix (rows = genes, columns = cells) from standard preprocessing
- DC3 scRNA-seq clustering results file (scRNA-result.txt) assigning cells to subpopulations
- scATAC-seq per-cell .bam files and openness matrix (O) with enhancer names, merged per subpopulation
- Gene symbol file and enhancer symbol file used by DC3

## Outputs

- Subpopulation-specific gene expression profile in FPKM (one per subpopulation)
- Key-regulator candidate list per subpopulation (FPKM > 10, motif enrichment score > 2, adjusted p < 0.01)
- Top-30 importance-ranked transcription factors per subpopulation
- DC3 cluster-specific outputs including cluster-specific HiChIP and cluster-specific peak-gene pairs

## How to apply

Take the DC3 scRNA-seq clustering output (scRNA-result.txt) that assigns each single cell to a subpopulation, then merge scRNA-seq reads from all cells assigned to each subpopulation (e.g., subpopulations 1, 2, 3 of the RA-day 4 mixture) and compute subpopulation-specific gene expression in FPKM. In parallel, merge scATAC-seq cells per subpopulation so expression and accessibility profiles remain coupled per cluster. Use the FPKM matrix to select candidate key regulators with thresholds of FPKM > 10, Homer motif enrichment score > 2 (score defined as the geometric mean of -log10(p-value) and fold change), and differential expression versus another subpopulation by t-test with adjusted p-value < 0.01. Rank TFs per subpopulation by importance = log2(FPKM) x expression specificity (maximum fold change versus the other two subpopulations) x motif enrichment score. Validate that the merged profiles reproduce known subpopulation-specific expression (e.g., Lhx1/Neurod1 in subpopulation 1, Gata4/Sox17/Foxa2 in subpopulation 2, Rfx4/Sox3 in subpopulation 3).

## Related tools

- **DC3** (Performs simultaneous deconvolution and coupled clustering to assign scRNA-seq (and scATAC-seq) cells to subpopulations whose merged reads are then quantified in FPKM; outputs scRNA-result.txt and cluster-specific peak-gene pairs) — github.com/SUwonglab/DC3
- **MACS2** (Calls open chromatin peaks on scATAC-seq reads merged per subpopulation, providing peaks whose Homer motif enrichment is combined with the subpopulation FPKM expression to identify key regulators)
- **Homer** (Computes motif enrichment scores (geometric mean of -log10(p-value) and fold change) on MACS2-called subpopulation peaks, used as the motif component of the key-regulator importance score)

## Examples

```
python dc3.py -k 3 -E exampledata/E.txt -O exampledata/O.txt -G_symbol exampledata/gene.txt -E_symbol exampledata/enhancer.txt -hichip exampledata/hichip.txt
```

## Evaluation signals

- Key regulator counts per subpopulation equal 58, 42, and 71 for subpopulations 1, 2, and 3 respectively
- TF sets {Lhx1, Neurod1}, {Gata4, Sox17, Foxa2}, and {Rfx4, Sox3} rank specifically in subpopulations 1, 2, and 3 as reported in the paper
- FPKM values exceed 10 and motif enrichment scores exceed 2 for every reported key regulator, with t-test adjusted p-value < 0.01 (scipy)
- DC3 produces the expected four output files (scATAC-result.txt, scRNA-result.txt, cluster-hichip.txt, cluster-specific-peaks-genes-pairs.txt)

## Limitations

- Thresholds (FPKM > 10, motif enrichment > 2, adjusted p < 0.01) and importance ranking were calibrated on the RA-day 4 retinoic-acid differentiation system; other cell mixtures may require re-tuned cutoffs
- Support for jointly-profiled single-cell data types (e.g., sci-CAR) was not yet incorporated into DC3 and was deferred to future work
- The DC3 README is a dcHiChIP-oriented variant; the number of clusters (-k) and hyper-parameters lambda1/lambda2 must be chosen, though DC3 auto-selects lambda1/lambda2 if omitted

## Evidence

- [abstract] DC3 can simultaneously identify distinct subpopulations, assign single cells to the subpopulations (i.e., clustering) and de-convolve the bulk data into subpopulation-specific data: "DC3 can simultaneously identify distinct subpopulations, assign single cells to the subpopulations (i.e., clustering) and de-convolve the bulk data into subpopulation-specific data"
- [supplementary] Merge scRNA-seq reads from cells assigned to each DC3 subpopulation (1, 2, 3) and compute subpopulation-specific gene expression in FPKM: [evidence span withheld — anchor status: not_found]
- [intro] we apply DC3 on a population obtained after four days of retinoic-acid (RA) induced differentiation of mouse embryonic stem cells: "we apply DC3 on a population obtained after four days of retinoic-acid (RA) induced differentiation of mouse embryonic stem cells"
- [readme] For preprocessing the scRNA-seq data, please following the standard processing pipline to get the expression matrix, where each row represents a gene, each column represents a cell.: "For preprocessing the scRNA-seq data, please following the standard processing pipline to get the expression matrix, where each row represents a gene, each column represents a cell."
