# IMDb Dataset Scraping

> **IMDb rating project series:** **Part 1: Dataset scraping** (this repo) · [Part 2: Rating prediction](https://github.com/roeiraz12/imdb-rating-prediction) · [Part 3: Predictor web app](https://github.com/roeiraz12/imdb-rating-predictor-app)

Collecting data from the web to create a large movie database

Collecting data from the web to create a large movie database

<div dir="rtl">

**תקציר:** חלק א' של פרויקט כריית נתונים (שנה ג'). בניית מאגר של 5,000 סרטים משילוב הטבלאות הציבוריות של IMDb עם נתונים שנאספו מוויקיפדיה ב-Web Scraping (תקציר עלילה, שפה, מדינה, תקציב והכנסות). המאגר שנוצר כאן משמש בחלקים ב' ו-ג' לחיזוי דירוג IMDb.

כדי להריץ את הקוד בהצלחה יש להוריד ולשמור באותה תיקייה של מחברת הקוד את הטבלאות הציבוריות של IMDb הזמינות בכתובת:
https://developer.imdb.com/non-commercial-datasets/
הטבלאות הרלוונטיות לאיסוף:
•	title.basics — title.basics.tsv.gz
•	title.ratings — title.ratings.tsv.gz
•	title.principals — title.principals.tsv.gz
•	name.basics — name.basics.tsv.gz

בהצלחה!

</div>

---

## Overview

Part 1 of a third-year Data Mining course project. The goal was to build a movie dataset from scratch: the IMDb non-commercial datasets supply the core fields (title, year, genres, runtime, rating, votes, lead actors), and Python web scraping of Wikipedia adds fields IMDb does not provide (plot, language, country, budget, box office). The result is `all_data.csv` – 5,000 movies × 13 columns – which is the input for the modeling work in Parts 2 and 3.

## Project story

- **Problem.** The later parts of the project predict a movie's IMDb rating, which needs a dataset that combines
  IMDb's structured fields with descriptive fields IMDb's public tables do not have, such as the plot, language,
  country, budget and box office.
- **Approach.** Filter and join the IMDb non-commercial tables, then scrape Wikipedia for the missing fields with a
  fallback chain of URL variants, parse the infoboxes, turn money text into numbers, and merge and validate
  everything into one CSV.
- **Context and role.** Academic data-science project for a third-year Data Mining course. This repository
  covers the data-collection stage, from the raw IMDb files to the final `all_data.csv` used in Parts 2 and 3.

## What was done

1. **Filtering IMDb `title.basics`** – kept titles where `titleType == "movie"`, start year < 2025, runtime between 60 and 300 minutes, and the title starts with "E" or "F". An extra filter (my addition) kept only titles made of English letters and spaces, because non-English titles were hard to find on Wikipedia.
2. **Joining the other IMDb tables on `tconst`** – ratings and votes from `title.ratings`; the top 5 billed actors/actresses from `title.principals`, read in chunks of 1M rows to keep memory low. Incomplete rows were dropped and the first 5,000 movies were kept.
3. **Web scraping Wikipedia** (`requests` + `BeautifulSoup`) – for every movie the scraper tries several URL variants in order (`Title_(YEAR_film)`, `Title_(film)`, plain title, title without commas, swapping a leading "The"/"A", and Swedish Wikipedia as a last resort) and skips disambiguation pages. It extracts:
   - the plot (lead section),
   - language and country (infobox),
   - budget and box office (infobox), converted from text like "$5 million" or ranges like "$10–15 million" into numbers.
4. **Merging and validation** – each scraped field was saved to its own CSV and then merged with the IMDb table into `all_data.csv`, followed by completeness and data-type checks.

## Tech stack

Python · Jupyter Notebook · pandas · requests · BeautifulSoup (bs4) · regular expressions · tqdm

## Results

From the notebook outputs (`Project_part_1.ipynb`):

- **5,000 movies** collected, released between **1912 and 2009**; mean IMDb rating **5.93** (min 1.8, max 9.5).
- Field completeness in the final dataset:

| Field | Complete |
|---|---|
| IMDb fields (title, year, genres, runtime, rating, votes, lead actors) | 100% |
| Plot | 100% |
| Language | 92.94% |
| Country | 87.28% |
| Budget | 11.94% |
| Box office | 13.42% |

Budget and box office are sparse: many Wikipedia infoboxes for these (mostly older, lesser-known) films do not list them.

## Repository structure

| File | Description |
|---|---|
| `Project_part_1.ipynb` | Full pipeline: IMDb filtering and joins, Wikipedia scraping, merge and validation |
| `all_data.csv` | Final dataset – 5,000 movies, 13 columns |
| `דוח פרויקט חלק א.docx` | Project report (Hebrew): design decisions, scraper decision tree, field descriptions |

## How to run

1. Install the dependencies: `pip install pandas requests beautifulsoup4 tqdm jupyter`
2. Download the IMDb files listed above from <https://developer.imdb.com/non-commercial-datasets/> and place them next to the notebook.
3. Open `Project_part_1.ipynb` and run the cells in order. The Wikipedia scraping steps make thousands of web requests and take a long time (the plot step alone took about 3 hours), so the ready-made `all_data.csv` is included in the repo.

## What I'd do next

- Read budget, box office, language and country from Wikidata or the Wikipedia API instead of parsing HTML
  infoboxes, to raise the low budget and box-office coverage (about 12-13%).
- Cache responses and run the scraper concurrently with rate limiting, since the plot step alone took about 3 hours.
- Sample titles across all letters and years instead of titles starting with "E" or "F", so the dataset is more
  representative and includes recent films.
- Convert budgets to one currency and adjust them for inflation before modeling.
- Move the notebook steps into scripts with a few tests, so the dataset can be rebuilt with one command.
