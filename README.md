# Circadian Distinctness, Login to Learning Management System, and Academic Performance  
### Analysis of 13,894 university students across multiple academic terms

This repository contains data processing and statistical analysis workflows for a large-scale study examining the relationship between circadian activity patterns, social jetlag, and academic performance among university students. 
Analyses focus exclusively on **non-class days**, which reflect endogenous patterns of daily activity rather than externally imposed schedules.

---

## Data Collection

All data were collected under Northeastern Illinois University IRB protocol **#16-073 MO1**.  
A total of **13,894 students** contributed de-identified login-timestamp data from a university Learning Management System (LMS). 

Semester GPAs were computed by converting letter grades into grade points (A = 4.0), then averaging across all courses per semester.

All analyses were performed using:

- **NumPy 1.26.4**  
- **Pandas 2.2.3**  
- **Statsmodels 0.14.4**  
- **Scikit-learn 1.1.3**  
- **Matplotlib 3.9.2**

---

## Circadian Characteristics

### Mean Resultant Length (R)
To quantify the **distinctness** of each student’s circadian rhythm, we computed the **mean resultant length (R)** based on circular statistics.
Properties:

- **R ∈ [0,1]**
- **R → 1** → highly concentrated (consistent login timing; strong rhythmicity)  
- **R → 0** → widely dispersed (irregular login timing; weak rhythmicity)

Interpretation:

- High R → narrow daily “activity window,” stable rhythm  
- Low R → broad or inconsistent activity window  

### Chronotype Classification

Circular login-phase values were computed from non-class-day login times (0–2π radians). The **median phase** defined chronotype:

- **Larks** → >1 SD earlier than group median  
- **Owls** → >1 SD later than group median  
- **Finches** → within ±1 SD of group median  

### Social Jetlag (SJL)

Social jetlag was computed as:

\[
\text{SJL} = \text{Median phase}_{\text{class day}} - \text{Median phase}_{\text{non-class day}}
\]

Values may be **positive or negative**, reflecting delayed or advanced shifts relative to class days.

---

## Data Preprocessing

Because login behavior is **irregularly sampled** and temporally uneven, traditional circadian tools (cosinor, periodogram) are inappropriate. We follow the event-based preprocessing workflow of **Karoly et al. (2021)** for irregular behavioral data.

### 1. Circular Uniformity Tests

Each student’s login distribution was tested for deviation from circular uniformity using three complementary tests:

1. **Rayleigh test** (assumes unimodal von Mises distribution)
2. **Watson U² test** (goodness-of-fit to von Mises)
3. **Hodges–Ajne test** (distribution-free; detects uni-, bi-, and multimodality)

Multiple comparison correction was applied using **Benjamini–Hochberg FDR** (q < 0.05).

### 2. Monte-Carlo Simulation for R Thresholds

Because R is biased upward at small sample sizes, we performed simulation-based calibration:

- For each observed login count n  
- Generate **10,000** sets of n uniform random angles  
- Compute their R values  
- Determine **95th** and **99th** percentile thresholds  
- Compare each student’s observed R against these significance cutoffs  

### 3. Minimum Login Count (n_stable)

To determine the minimum number of logins required for reliable R estimation:

- We evaluated gradients dR/dn of the threshold curves  
- **nstable** was defined as the smallest n for which:  
  \[
  \frac{dR}{dn} < 0.005
  \]
- The **99th percentile threshold** stabilized near **26 logins**

**All subsequent analyses were restricted to students with ≥26 non-class-day logins per term.**

---

## Statistical Analysis

### Group Differences in R

Normality was evaluated using:

- **Shapiro–Wilk** (n < 5000)  
- **D’Agostino–Pearson** (n ≥ 5000)

Group effects for semester, sex, and chronotype were assessed using:

- **Kruskal–Wallis tests**  
- **Pairwise Mann–Whitney U tests** with FDR correction  

Effect sizes:

- **ε²** for Kruskal–Wallis  
- **|r|** for Mann–Whitney U  

### Relationship Between R, Social Jetlag, and GPA

GPA was modeled as a function of **mean-centered R (R_bar)** using **Generalized Estimating Equations (GEE)** to account for repeated measures.

Model components:

- Linear and quadratic terms: R, R²   
- Chronotype-specific models  
- LOESS visualization (span = 0.6) with bootstrap 95% confidence intervals   

Outputs:

- **3D prediction surfaces** (GPA ~ R × SJL)  
- **2D contour maps** showing GPA isolines  
- Chronotype-stratified analyses  
- Model comparison using **Quasi-Information Criterion (QIC)**  
- Effect sizes using **Zhang’s pseudo-R²** and correlation-based pseudo-R²  



