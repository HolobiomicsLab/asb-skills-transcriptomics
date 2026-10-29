---
name: matched-case-control-expression-testing
description: Use when you need to test, edge by edge, whether knocking out a regulating
  TF via GRouNdGAN's --perturb mode significantly changes the expression of each gene
  it regulates in an imposed GRN.
when_to_use_negative:
- You want to predict TF knockout outcomes in real (non-simulated) cells — the paper
  notes this would require a high-quality GRN plus additional data modalities such
  as (sc)ATAC-seq or TF ChIP-seq
- Your imposed GRN is multi-layer or includes TF-gene-phenotype edges — the current
  GRouNdGAN version only supports imposing a bipartite GRN
- The task is quality assessment of simulated data or GRN-inference benchmarking rather
  than interventional significance testing
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: GRouNdGAN
  role: Samples interventional (TF-knockout) and matched unperturbed single-cell expression
    data from a trained causal GAN via the --perturb command
  repo: https://github.com/Emad-COMBINE-lab/GRouNdGAN
- name: GRNBoost2
  role: Infers the TF-gene edges of the imposed GRN that define which genes are tested
    for each knocked-out TF
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-48516-6
    title: 'GRouNdGAN: GRN-guided simulation of single-cell RNA-seq data using causal
      generative adversarial networks'
  source_content_hash: sha256:f5ea38e024054e74a329130df16d7c479e2557995c176cf3b42bf1602639ea8d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Emad-COMBINE-lab/GRouNdGAN
  tools:
  - GRNBoost2
  - GRouNdGAN
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# matched-case-control-expression-testing

## Summary

Quantifies whether in silico TF knockout in GRouNdGAN-simulated cells significantly alters target gene expression, by comparing matched perturbed vs. unperturbed cells with two-sided Wilcoxon signed-rank tests and Benjamini-Hochberg correction over TF-gene edges. It validates that the user-imposed causal GRN is actually realized in the simulated data.

## When to use

Use when you need to test, edge by edge, whether knocking out a regulating TF via GRouNdGAN's --perturb mode significantly changes the expression of each gene it regulates in an imposed GRN.

## When NOT to use

- You want to predict TF knockout outcomes in real (non-simulated) cells — the paper notes this would require a high-quality GRN plus additional data modalities such as (sc)ATAC-seq or TF ChIP-seq
- Your imposed GRN is multi-layer or includes TF-gene-phenotype edges — the current GRouNdGAN version only supports imposing a bipartite GRN
- The task is quality assessment of simulated data or GRN-inference benchmarking rather than interventional significance testing

## Inputs

- Trained GRouNdGAN checkpoint (.pth file, e.g. step_1000000.pth) referenced by [EXPERIMENT]/checkpoint
- Config file specifying [Perturbation]/tfs (TFs to knock out) and [Perturbation]/perturbation values set to 0
- Imposed ground-truth GRN (e.g. GRNBoost2- or PPCOR-inferred bipartite TF-gene graph)
- Matched perturbed and unperturbed single-cell expression samples drawn from the same fixed batch

## Outputs

- Per-edge knockout significance table written to knockout_edge_significance.csv
- Fraction of significantly perturbed TF-gene edges per imposed GRN (target: 66.5% for GRNBoost2, 75.2% for PPCOR on PBMC-CTL)
- Adjusted (Benjamini-Hochberg) p-values for each TF-gene edge

## How to apply

Load the trained GRouNdGAN checkpoint (e.g. step_1000000.pth) via [EXPERIMENT]/checkpoint in the config file for both the GRNBoost2-imposed and PPCOR-imposed models. Set the causal controller to deterministic mode and, one TF at a time, run the --perturb command with [Perturbation]/tfs set to the TF and [Perturbation]/perturbation set to 0, sampling perturbed and unperturbed cells from the same fixed batch so the comparison is matched case-control rather than two independent draws. For each knocked-out TF, apply a two-sided Wilcoxon signed-rank test on matched perturbed vs. control expression for each gene it regulates in the imposed GRN. Adjust all TF-gene edge p-values with the Benjamini-Hochberg FDR procedure and call edges significant at adjusted p-value < 0.05. The rationale is that GRouNdGAN's causal structure permits sampling from interventional distributions, so the fraction of significantly perturbed edges directly measures whether the imposed GRN is causally reflected in the simulated cells.

## Related tools

- **GRouNdGAN** (Samples interventional (TF-knockout) and matched unperturbed single-cell expression data from a trained causal GAN via the --perturb command) — https://github.com/Emad-COMBINE-lab/GRouNdGAN
- **GRNBoost2** (Infers the TF-gene edges of the imposed GRN that define which genes are tested for each knocked-out TF)

## Evaluation signals

- The fraction of significantly affected TF-gene edges reproduces 66.5% for the GRNBoost2 GRN and 75.2% for the PPCOR GRN on PBMC-CTL (within small numerical tolerance)
- Per-edge results are written to knockout_edge_significance.csv with BH-adjusted p-values and significance calls at adjusted p-value < 0.05
- Perturbed and control samples come from the same fixed batch, so each Wilcoxon test is a matched two-sided signed-rank comparison rather than an unpaired test
- The causal controller was in deterministic mode and each TF was perturbed individually with [Perturbation]/perturbation set to 0

## Limitations

- GRouNdGAN is a generative, not predictive, model: given a new dataset the model must be retrained before knockouts are meaningful
- Only bipartite GRNs can be imposed; multi-layer GRNs and TF-gene-phenotype causal graphs are not supported
- Resemblance to the reference dataset deteriorates if the imposed TF-gene relationships differ significantly from patterns in the reference data
- The underlying causal graph of an observational dataset is not necessarily unique; verifying the imposed GRN is the most probable one was beyond the study's scope
- The model generates only library-size normalized expression values; unnormalized count generation is planned for future iterations

## Evidence

- [methods] --perturb        Perform a perturbation experiment using a trained GRouNdGAN model: "--perturb        Perform a perturbation experiment using a trained GRouNdGAN model"
- [intro] causal structure enables interventional sampling: "the causal structure of GRouNdGAN enables it to be used for sampling from both interventional and observational data distributions, enabling in silico knockout experiments"
- [discussion] bipartite GRN restriction: "The current version of GRouNdGAN only allows imposing a bipartite GRN and does not support a multi-layer GRN"
- [discussion] generative not predictive: [evidence span withheld — anchor status: not_found]
- [methods] GRNBoost2 GRN creation: "This command uses GRNBoost2 (Moerman et al., 2018) to infer a GRN on the preprocessed train set."
