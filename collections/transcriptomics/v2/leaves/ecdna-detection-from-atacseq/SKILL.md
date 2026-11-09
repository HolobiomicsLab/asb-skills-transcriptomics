---
name: ecdna-detection-from-atacseq
description: Use when ATAC-Seq libraries from tumor/cancer samples must be scanned
  for ecDNA (focal circular amplifications, often >1 Mb carrying oncogenes like MYC,
  MYCN, EGFR) without WGS, or when comparing ecDNA callers like circlehunter vs Circle_finder.
when_to_use_negative:
- When the ecDNA/focal amplification is not detectable by WGS copy-number tools (e.g.,
  AmpliconArchitect requires CNV input) or involves complex structural variants beyond
  ecDNA — accuracy of calls for all focal amplification types is not guaranteed
- For single-cell ATAC-Seq data — application of circlehunter to single-cell ATAC-Seq
  has yet to be explored
- When the goal is gene set enrichment analysis rather than ecDNA detection — GSEApy
  is a separate tool unrelated to this pipeline
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0622
tools:
- name: circlehunter
  role: Primary pipeline that identifies and reconstructs ecDNA from ATAC-Seq data
    via enhanced chromatin accessibility
  repo: https://github.com/suda-huanglab/circlehunter
- name: Snakemake
  role: Workflow engine executing the circlehunter pipeline from a YAML config
- name: Circle_finder
  role: Comparison baseline ecDNA caller that discovers junctions from supplementary
    alignment reads in ATAC-Seq data
- name: ART
  role: Simulator used to generate sequencing reads for 500 mock ecDNAs sampled from
    GRCh38 in the benchmark
- name: ATAC-Seq
  role: Assay providing the input data exploiting ecDNA's high chromatin accessibility
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41389-023-00476-0
    title: 'Circlehunter: a tool to identify extrachromosomal circular DNA from ATAC-Seq
      data'
  source_content_hash: sha256:cf07f893136837d19965747b9d9a501b31cb65c4ec69f7aba070de644993d605
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/suda-huanglab/circlehunter
  tools:
  - ART
  - ATAC-Seq
  - Circle_finder
  - Snakemake
  - circlehunter
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# ecdna-detection-from-atacseq

## Summary

Detect extrachromosomal circular DNA (ecDNA) from ATAC-Seq data using circlehunter, which exploits the enhanced chromatin accessibility of ecDNA instead of resource-intensive WGS or RCA enrichment, and benchmark calls against simulated ground truth with precision/recall/F1.

## When to use

Use when ATAC-Seq libraries from tumor/cancer samples must be scanned for ecDNA (focal circular amplifications, often >1 Mb carrying oncogenes like MYC, MYCN, EGFR) without WGS, or when comparing ecDNA callers like circlehunter vs Circle_finder.

## When NOT to use

- When the ecDNA/focal amplification is not detectable by WGS copy-number tools (e.g., AmpliconArchitect requires CNV input) or involves complex structural variants beyond ecDNA — accuracy of calls for all focal amplification types is not guaranteed
- For single-cell ATAC-Seq data — application of circlehunter to single-cell ATAC-Seq has yet to be explored
- When the goal is gene set enrichment analysis rather than ecDNA detection — GSEApy is a separate tool unrelated to this pipeline

## Inputs

- ATAC-Seq sequencing libraries (e.g., FASTQ from tumor samples or simulated ecDNA reads blended with GM12878 background, ENCODE GSE170245)
- YAML config file enumerating samples and outputs (config.yaml from config-template.yaml)
- Simulated ecDNA ground-truth coordinates for benchmarking (ART-generated, GRCh38-based)

## Outputs

- Predicted ecDNA coordinates/reconstructions per sample
- Precision, recall, and F1 tables per local-depth and read-length condition (e.g., f1_scores_fig2ab.csv for depth- and read-length sweeps)

## How to apply

Install the circlehunter environment with mamba from environment.yml and activate it (conda activate circlehunter). Create a workspace, copy the config template (cp ../config-template.yaml config.yaml), and edit the YAML config to enumerate all samples and their outputs. Launch the Snakemake pipeline (snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml) to reconstruct ecDNA from the increased ATAC-Seq signals arising from ecDNA's high accessibility, relying on junction/supplementary-alignment reads. For benchmarking, simulate 500 mock ecDNAs from GRCh38 (5 kb–10 Mb, 1–50 genomic segments) with ART, blend reads with GM12878 ATAC-Seq background (ENCODE GSE170245) across local depths and read lengths, then compute precision, recall, and F1 per condition against simulated ground truth; expect F1 ≈ 0.93 at ≥30× local depth and near-invariance to read lengths down to 35 bp.

## Related tools

- **circlehunter** (Primary pipeline that identifies and reconstructs ecDNA from ATAC-Seq data via enhanced chromatin accessibility) — https://github.com/suda-huanglab/circlehunter
- **Snakemake** (Workflow engine executing the circlehunter pipeline from a YAML config)
- **Circle_finder** (Comparison baseline ecDNA caller that discovers junctions from supplementary alignment reads in ATAC-Seq data)
- **ART** (Simulator used to generate sequencing reads for 500 mock ecDNAs sampled from GRCh38 in the benchmark)
- **ATAC-Seq** (Assay providing the input data exploiting ecDNA's high chromatin accessibility)

## Examples

```
mamba env create -f environment.yml; conda activate circlehunter; cp ../config-template.yaml config.yaml; snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml
```

## Evaluation signals

- F1 score of ~0.93 at ≥30× local depth on simulated data (Fig. 2A depth sweep)
- Near-invariance of F1 to read length, including reads as short as 35 bp, where Circle_finder fails below 75 bp (Fig. 2B read-length sweep)
- Predicted ecDNA coordinates match simulated ground-truth coordinates when precision/recall are computed per condition
- Snakemake pipeline completes all rules defined in Snakefile without errors using the edited config.yaml and -j 16
- On real data, calls should recapitulate known ecDNA biology, e.g., 1312 ecDNAs from 94 ATAC-Seq datasets containing 37 amplified oncogenes

## Limitations

- Accuracy of ecDNA calls for all types of focal amplifications cannot be guaranteed; the benchmarking strategy has limitations
- Rate of failures on complex structural variants not yet explored; other focal amplification classes with misregulated chromatin may be detected as false positives
- Application to single-cell ATAC-Seq data has yet to be explored
- Short read-length sequencing may lead circlehunter to discover computational sub-structures (e.g., COLO320DM)
- Most analyzed samples were cell lines with skewed cancer-type proportions; additional data are needed, including on whether all ecDNAs share reduced higher-order compaction

## Evidence

- [abstract] circlehunter abstract: "We developed circlehunter ( https://github.com/suda-huanglab/circlehunter ), a tool for identifying ecDNA from ATAC-Seq data"
- [abstract] circlehunter performance: "circlehunter has an F1 score of 0.93 at 30× local depth and read lengths as short as 35 bp"
- [abstract] scale of application: "Based on 1312 ecDNAs predicted from 94 publicly available datasets of ATAC-Seq assays, we found 37 oncogenes contained in these ecDNAs"
- [methods] pipeline launch: "snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml"
- [readme] mamba install: "All dependencies can be installed using `mamba`"
- [readme] config setup: "circlehunter use a `yaml` format config file to figure out all samples and their output"
- [discussion] benchmark caveat: "it is important to note that the current benchmarking strategy has limitations, and we cannot guarantee the accuracy of ecDNA calls for all types of focal amplifications"
- [discussion] single-cell gap: "However, the application of circlehunter to single-cell ATAC-Seq data has yet to be explored"
