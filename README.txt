AI ASSIGNMENT 2026 - CLASSICAL REASONING SYSTEM
================================================

FILES
-----
reasoning_agent.ipynb   Full implementation (engine, translators, evaluation, submission writer)
report.pdf              Short report: representation, algorithm, experiments, reasoning traces
submission.csv         Predictions for the test file (columns: id, label)
README.txt              This file

WHAT THE SYSTEM DOES
--------------------
Input: a knowledge base (context) and a query (statement).
Output: True or False (is the query entailed by the knowledge base?).

Pipeline: text -> router -> translator -> facts/rules/interventions/goal
          -> forward chaining to a fixpoint -> True/False

One reasoning engine serves RuleTaker, CounterBench and FOLIO. Only the
translator in front of the engine differs per dataset. The translators are
regular-expression templates. No generative AI is used; the final decision
always comes from the classical forward-chaining engine.

REQUIREMENTS
------------
Python 3 (developed and run on Google Colab).
Libraries: datasets, huggingface_hub, pandas. The rest is the standard
library (re, time, csv, collections).

HOW TO RUN
----------
1. Open reasoning_agent.ipynb (Google Colab or Jupyter).
2. Run all cells from the top, in order.
   - The first cell downloads the public datasets from Hugging Face
     (jise/ruletaker, CounterBench/CounterBench with data_balanced_alpha_V1.json,
     yale-nlp/FOLIO). It needs internet access and, in Colab, a Hugging Face
     login prompt (notebook_login).
   - The evaluation cells print accuracy and total inference time on the
     public data. Expected output:
        RuleTaker     98.3%  (983/1000)
        CounterBench  96.0%  (960/1000)
        FOLIO         61.3%  (614/1001)
3. Generating submission.csv from a test file:
   - In the LAST cell of the notebook, set TEST_PATH to the test file
     (.csv, .tsv, .json, .jsonl or .parquet) and run that cell.
   - The cell finds the columns by name:
        context column:    context / given_info / premises
        statement column:  statement / question / conclusion
        id column:         id / question_id / example_id (row number if absent)
   - It writes submission.csv with columns "id,label" (label is True or False),
     prints the number of rows, the number of unreadable rows (should be 0)
     and the total inference time.
   - The reasoning engine itself does not need the public datasets, so this
     cell works even if the dataset download is unavailable.

FINAL SETTINGS USED
-------------------
NEG_MODE      = "naf"    negated rule conditions use negation as failure
NESTED_MODE   = "both"   nested counterfactuals apply both interventions
FOLIO_DEFAULT = False    FOLIO answer when the conclusion cannot be derived
(The alternatives were compared on the public data; see report.pdf.)

ASSUMPTIONS
-----------
- FOLIO label "Uncertain" is treated as "not entailed" (False).
- In CounterBench, root variables occur unless the context says they were
  observed to be absent; "A causes not B" means B holds when A does not.
- The router picks the translator from the text of the input (Facts:/Rules:
  markers for RuleTaker, the words "cause" and "occur" for CounterBench,
  anything else is treated as FOLIO).

KNOWN LIMITATIONS
-----------------
- FOLIO accuracy (61.3%) equals the always-False baseline: the translator
  covers only simple "X is a Y" facts and "All A are B" rules.
- The translators are templates for the formats in the public data. A test
  set with different phrasing would need new templates; sentences a
  translator cannot read are skipped.
- About 4% of CounterBench questions are wrong; the inspected errors involve
  an observation on a non-root variable together with a negated cause.

The private test set was not used during development.
