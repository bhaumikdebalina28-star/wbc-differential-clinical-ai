# Clinical Validation Framework — WBC Differential Classifier

**Prepared as a demonstration exercise, in the style of a real clinical AI validation file.**

- Overall test accuracy: 56.1%
- Per-class precision/recall (copy from classification report above):  Eosinophil: Precision 52.3%, Recall 36.9%
  - Lymphocyte: Precision 70.4%, Recall 44.5%
  - Monocyte: Precision 64.1%, Recall 65.2%
  - Neutrophil: Precision 47.3%, Recall 77.7%

- Which classes get confused with which, and does it matter clinically:  Eosinophil↔Neutrophil is the biggest confusion (243 cases) — clinically expected, since
  both are granulocytes distinguished mainly by subtle granule color/texture. Monocyte↔Eosinophil
  confusion (109 cases) is more clinically surprising, since these cell types look structurally
  different — likely a model limitation (frozen ImageNet backbone) rather than genuine visual
  ambiguity. The model also shows a clear bias toward over-predicting Neutrophil overall
  (1,025 predictions vs. 624 true cases).

- Any signs of overfitting (train vs. validation gap): Based on the training curves, there are no obvious signs of overfitting. Both training and validation accuracy are increasing, and notably, the validation accuracy is often slightly higher or very close to the training accuracy. Similarly, the validation loss is consistently lower than the training loss. This pattern suggests the model might not be overfitting and could potentially benefit from more training epochs or a more complex architecture if higher performance is desired.

- What I'd do with more time/data:  Unfreeze the top few layers of the pretrained base and fine-tune at a low learning rate
  directly on the blood cell images, rather than only training the classification head.
  Would also try a backbone pretrained on histology/medical images instead of general photos,
  and validate across multiple labs/scanners/stain batches before trusting generalization.

---

## 1. Intended Use Statement

**Intended use:** This model is intended to assist in flagging the likely white blood cell
type (Eosinophil, Lymphocyte, Monocyte, Neutrophil) from a microscopy image of a peripheral
blood smear, for pathologist review.

**Explicitly NOT intended for:**
- Autonomous diagnosis without pathologist confirmation
- Detection of abnormal, atypical, or blast cells (out of scope for this exercise — a real
  deployment would need a separate, dedicated validation for abnormal cell detection, since
  the clinical stakes and required sensitivity are much higher)
- Use on samples from populations, stains, or scanners not represented in the training data

**Why this matters:** a model's intended use statement defines the boundary of what it's
allowed to be evaluated against. Overclaiming scope here is one of the most common mistakes
in early-stage clinical AI work.

---

## 2. Ground Truth Definition and Limitations

**How ground truth was defined in this exercise:** dataset labels from the public
Kaggle "Blood Cell Images" dataset (Paul Mooney), originally labeled by [FILL IN: check
dataset documentation for labeling methodology if available].

**Known limitation:** in a real clinical deployment, ground truth would need to come from
pathologist consensus (ideally 2–3 independent reviewers with a documented adjudication
process for disagreements), not a single label per image. Single-source labels risk baking
in one reviewer's biases or errors as if they were ground truth. This is a genuine limitation
of this exercise, not something to gloss over.

---

## 3. Evaluation Results

**Overall test accuracy:** 56.1% (1,395 correct out of 2,487 test images)

**Per-class precision / recall / F1:**

| Cell type | Precision | Recall | F1 | Clinical note |
|---|---|---|---|---|
| Eosinophil | 52.3% | 36.9% | 0.43 | Lowest recall of all classes — the model misses nearly 2 in 3 true Eosinophils, mostly by calling them Neutrophil |
| Lymphocyte | 70.4% | 44.5% | 0.55 | Highest precision — when the model says Lymphocyte, it's usually right, but it misses over half of true Lymphocytes (mostly confused with Neutrophil) |
| Monocyte | 64.1% | 65.2% | 0.65 | Most balanced and best-performing class overall |
| Neutrophil | 47.3% | 77.7% | 0.59 | Highest recall but lowest precision — the model over-predicts this class; many "Neutrophil" calls are actually other cell types |

**Why not just accuracy:** overall accuracy hides a real problem here — the model isn't
uniformly mediocre, it's systematically biased toward predicting Neutrophil (predicted 1,025
times vs. only 624 true Neutrophils in the test set). A single accuracy number would mask this
directional bias entirely; per-class precision/recall is what surfaces it.

**Confusion matrix observations:**
- **Eosinophil → Neutrophil is the single largest confusion (243 cases).** This is the most
  clinically expected error: both are granulocytes with a multi-lobed nucleus, distinguished
  mainly by granule color/texture (Eosinophils have bright orange-red granules) — a subtle
  feature that a generic ImageNet-pretrained backbone was never tuned to detect.
- **Monocyte → Eosinophil confusion (109 cases) is more surprising** — these two cell types
  are structurally quite different (large single kidney-shaped nucleus vs. bilobed
  granulated nucleus), so this confusion is less biologically expected and more likely
  reflects a genuine modeling limitation rather than true visual ambiguity.
- **Root cause hypothesis:** the model uses a *frozen* ImageNet-pretrained backbone — none of
  its internal features were fine-tuned on stained blood cell images. ImageNet features are
  tuned for natural-photo shapes/textures, not the fine granule color and staining patterns
  that distinguish WBC subtypes. This is a reasonable, testable explanation, not just an excuse.
- **Improvement path if given more time:** unfreeze the top few layers of the base model and
  fine-tune at a low learning rate directly on the blood cell images (rather than only training
  the classification head), and/or start from a backbone pretrained on histology/medical images
  instead of general photos.

---

## 4. Risk Analysis (ISO 14971–style)

| Failure mode | Potential clinical impact | Likelihood (this exercise) | Mitigation |
|---|---|---|---|
| False negative on Eosinophil (36.9% recall — lowest of all classes) | Under-reporting Eosinophils could mask patterns relevant to allergic or parasitic conditions, where Eosinophil counts are clinically meaningful | **High** — the model misses nearly 2 in 3 true Eosinophils in this exercise, mostly by mislabeling them Neutrophil | Human-in-the-loop review of all outputs before reporting; flag low-confidence predictions for mandatory pathologist review; do not deploy without addressing this recall gap first |
| Over-prediction of Neutrophil (47.3% precision — model predicts this class far more than its true frequency) | Could dilute confidence in automated differential counts generally, since one class is systematically over-called | **High** — directly observed in this exercise's confusion matrix (1,025 predictions vs. 624 true cases) | Confidence thresholding; recalibrating the model's decision boundary; fine-tuning rather than relying on a frozen backbone |
| Model performance degrades on a different scanner, stain batch, or patient population than training data | Silent accuracy drop in deployment not reflected in this validation | **High** (untested in this exercise — single public dataset source, no cross-site validation) | Would require multi-site validation data before any real deployment; monitor performance drift post-deployment |
| Class imbalance in training data biases the model toward majority classes | Systematically worse performance on minority classes | **Low** in this exercise — test set classes were reasonably balanced (~620–624 images each); the observed Neutrophil bias appears to be a feature-learning issue (frozen backbone) rather than a data-volume imbalance issue | If imbalance were present: class-weighted loss function, oversampling, or targeted data collection for underrepresented classes |

---

## 5. Validation Protocol Outline (what a real validation would require beyond this exercise)

1. **Sample size justification** — a real validation would define a minimum sample size per
   class based on the acceptance criteria below and a statistical power calculation, not just
   "however many images happened to be in the public dataset."
2. **Acceptance criteria (example, illustrative)** — e.g. "recall must exceed 95% on each cell
   type before pilot deployment; no single class below 90% recall." (This project's actual
   results should be compared against a threshold like this, even if illustrative.)
3. **Multi-site / multi-scanner validation** — testing on data from more than one lab, scanner,
   and stain batch before claiming generalizability — not done in this exercise, and explicitly
   flagged as a gap.
4. **Discordance handling process** — a defined process for what happens when the model and
   the reviewing pathologist disagree: how it's logged, reviewed, and fed back into monitoring
   or retraining decisions.
5. **Periodic re-validation** — a plan for re-checking model performance on an ongoing basis
   post-deployment, since real-world data drifts over time (new scanners, new stain lots, etc.)

---

## 6. Human-in-the-Loop Deployment Recommendation

Based on this exercise, a responsible rollout — even for a much more mature version of this
model — would look like:

1. Model output is presented to the pathologist as a **suggestion with a confidence score**,
   never as a final result.
2. **Low-confidence predictions are flagged** for mandatory manual review rather than silently
   accepted.
3. **All pathologist overrides/disagreements are logged** in a structured way, creating a
   feedback dataset for future retraining and a monitoring signal for model drift.
4. Only after a defined period of strong concordance between model and pathologist across
   multiple sites would broader autonomy (e.g. pre-populating reports) be considered — and
   even then, subject to the regulatory clearance appropriate to that expanded claim.

---

## 7. Honest Summary of This Exercise's Limitations

- Single public dataset, single source, no multi-site or multi-scanner testing
- Ground truth is a single label per image, not multi-pathologist consensus
- Scope limited to 4 normal cell types — does not address abnormal/atypical cell detection,
  which is clinically the higher-stakes problem
- Built as a demonstration of *how to think about* clinical AI validation, not as a
  production-ready or regulatory-grade validation package

This document is intentionally structured the way an internal validation file might be
structured at a company like SigTuple — the goal is to show the validation *mindset*, not to
claim this small project meets a real regulatory bar.

**Overall conclusion of this exercise:** at 56.1% accuracy with a clear directional bias
toward over-predicting Neutrophil, this model would **not** meet a reasonable bar for even
pilot deployment. The value of this exercise isn't the accuracy number — it's the process of
identifying *why* the model fails the way it does (frozen, non-domain-specific backbone;
subtle granule-texture features not being learned) and mapping that to a concrete next step
(fine-tuning on domain data) rather than treating the model as a black box.
