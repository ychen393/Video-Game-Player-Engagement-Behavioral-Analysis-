# Predicting Player Retention on a Research Minecraft Server

## Overview

This project investigates **player retention on a research Minecraft server** by predicting whether a player will continue participating based on their past behavior. The dataset was provided by a **UBC computer science research team**, and the analysis supports their goal of improving **server resource allocation** and **player engagement strategies**.

Using supervised learning, we build and evaluate a **k-nearest neighbors (k-NN) classification model** to predict whether a player will continue to participate (`subscribe`).

---

## Research Question

**Can a player’s likelihood of continued participation be predicted from their past gameplay behavior and demographic information?**

---

## Dataset

* Source: UBC research Minecraft server
* File: `players.csv`
* Each row represents an individual player

### Key Variables

* **Response variable:**

  * `subscribe` (TRUE / FALSE) — whether the player continues participating

* **Predictor variables:**

  * `played_hours` — total hours played
  * `experience` — player experience level
  * `age` — player age
  * `gender` — self-reported gender

Unnecessary identifiers (e.g., `hashedEmail`, `individualId`, `organizationName`) were removed during preprocessing.

---

## Methods

### Data Preprocessing

* Removed irrelevant or missing identifier columns
* Converted categorical variables to factors
* Split data into **training and testing sets**
* Standardized numeric predictors using a **recipe-based workflow**

### Exploratory Data Analysis (EDA)

* Visualized distributions of age, played hours, experience, and gender
* Examined relationships between predictors and player retention

### Modeling

* Implemented a **k-nearest neighbors (k-NN) classifier**
* Tuned the number of neighbors (*k*) using **10-fold cross-validation**
* Selected the optimal *k* based on **classification accuracy**
* Integrated preprocessing and modeling using a unified workflow

### Model Evaluation

* Trained the final model on the full training dataset
* Evaluated performance on the unseen testing set
* Assessed results using **accuracy** and a **confusion matrix**

---

## Tools & Technologies

* **Language:** R
* **Libraries:**

  * `tidyverse`
  * `tidymodels`
  * `recipes`
  * `ggplot2`

---

## Results

* Cross-validation identified an optimal value of *k* for the k-NN classifier
* The final model demonstrated reasonable predictive accuracy on the testing set
* Results suggest that **played hours and experience** are informative predictors of continued participation

Detailed results, visualizations, and interpretations are provided in the final report.

---

## Project Structure

```
.
├── data/
│   └── players.csv
├── notebooks/
│   └── analysis.ipynb
├── project final report.html
└── README.md
```

---

## How to View the Report

Open the following file in any web browser:

```
project final report.html
```

This file contains the full analysis, visualizations, and discussion.

---

## Authors

* Catherine Zhao
* Hannah Zhao
* Bowen Zhu
* **Winnie Chen**

---

## Notes

This project was completed for **academic purposes** using real research data.
The dataset may not be publicly shareable outside this repository.


