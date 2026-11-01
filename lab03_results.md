# Lab 03 — What RAG Is Made Of: results

Measured with the unchanged lab notebook (CPU run, `intfloat/multilingual-e5-small` for retrieval, `Qwen/Qwen2.5-1.5B-Instruct` for generation, greedy decoding). The predictions below were written into the notebook **before** the cells that reveal the answers were run.

## 1. Part 1 — chunk counts

| | Predicted | Measured |
|---|---|---|
| article-level | 4 | **4** |
| sentence-level | 22 (6+6+5+5) | **22** |

Right on both: the count is just the number of articles and the number of sentences.

## 2. Part 2 — retrieval, predicted vs measured

| Query | Level | Predicted | Measured |
|---|---|---|---|
| literal | article | `cards-lost` | `cards-lost` (0.869) — **correct** |
| literal | sentence | "instant card for 3000 tenge" (cards-lost #5) | top hit is **cards-lost #4** "replacement card is free of charge… 5 working days" (0.851); the needle #5 is a close second (0.848) — **wrong by 0.003** |
| paraphrase | article | `cards-lost` | **`cards-expiry`** (0.881) — **wrong** |
| paraphrase | sentence | moves to another cards-lost sentence | top hit is **cards-expiry #4** "…a new one then costs 1500 tenge" (0.864); cards-lost #4 is second (0.861); the needle #5 is not even in the top 3 — **change predicted, but the wrong direction** |

What happened: the paraphrase ("lost my wallet… what will it cost me?") shares almost no words with the needle, but it does share "cost" and the idea of a new card with the 1500-tenge sentence in a different article. The scores of the top three differ by about 0.004, so the ranking is close to a coin flip here.

## 3. Part 3 — generation

- **Without context:** "I'm sorry, but I cannot provide you with that information as it may violate our terms of service… contact their customer support." It did not invent a number (my prediction was wrong), but it also did not say "I don't know the bank's policy"; it gave a made-up reason for refusing.
- **With context (the needle sentence):** "The cost of an instant replacement card at the branch is 3000 tenge." Correct.

One sentence on the no-context answer: it had no way to know this fictional bank's fee, and instead of saying so it produced a confident but false explanation (a terms-of-service violation) that has nothing to do with the question.

Note: in Part 3 the needle is handed to the model by hand (`needle_idx`). That is the best case. With the retrieval result from the paraphrase query the model would have received the wrong sentence.

## 4. Chunking choice

For a corpus like this one I would chunk **by sentence**, because each question is answered by exactly one sentence and a whole-article chunk dilutes it with unrelated facts (and gives the generator more to confuse). The paraphrase result shows the price: sentence-level hits can land in the wrong article, so I would retrieve the top few sentences and keep their article id, not trust the single top hit.

## AI-use declaration

I used Claude (Anthropic) as a coding assistant for this lab. It: located the assignment and the teacher's repository through the Canvas and GitHub APIs; ran the unchanged notebook headless on CPU; drafted the predictions (written into the notebook before the results were revealed) and drafted this summary from the measured output. No numbers were edited by hand; all values above are copied from the executed notebook `lab03_what_is_rag.executed.ipynb`. I need to be able to explain every line of the code and each answer orally, as the course AI policy requires.
