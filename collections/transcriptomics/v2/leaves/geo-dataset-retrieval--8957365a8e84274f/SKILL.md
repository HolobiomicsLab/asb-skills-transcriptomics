---
name: geo-dataset-retrieval
description: Use when the goal is to reproduce a published differential methylation
  analysis from raw array data, and the source dataset (e.g., GSE207426) is available
  on GEO as IDAT files with a sample sheet.
when_to_use_negative:
- The analysis starts from already normalized beta-value matrices — methylR's ChAMP
  module expects raw IDAT input, so only downstream modules (Volcano plot, PCA/MDS,
  enrichment) apply to preprocessed tables.
- The dataset is not a DNA methylation array study (e.g., WGBS or RNA-seq), since
  methylR covers 450K and EPIC Illumina arrays only.
- Only a quick functionality check of the interface is needed — use the bundled methylR
  test data (testData.zip) instead of downloading a full GEO dataset.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3295
tools:
- name: methylR
  role: Target analysis environment into which the retrieved GEO IDAT files and sample
    sheet are loaded for ChAMP-based differential methylation calling
  repo: https://github.com/JD2112/methylr
- name: Docker
  role: Container runtime used to launch methylR locally after data retrieval; image
    built for AMD64 architecture
  repo: https://github.com/JD2112/methylr-devel
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad184
    title: 'methylR: a graphical interface for comprehensive DNA methylation array
      data analysis'
  source_content_hash: sha256:4cd0d79e5901ddf97ec592c582a8912c652639244c064404225a4103b82ed319
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JD2112/methylr
  tools:
  - Docker
  - methylR
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# geo-dataset-retrieval

## Summary

Retrieve a publicly available GEO DNA methylation array dataset (raw IDAT files plus sample sheet) so it can be loaded into an analysis pipeline such as methylR and used to independently reproduce published differential methylation results.

## When to use

Use when the goal is to reproduce a published differential methylation analysis from raw array data, and the source dataset (e.g., GSE207426) is available on GEO as IDAT files with a sample sheet.

## When NOT to use

- The analysis starts from already normalized beta-value matrices — methylR's ChAMP module expects raw IDAT input, so only downstream modules (Volcano plot, PCA/MDS, enrichment) apply to preprocessed tables.
- The dataset is not a DNA methylation array study (e.g., WGBS or RNA-seq), since methylR covers 450K and EPIC Illumina arrays only.
- Only a quick functionality check of the interface is needed — use the bundled methylR test data (testData.zip) instead of downloading a full GEO dataset.

## Inputs

- GEO accession identifier (e.g., GSE207426)
- Raw IDAT files (Red and Green channel pairs) for 450K/EPIC Illumina arrays
- GEO sample sheet with per-sample phenotype/group information

## Outputs

- Local directory of IDAT files and sample sheet ready to load into the methylR Shiny interface
- Verified sample inventory matching the GEO record (sample count and group labels)

## How to apply

Identify the GEO accession (e.g., GSE207426 glioblastoma methylation dataset) cited by the publication whose results you intend to reproduce, and download the raw DNA methylation array files — the IDAT files plus the sample sheet describing group membership. Because methylR analyzes raw 450K/EPIC Illumina array data rather than preprocessed matrices, obtain raw IDATs rather than beta-value tables. Prepare the sample sheet so it maps each IDAT to its phenotype group, as methylR's ChAMP module requires group information for differential methylation calling. If only a smoke test is needed, the methylR project instead ships test data (testData.zip from SourceForge and testDataFile1/testDataFile2 in the repo's data directory), but full reproduction requires the GEO-derived raw files. Validate retrieval by checking that the number of samples in the sample sheet matches the GEO record and that all IDAT pairs (Red/Green channels) are present before launching the methylR Docker container (AMD64 only) at http://localhost:3838.

## Related tools

- **methylR** (Target analysis environment into which the retrieved GEO IDAT files and sample sheet are loaded for ChAMP-based differential methylation calling) — https://github.com/JD2112/methylr
- **Docker** (Container runtime used to launch methylR locally after data retrieval; image built for AMD64 architecture) — https://github.com/JD2112/methylr-devel

## Examples

```
docker run --rm -p 3838:3838 jd21/methylr:latest  # then load GEO IDAT files + sample sheet at http://localhost:3838
```

## Evaluation signals

- Number of samples in the downloaded sample sheet equals the sample count reported for the GEO accession (e.g., full GSE207426 cohort).
- All IDAT Red/Green channel file pairs are present and non-empty for every sample listed in the sample sheet.
- Loading the retrieved data into methylR's Shiny interface completes ChAMP loading, QC, and normalization without sample-drop errors.
- For reproduction of the reference analysis, the resulting DMC count matches the published figure (e.g., 123,510 DMCs for full-dataset GSE207426).

## Limitations

- methylR's Docker container is built for AMD64 only; the ARM64 chipset architecture is not supported, which constrains where retrieval-then-analysis can be run.
- The methylR paper validated the pipeline on Linux-based systems; other environments may need Singularity workarounds.
- methylR provides no sequencing-level raw data handling — retrieval must target array (IDAT) assets, not sequencer reads.

## Evidence

- [abstract] methylR analyzes raw Illumina array data: "We evaluated a publicly available dataset and compared the published results with those obtained with our tool"
- [methods] ChAMP or minfi analysis modules operate in methylR: "after one complete analysis (either with *ChAMP* or *minfi*), close the browser (to clear the temporary memory) and start again."
- [methods] ARM64 unsupported for local methylR runs: "NOTE: the ARM64 chipset architecture is actually not supported by methylR."
- [readme] methylR Docker run command for local analysis: [evidence span withheld — anchor status: external_doc]
- [readme] methylR test data location for smoke tests: [evidence span withheld — anchor status: external_doc]
