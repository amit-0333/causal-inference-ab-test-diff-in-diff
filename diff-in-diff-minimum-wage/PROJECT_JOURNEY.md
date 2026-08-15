# Project Journey: Proving Causation, Not Just Correlation

## Notebook Intro Cell

The block below is written to be pasted as the very first markdown cell in either notebook, before any code — so anyone opening it cold understands the project in under a minute.

> ### Causal Analysis: Did It Actually Cause That, or Did It Just Happen Alongside It?
>
> **Goal:** Most beginner analyses stop at "these two things moved together." This project goes further — proving whether a change genuinely *caused* an outcome, using proper statistical methods, on two different kinds of real data.
>
> **Two studies, two methods:**
> 1. **Cookie Cats mobile game A/B test** (90,189 users) — a randomized experiment, tested with chi-square and t-tests.
> 2. **Card & Krueger (1994) minimum wage study** (410 restaurants, NJ vs PA, 1992) — a real-world natural experiment with no randomization, tested with difference-in-differences regression.
>
> **Headline results:** Cookie Cats' gate change caused a statistically significant drop in 7-day retention (19.02% → 18.20%, p = 0.0016) but no significant change in 1-day retention or engagement. The NJ minimum wage hike showed no evidence of reducing employment — if anything, a positive effect (+2.75 FTE employees per restaurant) — replicating the real 1994 finding, though with a p-value (0.125) more fragile than the original paper reported. See `PROJECT_JOURNEY.md` for the full story, including a wrong dataset caught mid-project and an outlier that nearly skewed a result.

## What this project is

Two separate causal analyses, built to demonstrate the same underlying skill through two different lenses: separating "X caused Y" from "X and Y just happened around the same time."

**Study 1 — Cookie Cats:** A randomized A/B test on a mobile game's paywall gate placement (level 30 vs level 40), testing the effect on player retention.

**Study 2 — Card & Krueger:** A natural experiment using New Jersey's 1992 minimum wage increase, with Pennsylvania as a control state, testing the effect on fast-food employment via difference-in-differences.

**Tools:** Python, pandas, scipy.stats (chi-square, t-test), statsmodels (OLS regression), matplotlib.

---

## Phase 1: Choosing the Right Datasets

Before any analysis, the datasets themselves had to be chosen carefully, because the wrong dataset breaks the whole method before it starts.

**For the A/B test:** Needed something randomized, clean, and large enough that a real effect wouldn't get lost in noise. Cookie Cats was chosen over alternatives (a generic marketing A/B dataset) because it was small enough to fully understand every row, well-documented, and had a genuinely binary, real-world outcome (retention).

**For the natural experiment:** This choice mattered more, because diff-in-diff only works if there's a credible control group. Alternatives considered and rejected:
- **COVID mobility data** — more novel, but confounders (multiple regions locking down at different times) made a clean control group hard to defend.
- **Retail/e-commerce pricing datasets** — highest risk, since most Kaggle retail datasets don't actually have a built-in valid control group despite looking like they might.

**Decision:** Card & Krueger's 1992 minimum wage study was chosen — a textbook-respected diff-in-diff case study with a genuinely credible control group (Pennsylvania, which didn't change its minimum wage) built into the original research design.

---

## Phase 2: A Wrong Dataset, Caught Before It Caused Damage

**What went wrong:** The first attempt to source the Card & Krueger data pulled a file called `card.dta` from a well-known GitHub repository (`scunning1975/mixtape`), assuming it was the minimum wage dataset.

**The catch:** Inspecting the file's actual columns (`nearc2`, `nearc4`, `educ`, `IQ`, `KWW`) revealed this was a *completely different* David Card study — his 1995 paper on returns to education using college proximity as an instrument for schooling. Same author, unrelated topic. Using this file would have silently produced a nonsensical analysis.

**The fix:** Searched further and found `njmin3.csv` in a different repository (`AnthonyPuggs/CardKrueger1994Replication`), which contained the correct pre-built diff-in-diff structure: treatment indicator (`nj`), time period (`d`), interaction term (`d_nj`), and the employment outcome (`fte`).

**Verification before trusting it:** Rather than assuming this new file was correct just because the filename matched, the actual numbers were checked against the real published 1994 paper. They matched exactly:

| State | Pre (Feb 1992) | Post (Nov 1992) |
|---|---|---|
| Pennsylvania (control) | 23.33 | 21.17 |
| New Jersey (treatment) | 20.44 | 21.03 |

Only after this exact match was the dataset trusted and used.

---

## Phase 3: Cookie Cats — Exploration and the First Look

**Basic checks:** 90,189 users, no missing values, groups balanced close to 50/50 (44,700 vs 45,489).

**Simple group comparison (before any testing):**

| Group | 1-day retention | 7-day retention |
|---|---|---|
| gate_30 | 44.82% | 19.02% |
| gate_40 | 44.23% | 18.20% |

Both metrics were lower for gate_40, but a raw difference isn't proof — with 90,000 users, small differences can appear even from pure chance.

---

## Phase 4: Cookie Cats — Significance Testing

Ran chi-square tests on both retention outcomes:

| Metric | Chi-square | p-value | Significant? |
|---|---|---|---|
| 1-day retention | 3.16 | 0.0755 | No |
| 7-day retention | 9.96 | 0.0016 | Yes |

**Interpretation:** The gate change did not produce a confident short-term effect, but it did produce a real, statistically significant drop in longer-term (7-day) retention.

---

## Phase 5: An Outlier That Nearly Skewed the Engagement Comparison

**What was found:** Checking `sum_gamerounds` (total rounds played) revealed one single user had played **49,854 rounds**, against a dataset median of just **16**. This one user was in the gate_30 group, quietly inflating that group's average.

**Why it mattered:** Running a t-test on gamerounds *with* this outlier included gave a misleading comparison. Removing the single extreme row and rerunning:

| | t-statistic | p-value |
|---|---|---|
| With outlier | 0.891 | 0.3729 |
| Without outlier | 0.063 | 0.9495 |

Once removed, the apparent difference in engagement essentially vanished — confirming there was no real difference in how much people played, only in whether they came back a week later.

---

## Phase 6: Cookie Cats — Effect Size and Conclusion

Statistical significance alone doesn't say whether an effect actually matters. The 7-day retention drop was quantified directly:

**19.02% → 18.20% = a 0.82 percentage point (4.31% relative) decline, p = 0.0016**

**Conclusion:** Moving the gate to level 40 caused a real, if modest, drop in 7-day retention. Because this was a true randomized test, this conclusion rests on solid causal footing — no control group or diff-in-diff needed, since randomization already balanced the two groups by design.

---

## Phase 7: Card & Krueger — Manual Diff-in-Diff First

Before running any regression, the diff-in-diff logic was worked through by hand, using the validated dataset:

- Pennsylvania change: 23.33 → 21.17 = **−2.17**
- New Jersey change: 20.44 → 21.03 = **+0.59**
- Diff-in-diff = NJ's change − PA's change = 0.59 − (−2.17) = **+2.75**

This matched the historically reported effect from the original 1994 paper almost exactly — a strong signal the dataset and logic were both correct before moving to a formal regression.

---

## Phase 8: Card & Krueger — Regression and a Real Limitation

Ran a proper OLS regression (`fte ~ nj + d + d_nj`) with heteroscedasticity-robust standard errors, rather than relying on the manual calculation alone:

**d_nj coefficient (the causal estimate): +2.75, SE = 1.80, p = 0.1251, 95% CI [−0.77, 6.27]**

**The honest limitation:** The regression confirmed the same +2.75 effect size as the manual calculation and the original paper — but the p-value (0.125) sits above the conventional 0.05 threshold, meaning this specific robust-standard-error approach can't rule out the result being due to chance, even though the original paper reported it as statistically significant using different assumptions. Rather than smoothing over this discrepancy, it was kept in the write-up as a genuine finding: the *direction* of the effect is well-supported, but its *statistical certainty* is more fragile than the famous headline result implies.

**A second limitation, structural rather than statistical:** With only two time points (before and after), it's not possible to verify the "parallel trends" assumption — the idea that NJ and PA would have moved identically without the policy — the way a longer time series would allow. This is a limitation of the original 1994 study design itself, not something fixable after the fact.

---

## Phase 9: A Notebook Formatting Bug, Caught and Fixed

**What went wrong:** When the `.ipynb` files were first generated programmatically, the code that split each cell's content into lines stripped out the newline characters between lines. The result: multi-line code cells appeared in Kaggle as a single unreadable, squished line (e.g., four separate lines of pandas code glued into one).

**The fix:** Jupyter's notebook format expects each entry in a cell's `source` array to retain its trailing `\n` (except the final line). The generation script was corrected to preserve these line breaks, and both notebooks were rebuilt and re-verified before re-uploading.

---

## What This Project Demonstrates

- Verifying a dataset is actually the right one before building on it — not just trusting a filename or a plausible-looking source (the `card.dta` mix-up)
- Cross-checking a new data source against independently known, published numbers before trusting it (the Card & Krueger validation)
- Catching a single outlier that could have produced a misleading statistical conclusion, and showing the before/after comparison rather than hiding the fix
- Applying two different causal inference methods appropriately matched to two different data structures — significance testing for a randomized experiment, difference-in-differences for a natural experiment
- Reporting a p-value that didn't perfectly match a famous historical result, rather than adjusting the narrative to fit the expected answer
- Distinguishing statistical significance from real-world effect size in both studies, instead of stopping at "the p-value is small"
- Debugging and fixing a technical formatting issue in the deliverable itself before considering the project complete
