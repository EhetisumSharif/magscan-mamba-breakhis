# MagScan-Mamba on BreakHis: reliability under magnification shift

Notebooks and figures for the article

> **Reliability under magnification shift in breast histopathology: leave-one-magnification-out evaluation of four encoder families and MagScan-Mamba on BreakHis**
> Ehetisum Sharif, Nishat Tasnim, Anay Kumer Ghosh, Ferdaus Anam Jibon
> Department of Computer Science and Engineering, IUBAT, Dhaka, Bangladesh

Each of the four BreakHis magnifications (40×, 100×, 200×, 400×) is held out in turn. ConvNeXt-T, Swin-T, MedMamba
and a frozen Phikon linear probe are trained on the other three with patient-disjoint splits and three seeds
(13, 42, 2025), and every model is judged on accuracy, calibration, selective risk and shift detection.
MagScan-Mamba is a 1.62 M-parameter head that reads the frozen Phikon patch tokens with a four-direction Mamba scan.

## Repository layout

```
notebooks/
  01_binary_lomo/        one training session per held-out magnification
  02_ablation_and_cv/    MedMamba without SS2D, and 5-fold patient-level cross-validation
  03_magscan_mamba/      pilot 1, pilot 2 and the full MagScan-Mamba study
  04_subtype/            the two-stage eight-subtype pipeline: nine sessions + MagScan-Mamba
  05_analysis/           the analysis notebook that produces every figure and table
figures/                 the 23 figures of the article, as PNG (as printed) and PDF (vector, where available)
requirements.txt         the library versions the runs used
```

These are the notebooks that produced the article's results; their outputs have been cleared.

## Data

The images are **not** redistributed here. Both cohorts are public:

| Cohort | Use | Source used in the runs |
|---|---|---|
| BreakHis (Spanhol et al., 2016): 7,909 tiles of 700 × 460 px, 82 patient identifiers (24 benign, 58 malignant), 40×/100×/200×/400× | training, validation, test | Kaggle mirror `ambarish/breakhis`; original: https://web.inf.ufpr.br/vri/databases/breast-cancer-histopathological-database-breakhis/ |
| BACH, ICIAR 2018 (Aresta et al., 2019): 400 images of 2,048 × 1,536 px | external probe only, never trained on | Kaggle mirror `truthisneverlinear/bach-breast-cancer-histology-images` |

## Compute environment

All runs used Kaggle notebooks with a single NVIDIA Tesla T4 (16 GB), Python 3.12.13, PyTorch 2.10.0+cu128,
CUDA 12.8 and cuDNN 9.10.2, in float32 without mixed precision. `requirements.txt` lists the libraries that can
change a number.

## How to reproduce the results

Attach BreakHis (and BACH where noted) to each Kaggle notebook and select **Accelerator → GPU T4**. Each notebook
resumes from an attached earlier version of itself, so a session that hits the 12-hour limit can be continued.

1. **`01_binary_lomo/`** – four sessions, identical except `HELD_OUT_OVERRIDE` (40, 100, 200, 400). Each trains
   ConvNeXt-T and Swin-T (ImageNet init.), MedMamba (random init., public code pinned at commit
   `104beb3d641d67cef6f09b94ac299083d91bc417`) and the Phikon linear probe for three seeds, scores BACH, and writes
   `main_ho<mag>/`. About 5.6 h per session. The DSA-Mamba and MambaVision paths in these notebooks were inactive
   in the reported runs (`TRY_DSAMAMBA_RESTORE = False`; MambaVision was not in the installed `timm`).
2. **`02_ablation_and_cv/`** – `PARTC_SPLIT` 1: MedMamba without SS2D at held-out 400×, three seeds;
   2: 5-fold patient-level CV (seed 13) of the Phikon probe at all four magnifications and MedMamba fold 0 at 400×;
   3 and 4: MedMamba folds 1–2 and 3–4.
3. **`03_magscan_mamba/`** – `magscan_pilot1` (held-out 400× and 40×) → `magscan_pilot2` (attach pilot 1; adds the
   single-view controls) → `magscan_full_binary` (attach pilot 2, the four binary outputs, the Part C CV output and
   BACH; all variants at four magnifications plus 5-fold CV). Restored runs are accepted only under the same
   configuration hash, and every split is checked against the benchmark's stored test labels.
4. **`04_subtype/`** – per held-out magnification, session A trains ConvNeXt-T, Swin-T and the Phikon probe and
   session B trains MedMamba; session C is MedMamba without SS2D at 400×; `subtype_magscan_all_mags` adds
   MagScan-Mamba at all four magnifications. The split is image-level inside the training magnifications
   (`SPLIT_MODE = 'image'`), so these results measure robustness to magnification, not generalisation to new patients.
5. **`05_analysis/`** – attach every output above and BACH, then run all cells. It writes all figures and tables to
   `analysis_outputs/`.

## Figures

| Article | File | Produced by |
|---|---|---|
| Fig. 1 | `Figure_01_workflow` | authors' drawing |
| Fig. 2 | `Figure_02_magscan_architecture` | authors' drawing |
| Fig. 3–23 | `Figure_03_…` to `Figure_23_…` | `notebooks/05_analysis/analysis_notebook_v3.ipynb` |

## Patient identifiers

The splits are disjoint at the level of the 82 BreakHis patient identifiers. Ten numeric accession prefixes recur
under more than one identifier, so a test identifier can share a prefix, and possibly a patient, with a training one;
the article's Limitations section reports how much of each test set this affects.

## Stored outputs

The stored predictions, run logs, model checkpoints and cached features are not part of this repository because of
their size; they are available from the corresponding author on reasonable request.

## Citation

Please cite the article (details will be added on publication). See `CITATION.cff`.

## License

Code: MIT (see `LICENSE`). BreakHis, BACH, Phikon and MedMamba remain under their own terms.
