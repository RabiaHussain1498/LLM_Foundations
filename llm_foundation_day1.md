
## System baseline (before running models)
- CPU cores: 8
- RAM: 31Gi total, 25Gi available


## Item 1 — Tokenizer exercise
Text used: paragraph on F1-score and LOSO cross-validation "We used accuracy and F 1 -score as evaluation metrics. Accuracy
represents the number of correctly classified instances out of all
samples. The F 1 -score is defined as the harmonic mean of preci-
sion, indicating the reliability of the results in a certain class, and
recall, representing a measure of completeness. To obtain the final
F 1 -score, precision and recall were computed for each class sepa-
rately and then averaged. Applying the F 1 -score is recommended
for unbalanced classification tasks, which is the case when using
WESAD (since the various conditions were carried out at differ-
ent lengths during the study protocol). All models were evaluated
using the leave-one-subject-out (LOSO) cross-validation (CV) pro-
cedure. Hence, the results indicate how a model would generalise
and perform on data of a previously unseen subject."(from research paper, GPT-3 legacy tokenizer)
Tokens: 197 | Characters: 848 | Ratio: ~4.3 chars/token

Surprising splits:
- "F1-score" is split into `F`, `1`, `-score` — separating the number from the letter
- Line-break hyphenation from the source PDF ("preci-"/"sion", "sepa-"/"rately", "differ-"/"ent")
  became literal token boundaries, the tokenizer has no concept these were meant to be one word


  ## Item 2 — Context-window arithmetic
Using a real conversation about wearable-sensor stress detection, I estimated
an average of approximately 150 tokens for a user message and 250 tokens for
an assistant response.

One back-and-forth turn uses approximately: 150 + 250 = 400 tokens

For an 8,192-token context window:
8,192 ÷ 400 ≈ 20.48 → approximately 20 back-and-forth turns would exhaust
the context window.

This shows why a model may appear to "forget" earlier information in a long
conversation: once the conversation becomes too large for the available
context window, older parts of the conversation may no longer be included
in the request.


## Item 3 — Zero-shot vs. few-shot prompts

Task: Extract the model name, dataset, evaluation metric, and cross-validation
method from a sentence, output as strict JSON.

### Zero-shot prompt:
Extract the model name, dataset, evaluation metric, and cross-validation
method from the sentence below. Return ONLY valid JSON with keys:
"model", "dataset", "metric", "cv_method".

Sentence: "We evaluated a Random Forest classifier on the WESAD dataset
using F1-score, applying leave-one-subject-out cross-validation."

### Few-shot prompt (3 worked examples):
Extract the model name, dataset, evaluation metric, and cross-validation
method from each sentence. Return ONLY valid JSON with keys:
"model", "dataset", "metric", "cv_method".

Example 1:
Sentence: "The CNN was trained on CIFAR-10 and evaluated using accuracy
with 5-fold cross-validation."
Output: {"model": "CNN", "dataset": "CIFAR-10", "metric": "accuracy", "cv_method": "5-fold"}

Example 2:
Sentence: "An SVM classifier was tested on the MNIST dataset, reporting
precision under 10-fold cross-validation."
Output: {"model": "SVM", "dataset": "MNIST", "metric": "precision", "cv_method": "10-fold"}

Example 3:
Sentence: "We applied XGBoost to the Adult Income dataset and measured
recall using stratified k-fold cross-validation."
Output: {"model": "XGBoost", "dataset": "Adult Income", "metric": "recall", "cv_method": "stratified k-fold"}

Now extract from this sentence:
Sentence: "We evaluated a Random Forest classifier on the WESAD dataset
using F1-score, applying leave-one-subject-out cross-validation."
Output:


## Item 4 — Chain-of-thought prompts

Version A (no CoT requested):
A store sells notebooks for $3 each and pens for $2 each. Sarah buys 4
notebooks and some pens, spending $22 total. How many pens did she buy?

Version B (explicit CoT requested):
A store sells notebooks for $3 each and pens for $2 each. Sarah buys 4
notebooks and some pens, spending $22 total. How many pens did she buy?
Think step by step before giving your final answer.

## Item 5 — Ollama installed and confirmed running
ollama version is 0.34.0
WARNING: No NVIDIA/AMD GPU detected. Ollama will run in CPU-only mode.



## Item 6 — Llama 3.1 8B verification
Prompt: "What is the best technique for model evaluation to implement when
you have data of many subjects?"
Result: Coherent, well-structured answer covering cross-validation,
subject-level vs group-level evaluation, holdout method, and relevant
metrics (accuracy, precision, recall, F1, MSE/RMSE), plus working example
code. No unprompted reasoning trace shown — direct answer, as expected for
a non-reasoning model.
Verified working.

## Item 7 — DeepSeek-R1 8B verification
Prompt: "A train leaves station A at 60 mph. Two hours later, a second
train leaves the same station on the same track at 90 mph. How long
until the second train catches the first?"
Result: Model produced a long, fully unprompted <think>...</think> block —
set up the distance equation (90t = 60(t+2)), solved it, then independently
re-verified using relative speed (120mi / 30mph = 4hr) and a full distance
check (360mi both ways) before committing to the final answer.
Final answer: 4 hours — correct.
Verified working, and clearly demonstrates trained-in reasoning behavior
(no "think step by step" instruction was given).


## Item 8 — Zero-shot vs. few-shot test (Llama 3.1 8B)

Zero-shot output:
{"model": "Random Forest classifier", "dataset": "WESAD", "metric": "F1-score", "cv_method": "leave-one-subject-out"}

Few-shot output:
{"model": "Random Forest", "dataset": "WESAD", "metric": "F1-score", "cv_method": "leave-one-subject-out"}

Honest comparison: Both outputs were valid JSON with all four correct keys, format compliance was already high in zero-shot, so few-shot did not fix a reliability problem here (Llama 3.1 8B is capable enough to follow the
instruction alone). The one real effect of the 3 examples was on content style, not structure: zero-shot kept
"Random Forest classifier" (matching the input sentence's wording), while few-shot produced the terser "Random
Forest", matching the naming convention shown in the examples (CNN, SVM, XGBoost, none with a descriptor word). So few-shot's benefit here was subtle content-style alignment rather than format correction.

## Item 9 — Chain-of-thought comparison

Llama 3.1 8B — Version A (no CoT requested):
Produced step-by-step reasoning anyway ("## Step 1 / Step 2 / Step 3"),
unprompted. Correct answer: 5 pens.

Llama 3.1 8B — Version B (explicit CoT requested):
Nearly identical reasoning structure, plain numbered list instead of
markdown headers, slightly more conversational tone. Correct answer: 5 pens.
No meaningful difference from Version A on this problem.

DeepSeek-R1 8B (from Item 7, unprompted, trained-in reasoning):
Produced a long, raw <think> block, re-deriving the answer via multiple
independent methods, second-guessing the question's phrasing, self-verifying
before committing to a final boxed answer. Qualitatively different from
Llama's clean numbered steps.
DeepSeek-R1 produced a fundamentally different kind of output: a messy, exploratory visible reasoning 
process with self-doubt and repeated re-verification, rather than just a clean list of steps.


Two-sentence comparison:
A model reasoning because it was explicitly asked to (Llama, Version B)
produced reasoning that was barely distinguishable from its own unprompted
default (Version A), the instruction added little because the model
already tends to show its work on arithmetic. A model reasoning because
that's how it was trained to respond (DeepSeek-R1) produced a fundamentally different kind of output: a messy, exploratory visible reasoning process with self-doubt and repeated re-verification, rather than just a clean list of steps.

## Item 10 — Resource impression

\- RAM: 31Gi total, 18Gi available after running models

\- Ollama model storage: 9.5G

\- GPU: None detected (CPU-only mode)


## Model Comparison

The following models will be compared using the same test prompt and Ollama. Query time and generation speed will be recorded from the Ollama API response.

| Model       | Parameters | Quantization |               Query time |
| ----------- | ---------: | ------------ | -----------------------: |
| Llama 3.1   |       8.0B | Q4_K_M       |   **223.324 s (3m 43s)** |
| Qwen3       |       8.2B | Q4_K_M       |   **430.862 s (7m 11s)** |
| Granite 4.2 |       8.8B | Q4_K_M       |  **748.006 s (12m 28s)** |
| DeepSeek-R1 |       8.2B | Q4_K_M       | **1119.651 s (18m 40s)** |

**Test prompt:**

"What is the best technique for model evaluation to implement when you have data of many subjects?"

**Observation:**
Llama 3.1 was the fastest model in the comparison and provided a concise, to-the-point answer. Qwen3 was slower but still completed the task reasonably quickly, while Granite 4.2 and DeepSeek-R1 took considerably longer. This shows that models with similar parameter sizes and the same Q4_K_M quantization can still have noticeably different inference times and response styles.
