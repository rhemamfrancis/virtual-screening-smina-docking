# Structure-Based Virtual Screening of ALK Inhibitors with smina

Virtual screening of a 500-compound library sample against the ATP-binding pocket of anaplastic lymphoma kinase (ALK, PDB 4Z55) using smina, with method validation by redocking and ROC analysis against known ChEMBL actives and inactives. Completed for RBIF110 (Brandeis University, Fall 2026).

## Overview
ALK is a receptor tyrosine kinase driving cancers such as non-small cell lung cancer, anaplastic large cell lymphoma, and neuroblastoma. This project prepares and filters a compound library, validates a docking protocol against the co-crystallized ligand and known ALK actives and inactives from ChEMBL, and docks a random sample of 500 library compounds. The validated setup achieved an AUC of 0.784, and the top-scoring library compounds (-9.19 to -9.42 kcal/mol) were placed in the same cleft as the cognate ligand.

## Target Selection
- **Protein:** ALK kinase domain, PDB 4Z55 (1.55 Å, chain A, residues 1072-1410)
- **Selection process:** Compared the four highest-resolution kinase-domain structures (4Z55, 9GBE, 5A9U, 8ARJ) by inspecting the TITLE, HETNAM, and SEQADV records for bound ligands and mutations.
- **Why 4Z55:** Best resolution; its only mutation (S1281G) sits in the activation loop, away from the ligand site. 9GBE (G1202R/L1196M), 5A9U (C1156Y), and 8ARJ (unusual ligand chemistry) were excluded.
- **Preparation:** Receptor (protein only) and cognate ligand (4LO) separated and saved with PyMOL.
- **Binding site:** smina `--autobox_ligand` using the 4LO crystal pose. Polar contacts with the backbone of Glu1197 and Met1199 (3.5 Å cutoff) confirm the ligand sits in the ATP pocket.

## Library Preparation
- **Source:** Course-provided SDF of 18,748 compounds with 3D coordinates [not included in this repo]
- **Cleaning pipeline (RDKit):**
  1. Desalting with the largest-fragment chooser (0 multi-fragment molecules found)
  2. Canonicalization and de-duplication: 18,748 to 17,759 (989 duplicates removed)
  3. PAINS filtering: 4,468 flagged, leaving **13,291** compounds
  4. Descriptor profiling (MW, cLogP, HBD, HBA, TPSA, rotatable bonds, rings)
- **Library profile:** Mostly MW 250-450 Da, mean cLogP 0.7, mean TPSA 103 Å², about 5 rotatable bonds, 1-2 HBD, about 5 HBA.
- **Screening sample:** Docking all 13,291 compounds was estimated at about 40 hours (about 10.8 s per compound at exhaustiveness 1), which exceeds Colab session limits. A random sample of **500 compounds** (seed 42) was docked in two chunks of 250.

## Docking Protocol
- **Software:** smina (static build from SourceForge) [version]
- **Settings:** `--exhaustiveness 1`, `--num_modes 1`, search box from `--autobox_ligand` (4LO)
- **Scoring:** smina default empirical scoring function, reported as `minimizedAffinity` (kcal/mol; more negative means stronger predicted binding)
- **Redocking:** 4LO rebuilt from its SMILES (RDKit embedding plus MMFF minimization) and redocked. The best pose scored -9.3 kcal/mol and reproduced the crystallographic binding mode by visual overlay in PyMOL, with small deviations at the methylpiperidine terminus. RMSD was not calculated.

## Validation (ROC Analysis)
- **Data:** ALK activity data from ChEMBL (CHEMBL4247). Actives were pChEMBL > 8 plus the cognate ligand (890 after cleaning); inactives were > 10 µM (233).
- **Test set:** 50 random actives and 100 random inactives (seed 42), docked with the same protocol.
- **Result:** **AUC = 0.784** vs. 0.50 for random ranking. The four top-scoring compounds were all known actives, and about 74% of actives were recovered at a 26% false-positive rate.
- **Interpretation:** The setup enriches known ALK actives, which is notable given that the ChEMBL inactives are structurally similar to the actives and so harder to discriminate than random decoys. The AUC is approximate because the test set is small.

## Screening Results
- **Scores (500 compounds):** mean -7.20, median -7.20, SD 0.76, range -9.55 to -4.06 kcal/mol
- **Top 3 library compounds (excluding the known ChEMBL active that ranked first overall):**

| Rank | Compound | Score (kcal/mol) |
|---|---|---|
| 1 | ZINC000020869469 | -9.42 |
| 2 | ZINC000002104998 | -9.28 |
| 3 | ZINC000009009863 | -9.19 |

- **Poses:** All three sit in the same cleft as 4LO and overlap its central core, consistent with ATP-site binding. None reproduces the full footprint of 4LO.
- **Limitations:** Poses were compared by eye in PyMOL, with no hydrogen-bond or hinge-contact analysis. Docking used a rigid receptor and low exhaustiveness for speed, and only 500 of 13,291 compounds were screened. Results are predictions with no experimental validation.


## Repository Contents
- `ALK_virtual_screening.ipynb`: full workflow (target selection, library prep, validation, screening, pose analysis)
- `figures/`: descriptor histograms, score distribution, redocking and top-pose images
- `results/`: `libscores.csv` (docking scores for the 500 screened compounds)

## Tools and Versions
Python, RDKit 2026.09.1, smina, PyMOL (open-source), py3Dmol, mols2grid, pandas, NumPy, scikit-learn, matplotlib, Google Colab

## Setup Notes
smina was downloaded as a static binary in Colab:
`wget https://sourceforge.net/projects/smina/files/smina.static/download -O smina.static`
The full workflow takes several hours on Colab, so results are preserved in the saved notebook outputs rather than requiring a rerun.

## Data and Attribution
Receptor structure: Protein Data Bank (4Z55). Activity data: ChEMBL (CHEMBL4247).

## Author
Rhema Francis, MS Bioinformatics, Brandeis University
