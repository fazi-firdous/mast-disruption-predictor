# CLAUDE.md — Project rules for the MAST Disruption Predictor

Read this file and `docs/SPEC.md` at the start of every session. `SPEC.md` is the source of truth for *what* to build. This file is the source of truth for *how* to work.

## Who this is for
The owner is a Class 11 student in India (JEE / physics olympiad level). This started as a university-application project, but **the owner's goal is a long-term career in fusion research, and the project is to be treated as research-grade (PhD level or beyond), not a school project.** Aim for methods and results that would stand up in a real fusion / ML venue, and push past what published disruption predictors do (see `docs/ROADMAP.md`, which is the agreed long-term plan; stick to it). He is new to Python, studies the code daily, and explains it back before moving on. Two consequences:

1. **Readability beats cleverness, every time.** Prefer a plain loop over a dense one-liner if the loop is easier to follow. No metaprogramming, no decorators beyond `@dataclass`, no deep class hierarchies, no abstract base classes. A function should do one thing and fit on one screen (aim for under ~40 lines).
2. **Every decision must be explainable.** When you choose a parameter (window length, threshold, horizon), put it in `src/config.py` with a one-line comment saying why. When you make a judgement call, log it in `NOTES.md`.

## Hard rules (never break these)
- **No data leakage.** A prediction made at time `t` may only use samples with `time <= t`. No whole-shot statistics, no future samples, no normalisation fitted on validation/test data. Section 6 of the spec explains this; the causality tests in Section 12 must pass.
- **Split by shot, never by window.** Windows from one shot must all land in the same split.
- **Never fabricate results.** Every number in the README, notebook, or results files must come from code that actually ran. If something could not be run (network blocked, too slow), say so plainly and leave a placeholder like `TBD — not yet run`.
- **Never invent data details.** If you need a variable, group, unit, or sign convention not listed as *verified* in SPEC Section 3, inspect the real data first and record what you found in `NOTES.md`. If you can't inspect it, stop and say so.
- **Never commit data.** `data/`, caches, and large artefacts go in `.gitignore`. Only code, small configs, docs, and small result files (JSON, PNG under ~500 KB) are committed.
- **Seed everything.** `RANDOM_SEED = 42` from config, used for splits, sampling, and models.

## Comment style
Comments read like a human developer's notes to themselves: few, dry, and only where they earn their place. Explain the *why*, not the obvious *what*.

Good:
```python
# zarr<3 cannot read the re-ingested v3 stores, so pin >=3
# abs() because MAST's ip sign depends on field direction
```
Bad:
```python
import numpy as np  # import numpy
x = x + 1  # add one to x
```
No comments on imports. No banner comments. No emoji. Docstrings: one or two plain sentences on public functions, saying what goes in and what comes out.

## Workflow
- Work **one phase at a time** (see the phase prompts). Don't start the next phase until the current one's acceptance checks pass.
- At the end of each phase: run `pytest`, run the phase's script on a small subset, commit with a clear message (`phase 2: signal-based disruption labels`), and update `CODE_MAP.md`.
- When a phase says **STOP FOR REVIEW**, finish, summarise what you found in plain language, and wait. Don't continue on your own.
- If the data surprises you (odd units, sign flips, missing groups, weird time bases), don't silently patch it. Write it in `NOTES.md` with the shot IDs involved.
- Keep the dependency list small: `numpy`, `pandas`, `pyarrow`, `xarray`, `zarr>=3` (the stores are Zarr v3), `fsspec`, `aiohttp`, `requests`, `scikit-learn`, `matplotlib`, `pytest`, `tqdm`. Ask before adding anything else (LightGBM is allowed in Phase 6 if HistGradientBoosting underperforms).

## Environment notes
- Python 3.10+.
- Data is streamed over HTTPS from public servers (`mastapp.site`, `s3.echo.stfc.ac.uk`). If the cloud session can't reach them, say so immediately. Everything must still be runnable from the Colab notebook.
- The owner works on an iPad. Every workflow must be runnable end-to-end from `notebooks/colab_pipeline.ipynb`.

## Docs you maintain
- `README.md` — what, why, how to run, results, limitations, credits.
- `CODE_MAP.md` — a plain-English tour of the repo, written for a beginner: each file, each public function, what it takes, what it returns, why it exists, and the order to read them in. Update it every phase.
- `NOTES.md` — dated log of data surprises, decisions, and anything you weren't sure about.
