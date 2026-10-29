---
name: tsne-embedding-visualization
description: Use when assessing whether simulated single-cell RNA-seq data (e.g.,
  GRouNdGAN output vs 1000 held-out test cells) resembles the reference distribution,
  as the qualitative counterpart to metric-based evaluation in --evaluate.
when_to_use_negative:
- You need a rigorous quantitative verdict on simulation realism — t-SNE is qualitative
  and must be paired with the metric suite (RF AUROC near 0.5, miLISI near 2, distance
  comparisons)
- The expression data are raw unnormalized counts — GRouNdGAN operates on library-size
  normalized values (normalized to 20,000 over the 1000 selected HVGs), and t-SNE
  comparison should use the same preprocessed representation
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: GRouNdGAN
  role: Provides the --evaluate command whose t-SNE plotting option produces the qualitative
    resemblance visualization between simulated.h5ad and held-out test cells
  repo: https://github.com/Emad-COMBINE-lab/GRouNdGAN
- name: scGAN
  role: Baseline simulator whose output receives the identical t-SNE evaluation for
    visual comparison against GRouNdGAN
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-48516-6
    title: 'GRouNdGAN: GRN-guided simulation of single-cell RNA-seq data using causal
      generative adversarial networks'
  source_content_hash: sha256:b492ac19169e50b4867b483abd73151d7c71b056c89497ad05493b01c312e289
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Emad-COMBINE-lab/GRouNdGAN
  tools:
  - Docker
  - GRouNdGAN
  - SPARSIM
  - scDESIGN2
  - scGAN
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# t-SNE embedding visualization

## Summary

Qualitative evaluation skill from the GRouNdGAN workflow: compute a t-SNE embedding on simulated and real (held-out test) scRNA-seq cells to visually compare their joint distributions alongside quantitative resemblance metrics (Euclidean, cosine, MMD, RF AUROC, miLISI).

## When to use

Use when assessing whether simulated single-cell RNA-seq data (e.g., GRouNdGAN output vs 1000 held-out test cells) resembles the reference distribution, as the qualitative counterpart to metric-based evaluation in --evaluate.

## When NOT to use

- You need a rigorous quantitative verdict on simulation realism — t-SNE is qualitative and must be paired with the metric suite (RF AUROC near 0.5, miLISI near 2, distance comparisons)
- The expression data are raw unnormalized counts — GRouNdGAN operates on library-size normalized values (normalized to 20,000 over the 1000 selected HVGs), and t-SNE comparison should use the same preprocessed representation

## Inputs

- simulated.h5ad (1000 GRouNdGAN-simulated cells over 1000 highly variable genes)
- 1000 held-out test cells from the preprocessed reference dataset (e.g., PBMC-CTL 10x Genomics data)
- real-real control split: two halves of the test set (500 + 500 cells)

## Outputs

- t-SNE embedding plot overlaying simulated and real test cells
- companion quantitative metrics on the same pairs (Euclidean, cosine, MMD, RF AUROC, miLISI)

## How to apply

After generating simulated cells (e.g., 1000 cells written to simulated.h5ad), run GRouNdGAN's --evaluate with the t-SNE plotting option enabled together with the quantitative metrics (Euclidean distance, cosine distance, RF AUROC, MMD, miLISI), computing each comparison between the simulated data and the 1000 held-out test cells and, as a real-real control, between two halves of the test set. The t-SNE plot embeds simulated and real cells jointly so that overlap of the two clouds indicates realistic generation; it complements the numeric acceptance criteria that the RF AUROC is close to 0.5, miLISI is close to 2, and simulated-vs-real distances are comparable to or lower than the real-real control (demo reference: Euclidean 171.2 vs 242.8, MMD 0.030 vs 0.062, miLISI 1.897 vs 1.890). The same plotting step is applied identically to baseline simulators (scGAN, scDESIGN2, SPARSim) trained on the same train set so qualitative resemblance can be compared across simulators. Interpretation is qualitative: t-SNE supports the metric suite, it does not replace it.

## Related tools

- **GRouNdGAN** (Provides the --evaluate command whose t-SNE plotting option produces the qualitative resemblance visualization between simulated.h5ad and held-out test cells) — https://github.com/Emad-COMBINE-lab/GRouNdGAN
- **scGAN** (Baseline simulator whose output receives the identical t-SNE evaluation for visual comparison against GRouNdGAN)

## Examples

```
python main.py --evaluate -exp [EXPERIMENT_NAME] --plot_tsne --euclidean_distance --cosine_distance --rf_auroc --mmd --mi_lisi
```

## Evaluation signals

- Simulated and real cells visually overlap in the joint t-SNE embedding rather than forming separate clusters
- RF AUROC distinguishing simulated from real test cells is close to 0.5 (GRouNdGAN PBMC-CTL: 0.54)
- miLISI is close to 2 (GRouNdGAN PBMC-CTL: 1.891)
- Simulated-vs-real Euclidean/cosine/MMD distances are comparable to or lower than the real-real control (e.g., Euclidean 171.2 vs 242.8, MMD 0.030 vs 0.062)
- Cosine distance of simulated-vs-real matches reported values (GRouNdGAN PBMC-CTL: 0.00057 vs scGAN 0.00095)

## Limitations

- t-SNE is only a qualitative check; acceptance of the simulation rests on the quantitative criteria (RF AUROC ≈ 0.5, miLISI ≈ 2, distances vs real-real control)
- All simulators' performance can drop on small reference datasets (~2.7k cells, e.g., BoneMarrow), degrading both visual and metric-based resemblance
- Resemblance deteriorates if imposed TF-gene relationships differ significantly from reference data patterns

## Evidence

- [methods] evaluate flag enables t-SNE alongside metric suite: "--evaluate       Evaluate the data quality of the simulated dataset"
- [methods] qualitative and quantitative metrics listed together: "You can evaluate the quality of GRouNdGAN simulations using the following quantitative and qualitative metrics:"
- [other] workflow step: t-SNE plotted between simulated and test cells and within real-real control: [evidence span withheld — anchor status: not_found]
- [other] acceptance thresholds grounded in demo reference values: [evidence span withheld — anchor status: not_found]
