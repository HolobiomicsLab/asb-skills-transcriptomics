---
name: single-cell-pathway-dysregulation-analysis
description: Use when you have a scRNA-seq expression matrix (genes x cells, log2(TPM+1)-style),
  cell-level metadata with 1/0 condition labels, a contrast file, and you need to
  identify which signaling pathways are dysregulated per cell type across conditions.
when_to_use_negative:
- When the goal is differential gene expression or conventional ORA on DE genes —
  the article shows conventional ORA was inefficacious at identifying coherent molecular
  processes (migration-related pathways were not enriched among DE genes), so scBONITA-PA
  is the intended approach, but plain DE analysis is not this skill.
- When no condition contrast (pairs of cell-wise condition labels) can be defined
  — pathway analysis requires a contrast file comparing two conditions.
- When running on a desktop without adequate resources — rule inference and attractor
  analysis on large scRNA-seq datasets require in excess of 10G RAM.
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3407
tools:
- name: scBONITA
  role: Core pipeline for Boolean rule inference, pathway analysis, and attractor
    analysis on scRNA-seq data
  repo: https://github.com/Thakar-Lab/scBONITA
- name: Python 3.6
  role: Runtime for scBONITA pipeline.py and pathwayAnalysis.py scripts
  repo: https://github.com/Thakar-Lab/scBONITA
- name: conda
  role: Environment management via provided spec-file or yml file
- name: KEGG
  role: Source of hsa pathway topologies downloaded for rule inference
- name: SLURM
  role: Scheduler running generated sbatch jobs for rule inference and analysis
- name: GRAPHML
  role: Format for pathway networks passed via --pathwayList for pathway analysis
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41540-022-00246-5
    title: Executable models of immune signaling pathways in HIV-associated atherosclerosis
  source_content_hash: sha256:ee9dc273c9709685639dec17e3dffb40f780320ebec8466fa51c1b9544941dcf
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
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3407
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# single-cell-pathway-dysregulation-analysis

## Summary

Apply scBONITA pathway analysis (scBONITA-PA) to identify KEGG signaling pathways dysregulated between conditions (e.g., pre- vs post-HIV infection, AS+ vs AS- PLWH) in specific single-cell subpopulations such as cytotoxic T cells and monocytes, using inferred Boolean rules rather than conventional over-representation analysis.

## When to use

Use when you have a scRNA-seq expression matrix (genes x cells, log2(TPM+1)-style), cell-level metadata with 1/0 condition labels, a contrast file, and you need to identify which signaling pathways are dysregulated per cell type across conditions.

## When NOT to use

- When the goal is differential gene expression or conventional ORA on DE genes — the article shows conventional ORA was inefficacious at identifying coherent molecular processes (migration-related pathways were not enriched among DE genes), so scBONITA-PA is the intended approach, but plain DE analysis is not this skill.
- When no condition contrast (pairs of cell-wise condition labels) can be defined — pathway analysis requires a contrast file comparing two conditions.
- When running on a desktop without adequate resources — rule inference and attractor analysis on large scRNA-seq datasets require in excess of 10G RAM.

## Inputs

- scRNA-seq training data matrix (genes x cells, comma- or tab-delimited, first column 'Genes', first row cell IDs, log2(TPM+1)-style values)
- Metadata/conditions file (rows = cells, columns = 1/0 condition labels, first column cell IDs matching training data)
- Contrast file (each row a pair of conditions to compare, matching metadata column names)
- List of KEGG pathway IDs (e.g., '00010' '00020') or GRAPHML network files
- Pre-classified cell-type labels for subsetting cells per subpopulation

## Outputs

- Per-subpopulation Boolean regulatory rules and node importance scores
- Dysregulated-pathway flags per cell type and condition comparison
- Summary table of dysregulated optimized pathways (e.g., scp256_pathway_analysis_results.csv)

## How to apply

First download the specified KEGG (hsa) pathway topologies via scBONITA setup, e.g. 'python3.6 pipeline.py --dataFile <data.csv> --fullPipeline 1 --maxNodes <n> --separator "," --listOfKEGGPathways <KEGG IDs> --getKEGGPathways 1 --organism hsa cvThreshold None', after modifying the C file NODE and CELL parameters to match the dataset size and compiling with 'make'. Install the scBonita conda environment from the provided spec-file (PREFERRED) or yml file, and run the generated sbatch jobs through SLURM, monitoring log files (large datasets need >10G RAM). Run rule inference and node-importance scoring separately for each cell subpopulation (e.g., cytotoxic T cells and monocytes) using pre-classified cell labels from the metadata, which reduces variability and improves specificity of scBONITA-PA. Then run pathway analysis with pathwayAnalysis.py, passing the training data, conditions file (rows = cells, columns = 1/0 condition labels), contrast file (pairs of conditions to compare), and the GRAPHML pathway networks, spelling out separators as 'comma', 'space', or 'tab'. Finally, collect the optimized pathways flagged as dysregulated per cell type and compare the dysregulated sets across conditions (e.g., how many post-infection pathways are also dysregulated in AS+ PLWH), saving the summary table (e.g., scp256_pathway_analysis_results.csv).

## Related tools

- **scBONITA** (Core pipeline for Boolean rule inference, pathway analysis, and attractor analysis on scRNA-seq data) — https://github.com/Thakar-Lab/scBONITA
- **Python 3.6** (Runtime for scBONITA pipeline.py and pathwayAnalysis.py scripts) — https://github.com/Thakar-Lab/scBONITA
- **conda** (Environment management via provided spec-file or yml file)
- **KEGG** (Source of hsa pathway topologies downloaded for rule inference)
- **SLURM** (Scheduler running generated sbatch jobs for rule inference and analysis)
- **GRAPHML** (Format for pathway networks passed via --pathwayList for pathway analysis)

## Examples

```
python3.6 pathwayAnalysis.py --dataFile "data/trainingData.csv" --conditions "data/conditions.txt" --contrast "data/contrast.txt" --conditions_separator "comma" --contrast_separator "comma" --pathwayList "hsa00010"
```

## Evaluation signals

- The analysis reproduces the paper's qualitative result that cell migration pathways are dysregulated in the early stages of HIV infection (Kazer SCP256 dataset).
- Per-cell-type dysregulated pathway sets for cytotoxic T cells and monocytes are consistent with the published scBONITA pathway-analysis findings.
- Of the optimized pathways dysregulated after 1 year of HIV infection, 5 signaling pathways in cytotoxic T cells and 5 out of 19 optimized pathways in monocytes are dysregulated in both HIV infection and AS+ PLWH.
- The metadata file cell IDs match the training-data column names, and contrast-file condition names match metadata column labels.
- slurm log files show rule-inference jobs completed without dependency/package errors; output pathways are limited to the KEGG/GRAPHML networks specified at setup.

## Limitations

- Small sample size (four AS+ and four AS- subjects) can limit the power of the study; enrollment of more subjects is needed to further elucidate signaling dysregulations.
- Technical limitations of scRNA-seq cause nonspecific distortions of expression and fewer detected differences across conditions; DE methods are prone to false discoveries.
- scBONITA inference is expected to improve when pure cell populations are sequenced; current inference uses mixed PBMC subpopulations.
- In lieu of time-series data, scBONITA assumes scRNA-seq data samples the state space of a dynamic Boolean network.
- Large scRNA-seq datasets require in excess of 10G RAM; rule inference and attractor analysis are not recommended on a desktop computer.

## Evidence

- [readme] Setup command with KEGG download: "The following command runs scBONITA setup for a 20000*10000 comma-separated data set "example.csv". It downloads the KEGG pathways hsa00010 and hsa00020 for rule inference."
- [readme] Pathway analysis invocation: "`python3.6 pathwayAnalysis.py --dataFile "data/trainingData.csv" --conditions "data/conditions.txt" --contrast "data/contrast.txt" --conditions_separator "comma" --contrast_separator "comma""
- [readme] Conda environment setup: "Install the scBonita conda environment from the provided yml file OR the provided spec-file (PREFERRED), and activate the conda environment."
- [readme] Rule inference and node importance: "Step 2: Rule inference and calculation of node importance score for the networks specified in Step 1 (setup)."
- [results] Dysregulated shared pathways finding: [evidence span withheld — anchor status: not_found]
- [intro] Cell migration pathways in early HIV infection: "we used a publicly available dataset of PBMCs from persons before and after HIV infection38 to show that cell migration pathways are also dysregulated in the early stages of HIV infection"
- [discussion] Pre-classified subpopulations rationale: "While scBONITA is not strictly dependent on the clustering method used to classify scRNA-seq data into subpopulations, we used pre-classified subpopulations to reduce variability and to improve the"
- [discussion] ORA inefficacy: "This demonstrated the inefficacy of conventional ORA methods in identifying coherent molecular processes."
