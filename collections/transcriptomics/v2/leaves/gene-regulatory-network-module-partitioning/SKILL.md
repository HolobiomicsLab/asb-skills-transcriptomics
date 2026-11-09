---
name: gene-regulatory-network-module-partitioning
description: Use when you have a merged, edge-pruned GRN from single-cell or spatial
  transcriptomics data (e.g., snRNAseq or Visium) and need to identify gene modules,
  compute per-cell/spot module scores, and link modules to traits or pathways.
when_to_use_negative:
- The input network has not yet been built and edge-pruned by the SCING BuildNetwork
  step (supercells, subsampled GRN merging, conditional mutual information pruning
  must come first).
- The analysis goal is to infer the GRN itself or to utilize spatial coordinates explicitly
  during network construction — SCING GRN construction does not currently use spatial
  information.
- Causal or directional regulatory relationships are required — SCING GRN links are
  not necessarily causal or directional.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3308
tools:
- name: SCING
  role: Provides the merged, edge-pruned GRN that is partitioned into leiden subnetworks
  repo: https://github.com/XiaYangLabOrg/SCING
- name: leiden
  role: Graph partitioning algorithm used to identify GRN subnetwork modules
- name: SCENIC
  role: Its AUCell method (via pyscenic) computes module-specific expression scores
    for each cell or spatial spot
- name: enrichR
  role: R-based pathway enrichment of subnetwork genes against GO BP, DisGeNET, Reactome,
    BioCarta, and KEGG for module annotation
provenance:
  source_task_ids:
  - task_006
  - task_008
  source_papers:
  - doi: 10.1016/j.isci.2023.107124
    title: 'SCING: Inference of robust, interpretable gene regulatory networks from
      single cell and spatial transcriptomics'
  source_content_hash: sha256:16cdda0986bb9038e93b411a49af2c1e655abba27e5614dff0b88aa7eff06039
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiaYangLabOrg/SCING
  tools:
  - SCENIC
  - SCING
  - enrichR
  - leiden
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# gene-regulatory-network-module-partitioning

## Summary

Partition a merged, edge-pruned SCING gene regulatory network into gene subnetwork modules using the leiden graph partitioning algorithm, then score module-specific expression per cell with the AUCell method from SCENIC to enable clustering, phenotypic association, and pathway-based biological annotation.

## When to use

Use when you have a merged, edge-pruned GRN from single-cell or spatial transcriptomics data (e.g., snRNAseq or Visium) and need to identify gene modules, compute per-cell/spot module scores, and link modules to traits or pathways.

## When NOT to use

- The input network has not yet been built and edge-pruned by the SCING BuildNetwork step (supercells, subsampled GRN merging, conditional mutual information pruning must come first).
- The analysis goal is to infer the GRN itself or to utilize spatial coordinates explicitly during network construction — SCING GRN construction does not currently use spatial information.
- Causal or directional regulatory relationships are required — SCING GRN links are not necessarily causal or directional.

## Inputs

- Merged, edge-pruned SCING GRN (output of BuildNetwork.ipynb)
- snRNAseq expression matrix with per-cell clinical metadata (e.g., diagnosis, plaque stage, tangle stage, sex) or Visium spatial count matrix with spot metadata (e.g., beta-amyloid plaque staining annotations, mouse age)
- pyscenic installation for AUCell scoring

## Outputs

- Leiden subnetwork module assignments (gene-to-module mapping)
- Module-specific AUCell expression scores per cell or spatial spot
- Module-trait association statistics (regression coefficients, FDR-corrected p-values, or Pearson correlations vs permutation null)
- Module pathway annotations (module_pathway_annotations.csv with enrichR terms from GO BP, DisGeNET, Reactome, BioCarta, KEGG)

## How to apply

Take the final GRN produced by the SCING BuildNetwork step (supercells, ~100 subsampled GRNs merged, edges and cycles pruned via conditional mutual information) and apply the leiden graph partitioning algorithm to divide it into subnetworks (e.g., 33 modules were found for the mouse Visium GRN). Combine each leiden subnetwork with the AUCell method from SCENIC (pyscenic) to compute module-specific expression scores for every cell or spatial spot in the dataset. Use the resulting module score matrix for clustering, phenotypic association (e.g., linear regression of module scores on AD diagnosis, plaque stage, and tangle stage with sex regressed out, Benjamini-Hochberg FDR < 0.05; or Pearson correlation with plaque burden against a 1,000 random gene set null with Bonferroni correction), and biological annotation via enrichR pathway enrichment (GO biological process, DisGeNET, Reactome, BioCarta, KEGG) on each subnetwork's genes.

## Related tools

- **SCING** (Provides the merged, edge-pruned GRN that is partitioned into leiden subnetworks) — https://github.com/XiaYangLabOrg/SCING
- **leiden** (Graph partitioning algorithm used to identify GRN subnetwork modules)
- **SCENIC** (Its AUCell method (via pyscenic) computes module-specific expression scores for each cell or spatial spot)
- **enrichR** (R-based pathway enrichment of subnetwork genes against GO BP, DisGeNET, Reactome, BioCarta, and KEGG for module annotation)

## Examples

```
conda env create -n scing --file install/scing.environment.yml; conda activate scing; pip install pyitlib; pip install -e .; pip install pyscenic; jupyter notebook tutorials/ModuleBasedDimensionalityReduction.ipynb
```

## Evaluation signals

- The merged GRN is fully partitioned: every gene in the pruned network is assigned to exactly one leiden subnetwork (e.g., 33 modules for the mouse Visium GRN).
- AUCell module scores are produced for every cell in the dataset (or every spatial spot) and can be used for clustering or phenotypic association without missing values.
- Module-trait associations are called significant only after multiple-testing correction: Benjamini-Hochberg FDR < 0.05 across modules for regression, or Bonferroni-corrected Pearson correlation against a null of 1,000 random gene sets for plaque/age correlations.
- Module annotation is accepted only when enrichR returns terms from the specified pathway knowledge bases (GO biological process, DisGeNET, Reactome, BioCarta, KEGG).
- Biological plausibility check: plaque-associated modules reflect expected AD biology (e.g., module 25 enriched for microglial activation, lysosome, and cell migration).

## Limitations

- SCING GRN links are not necessarily causal or directional, so module interpretations are associative.
- Spatial information from spatial transcriptomics is not explicitly utilized during GRN construction, even though resulting GRNs show spatial patterns; explicitly leveraging it is deferred to future work.
- Many hyperparameters (e.g., leiden resolution) require selection, and while robustness across a range of hyperparameters was shown, selection can be dataset dependent.
- Additional omics modalities (scATAC-seq, scHi-C, ChIP-seq, cis/trans eQTL) are not incorporated and could improve GRN and module accuracy.

## Evidence

- [methods] We apply the leiden graph partitioning algorithm to identify GRN subnetworks: "We apply the leiden graph partitioning algorithm to identify GRN subnetworks"
- [methods] We combine these subnetworks with the AUCell method to get module specific expression for each cell: "We combine these subnetworks with the AUCell method to get module specific expression for each cell"
- [readme] ModuleBasedDimensionalityReduction.ipynb: Take the merged network, and divide it into subnetworks with the leiden graph partitioning algorithm, finally run the AUCell approach to get module scores for each cell: "ModuleBasedDimensionalityReduction.ipynb: Take the merged network, and divide it into subnetworks with the leiden graph partitioning algorithm, finally run the AUCell approach to get module scores"
- [readme] PathwayEnrichmentOfModule.ipynb: Run pathways enrichment analysis on the genes in each subnetowrk determined from the leiden algorithm.: "PathwayEnrichmentOfModule.ipynb: Run pathways enrichment analysis on the genes in each subnetowrk determined from the leiden algorithm. This enables biological annotation of modules."
- [intro] SCING GRNs are then partitioned into modules, to compute module specific expression for each cell. These modules can be used for clustering, phenotypic association, and biological annotation: "SCING GRNs are then partitioned into modules, to compute module specific expression for each cell. These modules can be used for clustering, phenotypic association, and biological annotation"
- [discussion] the links between genes are not necessarily causal or directional: "the links between genes are not necessarily causal or directional"
