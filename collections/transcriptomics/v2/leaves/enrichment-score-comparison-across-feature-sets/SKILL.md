---
name: enrichment-score-comparison-across-feature-sets
description: Use when two gene-selection strategies (e.g., scRNA-seq-only top-1000
  vs scRNA-seq+HiChIP Fisher-combined top-1000) must be compared via GO enrichment
  -log10(p) per subpopulation and term.
when_to_use_negative:
- Input lacks matched HiChIP loop data or subpopulation cluster assignments, so only
  a single gene-selection strategy is possible
- The question is deconvolution or coupled clustering itself (DC3's core task) rather
  than downstream GO enrichment comparison of alternative gene sets
edam_operation: http://edamontology.org/operation_3754
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0621
tools:
- name: DC3
  role: Produces cluster assignments and cluster-specific peak-gene pairs (cluster-specific-peaks-genes-pairs.txt)
    that define the subpopulations and gene sets compared here
  repo: github.com/SUwonglab/DC3
- name: scipy
  role: Implements the one-tailed t-test, binomial test (binom.test style), and Fisher's
    method p-value combination
- name: pandas
  role: Tabulates -log10(p) values per subpopulation/GO term and writes go_enrichment_comparison.csv
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-12547-1
    title: DC3 is a method for deconvolution and coupled clustering from bulk and
      single-cell genomics data
  source_content_hash: sha256:16e044a835c03db2842c461092c57738231e8b61247ca4ede6978087643c2e9a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DC3
  - Python
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_3754
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# enrichment-score-comparison-across-feature-sets

## Summary

Quantitative comparison of GO enrichment significance (-log10 p-values) between gene sets selected by scRNA-seq p-values alone versus gene sets selected by Fisher-combined scRNA-seq + HiChIP p-values, to test whether adding HiChIP loop information improves subpopulation characterization.

## When to use

Use when two gene-selection strategies (e.g., scRNA-seq-only top-1000 vs scRNA-seq+HiChIP Fisher-combined top-1000) must be compared via GO enrichment -log10(p) per subpopulation and term.

## When NOT to use

- Input lacks matched HiChIP loop data or subpopulation cluster assignments, so only a single gene-selection strategy is possible
- The question is deconvolution or coupled clustering itself (DC3's core task) rather than downstream GO enrichment comparison of alternative gene sets

## Inputs

- scRNA-seq expression matrix (e.g., exampledata/E.txt, genes x cells) with subpopulation cluster assignments
- Per-gene scRNA-seq one-tailed t-test p-values per subpopulation
- HiChIP enhancer-promoter interaction file with loop counts (gene index, enhancer index, loop count columns)
- HiChIP binomial-test p-values per gene (most significant per gene)
- Gene ontology (GO) term annotations

## Outputs

- Comparison table of -log10(enrichment p-values) per subpopulation and GO term for scRNA-seq-only vs scRNA-seq+HiChIP selection (go_enrichment_comparison.csv)
- Top-1000 gene sets per subpopulation for each selection strategy

## How to apply

For each subpopulation, build two gene sets: (a) the top 1000 genes by scRNA-seq one-tailed t-test p-values alone, and (b) the top 1000 genes by Fisher's-method combined p-values, where HiChIP p-values come from a one-tailed binomial test (binom.test with nk successes, expected proportion pk=1/K, alternative='greater') on loop counts normalized to N=10 per interaction, taking the most significant p-value per gene when a gene has multiple enhancer-promoter interactions. Run GO enrichment on both sets and transform enrichment p-values to -log10(p). Tabulate -log10(p) per subpopulation for representative terms such as neuron development, axonogenesis, cardiovascular system development, forebrain development, and brain development. Judge success by whether the combined-selection -log10(p) values exceed the scRNA-seq-only values across terms, reproducing the Table 2 pattern (e.g., forebrain development rising from 1.44 to 13.03 at low depth); write the table to go_enrichment_comparison.csv.

## Related tools

- **DC3** (Produces cluster assignments and cluster-specific peak-gene pairs (cluster-specific-peaks-genes-pairs.txt) that define the subpopulations and gene sets compared here) — github.com/SUwonglab/DC3
- **scipy** (Implements the one-tailed t-test, binomial test (binom.test style), and Fisher's method p-value combination)
- **pandas** (Tabulates -log10(p) values per subpopulation/GO term and writes go_enrichment_comparison.csv)

## Examples

```
python dc3.py -k 2 -E exampledata/E.txt -O exampledata/O.txt -G_symbol exampledata/gene.txt -E_symbol exampledata/enhancer.txt -hichip exampledata/hichip.txt -lambda1 10000 -lambda2 0.0005
```

## Evaluation signals

- -log10(p) values for scRNA-seq+HiChIP selection exceed those for scRNA-seq-only selection across all tested GO terms (neuron development, axonogenesis, cardiovascular system development, forebrain development, brain development)
- Largest improvement is observed at low sequencing depth (e.g., forebrain development -log10(p) rises from 1.44 to 13.03)
- Each compared gene set contains exactly the top 1000 genes ranked by the respective p-value (t-test-only or Fisher-combined)
- HiChIP loop counts are normalized so total loops equal N=10 per interaction before the binomial test with pk=1/K and alternative='greater'
- go_enrichment_comparison.csv has one row per subpopulation x GO term x selection strategy with finite -log10(p) values

## Limitations

- Comparison depends on HiChIP enhancer-promoter interactions being restricted to genes present in the scRNA-seq data and enhancers in the scATAC-seq data
- Improvement magnitude varies with sequencing depth; the gain is largest at low depth and may be smaller at high depth
- DC3/discussion text does not provide dataset accessions or quantitative benchmarks inline; practitioners must obtain example data from the repository

## Evidence

- [results] Adding HiChIP loop profile information improves subpopulation characterization over scRNA-seq alone at all sequencing depths, with the largest gains at low depth (e.g., forebrain development −log10(p) rises from 1.44 to 13.03).: [evidence span withheld — anchor status: not_found]
- [readme] DC3 outputs cluster-specific peak-gene pairs with gene and peak p-values: "cluster-specific-peaks-genes-pairs.txt  the cluster-specific peak-gene pairs. First column is the gene name, second column is the peak name, the third column is the p-value for gene and last column"
- [abstract] Subpopulation-specific profiles obtained by DC3 provide comprehensive characterization: "The subpopulation-specific profiles of gene expression, chromatin accessibility and enhancer-promoter contact obtained by DC3 provide a comprehensive characterization of the gene regulatory system"
- [discussion] DC3 can decompose bulk profiles into subpopulation-specific profiles and enhance clustering performance: "We showed that DC3 can decompose bulk profiles into subpopulation-specific profiles and at the same time enhance clustering performance of the single-cell data."
