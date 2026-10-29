---
name: te-tool-benchmarking-scrnaseq
description: Use when evaluating or selecting a computational tool for TE expression
  quantification in scRNA-Seq data, or when validating that a new TE tool recovers
  biologically expected marker TEs in a defined cell population.
when_to_use_negative:
- When the goal is bulk RNA-Seq TE analysis — locus-level bulk tools such as SQuIRE
  and Telescope already exist and this scRNA-Seq benchmarking design does not apply
- When the input is already a TE expression matrix — benchmarking requires reprocessing
  shared BAM/FASTQ data through both tools
- When assessing 3'-end 10X data with EM-based allocation of multi-mapped reads —
  the article states this performance has not yet been assessed
edam_operation: http://edamontology.org/operation_2478
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0749
tools:
- name: SoloTE
  role: Locus-level TE quantification tool under benchmark; produces the gene+TE matrix
    and is compared for speed and marker TE recovery
  repo: https://github.com/bvaldebenitom/SoloTE
- name: scTE
  role: Family-metagene TE quantification tool used as the baseline comparator for
    TE detection, UMAP, and marker TE counts
- name: STAR
  role: Aligner with multi-mapping options (--winAnchorMultimapNmax 100, --outFilterMultimapNmax
    100, --outSAMmultNmax 1) to prepare BAM input shared by both tools
- name: BEDTools
  role: Assesses read overlap with the TE annotation inside the SoloTE pipeline
  repo: https://github.com/arq5x/bedtools2
- name: Seurat
  role: Normalizes and clusters the combined gene+TE matrix and runs FindAllMarkers
    to enumerate marker TEs per tool
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42003-022-04020-5
    title: SoloTE for improved analysis of transposable elements in single-cell RNA-Seq
      data using locus-specific expression
  source_content_hash: sha256:3bf212633e2b4d96b056bb338fb50113ca77d232ae974deee7b68419282d0c9d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bvaldebenitom/SoloTE
  tools:
  - BEDTools
  - STAR
  - Seurat
  - SoloTE
  - scTE
  edam_operation: http://edamontology.org/operation_2478
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# te-tool-benchmarking-scrnaseq

## Summary

Benchmark single-cell RNA-Seq transposable element (TE) analysis tools by comparing locus-level tools such as SoloTE against family-metagene tools such as scTE on shared real datasets, judging performance by TE expression detection, UMAP dimensional reduction, computational cost, and recovery of known cell-marker TEs.

## When to use

Use when evaluating or selecting a computational tool for TE expression quantification in scRNA-Seq data, or when validating that a new TE tool recovers biologically expected marker TEs in a defined cell population.

## When NOT to use

- When the goal is bulk RNA-Seq TE analysis — locus-level bulk tools such as SQuIRE and Telescope already exist and this scRNA-Seq benchmarking design does not apply
- When the input is already a TE expression matrix — benchmarking requires reprocessing shared BAM/FASTQ data through both tools
- When assessing 3'-end 10X data with EM-based allocation of multi-mapped reads — the article states this performance has not yet been assessed

## Inputs

- scRNA-Seq FASTQ files from the Gene Expression Omnibus (accessions in Supplementary Table 3)
- STAR-aligned BAM file with multi-mapping reads retained
- mm10 RepeatMasker annotation converted to SoloTE BED format (chr|start|end|Subfamily:Family:Class|strand)
- Known cell-population marker genes used for ground truth (e.g., Zscan4c, Tcstv3 for 2C-like cells)

## Outputs

- Marker TE counts per tool (e.g., solute_2c_marker_TEs.csv reporting 106 SoloTE vs 28 scTE marker TEs)
- Combined gene and TE expression matrix compatible with Seurat
- Marker analysis table from Seurat FindAllMarkers filtered to adjusted P-value ≤0.05 and average log2(FC) ≥1
- Comparison of TE-expression detection, UMAP dimensional reduction, runtime, and computational resource use

## How to apply

Run both tools on identical preprocessed data: for SoloTE, align reads to mm10 with STAR using multi-mapping options --winAnchorMultimapNmax 100, --outFilterMultimapNmax 100, and --outSAMmultNmax 1, convert the RepeatMasker annotation to BED with SoloTE_RepeatMasker_to_BED.py (-g mm10), and run SoloTE_pipeline.py on the STAR BAM file with default MAPQ 255 for locus-level assignment. Run scTE on the same dataset for comparison, since scTE summarizes TE expression on family metagenes and loses locus information. Benchmark along three axes: (i) detecting TE expression, (ii) UMAP dimensional reduction of the combined gene+TE matrix in Seurat, and (iii) detection of cell marker TEs — e.g., define 2C-like cells by Zscan4c/Tcstv3 expression in the murine two-cell stage dataset, run Seurat FindAllMarkers with adjusted P-value ≤0.05 and average log2(Fold Change) ≥1, then count marker TEs per tool. Also record runtime and computational resource use, where SoloTE was found to be faster and cheaper than scTE. A correct benchmark recovers 106 marker TEs with SoloTE versus 28 with scTE in 2C-like cells, and confirms that MERVL-int and MT2-Mm show the expected 2C-marker expression pattern.

## Related tools

- **SoloTE** (Locus-level TE quantification tool under benchmark; produces the gene+TE matrix and is compared for speed and marker TE recovery) — https://github.com/bvaldebenitom/SoloTE
- **scTE** (Family-metagene TE quantification tool used as the baseline comparator for TE detection, UMAP, and marker TE counts)
- **STAR** (Aligner with multi-mapping options (--winAnchorMultimapNmax 100, --outFilterMultimapNmax 100, --outSAMmultNmax 1) to prepare BAM input shared by both tools)
- **BEDTools** (Assesses read overlap with the TE annotation inside the SoloTE pipeline) — https://github.com/arq5x/bedtools2
- **Seurat** (Normalizes and clusters the combined gene+TE matrix and runs FindAllMarkers to enumerate marker TEs per tool)

## Examples

```
python SoloTE_RepeatMasker_to_BED.py -g mm10 && python SoloTE_pipeline.py --threads 8 --bam aligned.bam --teannotation mm10.bed --outputprefix solote --outputdir out
```

## Evaluation signals

- SoloTE recovers 106 marker TEs in 2C-like cells (adjusted P ≤0.05, average log2 FC ≥1) versus 28 with scTE
- MERVL-int and MT2-Mm show the expected 2C-marker expression pattern in the defined 2C-like population
- Both tools are run on the same STAR BAM (multi-mapping options above) so marker TE counts are directly comparable
- SoloTE is faster and uses fewer computational resources than scTE on the same hardware
- SoloTE output matrix reports locus-level TE IDs (chr|start|end|Subfamily:Family:Class) whereas scTE indexes do not filter non-TE repeats such as Satellites

## Limitations

- scTE summarizes TE expression on family metagenes, losing the genomic location from which each TE is expressed
- scTE does not filter non-TE repeats in its indexes, so Satellites can appear in the resulting matrix
- Not all TEs can be uniquely mappable with short 100-nucleotide 3'-end 10X reads, and EM-allocation performance on such data has not been assessed
- Dataset accession IDs are only given in Supplementary Table 3, not in the main text

## Evidence

- [intro] First, we compare it to scTE, in terms of (i) detecting TE expression, (ii) UMAP dimensional reduction, and (iii) detection of cell marker TEs.: "First, we compare it to scTE, in terms of (i) detecting TE expression, (ii) UMAP dimensional reduction, and (iii) detection of cell marker TEs."
- [results] SoloTE recovers 106 marker TEs in 2C-like cells (vs 28 with scTE): [evidence span withheld — anchor status: not_found]
- [discussion] FindAllMarkers thresholds: [evidence span withheld — anchor status: not_found]
- [discussion] scTE metagene limitation: "which summarizes TE expression on metagenes corresponding to TE families, thus, losing the genomic location from which each TE is expressed"
- [readme] SoloTE RepeatMasker conversion usage: "python SoloTE_RepeatMasker_to_BED.py -g GenomeVersion"
