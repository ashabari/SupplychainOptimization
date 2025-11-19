# Optimization and Failure Analysis in Healthcare Supply Chains

This project uses the **Hospital Supply Chain** dataset from Kaggle to study
how hospital inventory behaves under stress and to build a small decision
support tool for administrators. The focus is on:

- Demand forecasting for medical supplies  
- Failure risk in the supply chain  
- Optimal allocation of limited inventory under surge scenarios  

The work is implemented in a single Jupyter notebook that is easy to run in
Google Colab.

---

## 1. Problem overview

Hospitals rely on a steady flow of critical supplies such as ventilators,
masks and IV sets. During shocks such as pandemics, demand can surge while
lead times increase, which leads to stockouts and delayed care.

This project asks:

- Which items and vendors contribute most to supply chain risk  
- How to forecast demand so that shortages can be anticipated  
- How to allocate limited stock to minimise unmet demand and cost  
- How the system behaves under different stress scenarios  

---

## 2. Data

Source: **Hospital Supply Chain** dataset on Kaggle.  

Tables used:

- **Inventory**: daily stock levels, minimum required, maximum capacity,
  unit cost, average usage per day, restock lead time and vendor.  
- **Vendor**: per vendor lead time and price information.  
- **Optional**: patient and staff tables are available but not required
  for the core optimisation analysis.

The notebook creates additional features such as:

- Stock buffer = Current_Stock minus Min_Required  
- Binary labels for delay failures and shortage risk  

---

## 3. Methods

The notebook is organised into the following sections.

### 3.1 Exploratory data analysis

- Basic checks on column types and missing values  
- Visualisations of:
  - Stock vs minimum required by item type  
  - Distribution of restock lead times  
  - Vendor level lead time patterns  

### 3.2 Failure analysis

- Define **delay failure** as restock lead time above the 90th percentile.  
- Define **shortage risk** as current stock close to or below minimum required.  
- Train a **RandomForestClassifier** to predict delay failures from:
  - Current_Stock, Min_Required, Max_Capacity  
  - Unit_Cost, Avg_Usage_Per_Day  
  - Item_Type, Vendor_ID  
- Report ROC AUC and classification metrics, and inspect feature importance.

### 3.3 Demand forecasting

- Build a daily time series of average usage per item.  
- Engineer time series features:
  - Day of week, month  
  - Lagged usage (1, 2 and 3 days)  
  - Rolling mean usage over 3 days  
- Train a **RandomForestRegressor** to forecast demand and report MAE and RMSE.  

This model serves as a simple demand forecasting engine that can be extended
with more advanced methods.

### 3.4 Optimisation model

- Focus on one planning date and aggregate items by name and type.  
- Use `Avg_Usage_Per_Day` as a proxy for demand on the planning horizon.  
- Construct a linear programming problem with PuLP:

  - Decision variables: allocation of stock to each item and unmet demand  
  - Objective: minimise total unmet demand plus a small cost term  
  - Constraints:
    - Total allocation cannot exceed available stock
    - Allocation plus unmet demand must equal demand for each item  

- Solve the problem and compute:
  - Allocation per item  
  - Unmet demand per item  
  - Shortage ratio (unmet demand divided by demand)  

### 3.5 Stress tests

- Multiply baseline demand by different **stress factors** (for example 1.0, 1.5, 2.0).  
- Keep total stock fixed and resolve the optimisation problem.  
- Record total unmet demand and total cost for each scenario.  
- Visualise how unmet demand grows as stress increases.

---

## 4. Key results and insights



- Demand forecasting model achieves approximately **X MAE** and **Y RMSE**,
  which improves over a naive average usage baseline.  
- Failure model achieves **ROC AUC ~ Z**, and highlights that:
  - Lead time, stock buffer and vendor identity are strong drivers of delay risk.  
- Under a severe stress scenario, optimisation reduces simulated unmet demand
  by about **A percent** compared to naive proportional allocation.  
- A small number of item categories account for most unmet demand, which
  suggests targeted safety stock policies.  
- Some vendors show both long average lead times and high failure rates, and
  are clear candidates for review or diversification.

---

## 5. How to run

### Option 1: Google Colab

1. Open the notebook file  
   `Optimization_&_Failure_Analysis_in_Supply_Chains.ipynb` in Google Colab.  
2. Make sure you have a Kaggle account and API token (`kaggle.json`).  
3. In Colab, upload `kaggle.json` when prompted in the first section.  
4. Run the notebook from top to bottom.  
5. All plots will display inline and you can adjust the stress factors or
   stock factor to explore different scenarios.

### Option 2: Local environment

1. Clone the repository:
   ```bash
   git clone https://github.com/ashabari/healthcare-supplychain-optimization.git
   cd healthcare-supplychain-optimization
