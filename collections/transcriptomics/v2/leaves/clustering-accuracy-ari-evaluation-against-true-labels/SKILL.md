---
name: clustering-accuracy-ari-evaluation-against-true-labels
description: Use when a simulation or labeled benchmark provides known cell-type labels
  and you must quantify how accurately a clustering (e.g., BUSseq celltypes()) recovers
  them, e.g., expecting ARI=1.0 at the true K.
when_to_use_negative:
- The input is real experimental data without any ground-truth cell-type labels —
  ARI against true labels is undefined and BIC alone must drive model selection
- The batches and cell types are completely confounded (e.g., batch 1 has cell types
  1–2, batch 2 has cell types 3–4), where the article states 'no method is applicable'
  and clustering accuracy cannot be meaningfully assessed
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3308
tools:
- name: BUSseq
  role: Fits the Bayesian hierarchical model via BUSseq_MCMC, provides BIC_BUSseq
    for K selection and celltypes() for extracting inferred cell-type assignments
    used in the ARI computation
  repo: https://github.com/songfd2018/BUSseq-Rpackage
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-16905-2
    title: Flexible experimental designs for valid single-cell RNA-sequencing experiments
      allowing batch effects correction
  source_content_hash: sha256:d4793806a7038a52cfdb6e929f347cc24fc9570fa037d843486e0826c07f2b2e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/songfd2018/BUSseq-Rpackage
  tools:
  - BUSseq
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# clustering-accuracy-ari-evaluation-against-true-labels

## Summary

Validates inferred cell-type assignments from a clustering method (here BUSseq on simulated scRNA-seq data) by comparing them to ground-truth labels with the adjusted Rand index, alongside BIC-based selection of the number of cell types K.

## When to use

Use when a simulation or labeled benchmark provides known cell-type labels and you must quantify how accurately a clustering (e.g., BUSseq celltypes()) recovers them, e.g., expecting ARI=1.0 at the true K.

## When NOT to use

- The input is real experimental data without any ground-truth cell-type labels — ARI against true labels is undefined and BIC alone must drive model selection
- The batches and cell types are completely confounded (e.g., batch 1 has cell types 1–2, batch 2 has cell types 3–4), where the article states 'no method is applicable' and clustering accuracy cannot be meaningfully assessed

## Inputs

- Simulated or labeled scRNA-seq read count matrix (e.g., 4 batches of 300/300/200/200 cells, 5 cell types, 3000 genes under the chain-type design)
- True cell-type label vector for each cell (from the simulation design)
- BUSseq fits (BUSseqfit objects) from BUSseq_MCMC runs at each candidate K

## Outputs

- BIC-per-K table (from BIC_BUSseq) used to select the optimal number of cell types
- Adjusted Rand index comparing celltypes() assignments to true labels (expected 1.0)
- busseq_simulation_report.csv containing the BIC table and ARI value

## How to apply

Fit the model over a range of cell-type numbers — in the BUSseq vignette workflow, n.celltypes is enumerated from 3 to 6 with BUSseq_MCMC running 4000 iterations (n.cores parallelized) — and compute BIC_BUSseq for each K, selecting the K with the minimum BIC (expected minimum at the true K=5). Then extract inferred assignments via celltypes() and compute the adjusted Rand index against the simulated true labels; in the paper's 4-batch, 5-cell-type chain-type simulation (300/300/200/200 cells, 3000 genes, dropout rates 26.79–31.29%), BUSseq recovers ARI=1.0, confirming perfect clustering. The ARI of 1.0 together with correct BIC selection of K constitutes the evidence that the method accurately learns structure, not just parameters.

## Related tools

- **BUSseq** (Fits the Bayesian hierarchical model via BUSseq_MCMC, provides BIC_BUSseq for K selection and celltypes() for extracting inferred cell-type assignments used in the ARI computation) — https://github.com/songfd2018/BUSseq-Rpackage

## Examples

```
fit <- BUSseq_MCMC(ObservedData, n.celltypes=3:6, n.iterations=4000, n.cores=4); bic <- BIC_BUSseq(fit); ari <- mclust::adjustedRandIndex(celltypes(fit), true_labels)
```

## Evaluation signals

- BIC per K shows the minimum exactly at the true number of cell types (K=5 in the 4-batch, 5-cell-type simulation)
- Adjusted Rand index between celltypes() output and true simulated labels equals 1.0 (perfect clustering)
- The ARI is computed over all 1000 simulated cells across the four batches with sizes 300/300/200/200
- Results (BIC table per K and ARI value) are written to busseq_simulation_report.csv without missing fields
- MCMC settings match the reference workflow: 4000 iterations per K with K enumerated from 3 to 6

## Limitations

- ARI=1.0 is only achievable on simulated data with known labels; on real data no ground truth exists and this validation is infeasible
- Completely confounded designs (each cell type confined to one batch) admit no applicable method, so clustering accuracy cannot be evaluated
- MCMC inference is computationally heavy; enumerating K from 3 to 6 with 4000 iterations each requires parallelization (n.cores), and most steps of the MCMC algorithm are parallelizable per the article

## Evidence

- [results] On the simulated 4-batch, 5-cell-type dataset, BUSseq correctly identifies the presence of five cell types among the cells, confirming that its BIC-based selection recovers the true number of cell types.: [evidence span withheld — anchor status: not_found]
- [methods] To select the optimal K, we need to compare the BIC values for different Ks. In the vignettes, we enumerate K from 3 to 6.: "To select the optimal *K*, we need to compare the BIC values for different *K*s. In the vignettes, we enumerate *K* from 3 to 6."
- [methods] BUSseq is able to simultaneously correct batch effects, clusters cell types: "BUSseq is able to simultaneously correct batch effects, clusters cell types"
- [readme] For general cases, you can change the working directory and run ... BUSseq -d./ -r/your/count/data/directory/ -p [your study name] -v [version] -K [the number of cell types]: [evidence span withheld — anchor status: external_doc]
