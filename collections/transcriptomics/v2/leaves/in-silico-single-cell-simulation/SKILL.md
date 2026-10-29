---
name: in-silico-single-cell-simulation
description: Use when you need simulated single-cell RNA-seq data that preserves gene
  identities, cell trajectories, pseudo-time ordering, and noise of a reference dataset,
  or ground-truth GRNs for benchmarking GRN inference.
when_to_use_negative:
- You need unnormalized count data — GRouNdGAN currently generates only library-size
  normalized expression values across the considered genes and TFs
- You need a predictive model of cellular response on a new dataset — GRouNdGAN is
  generative, not predictive, and must be retrained for each new dataset
- Your GRN is multi-layer or includes TF-gene-phenotype links — the current version
  only supports imposing a bipartite GRN
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: GRouNdGAN
  role: Preprocess (--preprocess), infer GRN (--create_grn), train (--train), and
    generate simulated cells (--generate); also --evaluate and --perturb
  repo: github.com/Emad-COMBINE-lab/GRouNdGAN
- name: GRNBoost2
  role: Infers the GRN from preprocessed training data; GRouNdGAN formats it as a
    causal graph with top k=15 regulating TFs per gene
  repo: Murali-group/Beeline
- name: scanpy
  role: Jointly embed experimental and simulated cells and run PAGA trajectory inference
    and diffusion pseudotime for validation
  repo: scverse/scanpy-tutorials
- name: PyTorch
  role: Backend for GAN training on a CUDA GPU with checkpoint resuming from .pth
    files
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-48516-6
    title: 'GRouNdGAN: GRN-guided simulation of single-cell RNA-seq data using causal
      generative adversarial networks'
  source_content_hash: sha256:92f06ddde874caac86b9bb11350ccd0ee37af6a7bf01774ebc1ab3805c649199
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GRNBoost2
  - GRouNdGAN
  - PyTorch
  - scanpy
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# in-silico single-cell simulation

## Summary

Generate realistic synthetic scRNA-seq data by training GRouNdGAN, a GRN-guided causal implicit GAN, on a preprocessed experimental reference dataset and a GRNBoost2-inferred causal gene regulatory network, then sampling cells in silico.

## When to use

Use when you need simulated single-cell RNA-seq data that preserves gene identities, cell trajectories, pseudo-time ordering, and noise of a reference dataset, or ground-truth GRNs for benchmarking GRN inference.

## When NOT to use

- You need unnormalized count data — GRouNdGAN currently generates only library-size normalized expression values across the considered genes and TFs
- You need a predictive model of cellular response on a new dataset — GRouNdGAN is generative, not predictive, and must be retrained for each new dataset
- Your GRN is multi-layer or includes TF-gene-phenotype links — the current version only supports imposing a bipartite GRN

## Inputs

- Raw or preprocessed experimental scRNA-seq reference dataset (e.g., GEO GSE72857 BoneMarrow, PBMC-CTL, PBMC-NaiveT, Dahlin GSE107727, Tumor-ALL)
- Causal gene regulatory network (bipartite TF-gene graph), either GRNBoost2-inferred or user-provided
- GRouNdGAN config file with [EXPERIMENT]/checkpoint field and number of cells to generate
- Trained model checkpoint (.pth state dictionary)

## Outputs

- simulated.h5ad file containing library-size normalized simulated single-cell expression values
- Gold-standard ground truth GRN (exportable as CSV) used for simulation
- Quantitative/qualitative data-quality evaluation metrics for the simulated dataset

## How to apply

Preprocess the reference dataset (e.g., the Mouse BoneMarrow dataset, Paul et al., 2015, GEO GSE72857) with GRouNdGAN's --preprocess step, then infer a causal GRN with --create_grn, which uses GRNBoost2 with the top k=15 regulating TFs per gene. Train with --train under default settings (maximum steps = 1000000, batch size = 1024, critic iterations = 5, gradient-penalty lambda = 10) on a CUDA GPU, resuming from a .pth checkpoint if needed; alternatively impose your own GRN and skip --create_grn. Once trained, populate the [EXPERIMENT]/checkpoint field with the .pth state dictionary path, set the number of cells to generate (e.g., 2230 for BoneMarrow), and run --generate to write simulated.h5ad. Judge quality with --evaluate and downstream checks such as joint embedding overlap with experimental cells (miLISI near 2, RF AUROC near 0.5) and preservation of trajectory/pseudo-time structure.

## Related tools

- **GRouNdGAN** (Preprocess (--preprocess), infer GRN (--create_grn), train (--train), and generate simulated cells (--generate); also --evaluate and --perturb) — github.com/Emad-COMBINE-lab/GRouNdGAN
- **GRNBoost2** (Infers the GRN from preprocessed training data; GRouNdGAN formats it as a causal graph with top k=15 regulating TFs per gene) — Murali-group/Beeline
- **scanpy** (Jointly embed experimental and simulated cells and run PAGA trajectory inference and diffusion pseudotime for validation) — scverse/scanpy-tutorials
- **PyTorch** (Backend for GAN training on a CUDA GPU with checkpoint resuming from .pth files)

## Examples

```
git clone https://github.com/Emad-COMBINE-lab/GRouNdGAN.git && python main.py --preprocess --create_grn --train && python main.py --generate
```

## Evaluation signals

- Simulated cells overlap experimental cells in a joint embedding, with miLISI near 2 and RF AUROC near 0.5 (the paper's similarity criteria)
- Pseudotime ordering and cell-trajectory structure (PAGA) are preserved between simulated and experimental cells
- Lineage marker gene sets activate in the correct sequence along trajectories (e.g., erythroid Gata2, Gata1, Klf1, Hba-a2; myeloid Mpo, Ctsg; neutrophil Elane, Cebpe)
- Downstream GRN inference on simulated data aligns with BEELINE's experimental results on real data
- The --evaluate command reports acceptable quantitative and qualitative data-quality metrics for the simulated dataset

## Limitations

- Model generates only library-size normalized expression values; unnormalized count generation is planned for future iterations
- The current version only supports bipartite GRNs; multi-layer GRNs and TF-gene-phenotype causal graphs are not supported
- GRouNdGAN is generative, not predictive; the model must be retrained for a new dataset
- Resemblance to the reference dataset deteriorates if imposed TF-gene relationships differ significantly from reference data patterns
- Hyperparameters may need re-tuning via a validation set for new datasets or datasets with very different gene/TF counts
- On small datasets (~2.7k cells, e.g., BoneMarrow), simulator performance dropped on the test set compared to the training set

## Evidence

- [methods] GRN inference via GRNBoost2 with k=15: "Infer a GRN from preprocessed data using GRNBoost2 and appropriately format as causal graph"
- [methods] Training defaults: "--train          Start or resume model training"
- [methods] Checkpoint-generate step: "Once training is done, populate the ``[EXPERIMENT]/checkpoint`` field with the path of the ``.pth`` checkpoint"
- [methods] Generation command: "--generate       Simulate single-cells RNA-seq data in-silico"
- [abstract] What the model preserves: "our model captures non-linear TF-gene dependencies and preserves gene identities, cell trajectories, pseudo-time ordering, and technical and biological noise"
- [results] Similarity criteria: [evidence span withheld — anchor status: not_found]
- [discussion] Normalization restriction: "the model generates expression values library-size normalized across the considered genes and TFs"
- [discussion] Bipartite-only limitation: "The current version of GRouNdGAN only allows imposing a bipartite GRN and does not support a multi-layer GRN"
- [readme] Custom GRN option: "It is possible to instead impose your own GRN onto GRouNdGAN. If you're opting for this option, skip the ``--create_grn`` command."
