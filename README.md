# County Economic Development Scorecard Ranking Kenyas 47 Counties

## Project Overview
This project ranks all 47 counties in Kenya based on key development indicators: healthcare access, education, economy, and service delivery. It uses publicly available data to create a composite scorecard, providing insights into regional disparities and opportunities for improvement. The scorecard is built using data pipelines for processing, scoring models for ranking, and interactive dashboards for visualization.
The goal is to assist policymakers, researchers, and stakeholders in understanding county level performance and prioritizing interventions.

## Key Indicators
### Healthcare
- Health facilities per capita
- Immunization coverage
- Maternal & child health statistics
- Health workforce density

### Education
- Literacy rates
- School enrollment (primary & secondary)
- Pupil–teacher ratio
- KCSE performance averages

### Economy
- County GDP / economic output
- Employment rates
- Household income levels
- Poverty index

### Service Delivery
- Access to clean water
- Infrastructure quality (roads, electricity)
- Public service efficiency metrics
- Budget absorption rates

## Data Sources
- Kenya Open Data Portal
- World Bank Kenya County Indicators
- KNBS reports
- County Integrated Development Plans (CIDPs)
- Ministry of Health, Education, and Economy datasets
- Healthcare Access: Download Kenya Master Health Facility List (KMHFL) from https://kmhfl.health.go.ke/ or KNBS County Statistical Abstracts (CSAs) from https://www.knbs.or.ke/county-statistical-abstracts/. Metrics: facilities per 100k population, hospitals/wards per county.​

Education: Fetch KNBS school census data (e.g., https://www.knbs.or.ke/reports_category/county-statistical-abstracts/) or statskenya.co.ke datasets. Metrics: primary/secondary schools per capita, enrollment rates, pupil-teacher ratios by county.​

Economy: Use KNBS Gross County Product (GCP) tables from https://www.knbs.or.ke/gross-county-product/ or Wikipedia KNBS-derived rankings (2022/2023 data). Metrics: GCP (KSh billions), GDP per capita, growth rates.​

Service Delivery: Extract from KNBS CSAs (all 47 counties available as PDFs with tables on water access, roads km/capita, electricity coverage). Metrics: % households with piped water, road density, sanitation coverage.​

## Skills Demonstrated
1. Data Engineering / Pipelines: ETL, cleaning, merging, automation
2. Scoring Model: Weighted composite indices, standardization, ranking
3. Data Visualization: Interactive dashboards, heatmaps, trend analysis
Based on the 2024/2025 Infotrak Research / CountyTrak Performance Index, the following counties stand out:

Additionally, when looking at broader development measures such as human development index (HDI), Nairobi County leads overall, followed by Mombasa County, Nyeri, Kiambu County, etc.

---
---

## 5️⃣ Starter Python Script Examples

### `scripts/data_pipeline.py`
```python
import pandas as pd

# Load raw datasets
health_df = pd.read_csv('../data/raw/health_data.csv')
education_df = pd.read_csv('../data/raw/education_data.csv')
economy_df = pd.read_csv('../data/raw/economy_data.csv')
service_df = pd.read_csv('../data/raw/service_data.csv')

# Merge datasets on County column
merged_df = health_df.merge(education_df, on='County') \
                     .merge(economy_df, on='County') \
                     .merge(service_df, on='County')

# Save processed dataset
merged_df.to_csv('../data/processed/county_data.csv', index=False)
print("Data pipeline completed. Processed file saved to data/processed/")

### `scripts/scoring_model.py

import pandas as pd
from sklearn.preprocessing import MinMaxScaler

# Load processed data
df = pd.read_csv('../data/processed/county_data.csv')

# Define pillars and weights
pillars = {
    'Healthcare': ['Health_facilities_per_capita','Immunization_coverage','Maternal_child_health','Health_workforce_density'],
    'Education': ['Literacy_rate','Primary_enrollment','Secondary_enrollment','Pupil_teacher_ratio','KCSE_avg'],
    'Economy': ['GDP','Employment_rate','Household_income','Poverty_index'],
    'Service': ['Clean_water_access','Infrastructure_quality','Public_service_efficiency','Budget_absorption']
}
weights = {'Healthcare': 0.25, 'Education': 0.25, 'Economy':0.25, 'Service':0.25}

# Normalize columns
scaler = MinMaxScaler()
for pillar, cols in pillars.items():
    df[cols] = scaler.fit_transform(df[cols])

# Compute pillar scores
for pillar, cols in pillars.items():
    df[pillar+'_score'] = df[cols].mean(axis=1)

# Compute overall score
df['Overall_score'] = sum(df[p+'_score']*w for p,w in weights.items())

# Rank counties
df['Rank'] = df['Overall_score'].rank(ascending=False)

# Save scored dataset
df.to_csv('../data/processed/county_scored.csv', index=False)
print("Scoring model completed. Results saved to data/processed/county_scored.csv")

dashboard/app.py

import streamlit as st
import pandas as pd
import plotly.express as px

df = pd.read_csv('../data/processed/county_scored.csv')

st.title("Kenya County Economic Development Scorecard")

# Table
st.subheader("County Rankings")
st.dataframe(df[['County','Overall_score','Rank']].sort_values('Rank'))

# Heatmap
st.subheader("County Score Heatmap")
fig = px.choropleth(df, locations='County', locationmode='geojson-id',
                    color='Overall_score', color_continuous_scale="Viridis")


### Acknowledgments
Data provided by Kenya National Bureau of Statistics (KNBS), World Bank, and Kenya Open Data.
Inspired by development indices like the Human Development Index (HDI).

For questions, contact marcuskiama97@gmail.com
