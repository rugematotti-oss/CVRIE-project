# CVRIE — Supervised Notebook: What I Did

**Task:** classify CT scans as Contrast-enhanced or No-Contrast, as the supervised half of the CVRIE V1 medical AI project.
**Dataset:** CT Medical Images (Kaggle) — 100 labeled scans, `tiff_images/` + `overview.csv`, balanced 50/50 Contrast vs No-Contrast.

## 1. Loading the data

Standard image readers can't open these files — Pillow fails with `UnidentifiedImageError`. The reason: these TIFFs store real, signed Hounsfield Unit (HU) values as floats (roughly -1024 to +894), not standard 0–255 pixel data. Switched to `tifffile`, which handles scientific TIFF variants correctly. Loaded all 100 images by joining `overview.csv`'s `tiff_name` column to the actual files — 100/100 loaded, 0 missing.

Sanity check: average HU is measurably higher for Contrast scans than No-Contrast scans, confirming the label reflects a real physical effect (contrast agent raises tissue radiodensity) rather than something arbitrary.

## 2. Feature engineering

**First attempt (rejected):** 22 features — 7 summary statistics plus a 15-bin pixel-value histogram — evaluated with a scaler fit on the whole dataset before cross-validation. Rejected for two reasons: too many features for only 100 samples (real overfitting risk), and the scaler fit leaked information from the test folds into training.

**Final approach:** 6 domain-grounded features per image — mean HU, standard deviation of HU, and the fraction of pixels falling into four clinically meaningful HU ranges (air, fat, soft tissue, bone). Every one of these six numbers is something a radiologist could interpret directly, and the feature-to-sample ratio stays safe for a 100-image dataset.

## 3. Model benchmarking

All models wrapped in a `Pipeline` (scaler + model) so `cross_val_score` refits the scaler inside each fold — no leakage.

| Model | Accuracy (5-fold CV) | ROC-AUC |
|---|---|---|
| Dummy (majority class) | 50.00% | — |
| Logistic Regression | 80.00% (± 13.78%) | 0.880 |
| Linear SVM | 78.00% (± 10.30%) | 0.880 |

LR and SVM are statistically tied (overlapping confidence ranges), so the choice wasn't based on the small accuracy gap. **Logistic Regression was picked for interpretability** — it gives a probability output and per-feature coefficients that can be explained in plain clinical terms, unlike an SVM's boundary.

## 4. Validation — confusion matrix and error analysis

Confusion matrix (LR, 5-fold CV, all 100 samples): 41 true negatives, 9 false positives, 11 false negatives, 39 true positives.

False negatives (missing a real Contrast scan) were treated as the more clinically costly error, since a missed contrast flag risks misreading vascular/tissue detail.

Checked whether errors were random or explainable:
- Both false-negative and false-positive cases lean, on average, toward the *predicted* class's feature values rather than their true class — meaning mistakes cluster on genuinely borderline scans, not random failures.
- Looking at the model's actual predicted probabilities: most false positives sat close to 50% (honest uncertainty), but a few false negatives were confidently wrong (as low as 9–13% predicted probability) — likely real patient-to-patient variation in how contrast shows up, which 6 simple features can't fully capture.

## 5. Data quality check — duplicate/leakage detection

Noticed two false-positive samples with identical feature values to five decimal places — essentially impossible for two unrelated patients by chance. A systematic pairwise-distance scan across all 100 images found **6 exact-duplicate pairs**, almost certainly the same patient filed under different case IDs — a patient-level leakage risk if a duplicate pair split across train/test folds.

Removed one image from each duplicate pair (94 samples left) and re-ran the same pipeline: **79.71% accuracy (± 10.92%), 0.863 ROC-AUC** — barely different from the original 80.00%/0.880. This confirmed the original result was already largely trustworthy, though the leak was real and worth catching and documenting.

## 6. ROC curve

Plotted the ROC curve for Logistic Regression: the curve sits clearly above the random-guessing diagonal, consistent with the 0.880 AUC. At the strictest possible cutoff (zero false alarms allowed), the model still catches about **66%** of real Contrast cases.

## 7. Threshold tuning

The default classification cutoff is 50%. Scanned cutoffs from 30% to 60% to see whether a different cutoff could better balance the two recall rates:

| Cutoff | Contrast recall | No-Contrast recall |
|---|---|---|
| 0.50 (default) | 78% | 82% |
| 0.47 (chosen) | 80% | 80% |

Chose **0.47** as the final threshold since it balances both error types exactly, a small, deliberate improvement over the default. Noted explicitly that this threshold was selected by testing against the same 94 images used throughout, so it should be treated as a reasonable starting point rather than a fixed, universally optimal value.

## 8. Loss function

Logistic Regression is trained using **log-loss**, which grades how trustworthy each probability prediction was, not just whether the final guess was right or wrong. A wrong prediction made with high confidence is punished far more heavily than a wrong prediction made with low confidence — this is exactly why the model's mistakes cluster on genuinely ambiguous, borderline scans rather than confident, clear-cut failures.

## 9. Limitations & honest conclusion

- **Sample size:** 94–100 scans is small. Each cross-validation fold evaluates on only ~18–19 images, so a single misclassification swings that fold's accuracy by roughly 5%. Can't be confident this model would generalize to new patients, different body types, or scans from other hospitals without testing on a larger, more varied dataset.
- **Error pattern:** the model usually gets confused only on genuinely borderline, ambiguous scans, but occasionally becomes confidently wrong on non-typical contrast cases — where the scan's physical numbers happen to resemble the opposite class.
- **Duplicate/leakage check:** found and fixed 6 duplicate image pairs; removing them barely changed the result (80.00% → 79.71%), showing the original number was largely trustworthy but the check was still necessary.
- **Bottom line:** the 80% accuracy shows the 6 engineered features capture a real, physical signal for detecting CT contrast — one that clearly beats random guessing and wasn't inflated by data leakage. It does **not** prove the model is ready for real clinical use: 94 scans from a single source isn't enough to guarantee it works on different patients or hospitals, and the model still gets confidently wrong on some borderline cases.

## Tooling note

`tifffile` was used for image loading (outside the five approved tools — Pandas, Numpy, Matplotlib, Scikit-learn, JupyterLab) because Pillow cannot parse this dataset's signed 16-bit HU TIFF format. Confirmed with the instructor that this is acceptable as long as it's justified in the notebook, which it is.
