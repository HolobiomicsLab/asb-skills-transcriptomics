---
name: transcription-factor-hit-prioritization
description: Use when you have a fitted PRESCIENT model (data_pt) on time-series scRNA-seq
  and need to identify which transcription factors significantly bias differentiation
  toward a target fate before designing an in vitro perturbational screen.
when_to_use_negative:
- You want causal validation that a gene drives cell fate — in silico association
  of genes with fates does not imply causality, and challenges remain to confidently
  suggest gene sets for experimental perturbation
- Your target effectors are not transcription factors and your PRESCIENT version/analysis
  assumes TF-only screening — non-TF targets such as signaling pathway effectors can
  also be tested but were not covered by this workflow
- The final time point of your dataset is not at steady-state — the model's assumptions
  break down and fate probability estimates would be unreliable
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: PRESCIENT
  role: Generative modeling framework used to simulate perturbed differentiation trajectories
    and compute fate probabilities for each overexpressed TF
  repo: https://github.com/gifford-lab/prescient
- name: PyTorch
  role: Backend for the serialized data_pt torch object and PRESCIENT model training/simulation
  repo: https://github.com/gifford-lab/prescient
- name: scikit-learn
  role: PCA object (data_pt['pca']) fit to normalized expression, used to map perturbations
    into the model's PC space
- name: UMAP
  role: UMAP object (data_pt['um']) fit to PCs, used for visualizing perturbed versus
    unperturbed fate distributions
- name: prescient-analysis
  role: Notebooks and pre-processing/training workflows for the Veres et al. beta-cell
    differentiation application
  repo: https://github.com/gifford-lab/prescient-analysis
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-23518-w
    title: Generative modeling of single-cell time series with PRESCIENT enables prediction
      of cell trajectories with interventions
  source_content_hash: sha256:772faa88c7a575c467bff6c7b460cbaaa2f960572e238ffa8efbf54cf48cbe76
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gifford-lab/prescient
  tools:
  - PRESCIENT
  - PyTorch
  - UMAP
  - prescient-analysis
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# transcription-factor-hit-prioritization

## Summary

Run a large-scale unbiased in silico perturbation screen with PRESCIENT over 200+ transcription factors from the Veres et al. in vitro beta-cell differentiation highly variable gene set, then rank significant fate-shifting hits (FDR < 0.01, log2FC > 0.5) for alpha-, beta-, and EC-cell fates to prioritize targets for experimental perturbational screens.

## When to use

Use when you have a fitted PRESCIENT model (data_pt) on time-series scRNA-seq and need to identify which transcription factors significantly bias differentiation toward a target fate before designing an in vitro perturbational screen.

## When NOT to use

- You want causal validation that a gene drives cell fate — in silico association of genes with fates does not imply causality, and challenges remain to confidently suggest gene sets for experimental perturbation
- Your target effectors are not transcription factors and your PRESCIENT version/analysis assumes TF-only screening — non-TF targets such as signaling pathway effectors can also be tested but were not covered by this workflow
- The final time point of your dataset is not at steady-state — the model's assumptions break down and fate probability estimates would be unreliable

## Inputs

- Serialized PRESCIENT torch object (data_pt): normalized gene expression matrix, gene feature list, cell type labels, timepoint assignments, PCA and UMAP objects
- List of transcription factors from the Veres et al. in vitro beta-cell differentiation highly variable gene set
- PRESCIENT-trained model on the Veres et al. dataset (e.g., from veres-fate.sh, 5 seeds of 2 layer 400 unit models with estimated proliferation rates)

## Outputs

- Ranked hit table (TF, fate, log2FC, t-statistic, FDR) written to tf_perturbation_hits.csv
- Per-TF fate probability distributions across 10 random initializations for perturbed and unperturbed conditions
- Lists of significant fate-specifying TFs (10 alpha, 18 beta, 19 EC hits at FDR < 0.01 and log2FC > 0.5)

## How to apply

Load the serialized PRESCIENT torch object (data_pt) containing the normalized expression matrix, gene features, cell type labels, time-point assignments, and PCA/UMAP objects. Compile the perturbation target list by intersecting data_pt gene features with transcription factors in the Veres et al. highly variable gene set. For each TF, simulate overexpression perturbed differentiation trajectories (z = 5) from the relevant starting cell population, repeating each simulation over 10 random initializations to capture stochasticity. Compute fate probabilities for alpha-, beta-, and EC-cell outcomes in perturbed versus unperturbed simulations and evaluate paired t-tests across the 10 random initializations. Apply Benjamini-Hochberg FDR correction across the screened TFs and retain hits passing FDR < 0.01 with log2FC > 0.5; in the published screen this yielded 10 alpha-fate, 18 beta-fate, and 19 EC-fate hits.

## Related tools

- **PRESCIENT** (Generative modeling framework used to simulate perturbed differentiation trajectories and compute fate probabilities for each overexpressed TF) — https://github.com/gifford-lab/prescient
- **PyTorch** (Backend for the serialized data_pt torch object and PRESCIENT model training/simulation) — https://github.com/gifford-lab/prescient
- **scikit-learn** (PCA object (data_pt['pca']) fit to normalized expression, used to map perturbations into the model's PC space)
- **UMAP** (UMAP object (data_pt['um']) fit to PCs, used for visualizing perturbed versus unperturbed fate distributions)
- **prescient-analysis** (Notebooks and pre-processing/training workflows for the Veres et al. beta-cell differentiation application) — https://github.com/gifford-lab/prescient-analysis

## Examples

```
prescient process_data --input veres_normalized.csv --output data_pt; python simulate_perturbations.py --data_pt data_pt --z 5 --n_init 10 --fdr 0.01 --log2fc 0.5 --out tf_perturbation_hits.csv
```

## Evaluation signals

- All retained hits satisfy FDR < 0.01 (Benjamini-Hochberg corrected across the screened TFs) and log2FC > 0.5
- Each TF's paired t-test is computed across 10 random simulation initializations, so per-TF statistics derive from n=10 replicates
- Hit counts should be in the expected range for the Veres et al. screen: 10 alpha-fate, 18 beta-fate, and 19 EC-fate significant hits
- Well-studied regulatory genes of beta-cell differentiation should appear among hits with expected fate effects, as reported in the paper
- Every screened TF must be present in both the data_pt gene feature list and the Veres et al. highly variable TF list (correct intersection)

## Limitations

- The model requires the final time point of the dataset to be at steady-state
- Information is lost about individual genes when transforming data into PCA space, and lowly-expressed genes important to cell fate decisions may be dropped altogether in the scRNA-seq data
- Association of certain genes with specific cell fates does not necessarily imply causality; there remain challenges to confidently suggest gene sets for experimental perturbation
- Model performance depends on growth rates and would benefit from better estimates of proliferation
- This work was limited to TFs; non-TF targets such as signaling pathway effectors can also be tested using PRESCIENT but were not screened here

## Evidence

- [discussion] Large-scale in silico TF screen with fate-specific significance filtering: "large-scale screen for in vitro β-cell differentiation in which we perturbed 200+ TFs and identified target genes that could cause significant shifts in β-cell, α-cell, and EC-cell fates."
- [intro] PRESCIENT enables unbiased in silico perturbation screens: "PRESCIENT enables large unbiased in silico perturbation experiments to aid the design of in vitro genetic perturbational screens"
- [methods] prescient process_data generates the data_pt torch object used as screen input: "The `prescient process_data` command will generate a torch pt file `data_pt` (serialized dictionary) that contains all the necessary information for downstream training, simulations, and"
- [abstract] Known modulators recovered in perturbation simulations: "We demonstrate how PRESCIENT can simulate trajectories for perturbed cells, recovering the expected effects of known modulators of cell fate in hematopoiesis and pancreatic β cell differentiation."
- [readme] prescient-analysis provides the Veres et al. training workflow: [evidence span withheld — anchor status: external_doc]
