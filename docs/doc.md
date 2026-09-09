# Household Electricity Usage Forecasting — Mentor's Project Guide

*A scoping and decision document for a 4-person, 5th-semester CSE mini-project (Sept–Dec 2026)*

---

## 1. What the Project Is Actually Asking You to Predict

Strip away the buzzwords and the brief boils down to one sentence: **given a household's past electricity usage (plus optional context like time, weather, occupancy), predict some future usage value.** That's it. Everything else — LSTM, occupancy sensors, peak-demand dashboards — is optional decoration around that core regression/forecasting task.

You have three realistic choices of *prediction target*, differing mainly in **horizon** (how far ahead) and **granularity** (how fine-grained):

| Target | What it means | Pros for a mini-project | Cons for a mini-project |
|---|---|---|---|
| **Next-hour consumption** | Predict kWh (or W) for the next 1-hour block from recent history | Plenty of short-term signal (recent lag values correlate strongly); easy to demo "live-ish" prediction; large sample count from a short dataset | Noisier target (short-term spikes from single appliances); harder to explain misses in the report; needs hourly-resolution data |
| **Next-day (24h-ahead) total or hourly profile** | Predict tomorrow's total kWh, or tomorrow's 24 hourly values | Aligns naturally with "help users understand peak-demand periods"; day-level patterns (weekday/weekend, season) are strong and easy to explain; good middle ground | Slightly fewer effective training rows if you aggregate to daily; predicting a full 24-value profile is more work than a single number |
| **Next-week / multi-day** | Predict aggregate usage over the coming week | Smooths out noise, easiest to get a "good-looking" R² | Least actionable for "peak-demand" framing; weakest fit to the official description, which explicitly mentions time-of-day/occupancy — those signals matter most at short horizons |

**Our recommendation: predict next-hour consumption for the primary model, and layer a next-day aggregate/profile view on top for the "peak demand" analysis and dashboard.** This gives you two horizons to talk about in the report (a common ask from evaluators) without doubling your workload — the next-day view can literally be a rolled-up version of your hourly predictions, or a second, simpler model trained on daily-aggregated data. Don't try to build three separate horizon models; pick one primary + one secondary.

---

## 2. Dataset Requirements

### What you actually need in a dataset

For the project brief to be honestly satisfiable, your dataset needs, at minimum:
- **A timestamped electricity consumption series** (kWh or kW) at hourly or finer resolution, for at least several months (ideally 1+ year, to capture seasonality).
- **Timestamps you can decompose** into hour/day/week/month/season features (this is basically free — any dataset with a proper datetime column gives you this).

"Nice to have," in decreasing order of how much they'll actually move your results:
- **Weather variables** (temperature especially — it's the single strongest external driver of household electricity use, via heating/cooling). Humidity, wind speed are secondary.
- **Occupancy information** — genuinely useful in theory, but **real public datasets almost never have direct occupancy sensors.** What you'll actually find are *proxies*: household metadata (number of occupants, dwelling type, "ACORN" socioeconomic classification), or you can *derive* a rough occupancy proxy from the consumption pattern itself (e.g., a household is "probably occupied" when consumption is above its own baseline). Don't plan around finding true occupancy labels — plan around proxies.
- **Device/appliance-level data (sub-metering)** — useful for feature engineering (e.g., "kitchen submeter is a good proxy for meal-time occupancy") but not required for the core forecasting task.

### Realistic public datasets (evaluated for this project)

**1. UCI "Individual Household Electric Power Consumption"**
- **What it contains:** One French household's electricity use, minute-level, over ~4 years (Dec 2006–Nov 2010). Fields: global active/reactive power, voltage, current intensity, and three sub-metering channels (kitchen, laundry, water-heater/AC).
- **Size:** ~2.07 million rows, single household.
- **Good for forecasting?** Yes — this is the single most commonly used dataset for exactly this kind of student project, precisely because it's clean-ish, long, and well documented. You'll resample it to hourly or daily to make it tractable.
- **Limitations:** No weather data included (you'd have to fetch historical weather for Sceaux, France, separately and join on date — doable but extra work). No occupancy data. Only one household, so you can't claim any cross-household generalization. About 1.25% of rows have missing values you'll need to handle.
- **Link:** https://archive.ics.uci.edu/dataset/235/individual+household+electric+power+consumption

**2. "Smart Meters in London" (UK Power Networks Low Carbon London project, distributed via Kaggle)**
- **What it contains:** Half-hourly electricity readings for ~5,567 London households (Nov 2011–Feb 2014), **plus a separate hourly weather file for London (temperature, humidity, etc.) and a household metadata file** (ACORN socioeconomic group, tariff type) — this is the closest thing to an "occupancy-ish" signal you'll find in a clean public dataset, since ACORN group correlates with household size/type.
- **Size:** Large — thousands of households, each with 1–2+ years of half-hourly data.
- **Good for forecasting?** Yes, and it's arguably the best fit for your brief because it already bundles consumption + weather + household metadata in one place, matching the "time, weather, and occupancy-related data" language in your project description almost exactly.
- **Limitations:** Larger and messier to wrangle (data split across many block files); picking one representative household (or a small cluster) rather than trying to model all 5,567 is the sane scope choice.
- **Link:** https://www.kaggle.com/datasets/jeanmidev/smart-meters-in-london

**3. UCI "Appliances Energy Prediction"**
- **What it contains:** A Belgian low-energy house, 10-minute readings over 4.5 months, with temperature/humidity from **9 rooms inside the house** plus outdoor weather from a nearby airport station (temperature, pressure, humidity, wind, visibility, dew point).
- **Size:** ~19,735 rows, single household.
- **Good for forecasting?** Usable, and its rich indoor sensor data is genuinely interesting for feature engineering, but 4.5 months is short — you won't see seasonal (summer/winter) effects, which weakens the "weather matters" story a bit.
- **Limitations:** Short time span; single household; the original paper's own best model only reached ~57% R², which is a useful, honest benchmark to cite if your numbers land in a similar range.
- **Link:** https://archive.ics.uci.edu/dataset/374/appliances+energy+prediction

**Our recommendation:** Start with **UCI Individual Household Electric Power Consumption** as your primary dataset (best documented, most widely used for this exact task type — meaning more tutorials/reference material exists if your team gets stuck) and treat the **London Smart Meters** dataset as your stretch/secondary option if you want richer weather+metadata integration and have time. Don't try to use both as core to the report — pick one, mention the other as "considered but not used, because..." in your synopsis. That sentence alone shows evaluators you did real dataset diligence.

---

## 3. ML Approach: Linear Regression vs Random Forest vs XGBoost vs LSTM

| Model | Role | Reasoning |
|---|---|---|
| **Linear Regression** | **Baseline (mandatory)** | Fast, interpretable, and — critically — gives you a number to beat. Every forecasting report needs a "does the fancy model actually help?" comparison, and linear regression is the cheapest way to get that number. Also genuinely competitive once you've engineered good lag/time features. |
| **Random Forest** | **Main model candidate** | Handles non-linear relationships and feature interactions (e.g., temperature × hour-of-day) without much tuning. Robust to messy/noisy household data. Gives you feature importance, which is a great, easy-to-present result ("temperature and hour-of-day were the two most important features"). |
| **XGBoost** | **Main model candidate** | Usually squeezes out somewhat better accuracy than Random Forest on tabular time-series-as-features data, with similar effort once you already have Random Forest working (same feature set, similar API). Worth including as your "best tabular model" if time allows — but if your team is under time pressure, Random Forest alone is a defensible main model and XGBoost becomes optional polish. |
| **LSTM** | **Optional / advanced, only if justified** | See below. |

### Is LSTM actually justified here? Be honest about it.

LSTM is suggested in your brief, and it's tempting because it "sounds advanced," but you should **not** default to it. Here's the honest evaluation:

- LSTMs are built to model *long, complex sequential dependencies* that simple lag features can't capture. For household electricity use, the dominant patterns (hour of day, day of week, season, temperature) are **strongly periodic and largely captured by well-engineered lag/calendar features fed into a tree-based model.** In multiple published comparisons on datasets very similar to yours, gradient-boosted trees (XGBoost/GBM) matched or beat LSTM on this kind of tabular-with-time-structure problem, especially with limited data.
- LSTMs need **more data, more tuning, and more training time** to actually outperform trees — resources a 3-month, 4-person, first-ML-project team doesn't have in abundance.
- LSTMs are **much harder to debug and explain** in a viva/defense. "Why did the model predict this?" has a real answer for Random Forest (feature importance) and a much weaker one for LSTM.

**Verdict:** treat LSTM as a **C — optional, only if time permits** item, and only after your tree-based models are working and evaluated. If you do add it, its main value isn't "better accuracy," it's "we investigated whether sequence modeling helps, and here's what we found" — which is a legitimate, honest finding even if the answer is "it didn't help much, and here's why." That is a *better* mini-project story than force-fitting an LSTM into the final result unexplained.

---

## 4. Feature Engineering

| Feature | Why it matters |
|---|---|
| **Hour of day** | The single strongest driver of short-term usage — morning/evening peaks, overnight lows. |
| **Day of week / weekday vs weekend** | Occupancy and routines shift on weekends (different peak times, generally higher daytime use). |
| **Month / season** | Captures heating/cooling load and daylight-driven behavior changes. |
| **Lag values** (e.g., usage 1h ago, 24h ago, same-hour-yesterday, same-hour-last-week) | Electricity use is highly autocorrelated — yesterday's pattern is often your best single predictor of today's. |
| **Rolling statistics** (rolling mean/std over past 3h, 24h, 7 days) | Smooths noise and captures recent "regime" (e.g., "has this household been unusually high-usage this week?"). |
| **Temperature** | Drives heating/cooling appliance load — often the single strongest *external* predictor. |
| **Humidity** | Secondary weather driver, sometimes relevant for cooling/dehumidifier load. |
| **Occupancy (or a derived proxy)** | Directly gates whether appliances are likely running at all — but as noted in Section 2, you'll likely need to *derive* this (e.g., a binary "active period" flag from consumption patterns) rather than get it as ground truth. |
| **Holiday / special-day flag** | Cheap to add, explains anomalies (holidays behave like weekends or unlike anything else). |

Don't over-engineer this list — 10–15 well-chosen features are plenty for a mini-project. A common trap is spending two weeks building 60 features when 12 good ones would get you 90% of the achievable accuracy.

---

## 5. Train/Test Strategy (Avoiding Data Leakage)

This is one of the most common places student time-series projects lose marks, so get it right:

- **Never shuffle-split time-series data randomly.** A random train/test split lets the model "see the future" (train on data from after the test period), which silently inflates your reported accuracy and won't hold up if an evaluator asks "how did you split this?"
- **Use a chronological split**: e.g., train on the first ~80% of the timeline, test on the final ~20% (a true future holdout). If your data spans multiple years, consider testing on the most recent full season so you're evaluating on data your model has never seen even indirectly.
- **For validation during model tuning**, use **time-series cross-validation** (also called "rolling-origin" or "walk-forward" validation): train on an expanding/sliding window, validate on the next chunk, slide forward, repeat. Scikit-learn's `TimeSeriesSplit` implements this directly — you don't need to build it yourself.
- **Watch for leakage in your engineered features specifically**: a rolling mean or lag feature computed using a window that accidentally includes the current or future row is a classic, easy-to-miss leakage bug. Always compute lag/rolling features using only *past* data relative to each row.

---

## 6. Evaluation Metrics

| Metric | What it tells you | Report it? |
|---|---|---|
| **MAE** (Mean Absolute Error) | Average absolute prediction error, in the same units as your target (kWh). Easy to explain to a non-technical evaluator: "on average, we're off by X kWh." | **Yes — report this.** |
| **RMSE** (Root Mean Squared Error) | Similar to MAE but penalizes large errors more heavily. Useful alongside MAE — if RMSE >> MAE, you have some bad outlier predictions worth discussing. | **Yes — report this.** |
| **R²** | Fraction of variance explained by the model (0–1, higher is better). Good headline number for a synopsis/summary slide. | **Yes — report this.** |
| **MAPE** (Mean Absolute Percentage Error) | Error as a percentage — intuitive, but **breaks down (divides by near-zero) when actual consumption is very low**, which happens often at low-usage hours in household data. | **Optional — report only if your target rarely goes near zero; otherwise skip or flag the caveat explicitly.** |

**Report MAE + RMSE + R² together** as your standard trio; mention MAPE only with a caveat if you use it at all. Always report metrics on your held-out chronological test set, never on training data.

---

## 7. System Architecture: What's Actually Necessary

Going through your considered stack:

| Technology | Verdict | Why |
|---|---|---|
| Python | **Must have** | Obvious base for everything below. |
| Pandas / NumPy | **Must have** | Core data wrangling — no way around this. |
| Scikit-learn | **Must have** | Linear Regression, Random Forest, `TimeSeriesSplit`, metrics — covers most of your modeling needs. |
| XGBoost | **Nice to have** | Add once your scikit-learn pipeline works; it's a drop-in addition, not a rearchitecture. |
| Matplotlib / Plotly | **Must have** | You need consumption graphs and actual-vs-predicted plots regardless of what else you build. Plotly if you want interactivity in the demo, Matplotlib is fine (and simpler) if you just need static report figures. |
| **FastAPI** | **Optional / nice to have** | Only needed if you want a real backend serving live predictions to a frontend. For a mini-project, a Jupyter notebook + a simple Streamlit app can demo everything a FastAPI+React stack would, with a fraction of the integration work. |
| **React** | **Optional, likely unnecessary** | Building a full React frontend is a significant, separate skill investment for a 3-month, 4-person team that's also learning ML from scratch. This is the single biggest over-engineering risk in your stack list. |
| **PostgreSQL** | **Unnecessary for this project** | Your entire dataset fits comfortably in CSV/Parquet files loaded into Pandas. A database adds real setup and maintenance overhead for zero forecasting benefit. Only justified if a professor specifically requires "a database layer" as a rubric item. |
| Git/GitHub | **Must have** | Non-negotiable for a 4-person team — you need version control and a shared history regardless of ML content. |

**Bottom line:** Python + Pandas/NumPy + scikit-learn + XGBoost + Matplotlib/Plotly + Git is a complete, sufficient stack for the ML core. FastAPI+React+PostgreSQL is a legitimate *production* stack, but for a mini-project it multiplies your integration surface area (auth, API contracts, DB schema, deployment) for a task that's supposed to be about forecasting, not full-stack engineering. If your rubric specifically requires "a web application," a single **Streamlit app** (pure Python, no separate frontend/backend split) delivers the same demo value — graphs, predictions, interactivity — in a fraction of the time.

---

## 8. Realistic Final Demo Scope

Keep the demo to things that directly showcase your forecasting work:

- **Historical consumption graph** — line chart of actual usage over time (with a date-range selector if using Streamlit/Plotly).
- **Predicted vs. actual overlay** on your test period — this is your single most important chart; it's the direct visual proof your model works.
- **Peak-demand highlighting** — mark/annotate the highest-usage hours or days on the chart. This directly satisfies "help users understand peak-demand periods" from your brief with minimal extra engineering (it's just sorting/thresholding your existing prediction output).
- **One basic energy-saving insight** — e.g., "your predicted peak is at 7–9 PM; shifting laundry to off-peak hours could reduce your peak load by X%." This can be a simple rule-based observation layered on top of your predictions — it doesn't need its own model.

**Skip:** user accounts/login, multi-household comparison dashboards, real-time streaming ingestion, mobile apps. None of these are asked for in your brief, and each is a multi-week side-project in its own right.

---

## 9. Suggested Division of Responsibilities (Team of 4)

Everyone should understand the whole pipeline end-to-end (you'll all be questioned individually in a viva), but for day-to-day ownership:

1. **Data & Preprocessing lead** — dataset acquisition, cleaning, missing-value handling, resampling, weather join, train/test split logic.
2. **Feature Engineering & Baseline lead** — building the lag/rolling/calendar features, implementing and tuning Linear Regression + Random Forest.
3. **Modeling & Evaluation lead** — XGBoost (and LSTM, if attempted), metric computation, cross-validation, model comparison table/charts.
4. **Application & Documentation lead** — the demo app (Streamlit/simple frontend), final report/synopsis assembly, and coordinating the presentation.

Rotate a "pairing" session every couple of weeks (e.g., whoever owns modeling walks the data-lead through the feature pipeline) so no one is a black box to the rest of the team by December.

---

## 10. Biggest Risks and How to Reduce Them

| Risk | Mitigation |
|---|---|
| **Unsuitable dataset** | Lock in your dataset choice in Week 1–2 (see Section 2) after actually downloading and eyeballing it — don't discover problems in Week 6. |
| **Insufficient data** | Prefer the UCI household or London Smart Meters datasets (both multi-month-to-multi-year) over shorter alternatives; resample to hourly/daily if minute-level data is too heavy to process on your machines. |
| **Missing occupancy/weather info** | Plan for *proxies* from day one (Section 2) rather than assuming you'll find ground-truth occupancy data partway through — this is the single most likely "we expected X but the data didn't have it" surprise. |
| **Data leakage** | Enforce chronological splitting and past-only feature computation as a team rule from the start (Section 5) — retrofitting this after building leaky features means redoing your evaluation. |
| **Overfitting** | Use time-series cross-validation, compare train vs. test metrics explicitly, and don't over-tune Random Forest/XGBoost hyperparameters against your one test set (that itself becomes a subtle leakage). |
| **Model complexity (LSTM temptation)** | Get baseline + tree models fully working and evaluated *first*. Only attempt LSTM with time left over, and treat a "trees beat LSTM here" result as a valid finding, not a failure. |
| **Unrealistic scope (React/FastAPI/PostgreSQL creep)** | Default to the minimal stack in Section 7; add complexity only if core forecasting work is done early and time genuinely remains. |
| **Lack of time** | Build a strict roadmap with hard checkpoints (Section 11) and treat "baseline model working end-to-end" as your first milestone, before any accuracy optimization — a working simple pipeline beats an unfinished sophisticated one. |

---

## 11. High-Level Roadmap (Sept–Nov 2026, report due December)

| Phase | Rough timing | Deliverable |
|---|---|---|
| **1. Dataset finalization** | Weeks 1–2 | Dataset chosen and downloaded, initial structure understood |
| **2. Preprocessing** | Weeks 2–3 | Cleaned, resampled, weather-joined (if applicable) dataset; missing values handled |
| **3. EDA** | Weeks 3–4 | Plots of usage patterns by hour/day/season; correlation with weather; write-up of observed patterns |
| **4. Feature engineering** | Weeks 4–5 | Final feature set built and validated (no leakage) |
| **5. Baseline model** | Week 5–6 | Linear Regression trained, evaluated — this is your first real milestone |
| **6. Model comparison** | Weeks 6–8 | Random Forest, XGBoost (and LSTM if pursuing) trained and compared against baseline using the same test set |
| **7. Evaluation & analysis** | Week 9 | Metrics table, actual-vs-predicted plots, feature importance discussion, error analysis |
| **8. Application** | Weeks 9–10 | Demo app (Streamlit or minimal frontend) wired to your best model |
| **9. Testing** | Week 11 | End-to-end run-through, edge cases, fix rough edges |
| **10. Documentation** | Week 12 (November→December) | Final report, synopsis polish, presentation prep |

Build in a buffer — Week 8–9 or so — since ML timelines slip more than typical software timelines (a model that "should just work" often doesn't on the first few tries).

---

## 12. Synopsis Section Guidance

- **Introduction:** Frame the real-world problem (rising household electricity costs, need for consumption awareness) and state plainly that you're building a forecasting model, not a smart-metering hardware system.
- **Objectives:** 3–4 concrete, testable objectives — e.g., "predict next-hour/next-day household consumption," "compare baseline vs. ensemble models," "identify peak-demand periods," "present predictions through a simple application." Avoid vague objectives like "leverage AI for energy efficiency."
- **Methodology:** Walk through dataset → preprocessing → feature engineering → model training/comparison → evaluation → application, referencing the models and metrics you've actually decided on (Sections 3, 6) — don't list every algorithm mentioned in the original brief if you're not using all of them.
- **Expected Outcome:** State it modestly and honestly — e.g., "a working forecasting model with quantified accuracy (MAE/RMSE/R²) and a simple demo application," not "a production-ready smart energy management system."
- **Application:** Household energy awareness, informing utilities/consumers about peak-demand patterns, foundation for future demand-response or smart-scheduling systems (frame future scaling as *future work*, not something you're delivering now).
- **Industry/Institution relevance:** Ties to smart-grid and energy-efficiency initiatives, relevance to IoT/smart-metering rollouts (mention India's or your utility's smart-meter initiatives if you want local relevance), and to VTU's push toward data-driven/ML coursework.
- **Budget:** Should be minimal/near-zero — public datasets, open-source libraries, and (if needed) free-tier cloud compute/hosting for the demo. Explicitly state "no proprietary data or paid infrastructure required" — this is a genuine strength of your project design, say so.

---

## 13. Final Recommendation

**A. MUST HAVE**
- Prediction target: **next-hour consumption**, with a rolled-up next-day view for peak-demand analysis.
- Dataset: **UCI Individual Household Electric Power Consumption** (resampled to hourly), or **London Smart Meters** if your team wants built-in weather + household metadata.
- Models: **Linear Regression (baseline) + Random Forest**, compared on identical chronological train/test splits.
- Features: hour, day-of-week, month/season, key lag values (1h, 24h, same-hour-last-week), rolling mean/std, temperature (if available).
- Evaluation: **MAE, RMSE, R²** on a proper chronological holdout, with time-series cross-validation during tuning.
- Stack: Python, Pandas/NumPy, scikit-learn, Matplotlib/Plotly, Git/GitHub.
- Demo: historical + actual-vs-predicted charts, peak-demand highlighting, one energy-saving insight.

**B. NICE TO HAVE**
- **XGBoost** as an additional model for comparison against Random Forest.
- Weather data joined in if your primary dataset doesn't already include it.
- A **Streamlit** (or similarly lightweight) demo app instead of a notebook-only deliverable.
- A derived occupancy proxy feature (from consumption patterns or household metadata like ACORN group).

**C. OPTIONAL / ONLY IF TIME PERMITS**
- **LSTM**, attempted only after tree-based models are fully working — treated as an investigation ("does sequence modeling help here?") rather than an assumed upgrade.
- FastAPI backend + React frontend, only if a full "web application" is explicitly required by your rubric and core ML work is finished early.
- PostgreSQL — essentially never justified for this project's actual data volume; skip unless mandated.
- MAPE reporting — only if your target values are reliably far from zero.

If your team accomplishes everything in **A**, you have a complete, defensible, well-scoped mini-project. Everything in **B** and **C** is genuine bonus value, not a requirement — resist the pull to chase them before A is solid and well-tested.

---

### Sources

- UCI Machine Learning Repository — Individual Household Electric Power Consumption: https://archive.ics.uci.edu/dataset/235/individual+household+electric+power+consumption
- Kaggle / London Datastore — Smart Meters in London (UK Power Networks, Low Carbon London project): https://www.kaggle.com/datasets/jeanmidev/smart-meters-in-london
- UCI Machine Learning Repository — Appliances Energy Prediction: https://archive.ics.uci.edu/dataset/374/appliances+energy+prediction
- Candanedo, Feldheim, Deramaix (2017), "Data driven prediction models of energy use of appliances in a low-energy house," *Energy and Buildings*, Vol. 140 (source of the ~57% R² GBM benchmark referenced in Section 3)
- Monash/UEA/UCR Time Series Extrinsic Regression Archive (dataset framing and preprocessing references): https://arxiv.org/pdf/2006.10996