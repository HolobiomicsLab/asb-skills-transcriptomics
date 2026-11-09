---
name: grn-inference-with-grnboost2
description: 'Use when preparing a GRN-guided causal GAN run: you have a preprocessed
  single-cell expression matrix (e.g., 1000 highly variable genes, library-size normalized
  to 20,000) and need a causal graph of TF-gene edges to impose during GRouNdGAN training.'
when_to_use_negative:
- You already have a trusted, user-provided GRN to impose on GRouNdGAN — in that case
  skip --create_grn and use the imposing-custom-GRNs route.
- You need a multi-layer or TF-gene-phenotype causal graph — the current GRouNdGAN
  version only supports imposing bipartite GRNs.
- You are simulating unnormalized count data — the model generates expression values
  library-size normalized across the considered genes and TFs.
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3308
tools:
- name: GRouNdGAN
  role: Containerizes and invokes the GRN inference workflow via its --create_grn
    command, formatting GRNBoost2 output as a causal graph for causal GAN training.
  repo: https://github.com/Emad-COMBINE-lab/GRouNdGAN
- name: GRNBoost2
  role: Underlying GRN inference algorithm run by --create_grn to infer TF-gene regulatory
    edges from the preprocessed train set (Moerman et al., 2018).
- name: Beeline
  role: Benchmarking framework whose experimental GRN inference results the GRouNdGAN-based
    benchmark is aligned against.
  repo: https://github.com/Murali-group/Beeline
- name: Docker
  role: Distribution mechanism for the GRouNdGAN image (yazdanz/groundgan:4b98686)
    used to run preprocessing and GRN creation.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-48516-6
    title: 'GRouNdGAN: GRN-guided simulation of single-cell RNA-seq data using causal
      generative adversarial networks'
  source_content_hash: sha256:673f750a1256033797fe2864e1d15f21df578ee745025935f5f0e734e3b274a4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Emad-COMBINE-lab/GRouNdGAN
  tools:
  - Beeline
  - Docker
  - GRNBoost2
  - GRouNdGAN
  - SPARSIM
  - scDESIGN2
  - scGAN
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# grn-inference-with-grnboost2

## Summary

Infers a gene regulatory network (GRN) from preprocessed scRNA-seq training data using GRNBoost2 via GRouNdGAN's --create_grn command, keeping the k=15 top TFs per gene and formatting the result as the pickled causal graph that guides GRouNdGAN's causal GAN.

## When to use

Use when preparing a GRN-guided causal GAN run: you have a preprocessed single-cell expression matrix (e.g., 1000 highly variable genes, library-size normalized to 20,000) and need a causal graph of TF-gene edges to impose during GRouNdGAN training.

## When NOT to use

- You already have a trusted, user-provided GRN to impose on GRouNdGAN — in that case skip --create_grn and use the imposing-custom-GRNs route.
- You need a multi-layer or TF-gene-phenotype causal graph — the current GRouNdGAN version only supports imposing bipartite GRNs.
- You are simulating unnormalized count data — the model generates expression values library-size normalized across the considered genes and TFs.

## Inputs

- GRouNdGAN preprocessed train set (single-cell expression data after --preprocess, 1000 highly variable genes)
- Transcription factor list (e.g., Homo_sapiens_TF.csv)
- Optionally, a user-provided custom GRN to impose instead of GRNBoost2 output

## Outputs

- Pickled causal graph (TF-gene regulatory network with k=15 top TFs per gene) consumed by GRouNdGAN --train
- Optionally, ground-truth GRN exported as CSV via the ground truth save path setting

## How to apply

After GRouNdGAN's --preprocess step (genes expressed in fewer than 3 cells and cells expressing fewer than 10 genes discarded; library-size normalized to 20,000; top 1000 highly variable genes retained), run the --create_grn command on the preprocessed train set. GRNBoost2 (Moerman et al., 2018) infers the regulatory network using a transcription factor list such as Homo_sapiens_TF.csv, and only the k=15 top TFs per gene are kept. The output is formatted as a causal graph and written as a pickled file that the causal_gan.cfg configuration consumes during --train. The article's authors recommend this pattern generally: derive regulatory edges from the reference dataset and impose a selected subset of them in the simulated data. If you have your own curated GRN, you can skip --create_grn entirely and impose the custom GRN instead.

## Related tools

- **GRouNdGAN** (Containerizes and invokes the GRN inference workflow via its --create_grn command, formatting GRNBoost2 output as a causal graph for causal GAN training.) — https://github.com/Emad-COMBINE-lab/GRouNdGAN
- **GRNBoost2** (Underlying GRN inference algorithm run by --create_grn to infer TF-gene regulatory edges from the preprocessed train set (Moerman et al., 2018).)
- **Beeline** (Benchmarking framework whose experimental GRN inference results the GRouNdGAN-based benchmark is aligned against.) — https://github.com/Murali-group/Beeline
- **Docker** (Distribution mechanism for the GRouNdGAN image (yazdanz/groundgan:4b98686) used to run preprocessing and GRN creation.)

## Examples

```
docker pull yazdanz/groundgan:4b98686; git clone https://github.com/Emad-COMBINE-lab/GRouNdGAN.git  # then run GRouNdGAN with --create_grn on the preprocessed train set
```

## Evaluation signals

- The --create_grn run completes on the preprocessed train set and writes a pickled causal graph at the configured path.
- The inferred graph contains only bipartite TF-gene edges, with at most k=15 top TFs per gene retained.
- The exported ground-truth GRN (if ground truth save path is set) is a valid CSV listing TF-gene relationships used for downstream --benchmark_grn evaluation.
- Training with the inferred causal graph yields simulated data whose resemblance metrics (e.g., RF AUROC close to 0.5, miLISI close to 2) indicate the imposed regulatory relationships do not severely deviate from the reference data patterns.
- A GRN inference method scored with --benchmark_grn against the imposed ground truth produces results aligned with BEELINE's experimental results.

## Limitations

- The underlying causal graph of an observational dataset is not necessarily unique; verifying that the imposed GRN is the most probable one is beyond the study's scope.
- GRouNdGAN's current version only allows imposing a bipartite GRN and does not support multi-layer GRNs or TF-gene-phenotype causal graphs.
- Resemblance of simulated data to the reference dataset deteriorates if the imposed TF-gene relationships are significantly different from those consistent with the patterns in the reference dataset.
- Hyperparameters may need re-tuning via a validation set when switching to a new dataset with significantly different numbers of genes and TFs.

## Evidence

- [methods] GRN inference via --create_grn: "--create_grn     Infer a GRN from preprocessed data using GRNBoost2 and appropriately format as causal graph"
- [methods] GRNBoost2 runs on the preprocessed train set: "This command uses GRNBoost2 (Moerman et al., 2018) to infer a GRN on the preprocessed train set."
- [discussion] Recommended workflow: infer edges then impose a subset: "we recommend first finding a set of regulatory edges from the reference dataset (e.g., using GRNBoost2 or even other data modalities)"
- [discussion] Only bipartite GRNs supported: "The current version of GRouNdGAN only allows imposing a bipartite GRN and does not support a multi-layer GRN"
