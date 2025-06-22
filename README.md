```markdown
# 🧾 Meta Data Extraction from Documents

## 📌 Problem Statement

Build an **AI/ML system** to extract key metadata fields from documents, which can be either `.docx` files or scanned images (`.png`, `.jpg`)—without using any **rule-based or static logic (regex, templates, etc.)**. The system should generalize well across varying document formats and templates.

### Target Fields to Extract:
- Agreement Start Date
- Agreement End Date
- Renewal Notice (Days)
- Party One
- Party Two
- Agreement Value

---

## 🗂️ Folder Structure

```
ml_document_extractor/
├── data/
│   ├── train/                       # .docx and image files for training
│   ├── test/                        # .docx and image files for testing
│   ├── train.csv                    # Ground truth for training files
│   └── test.csv                     # Ground truth for test files
├── processed_data/
│   ├── train_data.json              # Preprocessed + normalized training data
│   └── test_data.json               # Preprocessed + normalized test data
├── spacy_train_data.spacy           # spaCy DocBin for training
├── spacy_test_data.spacy            # spaCy DocBin for testing
├── 01_preprocess_data.py            # Extracts and normalizes both context and labels
├── 02_prepare_for_finetuning.py     # Generates .spacy training and test data
├── 03_fineTune_predict_and_evaluate.ipynb  # Fine-tunes model and evaluates on test set
└── README.md                        # This file
```

---

## 🧠 Solution Overview

This project uses a **spaCy Named Entity Recognition (NER)** model to extract metadata fields as entities from document text. The model is trained on a small, domain-specific dataset by:

1. Extracting text from `.docx` and `.png/.jpg` documents.
2. Normalizing both the **context** and **ground truth labels** for format consistency.
3. Preparing training-ready data using fuzzy span matching.
4. Fine-tuning a spaCy NER model and evaluating performance.

> ❗ **Note:** No rule-based logic, such as regex patterns or template heuristics, is used. Normalization ensures consistent formatting only.

---

## ⚙️ Scripts and Workflow

### 🔹 01_preprocess_data.py

- Extracts raw text from `.docx` files using `python-docx`, and from images using `pytesseract`.
- Loads `train.csv` and `test.csv` to get ground truth metadata.
- Cleans and **normalizes both the context and label values**:
  - Dates → e.g., `31.03.2022` or `March 31st, 2022` → `31 March 2022`
  - Numeric → e.g., `3,000` or `Three Thousand` → `3000`, `3,000.00`, etc.
  - Names → Proper casing and whitespace
  - Duration → `five months` or `5 month(s)` → `5`

📤 Outputs:
- `processed_data/train_data.json`
- `processed_data/test_data.json`

---

### 🔹 02_prepare_for_finetuning.py

- Reads the preprocessed JSON files from Step 1.
- Uses **fuzzy string matching (with multiple variants)** to find entity spans in the context text.
- Converts data to **spaCy's DocBin format**, required for training.

📤 Outputs:
- `spacy_train_data.spacy`
- `spacy_test_data.spacy`

✔️ Also logs unmatched entities and their file names for debugging purposes.

---

### 🔹 03_fineTune_predict_and_evaluate.ipynb

- Loads training `.spacy` data and fine-tunes a **blank English spaCy model** using the NER component.
- Uses a dropout + optimizer loop for 30 iterations to minimize training loss.
- Evaluates the model on the test data `.spacy` file using spaCy's built-in evaluation.

📊 Evaluation Metrics:
- **Per Field Recall** as specified:
```
Recall = True Matches / (True Matches + Misses)
```
- Also reports Precision, F1, and a breakdown per entity label.

📤 Outputs:
- Trained model in: `/kaggle/working/fine_tuned_model`
- Printed predictions and scores from the test set

---

## ✅ Key Highlights

- ✅ No regex, rule-based, or heuristic logic.
- ✅ Clean normalization on both context and labels ensures matching is based purely on format consistency.
- ✅ Robust fuzzy span matching handles inconsistencies between label and text spans.
- ✅ Precision, Recall, F1 reported entity-wise, exactly as required.

---

## 📊 Sample Evaluation Output

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

## 🔁 To Reproduce

1. Place `.docx` and `.png/.jpg` files inside `data/train/` and `data/test/`.
2. Update `train.csv` and `test.csv` with correct filenames and ground truth.
3. Run the scripts in this order:
   ```bash
   python 01_preprocess_data.py
   python 02_prepare_for_finetuning.py
   ```
4. Open and run all cells in:
   ```
   03_fineTune_predict_and_evaluate.ipynb
   ```

---

## 🧪 Optional REST API

> As an optional step, you can wrap the final trained spaCy model into a Flask/FastAPI web service. This allows the metadata extraction system to be accessed as a RESTful API.

---
