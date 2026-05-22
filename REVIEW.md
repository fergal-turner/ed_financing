# Full Review: Education Financing Analysis

**Review Date:** May 13, 2026  
**Analysis Folder:** Education Financing/  
**Status:** Early-to-mid stage; analysis incomplete but foundational work sound

---

## Executive Summary

| Dimension | Score | Status |
|-----------|-------|--------|
| Technical Soundness | 72/100 | 🟡 Mostly sound with reproducibility and documentation gaps |
| Methodology & Clarity | 65/100 | 🟡 Clear conceptually but strategic choices underjustified |
| Communication & Validity | 70/100 | 🟡 Visualizations effective, captions need refinement |
| **Overall** | **69/100** | **🟡 Solid foundation; address core concerns before finalization** |

### Key Findings

1. **Data aggregation issue**: Collapsing country-year data to single row per country using `.agg('first')` may not be theoretically justified and reduces statistical power.
2. **Assumptions not validated**: OLS models lack diagnostic checks (normality, homoscedasticity); interaction models use standardization inconsistently.
3. **Research question ambiguity**: Is this causal analysis or descriptive association? The framing shifts between sections without clear statement of intent.

### Recommended Priorities

1. **Critical**: Justify data aggregation strategy (why `agg('first')`? Consider averaging over years or using panel structure)
2. **Important**: Standardize OLS diagnostic approach across all models; document assumptions
3. **Important**: Clarify research question and causal framing in opening section
4. **Nice-to-have**: Refine figure captions and validate "visually apparent" claims statistically

---

## Expert Colleague Review — Technical Soundness

### Issues Found

#### **Issue 1: In-place Data Modification (Reproducibility)**
**Location:** 01. Spending, Income & LAYS.ipynb, Cell 11  
**Problem:** The line `all_data = all_data.drop(...)` modifies the global `all_data` object in-place. This affects all downstream analyses that depend on `all_data` and makes the analysis harder to reproduce if cells are re-run in different orders.  
**Code snippet:**
```python
all_data['expenditure_perchild_ppp'].describe()
all_data = all_data.drop(all_data[all_data['expenditure_perchild_ppp'] < 10].index, axis=0)
```
**Fix:** Create a copy: `df = all_data.copy()` before filtering. Document the exclusion logic (why < 10 PPP threshold?) in a markdown cell.

---

#### **Issue 2: Questionable Data Aggregation Strategy**
**Location:** 01. Spending, Income & LAYS.ipynb, Cell 12  
**Problem:** Using `.agg('first')` to collapse country-year data to one row per country is theoretically unclear. This takes only the first observation (likely lowest year) for each country. The analysis would have much more power using panel structure or averaging across years.  
**Code snippet:**
```python
df = df.groupby(["iso3"]).agg('first').reset_index()
```
**Why this matters:** 
- Discards temporal variation and reduces sample size from ~N*T to ~N observations
- The README mentions `lays_panels.csv` was created for panel analysis, but Cell 12 doesn't use it
- Results may be biased toward older year data

**Fix:** Either (a) use panel structure explicitly with fixed effects, (b) average key variables by country and model country-level relationships, or (c) clarify in methodology why single-year-per-country is appropriate for your research question.

---

#### **Issue 3: Inconsistent Standardization in Regression Models**
**Location:** 01. Spending, Income & LAYS.ipynb, Cell 14-16  
**Problem:** Models use different standardization approaches:
- Cell 14 (linear/log/quartic GDPPC models): Standardize GDPPC but not LAYS
- Cell 16 (expenditure models): Standardize expenditure but not LAYS
- Cell 20 (interaction models): Standardize predictors but not outcome

This inconsistency makes coefficients harder to compare and unclear what "standardized" means in context.  
**Fix:** Decide on a consistent approach: (a) standardize all predictors AND outcome for comparability, or (b) standardize predictors only and report unstandardized coefficients with 95% CIs in a summary table. Document the choice.

---

#### **Issue 4: Missing Model Diagnostics**
**Location:** 01. Spending, Income & LAYS.ipynb, All regression cells (14-23)  
**Problem:** No diagnostic tests for OLS assumptions:
- No residual plots or normality tests (Shapiro-Wilk, Q-Q plots)
- No heteroscedasticity tests (Breusch-Pagan)
- No multicollinearity checks (VIF) except mentioned in Notebook 02

**Why it matters:** With small subgroup samples (e.g., ~20 LICs), violations of OLS assumptions could inflate standard errors or bias estimates.  
**Fix:** After fitting each model, run and display:
```python
import matplotlib.pyplot as plt
from statsmodels.graphics.gofplots import ProbPlot
# Add diagnostic plots
fig, axes = plt.subplots(2, 2)
sm.qqplot(model.resid, line='45', ax=axes[0, 0])
...
```

---

#### **Issue 5: Unclear Panel Construction Logic**
**Location:** README.md, Section "How `lays_panels` Was Constructed"  
**Problem:** The panel construction in `lays_panels.csv` uses 5 different interval lengths (panel1: 7 yrs, panel2: 8 yrs, ..., panel5: 2 yrs) with different annualization denominators. The theoretical justification for this specific design is not stated.  
**Why it matters:** If panels are meant to test robustness, the varying time windows confound time-span effects with other changes.  
**Fix:** Document in a README or notebook cell: (a) Why these specific intervals? (b) Is there a sensitivity analysis showing robustness across intervals?

---

#### **Issue 6: Visualization Data Consistency**
**Location:** 01. Spending, Income & LAYS.ipynb, Figures across cells  
**Problem:** Different figures use different sample filters (e.g., Cell 15 filters for non-missing `expenditure_perchild_ppp` but earlier scatter plots may not). Check that sample sizes are consistent or clearly documented.  
**Fix:** Add a cell after data loading documenting: "Final analysis sample: N=[X] countries, across [Y] years."

---

#### **Issue 7: Statistical Significance Not Reported Consistently**
**Location:** 01. Spending, Income & LAYS.ipynb, Summary tables (Cell 23)  
**Problem:** The summary table exports coefficients and p-values but (a) doesn't mark significance stars (*/***/****), (b) doesn't include 95% CIs, (c) no note on sample size per subgroup.  
**Fix:** Add columns: `N`, `CI_lower`, `CI_upper`, `Sig` (star notation), and export to CSV or formatted table.

---

### Technical Soundness Score: **72/100**

**Strengths:**
- Code structure is clear and well-commented
- Correct use of statsmodels API
- Figures are reproducible and well-formatted
- Data sourcing documented in README

**Weaknesses:**
- Reproducibility issues (in-place modification)
- Data aggregation strategy unjustified
- Diagnostic tests missing
- Inconsistent model specifications

---

---

## Peer Reviewer — Methodology & Conceptual Clarity

### Concerns & Suggestions

#### **Concern 1: Research Question Not Clearly Stated**
**Impact:** Without a clear causal vs. descriptive framing, it's hard to evaluate whether your methodological choices (e.g., OLS vs. causal inference methods) are appropriate.

**Current approach:** The notebooks describe relationships (e.g., "GDPPC correlates with LAYS") but don't state:
- Is this descriptive (How much do countries with higher GDP per capita tend to have higher LAYS)?
- Is this predictive (Given a country's GDPPC, can we predict LAYS)?
- Is this causal (If a country increases GDPPC, will LAYS increase)?

**Questions for the author:**
- What is your primary research question? (Write it in one sentence at the top of Notebook 01.)
- Are you interested in within-country dynamics (over time) or between-country differences?
- Do you expect causal effects or are you mapping associations to inform theory?

**Suggested edits:**
- Add a "Research Questions" section to the README with 2–3 explicit questions
- Revise Notebook 01 opening to state: "This notebook describes the **cross-national association** between GDPPC and LAYS, controlling for [X]. We do not make causal claims."
- If causal inference is intended, consider: (a) IV strategy, (b) natural experiments, or (c) explicit DAG of confounders

---

#### **Concern 2: Data Aggregation Strategy Lacks Justification**
**Impact:** Collapsing to one-row-per-country loses the temporal structure of the data and reduces sample size, but the justification is not clear.

**Current approach:** Cell 12 uses `.agg('first')` to get one observation per country, discarding panel structure.

**Questions for the author:**
- Why not leverage the panel structure you carefully constructed in `lays_panels.csv`?
- Is this a cross-sectional analysis by design, or could it be a panel analysis?
- How many countries are retained in the final sample? (Report this explicitly.)
- Does the choice to use first-year observations introduce year-selection bias?

**Suggested edits:**
- If cross-sectional is intentional: Clearly state "We collapse to country-level using [method]. This gives us N=[X] unique countries." Document why this is the right level of analysis.
- If panel is preferred: Use fixed-effects or random-effects models instead, and leverage all time periods.
- Consider a robustness check: Repeat key analyses using (a) latest year per country, (b) average across years, and (c) panel FE. Do conclusions hold?

---

#### **Concern 3: Income-Group Heterogeneity Is Visually Suggestive but Not Rigorously Tested**
**Impact:** Notebook 01 observes that LICs have "negative" or "non-significant" slopes, but this is not formally tested.

**Current approach:** Fig 3 shows separate regression lines by income group, and Cell 22 notes "for LICs the relationship is negative and not statistically distinguishable from zero" based on visual inspection.

**Questions for the author:**
- Have you tested whether slopes differ significantly across income groups? (Formal test: Chow test or interaction term significance)
- Is the sample size for each income group adequate? (e.g., n=8 LICs may lack power)
- Could the "negative" slope in LICs be driven by outliers? (Do you have influence diagnostics?)

**Suggested edits:**
- Add a formal test of slope heterogeneity (e.g., interaction model with Wald test) and report in text
- State: "Slopes differ significantly across income groups (F-stat=[X], p=[Y])."
- Add a note on subsample sizes: "LIC sample: N=[X] countries."
- Consider robust regression or outlier diagnostics to check sensitivity

---

#### **Concern 4: Alternative Specifications Not Justified**
**Impact:** Notebook tests linear, log, and quartic models but doesn't justify why these forms.

**Current approach:** Cell 14-17 fit three models and compare R², but no prior reasoning for why quartic is theoretically plausible.

**Questions for the author:**
- What is the theoretical basis for a quartic relationship? The README mentions "diminishing returns," which would support a log or quadratic form, but quartic adds turning points that need justification.
- Why not test quadratic (inverted-U) if the hypothesis is diminishing returns?
- Are you doing model selection based on fit alone, or do you have priors?

**Suggested edits:**
- In Notebook 01 opening, add a sentence: "We test three functional forms: linear (baseline), log (diminishing returns hypothesis), and quartic (exploratory). We select the best-fitting model and present results with [appropriate penalty, e.g., AIC]."
- If quartic is exploratory, say so explicitly. If it's from prior work (Pritchett et al.), cite and explain the motivation.

---

#### **Concern 5: Macro Drivers Notebook (02) Conceptually Incomplete**
**Impact:** Notebook 02 states it will explore state fragility, SAP, language fractionalization, etc., but the methodology section lists 5 factors without prioritizing or sequencing them.

**Current approach:** Cell 2 of Notebook 02 describes 5 macro drivers but doesn't build a causal model or DAG.

**Questions for the author:**
- How do these 5 factors relate to each other? (e.g., is language fractionalization a confounder or mediator?)
- Are these simultaneous predictors or do they operate in a causal sequence?
- Why start with a "complete model" with all 5? This risks multicollinearity and interpretation issues.

**Suggested edits:**
- Add a causal diagram (DAG) or conceptual model showing how macro factors influence the spending→LAYS pathway
- Propose a sequence: (a) baseline (spending only), (b) + macro factors, (c) + interactions
- State hypothesis for each factor: "State fragility is expected to reduce the efficiency of spending because [mechanism]."

---

#### **Concern 6: Interpretation of Findings Lacks Depth**
**Impact:** Results are descriptive but don't address why low-income countries show weak GDPPC-LAYS relationship.

**Current approach:** Notebook 01 notes the LIC slope is "negative" but doesn't discuss implications.

**Questions for the author:**
- If LICs spend more but LAYS don't improve, what are the hypothesized blockers? (Teacher quality? Curriculum? Language?)
- Does Notebook 02 aim to explain this? (State clearly.)
- Are there outlier countries doing well despite low spending? (Identify and discuss.)

**Suggested edits:**
- Add a "Discussion" section to Notebook 01 interpretting the LIC finding: "Low-income countries show weak association between GDPPC and LAYS, suggesting that income growth alone is insufficient. Macro drivers (Notebook 02) test whether [X, Y, Z] explain this gap."
- Identify specific outlier countries and note them for case study follow-up

---

### Methodology & Clarity Score: **65/100**

**Strengths:**
- Clear data sourcing and construction documented
- Multiple analytical angles (functional forms, income groups, time panels)
- Macro drivers framed thoughtfully (5-factor model is comprehensive)

**Weaknesses:**
- Research question not explicitly stated
- Data aggregation strategy not justified
- Heterogeneity findings suggestive but not formally tested
- Interpretation lacks depth

---

---

## Expert in the Public — Communication & Validity

### Critical Questions & Observations

#### **Observation 1: Figure 3 Caption Vague About Highlighted Rectangles**
**Location:** Fig 3 caption (Cell 22) and visualization  
**What looks off:** The figure shows two gray rectangles overlaid on the scatter plot, but the caption doesn't explain what they highlight.  
**Caption reads:** "...we see that for Low-Income Countries the relationship is negative, and not statistical distinguishable from zero."  
**Questions:**
- What do the two rectangles represent? (Outlier regions? Confidence bands? Visually notable clusters?)
- Why are they only on two of the income groups' plots?
- Are these clusters statistically significant or just "look off"?

**Why it matters:** A reader can't interpret the figure without knowing what the rectangles mean. If they mark clusters, that finding deserves a formal statistical test and explicit caption.

**Suggested fix:**
```markdown
Figure 3: LAYS vs Log-GDPPC by Income Group

The gray rectangles highlight visual clusters: 
- Left rectangle (LIC panel): low-spending, low-LAYS cluster
- Right rectangle (LMIC/UMIC panels): high-spending, mid-range-LAYS outliers
These patterns are suggestive and not yet formally tested.
```

---

#### **Observation 2: "Negative" Relationship in LICs Needs Plain-English Translation**
**Location:** Cell 22 markdown and Fig 3 caption  
**What looks off:** The statement "for LICs the relationship is negative, and not statistical distinguishable from zero" is technically correct but confusing to a non-specialist.  
**Questions:**
- Does "negative and not significant" mean the slope is negative but the confidence interval crosses zero?
- What does this imply for policy? (Should LIC policymakers NOT spend more?)
- Is this surprising? Why?

**Why it matters:** A policy audience needs to understand: Does GDPPC-spending help LICs or not? The technical phrasing masks the answer.

**Suggested fix:** Add to Cell 22:
```markdown
**Plain-English Interpretation:**  
For low-income countries, the analysis finds a *weakly negative* association 
between per-capita income and learning outcomes. However, this relationship 
is not statistically reliable (could be due to chance or small sample size; 
n=8 LICs). The implication: **income growth alone does not guarantee learning gains 
for the poorest countries.** Other factors (explored in Notebook 02) likely matter more.
```

---

#### **Observation 3: Table of Coefficients Needs Context and Interpretation Notes**
**Location:** Cell 23, `income_group_log_gdppc_slopes.csv`  
**What looks off:** The table exports model coefficients and p-values but no interpretation or sample sizes.  
**Table structure:**
```
Model    | R2     | Income Group | Predictor | Coefficient | P-value
model_5a_1 | 0.147 | Low Income   | Log GDPPC | [value]    | [p]
...
```

**Questions:**
- What do the coefficient values mean? (For a 1% increase in GDPPC, LAYS increase by [X] units?)
- Why do some R² values look so low (0.147)? Is this expected?
- Sample sizes? Are some estimates based on n<10?

**Why it matters:** A reader can't evaluate whether findings are meaningful without context.

**Suggested fix:** Expand the table with:
- `N` (sample size per model)
- `CI_95` (confidence interval lower, upper)
- Interpretation note (e.g., "Coefficient is in LAYS units; unstandardized")
- Export to `.xlsx` with a second sheet explaining each column

Example:
```
Model    | Income Group | Predictor | Coef | 95% CI Low | 95% CI High | P-value | N
model_5a_1 | Low Income   | Log GDPPC | -0.12 | -0.45  | 0.21        | 0.51    | 8
```

---

#### **Observation 4: Figure Titles Are Technical; Audience May Not Understand "Log Scale"**
**Location:** All figures (especially Fig 1, 2)  
**What looks off:** Titles are accurate but assume technical literacy (e.g., "Fitted Curves for the Relationship Between GDPPC and LAYS").  
**Questions:**
- For a policy or practice audience (e.g., educators, development professionals), what does "fitted curves" mean?
- Why does a log scale matter?

**Why it matters:** Accessibility. A title should tell a story, not just name variables.

**Suggested fix:** Expand titles to be more descriptive:
```markdown
Before:  "LAYS vs Log-GDPPC by Income Group"
After:   "Learning Outcomes by Country Wealth: Richer Nations Show Stronger Gains"

Before:  "Fitted Curves Comparing Model Specifications"
After:   "How Does Country Wealth Predict Learning? Linear, Log, and Curved Relationships Compared"
```

---

#### **Observation 5: Missing Context for "Outliers" in Figure 3**
**Location:** Cell 22, markdown  
**What looks off:** The text mentions "visually apparent" outliers and clusters in Fig 3 but doesn't name countries or explain patterns.  
**Exact quote:** "For LMICs and UMICs we have a number positive outliers, i.e., countries achieving stronger LAYS than would be expected for their income level."  
**Questions:**
- Which countries are outliers? (Name 2–3 examples.)
- What are they doing differently? (e.g., private schooling, teacher training, curriculum reform?)
- Is this finding stable across time or driven by one year?

**Why it matters:** Naming outliers makes findings concrete and actionable. A policymaker reading this will ask "Who are the outliers and what can we learn from them?"

**Suggested fix:** After fitting the LIC/LMIC/UMIC regressions, add a small analysis:
```python
# Identify outliers: residuals > 1 SD
residuals = df_lmic['lays'] - model_lmic.predict(...)
outliers = df_lmic[abs(residuals) > residuals.std()]
print(outliers[['iso3', 'lays', 'log_gdppc', 'residuals']])
```
Then add to markdown:
```markdown
**Notable Outliers:**  
- Vietnam: Higher LAYS than expected for income (possible drivers: [hypothesis])
- [Country]: Lower LAYS than expected (possible drivers: [hypothesis])
```

---

#### **Observation 6: LAYS Scale Not Explained for Non-Specialist Readers**
**Location:** Figures and tables throughout  
**What looks off:** Figures show LAYS ranging from ~2 to ~9, but readers outside education may not know what this means.  
**Questions:**
- What is LAYS? (A year of schooling? A test score?)
- What is a "good" LAYS value? (9 years = [what?])
- Why does it matter?

**Why it matters:** Without context, figures are just numbers. A reader needs to understand why LAYS 2 is bad and LAYS 9 is good.

**Suggested fix:** Add a footnote or sidebar to the README:
```markdown
**What is LAYS?**  
Learning-Adjusted Years of Schooling (LAYS) combines quantity (years spent in school) 
with quality (learning gains). A LAYS of 5 means a child has had the learning benefit 
of 5 full years of quality schooling. High-income countries average ~9 LAYS; low-income 
countries average ~2 LAYS.  
Data source: World Bank Human Capital Index
```
Add this to figure captions as a parenthetical.

---

#### **Observation 7: Results Section Lacks Summary Table or Dashboard**
**Location:** Between figures and tables  
**What looks off:** Results are scattered across 3 notebooks and 7 figures + 1 summary table, but there's no one-page summary.  
**Questions:**
- What are the 3 main findings?
- How do Notebooks 01, 02, and 03 connect?
- Should a reader believe the conclusions?

**Why it matters:** A busy policymaker or educator won't read all 3 notebooks. A summary finding table makes results accessible.

**Suggested fix:** Create a `Results_Summary.md` or single-page infographic:
```markdown
# Key Findings: What Drives Learning Outcomes?

## Finding 1: Income Matters, But Not Equally
- Richer countries → higher learning, especially at low income levels  
- For poorest countries, income growth alone insufficient (Notebook 01, p.XX)

## Finding 2: Macro Instability Blocks Gains
- State fragility, rapid population growth reduce spending efficiency (Notebook 02, p.XX)

## Finding 3: How Governments Spend Matters More Than How Much
- Allocation to teacher salaries vs. capital (Notebook 03, p.XX)

[Include 1-2 key figures]
```

---

### Communication & Validity Score: **70/100**

**Strengths:**
- Figures are well-formatted and visually clear
- Color palette is consistent and accessible (sc_colors palette is good)
- Captions attempt to interpret findings

**Weaknesses:**
- Technical jargon not translated to plain language
- Rectangles on Fig 3 unexplained
- Outliers identified but not named or contextualized
- Missing contextual definitions (What is LAYS?)
- No executive summary across notebooks

---

---

## Next Steps

### Critical (Must address before publication/handoff)

1. **Clarify data aggregation strategy**
   - Document why `.agg('first')` is appropriate, OR switch to panel structure (fixed effects)
   - Report final sample size: N=[X] countries

2. **Formalize heterogeneity testing**
   - Run formal Chow test or interaction term significance test for income-group slope differences
   - Report with F-stat, p-value, and interpretation

3. **Standardize regression diagnostic approach**
   - Add residual plots, normality tests, and heteroscedasticity tests to at least one model per notebook
   - Document how assumptions are met or violated

4. **Define research question explicitly**
   - Write a 1-sentence research question at the top of Notebook 01
   - Clarify: descriptive, predictive, or causal intent?

---

### Important (Should address before finalization)

5. **Fix Figure 3 caption**
   - Explain what the gray rectangles represent
   - Add plain-English interpretation of "negative and non-significant"

6. **Expand summary table**
   - Add N, CI, and interpretation notes to coefficient tables
   - Export as `.xlsx` with a metadata sheet

7. **Complete Notebook 02 methodology**
   - Add causal diagram (DAG) showing how macro factors relate
   - State explicit hypotheses for each predictor

8. **Name and contextualize outliers**
   - Identify specific countries as positive/negative outliers
   - Speculate on drivers (teacher quality, curriculum, private schooling, etc.)

---

### Nice-to-have (Consider for future refinement)

9. **Sensitivity analyses**
   - Repeat key results using latest year, averaged data, and panel fixed effects
   - Document robustness across all three approaches

10. **Create results summary dashboard**
    - One-page summary of 3 key findings
    - Include 2–3 key figures and plain-English interpretations

11. **Expand Notebook 03 (COFOG) presentation**
    - Currently incomplete; clarify connection to overall research question
    - Frame as "Where is money spent?" → "Does allocation type predict LAYS?"

12. **Consider case studies of outlier countries**
    - Why does Vietnam do well for its income?
    - Why do some LICs underperform?
    - Qualitative follow-up could enrich the quantitative findings

---

### Strengths to Preserve

- **Data sourcing is meticulous**: The README documenting exact API indicators, transformations, and definitions is excellent. This is reproducibility gold.
- **Multiple analytical angles**: Testing functional forms, heterogeneity by income group, and macro drivers shows sophisticated thinking.
- **Clean code and documentation**: Notebook structure is clear; comments are helpful; Save the Children color palette applied consistently.
- **Transparent about incompleteness**: The analysis honestly notes visual patterns that are "not yet validated" (e.g., outlier clusters). This intellectual honesty is valuable.

---

### Completion Checklist for Final Handoff

- [ ] Data aggregation strategy justified in text
- [ ] Regression diagnostics run and reported for all models
- [ ] Research question stated explicitly in Notebook 01 opening
- [ ] Figure 3 rectangles explained in caption
- [ ] Coefficient tables expanded with N, CI, and interpretation
- [ ] Notebooks 01–03 connected with a summary finding table
- [ ] Outlier countries named and contextualized
- [ ] All three notebooks executed successfully with no errors
- [ ] LAYS and other key metrics defined for non-specialists

---

**End of Review**

---

*This review was conducted using a three-persona approach: Expert Colleague (technical soundness), Peer Reviewer (methodology & conceptual clarity), and Expert in the Public (communication & validity). Comments are intended as constructive feedback to strengthen the analysis before finalization and publication.*
