# Visualisation

* Quicksight (Md)
* Other: D3.js, Highcharts (Sm)

## Quicksight

Business analytics and viz tool in cloud

* Perform ad-hoc analysis
* Build visualisations
* Browser or mobile
* Build dashboard
* Serverless

Create ad-hoc expoloration of data

* Stories
  * Guided tours through specific views of an analysis

* Analyses visaulise data from
  * Logs in S3
  * On prem databases
  * Athena, Redshift,...
  * SaaS application, such as Salesforce
  * ANy JDBC/ODBC data source

To create visualisations in Quicksight:

* New Analysis
* Connect to dataset (S3, Upload file, Redshift, Teradata, GitHub, IoT, Twitter...)
  * Quicksight needs access to resources
* Select data set
* Visualize (auto by default)
* `Capture` to send to Story -> Storyboard

### Quicksight Datasources

* Redshift
* Aurura / RDS
* Athena
* EC2 hosted databases
* Files (S3 or on-premises)
  * Excel
  * CSV
  * Log formats
* Data preparation allows limited ETL (change field names or add calc or change types)

### SPICE import datasets into Quicksight, in memory calculation

Engine that enables in-memory (fast) powered viz and queries.
Can import data straight into memory.

* Each user gets 10GB of SPICE
* Scales to +100k users

### Security

* MFA for account
* VPC connectivity
  * Add QuickSight IP to database security group
* Row-level security
* Private VPC access
  * Elastic Network Interface
  * AWS Direct Connect for on-prem databases

User management (standard edition or enterprise edition)

* Email signup (standard)
* IAM (standard)
* Active Directory (enterprise)

### Pricing for QuickSight is by subscription + SPICE capacity over 10GB

* Enterprise edition gives you Encryption at rest for results

### Dashboard in QuickSight, read only snapshots

Share dashboards with other users that have access to QuickSight.

### QuickSight Machine Learning Insights

* ML Powered Anamoly detection
  * Uses Random Cut Forest
  * Identify top contributors to changes
  * Finds outliers in dataset

* ML Powered Forecasting
  * Also uses Random Cut Forest
  * Detects seasonality and trends
  * Exludes outliers and imputes missing values

* Autonarratives
  * Adds 'story of your data' to your dashboards

* Suggested insights
  * 'Insights' tab displays read-to-use suggested insights
  * Tells you if data is good or bad

### Choosing Visualisation Types in Quicksight

* **AutoGraph**: generates viz on auto
* **Bar Charts**: for comparison and distributions (compare quantities), histograms
* **Line Graph**: For changes over time, use a line graph
* **Scatter plots, Heat Map**: Good for corrolation. If you need to see corrolations, use a scatter plot or a heat map
* **Pie Graphs, Donut Charts, Tree Maps**: If you need to see aggrigation, how individual parts add up to a whole. Use a Pie Graph or a Tree Map. Tree maps are good for Heirarchical Aggregation.
* **Pivot Tables**: Pivot tables are great for tabular data.
* **Stories**: Making a slide show out of you data with stories.

* **KPIs**: Compare key value to a target value (display current revenue to target revenue)
* **Geospatial Charts**: Maps, plot distribution on map.
* **Gauge Charts**: Compare values in a measure (fuel in tank or bandwith usage or percentages of 100%)
* **Word Clouds**: Displays word or phrase frequency

## Other Visualisation Tools

* Web-based javascript libraries
  * D3.js
  * Chart.js
  * Highchart.js
* Business Intelligence Tools (may be used on top of Redshift)
  * Tableau
  * MicroStrategy
