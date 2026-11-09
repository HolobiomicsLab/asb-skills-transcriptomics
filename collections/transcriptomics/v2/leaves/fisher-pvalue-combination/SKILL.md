---
name: fisher-pvalue-combination
description: Use when a subpopulation-specific gene ranking must integrate expression
  evidence from scRNA-seq with enhancer-promoter contact evidence from bulk HiChIP
  loop counts, e.g. to improve GO enrichment of subpopulations at low sequencing depth.
when_to_use_negative:
- No bulk HiChIP (or other enhancer-promoter loop) data are available for the same
  cell population — only scRNA-seq p-values can be computed, so no combination is
  possible.
- Genes are not shared between the expression matrix and the HiChIP interaction file,
  since the README restricts HiChIP interactions to 'the genes in scRNA-seq data and
  enhancers in scATAC-seq data'.
- The goal is deconvolution or coupled clustering itself (DC3's dc3.py step), not
  downstream gene-set prioritization.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: DC3
  role: Produces the coupled clustering (scRNA-result.txt) and the subpopulation/peak-gene
    association files whose gene and peak p-values are combined here
  repo: github.com/SUwonglab/DC3
- name: scipy
  role: One-tailed t-tests and binomial tests per gene, and the chi-squared survival
    function used to compute Fisher-combined p-values
- name: pandas
  role: Tabulates -log10(p-values) per subpopulation and writes the go_enrichment_comparison.csv
    comparison table
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-12547-1
    title: DC3 is a method for deconvolution and coupled clustering from bulk and
      single-cell genomics data
  source_content_hash: sha256:10e689dc23db8b07a79e3598ffc439a7e12a00c606706a0fff47f87a6011592c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DC3
  - Python
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# fisher-pvalue-combination

## Summary

Combines per-gene scRNA-seq p-values (one-tailed t-test on the subpopulation-specific expression matrix) with HiChIP enhancer-promoter binomial-test p-values via Fisher's method, so that loop-profile evidence sharpens gene-set selection for subpopulation GO enrichment (Table 2 of DC3).

## When to use

Use when a subpopulation-specific gene ranking must integrate expression evidence from scRNA-seq with enhancer-promoter contact evidence from bulk HiChIP loop counts, e.g. to improve GO enrichment of subpopulations at low sequencing depth.

## When NOT to use

- No bulk HiChIP (or other enhancer-promoter loop) data are available for the same cell population — only scRNA-seq p-values can be computed, so no combination is possible.
- Genes are not shared between the expression matrix and the HiChIP interaction file, since the README restricts HiChIP interactions to 'the genes in scRNA-seq data and enhancers in scATAC-seq data'.
- The goal is deconvolution or coupled clustering itself (DC3's dc3.py step), not downstream gene-set prioritization.

## Inputs

- Subpopulation-specific scRNA-seq expression matrix (rows = genes, columns = cells; e.g. exampledata/E.txt)
- Cluster assignments for single cells (e.g. scRNA-result.txt from DC3)
- HiChIP enhancer-promoter interaction file with gene index, enhancer index and loop counts (e.g. exampledata/hichip.txt or cluster-hichip.txt)
- Gene symbol and enhancer symbol files (exampledata/gene.txt, exampledata/enhancer.txt)

## Outputs

- Per-gene combined Fisher p-values for each subpopulation
- Top-1000 combined-selection gene sets per subpopulation
- GO enrichment table comparing scRNA-seq+HiChIP vs scRNA-seq-only selection as -log10(p-values), e.g. go_enrichment_comparison.csv

## How to apply

First compute a one-tailed t-test p-value per gene from the scRNA-seq expression matrix (e.g. exampledata/E.txt) for each subpopulation cluster. For HiChIP, normalize total loop counts to N=10 for each enhancer-promoter interaction and run a one-tailed binomial test (R binom.test style: nk successes against expected proportion pk=1/K, alternative='greater'); for genes with multiple interactions take the most significant p-value. Combine each gene's scRNA-seq and HiChIP p-values with Fisher's method (sum of -2*ln(p) compared against chi-squared with 4 degrees of freedom) and select the top 1000 genes with the smallest combined p-values per subpopulation. Perform GO enrichment on these combined-selection gene sets and, as a control, on the top 1000 genes from scRNA-seq alone, then compare -log10(p-values) for representative terms (neuron development, axonogenesis, forebrain development, brain development, cardiovascular system development).

## Related tools

- **DC3** (Produces the coupled clustering (scRNA-result.txt) and the subpopulation/peak-gene association files whose gene and peak p-values are combined here) — github.com/SUwonglab/DC3
- **scipy** (One-tailed t-tests and binomial tests per gene, and the chi-squared survival function used to compute Fisher-combined p-values)
- **pandas** (Tabulates -log10(p-values) per subpopulation and writes the go_enrichment_comparison.csv comparison table)

## Examples

```
python dc3.py -k 2 -E exampledata/E.txt -O exampledata/O.txt -G_symbol exampledata/gene.txt -E_symbol exampledata/enhancer.txt -hichip exampledata/hichip.txt -lambda1 10000 -lambda2 0.0005
```

## Evaluation signals

- -log10(p-values) for scRNA-seq+HiChIP gene selection must exceed those for scRNA-seq-only selection across the tested GO terms, reproducing the Table 2 improvement pattern (e.g. forebrain development rising from 1.44 to 13.03).
- The improvement should be largest at low sequencing depth and persist at all tested sequencing depths.
- Per gene, the combined p-value must be no larger than the minimum of the two individual p-values (a Fisher's-method invariant).
- Selected gene sets must contain exactly the top 1000 genes with the smallest combined p-values per subpopulation.
- Output CSV must have one -log10(p) value per (subpopulation, GO term, selection method) triple with no missing representative terms.

## Limitations

- HiChIP loop counts must first be restricted to genes present in the scRNA-seq data and enhancers present in the scATAC-seq data, so interactions outside these sets contribute nothing.
- Genes with multiple enhancer-promoter interactions are collapsed to a single best binomial p-value, discarding the other interaction evidence.
- Results depend on the DC3 clustering (parameter k and lambda hyper-parameters); a poor clustering degrades the subpopulation-specific gene sets.

## Evidence

- [readme] cluster-specific-peaks-genes-pairs.txt  the cluster-specific peak-gene pairs. First column is the gene name, second column is the peak name, the third column is the p-value for gene and last column is the p-value for peak.: "the cluster-specific peak-gene pairs. First column is the gene name, second column is the peak name, the third column is the p-value for gene and last column is the p-value for peak."
- [abstract] DC3 can simultaneously identify distinct subpopulations, assign single cells to the subpopulations (i.e., clustering) and de-convolve the bulk data into subpopulation-specific data: "DC3 can simultaneously identify distinct subpopulations, assign single cells to the subpopulations (i.e., clustering) and de-convolve the bulk data into subpopulation-specific data"
- [readme] please restrict HiChIP enhancer-promoter interactions in the genes in scRNA-seq data and enhancers in scATAC-seq data.: "please restrict HiChIP enhancer-promoter interactions in the genes in scRNA-seq data and enhancers in scATAC-seq data."
- [intro] here we introduce DC3 as a method for the joint analysis of bulk and single cell data under various settings of input data combinations: "here we introduce DC3 as a method for the joint analysis of bulk and single cell data under various settings of input data combinations"
