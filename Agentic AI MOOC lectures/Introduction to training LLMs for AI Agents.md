# Introduction to training LLMs for AI Agents

# General LLM Training Pipeline (3 parts)

![Screenshot 2025-10-02 at 7.42.56 PM.png](Introduction%20to%20training%20LLMs%20for%20AI%20Agents%202804c0b8160880a8aec9f3a79308f655/Screenshot_2025-10-02_at_7.42.56_PM.png)

hacks → The model found a way of optimizing the reward, even though that's not what you were hoping they would do. For eg, removing tests in your environment or replacing it always returning true

# LLM Specializing pipeline

![Screenshot 2025-10-02 at 7.59.03 PM.png](Introduction%20to%20training%20LLMs%20for%20AI%20Agents%202804c0b8160880a8aec9f3a79308f655/Screenshot_2025-10-02_at_7.59.03_PM.png)

# 5 things to consider while training LLM

1. Architecture
2. Training algorithm/loss
3. Data & RL env
4. Evaluation
5. System and infra to scale 

# Pretraining

## Method

- Goal: teach the model everything in the world
- Task: predict the next word
- Data: any reasonable data on internet
    - > 10T tokens (20-40T for llama 4, 15T for DSv3)
    - > 20B unique web pages

**Examples:**

### AR Language Models

- Task: predict the next word
- Steps (using ) :
    1. tokenize → convert each words to tokens and index them like 1,2,3…
    2. forward pass → pass through the model (transformer)
    3. predict probability of next token
    4. sample → sample from probability distribution
    5. detokenize

Step 1, 2, 3 happens in training time and 4, 5 happens in inference time

### A Simple Language Model: N-grams

- Predicts the word by using Statistics
- Predicted probability for X is
P(X| the grass is) = Count(X| the grass is)/Count(the grass is)
- Problem:
    - You need to keep count of all occurrences for each n-gram
    - Most sentences are unique: this can’t generalize
- Solution: neural networks

### Neural Language Models

![Screenshot 2025-10-05 at 3.37.00 AM.png](Introduction%20to%20training%20LLMs%20for%20AI%20Agents%202804c0b8160880a8aec9f3a79308f655/Screenshot_2025-10-05_at_3.37.00_AM.png)

## Data

- Idea: use all of the clean internet
- Note: internet is dirty & not representative of what we want. Practice:
    1. Download all of internet. Common crawl: 250 billion pages, > 1PB (>1e6 GB), WARC file
    2. Text extraction from HTML (challenges: math, boiler plate, etc.)
    3. Filter undesirable content (e.g. NSFW, harmful content, PII)
    4. Deduplicates (url/document/line). E.g. all the headers/footers/menu in forums are always same
    5. Heuristic filtering. Rm low quality documents (e.g. # words, word length, outlier toks, dirty toks)
    6. Model based filtering. Predict if page could be references by Wikipedia.
    7. Data mix. Classify data categories (code/books/entertainment). Reweight domains using scaling laws to get high downstream performance.
- Also: lr annealing on high-quality data, continual pretraining with longer context

**Mid Training**

- Continued pretraining to adapt the model to desired properties / higher quality data (<1T toks)
- Data mix changes shifts: eg more scientific, coding, multilingual data
- Longer context extension: bump (eg 4 → 128k for DSv3)
- Desired formatting/instruction following
- Higher quality data
- Reasoning data

## Compute

- Empirically: for any type of data and model, the most important is how
much compute you spend on training (data & size)
- You can even predict performance with compute with scaling laws!

**Scaling Laws**

- You have 10K GPUs for a month, what model do you train?
- Old pipeline:
• Tune hyperparameters on big models (e.g. 30 models)
• Pick the best => final model is trained for as much as each filtered out ones (e.g. 1 day)
- New pipeline:
• Find scaling recipes (eg lr decrease with size)
• Tune hyperparameters on small models of different sizes (e.g. for <3 days)
• Extrapolate using scaling laws to larger ones
• Train the final huge model (e.g. >27 days)

<aside>
💡

*Post-training overview*: The goal of test-time scaling and alignment is to make models follow human intent. Pretraining provides general knowledge, while post-training adapts models to desired behaviors.

</aside>

# Post-training

- Background:
• data of desired behaviors is what we want but scarce and expensive
• pretraining data scales but is not what we want
- Idea: finetune pretrained LLM on a little desired data => “post-”training

## Method

### Supervised finetuning (SFT)

![Screenshot 2025-10-05 at 9.36.46 PM.png](Introduction%20to%20training%20LLMs%20for%20AI%20Agents%202804c0b8160880a8aec9f3a79308f655/Screenshot_2025-10-05_at_9.36.46_PM.png)

- Problem: human data is slow to collect and expensive
- Idea: use LLMs to scale data collection

**Data generation methods:**

- Problem: LLM-generated data ~assumes that you have a smarter LLM
- Idea: use rejection sampling based on verifiers
    1. Temporary LLM generates many answers
    2. Keep answer if it’s correct (eg, passes test case), or preferred over others

**Data requirements:**

- SFT doesn’t need huge datasets — e.g., LIMA used ~2K examples; DeepSeek-R1 used ~800K.
- Smaller datasets work if pretraining already learned core skills.

**Problems with SFT:**

SFT is behavior cloning of humans

1. Bound by human abilities: humans may prefer things that they are not able to generate
2. Hallucination: cloning correct answer teaches LLM to hallucinate if it didn’t know about it!
3.  Price: collecting ideal answers can be expensive

### Reinforcement learning

- Idea: maximizing rewards (desired behavior) instead of cloning behavior
- Key: where the reward comes from:
• Rule-based rewards (eg string match for close-ended QA, or passing coding function)
• Reward model trained to predict human preferences (RLHF)
• LLM as a judge

Example: *DeepSeek-R1’s GRPO algorithm*

- Samples multiple outputs, scores them via verifiers or LLM judges, then updates the model.
- Uses KL-regularization to stay close to the base model.

![Screenshot 2025-10-05 at 10.16.18 PM.png](Introduction%20to%20training%20LLMs%20for%20AI%20Agents%202804c0b8160880a8aec9f3a79308f655/Screenshot_2025-10-05_at_10.16.18_PM.png)

![Screenshot 2025-10-05 at 10.19.54 PM.png](Introduction%20to%20training%20LLMs%20for%20AI%20Agents%202804c0b8160880a8aec9f3a79308f655/Screenshot_2025-10-05_at_10.19.54_PM.png)

### RL from Human Feedback (RLHF)

- Idea: maximize human preference rather than clone their behavior
- Made ChatGPT
- Pipeline:
    1. For each instruction: generate 2 answers from a pretty good model (SFT)
    2. Ask labelers to select their preferred answers
    3. Finetune the model to generate more preferred answers (PPO or DPO)

![Screenshot 2025-10-05 at 10.25.12 PM.png](Introduction%20to%20training%20LLMs%20for%20AI%20Agents%202804c0b8160880a8aec9f3a79308f655/Screenshot_2025-10-05_at_10.25.12_PM.png)

**RLHF: challenges of human data**

- Slow & expensive
- Hard to focus on correctness rather than form (eg length)
- Annotator distribution shifts its behavior
- Crowdsourcing ethics

Solution: replace human preferences with LLM preferences

# Evaluation

**Importance of evaluation in AI**
Quantify progress towards desired task to:

- Identify improvements
- Select models
- Decide if production ready

## Close-ended

- Idea: if you turn problem into something where you have few possible answers => you can automatically verify whether your answer is correct
- Challenges:
    - Sensitivity to prompting/inconsistencies
    - Train & test contamination (~not important for development)

## Open-ended

- Challenges to evaluate something like ChatGPT:
• Large diversity
• Open-ended tasks => hard to automate
- Idea: ask for annotator preference between answers
- Idea: use LLM instead of human
- Steps:
• For each instruction: generate output by baseline and model to eval
• Ask GPT-4 which output is better
• Average win-probability => win rate
- Benefits:
• 98% correlation with ChatBot Arena
• < 3 min and < $10
• Challenge: spurious correlation

# Systems

- With growing model scale, compute becomes a bottleneck.
- One must optimize training pipelines, resource allocation, and GPU usage.

## GPUs

- Massively parallel: same instruction applied on all thread but different inputs.
=> Optimized for throughput!

![Screenshot 2025-10-06 at 4.23.48 AM.png](Introduction%20to%20training%20LLMs%20for%20AI%20Agents%202804c0b8160880a8aec9f3a79308f655/Screenshot_2025-10-06_at_4.23.48_AM.png)

Difference between CPUs and GPUs are that GPUs are massively parallel

- GPUs are optimized for fast matrix multiplications
- Bottleneck for GPU is not computation but feeding the processors with data as much as possible.
- Memory hierarchy:
• Closer to cores => faster but less memory
• Further from cores => more memory but slower

![Screenshot 2025-10-06 at 4.30.30 AM.png](Introduction%20to%20training%20LLMs%20for%20AI%20Agents%202804c0b8160880a8aec9f3a79308f655/Screenshot_2025-10-06_at_4.30.30_AM.png)

- Metric: Model Flop Utilization (MFU)
    - metric for system efficiency
    - Ratio: observed throughput / theoretical best for that GPU
    - 50% MFU is considered very good; many industry systems target 15–20%

## Techniques to improve throughput

- **Low-precision / mixed precision (e.g. BF16 vs FP32) to reduce memory** — reduces the number of bits used to represent data, allowing faster computation and lower memory usage with minimal loss in model accuracy.
- **Kernel fusing / operation fusion** to reduce communication— combines multiple sequential GPU operations into a single kernel call, cutting down costly memory transfers between global and local memory.
- **Tiling / reorder of operations** — breaks computations (like matrix multiplications) into smaller tiles so that data fits in faster caches, enabling reuse of loaded data and reducing redundant memory reads.

**FlashAttention & recomputation tradeoffs** — As a case study of combining kernel fusion, tiling, and recomputation, FlashAttention yields ~1.7× overall speedups by sometimes recomputing intermediate results rather than storing them.

## **Parallelization**

To train large models, you must split both computation and state across multiple GPUs.

### Data parallelism

- replicate model, split batches, communicate and reduce (sum) gradients
- S*harding* — each GPU updates subset of weights and communicate them before next step

### Model parallelism

- Problem: data parallelism only works if batch size >= # GPUS
- Idea: have every GPU take care of applying specific parameters (rather than updating)
• Pipeline parallel: every GPU has different layer
• Tensor parallel: split up the weights into different GPUs

## Architecture Sparsity

- Idea: models are huge => not every datapoint needs to go through every parameter
• Eg Mixture of Experts: use a selector layer to have less “active” parameter => same FLOPs more parameters