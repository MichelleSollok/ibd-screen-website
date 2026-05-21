
# ChronScreen

**ChronScreen** is an interactive web tool for designing and evaluating hierarchical screening pipelines for chronic inflammatory and autoimmune diseases.

It allows researchers and clinicians to combine multiple screening layers — such as genetic risk, family history, blood biomarkers, fecal biomarkers, omics panels, and imaging — and evaluate how different pipeline designs affect:

- Positive predictive value, PPV
- Sensitivity
- Final cohort size
- Cost per detected case
- Enrichment over background risk
- Equivalent single-test AUC
- Population flow through each screening stage

Available at: [https://ibdscreen.org](https://ibdscreen.org)

---

## Purpose

Many disease screening strategies are not based on a single test. Instead, they use a sequence of increasingly specific or expensive tests. For example, a low-cost population-level risk score may be followed by blood testing, fecal biomarkers, imaging, or endoscopy.

ChronScreen helps users explore these questions:

- How many people remain after each screening stage?
- How many true cases are retained?
- How many false positives are generated?
- What PPV can be achieved?
- What sensitivity is lost at each stage?
- How much would the pipeline cost?
- Could the same performance realistically be achieved by a single test?

The tool is intended for research, planning, teaching, and hypothesis generation. It is not a validated clinical decision-support system.

---

## Quick Start

1. Open [https://ibdscreen.org](https://ibdscreen.org).
2. Set the starting **Population** size.
3. Set the disease **Risk** or prevalence.
4. Add screening blocks from the left panel.
5. For each block, adjust:
   - **AUC**
   - **Selected cohort percentage**
   - **Cost**
   - Whether the test is routinely available
6. Review the real-time results in the right panel.
7. Open the **Dashboard** for additional visualizations.
8. Use **Share** to copy a URL containing the current pipeline.
9. Use **Config** to export or import a pipeline as JSON.

---

## Interface Overview

ChronScreen has three main areas.

### 1. Left panel: Screening blocks and presets

The left panel contains available screening layers. Examples include:

| Block | Description |
|---|---|
| Family History | Risk based on affected relatives or known familial burden |
| Polygenic Risk Score | Genetic risk prediction based on common variants |
| Demographics + History | Age, sex, symptoms, medical history, or questionnaire-based risk |
| Bloodwork | Blood-based biomarkers or routine laboratory measurements |
| Fecal Calprotectin | Fecal inflammatory marker commonly used in IBD evaluation |
| Omics Panel | Multi-marker molecular profiling |
| Imaging Biomarkers | Imaging-derived risk or diagnostic markers |
| Environment | Lifestyle, exposure, geography, or environmental risk |
| Custom Block | User-defined screening layer |

Blocks can be clicked or dragged into the pipeline.

The panel also contains example presets:

- **Default**
- **Cost optimized**
- **No expense spared**

These presets provide starting points for comparing different screening strategies.

---

### 2. Center panel: Pipeline builder

The center panel shows the screening pipeline from top to bottom.

The starting population appears at the top. Each screening layer filters the population before passing individuals to the next layer.

For each block, the user can adjust:

#### AUC

The discriminatory performance of the screening layer.

- `0.50` means no discrimination.
- `0.70–0.80` indicates moderate discrimination.
- `0.80–0.90` indicates strong discrimination.
- `>0.90` indicates very strong discrimination.

ChronScreen currently allows AUC values from approximately `0.55` to `1.00`.

#### Selected cohort

The percentage of the current cohort that passes to the next layer.

For example, if a layer has **Selected cohort = 20%**, then the top 20% of the current cohort, according to that layer's risk score, is retained.

This value controls the strictness of the screening threshold.

#### Cost

The per-person cost of applying the screening layer.

If a test is marked as **Routine**, its additional cost is treated as zero in the cost analysis.

#### Routine

A routine test is assumed to already be available or already performed, so it does not contribute additional screening cost.

---

### 3. Right panel: Results

The right panel updates in real time as the pipeline changes.

#### Final Cohort

| Metric | Meaning |
|---|---|
| PPV | Among people flagged by the pipeline, the percentage who are true cases |
| Sensitivity | Percentage of all true cases in the original population retained by the pipeline |
| Cohort Size | Number of people remaining after all screening stages |
| Enrichment | PPV divided by background prevalence |

#### Cost Analysis

| Metric | Meaning |
|---|---|
| Total Cost | Total cost of all non-routine tests applied across the pipeline |
| Cost/Case | Total cost divided by the number of true cases detected |

#### Single-Test Equivalent

This section estimates what AUC a single standalone test would need in order to match the combined sensitivity and specificity of the full pipeline.

This helps illustrate whether a multi-stage strategy achieves performance that would be difficult or unrealistic for one biomarker alone.

#### Cumulative Confusion Matrix

A waffle plot shows the final distribution of:

- True positives
- False positives
- False negatives
- True negatives

#### Population Flow

The population flow chart shows how true positives and false positives move through each screening stage.

#### Layer Performance

This section summarizes the retention and filtering effect of each pipeline layer.

---

## Example Workflow

Suppose we want to evaluate a staged screening strategy for Crohn's disease in a population of 100,000 people with 1% disease risk.

A possible pipeline could be:

1. Family history
2. Polygenic risk score
3. Fecal calprotectin

Example settings:

| Layer | AUC | Selected cohort | Cost | Routine |
|---|---:|---:|---:|---|
| Family history | 0.70 | 100% | €5 | Yes |
| Polygenic risk score | 0.75 | 30% | €50 | Yes |
| Fecal calprotectin | 0.85 | 15% | €25 | No |

ChronScreen will estimate:

- How many people pass each stage
- How many expected disease cases are retained
- Final PPV
- Final sensitivity
- Cost per detected case
- Equivalent single-test AUC

---

## Model Summary

ChronScreen uses a binormal signal detection model.

For each screening layer:

- Controls are modeled as normally distributed scores.
- Cases are modeled as normally distributed scores with the same variance but shifted mean.
- The distance between the two distributions is represented by `d′`.
- `d′` is derived from the user-specified AUC.

The relationship is:

`d′ = sqrt(2) × Φ⁻¹(AUC)`

where `Φ⁻¹` is the inverse standard normal cumulative distribution function.

For each layer, ChronScreen finds a threshold that selects the top `q%` of the current cohort, where `q` is the selected cohort percentage.

The threshold determines:

- Layer-specific sensitivity
- Layer-specific false positive rate
- Layer-specific specificity
- The number of cases and non-cases passed to the next layer

The disease prevalence is updated after each layer because the retained cohort becomes enriched for cases.

---

## Monte Carlo Simulation

ChronScreen uses Monte Carlo simulation to estimate uncertainty.

The number of simulations can be adjusted in the header.

For each simulation, individuals are passed through the pipeline using the estimated sensitivity and false positive rate for each layer.

The tool reports approximate 95% confidence intervals for:

- PPV
- Sensitivity

Increasing the number of simulations gives more stable estimates but may make the interface slower.

---

## Correlation Between Screening Layers

Screening layers may not be independent.

For example, a genetic risk score and family history may be correlated. Similarly, inflammatory blood markers and fecal calprotectin may capture overlapping biological signals.

ChronScreen includes a layer-correlation matrix where users can specify correlations between screening layers.

Correlation values range from:

- `0`: independent layers
- `1`: fully correlated layers

Higher correlation generally reduces the added value of combining layers, because later tests provide less independent information.

Correlation modeling should be interpreted cautiously and used mainly for sensitivity analysis.

---

## Configuration Import and Export

The **Config** button allows users to copy or paste a JSON representation of the current pipeline.

Example configuration:

```json
{
  "population": 100000,
  "prevalence": 1,
  "correlations": {},
  "pipeline": [
    {
      "type": "genetic",
      "name": "Polygenic Risk Score",
      "auc": 0.75,
      "cohortPct": 30,
      "cost": 50,
      "routinelyAvailable": true
    },
    {
      "type": "fecal",
      "name": "Fecal Calprotectin",
      "auc": 0.85,
      "cohortPct": 15,
      "cost": 25,
      "routinelyAvailable": false
    }
  ]
}
```

### Configuration fields

| Field | Meaning |
|---|---|
| population | Starting population size |
| prevalence | Disease risk in percent |
| correlations | Pairwise layer correlations |
| pipeline | Ordered list of screening layers |
| type | Block type |
| name | Display name |
| auc | AUC of the screening layer |
| cohortPct | Percentage of the current cohort retained |
| cost | Per-person cost |
| routinelyAvailable | Whether the test is treated as routine and cost-free |

---

## Sharing Pipelines

The **Share** button creates a URL containing the current pipeline settings.

This can be used to:

- Share screening scenarios with collaborators
- Save model configurations
- Include reproducible examples in manuscripts or presentations

---

## Dashboard

The **Dashboard** opens a more detailed visualization page using the current pipeline data.

The dashboard may include:

- Detailed population flow
- Layer-by-layer performance
- Cost breakdown
- Comparative visualizations
- Summary metrics for publication or presentation

---

## Trial Design

The **Trial Design** page provides additional tools for translating screening scenarios into study or trial planning contexts.

This section should be expanded with details once the trial design functionality is finalized.

---

## Interpretation Guidance

### High PPV does not necessarily mean good screening performance

A very strict pipeline may produce a high PPV but detect only a small fraction of true cases.

Always consider PPV together with sensitivity.

### High sensitivity may come at the cost of many false positives

A less restrictive pipeline may retain more true cases but also produce a larger follow-up cohort.

### Cost per case depends strongly on where expensive tests are placed

Expensive tests are usually more efficient when applied later in the pipeline, after cheaper layers have reduced the cohort size.

### Correlated layers provide less incremental value

If two tests capture similar information, the second test may not improve performance as much as expected under independence.

---

## Limitations

ChronScreen is a simplified modeling tool. Results should be interpreted as exploratory estimates.

Important limitations include:

- The model assumes binormal score distributions.
- AUC is treated as known and fixed.
- Real-world test calibration is not modeled.
- Operational constraints are not included.
- Downstream clinical consequences are not modeled.
- Correlation modeling is approximate.
- Costs are simplified and may not reflect full healthcare-system costs.
- The tool does not validate whether a proposed pipeline is clinically appropriate.

ChronScreen should not be used as a clinical diagnostic tool.

---

## Recommended Reporting

When reporting a ChronScreen scenario, include:

1. Population size
2. Baseline disease risk or prevalence
3. Number of simulations
4. Screening layers in order
5. AUC for each layer
6. Selected cohort percentage for each layer
7. Cost assumptions
8. Routine/non-routine status
9. Correlation assumptions
10. Final PPV
11. Final sensitivity
12. Final cohort size
13. Cost per detected case
14. Share URL or exported JSON configuration

---

## Citation

Sazonovs Lab, PREDICT, Aalborg University Copenhagen.

If used in academic work, please cite the associated manuscript or repository once available.

---

## Development Notes

The web application is currently implemented as a client-side browser tool.

Main features include:

- Drag-and-drop pipeline construction
- Real-time screening metric calculations
- Monte Carlo simulation
- Correlation matrix between screening layers
- Cost analysis
- JSON import/export
- Shareable URLs
- Dashboard integration
- Trial design page

---

## Suggested Future Documentation Additions

The documentation should eventually include:

- A full mathematical derivation
- Validation examples against simulated datasets
- Worked examples for Crohn's disease, rheumatoid arthritis, and type 1 diabetes
- Screenshots of the interface
- Detailed dashboard documentation
- Trial design documentation
- Version history
- Known issues
- API or developer notes if the model is separated into reusable code
