---
name: causal-controller-deterministic-inference
description: Use when a trained GRouNdGAN checkpoint (.pth) and an imposed GRN are
  available and you need to test whether knocking out a regulating TF significantly
  changes its target gene's expression in simulated cells (interventional sampling,
  not observational generation).
when_to_use_negative:
- You want realistic observational simulated cells or data-quality evaluation — use
  the --generate and --evaluate commands instead of --perturb.
- Your GRN is multi-layer or includes TF-gene-phenotype links — the current GRouNdGAN
  version only supports imposing a bipartite TF-gene GRN.
- You want to predict real biological TF knockout outcomes in the reference dataset
  — GRouNdGAN is a generative model, not a predictive one, and such prediction would
  require a high-quality GRN plus data modalities like scATAC-seq or TF ChIP-seq.
edam_operation: http://edamontology.org/operation_0242
edam_topics:
- http://edamontology.org/topic_0085
tools:
- name: GRouNdGAN
  role: Causal implicit generative model whose deterministic causal controller samples
    interventional (TF-knockout) distributions via the --perturb command using a trained
    checkpoint
  repo: https://github.com/Emad-COMBINE-lab/GRouNdGAN
- name: GRNBoost2
  role: Infers the TF-gene GRN from preprocessed data (--create_grn) that is imposed
    on the model and defines the edges tested in the knockout significance analysis
  repo: https://github.com/Emad-COMBINE-lab/GRouNdGAN
- name: yazdanz/groundgan:4b98686 (Docker image)
  role: Containerized GRouNdGAN environment from which the perturbation and testing
    workflow is run
  repo: https://hub.docker.com/r/yazdanz/groundgan
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-48516-6
    title: 'GRouNdGAN: GRN-guided simulation of single-cell RNA-seq data using causal
      generative adversarial networks'
  source_content_hash: sha256:10a8ad151ef530a1de2b2faa2ceeb652dac942113410bc245cfc5a5f9bb27e3c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Emad-COMBINE-lab/GRouNdGAN
  tools:
  - GRNBoost2
  - GRouNdGAN
  - yazdanz/groundgan:4b98686 (Docker image)
  edam_operation: http://edamontology.org/operation_0242
  edam_topics:
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# causal-controller-deterministic-inference

## Summary

Leverages GRouNdGAN's causal controller in deterministic mode to sample from interventional distributions, performing in silico single-TF knockout experiments on GRouNdGAN-simulated PBMC-CTL cells and quantifying the effect of each knockout on its regulated genes with adjusted Wilcoxon tests over TF-gene edges.

## When to use

Use when a trained GRouNdGAN checkpoint (.pth) and an imposed GRN are available and you need to test whether knocking out a regulating TF significantly changes its target gene's expression in simulated cells (interventional sampling, not observational generation).

## When NOT to use

- You want realistic observational simulated cells or data-quality evaluation — use the --generate and --evaluate commands instead of --perturb.
- Your GRN is multi-layer or includes TF-gene-phenotype links — the current GRouNdGAN version only supports imposing a bipartite TF-gene GRN.
- You want to predict real biological TF knockout outcomes in the reference dataset — GRouNdGAN is a generative model, not a predictive one, and such prediction would require a high-quality GRN plus data modalities like scATAC-seq or TF ChIP-seq.

## Inputs

- Trained GRouNdGAN model checkpoint file (e.g., step_1000000.pth) referenced by [EXPERIMENT]/checkpoint in the config file
- Imposed ground truth GRN (bipartite TF-gene graph) used for training (e.g., from GRNBoost2 or PPCOR)
- GRouNdGAN config file with [Perturbation]/tfs and [Perturbation]/perturbation entries

## Outputs

- Per-edge knockout significance table (knockout_edge_significance.csv) with Wilcoxon p-values and BH-adjusted p-values per TF-gene edge
- Fraction of significantly perturbed TF-gene edges (adjusted p < 0.05) per imposed GRN

## How to apply

Load the trained GRouNdGAN checkpoint (e.g., step_1000000.pth) specified via [EXPERIMENT]/checkpoint in the config file for each imposed-GRN model (e.g., GRNBoost2-imposed and PPCOR-imposed). Set the causal controller to deterministic mode and, for one TF at a time, run the --perturb command with [Perturbation]/tfs naming the TF and [Perturbation]/perturbation values set to 0, sampling perturbed and unperturbed cells from the same fixed batch so perturbed/control cells are matched for a case-control comparison. For each knocked-out TF, run a two-sided Wilcoxon signed-rank test comparing matched perturbed vs. control expression for each gene it regulates in the imposed GRN. Adjust all TF-gene edge p-values with the Benjamini-Hochberg FDR procedure and call edges significant at adjusted p < 0.05. The rationale is that GRouNdGAN's causal structure enables sampling from interventional distributions, so a gene whose expression shifts under its TF's knockout evidences that the imposed GRN was causally respected in the simulation.

## Related tools

- **GRouNdGAN** (Causal implicit generative model whose deterministic causal controller samples interventional (TF-knockout) distributions via the --perturb command using a trained checkpoint) — https://github.com/Emad-COMBINE-lab/GRouNdGAN
- **GRNBoost2** (Infers the TF-gene GRN from preprocessed data (--create_grn) that is imposed on the model and defines the edges tested in the knockout significance analysis) — https://github.com/Emad-COMBINE-lab/GRouNdGAN
- **yazdanz/groundgan:4b98686 (Docker image)** (Containerized GRouNdGAN environment from which the perturbation and testing workflow is run) — https://hub.docker.com/r/yazdanz/groundgan

## Examples

```
python main.py --perturb -c config.yaml  # [EXPERIMENT]/checkpoint=step_1000000.pth; [Perturbation]/tfs=<TF>; [Perturbation]/perturbation values=0
```

## Evaluation signals

- The fraction of TF-gene edges with BH-adjusted p < 0.05 must reproduce the reported values: 66.5% for the GRNBoost2-imposed GRN and 75.2% for the PPCOR-imposed GRN (within small numerical tolerance).
- Per-edge results are written to knockout_edge_significance.csv containing, for every edge in the imposed GRN, the two-sided Wilcoxon signed-rank p-value and its BH-adjusted value.
- Perturbed and unperturbed cells must be drawn from the same fixed batch (matched case-control), so sample sizes of paired perturbed/control cells are equal for every Wilcoxon test.
- Only genes directly regulated by the knocked-out TF in the imposed GRN are tested — the tested edge count per TF equals its out-degree in the imposed bipartite GRN.
- Perturbation values in the config ([Perturbation]/perturbation) are set to 0, i.e., complete knockout, not partial knockdown.

## Limitations

- GRouNdGAN only generates library-size normalized expression values across the considered genes and TFs; unnormalized count generation is not supported yet.
- Given an observational dataset, its underlying causal graph is not necessarily unique, and verifying that the imposed GRN is the most probable one is beyond the study's scope.
- The current version only supports imposing a bipartite GRN; multi-layer GRNs and TF-gene-phenotype causal graphs are not supported.
- GRouNdGAN is a generative, not predictive, model — the model must be retrained for a new dataset, and hyperparameters may need re-tuning via a validation set for datasets with very different gene/TF counts.
- Resemblance of the simulated data to the reference dataset deteriorates if the imposed TF-gene relationships differ significantly from patterns consistent with the reference data.

## Evidence

- [methods] Perturbation studies command: "--perturb        Perform a perturbation experiment using a trained GRouNdGAN model"
- [methods] Interventional sampling capability: "Once trained, GRouNdGAN can sample from interventional distributions and perform in-silico TF perturbation experiments."
- [intro] Causal structure rationale: "the causal structure of GRouNdGAN enables it to be used for sampling from both interventional and observational data distributions, enabling in silico knockout experiments."
- [abstract] Knockout purpose: "GRouNdGAN can synthesize cells under new conditions to perform in silico TF knockout experiments."
- [methods] Checkpoint requirement for perturbation: "Once training is done, populate the ``[EXPERIMENT]/checkpoint`` field with the path of the ``.pth`` checkpoint"
