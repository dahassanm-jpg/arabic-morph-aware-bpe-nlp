# Evaluating Morphology-Aware Tokenization for Arabic NLP and its Impact on Retrieval Systems

Student: NLP_Project_Dalya_222053121_Arabic_Morph.ipynb  
Student ID: 220253121  
Course: Natural Language Processing (Graduate Course)  
Instructor: Prof. Aiman Ahmed Abusamra  
Program: Master of Computer Engineering  
Institution: Islamic University of Gaza  
Date: June 2026  

## Overview

This project evaluates whether morphology-aware tokenization improves Arabic NLP performance compared with standard Byte Pair Encoding (BPE).

The project compares:

- Standard BPE trained on normalized Arabic text
- CAMeL-based morphology-aware BPE trained after Arabic morphological segmentation

The experiments evaluate the effect of tokenization on:

1. Arabic sentiment classification
2. Arabic candidate-pool retrieval

## Method

### Tokenization

- Baseline: Standard BPE
- Proposed method: CAMeL morphological segmentation followed by BPE training

### Classification

- Dataset: Arabic sentiment/classification dataset
- Features: TF-IDF over BPE subwords
- Classifier: Logistic Regression
- Metrics: Accuracy and Macro-F1
- Statistical testing: paired tests across seeds and McNemar test

### Retrieval

- Dataset: MIRACL-Arabic candidate-pool retrieval setting
- Retrieval model: BM25 over BPE subwords
- Primary metric: nDCG@10
- Secondary metrics: MRR, MAP, Precision@10, Hit@10

## How to Run

Install Python dependencies:

```bash
pip install -r requirements.txt
