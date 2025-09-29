# supplier_portfolio_analysis
This case study show analysis of % purchasing portfolio allocation amongst 5 suppliers, include clustering analysis sing K-means, various reallocation of supplier portfolio, estimation of potential savings and KPI improvement 

# Supplier Portfolio Analysis – Case Study  

---

## Structure  
From a local `.xlsx/.csv` file (imported via *choose file location*), build a robust Python workflow on Google Colab.  

---

## Objective  
1. Compute a **performance score (0–10)** for the 5 suppliers.  
2. Apply a **3-cluster ML segmentation (KMeans)**: Top Performers, Mid-Range Performers, Needs Improvement.  
3. Design a **new supplier allocation** based on clustering results and performance scores.  
4. Estimate the **savings generated** under different allocation scenarios.  

---

## Dataset  

**Shape**: 5 rows × 12 columns  

 0  supplier                 5 non-null object
 1  country                  5 non-null object
 2  yearly_turnover_$        5 non-null int64
 3  yearly_turnover_unit     5 non-null int64
 4  po_volume_yearly         5 non-null int64
 5  avg_quantity_per_po      5 non-null int64
 6  freigth_cost_per_po      5 non-null int64
 7  unit_cost_1_to_100       5 non-null float64
 8  unit_cost_100_to_1000    5 non-null float64
 9  unit_cost_over_1000      5 non-null float64
 10 conformity_rate_per_po   5 non-null float64
 11 otd_per_po               5 non-null float64

📋 --- Samples rows ---
  supplier  country  yearly_turnover_$  yearly_turnover_unit  po_volume_yearly ...
0   APEX       USA            700000              87282              ...
1   BLITZ    GERMANY           22500               2571              ...
...

# STEP 1 - Performance score
total_score_performance = (price_performance_norm*0.5 + quality_performance_norm*0.2 + delivery_performance_norm*0.3)

# 1a - Total cost performance
unit_cost_applied =
    if avg_quantity_per_po ≤ 100 → utiliser unit_cost_1_to_100
    if 100 < avg_quantity_per_po ≤ 1000 → utiliser unit_cost_100_to_1000
    if avg_quantity_per_po > 1000 → utiliser unit_cost_over_1000

unit_cost_total = yearly_turnover_unit × unit_cost_applied
unit_cost_total_norm = normalisation 0–10

freight_total_cost = freigth_cost_per_po × po_volume_yearly
freight_total_cost_norm = normalisation 0–10

non_quality_cost = (1 − conformity_rate_per_po) × unit_cost_applied × yearly_turnover_unit
non_quality_cost_norm = normalisation 0–10

price_performance = (unit_cost_total_norm*0.8 + freight_total_cost_norm*0.05 + non_quality_cost_norm*0.15)
price_performance_norm = normalisation 0–10

# 1b - Progressive discount rate
discount_1 = (unit_cost_1_to_100 - unit_cost_100_to_1000) / unit_cost_1_to_100 * 100
discount_2 = (unit_cost_100_to_1000 - unit_cost_over_1000) / unit_cost_100_to_1000 * 100
progressive_discount_rate = discount_1 + discount_2
progressive_discount_rate_norm = normalisation 0–10

total_cost_performance = (progressive_discount_rate_norm*0.2 + price_performance_norm*0.8)

# 1c - Quality performance
average_quality = moyenne(conformity_rate_per_po)
stdev_quality = écart-type(conformity_rate_per_po)
quality_performance = (conformity_rate_per_po - average_quality) / stdev_quality
quality_performance_norm = normalisation 0–10

# 1d - Delivery performance
average_otd = mean(otd_per_po)
stdev_otd = std-deviation(otd_per_po)
delivery_performance = (otd_per_po - average_otd) / stdev_otd
delivery_performance_norm = normalization 0–10

# 1e - Global performance
total_score_performance = (total_cost_performance*0.6 + quality_performance_norm*0.2 + delivery_performance_norm*0.2)

# STEP 2 - Clustering ML KMeans
kmeans sur total_score_performance avec 3 clusters
Attribuer : Needs Improvement, Mid-Range Performers, Top Performers

# STEP 3 - New Supplier allocation
total_purchase_volume = somme(yearly_turnover_$)
current_allocation = yearly_turnover_$ / total_purchase_volume

factorrs :
factor_conservative = {'Top Performers':1.15,'Mid-Range Performers':1.0,'Needs Improvement':0.85}
factor_aggressive   = {'Top Performers':1.5,'Mid-Range Performers':1.0,'Needs Improvement':0.8}
factor_risk_mit     = {'Top Performers':1.2,'Mid-Range Performers':1.0,'Needs Improvement':0.5}

Calculate new_allocation_% for each scenario with factors

# STEP 4 - Savings generated
supplier_total_cost = unit_cost_total + freight_total_cost + non_quality_cost
portfolio_total_cost_current = Σ(supplier_total_cost × current_allocation)
portfolio_total_cost_scenario = Σ(supplier_total_cost × new_allocation_%)

savings = absolute difference and %
savings_non_quality = difference of non_quality_cost weighted
delta_late_orders = difference on (1−otd_per_po) × po_volume_yearly

# STEP 5 - Multi sheets excel export
Columns exported :
supplier, current_allocation, new_allocation_conservative, new_allocation_aggressive, new_allocation_mit,
supplier_total_cost, non_quality_cost, late_orders,
Global savings and details
