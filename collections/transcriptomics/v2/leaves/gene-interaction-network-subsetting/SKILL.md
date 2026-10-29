---
name: gene-interaction-network-subsetting
description: Use when building a differential co-expression network from TCGA RNA-seq
  expression matrices and a prior gene-interaction network is available to restrict
  which gene-pair correlations are computed and tested.
when_to_use_negative:
- The expression matrix gene symbols have not been mapped/intersected with the DawnRank
  interaction network genes — the subgraph only makes sense on the Gene_Net gene space
- No prior gene-interaction network is available, so an unrestricted all-pairs co-expression
  analysis is intended
- The analysis is a single-condition network without a two-condition differential
  comparison, where the WMDS.net edge-significance framework does not apply
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: WMDS.net
  role: Framework whose downstream differential co-expression and SSN computations
    consume the subsetted largest-connected Gene_Net subgraph as the candidate edge
    space
  repo: https://github.com/chaofen123/WMDS.net
- name: DawnRank
  role: Source of the gene-interaction network from which Gene_Net (hub of its intersection
    with TCGA genes) is constructed
- name: Matlab R2020b
  role: Implementation environment in which the network handling and subsequent integer-programming
    steps are run (intlinprog in the Optimization ToolBox)
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad071
    title: 'WMDS.net: a network control framework for identifying key players in transcriptome
      programs'
  source_content_hash: sha256:26c0f35e759bf6ed47f05bd5e79693756d21e79b00633ffec50fdf30a7789d95
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chaofen123/WMDS.net
  tools:
  - DawnRank
  - Matlab R2020b
  - WMDS.net
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# gene-interaction-network-subsetting

## Summary

Restricts the DawnRank-derived Gene_Net hub network (intersection of the DawnRank gene-interaction network with TCGA genes) to its largest connected subgraph, defining the candidate gene-pair space on which WMDS.net differential co-expression and SSN analyses are computed.

## When to use

Use when building a differential co-expression network from TCGA RNA-seq expression matrices and a prior gene-interaction network is available to restrict which gene-pair correlations are computed and tested.

## When NOT to use

- The expression matrix gene symbols have not been mapped/intersected with the DawnRank interaction network genes — the subgraph only makes sense on the Gene_Net gene space
- No prior gene-interaction network is available, so an unrestricted all-pairs co-expression analysis is intended
- The analysis is a single-condition network without a two-condition differential comparison, where the WMDS.net edge-significance framework does not apply

## Inputs

- DawnRank-derived Gene_Net reference network (11,648 genes, 211,794 edges; intersection of DawnRank gene-interaction network with TCGA genes)
- FPKM-normalized TCGA expression matrices (rows = gene symbols, columns = sample ids; e.g. TCGA_BLCAtumor and TCGA_BLCAnormal)

## Outputs

- Largest connected subgraph of Gene_Net with an accompanying retained gene list used to restrict candidate gene pairs
- Restricted edge set over which ΔZ (population mode) or Δr (SSN mode) tests and P-values are subsequently computed

## How to apply

Load the Dawnrank-derived Gene_Net reference network, which is the hub network of the intersection between the DawnRank gene-interaction network and the genes present in TCGA (reported as 11,648 genes and 211,794 edges). Retain only the largest connected subgraph and discard nodes with no edge connection to the central part, so that all downstream node controllability calculations operate on a single connected component. Use the surviving gene list to restrict candidate gene pairs when computing Pearson correlations on FPKM-normalized expression matrices (rows = gene symbols, columns = sample ids) separately for the two conditions (e.g. TCGA_BLCAtumor and TCGA_BLCAnormal). The rationale is that subsetting both bounds the number of Fisher Z / ΔZ tests and anchors edge significance testing to biologically curated interactions rather than all pairwise correlations.

## Related tools

- **WMDS.net** (Framework whose downstream differential co-expression and SSN computations consume the subsetted largest-connected Gene_Net subgraph as the candidate edge space) — https://github.com/chaofen123/WMDS.net
- **DawnRank** (Source of the gene-interaction network from which Gene_Net (hub of its intersection with TCGA genes) is constructed)
- **Matlab R2020b** (Implementation environment in which the network handling and subsequent integer-programming steps are run (intlinprog in the Optimization ToolBox))

## Evaluation signals

- The retained subgraph equals the largest connected component of Gene_Net (11,648 genes / 211,794 edges before subsetting) with all orphaned nodes removed
- Every gene symbol in the subsetted network is present in both the DawnRank interaction network and the TCGA gene set
- All Pearson correlations and ΔZ / Δr significance tests downstream are computed only over gene pairs connected in the subsetted network
- The resulting differential co-expression network satisfies approximate scale-free topology with linear-regression R² > 0.80

## Limitations

- Discarding nodes with no edge connection to the central part removes genes from the analysis, so driver discovery is limited to genes inside the largest connected subgraph
- The candidate edge space is bounded by the quality and coverage of the DawnRank gene-interaction network, not by the expression data itself

## Evidence

- [intro] Gene_Net is the hub network of intersection of gene interaction network constructed in Dawnrank[1] and the genes in TCGA.: "Gene_Net is the hub network of intersection of gene interaction network constructed in Dawnrank[1] and the genes in TCGA."
- [readme] The rows of the expression matrices represent gene expression, and the columns represent samples.: "The rows of the expression matrices represent gene expression, and the columns represent samples. The row and column names of the expression matrices are gene symbols and sample ids, respectively."
- [readme] we applied WMDS.net to the discovery of cancer driver genes in RNA-seq datasets from The Cancer Genome Atlas: "To confirm its validity, we applied WMDS.net to the discovery of cancer driver genes in RNA-seq datasets from The Cancer Genome Atlas."
