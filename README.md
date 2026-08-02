# Python-Machine-Learning----Rental-Tier-Classification-Irish-Property-Market
## Business Problem: 
**Context:** The firm develops residential properties and prices them across a tiered scale Budget, Standard and Luxury to target distinct buyer and renter segments. Before committing capital to a site acquisition or development, the firm needs to determine which tier a proposed property will fall into and what market-supported rent or price it can achieve.

Currently, pricing decisions rely on senior staff experience and expensive periodic third-party valuations. This process is slow, inconsistent across sites, and difficult to scale as the firm expands into new counties.

**Scope clarification:** This model uses rental price data as the primary pricing signal. Rental prices in Ireland are strongly correlated with sale prices at the county and bedroom-type level, making them a valid and data rich proxy for development pricing, particularly for build-to-rent and private rental sector schemes which are a growing part of the Irish market. The model is therefore framed as a rental tier classifier, directly applicable to private rental sector development appraisals.

**Business Risks Without a Model:**
- **Over-pricing:** Properties sit vacant, tying up capital and incurring holding costs.
- **Under-pricing:** Margin is left on the table, reducing project ROI.
- **Misclassification:** A property positioned as Luxury when the evidence supports Standard leads to wrong specification, wrong marketing spend, and wrong buyer/tenant targeting.

**Goal:** Build a data-driven model that classifies proposed Irish residential properties into one of three price tiers Budget/ Standard / Luxury using county and property level features drawn from openly available Irish data sources.

## Data Sources

All data sources are publicly available from Irish sources.

| # | Source | URL | Format | Content |
|---|--------|-----|--------|---------|
| 1 | RTB Average Monthly Rent Report | https://data.gov.ie/dataset/ria02-rtb-average-monthly-rent-report | CSV | Average monthly rent per property by county, bedroom type, and property type |
| 2 | Revenue Net Receipts by County | https://data.gov.ie/dataset/revenue-net-receipts-by-county | CSV | County-level VAT, PAYE/USC, self-employed tax, corporation tax, CGT |
| 3 | Population per County | https://data.gov.ie/dataset/g0420-population-per-county | CSV | 2022 census population per county |
| 4 | Mean Net Household Income by County | https://data.cso.ie/GPIIA02 | CSV | CSO mean net household income per county (2022) |
| 5 | Average Agricultural Land Price by County | https://teagasc.ie/news--events/daily/map-land-prices-by-county/ | Manual (TXT) | Teagasc county-level average land price per acre (€), manually transcribed from published map visual |

---
## [Dataset Creation](https://github.com/killianglavo-coder/Python-Machine-Learning----Rental-Tier-Classification-Irish-Property-Market/blob/main/1_Data_Prep.ipynb)

Some pre-processing was done in excel to remove redundant and missing variables 

The following pipeline was followed in python:
1. Load each CSV source
2. Standardise county names and filter to year 2022 in Python (no external Excel pre-processing)
3. Engineer two derived features
4. Merge all sources on County
5. Drop redundant columns
6. Save the final dataset
## Task Definitions

### Clustering Task
**Objective:** Two unsupervised clustering algorithms are applied, [K-Means (k=3)](https://github.com/killianglavo-coder/Python-Machine-Learning----Rental-Tier-Classification-Irish-Property-Market/blob/main/2_KMeans_Clustering.ipynb) and [Agglomerative Hierarchical Clustering (k=3, Ward linkage)](https://github.com/killianglavo-coder/Python-Machine-Learning----Rental-Tier-Classification-Irish-Property-Market/blob/main/3_agglomerative_clustering_model2.ipynb). These are applied to the data with no knowledge of the manually assigned tier labels. The goal is simply to see whether the data finds its own natural groupings, and whether those groupings end up looking anything like the property market segments we'd expect.

**Features used for clustering:** Rent_Price, Land_Price, Population of County, Average_Value_County, Net_Income, Average_Value_Bedroom_Count

**Business value:** If the data splits into three recognisable market segments on its own, that provides rational to the three-tier framework and suggests the labels are capturing something genuine. It can also surface properties that sit awkwardly within their county group, which may be important to define mispricing.

### Multiclass Classification Task
**Objective:** Two supervised classifiers, Logistic Regression and Random Forest, are trained and compared on the task of predicting the Price_Category label (Budget / Standard / Luxury) from a property's county-level and property-level features.
- [**Logistic Regression**](https://github.com/killianglavo-coder/Python-Machine-Learning----Rental-Tier-Classification-Irish-Property-Market/blob/main/4_logistic_regression.ipynb) acts as the baseline. Its coefficients are easy to read and make it simple to explain why a property was assigned to a particular tier.
- [**Random Forest**](https://github.com/killianglavo-coder/Python-Machine-Learning----Rental-Tier-Classification-Irish-Property-Market/blob/main/5_randomforest_and_regression_models_comparison.ipynb) is the main model being evaluated. It works across an ensemble of decision trees, which means it can pick up on patterns and feature interactions that a straight linear model would miss. It also produces feature importance scores which are important for answering *which factors best determine a property's tier?*
**Target variable:** Price_Category, a three-class label (Budget / Standard / Luxury) derived from Rent_Price percentile thresholds as set out in Section 5.

**Features used:** All numeric columns after redundant identifiers are removed. Both models are run on the same 80/20 split and use class_weight='balanced' to stop any imbalance between class sizes from skewing results.

**Business value:** With a working classifier in place, the development team can put in the relevant details for a proposed site and get back a tier prediction straight away rather than waiting on a manual valuation. The Random Forest importance scores are also useful for working out which pieces of information are most influential when scoping out future sites.

# Findings and Business Use Case: 

## [Agglomerative Hierarchical Clustering (k=3, Ward linkage)](https://github.com/killianglavo-coder/Python-Machine-Learning----Rental-Tier-Classification-Irish-Property-Market/blob/main/3_agglomerative_clustering_model2.ipynb)
**Optimal Separation:**
- For all models, the clusters are divided along the horizontal axis, indicating that PC1 is the primary driver of cluster separation. PC1 likely reflects high value factors like Rent_Price and Population.
- All three models in the 'Dublin cluster' are clearly separated along the horizontal axis. However, when using 3 clusters the separation isn't as distinct.
- While the clusters are moderately separated in our models with 3 clusters, using 2 clusters creates an increased separation between clusters.
- The main difference in our k-means model with 3 clusters and our agglomerative model with 3 clusters, is within our k-means model there is a small amount of crossover between our 'luxury' and 'standard' clusters indicating less separation.
- Out of all models the Agglomerative Model with 2 clusters returns the highest silhouette score (0.6) indicating it returns the most well-defined and separated cluster.
- For this reason we believe this is the optimal model for our business to use, even if it does not line up with our 3-tiered pricing system. 

**Business Impact:**
Overall, a two-cluster agglomerative model is likely the most suitable for this data revealing the need for our business to treat the Dublin rental market entirely different to the rest of the country. Overall, this spurs further investigation into the drivers of rental prices within these two clusters, and on collection of more data this model should be re-deployed to employ this strategy. Although any use in a business scenario the model should be used a tool alongside employees with expert domain knowledge to ensure active over-sight of the model's decision-making.

## [K-Means (k=3)](https://github.com/killianglavo-coder/Python-Machine-Learning----Rental-Tier-Classification-Irish-Property-Market/blob/main/2_KMeans_Clustering.ipynb)
Price Category Breakdown:

- **Luxury**: performs the best with recall of 94%, meaning the model correctly identified 450 out of 481 Luxury properties. This makes intuitive sense as Luxury properties in high-value counties (ie Dublin) have very distinct Rent_Price, Land_price and Net_Income values that separate them from other tiers.

- **Budget**: achieves 66% recall, identifying 317 of 482 Budget properties. The 165 misclassified as Standard likely represent properties in counties where budget tier rents overlap with standard tier pricing, which is a known characteristic of the Irish rental market especially outside major urban centres.

- **Standard**: shows the weakest recall at 57%, correctly identifying 552 of 962 properties. This is something we expected to see as Standard is the middle tier and naturally shares feature overlap with both Budget and Luxury at its boundaries. 199 Standard properties were assigned to Budget and 211 to Luxury, reflecting the continuous nature of property pricing in practice.

**Business Implication:**

The strong performance on Luxury (F1: 0.79) and reasonable performance on Budget (F1: 0.63) is particularly valuable for a property developer. Correctly identifying premium developments avoids under-pricing, which directly protects revenue. The weaker Standard performance highlights that mid-range pricing decisions may benefit from additional features such as proximity to Eircode zones or local amenity data to sharpen tier boundaries in future iterations of the model.

## [Logistic Regression](https://github.com/killianglavo-coder/Python-Machine-Learning----Rental-Tier-Classification-Irish-Property-Market/blob/main/4_logistic_regression.ipynb) and [Random Forest Model Comparison and Business use](https://github.com/killianglavo-coder/Python-Machine-Learning----Rental-Tier-Classification-Irish-Property-Market/blob/main/5_randomforest_and_regression_models_comparison.ipynb)
Both models achieve very similar accuracy scores, Logistic Regression at 78.01% and Random Forrest at 78.63%. 

- Regarding the confusion matrices, the key difference is in how each model handles the budget tier. Logistic Regression correctly identifies 107 of 121 Budget properties, random forest only gets 88, misclassifying 33 as standard. 

- However, random forest does perform better on standard, correctly identifying 184 of 241 compared to logistic regression's 162. both models perform the same on luxury with 107 correct.

- In conclusion, Random Forrest achieves a marginally better accuracy score and performs better on standard, while logistic regression performs better on budget classification. For our business use case, random forest is the stronger performing model here as it has a slightly higher accuracy score and correctly classifies more standard properties, which would be the largest and most commercially common tier. 

- Logistic regression is still useful. This is because it achieves nearly an identical accuracy score to random forest and is significantly faster to train and easier to update as new data becomes available. 
