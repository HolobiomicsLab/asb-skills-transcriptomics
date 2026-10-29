---
name: differential-co-expression-network-analysis
description: Use when you have matched two-condition gene expression matrices (rows
  = gene symbols, columns = sample IDs) plus a reference interaction network, and
  you need driver-node selection by network controllability rather than hub ranking.
when_to_use_negative:
- Only a single condition's expression data is available, so no two-state differential
  co-expression contrast (ΔZ statistic) can be computed
- The task is personalized driver discovery in a directed signalling network (e.g.
  DawnRank-style PageRank) rather than dominating-set controllability on an undirected
  co-expression network
- A naive degree-only hub ranking is the intended baseline — the article shows WMDS.net's
  advantage comes from the MDS formulation plus weights, not degree alone
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: WMDS.net
  role: Weighted minimum dominating set model applied to the differential co-expression
    network to find driver nodes
  repo: github.com/chaofen123/WMDS.net
- name: Matlab R2020b
  role: Runs the algorithm; solves the binary integer-programming problem with intlinprog
    from the Optimization ToolBox
- name: DawnRank
  role: Source of the Gene_Net hub gene-interaction network intersected with TCGA
    genes
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad071
    title: 'WMDS.net: a network control framework for identifying key players in transcriptome
      programs'
  source_content_hash: sha256:1fd04bf097c676927a39c61725ba6f0386cec421599b136ac90a5e86af9d8775
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DawnRank
  - MATLAB
  - Matlab R2020b
  - WMDS.net
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# differential-co-expression-network-analysis

## Summary

Builds a differential gene co-expression network between two physiological states (e.g. TCGA_BLCAtumor vs TCGA_BLCAnormal FPKM matrices) and identifies driver nodes via a weighted minimum dominating set (WMDS.net), solved as a binary integer program in MATLAB intlinprog with node weights w_i = (Σ −log p_ij)^(−γ), γ = 0.01.

## When to use

Use when you have matched two-condition gene expression matrices (rows = gene symbols, columns = sample IDs) plus a reference interaction network, and you need driver-node selection by network controllability rather than hub ranking.

## When NOT to use

- Only a single condition's expression data is available, so no two-state differential co-expression contrast (ΔZ statistic) can be computed
- The task is personalized driver discovery in a directed signalling network (e.g. DawnRank-style PageRank) rather than dominating-set controllability on an undirected co-expression network
- A naive degree-only hub ranking is the intended baseline — the article shows WMDS.net's advantage comes from the MDS formulation plus weights, not degree alone

## Inputs

- Two FPKM expression matrices, one per condition (rows = gene symbols, columns = sample IDs), e.g. TCGA_BLCAtumor and TCGA_BLCAnormal
- Reference hub interaction network (Gene_Net: DawnRank gene-interaction network intersected with TCGA genes)
- Per-edge differential co-expression P-values p_ij from the ΔZ statistic

## Outputs

- Differential co-expression network retaining significant differential edges (node set V, adjacency sets ∂i, node degrees)
- Driver gene list CSV (driver_gene_list.csv) of selected dominating-set nodes
- Binary solution vector x ∈ {0,1}^n from intlinprog

## How to apply

Compute Pearson correlations on FPKM expression for gene pairs connected in the reference network (Gene_Net, the DawnRank interaction network intersected with TCGA genes) per condition, Fisher Z-transform them, and form the differential co-expression network with the ΔZ statistic Δzij = (zij1 − zij2)/sqrt(1/(n1−3) + 1/(n2−3)) ~ N(0,1), retaining only significant differential edges. Compute each node's weight as w_i = (Σ_{j∈∂i} −log p_ij)^(−γ) with γ = 0.01, which makes the weighted dominating set retain the same node count as unweighted minimum dominating set configurations. Formulate the binary integer program minimizing f(x) = Σ_i w_i·x_i over x_i ∈ {0,1} subject to x_i + Σ_{j∈∂i} x_j ≥ 1 for every gene, and solve with intlinprog from the MATLAB R2020b Optimization ToolBox. Validate that the returned binary vector is a valid dominating set (every gene selected or adjacent to a selected gene) and write the selected driver gene symbols to driver_gene_list.csv.

## Related tools

- **WMDS.net** (Weighted minimum dominating set model applied to the differential co-expression network to find driver nodes) — github.com/chaofen123/WMDS.net
- **Matlab R2020b** (Runs the algorithm; solves the binary integer-programming problem with intlinprog from the Optimization ToolBox)
- **DawnRank** (Source of the Gene_Net hub gene-interaction network intersected with TCGA genes)

## Examples

```
A = -[eye(n) | Adj]'; b = -ones(n,1); x = intlinprog(w, 0, [], [], A, b, zeros(n,1), ones(n,1)); % WMDS dominating-set BIP
```

## Evaluation signals

- The returned binary vector x is a valid dominating set: every gene i satisfies x_i + Σ_{j∈∂i} x_j ≥ 1
- The weighted dominating set (γ = 0.01) has the same node count as unweighted MDS configurations (γ = 0, w_i = 1) on the same differential network
- Fβ (β² = 0.2) against the 616-gene CGC benchmark orders as WMDS.net > randomly selected unweighted MDS > degree-only baseline with the same number of genes
- Only gene pairs connected in the Gene_Net reference network enter the differential correlation computation; only significant differential edges are retained
- Driver-gene symbols in driver_gene_list.csv are a subset of genes present in the expression matrices and the Gene_Net node set

## Limitations

- γ = 0.01 is a fixed hyperparameter chosen so the weighted dominating set retains unweighted MDS node counts; other values may change set size
- Requires a reference interaction network (Gene_Net from DawnRank ∩ TCGA genes); differential edges are only computed for pairs connected in it
- Performance depends on the ΔZ statistic's N(0,1) assumption, i.e. sufficient per-condition sample sizes n1 and n2
- No discussion-section content is available in the extracted article to document further caveats

## Evidence

- [readme] readme: "We developed a weighted MDS network model (WMDS.net) to find the drive nodes of differential gene co-expression networks."
- [readme] readme_inputs: "The inputs to the main program are the expression matrices of the two conditions (which are TCGA_BLCAtumor and TCGA_BLCAnormal in the tested data), and the output is the gene list of key nodes in the"
- [intro] intro_intlinprog: "solve the binary integer-programming problem using function “intlinprog” which is available in the Optimization ToolBox of MatLab version R2020b"
- [abstract] abstract_performance: "WMDS.net is powerful among various cancer datasets and outperformed the other top-tier tools with a better balance between precision and recall"
