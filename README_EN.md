# Predicting Player Retention on a Research Minecraft Server
## Exploring continued participation from historical player data

[简体中文](README.md) · English

This project uses player data supplied by a UBC computer science research team to explore an operational question: **Can past player information help identify who will continue participating on a research server?**

We build a k-nearest neighbors (k-NN) classification baseline using play hours, experience level, age, and gender to predict `subscribe` (whether a player continues to subscribe/participate). The analysis covers data cleaning, exploratory plots, feature standardization, a stratified train/test split, 10-fold cross-validation for tuning, and confusion-matrix evaluation.

> **Main result:** Cross-validation selected (k=26), and test accuracy was **73.47%**. However, the confusion matrix shows that the model predicted TRUE for all 49 test players: 36 TRUE cases were correct, while all 13 FALSE cases were missed. Since TRUE already accounts for 36/49 = **73.47%** of the test set, the model matches an “always predict the majority class” baseline; Cohen’s (kappa=0). The key takeaway is that **accuracy alone overstates performance: this k-NN baseline did not identify non-subscribing players and is not suitable for real churn detection.**

## Research question and context

The research team wanted to understand whether players would continue participating, with the broader goal of informing server-resource planning and future recruitment. We frame this as a binary classification task:

> Given a player's historical participation and basic profile, can a model predict whether `subscribe` is TRUE or FALSE?

The practical value depends on identifying the group of interest. If the goal is to find players who may stop participating, a model that labels most people as “will continue” cannot support targeted recruitment or retention actions even if its overall accuracy looks reasonable. That is why the project inspects the confusion matrix alongside accuracy.

## Data

- **Source:** Research Minecraft server data supplied by a UBC computer science research team.
- **Sample:** 196 player records; 9 columns in the original file.
- **Target:** `subscribe`, TRUE / FALSE.
- **Features:** `played_hours` (total hours played), `experience` (experience level), `age`, and `gender` (self-reported).
- **Preprocessing:** Remove names, hashed email, and empty identifier fields; convert categorical fields to factors; impute missing numeric values with each column's median.
- **Class balance:** 144 TRUE and 52 FALSE records overall; TRUE represents 73.47%.

The repository currently contains the analysis notebook, not the raw `players.csv`. The notebook reads the source data through an external link supplied by the research team; access and redistribution are controlled by the data provider. Do not infer player identities from notebook examples or outputs.

## Analysis workflow

1. **Clean and select fields:** Keep participation status, play hours, experience, age, and gender; remove names, hashed emails, and empty identifiers.
2. **Explore the data:** Plot play hours against age and compare TRUE/FALSE counts across experience and gender categories.
3. **Create a stratified split:** Use random seed `2000` and stratify on `subscribe`; allocate 75% to training and 25% to testing. The test set is held out during model selection.
4. **Standardize numeric features:** Scale and center numeric predictors in a tidymodels recipe so that differences in units do not dominate distance calculations.
5. **Tune with cross-validation:** Run 10-fold stratified cross-validation on the training set; compare mean accuracy across candidate neighbor counts such as (k=1,6,11,ldots,96).
6. **Evaluate on the test set:** Fit the final k-NN model with the selected $k$, compute accuracy and Cohen’s (kappa), and inspect the confusion matrix.

### Model setup

| Item | Configuration |
| --- | --- |
| Algorithm | k-Nearest Neighbors classifier (k-NN) |
| Predictors | `played_hours`, `experience`, `age`, `gender` |
| Target | `subscribe` (TRUE / FALSE) |
| Numeric preprocessing | Training recipe: scale + center |
| Split | 75% train / 25% test; stratified on `subscribe`; seed = 2000 |
| Tuning | 10-fold stratified cross-validation on training data |
| Selection metric | Mean cross-validation accuracy |
| Selected candidate | (k=26) |

## Results: why accuracy is misleading here

The notebook records a cross-validation mean accuracy of **73.50%** at (k=26); test accuracy is **73.47%**. Seen alone, these values could suggest that the model predicts continued participation reliably.

The test confusion matrix shows what it actually does:

| Actual \ Predicted | FALSE | TRUE |
| --- | ---: | ---: |
| FALSE | 0 | 13 |
| TRUE | 0 | 36 |

The model predicts TRUE for every test record and never predicts FALSE. Therefore:

- For the TRUE class: precision = (36/49) = **73.47%**; recall = (36/36) = **100%**.
- For the FALSE class: recall = (0/13) = **0%**.
- Cohen’s (kappa) = **0**, indicating no improvement over the chance-agreement baseline implied by the class distribution.
- A simple majority-class baseline that always predicts TRUE also gets (36/49 = 73.47%) accuracy on this test set.

The notebook reports an F1 score of approximately **0.85** for the TRUE class. That score does not mean the model handles both classes well: it completely misses FALSE. If the operational goal is to identify players who may stop participating, this model misses every such test case. The report’s original interpretation that recall = 1 means all likely churners are detected is incorrect: the positive class is `subscribe=TRUE`, so that recall measures TRUE-class detection, not FALSE/churn detection.

**Interpretation:** Accuracy-based tuning favored a $k$ value that performs well on the majority class, while the current features and k-NN decision boundary fail to identify the minority class. With only 196 records and 49 test cases, these findings should be treated as an exploratory course-project baseline, not a deployment result.

## Visualizations

The charts below are redrawn from the notebook's saved grouped summaries, cross-validation output, and test confusion matrix. The count chart is descriptive; the tuning chart includes the test-set majority-class baseline; and the confusion matrix shows the model's actual predictions.

### Participation labels by experience level

![TRUE and FALSE player counts across experience levels. Counts are descriptive and do not imply that experience causes participation.](assets/experience_participation.png)

TRUE counts exceed FALSE counts at each level in this sample. The figure shows counts, not subscription rates; group sizes differ, so it should not be read as causal evidence.

### k-NN tuning results

![Ten-fold cross-validation accuracy by k, with k=26 and the test-set majority-class baseline marked.](assets/knn_cv_accuracy.png)

As $k$ increases, cross-validation accuracy plateaus around 73.5%. The red dashed line is the test-set majority-class baseline of 73.47%. Their near overlap is a warning that high accuracy mostly reflects the prevalence of TRUE, rather than successful detection of FALSE players. Error bars show the standard error across the 10 folds.

### Held-out test confusion matrix

![Test-set confusion matrix: every case was predicted TRUE, including all 13 actual FALSE cases.](assets/test_confusion_matrix.png)

The model never predicts FALSE: all 13 of 13 actual FALSE test cases are missed. This is the key result when considering whether the model could identify players who may stop participating.

Open the [analysis notebook](https://github.com/ychen393/Video-Game-Player-Engagement-Behavioral-Analysis-/blob/main/project%20final%20report.ipynb) for source code, additional EDA plots, the complete cross-validation results, and model output.

## Conclusion and next steps

This model is useful as a k-NN baseline and as an example of an end-to-end classification workflow, but the test results do not establish effective player-retention prediction. The more meaningful finding is the mismatch between the metric and the operational goal: accuracy reaches 73.47%, yet equals the majority-class baseline, with 0% recall for the minority class.

A next iteration could:

1. Define the positive class around the operational target (for example, non-continuing players) and report minority-class recall, precision, specificity, balanced accuracy, and PR-AUC.
2. Try class weights or resampling within the training data while preserving an independent test set to avoid leakage.
3. Add behaviorally informative features such as login frequency, recency, consecutive active days, and research-task participation; the current dataset has only total play hours and a few profile fields.
4. Compare against a majority-class baseline, logistic regression, decision trees, and random forests; use repeated cross-validation to assess stability with a small sample.
5. Obtain more observations with the data provider’s permission and inspect error rates across age and gender groups so small-sample disparities are not hidden.

## Technology

- **Language:** R 4.3.3
- **Modeling and data:** tidyverse, tidymodels, recipes, kknn
- **Visualization:** ggplot2, RColorBrewer, GGally
- **Analysis format:** R Jupyter Notebook

## View and reproduce

### Open the full report

Open on GitHub:

[project final report.ipynb](https://github.com/ychen393/Video-Game-Player-Engagement-Behavioral-Analysis-/blob/main/project%20final%20report.ipynb)

The notebook includes written analysis, R code, plots, and saved execution outputs. To rerun it, use R 4.3.3 and install the R packages loaded in the notebook (including tidyverse, tidymodels, RColorBrewer, ggplot2, and GGally). Data access requires a valid source-data link and permission from the provider.

### Repository files

```text
README.md
project final report.ipynb
```

## Collaborators

Catherine Zhao · Hannah Zhao · Bowen Zhu · **Winnie Chen**

## Notes

This is a UBC course project using data supplied by a research team. Model output is exploratory and is not a definitive judgment about individual players or a retention model ready for operational deployment.
