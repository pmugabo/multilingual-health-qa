# Multilingual Health Question Answering in Low-Resource African Languages

**Competition:** [Zindi — Multilingual Health QA Challenge](https://zindi.africa/competitions/multilingual-health-question-answering-in-low-resource-african-languages)
**Best Zindi Score:** 0.266956 (Experiment 10)

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/pmugabo/multilingual-health-qa/blob/main/notebooks/HealthQA_Notebook1_EDA_Baseline.ipynb)
[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/pmugabo/multilingual-health-qa/blob/main/notebooks/HealthQA_Notebook2_Finetuning.ipynb)

## Project Overview

This project fine-tunes Google's multilingual T5-small (mT5-small) model to answer health-related questions on maternal, sexual, and reproductive health across five African languages: **English, Akan, Luganda, Kiswahili, and Amharic**.

The dataset comes from the [HASH project](https://hash.theacademy.co.ug/) , a crowdsourced collection of 29,815 health Q&A pairs validated by healthcare professionals across Uganda, Ghana, Ethiopia, and Kenya.


## Repository Structure

```
multilingual-health-qa/
├── notebooks/
│   ├── Mugabo_HealthQA_Notebook1_EDA_Baseline.ipynb       # EDA, preprocessing, zero-shot baseline
│   └── Mugabo_HealthQA_Notebook2_Finetuning.ipynb   # Fine-tuning pipeline, all 10 experiments
├── submissions/
│   ├── submission_exp1_zeroshot.csv
│   ├── submission_exp2_finetuned.csv
│   ├── submission_exp3_3epochs.csv
│   ├── submission_exp4_3epochs_lr1e4.csv
│   ├── submission_exp5_trainval.csv
│   ├── submission_exp6_5epochs.csv
│   ├── submission_exp7_greedy.csv
│   ├── submission_exp8_512tokens.csv
│   ├── submission_exp9_warmup500.csv
│   └── submission_exp10_beams6.csv
├── requirements.txt
└── README.md
```

> **Note:** The raw dataset (Train.csv, Val.csv, Test.csv) is not included in this repository. Download it from the [Zindi competition page](https://zindi.africa/competitions/multilingual-health-question-answering-in-low-resource-african-languages) after joining, then follow the data path instructions below.


## Dataset

| Split | Size | Description |
|---|---|---|
| Train | 29,815 | Question and answer pairs for fine-tuning |
| Validation | 6,686 | Question and answer pairs for evaluation |
| Test | 2,618 | Questions only and answers must be generated |

**Languages covered:**

| Subset | Language | Country |
|---|---|---|
| Eng_Uga | English | Uganda |
| Eng_Gha | English | Ghana |
| Aka_Gha | Akan | Ghana |
| Eng_Eth | English | Ethiopia |
| Lug_Uga | Luganda | Uganda |
| Eng_Ken | English | Kenya |
| Swa_Ken | Kiswahili | Kenya |
| Amh_Eth | Amharic | Ethiopia |


## Model

- **Base model:** `google/mt5-small` (300M parameters)
- **Framework:** HuggingFace Transformers
- **Task:** Sequence-to-sequence text generation
- **Training:** Google Colab T4 GPU / Kaggle T4 GPU


## Experiment Results 

|  | Description | Zindi Score |
|---|---|---|
| 1 | Zero-shot mt5-small (no fine-tuning) | 0.000926 |
| 2 | Fine-tuned, 1 epoch, lr=3e-4 | 0.232357 |
| 3 | Fine-tuned, 3 epochs, lr=3e-4 | 0.235209 |
| 4 | Fine-tuned, 3 epochs, lr=1e-4 | 0.226766 |
| 5 | Fine-tuned, Train+Val combined, 3 epochs, lr=3e-4 | 0.261357 |
| 6 | Train+Val combined, 5 epochs, beams=4 | 0.177526 |
| 7 | Train+Val combined, 5 epochs, greedy decoding | 0.179376 |
| 8 | Train+Val combined, 5 epochs, 512 max tokens | 0.177489 |
| 9 | Train+Val combined, 3 epochs, warmup=500, beams=4 | 0.262992 |
| 10 | Train+Val combined, 3 epochs, warmup=500, beams=6 | **0.266956 (best)** |


## How to Run

### Option 1; Google Colab (recommended)

1. Open `notebooks/Mugabo_HealthQA_Notebook1_EDA_Baseline.ipynb` in Google Colab
2. Set runtime to **T4 GPU**: Runtime → Change runtime type → T4 GPU
3. Mount Google Drive and upload `Train.csv`, `Val.csv`, `Test.csv`, `SampleSubmission.csv` to `/content/drive/MyDrive/health_qa_data/`
4. Run all cells in order to reproduce the EDA and zero-shot baseline (Experiment 1)
5. Then open `notebooks/Mugabo_HealthQA_Notebook2_Finetuning_FINAL.ipynb` and run Sections 0–4 for Experiment 2, or Section 6 for the final best model (Experiments 9 & 10)

### Option 2 ; Kaggle

1. Upload `Train.csv`, `Val.csv`, `Test.csv` as separate Kaggle datasets
2. Open `notebooks/Mugabo_HealthQA_Notebook2_Finetuning_FINAL.ipynb` in Kaggle
3. Enable GPU: Settings → Accelerator → GPU T4 x2
4. Update data paths to `/kaggle/input/datasets/<your-username>/<dataset-name>/<file>.csv`
5. Run all cells in order

### Data paths

```python
# Google Colab
DATA_PATH = '/content/drive/MyDrive/health_qa_data/'

# Kaggle
TRAIN_PATH = '/kaggle/input/datasets/<username>/train-csv/Train.csv'
VAL_PATH   = '/kaggle/input/datasets/<username>/val-csv/Val.csv'
TEST_PATH  = '/kaggle/input/datasets/<username>/test-csv/Test.csv'
```


## Preprocessing

1. Strip whitespace from inputs and outputs
2. Drop rows with null values
3. Add a language prefix to each input so the model knows which language to answer in:
   - `'answer in Akan: '` for Akan questions
   - `'answer in Amharic: '` for Amharic questions
   - `'answer in English: '` for English questions
   - `'answer in Luganda: '` for Luganda questions
   - `'answer in Kiswahili: '` for Kiswahili questions
4. Tokenize with max length 256 for both input and output