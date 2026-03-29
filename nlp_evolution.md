
[Back](./README.md)

---

## 📑 Table of Contents

| Section | Topic |
|---------|-------|
| [1](#-overview-of-nlp) | NLP Overview |
| [2](#-rule-based-systems-era-1950s1980s) | Rule-Based Systems |
| [3](#-statistical-era-1980s–early-2000s) | Statistical Era |
| [4](#-machine-learning-era-2000s2013) | Machine Learning Era |
| [5](#-embeddings-era-2013–2017) | Embeddings Era |
| [6](#-recurrent-neural-networks-rnns) | Recurrent Networks |
| [7](#-transformers-era-2017–present) | Transformers |
| [8](#-transformers) | Transformer Details |

---

# 🧠 **Natural Language Processing (NLP)**

## 📚 **Overview of NLP**

### 📜 **Rule-Based Systems Era (1950s–1980s)**

* Based on hand-crafted linguistic rules written by experts (linguists, logicians).
* Used **regular expressions**, **context-free grammars**, **lexicons**, and **pattern matching**:

  * *If a word ends in* `-ed`, *it may be past tense.*
  * *If a word ends in* `-ing`, *it may be present tense.*
  * *Logic was largely* `if-else` *style.*

🔻 **Limitations**:

* ❌ Not scalable: Too many rules for real-world language.
* ❌ Brittle: Fails on edge cases or unseen inputs.

---

### 📊 **Statistical Era (1980s–early 2000s)**

| Model              | Use Case                | Notes                                |
| ------------------ | ----------------------- | ------------------------------------ |
| **N-gram**         | Language modeling       | Simple, fast, but short context      |
| **HMM**            | POS tagging, NER        | Sequence modeling with hidden states |
| **PCFG**           | Parsing                 | Probabilistic grammar rules          |
| **MaxEnt**         | Classification, tagging | Powerful, flexible features          |
| **Naive Bayes**    | Document classification | Fast, assumes feature independence   |
| **Decision Trees** | Text classification     | Learns interpretable rules           |

🔻 **Limitations**:

* 📉 Requires large amounts of labeled data.
* 🔍 Poor at handling long-distance dependencies.
* ⚠️ Suffers from data sparsity and overfitting.

---

### 🤖 **Machine Learning Era (2000s–2013)**

* Used supervised learning algorithms: **SVMs**, **Decision Trees**, **CRFs**.
* Relied on **hand-crafted features**: suffixes, POS, word shape.

🔻 **Limitations**:

* Heavy reliance on manual feature engineering.
* Lacked **semantic understanding**.
* Struggled with **context** and **polysemy** (e.g., *“bank”* as river vs. finance).

---

### 🧱 **Embeddings Era (2013–2017)**

* Introduced **dense vector representations** of words.
* Key models: **Word2Vec**, **GloVe**, **FastText**.
* Trained on large corpora using **unsupervised learning**.

```plaintext
vector("king") - vector("man") + vector("woman") ≈ vector("queen")
```

🔻 **Limitations**:

* 🧍 Static embeddings (same vector in all contexts).
* ❌ Struggled with contextual nuance & polysemy.
* 🔧 Still required task-specific architectures.

---

### 🔁 **Recurrent Neural Networks (RNNs)**

* Became standard for sequence modeling in the mid-2010s.
* Maintained "memory" of past inputs.

🔹 **Variants**:

* **LSTM** (1997): Solved vanishing gradient with memory cells.
* **GRU** (2014): Simpler but similarly effective.

🔻 **Limitations**:

* 🚫 Still struggled with long-range dependencies.

---

### ⚡ **Transformers Era (2017–Present)**

* Introduced in **"Attention is All You Need"** (2017).
* Replaced recurrence with **self-attention** and deep networks.

🔻 **Limitations**:

* 💰 High computational cost and energy use.
* 🧠 Prone to hallucinations in open-ended generation.
* Requires **massive data** and infrastructure.

---

## 🔄 **Transformers**

Transformers revolutionized NLP by eliminating recurrence and relying entirely on **attention mechanisms**.

---

### 🧠 **Encoder (e.g., BERT)**

* Processes input all at once, producing **context-aware** representations.
* Used in **understanding tasks**: classification, QA, NER.

#### ➤ What It Does:

* Converts tokens into rich vector embeddings using **context**.

#### ➤ Analogy:

> Like reading a sentence and summarizing its meaning.

#### ➤ Input:

`"The cat sat on the mat."`

#### ➤ Output:

Contextual vector for each word.

#### ➤ Use Cases:

* ✅ Sentiment analysis
* ✅ Named Entity Recognition
* ✅ Input for summarization/translation

---

### ✍️ **Decoder (e.g., GPT)**

* Generates text one token at a time based on previous context.

#### ➤ What It Does:

* Predicts next word from context (and encoder output, if used).

#### ➤ Analogy:

> Like speaking a sentence out loud, one word at a time.

#### ➤ Input:

* Prompt or prior tokens (e.g., `"The cat sat"`)

#### ➤ Output:

* Predicted next token (e.g., `"on"`)

#### ➤ Use Cases:

* ✅ Text generation
* ✅ Translation
* ✅ Chatbots

---

### 🔀 **Encoder + Decoder (e.g., T5)**

* Combines **understanding (encoder)** + **generation (decoder)**.
* Format: **"text in → text out"**

---

### 🎯 **The Attention Mechanism**

> Lets each token **focus** on relevant words in the sequence.

🧠 Not all words are equally important — attention helps the model decide **what to prioritize**.

---

## 🔧 **Pre-training & Fine-tuning**

Two-step process for modern LLMs:

---

### 🏋️‍♂️ **Pre-training**

#### ➤ What It Is:

Learning language from **massive unlabeled corpora** (books, websites, etc.).

> *“Teach the model how language works before asking it to do a job.”*

#### ➤ Goal:

* Learn syntax, grammar, world knowledge, reasoning.

#### ➤ Tasks:

* **BERT**:

  * *Masked Language Modeling* (MLM)

    > `"The cat [MASK] on the mat"` → `"sat"`
  * *Next Sentence Prediction* (NSP)

* **GPT**:

  * *Causal Language Modeling* (CLM)

    > `"The cat sat on"` → `"the"`

* **T5**:

  * *Text-to-text* framework:

    > `"summarize: The cat sat..."` → `"cat on mat"`

✅ Result: A model with **general understanding**, but not yet task-specific.

---

### 🧪 **Fine-tuning**

#### ➤ What It Is:

Adapting a pretrained model to a **specific task** using labeled data.

> *“Now teach the model how to do **your** job.”*

#### ➤ Goal:

Make the model specialized (e.g., QA, NER, translation).

#### ➤ How:

* Add task-specific head (e.g., classifier).
* Train on a small, labeled dataset.

---

### 🎓 **Real-World Analogy**

| Stage            | Analogy                                        |
| ---------------- | ---------------------------------------------- |
| **Pre-training** | Learning English from books and the internet   |
| **Fine-tuning**  | Taking a course on law, customer service, etc. |

---

### ❓ **If fine-tuning is task-specific, how do LLMs like ChatGPT "know everything"?**

---

#### 🏗️ **Massive Pretraining (Foundation Stage)**

* Models like GPT-4/ChatGPT trained on:

  * Books, code, articles, forums, etc.
* Trained to:

  * Predict next word (CLM)
  * Understand facts, reasoning, and style

---

#### 🧑‍🏫 **Instruction Fine-Tuning (IFT)**

* Taught to **follow instructions**:

  * `"Translate this → <output>"`
  * `"Write a poem → <output>"`

✅ Result: Behaves like an **assistant**, not just a generator.

---

### 🤔 **So Why Does It "Know Everything"?**

Because:

* 📚 It was trained on nearly everything (public data).
* 🧠 Fine-tuned on a huge variety of tasks/instructions.

---

## 🔤 Tokenization & 🔡 Embeddings

Two **foundational steps** in any NLP pipeline — especially in Transformers.

---

## 1️⃣ 🔤 **Tokenization**

### ➤ What It Is:

Breaking text into **tokens**: words, subwords, characters, etc.

### ➤ Why It Matters:

Models can't process raw text — they need **numerical input**.

### ➤ Types:

| Type                | Example                         | Used By                     |
| ------------------- | ------------------------------- | --------------------------- |
| **Word-level**      | `"playing"` → `"playing"`       | Legacy models               |
| **Character-level** | `"playing"` → `"p", "l", ...`   | Some RNNs                   |
| **Subword-level**   | `"playing"` → `"play", "##ing"` | BERT (WordPiece), GPT (BPE) |

✅ Most models use **subword tokenization**: balances vocabulary size & flexibility.

### ➤ Example (BERT):

`"unhappiness"` → `"un"`, `"##happiness"`

---

## 2️⃣ 🔡 **Embeddings**

### ➤ What It Is:

Mapping tokens to **vectors of real numbers**.

> Vectors represent **meaning** in high-dimensional space.

### ➤ Why It Matters:

Semantically similar words (like *“king”* and *“queen”*) have **similar vectors**.

### ➤ How:

Each token ID maps to a vector from an **embedding matrix**.

```python
embedding_matrix[token_id] → [0.23, -0.57, 0.98, ...]
```

---

### ➤ Types:

| Type                      | Purpose              | Used In         |
| ------------------------- | -------------------- | --------------- |
| **Word embeddings**       | One vector per word  | Word2Vec, GloVe |
| **Contextual embeddings** | Varies with sentence | BERT, GPT       |
| **Positional embeddings** | Adds position info   | Transformers    |

✅ Transformers add **semantic + positional** embeddings together.

---

## 🧠 **Summary Table**

| Concept          | What It Does            | Output                                    |
| ---------------- | ----------------------- | ----------------------------------------- |
| **Tokenization** | Breaks text into tokens | `["The", "cat", "sat"]`                   |
| **Embedding**    | Maps tokens to vectors  | `[[0.1, 0.5, ...], [0.3, 0.8, ...], ...]` |

---

## 🧪 **Example: GPT Tokenization & Embedding**

> Input: `"Cats are great!"`

1. 🔤 **Tokenized**: `["Cats", " are", " great", "!"]`
2. 🔢 **Token IDs**: `[10965, 389, 920, 0]` *(example IDs)*
3. 📈 **Embeddings**: `[0.27, -0.14, ..., 0.09]` *(per token)*

---
[Back](./README.md)