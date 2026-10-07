# Decisions Log — Stereo Hs Exp V1

## 2026-10-07 — Pre-registration (before any data download or training)
- Task: direct Hs regression from stereo images; no reconstruction in training or inference.
- Records: BS01–07, AA01–03, YS01 (11). LJ01 excluded a priori (Hs 10.03 m extrapolation, stereo-derived GT, unique geometry); used only as OOD demonstration.
- Labels: BS01–07 from wave gauge (median of 6 gauges, 4·std over the image record span); AA, YS from Scientific Data Table 2.
- Sampling: 1000 evenly spaced stereo pairs per record over the full duration (LJ01: 300 for OOD).
- Main split: LORO; record-level median aggregation; n = 11.
- Required baselines: global mean (B0), station mean (B1).
- Training: fixed 20 epochs, no validation early stopping, 3 seeds.

## 2026-10-07 — S1 manifest findings
- Server pair counts match Table 1 within ±1 for all 12 records.
- BS01 lasts 12.03 min (Table 1 lists 7 min; Table 1 value is wrong).
- BS04: one cam01 image without a cam02 partner; excluded automatically.
- Frame spacing differs by record (0.60 s YS01, 0.72 s BS01, 1.80 s most, 3.60 s AA01). Labels are record-level, so this does not affect labels.
- Columns named cam01/cam02, not left/right. Left/right to be verified in S4 from ext_T (sign of T_x).

## 2026-10-07 — S2 download
- 22,600 images (100.37 GB), 88 config files, 7 wave gauge files. Verified with 02b_verify_download.py: no missing, empty, partial or unreadable files.

## 2026-10-07 — S3 wave gauge labels
- Gauge sampling: 10 Hz (2011), 20 Hz (2013); rate taken from timestamps.
- Time alignment: gauge start within ~0.1 s of image start for all BS records.
- BS03 gauge covers only 21.16 of 29.96 image minutes; label uses the overlap.
- Std and spectral (0.05–2.5 Hz) Hs agree within ~1 % for all records.
- 2011 labels agree with Table 2 within ±10 %.
- 2013 gauges show gain differences between gauges (ratio 0.85–1.13 vs 0.96–1.05 in 2011); 2013 labels carry ~±10 % uncertainty. Median over gauges reduces the effect.

## 2026-10-07 — BS06 label conflict (before any model training)
- Gauge Hs 0.958 m vs Table 2 Hs 0.41 m (+134 %).
- Gauge evidence: 6 gauges coherent (r >= 0.96), clean spectrum, stationary 5-min Hs (0.85–1.01 m), low-frequency energy 1.5 %.
- Rule, fixed before the audit: compute Hs from the dataset's own nc surfaces (audit only, never used for training).
  * If nc agrees with the gauge within 30 % -> keep the gauge label for BS06.
  * If nc agrees with Table 2 instead -> exclude BS06 from the main analysis.
- In every case, report main results with and without BS06 as a sensitivity analysis.
- Audit set: BS05 and BS07 (controls, gauge vs Table 2 within 15 %) and BS06 (conflict). Full nc files, point-temporal Hs at 5 central grid points.
