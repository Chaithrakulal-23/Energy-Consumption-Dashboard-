# Energy-Consumption-Dashboard

### Dashboard Link : https://app.powerbi.com/links/XD2BeAzs28?ctid=51697115-1ecd-42b5-b509-2d62c3919f76&pbi_source=linkShare
## Problem Statement

This dashboard helps energy analysts and policymakers understand renewable energy consumption patterns across regions, countries, and energy sources. It enables data-driven decisions by analyzing monthly energy usage (in kWh) and cost savings (in USD) broken down by region, energy source type, and country.

Since energy consumption and cost savings vary significantly across regions and income levels, this dashboard helps identify high-consumption zones and low-cost-efficiency areas, allowing stakeholders to focus on optimizing energy distribution and improving renewable energy adoption strategies.

Wind energy leads in both monthly usage (0.28M) and cost savings (59K USD), while regions like North America show relatively lower cost savings — indicating areas where policy or infrastructure improvements can yield better returns.

---

### Steps Followed

- **Step 1 :** Loaded the **Renewable Energy Usage** dataset (Excel file) into AWS S3 by creating a dedicated S3 bucket via the AWS Management Console and uploading the sampled dataset.

  ![S3 Bucket with Dataset](https://user-images.githubusercontent.com/your-id/s3-bucket-upload.jpg)

- **Step 2 :** Established a secure connection between AWS S3 and Snowflake by creating an IAM Role in AWS:
  - Navigated to IAM → Create Role → Selected AWS Account type.
  - Provided a placeholder External ID and attached the **AmazonS3FullAccess** permission policy.
  - Named the role and noted the generated **ARN** and **Trust Policy**.

  ![IAM Role ](https://user-images.githubusercontent.com/your-id/iam-role-arn.jpg)

- **Step 3 :** In Snowflake, created a **Storage Integration Object** using the IAM Role ARN to authorize Snowflake to read from the S3 bucket.

  ![Snowflake Storage Integration Code](https://user-images.githubusercontent.com/your-id/snowflake-integration-code.jpg)

- **Step 4 :** Retrieved the Snowflake-generated **ARN** and **External ID** using:
  ```sql
  DESC INTEGRATION <integration_name>;
  ```
  Pasted these values into the IAM Role's Trust Policy in AWS and updated it to complete the secure handshake.

  ![Updated Trust Policy](https://user-images.githubusercontent.com/your-id/desc-integration-trust-policy.jpg)

- **Step 5 :** In Snowflake, set up the data pipeline:
  - Created a **Database**, **Schema**, and **Table** named `Power_Dataset` to store the renewable energy data.
  - Created a **Stage** pointing to the S3 bucket location.
  - Loaded data from the Stage into the Snowflake table using COPY INTO.

  ![Snowflake Database Schema Stage Table](https://user-images.githubusercontent.com/your-id/snowflake-db-schema-table.jpg)

- **Step 6 :** Performed **Data Profiling** to understand the dataset structure — reviewed column types, value distributions, and identified key fields like `monthly_usage_kwh`, `income_level`, `energy_source`, `region`, and `country`.

- **Step 7 :** Copied data from the original table into a working table to perform transformations without affecting raw data. Transformations applied to the `monthly_usage_kwh` column based on income level:

  ```sql
  -- Low income level: 10% increase
  UPDATE energy_consumption
  SET monthly_usage_kwh = monthly_usage_kwh * 1.1
  WHERE income_level = 'Low';

  -- Middle income level: 20% increase
  UPDATE energy_consumption
  SET monthly_usage_kwh = monthly_usage_kwh * 1.2
  WHERE income_level = 'Middle';

  -- High income level: 30% increase
  UPDATE energy_consumption
  SET monthly_usage_kwh = monthly_usage_kwh * 1.3
  WHERE income_level = 'High';

  SELECT * FROM energy_consumption;
  ```

  ![Snowflake Transformation SQL - Monthly Usage KWH Update](https://user-images.githubusercontent.com/your-id/snowflake-transformation-sql.jpg)

- **Step 8 :** Connected **Power BI Desktop** to Snowflake:
  - Used **Get Data → Snowflake** in Power BI.
  - Pasted the Snowflake account URL and authenticated.
  - Selected the `energy_consumption` table and loaded it into Power BI Desktop.

- **Step 9 :** Built the **Energy Consumption Dashboard** with the following visuals using **Bar Charts**:

  **Monthly Usage Analysis (kWh)**
  - Monthly Usage by Region
  - Monthly Usage by Energy Source
  - Monthly Usage by Country

  **Cost Savings Analysis (USD)**
  - Cost Savings USD by Region
  - Cost Savings USD by Energy Source
  - Cost Savings USD by Country



- **Step 10 :** Published the completed report to **Power BI Cloud Service** for stakeholder access.

  ![Power BI Publish Success Message](https://user-images.githubusercontent.com/your-id/powerbi-publish-success.jpg)

---

# Snapshot of Dashboard (Power BI Service)

![Dashboard Snapshot Power BI Service](https://user-images.githubusercontent.com/your-id/dashboard-powerbi-service.jpg)

# Report Snapshot (Power BI Desktop)

![Report Snapshot Power BI Desktop](https://user-images.githubusercontent.com/your-id/dashboard-powerbi-desktop-full.jpg)

---

# Insights

A single page report was created on Power BI Desktop & it was then published to Power BI Service.

The following inferences can be drawn from the dashboard:

### [1] Monthly Usage by Region (kWh)

   - Europe & South America lead with **210K kWh** each — the highest among all regions.
   - Africa follows at **206K**, with Asia, North America, and Australia at **~191–195K**.

           thus, energy consumption is fairly distributed across regions, with Europe and South America consuming the most.

### [2] Monthly Usage by Energy Source

    a) Wind      - 0.28M kWh  ← Highest
    b) Hydro     - 0.24M kWh
    c) Biomass   - 0.24M kWh
    d) Solar     - 0.24M kWh
    e) Geothermal- 0.22M kWh  ← Lowest

           thus, Wind energy is the dominant renewable energy source by monthly consumption.

### [3] Monthly Usage by Country

   Top consuming countries:
   - **Australia** — 105K kWh
   - **New Zealand** — 86K kWh
   - **USA** — 69K kWh
   - **Canada** — 65K kWh
   - **Mexico** — 57K kWh

           thus, Australia leads all countries in monthly energy consumption.

### [4] Cost Savings USD by Energy Source

    a) Wind       - 59K USD  ← Highest savings
    b) Solar      - 51K USD
    c) Biomass    - 50K USD
    d) Hydro      - 47K USD
    e) Geothermal - 41K USD  ← Lowest savings

           thus, Wind energy delivers the highest cost savings, consistent with its usage dominance.

### [5] Cost Savings USD by Region

   - South America & Africa — **43K USD** each (highest)
   - Europe — **42K USD**
   - Asia & Australia — **41K USD**
   - North America — **38K USD** (lowest)

           thus, North America shows the lowest cost savings despite moderate energy usage — indicating scope for efficiency improvement.

### [6] Cost Savings USD by Country

   - **New Zealand** — 21K USD (highest savings)
   - **Australia** — 19K USD
   - **USA** — 14K USD
   - **Canada** — 13K USD

           thus, New Zealand achieves the best cost efficiency among all countries.

### [7] Data Transformations Applied

   - `monthly_usage_kwh` scaled by **+10%** for Low income level customers.
   - `monthly_usage_kwh` scaled by **+20%** for Middle income level customers.
   - `monthly_usage_kwh` scaled by **+30%** for High income level customers.

   These adjustments normalize usage data to reflect realistic consumption patterns across income segments.

---

### Tech Stack

| Layer | Tool |
|---|---|
| Cloud Storage | AWS S3 |
| Data Warehouse | Snowflake |
| Visualization | Microsoft Power BI |
| Dataset | Renewable Energy Usage (Sampled) |
| Integration | AWS IAM Role + Snowflake Storage Integration |
