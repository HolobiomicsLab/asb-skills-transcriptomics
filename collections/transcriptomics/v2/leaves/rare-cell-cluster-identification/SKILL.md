---
name: rare-cell-cluster-identification
description: Use when a large scRNA-seq expression matrix (thousands to tens of thousands
  of cells, e.g. 68k PBMC or 4491-cell intestinal crypt) must be scanned for minor
  cell populations that bulk clustering toolkits like Seurat or Scater miss.
when_to_use_negative:
- The goal is comprehensive characterisation of major cell types, for which toolkits
  like Seurat or Scater are the appropriate choice
- The input is not a cell-by-gene expression profile (e.g. already-aggregated feature
  tables, bulk RNA-seq, or flow cytometry data)
- RaceID-scale per-cell clustering is required on modest-size data where full clustering
  is computationally feasible — though note RaceID's unrealistic runtime on thousands
  of cells is exactly what GapClust avoids
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: GapClust
  role: 'Primary rare-cell detection algorithm: computes nearest-neighbour distances
    and second-order derivative gap score with skewness<k> > 2 criterion'
  repo: github.com/fabotao/GapClust
- name: Seurat
  role: FindMarkers-style differential expression to annotate detected minor clusters
    with marker genes (Dclk1, Trpm5, CD14, CD34)
- name: GiniClust
  role: Competing rare-cell detector used for cross-checking; missed GapClust's 3-cell
    CD14+ monocyte cluster R1 in 68k PBMC
- name: FiRE
  role: Competing rare-cell detector used for cross-checking; also missed the 3-cell
    R1 cluster and does not discriminate outliers from minor cell types
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-24489-8
    title: GapClust is a light-weight approach distinguishing rare cells from voluminous
      single cell expression profiles
  source_content_hash: sha256:88fafd67dcbfbb5a59b4ffb869c5d4ace752e53c26651ee647c337b8594c6c1d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FiRE
  - GapClust
  - GiniClust
  - Seurat
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# rare-cell-cluster-identification

## Summary

Detect rare cell populations in large scRNA-seq datasets using GapClust, which exploits the gap between a minor cluster and its neighbouring abundant cluster via a second-order derivative type distance score on nearest-neighbour distances.

## When to use

Use when a large scRNA-seq expression matrix (thousands to tens of thousands of cells, e.g. 68k PBMC or 4491-cell intestinal crypt) must be scanned for minor cell populations that bulk clustering toolkits like Seurat or Scater miss.

## When NOT to use

- The goal is comprehensive characterisation of major cell types, for which toolkits like Seurat or Scater are the appropriate choice
- The input is not a cell-by-gene expression profile (e.g. already-aggregated feature tables, bulk RNA-seq, or flow cytometry data)
- RaceID-scale per-cell clustering is required on modest-size data where full clustering is computationally feasible — though note RaceID's unrealistic runtime on thousands of cells is exactly what GapClust avoids

## Inputs

- scRNA-seq count/expression matrix (pandas/Scanpy- or Seurat-compatible IO), e.g. 4491-cell intestinal crypt matrix or full 68k PBMC matrix
- Optional known marker gene list for annotation (e.g. Dclk1, Trpm5 for tuft cells; CD14; CD34)
- Candidate k range and skewness threshold (default criterion skewness<k> > 2)

## Outputs

- Rare-cluster membership table (cell barcode, cluster ID, cluster size, marker status), e.g. gapclust_rare_cells.csv or rare_cells_68k_pbmc.csv
- Ranked list of candidate rare clusters passing skewness<k> > 2
- Marker-gene differential expression results validating cluster identity (e.g. Dclk1/Trpm5 enrichment in the 29-cell R2 tuft cluster)

## How to apply

Load the raw scRNA-seq count matrix (e.g. 4491 non-irradiated intestinal crypt cells from Arshad et al., or the full 68k PBMC matrix) and run GapClust (github.com/fabotao/GapClust), which computes nearest-neighbour distances for each cell and applies a second-order derivative gap score over candidate k, retaining candidates where skewness<k> > 2. Record the detected minor clusters and their sizes (e.g. a 29-cell R2 cluster in the intestine data; 3-cell R1 and 185-cell R2 clusters in 68k PBMC). Annotate clusters with marker genes using Seurat/FindMarkers-style differential expression: R2 intestine cells should be enriched for tuft-cell markers Dclk1 and Trpm5, PBMC R1 for CD14+ monocyte markers and R2 for CD34+ megakaryocyte markers. To discover additional hidden clusters, rerun GapClust with the already-identified rare cells removed, which the paper reports can be done 'in no time'.

## Related tools

- **GapClust** (Primary rare-cell detection algorithm: computes nearest-neighbour distances and second-order derivative gap score with skewness<k> > 2 criterion) — github.com/fabotao/GapClust
- **Seurat** (FindMarkers-style differential expression to annotate detected minor clusters with marker genes (Dclk1, Trpm5, CD14, CD34))
- **GiniClust** (Competing rare-cell detector used for cross-checking; missed GapClust's 3-cell CD14+ monocyte cluster R1 in 68k PBMC)
- **FiRE** (Competing rare-cell detector used for cross-checking; also missed the 3-cell R1 cluster and does not discriminate outliers from minor cell types)

## Examples

```
devtools::install_github("fabotao/GapClust")
```

## Evaluation signals

- Output rare-cell membership file must contain a 29-cell R2 cluster whose top marker genes include Dclk1 and Trpm5, matching the tuft-cell identification in the intestinal dataset
- 68k PBMC run must reproduce rare clusters of sizes 3 (CD14+ R1) and 185 (CD34+/megakaryocyte R2) as in Fig. 6a-d
- The 3-cell R1 cluster should be absent from GiniClust and FiRE outputs, confirming it is uniquely detected by GapClust
- All reported candidate k values must satisfy the skewness<k> > 2 selection criterion
- Rerunning GapClust with identified rare cells removed should surface additional hidden clusters if present, per the paper's iterative-discovery workflow

## Limitations

- No version numbers for compared tools (GiniClust, RaceID, CellSIUS, FiRE, EDGE) are documented in the paper's discussion section
- No specific GapClust parameter settings (e.g. K, skewness threshold beyond skewness<k> > 2) are reported for individual runs
- Requires R >= 3.4.0 with Seurat >= 3.1.0, rflann >= 1.8.4, irlba >= 2.3.3, and e1071 >= 1.7-0.1
- Results for several real datasets (e.g. mouse somatosensory cortex, Drosophila wing disc) are deferred to supplementary figures not included in the main text

## Evidence

- [abstract] Here, we develop GapClust, a light-weight algorithm to detect rare cell types from ultra-large scRNA-seq datasets: "Here, we develop GapClust, a light-weight algorithm to detect rare cell types from ultra-large scRNA-seq datasets"
- [discussion] Candidate k can be identified if the skewness<k> > 2.: "Candidate k can be identified if the skewness<k> > 2."
- [discussion] To discover these hidden clusters, users can rerun GapClust in no time with the identified rare cell types removed: "To discover these hidden clusters, users can rerun GapClust in no time with the identified rare cell types removed"
- [abstract] GapClust identifies the tuft cells and a previously unrecognised subtype of monocyte, respectively: "GapClust identifies the tuft cells and a previously unrecognised subtype of monocyte, respectively"
- [readme] R users  can easily install GapClust by running following code in R console. devtools::install_github("fabotao/GapClust"): "R users  can easily install GapClust by running following code in R console. "
