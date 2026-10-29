---
name: python-dependency-management
description: Use when installing a Python >= 3.5 tool such as visnormsc that pins
  external packages (matplotlib >= 2.0.2, numpy >= 1.12.1, pandas >= 0.20.1, scikit-learn
  >= 0.18.1, scipy >= 0.19.0, statsmodels >= 0.8.0) via Anaconda or the sharedCondaEnv.yml
  conda environment file.
when_to_use_negative:
- Python and all six external packages are already installed and 'python visnormscGUI.py'
  runs — re-provisioning the environment is unnecessary
- The goal is to run the normalization routine on a count matrix or interpret normalized_expression.csv
  — use the visnormsc normalization workflow instead; this skill covers only dependency/environment
  setup
- The toolchain is not Python-based (e.g., an R/Bioconductor or compiled-binary tool)
  — the sharedCondaEnv.yml/Anaconda route does not apply
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Anaconda
  role: Installer distribution providing the right Python version (>= 3.5) and all
    external dependencies in one platform-specific install; the README's recommended
    easiest route
  repo: https://www.continuum.io/downloads
- name: conda
  role: Environment manager invoked as 'conda env create -f sharedCondaEnv.yml' to
    recreate the exact pinned environment used for developing visnormsc
- name: visnormsc
  role: Target GUI program whose pinned dependency set the environment must satisfy;
    'python visnormscGUI.py' is the launch/verification check
  repo: https://github.com/solo7773/visnormsc
- name: matplotlib
  role: Pinned external dependency with minimum version 2.0.2 that the environment
    must resolve
- name: numpy
  role: Pinned external dependency with minimum version 1.12.1 that the environment
    must resolve
- name: pandas
  role: Pinned external dependency with minimum version 0.20.1 that the environment
    must resolve
- name: scikit-learn
  role: Pinned external dependency with minimum version 0.18.1 that the environment
    must resolve
- name: scipy
  role: Pinned external dependency with minimum version 0.19.0 that the environment
    must resolve
- name: statsmodels
  role: Pinned external dependency with minimum version 0.8.0 that the environment
    must resolve
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s12539-017-0277-9
    title: 'visnormsc: A Graphical User Interface to Normalize Single-cell RNA Sequencing
      Data'
  source_content_hash: sha256:8afb5a1ebd477a784c13c49a8e99238ac9303dddef1352c9bc4ea0aa3624306e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Anaconda
  - Python
  - conda
  - matplotlib
  - numpy
  - pandas
  - scikit-learn
  - scipy
  - statsmodels
  - visnormsc
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# python-dependency-management

## Summary

Provision a reproducible Python >= 3.5 runtime for the visnormsc single-cell RNA-seq normalization GUI using Anaconda or the repo's pinned sharedCondaEnv.yml conda environment (under directory bin), covering all six pinned external packages on Windows, GNU/Linux and macOS. Correct setup is verified by launching the GUI with 'python visnormscGUI.py'.

## When to use

Use when installing a Python >= 3.5 tool such as visnormsc that pins external packages (matplotlib >= 2.0.2, numpy >= 1.12.1, pandas >= 0.20.1, scikit-learn >= 0.18.1, scipy >= 0.19.0, statsmodels >= 0.8.0) via Anaconda or the sharedCondaEnv.yml conda environment file.

## When NOT to use

- Python and all six external packages are already installed and 'python visnormscGUI.py' runs — re-provisioning the environment is unnecessary
- The goal is to run the normalization routine on a count matrix or interpret normalized_expression.csv — use the visnormsc normalization workflow instead; this skill covers only dependency/environment setup
- The toolchain is not Python-based (e.g., an R/Bioconductor or compiled-binary tool) — the sharedCondaEnv.yml/Anaconda route does not apply

## Inputs

- sharedCondaEnv.yml pinned conda environment specification (located under directory bin of the visnormsc repository)
- Anaconda installer for the target platform (Windows, GNU/Linux, or macOS)
- Target machine with a Python >= 3.5 interpreter available (if not using the sharedCondaEnv.yml route)

## Outputs

- Active conda environment named 'visnormsc' containing the pinned external dependencies (matplotlib >= 2.0.2, numpy >= 1.12.1, pandas >= 0.20.1, scikit-learn >= 0.18.1, scipy >= 0.19.0, statsmodels >= 0.8.0)
- Working Python >= 3.5 interpreter on the system environment path from which 'python visnormscGUI.py' launches the visnormsc GUI

## How to apply

Install the latest Anaconda distribution suitable for the target platform (Windows, GNU/Linux or macOS), which the README names as the easiest way to obtain the right Python version (>= 3.5) and dependencies in one step. For exact reproducibility of the development environment, instead run 'conda env create -f sharedCondaEnv.yml', where the sharedCondaEnv.yml file sits under directory bin of the solo7773/visnormsc repository. Activate with 'source activate visnormsc' on Linux and macOS or 'activate visnormsc' on Windows — the verb is OS-specific. Verify the resolved versions satisfy the pinned minimums: matplotlib >= 2.0.2, numpy >= 1.12.1, pandas >= 0.20.1, scikit-learn >= 0.18.1, scipy >= 0.19.0, statsmodels >= 0.8.0, plus builtin stdlib modules such as tkinter. Validate the environment by typing 'python visnormscGUI.py' in the CMD prompt (Windows) or terminal (Linux/macOS); if the error 'unknown command python' appears, the installation wizard was not allowed to add python to the system environment path, so re-check the Anaconda installation.

## Related tools

- **Anaconda** (Installer distribution providing the right Python version (>= 3.5) and all external dependencies in one platform-specific install; the README's recommended easiest route) — https://www.continuum.io/downloads
- **conda** (Environment manager invoked as 'conda env create -f sharedCondaEnv.yml' to recreate the exact pinned environment used for developing visnormsc)
- **visnormsc** (Target GUI program whose pinned dependency set the environment must satisfy; 'python visnormscGUI.py' is the launch/verification check) — https://github.com/solo7773/visnormsc
- **matplotlib** (Pinned external dependency with minimum version 2.0.2 that the environment must resolve)
- **numpy** (Pinned external dependency with minimum version 1.12.1 that the environment must resolve)
- **pandas** (Pinned external dependency with minimum version 0.20.1 that the environment must resolve)
- **scikit-learn** (Pinned external dependency with minimum version 0.18.1 that the environment must resolve)
- **scipy** (Pinned external dependency with minimum version 0.19.0 that the environment must resolve)
- **statsmodels** (Pinned external dependency with minimum version 0.8.0 that the environment must resolve)

## Examples

```
conda env create -f sharedCondaEnv.yml && source activate visnormsc && python visnormscGUI.py
```

## Evaluation signals

- 'conda env create -f sharedCondaEnv.yml' completes without solver conflicts and creates an environment named 'visnormsc'
- 'source activate visnormsc' (Linux/macOS) or 'activate visnormsc' (Windows) activates cleanly and 'python --version' reports >= 3.5
- All six external packages import at or above their pinned minimums: matplotlib >= 2.0.2, numpy >= 1.12.1, pandas >= 0.20.1, scikit-learn >= 0.18.1, scipy >= 0.19.0, statsmodels >= 0.8.0
- 'python visnormscGUI.py' opens the GUI window (which requires tkinter) rather than raising 'unknown command python' or an ImportError
- The setup reproduces on all three supported platforms — Windows, GNU/Linux and macOS — with python present on the system environment path

## Limitations

- If 'unknown command python' appears, Anaconda was not added to the system environment path during installation; the README directs you to check the installation and re-allow the wizard's path option
- sharedCondaEnv.yml lives under directory bin, so the conda create step fails if invoked from a location where the file cannot be found
- The program also depends on builtin stdlib modules (tkinter, multiprocessing, webbrowser, etc.); a Python >= 3.5 build without tkinter will not launch the GUI
- Activation syntax is OS-specific — 'source activate visnormsc' on Linux/macOS versus 'activate visnormsc' on Windows — and the wrong form fails

## Evidence

- [readme] cross-platform GUI program requirement: "It is developed using the Python programming language so it is a cross-platform GUI program for Windows, GNU/Linux and macOS."
- [readme] Python version floor and package deps: "It depends on python >= 3.5 and following packages."
- [readme] Anaconda as easiest install route: "The easiest way to install the right Python version and dependencies is to download and install the latest version of [Anaconda](https://www.continuum.io/downloads) suitable for your platform."
- [readme] pinned conda environment alternative: "Another optional way is to create the same conda environment as the one used for delevoping visnormsc. After conda installation, run"
- [readme] system path failure mode: "If you encounter error like "unknown command python" then you need to check if you have installed Anaconda correctly and allowed the installation wizard"
- [methods] conda env create command and file location: "`conda env create -f sharedCondaEnv.yml`<br />where the *sharedCondaEnv.yml* file is under directory **bin**."
- [methods] OS-specific activation commands: "`source activate visnormsc` on Linux and macOS, or<br/>`activate visnormsc` on Windows."
