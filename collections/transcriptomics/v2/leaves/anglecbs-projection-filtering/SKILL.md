---
name: anglecbs-projection-filtering
description: Use when unrolling rolled intestinal tissue imaging data in IntestLine
  and misprojected points appear in the stretched output after backbone-based projection;
  the angleCBS parameter is the tool's dedicated control for removing them.
when_to_use_negative:
- The points to remove are noisy cells or contaminants rather than misprojected projections
  — use the Z-score filtering instead of angleCBS.
- The input provides no xy-coordinates or is not imaging data of rolled/Swiss-rolled
  intestinal tissue, which is outside IntestLine's stated scope.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: IntestLine
  role: Shiny application whose unroll step exposes the angleCBS filter used to remove
    misprojected points from the linear mapping.
  repo: https://github.com/SchlitzerLab/IntestLine
- name: Shiny
  role: R web-application framework in which IntestLine runs, providing the interactive
    panel where angleCBS is manipulated.
- name: FASTGenomics
  role: Hosted implementation of the IntestLine app where the angleCBS-based unroll
    can be run without local installation.
  repo: https://beta.fastgenomics.org/a/intestline
provenance:
  source_task_ids:
  - task_005
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad140
    title: 'IntestLine: a shiny-based application to map the rolled intestinal tissue
      onto a line'
  source_content_hash: sha256:bcdddb73af5ff8a3c1d61843e58c33c1ca86f446d9b9aaf254563f49dac81894
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SchlitzerLab/IntestLine
  tools:
  - Docker
  - FASTGenomics
  - IntestLine
  - Shiny
  - dplyr
  - ggplot2
  - ggtext
  - igraph
  - plotly
  - readr
  - tidyr
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# angleCBS-projection-filtering

## Summary

Applies the angleCBS filter inside IntestLine's unroll step to remove misprojected points when mapping Swiss-rolled intestinal tissue images (CODEX or 10x spatial transcriptomics xy-coordinates) onto a linear structure.

## When to use

Use when unrolling rolled intestinal tissue imaging data in IntestLine and misprojected points appear in the stretched output after backbone-based projection; the angleCBS parameter is the tool's dedicated control for removing them.

## When NOT to use

- The points to remove are noisy cells or contaminants rather than misprojected projections — use the Z-score filtering instead of angleCBS.
- The input provides no xy-coordinates or is not imaging data of rolled/Swiss-rolled intestinal tissue, which is outside IntestLine's stated scope.

## Inputs

- CODEX-exported .csv file containing x and y coordinates (also applicable to 10x spatial transcriptomics images providing xy-coordinates)
- Backbone selection (mouse-clicked points from the spiral center outward, or a previously uploaded backbone file)
- angleCBS filter parameter setting

## Outputs

- Unrolled (stretched) linear mapping of cells with misprojected points removed
- Report of how many cells and from which regions are lost by the angleCBS and Z-score filters
- Marker-overlay visualization along the length and thickness of the organ

## How to apply

After uploading a CODEX-exported .csv containing x and y coordinates and defining the ROI (center plus elliptical parameters a and b), create the backbone by clicking from the spiral center outward — the README warns 'You must start from the center and go outward !'. In the unroll panel, manipulate the angleCBS filter so that points misprojected during the directionally biased nearest-neighbor assignment to the backbone are filtered out. Use the Z-score filtering as a complementary control for noisy points or contaminants, keeping the two filters' roles distinct. At each adjustment, examine how many cells and from which regions are lost so that over-filtering does not discard biologically relevant regions of the intestinal wall. Judge success by previewing the backbone as a clean 3D spiral and confirming the unrolled, stretched image preserves proximal-distal and serosa-luminal structure for marker overlay.

## Related tools

- **IntestLine** (Shiny application whose unroll step exposes the angleCBS filter used to remove misprojected points from the linear mapping.) — https://github.com/SchlitzerLab/IntestLine
- **Shiny** (R web-application framework in which IntestLine runs, providing the interactive panel where angleCBS is manipulated.)
- **FASTGenomics** (Hosted implementation of the IntestLine app where the angleCBS-based unroll can be run without local installation.) — https://beta.fastgenomics.org/a/intestline

## Examples

```
docker run --name IntestLine -it -p 3838:3838 altayyuzeir/intestline bash
```

## Evaluation signals

- Backbone preview shows a high-quality 3D spiral shape before unrolling, confirming the backbone the angleCBS projection depends on is valid.
- The app reports how many cells and from which regions are lost, so the filter can be checked against excessive cell loss.
- The unrolled output shows misprojected points removed while the stretched image still supports marker overlay along both the length and thickness of the organ.
- On the FASTGenomics server (2 vCPUs @2.1GHz, 16 GB RAM), the unroll of the 10k-cell demo dataset completes in ~26 s, providing a runtime sanity check.

## Limitations

- The angleCBS filter removes misprojected points, but noisy points or contaminants require the separate Z-score filtering; neither setting is given a numeric default in the README.
- Filtering discards cells — the user must examine how many cells and from which regions are lost, since relevant tissue regions may be dropped.
- The unroll step is computationally expensive on limited hardware: ~26 s for 10k cells and on the order of minutes for 150k cells on the FASTGenomics server (2 vCPUs @2.1GHz, 16 GB RAM).
- The Docker image might not work on Macs with the M1-chip, and the FASTGenomics option was only available until mid-2024.

## Evidence

- [methods] Misprojected points can be filtered out by manipulating the angleCBS filter.: "Misprojected points can be filtered out by manipulating the angleCBS filter."
- [readme] Noisy points or contaminants can be eliminated by using the Z-score filtering.: "Noisy points or contaminants can be eliminated by using the Z-score filtering."
- [readme] In both cases you can examine how many cells and from which regions are lost.: "In both cases you can examine how many cells and from which regions are lost."
- [readme] Upload CODEX-exported .csv file, containing x and y coordinates.: "Upload CODEX-exported .csv file, containing x and y coordinates."
