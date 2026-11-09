---
name: gene-level-cnv-matrix-construction
description: Use when a CaSpER run has produced final.objects from runCaSpER and you
  need gene-level amplification/deletion calls (e.g. to benchmark 69% TPR / 0.3% FPR
  gene-based deletion detection against a genotyping array gold standard).
when_to_use_negative:
- You need chromosome-arm-scale events instead — use extractLargeScaleEvents(final.objects,
  thr=0.75) rather than collapsing to gene resolution.
- You only have raw expression/BAF signals and have not yet run CreateCasperObject
  and runCaSpER — the gene matrix requires the harmonized multiscale segment summary
  as input.
- The data are 3′-only sequencing (e.g. DropSeq) where degraded BAF shift signal from
  variants near 3′ gene ends can bias CNV calls.
tools:
- name: CaSpER
  role: Provides the gene.matrix() summarization function and the upstream runCaSpER/extractSegmentSummary
    pipeline that harmonizes multiscale HMM+BAF CNV calls before collapsing to genes
  repo: https://github.com/akdess/CaSpER
- name: BAFExtract
  role: Generates the B-allele frequency shift profiles that feed the BAF-shift thresholding
    integrated into the segment states summarized by gene.matrix
  repo: https://github.com/akdess/BAFExtract
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-13779-x
    title: CaSpER identifies and visualizes CNV events by integrative analysis of
      single-cell or bulk RNA-sequencing data
  source_content_hash: sha256:f07d4807fecb5a2ed04c45ccfb9b533cd11dc61a81a38a8a7f27fd61cd2fd08d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/akdess/CaSpER
  tools:
  - BAFExtract
  - CaSpER
  - mclust
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# gene-level-cnv-matrix-construction

## Summary

Constructs the final gene-by-sample CNV matrix in CaSpER by summarizing segment-level HMM+BAF-integrated calls across all N×M expression/BAF scale comparisons, encoding each gene in each sample/cell as 0 (neutral), 1 (amplification), or -1 (deletion).

## When to use

Use when a CaSpER run has produced final.objects from runCaSpER and you need gene-level amplification/deletion calls (e.g. to benchmark 69% TPR / 0.3% FPR gene-based deletion detection against a genotyping array gold standard).

## When NOT to use

- You need chromosome-arm-scale events instead — use extractLargeScaleEvents(final.objects, thr=0.75) rather than collapsing to gene resolution.
- You only have raw expression/BAF signals and have not yet run CreateCasperObject and runCaSpER — the gene matrix requires the harmonized multiscale segment summary as input.
- The data are 3′-only sequencing (e.g. DropSeq) where degraded BAF shift signal from variants near 3′ gene ends can bias CNV calls.

## Inputs

- final.objects (list of CNV calls from all N×M expression/BAF scale pairwise comparisons produced by runCaSpER)
- segment summary from extractSegmentSummary(final.objects) filtered at gamma=6 consistency
- normalized gene expression matrix (rows = genes with ensembl id or hgnc symbol, columns = samples)
- gene annotation object from generateAnnotation (e.g. id_type='ensembl_gene_id', ishg19=T, with centromere coordinates)
- list of all sample/cell names

## Outputs

- gene-by-sample CNV matrix with values 0 (neutral), 1 (amplification), -1 (deletion)
- gene-level deletion/amplification calls suitable for TPR/FPR benchmarking (e.g. calcROC against genotyping array gold standard)

## How to apply

After runCaSpER(object, removeCentromere=T, cytoband=cytoband, method="iterative") harmonizes all N×M pairwise expression/BAF scale comparisons, first obtain the segment-level summary with extractSegmentSummary(final.objects). Retain only segment calls whose consistency across the N×M scale comparisons meets the gamma=6 cutoff, so a gene/segment call is reported only where consistent calls exceed gamma out of the total comparisons. Then pass this summary, the full gene list, sample list, and gene annotation to gene.matrix() to build the gene-level CNV matrix encoding 0=neutral, 1=amplification, -1=deletion. The rationale is that harmonizing multiscale calls before collapsing to genes prevents single-scale HMM segmentation artifacts from propagating into per-gene states.

## Related tools

- **CaSpER** (Provides the gene.matrix() summarization function and the upstream runCaSpER/extractSegmentSummary pipeline that harmonizes multiscale HMM+BAF CNV calls before collapsing to genes) — https://github.com/akdess/CaSpER
- **BAFExtract** (Generates the B-allele frequency shift profiles that feed the BAF-shift thresholding integrated into the segment states summarized by gene.matrix) — https://github.com/akdess/BAFExtract

## Examples

```
final.objects <- runCaSpER(object, removeCentromere=T, cytoband=cytoband, method="iterative"); rna.matrix <- gene.matrix(seg=all.summary, all.genes=all.genes, all.samples=all.samples, genes.ann=genes.ann)
```

## Evaluation signals

- Matrix entries are strictly in {-1, 0, 1}, encoding deletion/neutral/amplification states only.
- Every gene-level call in the matrix is backed by segment calls that meet the gamma=6 consistency cutoff across the N×M scale comparisons.
- Benchmarking on the yale_meningioma dataset against the matched genotyping array with calcROC should reproduce ~69% TPR and 0.3% FPR for gene-based deletion events.
- Matrix dimensions match all.genes (columns/rows per gene) × all.samples (samples/cells) supplied at call time.
- Intermediate segment summaries show calls reported only when consistent across the expected number of scale comparisons.

## Limitations

- Ground-truth CNV calls can be unavailable for some datasets (e.g. MM135 single-cell data), so gene-level calls cannot always be benchmarked directly.
- RNA editing, PCR duplication artefacts, and sequencing errors may impact the BAF shift signal and bias gene-level CNV detection.
- Cells with a substantially large number of reads may dominate the BAF shift signal and potentially create biases in single-cell runs.
- The BAF shift generation model does not account for 3′ bias of sample preparation assays, which adversely impacts 3′-only sequencing data.

## Evidence

- [methods] gene.matrix summarization: "rna.matrix <- gene.matrix(seg=all.summary, all.genes=all.genes, all.samples=all.samples, genes.ann=genes.ann)"
- [methods] gene-based CNV summarization: "we generate a matrix where rows are the samples (cells) and columns are the genes"
- [methods] core CaSpER entry functions: "The main functions you will need to use to identify CNV events are CreateCasperObject() and runCaSpER(casper_object)."
- [abstract] multiscale harmonization rationale: "identification, visualization, and integrative analysis of focal and large-scale CNV events in multiscale resolution"
