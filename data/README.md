# Dataset

This project expects a `food_recipes.csv` file in this directory with at least the
following columns:

- `title` — the recipe name
- `directions` — the recipe's preparation steps, as free text

A suitable dataset (recipe title + directions, tens of thousands of rows) can be sourced
from any of the many recipe datasets on [Kaggle](https://www.kaggle.com/datasets) — search
for "recipe dataset" and pick one with a `directions`/`instructions` text column, renaming
columns to match the two above if necessary.

Place the CSV here as `data/food_recipes.csv` before running the notebook.
