---
name: boolean-rule-inference-on-kegg-pathway-topologies
description: Use when you have a cells x genes log2(TPM+1) scRNA-seq matrix with cell-level
  condition labels (1/0 metadata) and want topology-constrained Boolean rule inference
  and dysregulated-pathway calls per cell type, rather than conventional ORA/DE.
when_to_use_negative:
- You lack cell-level condition labels and a contrast specification (scBONITA pathway
  analysis requires conditions and contrast files).
- You need organism-specific topologies outside KEGG hsa without supplying your own
  GRAPHML networks.
- Only a desktop with <10G RAM is available — README warns rule inference on large
  scRNA-seq datasets needs in excess of 10G RAM and recommends an HPC/SLURM system.
edam_operation: http://edamontology.org/operation_1781
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3407
tools:
- name: scBONITA
  role: 'Core pipeline: downloads KEGG topologies, infers Boolean rules, computes
    node importance, and runs pathway analysis'
  repo: https://github.com/Thakar-Lab/scBONITA
- name: Python 3.6
  role: Runtime for pipeline.py and pathwayAnalysis.py (scBONITA requires Python 3.6
    or newer)
  repo: https://github.com/Thakar-Lab/scBONITA
- name: conda
  role: Environment creation from the provided spec-file ('conda create --name scBonita
    --file spec-file.txt')
  repo: https://github.com/Thakar-Lab/scBONITA
- name: KEGG
  role: Source of hsa pathway topologies downloaded via --getKEGGPathways for rule
    inference
- name: SLURM
  role: Queue system running the generated sbatch jobs for rule inference
- name: GRAPHML
  role: Format for non-KEGG networks supplied via --pathwayList
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41540-022-00246-5
    title: Executable models of immune signaling pathways in HIV-associated atherosclerosis
  source_content_hash: sha256:939796be88135971892195dd4b394a49b08db87de086754cda6cef3890dc68bb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Thakar-Lab/scBONITA
  tools:
  - GRAPHML
  - KEGG
  - Python
  - Python 3.6
  - SLURM
  - conda
  - scBONITA
  edam_operation: http://edamontology.org/operation_1781
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3407
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# boolean-rule-inference-on-kegg-pathway-topologies

## Summary

Infer Boolean regulatory rules for KEGG pathway topologies from scRNA-seq data using scBONITA, then run pathway analysis to identify signaling pathways dysregulated between conditions (e.g., pre- vs post-HIV infection, AS+ vs AS- PLWH).

## When to use

Use when you have a cells x genes log2(TPM+1) scRNA-seq matrix with cell-level condition labels (1/0 metadata) and want topology-constrained Boolean rule inference and dysregulated-pathway calls per cell type, rather than conventional ORA/DE.

## When NOT to use

- You lack cell-level condition labels and a contrast specification (scBONITA pathway analysis requires conditions and contrast files).
- You need organism-specific topologies outside KEGG hsa without supplying your own GRAPHML networks.
- Only a desktop with <10G RAM is available — README warns rule inference on large scRNA-seq datasets needs in excess of 10G RAM and recommends an HPC/SLURM system.

## Inputs

- Gene x cell expression matrix (comma/tab-delimited, first column 'Genes', first row cell IDs, log2(TPM+1)-style values), e.g. Kazer et al. SCP256 PBMC data
- Metadata/conditions file: rows = cells, columns = 1/0 condition or cell-type labels (e.g., pre-infection vs 1-year post-infection; CD14_monocyte etc.)
- Contrast file: pairs of condition column labels to compare
- KEGG pathway IDs (organism hsa) or GRAPHML network files

## Outputs

- Inferred Boolean regulatory rules per KEGG pathway topology per condition
- Node importance scores (high-impact genes from perturbation analysis)
- Per-pathway dysregulation calls between the contrasted conditions (e.g., scp256_pathway_analysis_results.csv)
- sbatch/log files from SLURM rule-inference runs

## How to apply

Format the expression matrix as genes x cells (rows = genes with a 'Genes' first column, columns = cell IDs matching metadata), a 1/0 metadata file (rows = cells, columns = condition/cell-type labels), and a contrast file listing condition pairs to compare. Modify the C file NODE and CELL parameters to match dataset size, compile with 'make', create the scBonita conda environment from the provided spec-file, and run setup (e.g., 'python3.6 pipeline.py --dataFile example.csv --fullPipeline 1 --maxNodes 20000 --separator "," --listOfKEGGPathways "00010" "00020" --getKEGGPathways 1 --organism hsa') to download KEGG hsa pathway topologies and generate sbatch files. Run rule inference and node-importance scoring per cell type (e.g., cytotoxic T cells, monocytes) using pre-classified cell labels from the metadata, monitoring the SLURM queue and logs. Then run pathway analysis with 'python3.6 pathwayAnalysis.py --dataFile trainingData.csv --conditions conditions.txt --contrast contrast.txt --conditions_separator comma --contrast_separator comma --pathwayList <GRAPHML networks>' to score pathway dysregulation between conditions. The rationale is that scRNA-seq samples the state space of a dynamic Boolean network, letting topology-constrained rules capture coherent signaling programs that DE/ORA miss.

## Related tools

- **scBONITA** (Core pipeline: downloads KEGG topologies, infers Boolean rules, computes node importance, and runs pathway analysis) — https://github.com/Thakar-Lab/scBONITA
- **Python 3.6** (Runtime for pipeline.py and pathwayAnalysis.py (scBONITA requires Python 3.6 or newer)) — https://github.com/Thakar-Lab/scBONITA
- **conda** (Environment creation from the provided spec-file ('conda create --name scBonita --file spec-file.txt')) — https://github.com/Thakar-Lab/scBONITA
- **KEGG** (Source of hsa pathway topologies downloaded via --getKEGGPathways for rule inference)
- **SLURM** (Queue system running the generated sbatch jobs for rule inference)
- **GRAPHML** (Format for non-KEGG networks supplied via --pathwayList)

## Examples

```
python3.6 pipeline.py --dataFile "data.csv" --fullPipeline 1 --maxNodes 20000 --separator "," --listOfKEGGPathways "00010" "00020" --getKEGGPathways 1 --organism hsa
```

## Evaluation signals

- Reproduces the qualitative published result that cell migration pathways are dysregulated in early-stage HIV infection for the SCP256/Kazer dataset.
- Per-cell-type dysregulated sets match published findings: 5 signaling pathways in cytotoxic T cells and 5 of 19 optimized pathways in monocytes shared between post-infection and AS+ PLWH.
- Input files pass schema checks: expression matrix genes x cells with matching cell IDs between data and metadata; metadata entries are 1/0; contrast pairs match metadata column names.
- KEGG hsa topologies downloaded successfully for all requested pathway IDs and C NODE/CELL parameters >= dataset dimensions before 'make'.
- SLURM jobs complete without package/dependency errors in log files for each cell type subset.

## Limitations

- Large scRNA-seq datasets require in excess of 10G RAM; rule inference and attractor analysis are not recommended on a desktop.
- scBONITA assumes scRNA-seq data samples the state space of a dynamic Boolean network (used in lieu of time-series data), which is rarely available in human studies.
- Inference is expected to improve with pure cell populations; the published runs used mixed PBMC subpopulations and pre-classified labels to reduce variability.
- Conventional ORA on DE genes failed to identify coherent migration-related processes, so DE/ORA outputs are not a substitute validation baseline.

## Evidence

- [readme] README setup command: "python3.6 pipeline.py --dataFile "example.csv" --fullPipeline 1 --maxNodes 20000 --separator "," --listOfKEGGPathways "00010" "00020" --getKEGGPathways 1 --organism hsa"
- [readme] RAM requirement: "We find that the amount of RAM required for large scRNA-seq datasets is in excess of 10G; we hence don't recommend running rule inference and attractor analysis on a desktop computer."
- [discussion] state-space hypothesis: "In lieu of using time-series data, scBONITA hypothesizes that scRNA-seq data represents samples in the state space of a dynamic Boolean network."
- [readme] pathway analysis command: "python3.6 pathwayAnalysis.py --dataFile "data/trainingData.csv" --conditions "data/conditions.txt" --contrast "data/contrast.txt" --conditions_separator "comma" --contrast_separator "comma""
- [discussion] pre-classified subpopulations rationale: "we used pre-classified subpopulations to reduce variability and to improve the"
