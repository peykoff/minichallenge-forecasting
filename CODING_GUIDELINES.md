# Coding guidelines for notebooks

Common rules for our forecasting / data-analysis notebooks. Follow them in every new project, and give this file to any person or AI assistant that writes code or text for us.

## Notebook structure

1. **One idea per cell.** Each code cell does one thing (loading, one plot, one model fit) and is followed by a markdown cell that explains it.
2. **Fixed pattern per section:** markdown (what and why) → code → markdown with **Observations**. Explanations go in markdown, not in code comments.
3. **Numbered sections** (1, 2.1, 2.2, ...) following the course workflow: data → EDA → transformations → CV and baselines → models → results → conclusions.
4. **Decisions are made inside the notebook**, not a priori: choices (e.g. which series to model) must come out of the analysis and be visible in a cell.
5. **Keep model selection and final test separate** (e.g. CV on one period, stress test on another).
6. **First cell:** goal, list of steps, runtime.

## Code style

7. **Blank lines** around every `for` loop, `if` block and function definition, and between logical steps inside a cell (setup / computation / plot).
8. **One statement per line.** No `;` to chain statements. A trailing `;` is only allowed on the last line of a plot cell, to hide the printed object.
9. **No one-line if/else** (`a if cond else b`): use a normal `if` / `else` block.
10. **Comprehensions only when simple** (`[f(x) for x in xs]`). Nested comprehensions, dict comprehensions that build tables, or comprehensions inside a function call become an explicit loop that fills a variable:
    ```python
    mae_h = pd.DataFrame()

    for name, (yt, yh) in res.items():
        mae_h[name] = np.abs(yt - yh).mean()
    ```
11. **No named lambdas** (`mape = lambda ...`): use `def`. Lambdas are fine inline or in a small dict of one-line rules (e.g. baseline forecasters).
12. **Name things instead of indexing them:** `y_true, y_hat = cv[best]`, not `cv[best][0][1]`.
13. **No alignment with extra spaces:** `'ETS': (...)`, not `'ETS':        (...)`.
14. **Use only the tools seen in the course** (pandas, numpy, matplotlib / seaborn, statsmodels, statsforecast). Avoid library internals and advanced patterns (closures, pandas Styler, tricks like `np.resize`) unless we can explain them.
15. **Short and fast:** the whole notebook must run in a few minutes. Rerun everything (`Restart & Run All`) before every push.

## Comments

16. **No docstrings** (`'''...'''`) in notebook functions: the markdown explains them.
17. **Comment only what is not obvious from the code**, in a few words, e.g. `# NaN in 2013, so the estimation starts in 2014`.

## Text and characters

18. **Plain ASCII in code, printed outputs and plot labels.** No Greek letters (Δ, λ, ρ), no `≈ ≤ ≥ → × ·`, no typographic minus `−`, no en / em dashes. Write `diff7`, `lambda`, `correlation`, `about`, `p < 0.001`, `-`.
19. **Math only in LaTeX** (`$m=7$`, `$\Delta y_t$`), never as loose Unicode in sentences.
20. **Never use `$` for currency in markdown**: Jupyter renders the text between two `$` as a formula. Write `USD`.
21. **No unnecessary quotes or bold.** Bold only for the key result of an observation.
22. **Simple sentences, written by us, with the exact numbers from the outputs.** After any code change, check that the numbers in the text still match.
23. **Course metrics first** (MAE, nMAE); extra metrics (e.g. MAPE) second.

## Teamwork

24. **`git pull` before starting.** One person at a time on the main notebook; experiments in a copy (`notebook_<name>.ipynb`).
25. **Close Jupyter without saving** if the file was changed on disk while it was open.
26. **Update `CONTEXT.md` and `TODO.md`** at the end of each session and commit them together with the notebook.
