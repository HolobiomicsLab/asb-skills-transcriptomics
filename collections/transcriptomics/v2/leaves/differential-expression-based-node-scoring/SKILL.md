---
name: differential-expression-based-node-scoring
description: Use when you have case/control RNA-seq count matrices (e.g. TCGA ESCA
  tumor vs normal) and need to weight nodes of a CLIP-derived RBP-ncRNA network before
  running fRNC's global or local module search.
when_to_use_negative:
- Expression data are not case/control count matrices but already-processed statistics
  — skip DEGs and feed existing type/logFC/PValue values directly to combinp (as done
  for the BRCA case with brca_miRNA_re_se, brca_RNA_re_se, brca_pro_re_se).
- The goal is protein-level integration without expression weighting — the BRCA protein-level
  case requires removing RBP mRNA rows (type != "rbp") before combining, so naive
  rbind of all DE results would double-score RBPs.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_3170
tools:
- name: fRNC
  role: R package providing the DEGs and combinp functions used to compute differential
    expression and convert it into node weight scores
  repo: https://github.com/BioinformaticsSTU/fRNC
- name: edgeR
  role: underlying differential expression engine wrapped by fRNC's DEGs function
    for RNAseq_counts data
- name: R
  role: execution environment for the fRNC scoring workflow
  repo: https://github.com/BioinformaticsSTU/fRNC
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2023.03.035
    title: '<i>f</i> RNC: Uncovering the dynamic and condition-specific RBP-ncRNA
      circuits from multi-omics data'
  source_content_hash: sha256:a76a384bbd407600f443c0a87ceef74f3a1e8e9ac1e1c2746ac0a37ee0e32cd9
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
  - R
  - edgeR
  - fRNC
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# differential-expression-based-node-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Convert differential expression results (type, logFC, PValue) for miRNAs, mRNAs and lncRNAs into node weight scores on an RBP-ncRNA interaction network using the fRNC 'combinp' function, so that downstream RNC module searches can rank nodes by expression significance.

## When to use

Use when you have case/control RNA-seq count matrices (e.g. TCGA ESCA tumor vs normal) and need to weight nodes of a CLIP-derived RBP-ncRNA network before running fRNC's global or local module search.

## When NOT to use

- Expression data are not case/control count matrices but already-processed statistics — skip DEGs and feed existing type/logFC/PValue values directly to combinp (as done for the BRCA case with brca_miRNA_re_se, brca_RNA_re_se, brca_pro_re_se).
- The goal is protein-level integration without expression weighting — the BRCA protein-level case requires removing RBP mRNA rows (type != "rbp") before combining, so naive rbind of all DE results would double-score RBPs.

## Inputs

- case and control expression count matrices (e.g. case.exp_miRNA, control.exp_miRNA, case.exp_rna, control.exp_rna from the packaged ESCA TCGA dataset)
- differential expression result data frame with genes, logFC, PValue and type columns
- RBP-ncRNA interaction network from ENCORI (CLIP evidence >= 3, high stringency) with node_gene_ID/type/target_gene_ID columns

## Outputs

- gene2weight named node weight vector/score table mapping each miRNA, mRNA and lncRNA to a combined differential expression significance score
- scored network ready for runmodule(method = "global" or "local")

## How to apply

Run the fRNC 'DEGs' function separately on case and control expression matrices (e.g. case.exp_miRNA vs control.exp_miRNA, case.exp_rna vs control.exp_rna) with data_type = "RNAseq_counts", which wraps edgeR to compute differential expression of miRNAs, mRNAs and lncRNAs. Rbind the miR and RNA DE result frames and pass only the type/logFC/PValue columns to 'combinp' with islog = TRUE to obtain the gene2weight node score vector. This scoring 'on nodes and edges according to differential analysis of expression data' is what allows 'runmodule' to find an RNC containing the global maximum of significant RBPs and ncRNAs. Keep the geneid parameter aligned to rownames of the control expression matrix so weight names match the network's node_gene_ID/target_gene_ID identifiers.

## Related tools

- **fRNC** (R package providing the DEGs and combinp functions used to compute differential expression and convert it into node weight scores) — https://github.com/BioinformaticsSTU/fRNC
- **edgeR** (underlying differential expression engine wrapped by fRNC's DEGs function for RNAseq_counts data)
- **R** (execution environment for the fRNC scoring workflow) — https://github.com/BioinformaticsSTU/fRNC

## Examples

```
result_miR <- DEGs(case.exp_miRNA,control.exp_miRNA, geneid=rownames(control.exp_miRNA), data_type="RNAseq_counts"); gene2weight <- combinp(rbind(result_miR$DEGs,result_rna$DEGs)[,c("type","logFC","PValue")], islog=T)
```

## Evaluation signals

- gene2weight entries are named with identifiers matching the network's node_gene_ID/target_gene_ID values (e.g. MIMAT0000089 for miRNAs).
- combinp was called with exactly the type, logFC and PValue columns and islog = TRUE.
- DEGs was called with data_type = "RNAseq_counts" and geneid = rownames(control.exp_*) so sample orientation is correct.
- The scored network yields non-empty modules from runmodule (e.g. global with FDR = 1e-5 on ESCA), confirming weights overlap the network nodes.
- No duplicate scoring of RBPs when mixing mRNA and protein DE results (RBP rows removed via type != "rbp" in the BRCA protein-level case).

## Limitations

- High-stringency ENCORI filtering (CLIP experimental evidence >= 3) restricts the network, so scores for genes outside it are unused.
- For proteomics-integrated cases (BRCA), mRNA expression of RBPs must be removed before combining, otherwise RBP nodes receive conflicting scores.
- Scoring depends on differential significance only; the paper reports results at specific FDR thresholds (1e-5 for ESCA global, 1e-20 for BRCA global), and module outcome varies with the chosen FDR.

## Evidence
