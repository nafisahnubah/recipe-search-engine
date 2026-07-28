# Recipe Search Engine

A from-scratch exploration of classic information-retrieval techniques, applied to a
database of ~50,000 recipes. Given a query like `butter sugar`, the goal is to retrieve
the most relevant recipes — and to compare several retrieval strategies head-to-head on
both **speed** and **result quality**.

## What's implemented

| Method | Idea | Notes |
|---|---|---|
| Boolean incidence matrix | `AND` / `OR` / `NOT` over a term-document matrix | Implemented both with scikit-learn's sparse `CountVectorizer` and as a hand-rolled dense NumPy matrix, for comparison |
| Inverted index | Classic postings-list lookup | Built with an explicit map → group → reduce pipeline |
| BST-backed inverted index | Same idea, indexed via a binary search tree | Tree implemented from scratch, no external library |
| Jaccard similarity | Ranks by set overlap between query and document terms | |
| TF-IDF | Ranks by cosine similarity over TF-IDF vectors | scikit-learn `TfidfVectorizer` |
| Whoosh / BM25F | Full-text search index with production-style scoring | Uses [Whoosh](https://whoosh.readthedocs.io/), a pure-Python search engine library |

Every method is benchmarked against the same queries so the latency numbers are directly
comparable, and the notebook ends with a written comparison of both speed and ranking
quality across all six approaches.

## Results

On a 50k-recipe corpus, service latency (averaged over multiple runs) breaks down roughly
as:

```
Boolean (dense, hand-rolled):  ~0.003s   <- fastest (no scoring, just postings intersection)
BST inverted index:            ~0.006s
Boolean (sklearn, sparse):     ~0.015s
Whoosh (BM25F):                ~0.016s
TF-IDF:                        ~0.030s
Jaccard similarity:            ~0.072s   <- slowest (scores every match)
```

Pure postings-lookup methods (Boolean search, inverted index) are the fastest since they
do no scoring — but they also can't rank results. TF-IDF and BM25F are slower per-query
but return meaningfully ranked results, with Whoosh's BM25F producing the most focused
top results thanks to term-frequency saturation and document-length normalization.

See the notebook for the full written analysis.

## Getting started

```bash
pip install -r requirements.txt
```

Then place `food_recipes.csv` in the `data/` directory (see [`data/README.md`](data/README.md))
and run `notebooks/recipe_search_engine.ipynb` top to bottom.

## Project structure

```
recipe-search-engine/
├── notebooks/
│   └── recipe_search_engine.ipynb   # all the code, results, and analysis
├── data/
│   └── README.md                    # where to get the dataset
├── requirements.txt
└── README.md
```

## Tech stack

Python · pandas · NumPy · scikit-learn · NLTK · Whoosh
