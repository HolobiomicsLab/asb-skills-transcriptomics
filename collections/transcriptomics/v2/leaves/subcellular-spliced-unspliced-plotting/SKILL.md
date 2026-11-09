---
name: subcellular-spliced-unspliced-plotting
description: Use when you have a Velocyto-format DGE with spatial coordinates from
  Seq-Scope (or other <1μm ST data) and need to visualize subcellular spliced/unspliced
  read patterns at ~1μm²/pixel resolution.
when_to_use_negative:
- When the analysis requires cell-type clustering or dimension reduction — use Step
  C1–C3/V2 with grid-collapsed DGEs instead, since per-barcode UMI counts at sub-micrometer
  resolution are too sparse for clustering
- When the DGE is in plain DGE format rather than Velocyto format (spliced/unspliced
  annotations missing) — Step V1 specifically requires the Velocyto-format output
  from Step A3
- When aiming for subcellular inference via sliding-window (MSSW) analysis — the article
  states each spatial segment must be larger than subcellular compartments (~10um),
  so MSSW is not feasible at subcellular scale
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: STtools
  role: Pipeline providing Step V1, which takes the Velocyto-format DGE from Step
    A3 and generates subcellular spliced/unspliced spatial plots without binning
  repo: https://github.com/seqscope/STtools
- name: STARsolo
  role: Alignment step (Step A3) that produces the Velocyto-format DGE consumed by
    Step V1
  repo: https://github.com/alexdobin/STAR
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbac061
    title: 'STtools: a comprehensive software pipeline for ultra-high-resolution spatial
      transcriptomics data'
  source_content_hash: sha256:23c75fa03b99076e23f6b5a0a2dd00065592e1d7aa637e4e7a967802696b6da3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/seqscope/STtools
  tools:
  - STARsolo
  - STtools
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# subcellular-spliced-unspliced-plotting

## Summary

Generate subcellular-resolution spatial plots of spliced versus unspliced read patterns from a Velocyto-format digital expression matrix using STtools Step V1, revealing sub-micrometer transcriptomic architecture (e.g., nuclear vs. mitochondrial localization) in Seq-Scope data.

## When to use

Use when you have a Velocyto-format DGE with spatial coordinates from Seq-Scope (or other <1μm ST data) and need to visualize subcellular spliced/unspliced read patterns at ~1μm²/pixel resolution.

## When NOT to use

- When the analysis requires cell-type clustering or dimension reduction — use Step C1–C3/V2 with grid-collapsed DGEs instead, since per-barcode UMI counts at sub-micrometer resolution are too sparse for clustering
- When the DGE is in plain DGE format rather than Velocyto format (spliced/unspliced annotations missing) — Step V1 specifically requires the Velocyto-format output from Step A3
- When aiming for subcellular inference via sliding-window (MSSW) analysis — the article states each spatial segment must be larger than subcellular compartments (~10um), so MSSW is not feasible at subcellular scale

## Inputs

- Velocyto-format digital expression matrix (DGE) from Step A3 STARsolo alignment, containing spliced/unspliced read counts per spatial barcode
- Spatial coordinates for Seq-Scope barcodes (from Step A1)
- SeqScope example data (STtools_example_data.zip) for validation runs

## Outputs

- Subcellular plots showing the spatial pattern of spliced versus unspliced reads at <1μm²/pixel resolution (PNG image)

## How to apply

Obtain the Velocyto-format DGE produced by Step A3's STARsolo alignment (or an equivalent externally supplied matrix annotated with spatial coordinates). Run STtools Step V1, which consumes this DGE directly without spatial binning or grid collapsing, so the native sub-micrometer barcode resolution is retained. Render the output plots so each pixel maps to normalized transcript abundance (~1μm²/pixel), letting nuclear-enriched spliced reads and unspliced (intronic/mitochondrial) signals appear as distinct spatial patterns. Optionally pair the output with the RGB geneset plots from the same DGE to co-visualize cell-type markers and subcellular compartments. Do not attempt subcellular analysis via the MSSW sliding-window path (Step C2/C3), since spatial segments there must be larger than subcellular compartments (~10μm).

## Related tools

- **STtools** (Pipeline providing Step V1, which takes the Velocyto-format DGE from Step A3 and generates subcellular spliced/unspliced spatial plots without binning) — https://github.com/seqscope/STtools
- **STARsolo** (Alignment step (Step A3) that produces the Velocyto-format DGE consumed by Step V1) — https://github.com/alexdobin/STAR

## Examples

```
python3 $STHOME/sttools.py --run-all --STtools $STHOME --first-fq $STDATA/stepA_extractCoordinates/liver-MiSeq-tile2106-sub-R1.fastq.gz --second-fq1 $STDATA/stepA_align/liver_tile2106_sub_R1.fastq.gz --second-fq2 $STDATA/stepA_align/liver_tile2106_sub_R2.fastq.gz --outdir $STOUT --genome $GENOMEINDEX --star-path $STARPATH --seqtk-path $SEQTKPATH --seqscope1st 'HiSeq' --clustering False --lane-tiles 1_2106 --binsize 300 --window 150 -l 20 -o 'Sample' -c 2
```

## Evaluation signals

- Output PNG exists and depicts spatial structure at <1μm²/pixel resolution
- Subcellular patterns (nucleus vs. mitochondria vs. surrounding cytoplasm) are visually discernible in the plot without sacrificing barcode resolution
- Cell-type differences (periportal vs. pericentral hepatocytes vs. macrophages) remain resolvable in the same field of view
- The input DGE is confirmed Velocyto-format (contains spliced/unspliced read categories) and traces back to the Step A3 STARsolo output

## Limitations

- Spatial smoothing algorithms for robust inference from extremely sparse expression profiles per spatial unit are not yet available in STtools
- No method to impute spatial expression profiles exists in STtools, limiting quantitative subcellular inference
- UMI count per spatial barcode is typically very low at sub-micrometer resolution due to the limited number of mRNAs captured, so plots are sparse
- No automatic overlay of histological images and spatial expressions is available (flagged as a future feature)

## Evidence

- [readme] Step V1 takes DGE (Velocyto-format) from Step A3 and generate subcellular plots showing pattern of spliced/unspliced reads: "**Step V1** takes DGE (Velocyto-format) from **Step A3** and generate subcellular plots showing pattern of spliced/unspliced reads"
- [intro] Subcellular architecture challenge at ultra-high resolution: "existing tools do not account for subcellular components in their analysis and visualization to accommodate the ultra-high-resolution from recent technologies"
- [abstract] STtools visualization capabilities: "It also generates various visualizations including transcriptome density, cell type mapping, marker gene highlighting, and subcellular architectures"
- [discussion] MSSW not feasible for subcellular analysis: "each spatial segment still needs to be larger than subcellular compartments (e.g. ~10um), so subcellular analysis with MSSW is not feasible"
- [intro] Sparse UMI per barcode at high resolution: "the UMI count per spatial barcode is typically much lower due to the limited number of mRNAs that can be captured"
