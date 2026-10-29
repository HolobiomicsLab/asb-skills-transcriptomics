---
name: hichip-binomial-loop-testing
description: Use when bulk H3K27ac HiChIP enhancer-promoter interactions (gene index,
  enhancer index, loop counts) are available alongside subpopulation cluster assignments
  and you need per-gene p-values quantifying subpopulation-specific loop signal.
when_to_use_negative:
- No bulk HiChIP enhancer-promoter interaction data exist for the population — the
  binomial test requires normalized loop counts per subpopulation
- HiChIP interactions have not been preprocessed/restricted to genes present in the
  scRNA-seq data and enhancers present in the scATAC-seq data
- The goal is deconvolution/clustering itself rather than per-gene loop significance
  — use the full dcHiChIP pipeline (dc3.py) instead
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: DC3
  role: Framework whose coupled clustering produces the subpopulation assignments;
    the binomial loop test operates on its HiChIP enhancer-promoter interactions input
  repo: https://github.com/SUwonglab/DC3
- name: scipy
  role: Provides the one-tailed binomial test (binom.test-equivalent) used to compute
    per-interaction HiChIP p-values
- name: pandas
  role: Loads and tabulates the HiChIP interaction file, gene/enhancer indices, and
    the final GO enrichment comparison table
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-12547-1
    title: DC3 is a method for deconvolution and coupled clustering from bulk and
      single-cell genomics data
  source_content_hash: sha256:044aff01caceea5c9e8b3641fd1afe72aba8029728d1db9d06a949e3b6c42264
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SUwonglab/DC3
  tools:
  - DC3
  - Python
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# hichip-binomial-loop-testing

## Summary

One-tailed binomial test on bulk HiChIP enhancer-promoter loop counts to rank genes per subpopulation, as used in the DC3 framework to assess whether loop information adds signal to scRNA-seq-based subpopulation characterization.

## When to use

Use when bulk H3K27ac HiChIP enhancer-promoter interactions (gene index, enhancer index, loop counts) are available alongside subpopulation cluster assignments and you need per-gene p-values quantifying subpopulation-specific loop signal.

## When NOT to use

- No bulk HiChIP enhancer-promoter interaction data exist for the population — the binomial test requires normalized loop counts per subpopulation
- HiChIP interactions have not been preprocessed/restricted to genes present in the scRNA-seq data and enhancers present in the scATAC-seq data
- The goal is deconvolution/clustering itself rather than per-gene loop significance — use the full dcHiChIP pipeline (dc3.py) instead

## Inputs

- Bulk HiChIP enhancer-promoter interaction file (three columns: gene index, enhancer index, loop counts), e.g. exampledata/hichip.txt
- scRNA-seq expression matrix (rows = genes, columns = cells), e.g. exampledata/E.txt
- Subpopulation cluster assignments / number of subpopulations K

## Outputs

- Per-gene one-tailed binomial p-values for subpopulation-specific HiChIP loop enrichment
- Combined per-gene Fisher-method p-values (scRNA-seq + HiChIP) per subpopulation
- Top-1000 gene sets per subpopulation for GO enrichment comparison

## How to apply

Normalize total HiChIP loop counts to a fixed total N=10 for each subpopulation, then for each enhancer-promoter interaction run a one-tailed binomial test (R binom.test style: nk observed loop successes, expected proportion pk=1/K for K subpopulations, alternative='greater') to obtain a HiChIP p-value. For genes with multiple linked interactions, keep the most significant (smallest) p-value per gene. Combine the HiChIP p-value with the subpopulation-specific scRNA-seq t-test p-value (one-tailed t-test per gene on the expression matrix, e.g. exampledata/E.txt) using Fisher's method, and select the top 1000 genes by smallest combined p-value per subpopulation. The rationale is that bulk HiChIP loop counts link active regulatory elements to target genes in a subpopulation-specific manner, so loop signal should enrich biologically meaningful gene sets (e.g. forebrain development, neuron development) beyond expression alone.

## Related tools

- **DC3** (Framework whose coupled clustering produces the subpopulation assignments; the binomial loop test operates on its HiChIP enhancer-promoter interactions input) — https://github.com/SUwonglab/DC3
- **scipy** (Provides the one-tailed binomial test (binom.test-equivalent) used to compute per-interaction HiChIP p-values)
- **pandas** (Loads and tabulates the HiChIP interaction file, gene/enhancer indices, and the final GO enrichment comparison table)

## Examples

```
from scipy.stats import binomtest; p = binomtest(n_k, 10, 1.0/K, alternative='greater').pvalue  # one-tailed loop test, normalized N=10, expected pk=1/K
```

## Evaluation signals

- -log10(p-values) for GO terms (e.g. forebrain development, neuron development, axonogenesis) from scRNA-seq+HiChIP selection should exceed scRNA-seq-only selection, reproducing the Table 2 improvement pattern (e.g. forebrain development 1.44 → 13.03)
- Gains from adding HiChIP information should be largest at low sequencing depths
- Each subpopulation's loop counts must sum to the normalized total N=10 before testing
- Genes with multiple enhancer-promoter interactions must be represented by their single most significant p-value
- Selected gene set size equals 1000 genes per subpopulation, ranked by smallest Fisher-combined p-value

## Limitations

- The binomial null assumes uniform expected proportion pk=1/K across subpopulations; results depend on the normalization choice of N=10 total loop counts
- HiChIP data must first be restricted to genes in the scRNA-seq matrix and enhancers in the scATAC-seq openness matrix (preprocessed with HiC-pro and hichipper)
- DC3's joint analysis assumes the bulk and single-cell samples come from the same heterogeneous cell population

## Evidence

- [readme] HiChIP file format: "The first column indicates the index of gene, the second column indicates the index of enhancer, the third column indicates the HiChIP loop counts."
- [readme] HiChIP preprocessing restriction: "please restrict HiChIP enhancer-promoter interactions in the genes in scRNA-seq data and enhancers in scATAC-seq data."
- [readme] Recommended HiChIP preprocessing tools: "We recommend use HiC-pro and hichipper to preprocess the HiChIP data."
- [abstract] Joint-analysis motivation for HiChIP: "as a method for the joint analysis of various bulk and single-cell data such as HiChIP, RNA-seq and ATAC-seq from the same heterogeneous cell population"
- [discussion] Simultaneous deconvolution and coupled clustering: "DC3 for simultaneous deconvolution and coupled clustering based on the joint analysis of different combinations of bulk and single-cell level RNA-seq, ATAC-seq, and HiChIP data"
- [readme] Subpopulation-specific peak-gene pair output: "cluster-specific-peaks-genes-pairs.txt  the cluster-specific peak-gene pairs."
