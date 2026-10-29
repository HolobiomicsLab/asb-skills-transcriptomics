---
name: spatial-cluster-map-visualization
description: Use when a Seq-Scope, Slide-seq, or VISIUM spatial transcriptomics dataset
  has been clustered in Seurat (Step C3) and you need to visualize cell-type cluster
  identities across tissue coordinates, or to qualitatively compare MSSW versus simple-aggregation
  map resolution.
when_to_use_negative:
- 'Attempting subcellular-resolution spatial clustering maps: MSSW spatial segments
  must be larger than subcellular compartments (~10 μm), so subcellular analysis with
  MSSW is not feasible'
- When input is raw FASTQ or an uncollapsed spatial DGE — Steps A1–A3 and C1/C2 must
  run first before Step V2 can plot clusters
- When quantitative cluster-metric comparison is needed — the paper's validation is
  qualitative visual inspection of side-by-side maps, not a numerical resolution metric
edam_operation: http://edamontology.org/operation_3204
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0092
tools:
- name: STtools
  role: Runs Step V2 to generate UMAP, spatial clustering, and feature violin plots;
    Step C2 implements the MSSW sliding-window strategy (--binsize 300 --window 150)
    whose maps are visualized
  repo: https://github.com/seqscope/STtools
- name: Seurat
  role: Performs dimension reduction, clustering, and reference mapping whose cluster
    identities are rendered on the spatial maps
  repo: https://satijalab.org/seurat/articles/spatial_vignette.html
- name: STARsolo
  role: 'Upstream: aligns cDNA and produces the DGE from which collapsed-grid Seurat
    objects are built'
  repo: https://github.com/alexdobin/STAR
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbac061
    title: 'STtools: a comprehensive software pipeline for ultra-high-resolution spatial
      transcriptomics data'
  source_content_hash: sha256:a9ab689cb6ddbfeb6bc7f0ff885697cd684c6081180d043016fc84dfaf7c953a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/seqscope/STtools
  tools:
  - STARsolo
  - STtools
  - Seurat
  edam_operation: http://edamontology.org/operation_3204
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# spatial-cluster-map-visualization

## Summary

Generate UMAP embeddings and spatial cluster maps from STtools' Seurat objects (Step V2) to render cell-type identities across tissue coordinates, enabling side-by-side comparison of simple square-grid aggregation versus multiscale sliding window (MSSW) cluster maps.

## When to use

Use when a Seq-Scope, Slide-seq, or VISIUM spatial transcriptomics dataset has been clustered in Seurat (Step C3) and you need to visualize cell-type cluster identities across tissue coordinates, or to qualitatively compare MSSW versus simple-aggregation map resolution.

## When NOT to use

- Attempting subcellular-resolution spatial clustering maps: MSSW spatial segments must be larger than subcellular compartments (~10 μm), so subcellular analysis with MSSW is not feasible
- When input is raw FASTQ or an uncollapsed spatial DGE — Steps A1–A3 and C1/C2 must run first before Step V2 can plot clusters
- When quantitative cluster-metric comparison is needed — the paper's validation is qualitative visual inspection of side-by-side maps, not a numerical resolution metric

## Inputs

- DGE in Velocyto format produced by STtools Step C3
- Seurat RDS objects from Step C1 (simple square-grid collapsed DGE) and Step C2 (sliding-window collapsed DGE)
- Spatial coordinates of spatial barcodes (.txt files from Step A1)
- Cluster assignments/reference-mapped cell-type labels from Seurat clustering (Step C3)

## Outputs

- UMAP embedding plots
- Spatial clustering plots (cell-type maps across tissue coordinates)
- Feature violin plots
- Side-by-side comparison figure (e.g. spatial_cluster_maps.png or slideseq_cluster_maps.png)

## How to apply

Run STtools Step V2, which takes the DGE (Velocyto-format) output from Step C3 and generates UMAP, spatial clustering, and feature violin plots from the Seurat objects. Compose a side-by-side panel comparing the simple square-grid bin cluster map (Step C1, e.g. 2500 μm² bins) against the MSSW overlapping-bin map (Step C2, e.g. --binsize 300 --window 150 on Seq-Scope). Judge correctness by verifying that cluster labels transferred via Seurat reference mapping align with known tissue anatomy (e.g. liver zonation boundaries or cerebellar layers) and that boundaries on the MSSW map are delineated at several-fold finer resolution than the simple-bin map. Note the expected platform-dependent outcome: on Seq-Scope mouse liver (GSM5212844) the MSSW map shows visibly finer boundaries, whereas on Slide-seq cerebellum the MSSW benefit is not visually pronounced due to the technology's lower (10 μm) resolution.

## Related tools

- **STtools** (Runs Step V2 to generate UMAP, spatial clustering, and feature violin plots; Step C2 implements the MSSW sliding-window strategy (--binsize 300 --window 150) whose maps are visualized) — https://github.com/seqscope/STtools
- **Seurat** (Performs dimension reduction, clustering, and reference mapping whose cluster identities are rendered on the spatial maps) — https://satijalab.org/seurat/articles/spatial_vignette.html
- **STARsolo** (Upstream: aligns cDNA and produces the DGE from which collapsed-grid Seurat objects are built) — https://github.com/alexdobin/STAR

## Examples

```
python3 $STHOME/sttools.py --run-all --STtools $STHOME --first-fq $STDATA/stepA_extractCoordinates/liver-MiSeq-tile2106-sub-R1.fastq.gz --second-fq1 $STDATA/stepA_align/liver_tile2106_sub_R1.fastq.gz --second-fq2 $STDATA/stepA_align/liver_tile2106_sub_R2.fastq.gz --outdir $STOUT --genome $GENOMEINDEX --star-path $STARPATH --seqtk-path $SEQTKPATH --seqscope1st 'HiSeq' --clustering False --lane-tiles 1_2106 --binsize 300 --window 150 -l 20 -o 'Sample' -c 2
```

## Evaluation signals

- Step V2 runs without error and emits UMAP, spatial clustering, and feature violin plots for both the C1 (simple grid) and C2 (MSSW) Seurat objects
- Cluster labels transferred from simple square-grid bins onto MSSW overlapping bins show contiguous spatial domains consistent with tissue anatomy (e.g. liver or cerebellum layers)
- Qualitative side-by-side inspection confirms the MSSW map delineates cell-type boundaries at several-fold higher spatial resolution than the simple-bin map on Seq-Scope (matching the paper's claim)
- On Slide-seq cerebellum, the expected (negative-control) result is that the MSSW benefit is NOT visually pronounced compared to simple aggregation, matching Fig. 1E–H
- Spatial cluster maps align with barcode/HDMI density plots (Step A2) and estimated tissue boundaries

## Limitations

- On low-resolution platforms such as Slide-seq (10 μm), the MSSW strategy's benefit is not visually pronounced compared to simple aggregation, so the side-by-side map comparison may show little difference
- Subcellular analysis with MSSW is not feasible — each spatial segment must be larger than subcellular compartments (~10 μm)
- UMI counts per spatial barcode are much lower at sub-micrometer resolutions, so per-barcode expression is extremely sparse and robust spatial smoothing/imputation methods are not yet available in STtools
- Validation of the comparison is qualitative (visual inspection or LLM review of the saved PNG), not a quantitative metric

## Evidence

- [readme] Step V2 takes DGE (Velocyto-format) from Step C3 and generate UMAP, Spatial clustering, and feature violin plots.: "**Step V2** takes DGE (Velocyto-format) from **Step C3** and generate UMAP, Spatial clustering, and feature violin plots."
- [abstract] runs downstream analyses at several folds higher resolution than existing methods: "runs downstream analyses at several folds higher resolution than existing methods"
- [readme] clustering cell types, and high-resolution analysis with sliding window strategy: "The STTools pipeline includes preprocessing of raw sequence reads, 
alignment, collapsing barcodes into grids, clustering cell types, and
high-resolution analysis with sliding window strategy."
- [results] the benefit of the MSSW strategy ... was not visually pronounced compared to simple aggregation, primarily due to the low resolution of the Slide-seq technology: [evidence span withheld — anchor status: not_found]
