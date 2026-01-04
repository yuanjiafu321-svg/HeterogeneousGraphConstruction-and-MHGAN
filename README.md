# Heterogeneous Graph Construction + MHGAN (Dynamic Edge Updating)

This repository provides a reproducible implementation of a **heterogeneous graph learning** pipeline for **slope-unit landslide susceptibility mapping (LSM)**, including:

- **Static heterogeneous graph construction** from a preprocessed (clustered + normalized) node table
- **Node–edge fusion attention** (node-attention + edge-semantic attention) 
- **Dynamic edge updating** for unlabeled nodes using **high-confidence pseudo labels**

---

## Data Confidentiality and Availability

- **This repository only releases the methodology implementation and the reproducible experimental workflow. It does not contain any raw data, sensitive attributes, or information that could lead to the re-identification of the study subjects.

- **The raw data used in this study were provided by the Fujian Geological Survey Institute and are subject to strict data sharing and confidentiality requirements. Consequently, the relevant raw data and derived data are not published in this repository.

- **Requests for data access should be directed to the author team and are subject to compliance with the corresponding data usage agreements and ethical guidelines.

- **To facilitate reproducibility and code verification, we have provided a synthetic dataset (mock data) in the repository. Users can use this data to execute the pipeline and validate the implementation.

---

## Repository layout

Suggested structure (you may rename files as needed):

```
.
├─ train_mhgan.py                 # main training script (PyTorch Geometric)
├─ README.md
├─ data/
│  └─ processed_nodes.xlsx        # NOT included (user-provided)
└─ outputs/
   └─ predictions_all_nodes.csv   # generated
```

---

## Requirements

-numpy==1.26.3
-pandas==2.2.3
-scikit-learn==1.6.1

-torch==2.5.0+cu124
-torchvision==0.20.0+cu124
-torchaudio==2.5.0+cu124

-torch-geometric==2.6.1
-pyg-lib==0.4.0+pt25cu124
-torch-scatter==2.1.2+pt25cu124
-torch-sparse==0.6.18+pt25cu124
-torch-cluster==1.6.3+pt25cu124
-torch-spline-conv==1.2.2+pt25cu124

openpyxl==3.1.5


---

## Input data format

The code expects a **preprocessed** node table (Excel or CSV), where features are already normalized (e.g., to `[0, 1]`) and a type label is provided for positive nodes.

Required columns:

| Column | Type | Meaning |
|---|---:|---|
| `Id` | int / str | unique slope-unit identifier |
| `label` | int | `1` = landslide, `0` = non-landslide, `-1` = unlabeled |
| `ls_type` | int | cluster/type id for positives (typically `1..5`); others can be `0` |
| 13 feature columns | float | normalized factors used as node features |

Default 13 factors expected by the code:

- `NDVI`, `Disroad`, `Driver`, `Landuse`, `TWI`, `SPI`, `Relief`, `Curvature`,
  `aspect`, `Slope`, `Lil`, `Elevation`, `rainfall`

If your columns differ, update `FEATURE_COLS` in `train_mhgan.py`.

---

## Graph construction (edge semantics)

The implementation uses five edge semantics:

- `E_LL (0)`: landslide–landslide similarity edges (within each `ls_type`)
- `E_NN (1)`: non-landslide–non-landslide similarity edges
- `E_LN (2)`: landslide–non-landslide contrast edges (bidirectional)
- `E_UL (3)`: dynamic edges (pseudo-positive unlabeled ↔ training anchors)
- `E_UN (4)`: dynamic edges (pseudo-negative unlabeled ↔ training anchors)

Static edges are constructed once from the input table. Dynamic edges are refreshed periodically during training.

---

## Model

`NodeEdgeFusionGAT` implements two attention branches:

- **Node-attention** branch (without edge features)
- **Edge-semantic attention** branch (with edge-type embeddings)

The two branch outputs are **summed** and passed to an **MLP classifier**.

---

## Training and evaluation

- The code creates train/validation/test masks and reports metrics on each split.
- `edge_type` is checked to ensure it lies in `[0, 4]` before training starts.

---

## How to run

1) Place your processed file at `data/processed_nodes.xlsx` (or use `--data_path`).

2) Run:

```bash
python train_mhgan.py --data_path data/processed_nodes.xlsx
```

Common options:

```bash
python train_mhgan.py   --data_path data/processed_nodes.xlsx   --seed 2   --epochs 130   --lr 0.005   --weight_decay 5e-8   --eta_intra 7 --eta_nn 7 --eta_ln 7   --update_every 5 --thr 0.8 --b 7 --c 7
```

---

## Outputs

- `outputs/predictions_all_nodes.csv` containing:
  - `Id`: node identifier
  - `prob`: predicted probability for all nodes

---

## Reproducibility

- Random seeds are fixed via `set_seed(seed)`.
- Results are deterministic given the same seed, data, and hardware/software stack.

---

## License

Choose a license before releasing publicly (e.g., MIT, Apache-2.0). If uncertain, keep the repository private until you decide.

---

## Citation

If you use this code in academic work, please cite the associated manuscript (once available).
