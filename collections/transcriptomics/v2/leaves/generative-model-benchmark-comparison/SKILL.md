---
name: generative-model-benchmark-comparison
description: Use when synthetic scRNA-seq data from a trained generative model must
  be quantitatively compared to held-out real cells and rival simulators (e.g., 10x
  Genomics PBMC-CTL, 1000 generated cells vs 1000 test cells).
when_to_use_negative:
- The goal is benchmarking GRN inference algorithms rather than simulated-data quality
  — use GRouNdGAN's --benchmark_grn instead of the resemblance-metric suite.
- 'The reference dataset is very small (~2.7k cells, e.g., BoneMarrow): all simulators''
  performance dropped on the test set, so metric comparisons are unreliable.'
- 'The input is PBMC-All or another multi-cell-type mixture: the article states benchmarking
  results there are not conclusive and follow-up studies are required.'
- 'You need unnormalized count data: GRouNdGAN only generates library-size normalized
  expression values.'
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: GRouNdGAN
  role: Reference-based causal GAN whose --train, --generate, and --evaluate commands
    produce the simulated cells and the metric suite (cosine, Euclidean, MMD, RF AUROC,
    miLISI) being benchmarked
  repo: github.com/Emad-COMBINE-lab/GRouNdGAN
- name: GRNBoost2
  role: Infers the causal GRN (k=15 top TFs per gene via --create_grn) that guides
    the simulated data whose quality is benchmarked
- name: scGAN
  role: Baseline generative simulator trained via gan.cfg on the same PBMC-CTL train
    set for head-to-head metric comparison (cosine 0.00095, Euclidean 222, MMD 0.031,
    RF AUROC 0.59, miLISI 1.888)
- name: scDESIGN2
  role: Baseline probabilistic simulator run on the same train set and scored with
    the identical metric suite
- name: SPARSim
  role: Baseline simulator cloned as a GRouNdGAN submodule and scored with the identical
    metric suite
- name: Docker
  role: Container runtime for the prebuilt GRouNdGAN image (yazdanz/groundgan:4b98686)
    executing the benchmark commands
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-48516-6
    title: 'GRouNdGAN: GRN-guided simulation of single-cell RNA-seq data using causal
      generative adversarial networks'
  source_content_hash: sha256:1807020b6c9395a1c0624eb9337548e4dcd99977fba4e46c493ccd1ab07e46a0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Docker
  - GRNBoost2
  - GRouNdGAN
  - SPARSIM
  - SPARSim
  - scDESIGN2
  - scGAN
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# generative-model-benchmark-comparison

## Summary

Benchmarks GRN-guided generative simulators (GRouNdGAN vs scGAN, scDESIGN2, SPARSim) on scRNA-seq realism using a shared resemblance-metric suite (cosine, Euclidean, MMD, RF AUROC, miLISI) against held-out real cells and a real-real control.

## When to use

Use when synthetic scRNA-seq data from a trained generative model must be quantitatively compared to held-out real cells and rival simulators (e.g., 10x Genomics PBMC-CTL, 1000 generated cells vs 1000 test cells).

## When NOT to use

- The goal is benchmarking GRN inference algorithms rather than simulated-data quality — use GRouNdGAN's --benchmark_grn instead of the resemblance-metric suite.
- The reference dataset is very small (~2.7k cells, e.g., BoneMarrow): all simulators' performance dropped on the test set, so metric comparisons are unreliable.
- The input is PBMC-All or another multi-cell-type mixture: the article states benchmarking results there are not conclusive and follow-up studies are required.
- You need unnormalized count data: GRouNdGAN only generates library-size normalized expression values.

## Inputs

- Raw 10x Genomics scRNA-seq count matrix (e.g., PBMC-CTL, CD8+ cytotoxic T-cells from Zheng et al. 2017)
- Transcription factor list (Homo_sapiens_TF.csv) for GRN inference with GRNBoost2
- causal_gan.cfg configuration file with [EXPERIMENT]/checkpoint pointing to the trained .pth model
- Trained checkpoints of baseline simulators (scGAN via gan.cfg, scDESIGN2, SPARSim cloned as submodules)
- Held-out test set of 1000 real cells for simulated-vs-real and real-real control evaluations

## Outputs

- simulated.h5ad containing 1000 synthetic scRNA-seq cells
- Resemblance metric table per simulator: cosine distance, Euclidean distance, MMD, RF AUROC, and miLISI
- Real-real control metric values from two halves of the held-out test set (e.g., Euclidean 242.8, MMD 0.062, miLISI 1.890)
- t-SNE qualitative comparison plots of simulated vs real data

## How to apply

Preprocess the reference 10x dataset with GRouNdGAN's --preprocess (drop genes in <3 cells and cells with <10 genes, library-size normalize to 20,000, keep 1000 highly variable genes, split off 1000-cell validation and test sets), infer the GRN with --create_grn (GRNBoost2, Homo_sapiens_TF.csv, k=15 top TFs per gene), and train the causal GAN (batch size 1024, 5 critic iterations per generator iteration, latent dim 128, gradient penalty lambda 10, up to 1,000,000 steps). Populate the .pth checkpoint in causal_gan.cfg and run --generate with 1000 cells to produce simulated.h5ad. Run --evaluate with plot tsne, euclidean distance, cosine distance, RF AUROC, MMD, and miLISI enabled, computing every metric twice: simulated-vs-real (1000 synthetic vs 1000 held-out test cells) and real-real (two halves of the test set) as the control. Train scGAN (gan.cfg), scDESIGN2, and SPARSim (cloned as submodules) on the identical train set and compute the identical metric suite for side-by-side Table 1 comparison. Accept the simulation when RF AUROC is near 0.5 (GRouNdGAN 0.54 vs scGAN 0.59), miLISI is near 2 (1.891 vs 1.890 control), and distances are comparable to or lower than the real-real control (e.g., Euclidean 182 vs 242.8 control, MMD 0.026 vs 0.062 control, cosine 0.00057).

## Related tools

- **GRouNdGAN** (Reference-based causal GAN whose --train, --generate, and --evaluate commands produce the simulated cells and the metric suite (cosine, Euclidean, MMD, RF AUROC, miLISI) being benchmarked) — github.com/Emad-COMBINE-lab/GRouNdGAN
- **GRNBoost2** (Infers the causal GRN (k=15 top TFs per gene via --create_grn) that guides the simulated data whose quality is benchmarked)
- **scGAN** (Baseline generative simulator trained via gan.cfg on the same PBMC-CTL train set for head-to-head metric comparison (cosine 0.00095, Euclidean 222, MMD 0.031, RF AUROC 0.59, miLISI 1.888))
- **scDESIGN2** (Baseline probabilistic simulator run on the same train set and scored with the identical metric suite)
- **SPARSim** (Baseline simulator cloned as a GRouNdGAN submodule and scored with the identical metric suite)
- **Docker** (Container runtime for the prebuilt GRouNdGAN image (yazdanz/groundgan:4b98686) executing the benchmark commands)

## Examples

```
docker run yazdanz/groundgan:4b98686 --evaluate
```

## Evaluation signals

- RF AUROC of simulated-vs-real classifier is close to 0.5 (GRouNdGAN 0.54; higher values like scGAN's 0.59 mean the discriminator separates synthetic from real cells)
- miLISI is close to 2 (expected ~1.891 for GRouNdGAN, ~1.890 for the real-real control)
- Euclidean distance of simulated-vs-real is lower than or comparable to the real-real control (182 vs 242.8; demo reference 171.2 vs 242.8)
- MMD of simulated-vs-real is at or below the real-real control (0.026 vs 0.062; demo reference 0.030 vs 0.062)
- Cosine distance reproduces Table 1 values (GRouNdGAN 0.00057 vs scGAN 0.00095 on PBMC-CTL)

## Limitations

- On small datasets like BoneMarrow (~2.7k cells), all simulators' performance dropped on the test set compared to the training set, weakening metric comparisons.
- Hyperparameters may need re-tuning via a validation set when moving to new datasets or datasets with significantly different numbers of genes and TFs.
- Resemblance to the reference deteriorates if the imposed TF-gene relationships differ significantly from patterns in the reference data.
- The model only outputs library-size normalized expression values across the considered genes and TFs; unnormalized count generation is planned for future iterations.

## Evidence

- [methods] You can evaluate the quality of GRouNdGAN simulations using the following quantitative and qualitative metrics:: "You can evaluate the quality of GRouNdGAN simulations using the following quantitative and qualitative metrics:"
- [intro] Baselines are the established simulators: "For example, scGAN24, cscGAN24, scDESIGN225, and SPARSIM26 capture inter-gene correlations in their datasets."
- [discussion] Small-dataset caveat for cross-simulator comparison: "In the BoneMarrow dataset (which had only ~2.7k cells), the performance of all simulators dropped on the test set compared to the training set."
- [discussion] Hyperparameter re-tuning caveat: "when trained on a completely new dataset, or datasets containing significantly different number of genes and TFs, it is advisable to try different choices of the hyperparameters"
- [readme] Docker image for running the benchmark: "$ docker pull yazdanz/groundgan:4b98686"
