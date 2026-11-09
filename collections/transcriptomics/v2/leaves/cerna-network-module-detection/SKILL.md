---
name: cerna-network-module-detection
description: Use when you have case/control RNA-seq count matrices (e.g. TCGA tumor
  vs normal) plus an RBP-ncRNA interaction network (ENCORI CLIP-seq evidence) and
  need to identify condition-specific, globally or locally optimal RBP-ncRNA sub-networks.
when_to_use_negative:
- Your interaction evidence is predicted (non-CLIP/PARE) or lacks the required experimental
  support (ENCORI high stringency requires >=3 CLIP experiments)
- You have only expression data with no RBP-ncRNA interaction network — fRNC scores
  nodes on an existing experimentally derived network, not a co-expression network
  like WGCNA
- You need protein-level RNCs but lack matched mRNA-miRNA-protein expression from
  the same samples
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0659
tools:
- name: fRNC
  role: 'Core R package: DEGs scoring, combinp weighting, interStringency network
    filtering, and runmodule global/local module detection'
  repo: https://github.com/BioinformaticsSTU/fRNC
- name: edgeR
  role: Differential expression analysis of miRNAs, mRNAs and lncRNAs used to score
    nodes
- name: devtools
  role: Installation of fRNC from GitHub (devtools::install_github)
  repo: https://github.com/leiming8886/fRNC
- name: Cytoscape
  role: Visualization of saved XGMML RNC module results
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2023.03.035
    title: '<i>f</i> RNC: Uncovering the dynamic and condition-specific RBP-ncRNA
      circuits from multi-omics data'
  source_content_hash: sha256:94d7a0de965d28bd9ce05fb230999362b4a99e855edf2167b4c614b68538222f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/BioinformaticsSTU/fRNC
  repo_url: https://github.com/BioinformaticsSTU/fRNC
  tools:
  - Cytoscape
  - R
  - devtools
  - edgeR
  - fRNC
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 20
---

# ceRNA-network-module-detection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Detect RBP-ncRNA circuit (RNC) modules by integrating experimentally derived CLIP-seq/PARE interaction networks with differential-expression scores using the fRNC R package, via either global maximum-scoring module search or greedy local search from user-defined seed nodes.

## When to use

Use when you have case/control RNA-seq count matrices (e.g. TCGA tumor vs normal) plus an RBP-ncRNA interaction network (ENCORI CLIP-seq evidence) and need to identify condition-specific, globally or locally optimal RBP-ncRNA sub-networks.

## When NOT to use

- Your interaction evidence is predicted (non-CLIP/PARE) or lacks the required experimental support (ENCORI high stringency requires >=3 CLIP experiments)
- You have only expression data with no RBP-ncRNA interaction network — fRNC scores nodes on an existing experimentally derived network, not a co-expression network like WGCNA
- You need protein-level RNCs but lack matched mRNA-miRNA-protein expression from the same samples

## Inputs

- Case and control RNA-seq count matrices for miRNA and mRNA/lncRNA (e.g. case.exp_miRNA, control.exp_miRNA, case.exp_rna, control.exp_rna from TCGA ESCA; BRCA mRNA/miRNA/protein with 852 cancer and 18 normal samples)
- Pre-computed differential expression results as SummarizedExperiment-like objects (brca_miRNA_re_se, brca_RNA_re_se, brca_pro_re_se)
- RBP-ncRNA interaction network from CLIP-seq/PARE data (ENCORI, high stringency, CLIP evidence >= 3)

## Outputs

- Global maximum significant RNC module (res.list_global$module)
- Local maximum-scoring RNC modules seeded at user-defined nodes (res.list_local)
- XGMML network file (e.g. ceRNA_module_transcription) for Cytoscape visualization

## How to apply

First run differential expression on miRNA, mRNA/lncRNA (and optionally protein) count matrices with the fRNC DEGs function (data_type='RNAseq_counts') using edgeR, then convert logFC/PValue columns into node weights with combinp. Build the interaction network with interStringency (e.g. type='transcription' or 'Protein', spec='hg', stringency='high', meaning >=3 CLIP experimental evidence records from ENCORI) and keep columns node_gene_ID, type, target_gene_ID. Detect modules with runmodule: method='global' finds the RNC containing the global maximum significant RBPs and ncRNAs at a chosen FDR (1e-5 for ESCA transcription-level; 1e-20 for the BRCA protein-level case), while method='local' with maxsize (e.g. 15) and seletN seed node (e.g. 'MIMAT0000089') performs greedy search for locally maximum-scoring modules. Save global modules as XGMML via saveNetwork for Cytoscape visualization and local modules via savelocalM; benchmarked sensitivity and 100-fold half-sample resampling robustness show fRNC outperforms WGCNA and LncACT on SimSeq-simulated data.

## Related tools

- **fRNC** (Core R package: DEGs scoring, combinp weighting, interStringency network filtering, and runmodule global/local module detection) — https://github.com/BioinformaticsSTU/fRNC
- **edgeR** (Differential expression analysis of miRNAs, mRNAs and lncRNAs used to score nodes)
- **devtools** (Installation of fRNC from GitHub (devtools::install_github)) — https://github.com/leiming8886/fRNC
- **Cytoscape** (Visualization of saved XGMML RNC module results)

## Examples

```
interac_rbp <- interStringency(type = "transcription", spec = "hg", stringency = "high"); res.list_global <- runmodule(network = interac_rbp, gene2weight, method = "global", FDR = 1e-5)
```

## Evaluation signals

- runmodule with method='global' converges at the specified FDR (1e-5 for ESCA transcription-level; 1e-20 for BRCA protein-level) and returns res.list_global$module non-empty
- Local search respects maxsize (e.g. 15) and returns a module containing the specified seed node (e.g. MIMAT0000089)
- Sensitivity against embedded true sub-networks on SimSeq simulations (fold ratios 0.5/1/2/5, sizes 30/50/70 nodes) exceeds WGCNA and LncACT per ANOVA comparison
- Sub-network detection recovers consistent modules across 100 half-sample resampled datasets (robustness consistency)
- Saved XGMML opens in Cytoscape and network columns are node_gene_ID, type, target_gene_ID

## Limitations

- Interactions restricted to ENCORI entries with >=3 supported CLIP experimental evidence at high stringency, which may exclude weaker/less-studied interactions
- For large BRCA data (852 cancer/18 normal samples), the package authors pre-computed differential expression internally to save running time, implying runtime cost on large datasets
- The benchmarked advantage over WGCNA and LncACT was established on SimSeq-simulated data; discussion-section content validating real-data comparisons is not available in the provided context

## Evidence

- [methods] Global maximum significant RNC detection: "res.list_global <- runmodule(network = interac_rbp, gene2weight, method = "global", FDR = 1e-5)"
- [methods] Local maximum RNC detection with seed node: "res.list_local <- runmodule(network = interac_rbp, gene2weight, method = "local", maxsize=15, seletN = c("MIMAT0000089") )"
- [methods] High-stringency interaction filtering rationale: "were extracted from ENCORI with a high stringency, where the number of the supported CLIP experimental evidence is 3 or greater."
- [abstract] Comparison finding vs existing tools: "When compared with existing tools, f RNC can detect more accurate and robust sub-network with scalability."
- [readme] Module saving and visualization workflow: "saveNetwork(res.list_global$module,file="ceRNA_module_transcription",type = "XGMML")"
