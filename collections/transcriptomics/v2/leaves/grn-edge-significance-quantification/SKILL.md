---
name: grn-edge-significance-quantification
description: Use when GRouNdGAN has been trained with an imposed GRN (e.g., GRNBoost2-
  or PPCOR-derived) on PBMC-CTL or a similar reference dataset and you need to test
  whether knocking out each regulating TF significantly changes its target genes'
  simulated expression.
when_to_use_negative:
- The model was trained without an imposed bipartite GRN (GRouNdGAN's current version
  only supports bipartite GRNs, so multi-layer or TF-gene-phenotype graphs cannot
  be tested)
- You need to predict TF knockout outcomes in real (not simulated) cells — the paper
  notes this would require a high-quality GRN plus additional modalities such as (sc)ATAC-seq
  or TF ChIP-seq
- Perturbed and control cells come from different simulated batches — the Wilcoxon
  signed-rank comparison requires matched case-control pairs from the same fixed batch
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: GRouNdGAN
  role: Samples interventional (TF-knockout, perturbation=0) and matched observational
    expression data from the trained causal GAN for each TF in [Perturbation]/tfs
  repo: github.com/Emad-COMBINE-lab/GRouNdGAN
- name: GRNBoost2
  role: Provides the reference TF-gene edges (imposed GRN) whose significance under
    knockout is being quantified
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-48516-6
    title: 'GRouNdGAN: GRN-guided simulation of single-cell RNA-seq data using causal
      generative adversarial networks'
  source_content_hash: sha256:41bee1d85a82ecae80298fa4814f2f97cb6063d0538eee514dd2f8270872a3b5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GRNBoost2
  - GRouNdGAN
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# grn-edge-significance-quantification

## Summary

Quantifies the fraction of imposed TF-gene edges whose target-gene expression is significantly altered by in silico TF knockout in GRouNdGAN-simulated cells, using two-sided Wilcoxon signed-rank tests with Benjamini-Hochberg FDR correction. It validates that the imposed causal GRN is reflected in the interventional (perturbed) simulations.

## When to use

Use when GRouNdGAN has been trained with an imposed GRN (e.g., GRNBoost2- or PPCOR-derived) on PBMC-CTL or a similar reference dataset and you need to test whether knocking out each regulating TF significantly changes its target genes' simulated expression.

## When NOT to use

- The model was trained without an imposed bipartite GRN (GRouNdGAN's current version only supports bipartite GRNs, so multi-layer or TF-gene-phenotype graphs cannot be tested)
- You need to predict TF knockout outcomes in real (not simulated) cells — the paper notes this would require a high-quality GRN plus additional modalities such as (sc)ATAC-seq or TF ChIP-seq
- Perturbed and control cells come from different simulated batches — the Wilcoxon signed-rank comparison requires matched case-control pairs from the same fixed batch

## Inputs

- Trained GRouNdGAN model checkpoint file (e.g., step_1000000.pth) referenced in [EXPERIMENT]/checkpoint
- Imposed ground truth GRN (TF-gene edge list) such as a GRNBoost2-inferred or PPCOR-inferred causal graph
- GRouNdGAN configuration YAML with [Perturbation]/tfs and [Perturbation]/perturbation fields
- Unperturbed and TF-knockout (perturbation value 0) simulated expression matrices sampled from the same fixed cell batch

## Outputs

- knockout_edge_significance.csv with per-TF-gene-edge Wilcoxon statistics, raw and BH-adjusted p-values, and significance calls
- Fraction of significantly perturbed TF-gene edges per imposed GRN (e.g., 66.5% GRNBoost2, 75.2% PPCOR)

## How to apply

Load the trained GRouNdGAN checkpoint (e.g., step_1000000.pth) specified via [EXPERIMENT]/checkpoint in the config file for the GRN-imposed model. Set the causal controller to deterministic mode and, for each TF one at a time, run the --perturb command with [Perturbation]/tfs set to the TF and [Perturbation]/perturbation set to 0, sampling perturbed and unperturbed cells from the same fixed batch to enable matched case-control comparison. For each knocked-out TF, apply a two-sided Wilcoxon signed-rank test comparing matched perturbed vs. control expression for each gene it regulates in the imposed GRN, then adjust all TF-gene edge p-values with Benjamini-Hochberg FDR and call edges significant at adjusted p-value < 0.05. Compute the fraction of significantly perturbed TF-gene edges per imposed GRN (paper reproduces 66.5% for GRNBoost2 and 75.2% for PPCOR on PBMC-CTL) and export per-edge results to knockout_edge_significance.csv.

## Related tools

- **GRouNdGAN** (Samples interventional (TF-knockout, perturbation=0) and matched observational expression data from the trained causal GAN for each TF in [Perturbation]/tfs) — github.com/Emad-COMBINE-lab/GRouNdGAN
- **GRNBoost2** (Provides the reference TF-gene edges (imposed GRN) whose significance under knockout is being quantified)

## Examples

```
python main.py --config config.yaml --perturb
```

## Evaluation signals

- The computed fraction of significantly affected TF-gene edges reproduces 66.5% for the GRNBoost2 GRN and 75.2% for the PPCOR GRN on PBMC-CTL within small numerical tolerance
- Every tested edge's adjusted p-value is present in knockout_edge_significance.csv and significance is called at adjusted p < 0.05 under Benjamini-Hochberg correction
- Each Wilcoxon test compares matched perturbed vs. control cells sampled from the same fixed batch (paired, two-sided signed-rank)
- Perturbed simulations were generated with the causal controller in deterministic mode and [Perturbation]/perturbation set to 0 for exactly one TF at a time

## Limitations

- GRouNdGAN is a generative, not predictive, model; it must be retrained for a new dataset before knockout significance can be assessed
- Reproduction of significance rates depends on the imposed GRN; resemblance to the reference dataset deteriorates if imposed TF-gene relationships differ significantly from reference data patterns
- The model generates library-size normalized expression values only, which are the values the Wilcoxon tests operate on
- The underlying causal graph of an observational dataset is not necessarily unique, so a high edge-significance fraction supports but does not prove the imposed GRN is the true causal one

## Evidence

- [abstract] GRouNdGAN can synthesize cells under new conditions to perform in silico TF knockout experiments.: "GRouNdGAN can synthesize cells under new conditions to perform in silico TF knockout experiments."
- [methods] --perturb performs perturbation experiments: "--perturb        Perform a perturbation experiment using a trained GRouNdGAN model"
- [intro] causal structure enables interventional sampling: "the causal structure of GRouNdGAN enables it to be used for sampling from both interventional and observational data distributions, enabling in silico knockout experiments."
- [results] finding: 66.5% of edges significant after knockout: [evidence span withheld — anchor status: not_found]
- [discussion] resemblance deteriorates if imposed relations mismatch reference: "the resemblance of the data to the reference dataset deteriorates if the imposed TF-gene relationships are significantly different from those consistent with the patterns in the reference dataset"
