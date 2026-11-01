# Baseline classifier (TF-IDF + Logistic Regression)

The first version of CallSense used a lightweight classifier for **intent** (6 classes) and **escalation** (binary). It was replaced by an LLM-based classifier (Qwen2.5-7B-Instruct with a scope-first prompt) after live tests. It is kept here as a documented baseline.

## Files

| File | Description |
|---|---|
| `CallSense_Classifier_TFIDF.ipynb` | Training and evaluation notebook (outputs included) |
| `callsense_classifier.joblib` | Saved models (requires `scikit-learn==1.6.1`) |

## Method

- Features: character n-gram **TF-IDF** (`char_wb`, n = 2 to 5, sublinear tf).
- Models: two `LogisticRegression` pipelines (`C=10`, `class_weight="balanced"`): one for intent and one for escalation.
- Escalation decision threshold tuned to **0.4** (default 0.5).
- Compared against a fine-tuned **CAMeLBERT** model using the same 5-fold protocol.

## Results (5-fold cross-validation on the 80 labeled calls)

| Model | Intent accuracy | Intent macro-F1 | Escalation F1 |
|---|---:|---:|---:|
| TF-IDF + LR (threshold 0.5) | 0.950 | 0.950 | 0.706 |
| TF-IDF + LR (tuned threshold) | 0.950 | 0.950 | 0.837 (P 0.78, R 0.90) |
| CAMeLBERT (fine-tuned) | 0.800 | 0.796 | 0.722 |

## Why it was replaced

Cross-validation looked strong, but the data comes from one source with similar vocabulary. When the team tested the deployed app with **new recordings** (new speakers, new wording, noisy transcripts), complaint calls were labeled as `information` with about 28% confidence. This is a classic **distribution shift / generalization gap**: a high in-distribution score on a small dataset does not guarantee behavior on unseen calls. The final system uses an LLM with a scope check, and keeps the retrieved labeled calls as few-shot examples.

## Load it

```python
import joblib

bundle = joblib.load("callsense_classifier.joblib")   # scikit-learn==1.6.1
labels = bundle["label_names"]                        # ['billing','information','cancellation','complaint','technical','out_of_scope']
text = "ابغى ألغي اشتراكي لأن الإنترنت يفصل كل يوم"

probs = bundle["intent_model"].predict_proba([text])[0]
intent = labels[int(probs.argmax())]
escalate = bundle["escalation_model"].predict_proba([text])[0][1] >= bundle["escalation_threshold"]
print(intent, escalate)
```
