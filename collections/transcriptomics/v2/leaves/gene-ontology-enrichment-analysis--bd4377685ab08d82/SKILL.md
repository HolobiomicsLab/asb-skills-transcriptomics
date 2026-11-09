---
name: gene-ontology-enrichment-analysis
description: Use when comparing functional enrichment of subpopulation gene sets derived
  from scRNA-seq alone versus scRNA-seq combined with bulk HiChIP loop data, to quantify
  whether loop information improves GO enrichment (Table 2).
when_to_use_negative:
- No HiChIP enhancer-promoter interaction data exist for the population, so only the
  scRNA-seq-only arm of the comparison is possible
- Input is already a table of enrichment p-values rather than gene-level p-values
  requiring selection of top-1000 gene sets
- The analysis goal is subpopulation discovery or deconvolution itself rather than
  post-hoc functional enrichment of DC3-inferred subpopulations
edam_operation: http://edamontology.org/operation_3672
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: DC3
  role: Produces the coupled clustering assignments and cluster-specific peak-gene
    pairs whose gene-level p-values feed the top-1000 gene sets used for GO enrichment
  repo: https://github.com/SUwonglab/DC3
- name: scipy
  role: Runs the one-tailed binomial tests on HiChIP loop counts and Fisher's-method
    combination of scRNA-seq and HiChIP p-values
- name: pandas
  role: Tabulates -log10(p) enrichment values per subpopulation and writes go_enrichment_comparison.csv
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-12547-1
    title: DC3 is a method for deconvolution and coupled clustering from bulk and
      single-cell genomics data
  source_content_hash: sha256:cab3a8f59405d09b732141a7eb69f653b1977a620068e409bbe8599ff2a40820
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SUwonglab/DC3
  tools:
  - Anaconda
  - DC3
  - GO
  - PyWGCNA
  - Python
  - VASC
  - h5py
  - numpy
  - pandas
  - scipy
  - sklearn
  edam_operation: http://edamontology.org/operation_3672
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# gene-ontology-enrichment-analysis

## Summary

Performs GO term enrichment on subpopulation-specific gene sets selected from scRNA-seq data, either alone or after coupling with HiChIP enhancer-promoter p-values via Fisher's method, and tabulates -log10(p) values to quantify whether HiChIP loop information improves subpopulation characterization.

## When to use

Use when comparing functional enrichment of subpopulation gene sets derived from scRNA-seq alone versus scRNA-seq combined with bulk HiChIP loop data, to quantify whether loop information improves GO enrichment (Table 2).

## When NOT to use

- No HiChIP enhancer-promoter interaction data exist for the population, so only the scRNA-seq-only arm of the comparison is possible
- Input is already a table of enrichment p-values rather than gene-level p-values requiring selection of top-1000 gene sets
- The analysis goal is subpopulation discovery or deconvolution itself rather than post-hoc functional enrichment of DC3-inferred subpopulations

## Inputs

- scRNA-seq expression matrix (genes x cells, e.g. exampledata/E.txt)
- subpopulation cluster assignments (scRNA-result.txt from DC3 coupled clustering)
- HiChIP enhancer-promoter interaction table (gene index, enhancer index, loop counts)
- cluster-specific peak-gene pairs file (cluster-specific-peaks-genes-pairs.txt)

## Outputs

- Top-1000 gene sets per subpopulation (combined scRNA-seq+HiChIP selection and scRNA-seq-only selection)
- GO enrichment p-values per subpopulation for representative terms
- go_enrichment_comparison.csv with -log10(p) values per subpopulation and GO term

## How to apply

Run one-tailed t-tests per gene on the scRNA-seq expression matrix (e.g. exampledata/E.txt) with cluster assignments to get subpopulation-specific p-values; normalize total loop counts to N=10 per HiChIP enhancer-promoter interaction and run a one-tailed binomial test (nk successes, expected pk=1/K, alternative='greater'), keeping the most significant p-value per gene when a gene has multiple interactions. Combine scRNA-seq and HiChIP p-values per gene with Fisher's method and select the top 1000 genes by smallest combined p-value per subpopulation; separately select the top 1000 genes using scRNA-seq p-values alone as the control. Run GO enrichment on both gene-set types, transform enrichment p-values to -log10(p), and tabulate representative terms (neuron development, axonogenesis, cardiovascular system development, forebrain development, brain development) per subpopulation into go_enrichment_comparison.csv. The rationale is that DC3-derived cluster-specific peak-gene pairs link active regulatory elements to target genes, so coupling loop evidence sharpens the gene sets used for enrichment.

## Related tools

- **DC3** (Produces the coupled clustering assignments and cluster-specific peak-gene pairs whose gene-level p-values feed the top-1000 gene sets used for GO enrichment) — https://github.com/SUwonglab/DC3
- **scipy** (Runs the one-tailed binomial tests on HiChIP loop counts and Fisher's-method combination of scRNA-seq and HiChIP p-values)
- **pandas** (Tabulates -log10(p) enrichment values per subpopulation and writes go_enrichment_comparison.csv)

## Examples

```
from scipy.stats import combine_pvalues; stat, p_combined = combine_pvalues([p_rna, p_hichip], method='fisher')
```

## Evaluation signals

- The -log10(p) values for scRNA-seq+HiChIP selection exceed those for scRNA-seq-only selection across all tested GO terms and subpopulations
- Forebrain development enrichment rises from -log10(p)=1.44 (scRNA-seq alone) to 13.03 (scRNA-seq+HiChIP) at low depth
- Each selected gene set contains exactly the top 1000 genes ranked by the smallest combined Fisher p-value per subpopulation
- HiChIP binomial tests use total loop counts normalized to N=10 with expected proportion pk=1/K and alternative='greater'
- go_enrichment_comparison.csv contains rows for the representative terms (neuron development, axonogenesis, cardiovascular system development, forebrain development, brain development) per subpopulation

## Limitations

- The improvement is largest at low sequencing depths; gains at high depth may be smaller
- Genes with multiple HiChIP interactions collapse to their single most significant p-value, discarding per-interaction information
- The comparison depends on DC3's coupled clustering quality; the README notes hyperparameters lambda1 and lambda2 may need manual tuning (e.g. -lambda1 10000 -lambda2 0.0005) if automatic selection fails

## Evidence

- [abstract] DC3 jointly analyzes bulk and single-cell data: [evidence span withheld — anchor status: not_found]
- [abstract] Subpopulation profiles from DC3: "The subpopulation-specific profiles of gene expression, chromatin accessibility and enhancer-promoter contact obtained by DC3 provide a comprehensive characterization"
- [discussion] Loop profiles as analysis foundation: "the accessibility, expression and loop profiles inferred by DC3 can serve as a foundation for further analyses of the regulatory systems, such as the extraction of core subnetworks"
- [readme] Cluster-specific peak-gene pairs output format: "cluster-specific-peaks-genes-pairs.txt  the cluster-specific peak-gene pairs. First column is the gene name, second column is the peak name, the third column is the p-value"
- [intro] Linking active REs to target genes: "even with these two profiles, our understanding of the subpopulation-specific regulatory networks remains incomplete if we cannot link the active REs to their target genes"
