# Analytics Lifecycle Alignment: Churn Project

Oct 12, 2026 · @Atul Gupta

The six-phase Data Analytics Lifecycle and OSEMN describe the same work from two angles. This doc pins each lifecycle phase to its OSEMN step, its days in the three-day schedule from Doc, and a gate to pass before moving on. The schedule is three working days of about 6.5 hours each.

## Phase crosswalk

Six lifecycle phases map onto four OSEMN steps (Obtain is skipped). Explore straddles two lifecycle phases: surveying the data belongs to Data Preparation, and choosing features and models belongs to Model Planning.

| Lifecycle phase | OSEMN step | When | Core activities | Exit gate |
| --- | --- | --- | --- | --- |
| 1. Discovery | Project setup (before Scrub) | Day 1, first hour | Business question, churn definition, success criteria, dataset choice, repo setup | Written problem statement and metric target |
| 2. Data Preparation | Scrub, plus first-pass Explore | Day 1, 5.5 h of cleaning; Day 2, first 1.5 h of EDA | Profile, clean, build target, aggregate to customer level, survey and visualize | Clean customer table passes validation checks |
| 3. Model Planning | Explore (second half) | Day 2, next 1 h | Correlations, feature engineering, leakage check, pick candidate models and metrics | Final feature list and modeling plan |
| 4. Model Building | Model | Day 2, 4 h; Day 3, first 1.5 h | Split, baseline, cross-validation, tuning, threshold, one test-set check | Saved model beating the baseline on the agreed metric |
| 5. Communicate Results | iNterpret | Day 3, 3 h | SHAP or coefficients, at-risk segments, impact estimate, report | Report reviewed against the success criteria from Day 1 |
| 6. Operationalize | iNterpret (stretch) | Day 3, last 2 h | Batch scoring script or Streamlit app, A/B test plan, retraining and monitoring notes | Repeatable scoring path and a written pilot plan |

## Timeline

&#91;embedded content: lifecycle phases over 3 working days · 6 phases, 6 gates\]

Each bar runs from a phase's first to last day, and the diamond after it is its gate. Operationalize shares Day 3 with Communicate Results, so treat it as a stretch goal.

## Phase gates

Tick every box in a phase before starting the next. A gate you cannot tick is a reason to loop back, not to skip ahead.

**Gate 1: Discovery (Day 1, after the first hour)**

- [ ] Business question and decision owner written down
- [ ] Churn defined precisely (cutoff date, observation window)
- [ ] Success criteria set (e.g. recall and precision targets, beating the baseline)
- [ ] Dataset chosen and raw file stored untouched

**Gate 2: Data Preparation (Day 2, late morning)**

- [ ] Every cleaning step is code that runs from raw to clean
- [ ] Data-quality log lists each issue, count, fix and reason
- [ ] Validation checks pass (unique customer IDs, no nulls in key columns, valid ranges)
- [ ] 3–4 labeled insights from the first EDA pass

**Gate 3: Model Planning (Day 2, midday)**

- [ ] Leakage check done for every feature
- [ ] Final feature list with a one-line rationale each
- [ ] Candidate models, validation scheme and metrics chosen

**Gate 4: Model Building (Day 3, late morning)**

- [ ] Baseline and at least two candidates compared with cross-validation
- [ ] Threshold chosen from a stated cost assumption
- [ ] Test set used once; model and pipeline saved

**Gate 5: Communicate Results (Day 3, afternoon)**

- [ ] Report answers the Day 1 business question
- [ ] Every impact figure has its assumptions labeled
- [ ] Limits and fairness checks stated

**Gate 6: Operationalize (stretch, Day 3 evening)**

- [ ] A script or app scores new customers into a ranked list
- [ ] A/B test plan and a retraining trigger are written down

## Sync rules

The lifecycle is iterative, while the schedule is linear. These rules keep the two from drifting apart.

1. **Gates drive the calendar.** A phase ends when its gate is ticked, not when its time block runs out. If a gate is open at the end of its block, record the slip in a one-line log entry.
2. **Scrub gets the buffer.** Cleaning is the main learning goal here, so overruns on Day 1 take time from the Day 2 Explore block, never from Model or Communicate.
3. **Loops are allowed, and named.** Allowed back-edges: Model Planning → Data Preparation (a feature needs new cleaning), Model Building → Model Planning (weak results, new features), Communicate → Model Building (reviewers question a metric). Log each loop with its cause.
4. **Freeze before you build.** After Gate 3, the feature list and target definition change only through a logged loop-back, so results stay comparable.
5. **One status line per day.** Each day ends with: phase, gate items done, blockers, and tomorrow's first task.
6. **Cut in a fixed order.** Three days leave no slack to shift the calendar. If Gate 2 is not ticked by late morning on Day 2, drop the Operationalize stretch first, then hyperparameter tuning, then extra candidate models.
