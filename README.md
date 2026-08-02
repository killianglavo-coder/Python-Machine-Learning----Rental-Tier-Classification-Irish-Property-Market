# Python-Machine-Learning----Rental-Tier-Classification-Irish-Property-Market
# Project Overview: 
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
## Task Definitions

### Clustering Task
**Objective:** Two unsupervised clustering algorithms are applied, K-Means (k=3) and Agglomerative Hierarchical Clustering (k=3, Ward linkage). These are applied to the data with no knowledge of the manually assigned tier labels. The goal is simply to see whether the data finds its own natural groupings, and whether those groupings end up looking anything like the property market segments we'd expect.

**Features used for clustering:** Rent_Price, Land_Price, Population of County, Average_Value_County, Net_Income, Average_Value_Bedroom_Count

**Business value:** If the data splits into three recognisable market segments on its own, that provides rational to the three-tier framework and suggests the labels are capturing something genuine. It can also surface properties that sit awkwardly within their county group, which may be important to define mispricing.

### Multiclass Classification Task
**Objective:** Two supervised classifiers, Logistic Regression and Random Forest, are trained and compared on the task of predicting the Price_Category label (Budget / Standard / Luxury) from a property's county-level and property-level features.
- **Logistic Regression** acts as the baseline. Its coefficients are easy to read and make it simple to explain why a property was assigned to a particular tier.
- **Random Forest** is the main model being evaluated. It works across an ensemble of decision trees, which means it can pick up on patterns and feature interactions that a straight linear model would miss. It also produces feature importance scores which are important for answering *which factors best determine a property's tier?*
- 
**Target variable:** Price_Category, a three-class label (Budget / Standard / Luxury) derived from Rent_Price percentile thresholds as set out in Section 5.

**Features used:** All numeric columns after redundant identifiers are removed. Both models are run on the same 80/20 split and use class_weight='balanced' to stop any imbalance between class sizes from skewing results.

**Business value:** With a working classifier in place, the development team can put in the relevant details for a proposed site and get back a tier prediction straight away rather than waiting on a manual valuation. The Random Forest importance scores are also useful for working out which pieces of information are most influential when scoping out future sites.

# Findings: 

## Logistic Regression
* **Luxury**: This is the best performing class with the highest f1-score and recall score of 0.82 and 0.89 respectively. The model correctly identified 107 of 120 luxury properties with all misclassifications (13) landing in standard. Correctly identifying lxury properties is critical as it prevents under-pricing of premium properties in high-values counties.

* **Budget**: Budget also performs strongly with a recall score of 0.88 and an f1-score of 0.78. It correctly identifies 107 of 121 budget properties with all 14 misclassifications landing in standard, similar to luxury.

* **Standard**: Standard is the weakest performer across the three with the weakest recall of 0.67 and f1-score of 0.75. It correctly identifies 162 of 241 Standard properties. 46 misclassifications went to budget and 33 to luxury. The high precision of 0.86 shows that when the model does predict Standard, it's usually correct.

* There are zero budget-luxury misclassifications or luxury-budget, all errors occur at the standard boundary where cost to a developer would be much lower than a luxury budget misclassification
## Model Accuracy Score
* The model achieves a test accuracy of 78%
* This is a strong result considering that `Rent_Price`, which would be the most direct indicator of a property's tier, was deliberately excluded to prevent data leakage. 
* The model is predicting tier level entirely from county-level economic features, land price, property values, and bedroom type, which are all available before any rental valuation is done.

## Agglomerative Clustering Model
**Optimal Separation:**
* For all models, the clusters are divided along the horizontal axis, indicating that PC1 is the primary driver of cluster separation. PC1 once again likely reflects high value factors like Rent_Price and Population.
* In all three models the 'Dublin cluster' is clearly separated along the horizontal axis. However, when using 3 clusters the separation isn't as distinct. While the clusters are moderately separated in our models with 3 clusters, using 2 clusters creates an increased separation between clusters.
* The main difference in our k-means model with 3 clusters and our agglomerative model with 3 clusters, is within our k-means model there is a small amount of crossover between our 'luxury' and 'standard' clusters indicating less separation. Out of all models the Agglomerative Model with 2 clusters returns the highest silhouette score (0.6) indicating it returns the most well-defined and separated cluster. For this reason we believe this is the optimal model for our business to use, even if it does not line up with our 3-tiered pricing system. 

* **Business Impact:**
Overall, a two-cluster agglomerative model is likely the most suitable for this data revealing the need for our business to treat the Dublin rental market entirely different to the rest of the country. Overall, this spurs further investigation into the drivers of rental prices within these two clusters, and on collection of more data this model should be re-deployed to employ this strategy. Although any use in a business scenario the model should be used a tool alongside employees with expert domain knowledge to ensure active over-sight of the model's decision-making.


## Logistic Regression and Random Forest Model Comparison and Business use

* Both models achieve very similar accuracy scores, Logistic Regression at 78.01% and Random Forrest at 78.63%. 

* Regarding the confusion matrices, the key difference is in how each model handles the budget tier. Logistic Regression correctly identifies 107 of 121 Budget properties, random forest only gets 88, misclassifying 33 as standard. 

* However, random forest does perform better on standard, correctly identifying 184 of 241 compared to logistic regression's 162. both models perform the same on luxury with 107 correct.

* In conclusion, Random Forrest achieves a marginally better accuracy score and performs better on standard, while logistic regression performs better on budget classification. For our business use case, random forest is the stronger performing model here as it has a slightly higher accuracy score and correctly classifies more standard properties, which would be the largest and most commercially common tier. 

* Logistic regression is still useful. This is because it achieves nearly an identical accuracy score to random forest and is significantly faster to train and easier to update as new data becomes available. 
