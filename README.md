# 🧬 Structural Mapping of EGFR Mutations to Predict Drug Resistance

> A structural bioinformatics pipeline that links clinical **EGFR** mutations (DepMap) to **binding-pocket changes** (FoldX + fpocket) and **drug-binding energies** (AutoDock Vina) for four tyrosine kinase inhibitors used in Non-Small Cell Lung Cancer (NSCLC).

**Course:** CENG 4525 – Final Project
**Team:** Begüm Başovalı · Bartu Paçal · Vuslat Sülbiye Türk

---

## 📌 Overview

Resistance to EGFR tyrosine kinase inhibitors (TKIs) is a major problem in NSCLC treatment. This project asks:

> *Can changes in the ATP-binding pocket caused by EGFR mutations explain and predict how well different TKIs bind?*

Pipeline summary:

1. Integrate mutation, drug-information and drug-response data from **DepMap** via `ModelID`.
2. Filter for **EGFR** and four TKIs: **Gefitinib, Erlotinib, Afatinib, Osimertinib**.
3. Clean the PDB structures (remove water, ions and other heteroatoms).
4. Introduce point mutations into the wild-type scaffold **7SI1** with **FoldX**.
5. Measure pocket volume, druggability, hydrophobicity and surface area with **fpocket**.
6. Dock each drug against WT and mutant receptors with **AutoDock Vina**.
7. Correlate pocket descriptors, docking energies and cell-line IC50 values.

---

## 📁 Repository Contents

| File | Description |
|------|-------------|
| `CENG_4525_EGFR_Final.ipynb` | Full Google Colab pipeline: cleaning, FoldX, fpocket, docking, analysis and plots |
| `CENG4525_FinalReport.pdf` | Final project report |

> Adjust the file names above to match the names in your repository.

---

## 🗃️ Data Sources

| Source | Data | Used for |
|--------|------|----------|
| DepMap Public 25Q3 | `OmicsSomaticMutations.csv` | EGFR somatic mutations per cell line |
| PRISM Repurposing 24Q2 | Primary data matrix + compound list | Drug metadata and response (LFC / IC50) |
| RCSB PDB | `7SI1` | Wild-type EGFR kinase domain scaffold for mutagenesis and docking |
| RCSB PDB | `1M17`, `4WKQ`, `4G5J`, `4ZAU` | Holo structures (erlotinib, gefitinib, afatinib, osimertinib) used to locate the binding site |

Derived files used in the analysis: `A_mutation_summary.csv`, `EGFR_drug_response.csv`, `docking_analysis_complete.csv`.

---

## 🔬 Methods

### 1. Structure preparation
Raw PDB files were cleaned with **Biopython** (water, ions and other heteroatoms removed; protein chains and target ligands kept). A before/after atom-count comparison was used as quality control.

### 2. Binding-site coordinates
fpocket finds pockets by geometry, not function, so the largest pocket is not necessarily the drug site. The docking box was therefore defined **manually in PyMOL** (local GUI) from the ligand position in each holo structure, then fed back into the Colab pipeline.

| Holo structure | Center (x, y, z) |
|----------------|------------------|
| 4G5J (afatinib) | 50.80, 1.68, -20.39 |
| 4ZAU (osimertinib) | 49.46, 0.95, -17.32 |
| 4WKQ (gefitinib) | 49.16, 5.82, -21.61 |
| 1M17 (erlotinib) | 50.81, 0.58, -20.14 |

### 3. In silico mutagenesis (FoldX)
17 candidate mutations were retrieved; only three fall inside the resolved 7SI1 kinase-domain region and were modeled:

**E758K · L858R · A864V**

### 4. Pocket analysis (fpocket)
Volume, druggability score, hydrophobicity, surface area and number of alpha spheres were compared between WT and each mutant.

### 5. Molecular docking (AutoDock Vina)
Ligands were built from SMILES with Open Babel, receptors and ligands converted to PDBQT, and each of the four drugs docked against WT and the three mutants (16 runs). ΔΔG = mutant − WT best-mode affinity.

### 6. Statistical analysis
Pearson correlations between pocket-descriptor changes, docking ΔΔG and cell-line `LN_IC50`; comparison of physicochemical properties of the four drugs.

---

## 📊 Results

### Pocket changes vs. wild type

| Variant | Volume (Å³) | Druggability | Hydrophobicity | Surface area (Å²) |
|---------|:-----------:|:------------:|:--------------:|:-----------------:|
| WT | 347.09 | 0.466 | -0.857 | 98.59 |
| E758K | 362.54 (+15.4) | 0.192 (-0.27) | -3.625 (-2.77) | 132.27 (+33.7) |
| L858R | 204.95 (-142.1, ≈ -41%) | 0.186 (-0.28) | ≈ WT (+0.02) | 61.82 (-36.8) |
| A864V | 348.64 (+1.6) | 0.466 (0.00) | ≈ WT | 98.59 (0.00) |

E758K was also the only variant whose top fpocket pocket differed from WT.

### Best docking affinity (kcal/mol, more negative = stronger)

| Receptor | Gefitinib | Erlotinib | Afatinib | Osimertinib |
|----------|:---------:|:---------:|:--------:|:-----------:|
| WT | -2.016 | -6.913 | -7.675 | -7.777 |
| E758K | -1.908 | -7.136 | -7.759 | -8.120 |
| L858R | -1.953 | -6.990 | -7.715 | -7.944 |
| A864V | -1.972 | -7.037 | -7.659 | -8.045 |

### Main observations
- **Osimertinib** binds most strongly overall and improves in all three mutants.
- **Gefitinib** is the only drug that loses affinity in every mutant.
- **Erlotinib** shows slightly improved affinity in all mutants.
- **Afatinib** stays roughly constant (between -7.66 and -7.76 kcal/mol).
- Pocket **volume** change alone does not predict docking affinity (R² between 0.002 and 0.082, all non-significant).
- Across 12 drug–mutation pairs, docking ΔΔG shows only a weak, non-significant correlation with cell-line LN_IC50 (**r = -0.12, p = 0.71**).

---

## ⚠️ Limitations

- **Only three mutations** (and three mutant models) could be mapped onto the 7SI1 scaffold, so all correlations rest on very few points.
- **Small docking differences.** Most ΔΔG values are between 0.02 and 0.35 kcal/mol, which is within the typical error of Vina scoring and should not be read as proof of resistance or sensitivity.
- **Gefitinib scores (~ -2 kcal/mol) are unusually weak** compared with the other drugs and with typical values for a nanomolar inhibitor. This suggests a docking-setup issue (box position/size, ligand preparation) rather than real biology and should be checked before interpreting Gefitinib results.
- **No experimental validation.** Static docking against single FoldX models ignores protein flexibility, covalent binding (afatinib and osimertinib bind Cys797 covalently), drug transport and metabolism. Molecular dynamics and covalent docking would be natural next steps.
- Cell-line IC50 data come from a small number of EGFR-mutant lines, and docking-vs-IC50 agreement is weak.

---

## 🛠️ Tools & Environment

Everything ran on **Google Colab**, except the PyMOL coordinate extraction which used a local desktop PyMOL.

| Purpose | Tool |
|---------|------|
| Structure handling | Biopython, PyMOL |
| Mutagenesis | FoldX |
| Pocket detection | fpocket |
| Ligand preparation | Open Babel, MGLTools / AutoDockTools |
| Docking | AutoDock Vina |
| Analysis & plots | pandas, NumPy, SciPy, Matplotlib, seaborn |

---

## 🚀 How to Reproduce

1. Open `CENG_4525_EGFR_Final.ipynb` in **Google Colab**.
2. Mount Google Drive and place the input PDB files and CSV files where the first cells expect them (edit the paths at the top of the notebook if needed).
3. Obtain the **FoldX** binary yourself (academic license, not redistributed here) and upload it to the Colab session.
4. Run the cells in order: cleaning → fpocket → FoldX → fpocket on mutants → docking → analysis.
5. The PyMOL GUI step for the grid-box centers must be done locally; the resulting coordinates are listed above.

---

## 📚 Report

Full methodology, figures and discussion are in [`CENG4525_FinalReport.pdf`](./CENG4525_FinalReport.pdf).
