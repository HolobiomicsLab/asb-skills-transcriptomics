---
name: in-silico-tf-knockout-simulation
description: Use when you have a trained GRouNdGAN checkpoint and an imposed (e.g.,
  GRNBoost2- or PPCOR-derived) bipartite TF-gene GRN, and you want to test whether
  knocking out each TF significantly alters its target genes in simulated cells.
when_to_use_negative:
- The goal is to predict real experimental TF knockout outcomes in biological cells
  — GRouNdGAN is a generative simulator, not a predictive model, and outcome prediction
  would require a high-quality GRN plus additional data modalities such as (sc)ATAC-seq
  or TF ChIP-seq
- The GRN contains multi-layer or TF-gene-phenotype edges — the current GRouNdGAN
  version only supports imposing a bipartite GRN
- The model checkpoint was trained on a different reference dataset — GRouNdGAN must
  be retrained for a new dataset rather than perturbed out of context
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: GRouNdGAN
  role: Causal implicit generative model whose causal controller samples from interventional
    distributions for in-silico TF knockout via the --perturb command
  repo: github.com/Emad-COMBINE-lab/GRouNdGAN
- name: GRNBoost2
  role: Infers the TF-gene edges from preprocessed data (--create_grn) that are imposed
    on the simulator and evaluated in the knockout test
- name: PPCOR
  role: Alternative GRN inference method used to derive a second imposed GRN for cross-checking
    knockout significance rates
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-48516-6
    title: 'GRouNdGAN: GRN-guided simulation of single-cell RNA-seq data using causal
      generative adversarial networks'
  source_content_hash: sha256:c3adecd49c36b1709c706a2603b3cfd5677c0b0ae6d0258c79b63e9babcd0fc7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GRNBoost2
  - GRouNdGAN
  - PPCOR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# in-silico-tf-knockout-simulation

## Summary

Use GRouNdGAN, a causal-implicit GAN, to sample from interventional distributions in which a transcription factor is knocked out, then quantify the effect of the knockout on each TF-gene edge in the imposed GRN via matched Wilcoxon testing with Benjamini-Hochberg adjustment. This validates that the imposed causal GRN is actually reflected in the simulated scRNA-seq cells.

## When to use

Use when you have a trained GRouNdGAN checkpoint and an imposed (e.g., GRNBoost2- or PPCOR-derived) bipartite TF-gene GRN, and you want to test whether knocking out each TF significantly alters its target genes in simulated cells.

## When NOT to use

- The goal is to predict real experimental TF knockout outcomes in biological cells — GRouNdGAN is a generative simulator, not a predictive model, and outcome prediction would require a high-quality GRN plus additional data modalities such as (sc)ATAC-seq or TF ChIP-seq
- The GRN contains multi-layer or TF-gene-phenotype edges — the current GRouNdGAN version only supports imposing a bipartite GRN
- The model checkpoint was trained on a different reference dataset — GRouNdGAN must be retrained for a new dataset rather than perturbed out of context

## Inputs

- Trained GRouNdGAN model checkpoint (.pth file, e.g., step_1000000.pth) referenced via [EXPERIMENT]/checkpoint
- Imposed bipartite TF-gene ground-truth GRN (e.g., GRNBoost2-inferred or PPCOR-inferred edges)
- GRouNdGAN config YAML with [Perturbation]/tfs and [Perturbation]/perturbation fields
- Reference scRNA-seq dataset (e.g., PBMC-CTL, CD8+ cytotoxic T-cells from Zheng et al. 2017)

## Outputs

- Per-edge significance table knockout_edge_significance.csv (TF, gene, Wilcoxon statistic, raw and BH-adjusted p-value)
- Fraction of significantly perturbed TF-gene edges per imposed GRN (reported 66.5% for GRNBoost2 GRN, 75.2% for PPCOR GRN on PBMC-CTL)
- Matched perturbed and unperturbed simulated single-cell expression matrices

## How to apply

Load the trained GRouNdGAN checkpoint (e.g., step_1000000.pth) specified via [EXPERIMENT]/checkpoint in the config file for both the GRNBoost2-imposed and PPCOR-imposed models. Set the causal controller to deterministic mode and, for each TF one at a time, run the --perturb command with [Perturbation]/tfs set to the TF and [Perturbation]/perturbation values set to 0 (knockout), sampling perturbed and unperturbed cells from the same fixed batch so perturbed-vs-control expression is matched case-control. For each knocked-out TF, apply a two-sided Wilcoxon signed-rank test comparing matched perturbed vs. control expression of each gene it regulates in the imposed GRN. Adjust Wilcoxon p-values across all TF-gene edges with Benjamini-Hochberg FDR and call edges significant at adjusted p < 0.05. Compute the fraction of significantly perturbed TF-gene edges per GRN and write per-edge results to knockout_edge_significance.csv; the fraction quantifies how much of the imposed GRN is causally realized in the simulated cells.

## Related tools

- **GRouNdGAN** (Causal implicit generative model whose causal controller samples from interventional distributions for in-silico TF knockout via the --perturb command) — github.com/Emad-COMBINE-lab/GRouNdGAN
- **GRNBoost2** (Infers the TF-gene edges from preprocessed data (--create_grn) that are imposed on the simulator and evaluated in the knockout test)
- **PPCOR** (Alternative GRN inference method used to derive a second imposed GRN for cross-checking knockout significance rates)

## Examples

```
python main.py --config config.yaml --perturb  # with [Perturbation]/tfs set per TF, [Perturbation]/perturbation = 0 (knockout), [EXPERIMENT]/checkpoint = step_1000000.pth
```

## Evaluation signals

- The fraction of significantly perturbed TF-gene edges (BH-adjusted p < 0.05) reproduces ~66.5% for the GRNBoost2-imposed model and ~75.2% for the PPCOR-imposed model on PBMC-CTL within small numerical tolerance
- Per-edge results are fully written to knockout_edge_significance.csv with one row per TF-gene edge in the imposed GRN, including adjusted p-values
- Perturbed and control cells are sampled from the same fixed batch so the Wilcoxon signed-rank test compares matched case-control pairs
- The causal controller runs in deterministic mode so knockout effects are attributable to the causal graph rather than sampling noise

## Limitations

- The current version of GRouNdGAN only allows imposing a bipartite GRN and does not support a multi-layer GRN
- The model generates expression values library-size normalized across the considered genes and TFs; unnormalized count generation is not yet supported
- The resemblance of the simulated data to the reference dataset deteriorates if the imposed TF-gene relationships differ significantly from patterns in the reference data
- GRouNdGAN is a generative, not predictive, model — given a new dataset, the model needs to be retrained before knockouts are meaningful

## Evidence

- [methods] In-silico perturbation experiments: "Once trained, GRouNdGAN can sample from interventional distributions and perform in-silico TF perturbation experiments."
- [methods] --perturb command: "--perturb        Perform a perturbation experiment using a trained GRouNdGAN model"
- [abstract] Interventional sampling capability: "GRouNdGAN can synthesize cells under new conditions to perform in silico TF knockout experiments."
- [discussion] Bipartite GRN restriction: "The current version of GRouNdGAN only allows imposing a bipartite GRN and does not support a multi-layer GRN"
- [intro] Interventional and observational distributions: "the causal structure of GRouNdGAN enables it to be used for sampling from both interventional and observational data distributions, enabling in silico knockout experiments."
