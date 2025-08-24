> Assets and results for a thesis on **vision-language models (VLMs)** for **micro-navigation** supporting blind/low-vision (BLV) users.

- **Images** across: _Home_, _Supermarket_, _Campus-Indoor_, _Campus-Outdoor_

- **Annotations** for: _Navigational Guidance_, _Approximate Distance_, _Environmental Understanding_

- **Predictions** from VLM-only and a **VLM → LLM** pipeline

---

## At a Glance

- **Why a VLM→LLM pipeline?** VLMs are strong at perception but inconsistent at multi-step reasoning and schema-constrained phrasing. A lightweight LLM stage (no training) aggregates context, enforces an action schema, adds safety/tactile cues, and improves user-centric clarity.

- **Human eval** follows VIALM: _Correctness_ (Grounding) and _Actionability_ (Fine-grainedness) with fail rates: **Not Grounded** (≤3/5) and **Not Fine-grained** (≤3/5).

## Annotation Formats (JSON)

### 1) Navigational Guidance

**Goal:** mid-level guidance with tactile/safety cues.

`{
  "index": 2,
  "question": "Approach the kitchen faucet ...",
  "ans_gt": "You are standing to the left ... turn right 90 degrees; ... The target is at your Front ..."
}`

**Fields**

- `index`: image/sample id

- `question`: user request (names target object)

- `ans_gt`: ground-truth paragraph (stepwise actions + tactile/safety notes)

---

### 2) Approximate Distance

**Goal:** straight-line 3D distance from camera center to nearest target instance (m, 2 dp).

`{
  "index": 2,
  "question": "... kitchen faucet ...",
  "ans_gt": "...",
  "distanceQ": "Report the straight-line 3D distance (m, 2 dp) ...",
  "distanceA": 6.08
}`

**Fields**

- `distanceQ`: fixed instruction

- `distanceA`: distance in **metres (two decimals)**

---

### 3) Environmental Understanding

**Goal:** one-sentence scene description (no numbers).

`{
  "index": 1,
  "question": "... sofa ...",
  "ans_gt": "...",
  "EnvQ": "One-sentence global scene description (no numbers).",
  "EnvA": "A serene, minimalist living room ..."
}`

**Fields**

- `EnvQ`: fixed instruction

- `EnvA`: human-authored sentence (reviewed for clarity)

---

## Provenance & Labeling Protocol

- **Home & Supermarket** annotations are taken from---and follow---the protocol in _VIALM_ (Zhao et al., 2024): curated images; labels by trained English-fluent annotators; supervision by VIA experts.

- **Campus-Indoor & Campus-Outdoor** annotations were **authored by the researcher**.

- **Distances:**

  - Home/Supermarket: estimated via visual cues (relative size, perspective, layout).

  - Campus-Indoor/Outdoor: same heuristics **plus** iOS **Measure** (AR) pinning; recorded in metres (2 dp).

---

## Models & Settings (what's included)

- **VLM-only:** Qwen-VL, LLaVA, GPT-4o

- **VLM → LLM pipeline:** Qwen-VL → Qwen-Chat, GPT-4o → GPT-4o

- **Outputs stored in** `predictions/*.json` (format depends on task; see examples above)

---

## Using This Repo in Google Colab

> No `requirements.txt` needed. Open a Colab, then:

1.  **Mount Drive** and point paths to your repo folder.

2.  **Run your inference/eval cells** (the same ones used for generating files in `predictions/`).

3.  Save results back to `predictions/`.

**Example path setup (Colab):**

`from google.colab import drive
drive.mount('/content/drive')

ROOT = "/content/drive/MyDrive/YourRepo"
ANN = f"{ROOT}/annotations/outdoor_qa_pair_gt.json"
IMGS = f"{ROOT}/images/campus_outdoor"
OUT = f"{ROOT}/predictions/llava_outdoor_navigation_instructions.json"`

---

## Evaluation Summary (what metrics mean)

- **ROUGE-based Region Guidance (RG_1 / RG_2 / RG_L)**\
  ROUGE on a normalized **region-tag sequence** extracted from text.

  - **RG_1** = ROUGE-1 (unigrams; "coarse" region coverage)

  - **RG_2** = ROUGE-2 (bigrams; "fine" composite cues & local order)

  - **RG_L** = ROUGE-L (longest-common-subsequence; global order/coverage)

- **BERT_S** (BERTScore-F1)\
  Semantic similarity with contextual embeddings (default: `roberta-large`, IDF weighting + baseline rescale recommended). Higher is better.

- **Len**\
  Absolute difference in token/word length between candidate and reference (verbosity proxy).

- **Human Evaluation**\
  _Correctness (Grounding)_ and _Actionability (Fine-grainedness)_ on a 1--5 scale.

  - **Not Grounded Rate** = proportion with _Correctness ≤ 3_

  - **Not Fine-grained Rate** = proportion with _Actionability ≤ 3_

---

## Contact

Questions, corrections, or requests?\
Open a GitHub issue with:

- the JSON file(s) involved,

- a short description of the task,

- the Colab cell(s) you ran and any logs.
