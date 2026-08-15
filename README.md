<div align="center">

```
 ██████╗ █████╗ ██╗   ██╗███████╗ █████╗ ██╗     
██╔════╝██╔══██╗██║   ██║██╔════╝██╔══██╗██║     
██║     ███████║██║   ██║███████╗███████║██║     
██║     ██╔══██║██║   ██║╚════██║██╔══██║██║     
╚██████╗██║  ██║╚██████╔╝███████║██║  ██║███████╗
 ╚═════╝╚═╝  ╚═╝ ╚═════╝ ╚══════╝╚═╝  ╚═╝╚══════╝
```

### 🔬 Causal Inference: A/B Testing & Difference-in-Differences

> Proving whether a change *actually caused* an outcome — not just happened alongside it. Two real studies, two causal methods, with proper statistical testing and honest limitations.

<br/>

![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-150458?style=for-the-badge&logo=pandas&logoColor=white)
![Jupyter](https://img.shields.io/badge/Jupyter-F37626?style=for-the-badge&logo=jupyter&logoColor=white)
![SciPy](https://img.shields.io/badge/SciPy-8CAAE6?style=for-the-badge&logo=scipy&logoColor=white)
![Statsmodels](https://img.shields.io/badge/Statsmodels-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Matplotlib](https://img.shields.io/badge/Matplotlib-11557C?style=for-the-badge&logo=python&logoColor=white)

</div>

---

## 📌 About

Most beginner analyses stop at "these two things moved together." This repo goes further — proving whether a change genuinely **caused** an outcome, using proper statistical methods, on two structurally different kinds of real data:

1. **A randomized A/B test**, where causation is easier to argue because the two groups were split by chance.
2. **A real-world natural experiment**, where nothing was randomized, requiring a more careful method (difference-in-differences) to isolate cause from coincidence.

Each study is fully self-contained — its own notebook, its own dataset, and its own `PROJECT_JOURNEY.md` documenting the actual process: the decisions made, the mistakes caught, and the honest limitations of the result.

---

## 🗺️ Repository Structure

```
causal-inference-ab-test-diff-in-diff/
│
├── 📁 ab-test-cookie-cats/
│   ├── 📄 PROJECT_JOURNEY.md              # Full process write-up: EDA, outlier fix, significance testing
│   ├── 📓 ab-test-causal-analysis.ipynb   # Chi-square + t-test analysis on Cookie Cats A/B test
│   └── 🗃️ cookie_cats.csv.zip             # 90,189 users, gate_30 vs gate_40 paywall placement
│
├── 📁 diff-in-diff-minimum-wage/
│   ├── 📄 PROJECT_JOURNEY.md              # Full process write-up: dataset validation, DiD regression
│   ├── 📓 card_krueger_diff_in_diff_analysis.ipynb  # Diff-in-diff replication of Card & Krueger (1994)
│   └── 🗃️ njmin3.csv                      # 410 NJ/PA fast-food restaurants, pre/post minimum wage hike
│
└── 📄 README.md
```

---

## 🧪 The Two Studies

**📱 Cookie Cats — Randomized A/B Test**
Does moving a mobile game's paywall gate from level 30 to level 40 affect player retention?
- Chi-square test on 1-day and 7-day retention
- T-test on engagement, with a real outlier (one user, 49,854 game rounds) identified and removed
- **Result:** 7-day retention dropped significantly (19.02% → 18.20%, p = 0.0016); 1-day retention did not

📂 See [`ab-test-cookie-cats/`](./ab-test-cookie-cats/) for the full notebook and journey doc.

**💵 Card & Krueger (1994) — Natural Experiment**
Did New Jersey's 1992 minimum wage increase reduce fast-food employment, relative to neighboring Pennsylvania (which didn't raise its wage)?
- Manual diff-in-diff calculation, cross-checked against the real published 1994 numbers
- OLS regression with heteroscedasticity-robust standard errors
- **Result:** No evidence of reduced employment — effect was +2.75 FTE employees per restaurant, replicating the original counterintuitive finding (though with a more fragile p-value than originally reported)

📂 See [`diff-in-diff-minimum-wage/`](./diff-in-diff-minimum-wage/) for the full notebook and journey doc.

---

## 🛠️ Tech Stack

| Tool | Purpose |
|------|---------|
| 🐍 **Python** | Core language |
| 🐼 **Pandas** | Data wrangling and analysis |
| 📓 **Jupyter Notebook** | Step-by-step workflow |
| 📊 **SciPy (scipy.stats)** | Chi-square and t-test significance testing |
| 📈 **Statsmodels** | Difference-in-differences OLS regression |
| 🎨 **Matplotlib** | Trend and comparison visualizations |

---

## 👨‍💻 Author

**Amit Kumar**

[![GitHub](https://img.shields.io/badge/GitHub-amit--0333-181717?style=flat&logo=github)](https://github.com/amit-0333)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Amit%20Kumar-0077B5?style=flat&logo=linkedin)](https://www.linkedin.com/in/amit-kumar-a62a3640a/)

---

<div align="center">

> 📝 *Built as part of my Data Science and Python learning journey.*

⭐ **Star this repo if you found it useful!**

</div>
