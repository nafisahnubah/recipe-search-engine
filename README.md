# Recipe Search Engine

A from-scratch exploration of classic information-retrieval techniques, applied to a
database of ~50,000 recipes. Given a query like `butter sugar`, the goal is to retrieve
the most relevant recipes, and to compare several retrieval strategies head-to-head on
both **speed** and **result quality**.

## What's implemented

| Method | Idea | Notes |
|---|---|---|
| Boolean incidence matrix | `AND` / `OR` over a term-document matrix | Implemented both with scikit-learn's sparse `CountVectorizer` and as a hand-rolled dense NumPy matrix, for comparison |
| Inverted index | Classic postings-list lookup (all query terms must match) | Built with an explicit map → group → reduce pipeline |
| BST-backed inverted index | Same idea, indexed via a binary search tree | Tree implemented from scratch, no external library |
| Jaccard similarity | Ranks matching recipes by set overlap between query and recipe words | |
| TF-IDF | Ranks by cosine similarity over TF-IDF vectors | scikit-learn `TfidfVectorizer` |
| Whoosh / BM25F | Full-text search index with production-style scoring | Uses [Whoosh](https://whoosh.readthedocs.io/), a pure-Python search engine library; indexes a 10% random sample of the recipes |

Every method is run on the same query (`butter` + `sugar`), and the notebook ends with a
written comparison of speed and ranking quality across all six approaches.

## Results

Search latency from the notebook's saved run (machine-dependent):

```
Boolean (dense, hand-rolled):  ~0.003s   <- fastest (column lookup + bitwise AND, no scoring)
BST inverted index:            ~0.006s
Boolean (sklearn, sparse):     ~0.015s
Whoosh (BM25F):                ~0.016s   (10% sample of the recipes)
TF-IDF:                        ~0.030s
Jaccard similarity:            ~0.072s   <- slowest (scores every match)
```

The BST, Whoosh, TF-IDF and Jaccard figures are averages over 10 runs; the two Boolean
figures are from a single run.

Pure lookup methods (Boolean search, inverted index) are the fastest since they do no
scoring, but they also can't rank results. TF-IDF and BM25F are slower per query but
return ranked results, with Whoosh's BM25F producing the most focused top results thanks
to term-frequency saturation and document-length normalization.

See the notebook for the full written analysis.

## Requirements

- Python 3 with Jupyter (JupyterLab, Jupyter Notebook, or VS Code)
- Internet access on first run: the setup cell downloads NLTK's `stopwords`, `wordnet`
  and `omw-1.4` data
- A recipe dataset saved as `data/food_recipes.csv` with `title` and `directions`
  columns (not included; see [`data/README.md`](data/README.md))

## Getting started

```bash
pip install -r requirements.txt
```

Place `food_recipes.csv` in the `data/` directory, then start Jupyter from the
`notebooks/` folder and run `recipe_search_engine.ipynb` top to bottom. The notebook
reads `../data/food_recipes.csv` and writes its Whoosh index to
`notebooks/whoosh_recipe_idx/` (git-ignored).

## Known limitations

- **Boolean `NOT`:** only the postfix form works (`butter NOT` returns recipes without
  butter). `NOT butter` and `butter AND NOT sugar` raise a `ValueError`, as does a query
  of plain terms with no operator (`butter sugar`).
- **Query terms aren't normalized** for the inverted index, BST and Jaccard searches.
  Recipe text is lowercased and lemmatized but queries are not, so `eggs` finds nothing
  (the index stores `egg`). TF-IDF does normalize its query.
- **Jaccard similarity is computed on the raw `directions` text** rather than the
  cleaned text, so punctuation and capitalization (`sugar.`, `Butter`) can stop a word
  from counting as a match.

## Project structure

```
recipe-search-engine/
├── notebooks/
│   └── recipe_search_engine.ipynb   # all the code, results, and analysis
├── data/
│   └── README.md                    # where to get the dataset
├── requirements.txt
├── LICENSE
└── README.md
```

## Tech stack

Python · pandas · NumPy · scikit-learn · NLTK · Whoosh

## License

[MIT](LICENSE)
