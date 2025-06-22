# Meta Data Extraction from Documents

## Objective

The objective of this project is to build an **AI/ML-based metadata extraction system** that identifies and extracts key fields from legal documents (`.docx`, `.png`, `.jpg`) **without using rule-based logic** like regex or templates.

The target is to generalize across varying layouts and content styles using Named Entity Recognition (NER).

### Target Fields to Extract

* `Agreement Start Date`
* `Agreement End Date`
* `Renewal Notice (Days)`
* `Party One`
* `Party Two`
* `Agreement Value`

---

## Dataset & Input Format

Documents are provided as `.docx` files or scanned images (`.png`, `.jpg`).
Ground truth is available in `train.csv` and `test.csv` for evaluation.

---

## Google Drive / Local Folder Structure

```
ml_document_extractor/
├── data/
│   ├── train/                      # Training documents (.docx/.png/.jpg)
│   ├── test/                       # Testing documents
│   ├── train.csv                   # Ground truth for training files
│   └── test.csv                    # Ground truth for test files
│
├── processed_data/
│   ├── train_data.json             # Preprocessed & normalized training samples
│   └── test_data.json              # Preprocessed & normalized test samples
│
├── spacy_train_data.spacy          # spaCy DocBin training file
├── spacy_test_data.spacy           # spaCy DocBin test file
│
├── 01_preprocess_data.py           # Raw text extraction + normalization
├── 02_prepare_for_finetuning.py    # Fuzzy span matching + spaCy data formatting
├── 03_fineTune_predict_and_evaluate.ipynb  # spaCy NER fine-tuning + evaluation
└── README.md
```

---

## Step-by-Step Pipeline

### 1. Preprocessing (Script: `01_preprocess_data.py`)

* Extracts text:

  * From `.docx` using `python-docx`
  * From images using `pytesseract`
* Normalizes:

  * Dates like `31.03.2022` → `31 March 2022`
  * Values like `Three Thousand` → `3000`
  * Durations like `five months` → `5`
  * Names → Proper casing
* Aligns context with labels from `train.csv` / `test.csv`
* Saves preprocessed JSONs.

**Output:**

* `processed_data/train_data.json`
* `processed_data/test_data.json`

---

### 2. Span Matching + Data Preparation (Script: `02_prepare_for_finetuning.py`)

* Loads JSONs from step 1.
* Performs **fuzzy string matching** to find label spans in context.
* Converts annotated text into **spaCy DocBin** format.
* Logs unmatched entities for debugging.

**Output:**

* `spacy_train_data.spacy`
* `spacy_test_data.spacy`

---

### 3. Model Fine-Tuning & Evaluation (Notebook: `03_fineTune_predict_and_evaluate.ipynb`)

* Fine-tunes a **blank English spaCy NER model**.
* Trains over 30 iterations using spaCy training loop.
* Evaluates on test set.

**Metrics:**

* Per-entity **Precision**, **Recall**, and **F1 Score**
* Match logic:

```text
Recall = True Matches / (True Matches + Misses)
```

**Output:**

* Trained model: `/kaggle/working/fine_tuned_model`
* Console: Prediction + Evaluation Report

---

## Sample Evaluation Output

```
📦 Loading fine-tuned model...
📂 Loading test data...

🔍 Evaluation Results:
✅ Precision: 1.00
✅ Recall:    1.00
✅ F1 Score:  1.00
📊 Number of Examples Evaluated: 4

📋 Entity-wise Breakdown:
* Agreement End Date: Precision=1.00, Recall=1.00, F1=1.00
```

---

## To Reproduce

1. Place `.docx` or `.png` files in `data/train/` and `data/test/`
2. Provide `train.csv` and `test.csv` with correct metadata
3. Run preprocessing and training:

```bash
python 01_preprocess_data.py
python 02_prepare_for_finetuning.py
```

4. Run the notebook:

```bash
03_fineTune_predict_and_evaluate.ipynb
```

---

## Optional: REST API

As an enhancement, wrap the final spaCy model in a **Flask** or **FastAPI** service
to deploy metadata extraction as a web endpoint.

---

## Summary of Key Decisions

| Step           | Decision & Reason                                               |
| -------------- | --------------------------------------------------------------- |
| Preprocessing  | Normalize both labels and context for consistent span detection |
| No Regex       | Fully ML-based, avoiding rule-based methods                     |
| Fuzzy Matching | Handles text span mismatches in training data                   |
| Evaluation     | Custom recall + spaCy metrics, reported per entity              |
| Modularity     | Split into well-documented steps and files                      |

---

## Scripts Provided

* `01_preprocess_data.py` : Text extraction + normalization
* `02_prepare_for_finetuning.py` : Fuzzy matching + DocBin conversion
* `03_fineTune_predict_and_evaluate.ipynb` : Training + evaluation pipeline

---

## Final Notes

* The pipeline is designed to avoid manual rules, focusing only on learning from labeled examples.
* It supports both `.docx` and image-based document formats.
* Ideal for legal document metadata extraction use cases.

---

**Maintainer:** Pratik Kumar
**Institute:** NIT Silchar
**Task Deadline:** 23rd June 2025
