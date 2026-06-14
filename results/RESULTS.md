# Morphology-aware BPE for Arabic — Results

_Configured backend: `camel` · Effective backend: `camel` · RUN_MODE: `medium` · BPE vocab: 8000 · seeds: [42, 123, 2024, 7, 31]_

## 1. Research Question

Does training a BPE tokenizer on morphologically-segmented Arabic text ("morphology-aware BPE") yield better downstream task performance than a standard BPE trained on raw Arabic text, on (a) sentiment classification (LABR) and (b) lexical retrieval (MIRACL-Arabic)?

## 2. Tokenizer Design

- **Standard BPE**: trained from scratch on the LABR training text (no normalization beyond the LABR loader's default).
- **Morphology-aware BPE**: each training sentence is segmented with the `camel` backend (`mode=full`) BEFORE BPE training, so the BPE merges learn morpheme-aware subwords.
- **Vocabulary**: 8000 merges; min frequency 2.
- **No leakage**: BPE never sees test text; overlapping items are removed automatically by `build_bpe_tokenizers`.

## 3. Tokenization Behaviour and Fertility

| Tokenizer | Pieces/orig-word | Pieces/input-unit | UNK rate | Vocab coverage |
|---|---|---|---|---|
| Standard BPE | 1.402 | 1.402 | 0.0000 | 0.9669 |
| Morphology-aware BPE | 1.688 | 1.104 | 0.0000 | 0.9130 |
_Pieces/orig-word is the fair comparison: number of BPE pieces produced per ORIGINAL Arabic word (before any morphological segmentation). Pieces/input-unit uses the units actually fed to BPE — these are identical for standard BPE but smaller for the morph-aware case, where the input has already been split into morphemes. Morphology-aware BPE is expected to yield MORE pieces per original word, since it produces smaller linguistically meaningful units; that is a property, not a failure._

### Segmentation quality

| Metric | Value | What it means |
|---|---|---|
| Segmentation rate | 0.544 | fraction of inspected words split into ≥2 pieces |
| Avg pieces per word | 1.716 | average number of pieces produced per word |
| Valid one-char clitic rate | 0.118 | fraction of pieces that are CORRECT one-letter Arabic clitics (و, ف, ب, ك, ل, س) - not errors |
| Non-proclitic one-character piece rate | 0.134 | fraction of pieces that are one-character but NOT valid Arabic clitics - diagnostic indicator requiring manual review; it may include valid suffix pronouns, punctuation, or true over-segmentation |

_The previous 'over-segmentation rate' is now split into valid Arabic clitic splits and suspicious short pieces. This avoids counting legitimate proclitics such as و, ف, ب, ل, ك, and س as segmentation errors._

**Segmentation examples** (backend = `camel`)

| Word | normalization | prefixes | suffixes | full |
|---|---|---|---|---|
| المعلمون | المعلمون | ال معلمون | المعلمون | ال معلمون |
| بالمدرسة | بالمدرسة | ب ال مدرسة | بالمدرسة | ب ال مدرسة |
| وكتابهم | وكتابهم | و كتابهم | وكتاب هم | و كتاب هم |
| فسيذهبون | فسيذهبون | ف س يذهبون | فسيذهبون | ف س يذهبون |
| للأطفال | لالأطفال | ل ال أطفال | لالأطفال | ل ال أطفال |
| كتبتها | كتبتها | كتبتها | كتبت ها | كتبت ها |
| يكتبون | يكتبون | يكتبون | يكتبون | يكتبون |
| الطالبات | الطالبات | ال طالبات | الطالبات | ال طالبات |
| وقالوا | وقالوا | و قالوا | وقالوا | و قالوا |
| بمدرستنا | بمدرسةنا | ب مدرسةنا | بمدرسة نا | ب مدرسة نا |

## 4. Classification Results: BPE vs morph-aware BPE

| Tokenizer | Accuracy | Precision | Recall | Macro F1 |
|---|---|---|---|---|
| Standard BPE | 0.7948 | 0.7951 | 0.7948 | 0.7948 |
| Morphology-aware BPE | 0.8075 | 0.8077 | 0.8075 | 0.8075 |

_Dataset: LABR (polarity)._ _Seeds: [42, 123, 2024, 7, 31] (per-seed BPE retraining + LR fit)._

### 4b. Vocabulary-size sensitivity

_Single-seed sweep across vocab sizes [4000, 8000, 16000] (seed = 42). Cheap robustness check on top of the multi-seed headline result above._

| Vocab size | BPE F1 | Morph-BPE F1 | Δ F1 |
|---|---|---|---|
| 4000 | 0.7980 | 0.8010 | 0.0030 |
| 8000 | 0.7954 | 0.8040 | 0.0085 |
| 16000 | 0.7878 | 0.8105 | 0.0226 |

## 5. Retrieval Results (candidate-pool reranking): BPE vs morph-aware BPE

| Tokenizer | MRR | MAP | nDCG@10 | P@10 | Hit@10 |
|---|---|---|---|---|---|
| Standard BPE | 0.3319 | 0.2923 | 0.3270 | 0.0740 | 0.5700 |
| Morphology-aware BPE | 0.3007 | 0.2717 | 0.3076 | 0.0750 | 0.5300 |

_Dataset: MIRACL-Arabic. **Task = candidate-pool reranking** of the judged passages for each query (we do NOT index the full corpus). Signal: BM25 over BPE subwords._

_**Primary metric (pre-declared): nDCG@10.** Secondary metrics (MRR, MAP, P@10, Hit@10) are reported alongside for context but the verdict and abstract claims are anchored to the primary._

## 6. Statistical Significance

- Classification: morph-aware BPE shows a statistically significant positive effect for macro-F1 (Δ macro-F1 = +0.0127, paired-t p = 0.0023).
- Classification: morph-aware BPE shows a statistically significant positive effect for accuracy (Δ accuracy = +0.0127, paired-t p = 0.0023).
- Classification (example-level McNemar on the last seed): morph-aware BPE fixed 129 test examples and broke 111 (net +18); chi2 = 1.204, p = 0.2725, not statistically significant.
- Retrieval (PRIMARY = nDCG@10): morph-aware BPE shows a non-significant negative trend for nDCG@10 (Δ nDCG@10 = -0.0194, paired-t p = 0.3896).
- Retrieval secondary metrics are **MIXED**: improved on Precision@10 (Δ=+0.0010); declined on MRR (Δ=-0.0312), MAP (Δ=-0.0206), Hit@10 (Δ=-0.0400). The verdict above is anchored to the pre-declared primary metric (nDCG@10); these secondary numbers are for context.

## 7. Error Analysis

**Classification flips** (last seed): morphology fixed 129 test items (baseline wrong → morph correct) and broke 111 (net +18).

- HELPS  true=0  BPE=1  morph=0  "كنت انتظر اكثر من احلام مستغانمي في هذا الكتاب، ربما لاجل ذلك كان تقيمي منخفض الي هذه الدرجة. عموما لا انصح قراء مستغانمي الذين شغفتهم الثلا"
- HELPS  true=1  BPE=0  morph=1  "بسيطة مؤثرة ماساوية تستحق القراءة"
- HELPS  true=1  BPE=0  morph=1  "كتيب علي الرغم من صغره الا انه يمثل صدمة كبيرة لمن قراه بقلبه لا بعقله"
- HURTS  true=1  BPE=1  morph=0  "اذا اردت ان تعرف معني الكتابة الساخرة فاقرا لمحمد عفيفي :)"
- HURTS  true=1  BPE=1  morph=0  "عجبني جدا اسلوب الكاتب وثقافته المتنوعة وانه حكي اكتر من قصة باكتر من وجهة نظر واكتر من قصة مرتبطة بالتانية وعجبني انه وضح وجهة نظر اكتر من "
- HURTS  true=0  BPE=0  morph=1  "يعني عذرا من كل الذين "يهيصون" للكتاب، ما الذي اعجبكم فيه؟ لم اجد سوي فلسفة باولو المعتادة، فلسفته الغريبة طوال صفحات، وبعض السرد هنا وهناك."

**Retrieval per-query Δ nDCG@10**: morphology helps 23 queries and hurts 25 (mean Δ = -0.0194).

**Word-level tokenization examples**:

- HELPS  بمدرستنا  BPE=5 → morph=3  (ب مدرسة نا)
- HELPS  وللأولاد  BPE=5 → morph=4  (و ل ال أولاد)
- HURTS (over-seg)  فسيذهبون  BPE=3 → morph=4  (ف س يذهبون)
- HURTS (over-seg)  ومعلميهم  BPE=3 → morph=4  (و معلمي هم)
- HURTS (over-seg)  فبالكتب  BPE=3 → morph=4  (ف ب ال كتب)


**Readable examples for manual review** (auto-classified):

| Word | CAMeL segmentation | BPE tokens | Morph-BPE tokens | Auto comment |
|---|---|---|---|---|
| بمدرستنا | ب مدرسة نا | ب مد ر ست نا | ب مدرسة نا | good (fewer pieces) |
| وللأولاد | و ل ال أولاد | ول ل [UNK] ولا د | و ل ال أولاد | good (fewer pieces) |
| فسيذهبون | ف س يذهبون | فسي ذهب ون | ف س يذهب ون | good (more morphemes, all valid) |
| ومعلميهم | و معلمي هم | ومع لم يهم | و معل مي هم | good (more morphemes, all valid) |
| فبالكتب | ف ب ال كتب | ف ب الكتب | ف ب ال كتب | good (more morphemes, all valid) |
| المعلمون | ال معلمون | المع لم ون | ال معلم ون | neutral (same count, different split) |
| بالمدرسة | ب ال مدرسة | بالم در سة | ب ال مدرسة | neutral (same count, different split) |
| وكتابهم | و كتاب هم | و كتاب هم | و كتاب هم | neutral (same count, different split) |
| للأطفال | ل ال أطفال | لل [UNK] طفال | ل ال أطفال | neutral (same count, different split) |
| كتبتها | كتبت ها | كتب تها | كتبت ها | neutral (same count, different split) |
| يكتبون | يكتبون | يكتب ون | يكتب ون | neutral (same count, different split) |
| الطالبات | ال طالبات | الط الب ات | ال طال بات | neutral (same count, different split) |
| وقالوا | و قالوا | وق الوا | و قالوا | neutral (same count, different split) |
| كالأبطال | ك ال أبطال | كال [UNK] بطال | ك ال أبطال | neutral (same count, different split) |

_The 'Auto comment' column is a rule-based label (`good` / `questionable` / `neutral` / `redundant`); the supervisor is expected to refine these labels by hand on the final report._

## 8. Conclusion

### 8a. Hypothesis evaluation

| Hypothesis | Result | Supported? |
|---|---|---|
| H1: Morphology-aware BPE improves classification macro-F1 | Δ F1 = +0.0127, paired-t p = 0.0023 | **Yes** (significant) |
| H2: Morphology-aware BPE improves retrieval nDCG@10 | Δ nDCG@10 = -0.0194, paired-t p = 0.3896 | No (neutral or non-significant decline) |
| H3: Morphology-aware BPE changes token fertility | morph=1.688 vs BPE=1.402 pieces / original word (more pieces/word) | **Yes** (descriptive) |
| H4: Segmentation produces few suspicious short pieces (diagnostic) | suspicious=0.134, valid-clitic=0.118 | Diagnostic |

---

**Classification.** Morphology-aware BPE is positive and statistically significant (Δ macro-F1 = +0.0127, paired-t p = 0.0023).

At the example level on the last test set, the change is not statistically significant by McNemar's test (fixed 129, broke 111, p = 0.2725).

**Retrieval.** Morphology-aware BPE shows a negative trend on the primary metric nDCG@10 (Δ nDCG@10 = -0.0194, paired-t p = 0.3896).

Secondary retrieval metrics are mixed: improved on Precision@10 but declined on MRR, MAP, Hit@10. We report this transparently and anchor the headline conclusion to the pre-declared primary metric to avoid metric-shopping.

---
**Verdict bullets (auto-generated from Section 6):**

- Classification: morph-aware BPE shows a statistically significant positive effect for macro-F1 (Δ macro-F1 = +0.0127, paired-t p = 0.0023).
- Classification: morph-aware BPE shows a statistically significant positive effect for accuracy (Δ accuracy = +0.0127, paired-t p = 0.0023).
- Classification (example-level McNemar on the last seed): morph-aware BPE fixed 129 test examples and broke 111 (net +18); chi2 = 1.204, p = 0.2725, not statistically significant.
- Retrieval (PRIMARY = nDCG@10): morph-aware BPE shows a non-significant negative trend for nDCG@10 (Δ nDCG@10 = -0.0194, paired-t p = 0.3896).
- Retrieval secondary metrics are **MIXED**: improved on Precision@10 (Δ=+0.0010); declined on MRR (Δ=-0.0312), MAP (Δ=-0.0206), Hit@10 (Δ=-0.0400). The verdict above is anchored to the pre-declared primary metric (nDCG@10); these secondary numbers are for context.

## 9. Limitations

- Single morphological backend used in the final run: CAMeL Tools. A rule-based fallback exists only for development when REQUIRE_CAMEL=0, but it was not used in the reported results.
- BM25 is the only retrieval signal tested; dense retrieval is outside the scope of this BPE-focused comparison.
- LABR is small (~63k reviews after polarity mapping) and Arabic-specific; results may not transfer to other Arabic NLP tasks unchanged.
- MIRACL is evaluated by reranking the judged candidate pool per query rather than indexing the full multi-million passage corpus.
- Classification significance is tested over 5 seeds — adequate for trends, but small-sample paired t-tests are still indicative rather than definitive.
- Retrieval significance is computed across 100 evaluation queries after splitting MIRACL queries into tokenizer-training and evaluation sets. Larger query counts would provide more stable estimates.
