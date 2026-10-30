# TODO – Mini Challenge 2026

**Deadline: 5 November 2026.** Notebook: `hotels_LA.ipynb`. Background and decisions: `CONTEXT.md`.

Rules: `git pull` before working, one person at a time on the notebook, tick items here and push when done.

## Must do

- [ ] **Review and personalise the 21 text cells** (drafts already in the notebook). Reword in our own words and check that we can explain every point.
  Cell numbers (counted from the top, 63 cells in total):

  | Priority | Cells | Content |
  |---|---|---|
  | 1 | **63** | Conclusions (most important) |
  | 1 | 9, 13, 16, 19, 22 | EDA observations (city choice, time plot, calendar, correlations, ACF) |
  | 1 | 27, 30 | Transformations, stationarity |
  | 1 | 38, 41, 46, 50 | Baselines, fitted models, model selection |
  | 1 | 53, 57, 62 | Residuals, outliers, forecasts vs actual |
  | 2 | 1, 7, 31, 42, 44, 54 | Intro and method explanations |

  Suggested split: A = cells 1-22, B = cells 27-46, C = cells 50-63.
- [ ] After any code change, **check that the numbers quoted in the text** still match the outputs.
- [ ] Final `Restart & Run All` and push before submission.
- [ ] Submit the notebook.

## Exam preparation

Questions the professor listed in the course; make sure everyone can answer them from the notebook:
- [ ] Does Box-Cox induce stationarity? (no, see 3.2)
- [ ] Why do we care about stationarity? (3.2)
- [ ] Why do we care about outliers? Remove or impute? (6.3)
- [ ] Why Los Angeles, and why not Boston even though it scores better on variance ratio / spectral entropy? (2.1)
- [ ] Why is 2020 a stress test and not part of model selection? (4.1)
- [ ] Why does outlier cleaning not improve the model? (6.3)

## Optional (only if time is left)

- [ ] Skill score vs the two seasonal naive baselines, per horizon
- [ ] Empirical coverage of the 80% / 95% prediction intervals on the 2019 CV
- [ ] Diebold-Mariano test: ARIMA + lag364 vs yearly seasonal naive
- [ ] Combination of ETS and ARIMA + lag364 (robustness after breaks)
- [ ] Dummies for moving holidays (Thanksgiving, ...)
- [ ] Lagged `Demand` as exogenous regressor (probably little gain)

## Done

- [x] Dataset and city choice from the EDA (all 8 cities, forecastability indicators)
- [x] Full pipeline: EDA, transformations, stationarity tests, rolling CV, baselines, ETS / SARIMA / Fourier / lag364
- [x] Course guidelines: Box-Cox/stationarity text, Hampel outlier treatment, forecasts vs actual and error plot
- [x] Draft rewrite of all text cells, MAE/nMAE first
- [x] Rerun on every PC, full read-through
