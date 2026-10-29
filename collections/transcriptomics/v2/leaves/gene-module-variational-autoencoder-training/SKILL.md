---
name: gene-module-variational-autoencoder-training
description: Use when a single-cell RNA-seq dataset (AnnData/scanpy) must be embedded
  in a biologically interpretable latent space defined by prior gene annotations (MSigDB,
  Reactome, GO, cell type markers), e.g. for pathway activity or drug-response interpretation.
when_to_use_negative:
- When no prior biological gene-module annotation (GMT) is available and an unrestricted
  latent representation suffices — VEGA's latent variables are hard-wired to user-provided
  gene modules
- When the priority is generative/predictive performance rather than interpretability
  — the sparse single-layer decoder sacrifices predictive performance for interpretability
- When the prior knowledge is likely incomplete or not context-specific and needs
  to be corrected during training — the hard-coded decoder connections leave no room
  for correcting prior knowledge
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0605
tools:
- name: VEGA
  role: Sparse VAE trained on the AnnData object with GMT-defined gene modules to
    produce interpretable GMV activities and Bayesian differential activity tests
  repo: https://github.com/LucasESBS/vega
- name: scanpy
  role: Wraps and preprocesses the single-cell dataset (filtering, normalization)
    before passing it to VEGA
- name: scVI
  role: scvi-tools ecosystem used alongside VEGA for training infrastructure and differential-expression-inspired
    testing
provenance:
  source_task_ids:
  - task_005
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-26017-0
    title: VEGA is an interpretable generative model for inferring biological network
      activity in single-cell transcriptomics
  source_content_hash: sha256:637e1a848aaea50f8c13ad88708844b548ecf025f239e3664fd2aac0c0cf8c96
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LucasESBS/vega
  tools:
  - MSigDB
  - VEGA
  - scVI
  - scanpy
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# gene-module-variational-autoencoder-training

## Summary

Train VEGA (VAE Enhanced by Gene Annotations), a sparse variational autoencoder whose decoder wiring mirrors user-provided gene modules (GMT files), to embed single-cell transcriptomics data into an interpretable latent space of gene module variational activities (GMVs) for pathways, transcription factor regulons, or cell type markers.

## When to use

Use when a single-cell RNA-seq dataset (AnnData/scanpy) must be embedded in a biologically interpretable latent space defined by prior gene annotations (MSigDB, Reactome, GO, cell type markers), e.g. for pathway activity or drug-response interpretation.

## When NOT to use

- When no prior biological gene-module annotation (GMT) is available and an unrestricted latent representation suffices — VEGA's latent variables are hard-wired to user-provided gene modules
- When the priority is generative/predictive performance rather than interpretability — the sparse single-layer decoder sacrifices predictive performance for interpretability
- When the prior knowledge is likely incomplete or not context-specific and needs to be corrected during training — the hard-coded decoder connections leave no room for correcting prior knowledge

## Inputs

- Single-cell RNA-seq dataset wrapped as a Scanpy AnnData object (e.g. Kang et al. PBMCs, Field et al. week-2 cortical organoid GSE106245, MIX-Seq drug-treatment data)
- GMT file(s) with gene module membership defining the latent space (MSigDB hallmark, Reactome pathways, GRNs, cell type marker sets)

## Outputs

- Trained VEGA model with a sparse linear decoder mirroring the input gene modules
- Per-cell gene module variational (GMV) activity scores at single-cell level
- Bayesian differential activity results (log_e Bayes Factors) per gene module per condition/cell type

## How to apply

Wrap the single-cell dataset as an AnnData object with scanpy and preprocess it (filter cells/genes, normalize) before passing it to VEGA, as the authors recommend. Supply one or more GMT files specifying gene module variables (GMVs) and gene membership — e.g. MSigDB hallmark sets, Reactome pathways, or cell type marker sets — which define the sparse linear decoder; for mixed tasks (e.g. cortical organoid cell-type disentangling) merge the pathway and marker GMTs into a single combined membership matrix. Instantiate and train the VEGA model (PyTorch, scvi-tools ecosystem, installable via pip as scvega) to infer per-cell GMV activities at single-cell resolution. After training, run VEGA's Bayesian differential activity testing (one-vs-rest for annotated cell types) and retain significant modules at |log_e(BF)| > 3; validate biologically, e.g. by correlating CELL_CYCLE_MITOTIC GMV activity against MKI67 expression (expected R2 ≈ 0.64) or checking that Trametinib decreases G2M checkpoint GMV activity. For exact paper reproduction, use the vega-reproducibility repository ('revision1' branch): download the preprocessed data archive, unpack with tar into ./data, run train_all_models.sh, then use the provided jupyter notebooks to reproduce figures.

## Related tools

- **VEGA** (Sparse VAE trained on the AnnData object with GMT-defined gene modules to produce interpretable GMV activities and Bayesian differential activity tests) — https://github.com/LucasESBS/vega
- **scanpy** (Wraps and preprocesses the single-cell dataset (filtering, normalization) before passing it to VEGA)
- **scVI** (scvi-tools ecosystem used alongside VEGA for training infrastructure and differential-expression-inspired testing)

## Examples

```
pip install scvega; import vega; model = vega.VEGA(adata, gmt_files=['hallmark.gmt','celltype_markers.gmt']); model.train(); gmv = model.get_latent(adata)
```

## Evaluation signals

- Trained model's GMVs correctly segregate annotated cell populations (e.g. cell-type marker-set GMVs in the week-2 cortical organoid data match original author annotations)
- One-vs-rest Bayesian differential activity tests yield significant modules at |log_e(BF)| > 3 for the expected cell types
- CELL_CYCLE_MITOTIC GMV activity correlates with MKI67 expression at R2 ≈ 0.64
- Trametinib treatment produces the expected decrease in G2M checkpoint GMV activity relative to DMSO, plus Trametinib-specific interferon-alpha/gamma response
- Dabrafenib shows larger average |log_e(BF)| in BRAF-mutant melanoma lines (0.763) than other lines (0.668)

## Limitations

- The sparse, single-layer decoder architecture limits generalizability and generative capacity, sacrificing predictive performance for biological interpretability
- Hard-coded decoder connections cannot correct prior knowledge about gene modules when the context requires it
- Prior biological knowledge from databases like MSigDB can be incomplete or not context-specific, since additional unannotated genes can play important roles in certain gene modules

## Evidence

- [abstract] Sparse VAE wiring mirrors GMT annotations: "we introduce a novel sparse Variational Autoencoder architecture, VEGA (VAE Enhanced by Gene Annotations), whose decoder wiring mirrors user-provided gene modules"
- [methods] GMV inference and Bayesian testing workflow: [evidence span withheld — anchor status: not_found]
- [readme] README usage requirements: [evidence span withheld — anchor status: not_found]
- [methods] Preprocessing recommendation: "Note: We recommend to preprocess the single-cell dataset before passing it to VEGA."
- [intro] Annotation databases defining latent space: "guided by gene module membership as recorded in gene annotation databases (e.g., Gene Ontology, PANTHER, MolSigDB, or Reactome)"
