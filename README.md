# IMDB Movie Dataset Analysis & Recommendation System

**A Spark-based analytical pipeline and interactive recommendation app leveraging a 4K-movie IMDB-derived dataset (1980–2000) for exploratory insights and lightweight content-based recommendations.**

---
## 🧠 Abstract
This project conducts a comprehensive exploratory and analytical study of a curated IMDB movie dataset (≈4,000 films, 13 attributes) to derive production, talent, geographic, financial, and temporal insights while delivering a minimal viable recommendation engine. Using Apache Spark for distributed preprocessing and analytical aggregation, we engineer a cleansed, reliable dataset (handling nulls, duplicates, and zero-value artifacts) and compute multi-faceted KPIs: profitability, audience preference (IMDb score, votes), creative influence (director/star consistency), temporal evolution (runtime, score trends), and corporate performance (studio output & profit). A Streamlit + PySpark interface exposes genre-, star-, and production-house-based Top‑5 recommendations using a deterministic, content-aware filtering heuristic. The system serves as an extensible foundation for future collaborative filtering (ALS) or hybrid recommenders and illustrates end-to-end data engineering, analytical modeling, and user-facing delivery within a scalable architecture.

---
## 📌 Table of Contents
1. Introduction / Motivation  
2. Dataset Description  
3. Architecture & Pipeline  
4. Data Preprocessing  
5. Analytical Methodology  
6. Recommendation Engine  
7. Results & Key Findings  
8. Challenges & Solutions  
9. Key Insights  
10. Future Enhancements  
11. Tech Stack  
12. Getting Started  
13. Repository Structure  
14. References

---
## 1. Introduction / Motivation
Content discovery remains a central problem in media platforms. While large-scale streaming services deploy complex hybrid recommenders, mid-scale analytical workflows still benefit from interpretable, data-driven foundations. This project targets:  
- Curating a clean analytical layer from semi-structured movie metadata.  
- Deriving executive-ready KPIs (creative success, financial leverage, geographic advantage).  
- Demonstrating a transparent, content-based recommendation interface.  
- Establishing a modular Spark workflow adaptable to advanced modeling (ALS, embeddings).

---
## 2. Dataset Description
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
Target Variables (implicit): Profit (gross − budget), score (quality proxy), votes (engagement proxy).  

---
## 3. Architecture & Pipeline
```mermaid
graph TD;
  A[Raw CSV] --> B[Spark Ingestion]
  B --> C[Data Cleaning\n(null/duplicate/zero filtering)]
  C --> D[Analytical Aggregations\n(KPIs, Top-N, Trends)]
  C --> E[Feature Layer\n(genre, star, company, profit)]
  E --> F[Content-Based Recommender]
  D --> G[Visualization / Insights]
  F --> H[Streamlit UI]
```

---
## 4. Data Preprocessing
Steps executed in Spark (spark_processing.sc):  
1. Null Handling: rows with nulls in critical analytical columns removed.  
2. Duplicate Audit: duplicate count check; duplicates eliminated if present.  
3. Zero-Value Filtering: removal of rows with nonsensical zeros in monetized or runtime fields.  
4. Profit Engineering: profit = gross − budget (used in company/star/country profitability rankings).  
5. Temporal Parsing: release date converted to year & month aggregates for longitudinal trend plots.  
6. Runtime Categorization: Bucketing relative to global mean for preference analysis.

---
## 5. Analytical Methodology
Spark SQL & DataFrame API operations: groupBy(), agg(), avg(), sum(), count(), orderBy(), limit(), window functions.  
Key Analytical Segments:  
- Top‑10 by IMDb score, director consistency, star performance.  
- Company production volume vs average quality.  
- Profit concentration by company & country.  
- Temporal trends: quality vs year, runtime evolution, release seasonality.  
- Collaboration profit networks (company–star, company–director pairs).  
- Certification impact on score & volume.  

---
## 6. Recommendation Engine
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
## 7. Results & Key Findings
(Representative illustrative summaries)

### 7.1 Top Performance Samples
| Category | Metric | Example Outcome |
|----------|--------|-----------------|
| Directors | Avg Score | High-score cluster among select auteurs |
| Stars | Avg Score | Stable quality for recurring bankable leads |
| Companies | Total Profit | Profit skew toward a few major studios |
| Countries | Profit | Dominance of US; emerging secondary markets |

### 7.2 Temporal Trends
| Trend | Observation |
|-------|------------|
| Average Runtime | Moderate variance; clustering around commercial length |
| Quality Over Years | Stability with mild upward drift in mid-1990s |
| Release Month Distribution | Peaks in summer & holiday corridors |

### 7.3 Collaboration Profitability
| Pair Type | Insight |
|-----------|---------|
| Company–Star | Certain actor partnerships yield multi-billion cumulative profit |
| Company–Director | Director franchises correlate with sustained profitability |

---
## 8. Challenges & Solutions
| Challenge | Impact | Mitigation |
|-----------|--------|------------|
| Incomplete Rows | Skewed aggregates | Strict null-drop on critical columns |
| Zero Budgets/Gross | Distorted profit metrics | Filtered anomalous zeros |
| Date Parsing Variants | Temporal trend accuracy | Uniform dd-MMM-yy parsing & year extraction |
| Memory During Local Runs | Shuffle overhead | Column pruning & lazy evaluation awareness |
| Lack of Explicit User Ratings | Limiting recommender sophistication | Content-based proxy + future ALS plan |

---
## 9. Key Insights
1. Profit highly concentrated: a minority of studios account for disproportionate cumulative earnings.  
2. High average director score correlates with moderate but consistent runtime bands (no strong incentive for extreme length).  
3. Star profitability networks suggest repeatable casting strategies.  
4. Genre high-scorers cluster in quality-driven niches (e.g., Drama/Thriller) while high-vote films align with mainstream genres (Action/Adventure).  
5. Certification mix influences both volume and average score (stricter certifications show niche but sometimes higher score variance).  

---
## 10. Future Enhancements
- Implement ALS implicit-factor model using votes as confidence-weighted signals.  
- Add plot-level or synopsis NLP embeddings (Sentence-BERT) for semantic similarity.  
- Integrate MySQL persistence layer for curated dimensional model (fact_movie, dim_company, dim_person).  
- Introduce MLOps pipeline (Delta Lake + scheduled batch retraining).  
- Deploy via containerization (Docker + Spark cluster) & CI (GitHub Actions).  
- Add fairness & bias audits (e.g., geographic representation).  

---
## 11. Tech Stack
| Layer | Tools |
|-------|-------|
| Data Processing | Apache Spark (Scala & PySpark) |
| Language | Scala, Python |
| Interface | Streamlit |
| Storage (current) | Local CSV (extensible to MySQL) |
| Visualization | Streamlit Tables (extensible to Tableau) |
| Orchestration (future) | Airflow / Prefect (planned) |

---
## 12. Getting Started
### Prerequisites
- Python 3.9+
- Java 8/11 & Spark installed
- pip install streamlit pyspark

### Run Analytics (Scala REPL / spark-shell)
```bash
spark-shell -i main/scala/spark_processing.sc
```

### Launch Recommendation App
```bash
streamlit run movie_recommadation.py
```

### Environment Variables (Optional)
- SPARK_DRIVER_MEMORY=4g  
- JAVA_HOME configured appropriately.

---
## 13. Repository Structure
```
├── main/scala/
│   ├── Main.scala              # Entry placeholder
│   ├── spark_processing.sc     # Spark preprocessing & analytics
│   └── movies.csv              # Source dataset (local)
├── movie_recommadation.py      # Streamlit recommender app
├── README.md                   # Project documentation
└── FINAL REPORT.pdf / PPT_SLIDES.pptx
```

---
## 14. References
- Apache Spark Documentation  
- Recommender Systems Handbook (Ricci et al.)  
- Implicit Feedback Matrix Factorization (Hu et al., 2008)  
- Kaggle IMDB-like movie metadata sources  

---
## Citation
If you use this project as a baseline, please cite:  
Movie Dataset Analysis & Recommendation System (2025). Apache Spark Analytical Pipeline + Content-Based Recommender.

---
## License
Educational / research use. Extend with proper attribution.
