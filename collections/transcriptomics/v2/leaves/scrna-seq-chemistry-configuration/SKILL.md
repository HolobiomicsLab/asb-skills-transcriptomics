---
name: scrna-seq-chemistry-configuration
description: Use when FASTQ files from a UMI-based 3' scRNA-seq platform (e.g., Chromium,
  Drop-seq, ICELL8, SmartSeq3) must be processed with a single unified pipeline and
  the platform-specific chemistry is unknown or should be auto-detected.
when_to_use_negative:
- The data are 5' scRNA-seq from a technology other than Chromium — the article states
  such technologies are not supported at the time of writing
- The input is not UMI-based scRNA-seq data — UniverSC supports only UMI-based platforms
- Cell Ranger's own chemistry/whitelisting behavior must be bypassed entirely, since
  UniverSC calls Cell Ranger as an external dependency and inherits its processing
  model
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: UniverSC
  role: Universal processing pipeline that auto-detects the chemistry for 3' scRNA-seq
    technologies and wraps Cell Ranger to produce GBMs across platforms
  repo: https://github.com/minoda-lab/universc
- name: Cell Ranger 3.0.2
  role: External dependency invoked by UniverSC to perform the actual single-cell
    data processing after chemistry configuration
- name: Docker
  role: Container image (tomkellygenetics/universc) providing a consistent execution
    environment for the chemistry-configured pipeline
- name: bamtofastq
  role: Converts Cell Ranger 1.1.0 BAM output back to FASTQ so it can be reprocessed
    under the unified chemistry configuration
  repo: https://github.com/10XGenomics/bamtofastq
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-34681-z
    title: A flexible cross-platform single-cell data processing pipeline
  source_content_hash: sha256:154b8be8e703cd47a113def85dd34afbf12303a2bdbd708e4f5522526aad5933
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/minoda-lab/universc
  tools:
  - Cell Ranger 3.0.2
  - Docker
  - UniverSC
  - bamtofastq
  - dropSeqPipe
  - seqtk
  - zUMIs
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# scRNA-seq-chemistry-configuration

## Summary

Configure UniverSC (v1.2.7) to automatically determine the correct UMI-based 3' scRNA-seq chemistry for a FASTQ dataset, wrapping Cell Ranger 3.0.2 so that one pipeline can process Chromium, Drop-seq, ICELL8, and SmartSeq3 data identically.

## When to use

Use when FASTQ files from a UMI-based 3' scRNA-seq platform (e.g., Chromium, Drop-seq, ICELL8, SmartSeq3) must be processed with a single unified pipeline and the platform-specific chemistry is unknown or should be auto-detected.

## When NOT to use

- The data are 5' scRNA-seq from a technology other than Chromium — the article states such technologies are not supported at the time of writing
- The input is not UMI-based scRNA-seq data — UniverSC supports only UMI-based platforms
- Cell Ranger's own chemistry/whitelisting behavior must be bypassed entirely, since UniverSC calls Cell Ranger as an external dependency and inherits its processing model

## Inputs

- FASTQ files (R1/R2) from a UMI-based 3' scRNA-seq platform (Chromium, Drop-seq, ICELL8, SmartSeq3)
- Cell Ranger BAM output converted to FASTQ with 10x Genomics bamtofastq (for Chromium 1.1.0 data)
- Reference genome (GRCh38/hg38 chromosome 21 for the benchmarking setup)
- Cell Ranger 3.0.2 installation (local or via the tomkellygenetics/universc Docker image)

## Outputs

- Gene-barcode matrix (GBM) processed by a single unified pipeline
- Chemistry determination result for the input platform (automatic for 3' technologies)
- Pearson correlation value versus a platform-native pipeline GBM (validation)

## How to apply

Run UniverSC on the platform FASTQ inputs and let it automatically determine the chemistry appropriate for each 3' scRNA-seq technology rather than hard-coding platform-specific settings. UniverSC operates as a shell wrapper around Cell Ranger 3.0.2 (the version it uses due to licensing), available as a Docker image (tomkellygenetics/universc) for reproducible execution; `launch_universc.sh` needs write access to a Cell Ranger installation in a user's home directory, and no admin rights are required. Use the GRCh38 (hg38) reference as in the paper when benchmarking against platform-native pipelines (Cell Ranger, dropSeqPipe, CogentAP, zUMIs). After generating the gene-barcode matrix (GBM), harmonize with the native pipeline's GBM — keep only barcodes present in both matrices and add genes found in only one with 0 UMIs — then compute Pearson correlation per gene/cell UMI count pairs to confirm the cross-pipeline agreement (r = 1 for Chromium vs Cell Ranger 3.0.2; r >= 0.94 for the other three platforms).

## Related tools

- **UniverSC** (Universal processing pipeline that auto-detects the chemistry for 3' scRNA-seq technologies and wraps Cell Ranger to produce GBMs across platforms) — https://github.com/minoda-lab/universc
- **Cell Ranger 3.0.2** (External dependency invoked by UniverSC to perform the actual single-cell data processing after chemistry configuration)
- **Docker** (Container image (tomkellygenetics/universc) providing a consistent execution environment for the chemistry-configured pipeline)
- **bamtofastq** (Converts Cell Ranger 1.1.0 BAM output back to FASTQ so it can be reprocessed under the unified chemistry configuration) — https://github.com/10XGenomics/bamtofastq

## Examples

```
./launch_universc.sh -1 sample_R1.fastq.gz -2 sample_R2.fastq.gz --reference refdata-grch38-chr21
```

## Evaluation signals

- UniverSC's automatic chemistry determination succeeds for the input FASTQs without manual technology specification
- Chromium GBM from UniverSC has Pearson correlation r = 1 with Cell Ranger 3.0.2 GBM after barcode/gene harmonization
- Drop-seq, ICELL8, and SmartSeq3 GBMs each reach Pearson correlation r >= 0.94 versus dropSeqPipe, CogentAP, and zUMIs outputs
- Only barcodes present in both compared GBMs are retained, and genes unique to one GBM are added to the other with 0 assigned UMIs before correlation
- Clustering (Seurat) and ARI between UniverSC and native-pipeline outcomes are consistent with the Fig. 2 left-column benchmark result

## Limitations

- 5' scRNA-seq technologies other than Chromium are not supported at the time of writing
- UniverSC utilises Cell Ranger version 3.0.2 due to licensing; later Cell Ranger versions are not used
- Only reads mapped to chromosome 21 were kept in the benchmark, so correlation figures apply to the reduced reference; results depend on identical barcode handling when comparing pipelines
- UniverSC is not affiliated with or endorsed by 10X Genomics; it calls Cell Ranger as an external dependency requiring write access in a user's home directory

## Evidence

- [methods] Automatic chemistry determination: [evidence span withheld — anchor status: not_found]
- [discussion] Cell Ranger dependency: "It is of note that UniverSC utilises Cell Ranger version 3.0.2 due to licensing"
- [methods] 5' technology limitation: [evidence span withheld — anchor status: not_found]
- [intro] Wrapper architecture: "UniverSC; a shell utility that operates as a wrapper for Cell Ranger (10x Genomics) that can handle datasets generated by a wide range of single-cell technologies"
- [readme] Installation requirement: [evidence span withheld — anchor status: external_doc]
- [readme] Supported scope: [evidence span withheld — anchor status: external_doc]
