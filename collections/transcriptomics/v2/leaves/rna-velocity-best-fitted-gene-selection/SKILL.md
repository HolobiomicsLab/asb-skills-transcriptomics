---
name: rna-velocity-best-fitted-gene-selection
description: Use when a TFvelo run has completed (rc.h5ad output exists) and you need
  to extract -fitted genes, test KEGG pathway enrichment among them, and verify the
  sign consistency of top-weighted TFs on specific pathway gene sets.
when_to_use_negative:
- The dataset lacks suitable TF expression data or you have not decompressed the ENCODE
  TF-target database (ENCODE.zip) after downloading the package
- The analysis goal is splicing-based velocity (unspliced/spliced counts with scVelo
  dynamics) rather than TF-regulation-based dynamics; TFvelo is also slower than scVelo,
  a weakness for large-scale datasets
- You need a fitted model for every gene — TFvelo still cannot fit the dynamics of
  some genes well (Fig. S14), so genes not selected as best-fitted should not be force-analyzed
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: TFvelo
  role: Fits gene dynamics using TF expression, yielding best-fitted genes and the
    TF weight matrix in rc.h5ad
  repo: github.com/xiaoyeye/TFvelo
- name: gseapy
  role: Performs over-representation (Fisher exact) enrichment of the best-fitted
    gene list against the KEGG library
- name: scvelo
  role: Framework on which TFvelo is built; scvelo==0.2.4 is installed in the TFvelo
    environment
  repo: theislab/scvelo
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-45661-w
    title: 'TFvelo: gene regulation inspired RNA velocity estimation'
  source_content_hash: sha256:b05d101b5995ca206de0222477c0b9a8d5c9b27b8cb5c9eceb770ff94391eb78
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - TFvelo
  - gseapy
  - scvelo
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# rna-velocity-best-fitted-gene-selection

## Summary

Selects the best-fitted genes from a TFvelo RNA velocity run (stored in 'TFvelo_pancreas/rc.h5ad'), whose TF weight matrix is then used for KEGG over-representation enrichment and verification of TF weight-sign signatures (e.g. REST negative, HMGN3 positive on insulin secretion pathway genes in pancreas).

## When to use

Use when a TFvelo run has completed (rc.h5ad output exists) and you need to extract best-fitted genes, test KEGG pathway enrichment among them, and verify the sign consistency of top-weighted TFs on specific pathway gene sets.

## When NOT to use

- The dataset lacks suitable TF expression data or you have not decompressed the ENCODE TF-target database (ENCODE.zip) after downloading the package
- The analysis goal is splicing-based velocity (unspliced/spliced counts with scVelo dynamics) rather than TF-regulation-based dynamics; TFvelo is also slower than scVelo, a weakness for large-scale datasets
- You need a fitted model for every gene — TFvelo still cannot fit the dynamics of some genes well (Fig. S14), so genes not selected as best-fitted should not be force-analyzed

## Inputs

- TFvelo run output 'TFvelo_pancreas/rc.h5ad' (h5ad AnnData with best-fitted genes and TF weight matrix)
- Preprocessed scRNA-seq dataset (e.g. pancreas) ingested via the preprocess() function in TFvelo_run_demo.py
- Candidate TF lists from ENCODE and ChEA databases (ENCODE.zip must be decompressed)

## Outputs

- 'TFvelo_pancreas/rc.h5ad' with best-fitted genes and TF weights
- 'TFvelo_pancreas_demo/TFvelo.h5ad' plus figures in folder 'figures' (pseudotime, streamplot, phase portrait fits)
- 'KEGG_enrichment_results.csv' with enrichment table and REST/HMGN3 weight-signature verification results

## How to apply

Run 'python TFvelo_run_demo.py --dataset_name pancreas', which downloads, preprocesses and runs the TF model, storing the result in 'TFvelo_pancreas/rc.h5ad'; this file contains the best-fitted genes and their TF weight matrix. The TFvelo analysis demo then reports pseudotime, streamplot on UMAP and the phase portrait fitting of best fitted genes, which is how best-fitted genes are identified. Run over-representation (Fisher exact test) enrichment of the best-fitted gene list with gseapy.enrichr against the KEGG library to confirm terms such as insulin secretion and glucagon signaling. Restrict the TF weight matrix to genes in the insulin secretion pathway, normalize TF weights, and flag TFs whose absolute normalized weight exceeds 0.5; verify expected signs (REST negative, HMGN3 positive). Weight optimization uses lsq_linear with WX_thres defining the max absolute value for weights.

## Related tools

- **TFvelo** (Fits gene dynamics using TF expression, yielding best-fitted genes and the TF weight matrix in rc.h5ad) — github.com/xiaoyeye/TFvelo
- **gseapy** (Performs over-representation (Fisher exact) enrichment of the best-fitted gene list against the KEGG library)
- **scvelo** (Framework on which TFvelo is built; scvelo==0.2.4 is installed in the TFvelo environment) — theislab/scvelo

## Examples

```
python TFvelo_run_demo.py --dataset_name pancreas
```

## Evaluation signals

- Insulin secretion and glucagon signaling KEGG terms appear as enriched terms in the Fisher-exact (gseapy.enrichr) enrichment table
- REST exhibits consistently negative weights and HMGN3 consistently positive weights on insulin secretion pathway genes
- Both REST and HMGN3 exceed the normalized |weight| > 0.5 threshold among high-absolute-weight regulators
- Run artifacts exist at the documented paths ('TFvelo_pancreas/rc.h5ad', 'TFvelo_pancreas_demo/TFvelo.h5ad', 'figures' folder)
- Phase portrait fits of best-fitted genes show a desired clockwise curve between the TFs representation and the target

## Limitations

- TFvelo is slower than scVelo baselines in the same framework, which could be a weakness when being applied to large-scale datasets
- TFvelo still cannot fit the dynamics of some genes well (Fig. S14); improving modeling for more genes remains a challenge
- The current linear model for estimating TF weights may be insufficient; no implementation of alternatives is provided

## Evidence

- [readme] readme run output: "This will automatically download, preprocess and run TF model on pancrease dataset. The result will be stored in 'TFvelo_pancreas/rc.h5ad'."
- [methods] phase portrait fitting: "This will show the pseudotime and streamplot on UMAP, and also the phase portrait fitting of best fitted genes."
- [readme] weight threshold parameter: "--WX_thres: The max absolute value for weights."
- [discussion] fitting limitation: "Meanwhile, TFvelo still cannot fit the dynamics of some genes well (Fig. S14)."
- [discussion] clockwise phase portrait: "TFvelo can fit the gene dynamics well, showing a desired clockwise curve on the phase portrait between the TFs representation and the target"
