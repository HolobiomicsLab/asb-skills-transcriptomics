---
name: simulation-based-gene-recovery-validation
description: Use when you need to verify that a Bayesian-FDR intrinsic/differential
  gene selection procedure recovers known true DE genes in simulation and yields biologically
  interpretable enriched pathways on real data.
when_to_use_negative:
- Input is raw count data that has not yet been fitted with BUSseq_MCMC — the indicator
  probabilities xi_gk required for FDR ranking do not exist before model fitting
- The analysis goal is batch-effect correction or cell-type clustering rather than
  validating differential gene detection — use corrected_read_counts or cell-type
  clustering steps instead
- The experimental design is completely confounded (each batch measures disjoint cell
  types) — the paper states no method is applicable in that setting
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: BUSseq
  role: Fits the Bayesian hierarchical model and exposes intrinsic_genes_BUSseq()
    to extract posterior spike-and-slab indicator probabilities and Bayesian-FDR-ranked
    intrinsic genes
  repo: https://github.com/songfd2018/BUSseq
- name: SingleCellExperiment
  role: Container holding the fitted BUSseq results passed to intrinsic_genes_BUSseq()
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-16905-2
    title: Flexible experimental designs for valid single-cell RNA-sequencing experiments
      allowing batch effects correction
  source_content_hash: sha256:c0eac0d58b962e6712f80936c742715de214c3f8a272d9d60a2528ded3174d7c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/songfd2018/BUSseq
  tools:
  - BUSseq
  - SingleCellExperiment
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# simulation-based-gene-recovery-validation

## Summary

Validates a spike-and-slab Bayesian-FDR intrinsic gene detection procedure by running it on simulated scRNA-seq data with planted differentially expressed genes, checking recovery of the 500 true intrinsic genes at approximately 2% true FDR under a 5% Bayesian FDR cutoff, and on real data by confirming biologically meaningful KEGG enrichment.

## When to use

Use when you need to verify that a Bayesian-FDR intrinsic/differential gene selection procedure recovers known true DE genes in simulation and yields biologically interpretable enriched pathways on real data.

## When NOT to use

- Input is raw count data that has not yet been fitted with BUSseq_MCMC — the indicator probabilities xi_gk required for FDR ranking do not exist before model fitting
- The analysis goal is batch-effect correction or cell-type clustering rather than validating differential gene detection — use corrected_read_counts or cell-type clustering steps instead
- The experimental design is completely confounded (each batch measures disjoint cell types) — the paper states no method is applicable in that setting

## Inputs

- Fitted BUSseq model objects (e.g., BUSseqfits_example) for simulated and hematopoietic scRNA-seq data
- SingleCellExperiment container holding the BUSseq fit
- Simulated count data with 500 planted intrinsic (differentially expressed) genes and known truth labels
- Real read count matrices (e.g., mouse hematopoietic study raw counts)

## Outputs

- Set of intrinsic genes ranked by cumulative indicator mass kappa0 at Bayesian FDR < 0.05
- Estimated true FDR on simulated data (~2%) against the 500 planted intrinsic genes
- KEGG enrichment results (51 enriched pathways for the hematopoietic selection)
- Final gene table written to intrinsic_genes.csv

## How to apply

Fit the BUSseq Bayesian hierarchical model to simulated and real scRNA-seq count data via BUSseq_MCMC, then apply intrinsic_genes_BUSseq() to each fit to obtain posterior indicator probabilities xi_gk from the normal-mixture spike-and-slab prior on cell-type effects beta_gk with latent indicators L_gk. Rank genes by decreasing cumulative indicator mass kappa0 and select the largest set whose estimated Bayesian FDR stays below alpha = 0.05 (the fdr setting exposed in the BUSseq_MCMC signature). On the simulated dataset, compare the selected genes against the 500 planted intrinsic genes to estimate the true FDR, which should land around 2%; on the hematopoietic fit, confirm the selection returns 1,419 intrinsic genes. Perform KEGG enrichment on the real-data gene set to check that 51 pathways are enriched. Accept the implementation only if both the ~2% true FDR recovery in simulation and the 1,419-gene/51-pathway KEGG result on hematopoietic data are reproduced, then write the final gene table to intrinsic_genes.csv.

## Related tools

- **BUSseq** (Fits the Bayesian hierarchical model and exposes intrinsic_genes_BUSseq() to extract posterior spike-and-slab indicator probabilities and Bayesian-FDR-ranked intrinsic genes) — https://github.com/songfd2018/BUSseq
- **SingleCellExperiment** (Container holding the fitted BUSseq results passed to intrinsic_genes_BUSseq())

## Examples

```
intrinsic_genes <- intrinsic_genes_BUSseq(sce_BUSseqfit)
```

## Evaluation signals

- Simulated-data run recovers all 500 planted intrinsic genes with a true FDR of approximately 2% at the 5% Bayesian FDR cutoff
- Hematopoietic fit selection returns exactly 1,419 intrinsic genes
- KEGG enrichment on the hematopoietic gene set identifies 51 enriched pathways
- Each selected gene's cumulative indicator mass kappa0 threshold keeps the estimated Bayesian FDR below alpha = 0.05
- Final selected gene table is produced as intrinsic_genes.csv

## Limitations

- MCMC algorithms are known for heavy computation load, so the validation run can be expensive (though most MCMC steps are parallelizable)
- Multi-stage pipelines that ignore clustering uncertainty can produce false positives and false negatives in gene detection
- Fitting requires enumerating cell-type numbers K (e.g., K from 3 to 6) and comparing BIC to select the optimal K before gene detection

## Evidence

- [results] In the simulation study, controlling the Bayesian False Discovery Rate at 5% allowed BUSseq to identify all intrinsic genes that differentiate cell types, achieving a true FDR of 2%.: [evidence span withheld — anchor status: not_found]
- [methods] Intrinsic gene extraction function: "intrinsic_genes_BUSseq(sce_BUSseqfit)"
- [discussion] BUSseq intrinsic gene identification capability: "BUSseq simultaneously corrects batch effects, clusters cell types, imputes missing data, and identifies intrinsic genes that differentiate cell types."
- [methods] MCMC function signature including fdr parameter: "BUSseq_MCMC(ObservedData, n.celltypes, seed, n.cores, n.iterations, n.burnin, n.unchanged, n.output, working_dir, Drop_ind, fdr, hyper_pi, hyper_gamma0, hyper_gamma1, hyper_alpha, tau1sq, hyper_p,"
- [readme] BUSseq overview from README: "BUSseq is able to simultaneously correct batch effects, clusters cell types, and takes care of the count data nature, the overdispersion, the dropout events, and the cell-specific sequencing depth"
