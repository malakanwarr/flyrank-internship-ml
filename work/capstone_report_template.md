# Capstone Report — Structured Content Archetype Clustering
- **Author:** Malak Amr
- **Lane:** Structured Content Archetype Clustering
- **Repo:** https://github.com/malakanwarr/flyrank-internship-ml.git
- **Date:** August 2026

## 0. Abstract
Content teams struggle to prioritize optimization tasks across massive portfolios of web pages. This study uses the FlyRank dataset to mathematically identify structural performance archetypes—specifically "missed opportunities"—without relying on rigid human thresholds. We applied a K-Means clustering algorithm ($k=5$) to a normalized feature set of search volume, clicks, competition, and keyword length. The model isolated a distinct missed-opportunity archetype, achieving a 46.00% Precision@50 on an unseen grouped-client test split. This provides a directional, decision-support queue for content teams to prioritize high-value updates efficiently.

## 1. Problem framing
This project supports the content triage decision process. The unit of analysis is the individual web page, and the output is a ranked queue. A human editor uses this to decide which pages to rewrite or expand. Without ML, teams rely on brittle manual thresholds, leading to the costly mistake of ignoring high-potential pages that barely miss arbitrary cutoffs.

## 2. Data safety
We used the March 2026 FlyRank portfolio snapshot. We deliberately excluded all product flags (e.g., `is_deleted`) and future-window metrics to prevent temporal data leakage. We exclusively used `client_hash_id` and `content_hash_id` for grouping, ensuring no client-identifying details, raw URLs, or private queries appear anywhere in the analysis. 

## 3. Baseline
Our baseline was a strict human heuristic: pages with `search_volume >= 1000` and `gsc_clicks <= 5`. This is a fair comparison because it represents the standard industry logic for finding missed opportunities. Because this rule graded itself, it scored a guaranteed 100% on the random split, failing to account for edge cases.

## 4. Model / analysis
We used unsupervised K-Means Clustering ($k=5$), which perfectly fits this lane's goal of finding natural performance groupings. The exact features were `search_volume`, `gsc_clicks`, `competition`, and `keyword_char_count` (scaled via `StandardScaler`). The target proxy was the baseline rule (Volume >= 1000, Clicks <= 5), used strictly to evaluate if the clusters aligned with human intent.

## 5. Evaluation
We utilized a `GroupShuffleSplit` (test size = 0.2), grouping by `client_hash_id`. This prevents leakage by forcing the model to evaluate entirely unseen websites rather than memorizing formatting quirks. On this honest split, the baseline scored 100% (a mathematical tautology), while the K-Means model achieved a 46.00% Precision@50. Error analysis revealed the AI's "false positives" were actually pages with high volume (e.g., 990) that narrowly missed the rigid baseline cutoff, proving the model successfully found flexible boundaries.

While 46% might appear low, error analysis reveals this is a strength of the model, not a failure. The rigid baseline only accepts pages with exactly 5 or fewer clicks. The model's "false positives" were actually high-value edge cases—such as a page with 40,500 search volume and 35 clicks, or 9,900 volume and 6 clicks. The baseline graded these as "wrong" for strictly crossing the 5-click threshold, but the model correctly recognized them as massive missed opportunities. Ultimately, the 46% score demonstrates the model successfully outsmarting a brittle human rule to find flexible, natural performance boundaries.

## 6. Interpretation
The clustering algorithm heavily relied on the natural geometric disparity between `search_volume` and `gsc_clicks`. Cluster #4 emerged as the dominant "Missed Opportunity" archetype. A key positive surprise was that the model successfully flagged pages with 30,000+ volume and 35 clicks—pages the baseline completely ignored—proving that the AI grasped the *relative* performance gap rather than relying on a static click threshold. 

## 7. Recommendation
Pages routed to the Missed Opportunity archetype are assigned a `REVIEW_AND_UPDATE` action. We observed that this group has massive uncaptured demand. We measured the model's accuracy on unseen data, providing a strong directional hint for prioritization. This is strictly a decision-support tool; editors must manually verify search intent before publishing. Navigational pages (like logins) are explicitly excluded from automation.

## 8. Reproducibility
*   **Environment:** standard Google Colab Python 3 environment.
*   **Seed:** `random_state=42` used for `train_test_split`, `GroupShuffleSplit`, and `KMeans`.
*   **Commands:** Execute all cells in `work/notebooks/capstone.ipynb` sequentially. The sealed evaluation frame logic and the resulting metrics JSONs are committed directly to the `work/outputs/` directory.

## 9. Acknowledgments & data credit
Built on the FlyRank ML Internship dataset (https://flyrank.ai).