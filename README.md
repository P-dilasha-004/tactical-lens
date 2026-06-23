# tactical-lens


## 1. Comparative Performance

This project evaluated two distinct approaches to classifying Reddit comments: a Zero-Shot LLM (baseline) and a fine-tuned DistilBERT model.

### Overall Accuracy

- **Baseline (Zero-Shot LLM):** 0.533
- **Fine-Tuned Model (DistilBERT):** 0.733

### Per-Class Metrics Comparison

| Label | Baseline Precision | Baseline Recall | Baseline F1 | Fine-Tuned Precision | Fine-Tuned Recall | Fine-Tuned F1 |
|---------|---------|---------|---------|---------|---------|---------|
| Substantive | 0.75 | 0.30 | 0.43 | 0.79 | 0.73 | 0.76 |
| Partisan | 0.58 | 0.70 | 0.64 | 0.82 | 0.64 | 0.72 |
| Low-Effort | 0.43 | 0.60 | 0.50 | 0.65 | 0.81 | 0.72 |

---

## 2. Confusion Matrix (Fine-Tuned Model)

[View Image](confusion_matrix.png)

---

## 3. Failure Analysis

While the fine-tuned model achieved an accuracy of **0.733**, it still struggles with specific linguistic boundaries.

### Analysis of Misclassifications

#### Example 1

> "Jordan looked so dead the 2nd half, was inevitable."

**True Label:** Substantive  
**Predicted Label:** Low-Effort

**Analysis:**  
This demonstrates a **Brevity Bias**. The model equates short length with low quality. The tactical observation ("looked so dead") was ignored because the comment lacked the length typically associated with substantive analysis in the training set.

**Fix:**  
Include more short, concise analytical sentences in the training data to teach the model that length is not a requirement for substance.

---

#### Example 2

> "Agreed. Haaland is a fucking monster"

**True Label:** Partisan  
**Predicted Label:** Low-Effort

**Analysis:**  
This is a **Sentiment Trap**. The model detects the profanity ("fucking") and the brevity, triggering a classification of Low-Effort/Noise. It fails to identify the Partisan intent—praising a player in a way that signals emotional bias or fandom.

**Fix:**  
Expand the Partisan label definition to explicitly include high-sentiment player praise. The current definition focuses too heavily on hostile bias.

---

#### Example 3

> "48 is too many for me, the Qatar v Canada game was deplorable. Maybe 35 teams?"

**True Label:** Substantive  
**Predicted Label:** Partisan

**Analysis:**  
This is a **Semantic Mimicry** error. The model correctly identifies the substantive content (a debate about tournament structure), but gets distracted by the mention of a specific match ("Qatar v Canada"). Because match-specific complaints are often Partisan, the model incorrectly shifts the prediction toward that label.

**Fix:**  
Add more examples of substantive opinions that mention specific matches so the model learns that referencing a match does not automatically imply partisan sentiment.

---

## 4. Sample Classifications

| Post Text | Predicted Label | Confidence | Reasoning |
|------------|----------------|------------|------------|
| "The high press from the midfield disrupted transition play." | Substantive | 0.98 | Correct: Uses tactical lexicon such as "high press" and "transition play." |
| "We are the best country in the world, trophy coming home!" | Partisan | 0.94 | Correct: Tribal, nationalistic, and emotionally charged language. |
| "Bruh 😂" | Low-Effort | 0.99 | Correct: Reactionary comment with no informational content. |

---

## 5. Reflections & AI Usage

### Gap Analysis

The model captured the vocabulary of the community effectively but struggled with intent. It learned that tactical terminology often signals substantive discussion, but it also overfit to comment length as a proxy for quality. As a result, it sometimes failed to recognize that a comment can be both brief and analytically meaningful.

### Specification Reflection

The specification's requirement to document a failure analysis proved valuable because it required examination of individual errors rather than relying solely on aggregate metrics. This process revealed the model's Brevity Bias and highlighted opportunities for targeted dataset improvements.

I also diverged from the original project plan by expanding the dataset to **239 labeled examples**. The initial dataset of 139 examples led to overfitting and poor generalization. Increasing the dataset size through carefully reviewed synthetic examples was a necessary data-centric adjustment that helped the model surpass the 70% accuracy target.

# Training Iteration & Epoch Optimization

To address the performance ceiling encountered during initial experimentation, I performed a systematic hyperparameter sweep of the training duration.

## Underfitting Phase

The initial baseline run utilized **3 epochs**, which proved insufficient for the model to capture the complex linguistic nuances of the target labels. This resulted in **high training and validation loss**, indicating underfitting.

## Overfitting Phase

A subsequent attempt at **25 epochs** led to model divergence. This was characterized by:

- Steadily decreasing training loss  
- Increasing validation loss  

This pattern is a clear signal of **overfitting**, where the model begins to memorize noise in the training data rather than learning generalizable patterns.

## Convergence Phase

The model was optimized at **15 epochs** by closely monitoring the validation loss trajectory. This configuration:

- Achieved stable validation loss  
- Mitigated overfitting risk  
- Improved generalization performance  

This adjustment acted as a critical inflection point in the training pipeline, improving final test-set accuracy from **63.3% → 73.3%**.

### AI Usage

#### Failure Analysis

I provided a set of 12 misclassified comments to an LLM and used its observations to identify recurring error patterns. The model suggested themes such as **Brevity Bias** and **Sentiment Trap**, which I then manually validated by reviewing the underlying examples. This process helped distinguish systematic weaknesses from random classification errors.

#### Data Augmentation

I used an LLM to generate 100 synthetic training examples distributed across the three labels. Every generated example was manually reviewed before inclusion in the dataset. Ambiguous examples or examples that did not align with the project's labeling guidelines were modified or discarded. This human review process ensured that AI-generated data supported the labeling framework rather than introducing additional noise.