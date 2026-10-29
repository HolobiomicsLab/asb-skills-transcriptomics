---
name: tf-motif-enrichment-scoring
description: Use when scATAC-seq cells have been assigned to DC3 subpopulations and
  you need to rank transcription factors as key regulators per subpopulation from
  open-chromatin motif enrichment plus FPKM expression evidence.
when_to_use_negative:
- Input is a bulk-only sample without single-cell data to derive subpopulation assignments
  and merged subpopulation-specific peaks/expression.
- Only one subpopulation exists, so differential expression versus 'at least one other
  subpopulation' and expression-specificity fold changes cannot be computed.
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3170
tools:
- name: DC3
  role: Provides the coupled clustering of scRNA-seq/scATAC-seq cells into subpopulations
    whose merged data feed the key regulator analysis
  repo: https://github.com/SUwonglab/DC3
- name: MACS2
  role: Calls open chromatin peaks on scATAC-seq reads merged per DC3 subpopulation
    before motif enrichment scoring
- name: Homer
  role: Computes motif enrichment scores on the MACS2-called peaks, scored as the
    geometric mean of -log10(p-value) and fold change
- name: scipy
  role: Performs the t-test for differential expression of TFs across subpopulations
    with adjusted p-value < 0.01
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-12547-1
    title: DC3 is a method for deconvolution and coupled clustering from bulk and
      single-cell genomics data
  source_content_hash: sha256:7a955837e50e4c7aad57073729282a5aa9109349871067744b67f46132e69372
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SUwonglab/DC3
  tools:
  - DC3
  - Homer
  - MACS2
  - scipy
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# tf-motif-enrichment-scoring

## Summary

A method for identifying subpopulation-specific key transcription factor regulators by combining Homer motif enrichment scores (geometric mean of -log10 p-value and fold change) on MACS2-called peaks with expression and differential-expression filters, then ranking TFs by a multiplicative importance score.

## When to use

Use when scATAC-seq cells have been assigned to DC3 subpopulations and you need to rank transcription factors as key regulators per subpopulation from open-chromatin motif enrichment plus FPKM expression evidence.

## When NOT to use

- Input is a bulk-only sample without single-cell data to derive subpopulation assignments and merged subpopulation-specific peaks/expression.
- Only one subpopulation exists, so differential expression versus 'at least one other subpopulation' and expression-specificity fold changes cannot be computed.

## Inputs

- scRNA-seq expression matrix with DC3 subpopulation cluster assignments (scRNA-result.txt)
- scATAC-seq per-cell .bam files grouped by DC3 subpopulation
- Subpopulation-specific FPKM expression profiles
- Transcription factor list with motif databases for Homer

## Outputs

- Subpopulation-specific motif enrichment scores (geometric mean of -log10(p-value) and fold change) per TF
- Filtered key regulator list per subpopulation (FPKM > 10, motif enrichment score > 2, adjusted p-value < 0.01)
- Top 30 ranked key TFs per subpopulation by importance score (log2(FPKM) x expression fold-change specificity x motif enrichment score)

## How to apply

Merge scRNA-seq reads from cells assigned to each DC3 subpopulation (1, 2, 3) and compute subpopulation-specific gene expression in FPKM; merge scATAC-seq cells per subpopulation analogously. Call open peaks on each merged scATAC-seq sample with MACS2 and compute motif enrichment with Homer, defining the motif enrichment score as the geometric mean of the -log10(p-value) and the fold change. Select candidate key regulators as transcription factors with FPKM > 10, motif enrichment score > 2, and significant differential expression versus at least one other subpopulation via a t-test with adjusted p-value < 0.01 (scipy). Rank TFs in each subpopulation by the importance score, defined as the product of log2(FPKM), expression specificity (maximum expression fold change versus the other two subpopulations), and motif enrichment score, then output the Top 30 per subpopulation.

## Related tools

- **DC3** (Provides the coupled clustering of scRNA-seq/scATAC-seq cells into subpopulations whose merged data feed the key regulator analysis) — https://github.com/SUwonglab/DC3
- **MACS2** (Calls open chromatin peaks on scATAC-seq reads merged per DC3 subpopulation before motif enrichment scoring)
- **Homer** (Computes motif enrichment scores on the MACS2-called peaks, scored as the geometric mean of -log10(p-value) and fold change)
- **scipy** (Performs the t-test for differential expression of TFs across subpopulations with adjusted p-value < 0.01)

## Examples

```
python dc3.py -k 2 -E exampledata/E.txt -O exampledata/O.txt -G_symbol exampledata/gene.txt -E_symbol exampledata/enhancer.txt -hichip exampledata/hichip.txt -lambda1 10000 -lambda2 0.0005
```

## Evaluation signals

- Key regulator counts per subpopulation equal 58, 42, and 71 for subpopulations 1, 2, and 3 respectively.
- Lhx1 and Neurod1 rank specifically in subpopulation 1; Gata4, Sox17, and Foxa2 in subpopulation 2; Rfx4 and Sox3 in subpopulation 3, matching the paper.
- Every retained key TF passes all three filters: FPKM > 10, motif enrichment score > 2, and t-test adjusted p-value < 0.01.
- Motif enrichment scores are strictly positive geometric means of -log10(p-value) and fold change from Homer on MACS2 peaks.
- Top-ranked lists contain exactly 30 TFs per subpopulation with importance scores reproducible from log2(FPKM) x specificity fold change x motif score.

## Limitations

- Regulatory network derivation depends on DC3-inferred accessibility and expression profiles; errors in coupled clustering propagate into the key regulator rankings.
- The skill assumes the same heterogeneous cell population is shared across scRNA-seq and scATAC-seq samples; jointly-profiled single-cell data (e.g., sci-CAR) is not supported and was deferred to future work.
- Thresholds (FPKM > 10, motif score > 2, adjusted p < 0.01) and the importance-score formula are specific to the RA-day 4 mouse embryonic stem cell differentiation study and may need retuning for other systems.

## Evidence

- [discussion] DC3 for simultaneous deconvolution and coupled clustering based on the joint analysis of different combinations of bulk and single-cell level RNA-seq, ATAC-seq, and HiChIP data: "we developed DC3 for simultaneous deconvolution and coupled clustering based on the joint analysis of different combinations of bulk and single-cell level RNA-seq, ATAC-seq, and HiChIP data"
- [intro] Using DC3 results to derive core regulatory networks per subpopulation: "we illustrate the value of results from DC3 by using them to derive the core regulatory network and their downstream effectors in each of the subpopulation"
- [readme] DC3 outputs clustering results for scATAC-seq used to group cells for merged peak calling: "scATAC-result.txt                       the clustering results for scATAC-seq"
- [abstract] DC3 subpopulation-specific profiles of accessibility and expression: "The subpopulation-specific profiles of gene expression, chromatin accessibility and enhancer-promoter contact obtained by DC3 provide a comprehensive characterization of the gene regulatory system"
