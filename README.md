# 🎬 Movie Dataset Analysis & Recommendation System with Apache Spark

**A Spark-based analytical pipeline and interactive recommendation app leveraging a 4K-movie IMDB-derived dataset (1980–2000) for exploratory insights and lightweight content-based recommendations.**

<p align="center">
  <img src="image (26).png" alt="Patrol Robot" width="1000"/>
</p>


---
## 🎯 Project Goals
The primary objective is to build a live movie dashboard and recommendation system. This is broken down into the following key goals:

- **Data Engineering**: Ingest, clean, and structure the raw Kaggle dataset into a robust MySQL database, making it persistent and queryable.
- **Distributed Analysis**: Leverage Apache Spark and its DataFrame API to perform large-scale Exploratory Data Analysis (EDA) and complex aggregations that would be inefficient on a single machine.
- **Insight Generation**: Answer specific business-driven questions, such as identifying top-performing directors, financially successful genres, and high-profit-margin production companies.
- **System Development**: Build a functional, content-based recommendation engine that suggests movies to users based on their preferences.
- **Visualization**: (Goal) Create a user-facing UI (dashboard) to present the analytical findings and host the recommendation system.

---
## Dataset Description
Source: Kaggle-derived IMDB-style dataset (local copy).  
Scope: Films released 1980–2000.

| Attribute | Description | Type | Example |
|----------|-------------|------|---------|
| title | Movie title | String | "Jurassic Park" |
| rating | Age certification | Categorical | PG-13 |
| genre | Primary genre | String | Action |
| released | Release date (dd-MMM-yy) | String | 11-Jun-93 |
| score | IMDb-like rating (0–10) | Float | 8.1 |
| votes | User votes | Integer | 900000 |
| director | Director name | String | Steven Spielberg |
| star | Lead actor | String | Leonardo DiCaprio |
| country | Production country | String | USA |
| budget | Production budget | Long | 63000000 |
| gross | Box office gross | Long | 1040000000 |
| company | Production company | String | Universal Pictures |
| runtime | Duration (minutes) | Integer | 127 |

Size: ~4,000 rows × 13 columns (post-cleaning slightly reduced).  
---
## Data Preprocessing
Steps executed in Spark (spark_processing.sc):  
1. Null Handling: rows with nulls in critical analytical columns removed.  
2. Duplicate Audit: duplicate count check; duplicates eliminated if present.  
3. Zero-Value Filtering: removal of rows with nonsensical zeros in monetized or runtime fields.  
4. Profit Engineering: profit = gross − budget (used in company/star/country profitability rankings).  
5. Temporal Parsing: release date converted to year & month aggregates for longitudinal trend plots.  
6. Runtime Categorization: Bucketing relative to global mean for preference analysis.

---
## Analytical Methodology
Spark SQL & DataFrame API operations: groupBy(), agg(), avg(), sum(), count(), orderBy(), limit(), window functions.  
Key Analytical Segments: 

| 📈 Focus Area                 | 🔍 Insight Objective                                          |
| ----------------------------- | ------------------------------------------------------------- |
| 🎯 **Top-10 Rankings**        | IMDb score leaders, director consistency, star performance    |
| 🏢 **Studio Analysis**        | Company production volume vs average quality                  |
| 💵 **Profit Clusters**        | Concentration by company & country                            |
| 📅 **Temporal Trends**        | Yearly quality shifts, runtime evolution, release seasonality |
| 🤝 **Collaboration Networks** | Company–Star & Company–Director profit pairing                |
| 🧾 **Certification Impact**   | How ratings affect score & volume                             |


---
## Recommendation Engine

<p align="center">
  <img src="image (27).png" alt="Patrol Robot" width="1000"/>
</p>

Type: Deterministic content-based filtering (genre/star/production house).  
Logic:  
- Movie-based: infer genre of selected title → rank peers in same genre by score (top 5).  
- Star-based: filter by actor → top 5 by score.  
- Production House-based: filter by company → top 5 by score.  
Framework: Streamlit + PySpark session (movie_recommadation.py).  
Potential Extensions:  
- Collaborative Filtering (ALS) on (user, movie, rating) if implicit feedback derived from votes.  
- Hybrid fusion: content embeddings (TF-IDF over synopsis if added) + latent factors.  
- Diversity-aware re-ranking (MMR) & popularity bias correction.

Example Simplified Pseudocode:
```python
# Select Top-5 genre recommendations
genre = df.filter(col("title") == input_title).select("genre").first()[0]
recs = (df.filter(col("genre") == genre)
          .orderBy(col("imdb_rating").desc())
          .limit(5))
```

---
## Results & Key Findings
(Representative illustrative summaries)

###  🎬 Company-Based Analysis: Top Actor Collaborations by Profit

This analysis identifies the most profitable collaborations between production companies and their lead actors, based on total cumulative profit (Gross - Budget).

| Production Company | Star | Total Profit (USD) |
| :--- | :--- | :--- |
| Twentieth Century Fox | Leonardo DiCaprio | $2,001,647,264 |
| Paramount Pictures | Tom Cruise | $1,276,761,018 |
| Universal Pictures | Sam Neill | $1,246,709,112 |
| Paramount Pictures | Harrison Ford | $1,177,913,541 |
| Paramount Pictures | Eddie Murphy | $1,152,676,159 |
| Walt Disney Pictures | Matthew Broderick | $1,083,123,989 |
| Lucasfilm | Mark Hamill | $962,981,244 |
| Lucasfilm | Ewan McGregor | $912,082,707 |
| Universal Pictures | Michael J. Fox | $882,585,318 |
| Warner Bros. | Daniel Radcliffe | $881,968,171 |


### 🌟 Top 10 Most Profitable Actor-Company Collaborations

This table highlights the most financially successful partnerships between production companies and lead actors, ranked by total profit generated.

| Production Company | Star | Total Profit (USD) |
| :--- | :--- | :--- |
| Twentieth Century Fox | Leonardo DiCaprio | $2,001,647,264 |
| Paramount Pictures | Tom Cruise | $1,276,761,018 |
| Universal Pictures | Sam Neill | $1,246,709,112 |
| Paramount Pictures | Harrison Ford | $1,177,913,541 |
| Paramount Pictures | Eddie Murphy | $1,152,676,159 |
| Walt Disney Pictures | Matthew Broderick | $1,083,123,989 |
| Lucasfilm | Mark Hamill | $962,981,244 |
| Lucasfilm | Ewan McGregor | $912,082,707 |
| Universal Pictures | Michael J. Fox | $882,585,318 |
| Warner Bros. | Daniel Radcliffe | $881,968,171 |

###  🌎 Top 10 Countries by Total Movie Profit

This table shows the cumulative box office profit (Gross - Budget) generated by movies, aggregated by their primary country of production.

| Country | Total Profit (USD) |
| :--- | :--- |
| United States | $7,771,014,765 |
| United Kingdom| $3,375,794,849 |
| Australia | $933,226,691 |
| Japan | $878,975,550 |
| New Zealand | $823,613,493 |
| Germany | $701,578,804 |
| Hong Kong | $323,710,749 |
| France | $301,733,573 |
| Spain | $203,324,542 |
| Taiwan | $202,459,195 |

---
## Key Insights
1. Profit highly concentrated: a minority of studios account for disproportionate cumulative earnings.  
2. High average director score correlates with moderate but consistent runtime bands (no strong incentive for extreme length).  
3. Star profitability networks suggest repeatable casting strategies.  
4. Genre high-scorers cluster in quality-driven niches (e.g., Drama/Thriller) while high-vote films align with mainstream genres (Action/Adventure).  
5. Certification mix influences both volume and average score (stricter certifications show niche but sometimes higher score variance).  

