---
name: protein-level-network-module-detection
description: Use when matched mRNA, miRNA and protein expression data from tumor and
  normal samples are available and the goal is to identify RBP-ncRNA circuits in which
  ncRNAs modulate RBP function at the protein level.
when_to_use_negative:
- Only transcript-level data (no matched protein expression) is available — use the
  transcription-level workflow (interStringency type='transcription') instead
- The goal is a locally seeded module around a user-defined node — use runmodule(method='local',
  maxsize=..., seletN=...) rather than the global search
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3523
- http://edamontology.org/topic_3308
tools:
- name: fRNC
  role: Core R package providing interStringency, combinp, runmodule (global search)
    and saveNetwork for protein-level RNC detection
  repo: https://github.com/leiming8886/fRNC
- name: edgeR
  role: Used for the upstream differential expression analysis of miRNAs, mRNAs and
    lncRNAs whose logFC/PValue become node weights
- name: Cytoscape
  role: Visualization of the exported XGMML module file
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2023.03.035
    title: '<i>f</i> RNC: Uncovering the dynamic and condition-specific RBP-ncRNA
      circuits from multi-omics data'
  source_content_hash: sha256:8ac16505ad7063f9ce5b210987d4b07ec86b14fdbf1752e659342bd38a639911
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/BioinformaticsSTU/fRNC
  repo_url: https://github.com/leiming8886/fRNC
  tools:
  - Cytoscape
  - R
  - edgeR
  - fRNC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3523
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# protein-level-network-module-detection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Detect a maximum-scoring RBP-ncRNA circuit (RNC) where RBPs are represented at the protein level, using the fRNC R package on matched mRNA, miRNA and protein expression data (e.g. 852 tumor / 18 normal BRCA samples from TCGA/TCPA).

## When to use

Use when matched mRNA, miRNA and protein expression data from tumor and normal samples are available and the goal is to identify RBP-ncRNA circuits in which ncRNAs modulate RBP function at the protein level.

## When NOT to use

- Only transcript-level data (no matched protein expression) is available — use the transcription-level workflow (interStringency type='transcription') instead
- The goal is a locally seeded module around a user-defined node — use runmodule(method='local', maxsize=..., seletN=...) rather than the global search

## Inputs

- Pre-computed differential expression result objects for miRNA, mRNA and protein (brca_miRNA_re_se, brca_RNA_re_se, brca_pro_re_se) with genes, logFC, PValue, type columns
- Protein-level RBP-ncRNA interaction network from ENCORI (high stringency, CLIP evidence >= 3), type='Protein', species hg

## Outputs

- A global maximum-scoring protein-level RBP-ncRNA module (res.list_global$module)
- Node weight table (gene2weight) from combinp()
- XGMML network file for Cytoscape visualization

## How to apply

Load the pre-computed differential expression objects (brca_miRNA_re_se, brca_RNA_re_se, brca_pro_re_se) shipped with fRNC, extract the genes/logFC/PValue/type columns, and remove RBP-type mRNAs from the mRNA table so RBPs appear only as proteins. rbind the protein, non-RBP mRNA, and miRNA tables and convert them to node weights with combinp() on type/logFC/PValue with islog=TRUE. Retrieve the protein-level interaction network with interStringency(type='Protein', spec='hg', stringency='high') and keep node_gene_ID, type, target_gene_ID. Run the global search with runmodule(method='global', FDR=1e-20) and export the module with saveNetwork as XGMML for inspection in Cytoscape.

## Related tools

- **fRNC** (Core R package providing interStringency, combinp, runmodule (global search) and saveNetwork for protein-level RNC detection) — https://github.com/leiming8886/fRNC
- **edgeR** (Used for the upstream differential expression analysis of miRNAs, mRNAs and lncRNAs whose logFC/PValue become node weights)
- **Cytoscape** (Visualization of the exported XGMML module file)

## Examples

```
data("brca_miRNA_re_se"); data("brca_RNA_re_se"); data("brca_pro_re_se"); interac_RBP_age <- interStringency(type="Protein",spec="hg",stringency="high"); res.list_global_brca_e_20 <- runmodule(network=interac_RBP_age, gene2weight=gene2weight_age, method="global", FDR=1e-20)
```

## Evaluation signals

- runmodule with method='global' and FDR=1e-20 completes and returns a non-empty module (res.list_global_brca_e_20$module)
- RBP-type mRNAs are absent from the combined weight table (RNA_re_se_type$type != 'rbp' filter applied)
- interac_RBP_age contains only the columns node_gene_ID, type, target_gene_ID
- saveNetwork writes a valid XGMML file that opens in Cytoscape
- Node weights derive from combinp on type/logFC/PValue with islog=TRUE

## Limitations

- Requires matched multi-omics data simultaneously measuring mRNA, miRNA and protein expression (852 cancer / 18 normal samples in the BRCA case)
- Protein-level interactions are restricted to high-stringency ENCORI entries with CLIP experimental evidence >= 3
- The global search can be computationally heavy; the authors pre-computed differential expression and shipped internal data to save running time

## Evidence
