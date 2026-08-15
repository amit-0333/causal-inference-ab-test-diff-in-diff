# Project Journey: Cookie Cats A/B Test — Does Gate Placement Affect Retention?

## Notebook Intro Cell

The block below is written to be pasted as the very first markdown cell in the notebook, before any code — so anyone opening it cold understands the project in under a minute.

> ### Does Moving a Paywall Gate Actually Hurt Player Retention?
>
> **Goal:** In the mobile game Cookie Cats, a "gate" temporarily locks players unless they wait or pay. Players were randomly split into two groups — one saw the gate at level 30, the other at level 40. This project tests whether that placement change actually affected retention, using proper statistical significance testing rather than just comparing raw numbers.
>
> **Dataset:** Cookie Cats A/B test data — 90,189 users, randomly assigned to `gate_30` or `gate_40`, with 1-day and 7-day retention recorded.
>
> **Method:** Chi-square tests for retention (a binary yes/no outcome), t-test for engagement (game rounds played), with an outlier check before trusting the engagement comparison.
>
> **Headline result:** 7-day retention dropped significantly with the later gate (19.02% → 18.20%, p = 0.0016), but 1-day retention showed no significant difference (p = 0.0755). Engagement showed no real difference once a single extreme outlier was removed. See `PROJECT_JOURNEY.md` for the full story, including the outlier that nearly skewed the engagement comparison.

## What this project is

A randomized A/B test analysis testing whether moving a mobile game's paywall gate from level 30 to level 40 causally affects player retention. Because this is a true randomized experiment (not a natural experiment), proving causation is more straightforward than in a natural-experiment setting — the two groups should be comparable by design, so any statistically significant difference can be attributed to the gate placement itself.

**Dataset:** Cookie Cats mobile game A/B test (Kaggle) — 90,189 users, `version` (gate_30 / gate_40), `sum_gamerounds`, `retention_1`, `retention_7`.

**Tools:** Python, pandas, scipy.stats (chi-square, t-test), matplotlib.

---

## Phase 1: Loading and Inspecting the Data

**Basic checks before trusting the dataset:**
- Shape: 90,189 rows, 5 columns
- No missing values in any column
- Group sizes: `gate_30` = 44,700 users, `gate_40` = 45,489 users — close to a balanced 50/50 split, confirming the randomization worked as intended

This mattered because an unbalanced or corrupted split would undermine the whole premise of using a chi-square/t-test to compare the groups directly.

---

## Phase 2: First Look — Simple Group Comparison

Before running any statistical test, the two groups were compared on raw averages:

| Group | 1-day retention | 7-day retention |
|---|---|---|
| gate_30 | 44.82% | 19.02% |
| gate_40 | 44.23% | 18.20% |

Both retention metrics were lower in the gate_40 group — but a raw difference alone isn't proof of anything. With 90,000+ users, small gaps can appear from pure random variation. This is exactly the reasoning gap the project set out to close: "looks different" is not the same as "is actually different."

---

## Phase 3: Significance Testing

Ran a chi-square test on each retention outcome, since both are binary (came back / didn't come back):

| Metric | Chi-square statistic | p-value | Statistically significant? |
|---|---|---|---|
| 1-day retention | 3.159 | 0.0755 | No (above the 0.05 threshold) |
| 7-day retention | 9.959 | 0.0016 | Yes |

**Interpretation:** The 1-day retention gap is small enough that it could plausibly be random noise — we can't confidently say the gate placement affected next-day return behavior. The 7-day gap, however, is unlikely to be chance. This is a genuine, real effect of moving the gate.

---

## Phase 4: An Outlier That Nearly Skewed the Engagement Comparison

**What was checked next:** average game rounds played (`sum_gamerounds`) per group, as a secondary engagement metric beyond retention.

**What was found:** the maximum value in the entire dataset was **49,854 rounds** — compared to a dataset median of just **16 rounds**. This single user, sitting in the `gate_30` group, was an extreme outlier by any reasonable standard.

**Why it mattered:** Running a t-test on gamerounds with this outlier included gave a result that could be misleading, since one abnormal data point can distort a group average when using raw means.

**The fix and comparison:**

| | t-statistic | p-value |
|---|---|---|
| With outlier included | 0.891 | 0.3729 |
| With outlier removed | 0.063 | 0.9495 |

After removing the single extreme row and rerunning the test, the apparent difference in engagement essentially disappeared (p went from 0.37 to 0.95). This confirmed there was no meaningful difference in how much players engaged with the game between the two groups — only in whether they returned a week later.

---

## Phase 5: Effect Size — How Big Is the Real Difference?

Statistical significance alone doesn't say whether an effect actually matters in practice. The 7-day retention drop was quantified directly:

- gate_30: 19.02%
- gate_40: 18.20%
- **Absolute difference: 0.82 percentage points**
- **Relative drop: 4.31%**

**Why this step matters:** a p-value of 0.0016 tells you the difference is *real*, but not whether it's *big*. At the scale of millions of daily active users, even a 4.31% relative drop in 7-day retention translates into a meaningful number of lost players — so this is a small-looking number with real business weight behind it.

---

## Phase 6: Conclusion and Honest Limitations

**Conclusion:** Moving the paywall gate from level 30 to level 40 caused a statistically significant drop in 7-day retention, but did not produce a significant change in 1-day retention or in overall engagement. Because this was a genuine randomized A/B test, this conclusion rests on solid causal footing — no control group or diff-in-diff adjustment is needed, since random assignment already balanced the two groups by design.

**Limitations, stated honestly:**
- The effect size (0.82 percentage points) is small in absolute terms — whether it matters commercially depends entirely on the game's scale and business context.
- The data explains *that* retention dropped, not *why* — there's no information here on player sentiment, purchase behavior, or session-level detail that could explain the mechanism.
- One extreme outlier was found and removed for the engagement comparison; this is a standard and defensible step, but it's disclosed explicitly rather than silently applied.
- Results are specific to this game and this exact gate mechanic — they may not generalize to other games or other types of paywalls.

---

## What This Project Demonstrates

- Distinguishing "the numbers look different" from "the difference is statistically real" using an appropriate significance test for a binary outcome
- Catching a single extreme outlier before it silently distorted a comparison, and showing the before/after result rather than hiding the fix
- Reporting both statistical significance and real-world effect size, rather than stopping at a p-value
- Being explicit about what a randomized A/B test does and doesn't require methodologically (no control-group adjustment needed, unlike a natural experiment)
- Writing an honest limitations section that acknowledges what the data can and can't explain
