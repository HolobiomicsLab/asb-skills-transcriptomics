---
name: subpopulation-specific-gene-ranking
description: Use when you have cluster assignments plus per-gene subpopulation scRNA-seq
  p-values and cluster-specific HiChIP loop counts, and need a per-subpopulation ranked
  gene list or GO enrichment characterization.
when_to_use_negative:
- No bulk HiChIP or enhancer-promoter interaction data are available — Fisher combination
  degenerates to the scRNA-seq-only t-test ranking
- Cells have not been assigned to subpopulations (no cluster assignments or K estimate),
  since the binomial null pk=1/K and per-cluster t-tests both require cluster membership
- The goal is deconvolution of the bulk profiles or coupled clustering itself, rather
  than ranking genes within already-defined subpopulations
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3308
tools:
- name: DC3
  role: produces the clustering results (scRNA-result.txt) and cluster-specific HiChIP
    / peak-gene pairs with gene and peak p-values that feed the ranking
  repo: github.com/SUwonglab/DC3
- name: scipy
  role: one-tailed t-tests per gene and Fisher's-method p-value combination for the
    gene ranking
- name: pandas
  role: tabulation of -log10(p) enrichment values per subpopulation into the comparison
    CSV
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-12547-1
    title: DC3 is a method for deconvolution and coupled clustering from bulk and
      single-cell genomics data
  source_content_hash: sha256:afdcae86aeb657429012ddfeaeba6749533e20163e1714ebe294044979f4728b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DC3
  - Python
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# subpopulation-specific gene ranking

## Summary

Ranks genes per cell subpopulation by statistically combining subpopulation-specific scRNA-seq t-test p-values with HiChIP enhancer-promoter binomial-test p-values (Fisher's method), selecting the top 1000 genes for downstream GO enrichment comparison against scRNA-seq-only selection.

## When to use

Use when you have cluster assignments plus per-gene subpopulation scRNA-seq p-values and cluster-specific HiChIP loop counts, and need a per-subpopulation ranked gene list or GO enrichment characterization.

## When NOT to use

- No bulk HiChIP or enhancer-promoter interaction data are available — Fisher combination degenerates to the scRNA-seq-only t-test ranking
- Cells have not been assigned to subpopulations (no cluster assignments or K estimate), since the binomial null pk=1/K and per-cluster t-tests both require cluster membership
- The goal is deconvolution of the bulk profiles or coupled clustering itself, rather than ranking genes within already-defined subpopulations

## Inputs

- scRNA-seq expression matrix (genes x cells, e.g., exampledata/E.txt)
- subpopulation cluster assignments (e.g., scRNA-result.txt from DC3)
- cluster-specific HiChIP enhancer-promoter interaction counts (cluster-hichip.txt; gene index, enhancer index, loop counts)
- number of subpopulations K (used as expected binomial proportion pk=1/K)

## Outputs

- Per-subpopulation ranked gene list (top 1000 genes by smallest Fisher-combined p-value)
- Per-gene scRNA-seq t-test p-values and HiChIP binomial-test p-values (cf. cluster-specific-peaks-genes-pairs.txt)
- GO enrichment -log10(p) table comparing scRNA-seq+HiChIP vs scRNA-seq-only selection (e.g., go_enrichment_comparison.csv)

## How to apply

Apply a one-tailed t-test per gene on the scRNA-seq expression matrix (e.g., exampledata/E.txt) grouped by subpopulation cluster assignment to get subpopulation-specific p-values. For HiChIP, normalize total loop counts to N=10 per enhancer-promoter interaction and run a one-tailed binomial test (R binom.test style: nk successes, expected proportion pk=1/K, alternative='greater'); when a gene maps to multiple interactions keep the smallest p-value. Combine each gene's scRNA-seq and HiChIP p-values with Fisher's method and keep the top 1000 genes with the smallest combined p-value per subpopulation. Rank justification: the loop profile links active regulatory elements to target genes, so genes supported by both expression and subpopulation-specific enhancer-promoter contact better characterize the subpopulation. Evaluate by running GO enrichment on the combined-selection genes and on the top 1000 scRNA-seq-only genes, converting to -log10(p), and confirming the combined selection yields higher -log10(p) values (e.g., forebrain development rising from 1.44 to 13.03).

## Related tools

- **DC3** (produces the clustering results (scRNA-result.txt) and cluster-specific HiChIP / peak-gene pairs with gene and peak p-values that feed the ranking) — github.com/SUwonglab/DC3
- **scipy** (one-tailed t-tests per gene and Fisher's-method p-value combination for the gene ranking)
- **pandas** (tabulation of -log10(p) enrichment values per subpopulation into the comparison CSV)

## Examples

```
python dc3.py -k 2 -E exampledata/E.txt -O exampledata/O.txt -G_symbol exampledata/gene.txt -E_symbol exampledata/enhancer.txt -hichip exampledata/hichip.txt -lambda1 10000 -lambda2 0.0005
```

## Evaluation signals

- -log10(p) GO enrichment values for the scRNA-seq+HiChIP selection should exceed those of the scRNA-seq-only top-1000 selection across tested terms (Table 2 pattern), with largest gains at low sequencing depth
- For genes with multiple HiChIP interactions, verify only the most significant (smallest) binomial p-value is retained
- Each subpopulation's ranked list should contain exactly 1000 genes after selection
- Binomial tests should use expected proportion pk=1/K with alternative='greater' on loop counts normalized to N=10 per interaction
- Cluster-specific peak-gene pair output schema: gene name (col 1), peak name (col 2), gene p-value (col 3), peak p-value (col 4)

## Limitations

- Enrichment gains depend on HiChIP enhancer-promoter interactions being restricted to genes present in the scRNA-seq data and enhancers present in the scATAC-seq data
- Hyperparameters lambda1/lambda2 of the upstream DC3 NMF affect the cluster-specific loop profiles feeding the ranking; if unset, DC3 chooses them automatically
- Gains at high sequencing depth are smaller than at low depth, so benefit may be marginal on deeply sequenced data

## Evidence

- [readme] cluster-specific-peaks-genes-pairs.txt: "cluster-specific-peaks-genes-pairs.txt  the cluster-specific peak-gene pairs. First column is the gene name, second column is the peak name, the third column is the p-value"
- [readme] joint analysis rationale: "it is of considerable scientific significance to be able to deconvolve the bulk sample HiChIP signal (i.e., bulk sample loop counts) into subpopulation-specific HiChIP signals"
- [abstract] subpopulation profiles: "The subpopulation-specific profiles of gene expression, chromatin accessibility and enhancer-promoter contact obtained by DC3 provide a comprehensive characterization of the gene regulatory system"
- [abstract] clustering and deconvolution: "DC3 can simultaneously identify distinct subpopulations, assign single cells to the subpopulations (i.e., clustering) and de-convolve the bulk data into subpopulation-specific data"
- [intro] RE-to-gene linking motivation: "even with these two profiles, our understanding of the subpopulation-specific regulatory networks remains incomplete if we cannot link the active REs to their target genes"
