---
name: tf-database-candidate-selection
description: Use when fitting TFvelo gene dynamics or any per-gene model that needs
  a priori candidate TF regulators per target gene from ENCODE/ChEA, with a capped
  number of TFs per gene.
when_to_use_negative:
- The dataset already contains curated, experimentally validated TF-gene regulatory
  edges and database-derived candidates would add noise
- The analysis goal is splicing-based velocity (scVelo) rather than TF-expression-based
  velocity, where TF-target priors are not part of the model
- The target gene has no TF hits in ENCODE or ChEA, so no regulatory prior is available
  for initialization
edam_topics:
- http://edamontology.org/topic_0204
tools:
- name: TFvelo
  role: Implements the candidate TF selection from ENCODE/ChEA, correlation-based
    weight initialization, and the generalized EM refit of weights and dynamics
  repo: github.com/xiaoyeye/TFvelo
- name: ScVelo
  role: Framework on which TFvelo is built; its EM-based dynamical model code base
    hosts the TF selection and fitting routines
  repo: theislab/scvelo
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-45661-w
    title: 'TFvelo: gene regulation inspired RNA velocity estimation'
  source_content_hash: sha256:0989242a8e92e1ae1c1dbb018dcc89997293ad1c3cac5c51a25dbf9b1d57e96c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ScVelo
  - TFvelo
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0204
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# tf-database-candidate-selection

## Summary

Selects candidate transcription factors for each target gene from curated TF-target databases (ENCODE and ChEA by default in TFvelo), then initializes TF weights for gene-dynamics fitting. This links per-gene velocity models to known regulation instead of treating genes independently.

## When to use

Use when fitting TFvelo gene dynamics or any per-gene model that needs a priori candidate TF regulators per target gene from ENCODE/ChEA, with a capped number of TFs per gene.

## When NOT to use

- The dataset already contains curated, experimentally validated TF-gene regulatory edges and database-derived candidates would add noise
- The analysis goal is splicing-based velocity (scVelo) rather than TF-expression-based velocity, where TF-target priors are not part of the model
- The target gene has no TF hits in ENCODE or ChEA, so no regulatory prior is available for initialization

## Inputs

- Preprocessed scRNA-seq AnnData object (e.g. pancreas dataset auto-downloaded by TFvelo_run_demo.py)
- TF-target databases: ENCODE (requires ENCODE.zip decompression) and ChEA
- Hyperparameters: max_n_TF, TF_databases, init_weight_method, WX_method, WX_thres

## Outputs

- Per-gene candidate TF list (up to max_n_TF per gene)
- Per-gene initialized TF weight matrix W (Spearman correlation-based)
- Fitted model stored in h5ad format, e.g. 'TFvelo_pancreas/rc.h5ad', containing fitted per-gene dynamics and per-cell latent times

## How to apply

For each target gene, query the ENCODE and ChEA TF-target databases (the ENCODE archive must be decompressed, e.g. 'unzip ENCODE.zip', before use) to gather candidate TFs. Cap the candidate list at --max_n_TF TFs per gene so the per-gene model stays tractable. Initialize the TF weight vector W with --init_weight_method Correlation, i.e. Spearman correlation between TF and target expression across cells; initialize gamma from the steady-state estimate and set multiple starting values for theta. During optimization, the weights are refined in the M step by bounded linear regression (lsq_linear, trust-region reflective) with weights clipped to the (-20, 20) range enforced by --WX_thres. Rationale: encoding regulatory priors lets TFvelo model the time derivative of RNA abundance from TF expression rather than splicing information, enabling application to datasets without unspliced/spliced counts.

## Related tools

- **TFvelo** (Implements the candidate TF selection from ENCODE/ChEA, correlation-based weight initialization, and the generalized EM refit of weights and dynamics) — github.com/xiaoyeye/TFvelo
- **ScVelo** (Framework on which TFvelo is built; its EM-based dynamical model code base hosts the TF selection and fitting routines) — theislab/scvelo

## Examples

```
python TFvelo_run_demo.py --dataset_name pancreas
```

## Evaluation signals

- Each gene's candidate TF list contains no more than max_n_TF entries sourced from ENCODE/ChEA
- Initialized weights equal Spearman correlations between candidate TF and target expression (init_weight_method = Correlation), and fitted weights lie within the (-20, 20) WX_thres bound
- The run completes max_iter generalized EM iterations and writes 'TFvelo_pancreas/rc.h5ad' with per-gene dynamics and per-cell latent times
- Phase portraits for best-fit genes show the desired clockwise curve between the TF representation and the target

## Limitations

- The linear model for estimating TF weights may be insufficient; the authors note 'more complex models may be explored to replace the current linear model in TFvelo for estimating the weight of each TFs'
- TFvelo cannot fit the dynamics of some genes well (Fig. S14), so candidate selection does not guarantee a good phase portrait fit for every gene
- TFvelo is slower than scVelo; parallelizing over more CPUs is needed to reduce runtime on large-scale datasets

## Evidence

- [methods] TF selection via ENCODE/ChEA with per-gene cap: "--TF_databases: The way to select candidate TFs. Use ENCODE and ChEA by default."
- [readme] max_n_TF cap per gene: "--max_n_TF: Max number of TFs used for modeling each gene."
- [readme] Correlation-based weight initialization: "--init_weight_method: The method to initialize the weights. Correlation is adopted by default."
- [abstract] Rationale: regulation-based modeling without splicing: "we propose TFvelo, which expands RNA velocity concept to various single-cell datasets without relying on splicing information"
- [discussion] Modeling basis: TF expression drives target dynamics: "TFvelo models the dynamics of a target gene with the expression level of TFs and the target gene itself"
