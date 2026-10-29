---
name: negative-binomial-count-likelihood-modeling
description: Use when raw single-cell count matrices (RNA_count.mtx, Protein_count.mtx)
  feed a deep generative model and you want likelihoods that match overdispersed count
  distributions, avoiding log-normalization preprocessing.
when_to_use_negative:
- Input matrices are already normalized (log-normalized, CPM, or scaled values) —
  the NB likelihood expects raw integer counts and normalization breaks the generative
  model
- Modalities are non-count data (e.g. images or continuous measurements), where a
  Gaussian or Bernoulli likelihood from the MMVAE framework is more appropriate
- The goal is differential expression or classical count modeling only, not end-to-end
  latent representation learning across modalities
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scMM
  role: Framework whose modality-specific decoders implement the negative-binomial
    likelihoods on raw CITE-seq count data
  repo: https://github.com/kodaim1115/scMM
- name: MMVAE
  role: Reference PyTorch implementation from which scMM's likelihood and mixture-of-experts
    code were adopted
  repo: https://github.com/iffsid/mmvae
- name: Google Colab
  role: GPU environment used to run the end-to-end NB-likelihood training via colab_tutorial.ipynb
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100071
    title: A mixture-of-experts deep generative model for integrated analysis of single-cell
      multiomics data
  source_content_hash: sha256:f835c8af1aedb041514d64c8dd1f12e7adad7b744345aabff1b799435e844b45
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/kodaim1115/scMM
  repo_url: https://github.com/kodaim1115/scMM
  tools:
  - Google Colab
  - MMVAE
  - R
  - scMM
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# negative-binomial-count-likelihood-modeling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Pairs each modality-specific decoder of the scMM mixture-of-experts VAE with a negative-binomial likelihood over raw integer counts (e.g. CITE-seq transcriptome and surface-protein counts), enabling end-to-end training without pre-normalizing the data.

## When to use

Use when raw single-cell count matrices (RNA_count.mtx, Protein_count.mtx) feed a deep generative model and you want likelihoods that match overdispersed count distributions, avoiding log-normalization preprocessing.

## When NOT to use

- Input matrices are already normalized (log-normalized, CPM, or scaled values) — the NB likelihood expects raw integer counts and normalization breaks the generative model
- Modalities are non-count data (e.g. images or continuous measurements), where a Gaussian or Bernoulli likelihood from the MMVAE framework is more appropriate
- The goal is differential expression or classical count modeling only, not end-to-end latent representation learning across modalities

## Inputs

- Raw scRNA-seq count matrix (RNA_count.mtx) with gene.tsv and barcode.tsv feature/barcode files
- Raw CITE-seq surface-protein count matrix (Protein_count.mtx) with protein.tsv and barcode.tsv
- Optional ATAC-seq peak count matrix (peak counts) with peak.tsv, for chromatin-accessibility modality

## Outputs

- Trained model parameters (scmm_model.pt) whose decoders are negative-binomial heads over raw counts
- Inferred joint latent representations per cell (latent_embeddings.tsv)
- Per-modality NB reconstruction parameters (mean/dispersion) usable for crossmodal generation

## How to apply

Attach one decoder network per modality (transcriptome, surface protein) that emits negative-binomial parameters for the raw count vector, so the ELBO sums per-modality NB log-likelihoods on unnormalized integer data. Keep the raw counts as loaded — scMM 'achieves end-to-end learning by modeling raw count data in each modality based on different probability distributions', so do not log-normalize or scale the matrices before training. Combine the NB decoders with modality-specific encoders, an equally-weighted mixture-of-experts posterior, and a Laplace prior with learnable per-dimension scale, then train end-to-end on the 15,000-cell BMNC CITE-seq subsample (5,000 most variable genes for the transcriptome). If modalities differ greatly in dimensionality and one likelihood dominates the bound, consider a likelihood-scaling factor as done in the parent MMVAE implementation (--llik_scaling).

## Related tools

- **scMM** (Framework whose modality-specific decoders implement the negative-binomial likelihoods on raw CITE-seq count data) — https://github.com/kodaim1115/scMM
- **MMVAE** (Reference PyTorch implementation from which scMM's likelihood and mixture-of-experts code were adopted) — https://github.com/iffsid/mmvae
- **Google Colab** (GPU environment used to run the end-to-end NB-likelihood training via colab_tutorial.ipynb)

## Examples

```
torch.save(model.state_dict(), "scmm_model.pt"); pd.DataFrame(model.get_latent(rna_counts, protein_counts)).to_csv("latent_embeddings.tsv", sep="\t")
```

## Evaluation signals

- Training ELBO (sum of per-modality NB log-likelihoods, KL terms) decreases monotonically toward convergence with no NaN losses
- scmm_model.pt and latent_embeddings.tsv are produced, loadable, and contain one latent embedding per input cell barcode
- No normalization step was applied upstream: decoder inputs equal the raw integer values in RNA_count.mtx and Protein_count.mtx
- Each modality's decoder uses its own count-appropriate distribution, so transcriptome and protein reconstructions are both valid overdispersed count predictions
- Crossmodal generation from the trained NB decoders yields count-valued predictions that support downstream integration quality checks (e.g. against the R/tutorial.R workflow)

## Limitations

- The approach presumes raw count data; feeding pre-normalized or transformed values invalidates the negative-binomial likelihood and the end-to-end rationale
- Modalities with very different dimensionalities (e.g. 5,000 genes vs a small protein panel) can contribute unevenly to the ELBO, requiring likelihood scaling as in the MMVAE codebase
- Training a full deep generative model end-to-end offers limited interpretability, which scMM addresses separately via pseudocell generation

## Evidence

- [readme] Likelihood scaling option in parent MMVAE code: [evidence span withheld — anchor status: not_found]
