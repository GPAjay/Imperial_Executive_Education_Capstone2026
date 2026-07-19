# Imperial_Executive_Education_Capstone2026
Final capstone project submission towards the Imperial Business Executive Course in Professional Certification in Machine Learning and Artificial Intelligence

# 🚀 Bayesian Black-Box Optimisation — Capstone Project

![Status](https://img.shields.io/badge/Status-COMPLETE-brightgreen)
![Python](https://img.shields.io/badge/Python-3.8%2B-blue)
![Type](https://img.shields.io/badge/Type-Capstone-green)
![Modules](https://img.shields.io/badge/Modules-12--24-purple)
![Week](https://img.shields.io/badge/All%2013%20Weeks-Done-brightgreen)

## 📋 Documentation

| Document | Description |
|----------|-------------|
| [DATASHEET.md](BBO_datasheet.md) | Dataset documentation — composition, collection process, preprocessing and intended uses |
| [MODEL_CARD.md](BBO_Modelcard.md) | Model card — GP-BBO approach, performance and limitations, final Per-Function results summary, lessons learned |
| [Strategy_Approach.md](BBO_Strategy_&_Approach.md) | Core method, strategy evolution, pipeline, Key Techniques, technologies Used |

---
## 📄 Overview

This project is the Capstone requirement for the **Imperial College AI/ML Programme**, running from Module 12 through to Module 24.

The challenge involves optimising **8 unknown black-box functions** of increasing dimensionality (2D to 8D) using a limited number of weekly queries. Each week, one new input point per function is submitted via the capstone portal, and the resulting output is used to refine the search strategy.

The goal is to find the input that **maximises** the output of each function through intelligent, iterative, data-driven decisions — reflecting how optimisation is approached in real-world ML research and industry applications such as hyperparameter tuning, drug discovery, and industrial process optimisation.

### 💼 Career Relevance

This project directly builds skills needed for data science and ML engineering roles — making decisions under uncertainty, working with limited and expensive-to-acquire data, and iterating strategies based on evidence. These competencies apply to model tuning, A/B testing, experimental design, and any scenario where complete knowledge of the system is unavailable.

---

## ❓ Problem Statement

Eight synthetic black-box functions are provided, each simulating a real-world optimisation challenge such as radiation detection, drug discovery, or hyperparameter tuning. The internal equations are unknown — only inputs and outputs are observable.

Each function must be **maximised** using limited weekly queries, making smart search strategy essential.

---

## 🎯 Project Goals

1. **Explore** unknown function landscapes intelligently
2. **Exploit** promising regions as data grows each week
3. **Iterate** and refine strategy based on weekly results
4. **Reflect** on approach and document decision-making throughout

## 📊 Functions Overview
| Function | Dimensions | Description | Application |
|----------|-----------|-------------|-------------|
| F1 | 2D | Radiation Detection | Identify contamination sources in a 2D field |
| F2 | 2D | Noisy ML Model | Maximise log-likelihood with noisy outputs |
| F3 | 3D | Drug Discovery | Minimise side effects (negated for maximisation) |
| F4 | 4D | Warehouse Placement | Optimise product placement across warehouses |
| F5 | 4D | Chemical Yield | Maximise yield of a chemical process (unimodal) |
| F6 | 5D | Cake Recipe Optimisation | Maximise recipe score (negative by design, push toward zero) |
| F7 | 6D | ML Hyperparameter Tuning | Maximise model accuracy/F1 score |
| F8 | 8D | Complex 8D Optimisation | Maximise validation accuracy across 8 hyperparameters |

## 📥 Inputs and Outputs

## 🗓️ Weekly Progress

## 📈 Week-by-Week outcomes

### Week 1 (Bayesian Optimisation)

### Week 2 (Logistic Regression)

### Week 3 (Support Vector Machines)

### Week 4 (Neural Networks and Deep Learning: Part One: Introduction)

### Week 5 (Neural Networks and Deep Learning: Part Two: Advanced Concepts)

### Week 6 (Module 17: Neural Networks and Deep Learning: Part Three: Convolutional Neural Networks)

### Week 7 (Hyperparameters and Hyperparameter Tuning)

### Week 8 (Foundations of Generative AI and Large Language Models (LLMs)

### Week 9 (Advanced Gen AI and LLMs)

### Week 10 (Transparency and Interpretability)

### Week 11 (Unsupervised Learning: Part One: Clustering Techniques)

### Week 12 (Unsupervised Learning: Part Two: Principal Component Analysis)

### Week 13 (Reinforcement Learning)

## 🏆 All-Time Best Results — FINAL

### 🆕 Week 4 Improvements

### 🆕 Week 5–6 Improvements

### 🆕 Week 7–8 Improvements

### 🆕 Week 9 Improvements

### 🆕 Week 10–11 Improvements


## 📁 Repository Structure


```
imperial-aiml-capstone/
│
├── README.md
├── REFERENCES.md
│
├── data/
│   ├── function_1/   (initial_inputs.npy, initial_outputs.npy)
│   ├── function_2/
│   └── ... (function_3 through function_8)
│
├── module-12/          ← Week 1: Broad GP+UCB exploration
│   ├── notebooks/
│   │   └── Module_12_Bayesian_Optimisation_Capstone.ipynb
│   └── plots/
│
├── module-13/          ← Week 2: UCB with log transforms
│   ├── notebooks/
│   │   └── Module_13_Week2_Capstone.ipynb
│   └── plots/
│
├── module-14/          ← Week 3: SVM filter (failed)
│   ├── notebooks/
│   │   └── Module_14_Week3_Capstone.ipynb
│   └── plots/
│
├── module-15/          ← Week 4: Trust regions + EI ensemble
│   ├── notebooks/
│   │   └── Module_15_Week4_Capstone.ipynb
│   └── plots/
│
├── module-16/          ← Week 5: Anchored trust regions
│   ├── notebooks/
│   │   └── Module_16_Week5_Capstone.ipynb
│   └── plots/
│
├── module-17/          ← Week 6: Per-function beta tuning (F1, F2, F5 new bests)
│   ├── notebooks/
│   │   └── Module_17_Week6_Capstone.ipynb
│   └── plots/
│
├── module-18/          ← Week 7: Tighter radii, lower beta
│   ├── notebooks/
│   │   └── Module_18_Week7_Capstone.ipynb
│   └── plots/
│
├── module-19/          ← Week 8: F5, F6, F7 all-time bests
│   ├── notebooks/
│   │   └── Module_19_Week8_Capstone.ipynb
│   └── plots/
│
├── module-20/          ← Week 9: F1, F5 all-time bests; F7 momentum
│   ├── notebooks/
│   │   └── Module_20_Week9_Capstone.ipynb
│   └── plots/
│
├── module-21/          ← Week 10: F7, F8 all-time bests
│   ├── notebooks/
│   │   └── Module_21_Week10_Capstone.ipynb
│   └── plots/
│
├── module-22/          ← Week 11: F7 (3.1034), F8 (9.9750) all-time bests
│   ├── notebooks/
│   │   └── Module_22_Week11_Capstone.ipynb
│   └── plots/
│
├── module-23/          ← Week 12: F8 (9.9799) all-time best
│   ├── notebooks/
│   │   └── Module_23_Week12_Capstone.ipynb
│   └── plots/
│
└── module-24/          ← Week 13 FINAL: F1 (0.33641), F5 (4039.88), F7 (3.1050) NEW BESTS
    ├── notebooks/
    │   └── Module_24_Week13_Capstone.ipynb
    └── plots/
```

---
## 💻 How to Run

## 📬 Contact

**Ajay Kumar Ginka Panasa** — Imperial College AI/ML Programme

🔗 [GitHub Profile](https://github.com/GPAjay/Imperial_Executive_Education_Capstone2026)















