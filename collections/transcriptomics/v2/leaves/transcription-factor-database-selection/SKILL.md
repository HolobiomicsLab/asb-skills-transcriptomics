---
name: transcription-factor-database-selection
description: Use when running TFvelo on scRNA-seq data and you need to restrict the
  TF pool per target gene — e.g., preparing the gene regulatory input before weight
  initialization and velocity inference.
when_to_use_negative:
- You need splicing-based velocity from unspliced/spliced counts — use scVelo's dynamical
  model instead; TFvelo's TF selection targets datasets without splicing information.
- You intend to fit gene dynamics for targets with no TFs present in ENCODE/ChEA —
  TFvelo still cannot fit the dynamics of some genes well (Fig. S14 limitation).
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: TFvelo
  role: Consumes the ENCODE/ChEA candidate TF sets via --TF_databases and --max_n_TF
    to initialize per-gene TF weights (correlation init, lsq_linear optimization)
    for TF-based RNA velocity
  repo: https://github.com/xiaoyeye/TFvelo
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-45661-w
    title: 'TFvelo: gene regulation inspired RNA velocity estimation'
  source_content_hash: sha256:6395e5af3980fead195d248df458ad8b8714bfc7289c76bf64363d8cb2239a1e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xiaoyeye/TFvelo
  tools:
  - MultiVelo
  - TFvelo
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# transcription-factor-database-selection

## Summary

Selects candidate transcription factors (TFs) from curated TF-target databases (ENCODE and ChEA by default) to define which regulators TFvelo uses to model each target gene's RNA dynamics without splicing information.

## When to use

Use when running TFvelo on scRNA-seq data and you need to restrict the TF pool per target gene — e.g., preparing the gene regulatory input before weight initialization and velocity inference.

## When NOT to use

- You need splicing-based velocity from unspliced/spliced counts — use scVelo's dynamical model instead; TFvelo's TF selection targets datasets without splicing information.
- You intend to fit gene dynamics for targets with no TFs present in ENCODE/ChEA — TFvelo still cannot fit the dynamics of some genes well (Fig. S14 limitation).

## Inputs

- Decompressed ENCODE TF-target database file (shipped as ENCODE.zip in the TFvelo repository)
- ChEA TF-target database (loaded by default via --TF_databases)
- Preprocessed scRNA-seq AnnData object (.h5ad) of the dataset to be analyzed

## Outputs

- Candidate TF set per target gene used by TFvelo for weight initialization and velocity estimation
- rc.h5ad with fitted TF-weights, latent time, and RNA velocity per gene/cell

## How to apply

After installing TFvelo, first decompress the ENCODE TF-target database shipped with the package (`unzip ENCODE.zip`). Then rely on the `--TF_databases` parameter to choose candidate TFs; ENCODE and ChEA are used by default and reproduce the manuscript results, including the 10x multi-omics mouse brain comparison where TFvelo matched MultiVelo without ATAC-seq. Cap the number of regulators per target with `--max_n_TF`, since each target gene is modeled using at most that many TFs. TF weights are initialized by correlation (`--init_weight_method correlation`) and optimized with `--WX_method lsq_linear`, with `--WX_thres` bounding absolute weights. Decision point: deviating from the ENCODE+ChEA default changes the regulatory prior and risks non-reproducibility of published pseudotime and phase-portrait results.

## Related tools

- **TFvelo** (Consumes the ENCODE/ChEA candidate TF sets via --TF_databases and --max_n_TF to initialize per-gene TF weights (correlation init, lsq_linear optimization) for TF-based RNA velocity) — https://github.com/xiaoyeye/TFvelo

## Examples

```
unzip ENCODE.zip && python TFvelo_run_demo.py --dataset_name your_dataset_name
```

## Evaluation signals

- Running with default parameters (TF_databases ENCODE and ChEA, WX_method lsq_linear, init_weight_method correlation) reproduces the manuscript results, including matching MultiVelo's pseudotime and differentiation directions on the 10x multi-omics mouse brain dataset.
- The ENCODE database decompresses cleanly (`unzip ENCODE.zip`) and TFvelo proceeds past TF candidate loading without missing-database errors.
- Per-gene fitted dynamics show the expected clockwise curve on the phase portrait between the TFs representation and the target (e.g., AHI1, NTRK2, GRIN2B), rather than spurious cyclic dynamics like MultiVelo's AHI1 fit.
- Fitted weights respect --WX_thres bounds and each target uses at most --max_n_TF regulators.

## Limitations

- The current linear model for estimating TF weights may be insufficient; the article notes more complex models may be explored to replace the linear model.
- TF databases provide no accessions/identifiers in the article for the 10x multi-omics context, so dataset provenance must be secured separately.
- Default ENCODE+ChEA settings reproduce published results, but alternative database choices are unvalidated.

## Evidence

- [methods] methods: "--TF_databases: The way to select candidate TFs. Use ENCODE and ChEA by default."
- [readme] readme: "Please decompress the ENCODE TF-target database file firstly after downloading this package."
- [discussion] discussion: "TFvelo models the dynamics of a target gene with the expression level of TFs and the target gene itself"
