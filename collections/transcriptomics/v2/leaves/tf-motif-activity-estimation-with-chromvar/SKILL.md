---
name: tf-motif-activity-estimation-with-chromvar
description: Use when you have a paired/single-cell multiome Seurat object with an
  ATAC peak assay and need per-cell TF activity scores (e.g., to select candidate
  TFs and compute TF-gene correlations for eGRN construction along a pseudotime trajectory).
when_to_use_negative:
- Input lacks a single-cell chromatin accessibility assay (e.g., RNA-only data) —
  chromVAR deviation scores require ATAC peaks over which to scan motifs
- The analysis targets a non-human genome or requires motif sets outside JASPAR2020;
  the tutorial binds the pipeline to hg38 (BSgenome.Hsapiens.UCSC.hg38) and JASPAR2020
  CORE matrices
- You only need bulk peak counts or differential accessibility rather than per-cell
  TF activity to build a regulatory network
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scMEGA
  role: Provides the AddMotifs and RunChromVAR wrappers that perform motif matching
    and compute chromVAR deviation scores on Seurat objects
  repo: https://github.com/CostaLab/scMEGA
- name: Seurat v5
  role: Hosts the multiome object and the 'chromvar' assay holding TF activity scores
    used by downstream scMEGA functions
  repo: https://github.com/satijalab/seurat
- name: Signac
  role: Underlying single-cell ATAC-seq infrastructure for peak assays and motif matching
    in the Seurat v5 branch
  repo: https://github.com/stuart-lab/signac
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbad003
    title: 'scMEGA: single-cell multi-omic enhancer-based gene regulatory network
      inference'
  source_content_hash: sha256:029fdd754da4b66d9ebc50b1ee80c5a114300e4fee7427b1e0602219848429da
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CostaLab/scMEGA
  tools:
  - ArchR
  - Seurat v5
  - Signac
  - ggraph
  - igraph
  - scMEGA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# TF motif activity estimation with chromVAR

## Summary

Estimates per-cell transcription factor activity from scATAC-seq peaks by matching JASPAR2020 motifs against hg38 (AddMotifs) and computing chromVAR deviation scores (RunChromVAR), as wrapped by scMEGA on Seurat/Signac objects. The resulting 'chromvar' assay feeds downstream TF selection and TF-gene correlation steps for enhancer-based GRN inference.

## When to use

Use when you have a paired/single-cell multiome Seurat object with an ATAC peak assay and need per-cell TF activity scores (e.g., to select candidate TFs and compute TF-gene correlations for eGRN construction along a pseudotime trajectory).

## When NOT to use

- Input lacks a single-cell chromatin accessibility assay (e.g., RNA-only data) — chromVAR deviation scores require ATAC peaks over which to scan motifs
- The analysis targets a non-human genome or requires motif sets outside JASPAR2020; the tutorial binds the pipeline to hg38 (BSgenome.Hsapiens.UCSC.hg38) and JASPAR2020 CORE matrices
- You only need bulk peak counts or differential accessibility rather than per-cell TF activity to build a regulatory network

## Inputs

- Seurat/Signac object containing a single-cell ATAC-seq peak (chromatin accessibility) assay from paired multiome data (e.g., 10x Genomics PBMC multiome, pbmc_granulocyte_sorted_10k)
- JASPAR2020 position frequency matrices (CORE vertebrate set) retrieved via getMatrixSet
- Reference genome for motif scanning, BSgenome.Hsapiens.UCSC.hg38
- Gene annotation GRanges derived from EnsDb.Hsapiens.v86 (via GetGRangesFromEnsDb)

## Outputs

- Seurat object with a 'chromvar' assay holding per-TF deviation (activity) scores per cell
- Motif-matched ATAC assay enriched with motif occurrence annotations
- Per-cell TF activity matrix used downstream by SelectTFs (cor.cutoff = 0.4) and GetTFGeneCorrelation (tf.assay = 'chromvar')

## How to apply

First add motif annotations to the ATAC assay with scMEGA's AddMotifs using the JASPAR2020 position frequency matrices (via getMatrixSet from JASPAR2020) and BSgenome.Hsapiens.UCSC.hg38 as reference genome, plus EnsDb.Hsapiens.v86-derived gene annotations. Then run RunChromVAR on the same object/genome to compute a deviation score for each TF in each cell, stored in a 'chromvar' assay. In the scMEGA workflow this activity matrix is the TF-level input for candidate TF selection (SelectTFs with cor.cutoff = 0.4 against trajectory expression) and for GetTFGeneCorrelation, where tf.assay = 'chromvar' and gene.assay = 'RNA' correlate TF deviation scores with gene expression along the pseudotime to assemble eGRN edges. Rationale: enhancer-based GRN inference requires an activity score for each TF in each cell, which scMEGA obtains by motif matching followed by chromVAR deviation scoring rather than TF mRNA abundance alone.

## Related tools

- **scMEGA** (Provides the AddMotifs and RunChromVAR wrappers that perform motif matching and compute chromVAR deviation scores on Seurat objects) — https://github.com/CostaLab/scMEGA
- **Seurat v5** (Hosts the multiome object and the 'chromvar' assay holding TF activity scores used by downstream scMEGA functions) — https://github.com/satijalab/seurat
- **Signac** (Underlying single-cell ATAC-seq infrastructure for peak assays and motif matching in the Seurat v5 branch) — https://github.com/stuart-lab/signac

## Examples

```
obj <- AddMotifs(object = obj, genome = BSgenome.Hsapiens.UCSC.hg38); obj <- RunChromVAR(object = obj, genome = BSgenome.Hsapiens.UCSC.hg38)
```

## Evaluation signals

- The Seurat object contains a new assay named 'chromvar' whose rows are TFs and whose columns are cells with finite deviation scores
- Motif scanning was restricted to JASPAR2020 CORE vertebrate matrices retrieved through getMatrixSet(x = JASPAR2020, ...)
- Genome passed to both AddMotifs and RunChromVAR matches the genome build of the ATAC fragment file (BSgenome.Hsapiens.UCSC.hg38)
- Downstream SelectTFs applied to the chromvar assay uses correlation cutoff cor.cutoff = 0.4 to keep candidate TFs
- GetTFGeneCorrelation is called with tf.assay = 'chromvar' and gene.assay = 'RNA' along the trajectory (trajectory.name = 'Trajectory'), confirming the activity scores were actually consumed

## Limitations

- The tutorials do not pin software/package versions — they only advise 'Check session information sessionInfo()', so reproducibility across chromVAR/Signac versions is not guaranteed
- The demonstrated configuration is human-specific (hg38 genome, JASPAR2020 vertebrate matrices); other genomes or motif collections require substituting the genome and pfm inputs
- Deviation scores are the article's TF-activity proxy; TF recovery quality in the downstream CD4 T cell benchmark depends on this proxy (>75% TF recovery from predicted pairs), so errors in motif matching propagate to eGRN edges

## Evidence

- [methods] TF activity estimation via motif matching and chromVAR deviation scores: "we first estimate an acitivty score for each TF in each cell. This is done by first performing motif matching and then computing deviation scores"
- [methods] AddMotifs/RunChromVAR wrapper calls on the Seurat object: "obj <- RunChromVAR(
  object = obj,
  genome = BSgenome.Hsapiens.UCSC.hg38,"
- [methods] JASPAR2020 motif matrices as motif source: "pfm <- getMatrixSet(
  x = JASPAR2020,"
- [readme] scMEGA tutorial coverage of the CD4 T cell 10x multiome use case: "We provided the following tutorials to show how to use scMEGA to build GRN by using single-cell multiomics/multimodal data"
- [intro] Installation prerequisites for running the chromVAR-based TF activity step: "devtools::install_github("satijalab/seurat", "seurat5", quiet = TRUE)"
