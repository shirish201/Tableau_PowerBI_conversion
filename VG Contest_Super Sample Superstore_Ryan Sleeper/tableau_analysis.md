# Tableau Workbook Analysis: VG Contest - Super Sample Superstore (Ryan Sleeper)

**Source file:** `VG Contest_Super Sample Superstore_Ryan Sleeper.twb`
**Tableau version:** 2023.1.0 (build 20231.23.0417.1258)
**Published to:** Tableau Public (`https://public.tableau.com/workbooks/VGContest_SuperSampleSuperstore_RyanSleeper`)

---

## 1. Data Source Connections

### Datasource: "Sample - Superstore"

| Property | Value |
|----------|-------|
| Connection class | `federated` |
| Named connection | `Sample - Superstoreleaf` |
| Inner connection class | `excel-direct` |

#### Tables

| Table Name | Physical Table | Type |
|------------|---------------|------|
| Orders$ | `[Orders$]` | table |
| Returns | `[Returns$]` | table |
| Extract | `[Extract].[Extract]` | table (extract cache) |

#### Join Definition

- **Join type:** LEFT JOIN
- **Condition:** `[Orders$].[Order ID] = [Returns].[Order ID]`

#### Orders$ Columns (21 columns)

| Column | Data Type | Ordinal |
|--------|-----------|---------|
| Row ID | integer | 0 |
| Order ID | string | 1 |
| Order Date | date | 2 |
| Ship Date | date | 3 |
| Ship Mode | string | 4 |
| Customer ID | string | 5 |
| Customer Name | string | 6 |
| Segment | string | 7 |
| Country | string | 8 |
| City | string | 9 |
| State | string | 10 |
| Postal Code | integer | 11 |
| Region | string | 12 |
| Product ID | string | 13 |
| Category | string | 14 |
| Sub-Category | string | 15 |
| Product Name | string | 16 |
| Sales | real | 17 |
| Quantity | integer | 18 |
| Discount | real | 19 |
| Profit | real | 20 |

#### Returns$ Columns (2 columns)

| Column | Data Type | Ordinal |
|--------|-----------|---------|
| Returned | string | 0 |
| Order ID | string | 1 |

#### Column Mappings (federated to source)

| Workbook Field | Source Column |
|----------------|-------------|
| [Category] | [Orders$].[Category] |
| [City] | [Orders$].[City] |
| [Country] | [Orders$].[Country] |
| [Customer ID] | [Orders$].[Customer ID] |
| [Customer Name] | [Orders$].[Customer Name] |
| [Discount] | [Orders$].[Discount] |
| [Order Date] | [Orders$].[Order Date] |
| [Order ID] | [Orders$].[Order ID] |
| [Order ID (Returns)] | [Returns].[Order ID] |
| [Postal Code] | [Orders$].[Postal Code] |
| [Product ID] | [Orders$].[Product ID] |
| [Product Name] | [Orders$].[Product Name] |
| [Profit] | [Orders$].[Profit] |
| [Quantity] | [Orders$].[Quantity] |
| [Region] | [Orders$].[Region] |
| [Returned] | [Returns].[Returned] |
| [Row ID] | [Orders$].[Row ID] |
| [Sales] | [Orders$].[Sales] |
| [Segment] | [Orders$].[Segment] |
| [Ship Date] | [Orders$].[Ship Date] |
| [Ship Mode] | [Orders$].[Ship Mode] |
| [State] | [Orders$].[State] |
| [Sub-Category] | [Orders$].[Sub-Category] |

### Datasource: "federated.1yv3g051gjjnme1dofc3r1i85dld" (Annotations helper)

| Property | Value |
|----------|-------|
| Connection class | `federated` |
| Named connection | `excel-direct.0xx813q14jqa1d1gipqda0c1jzei` |
| Inner connection class | `excel-direct` |

#### Tables

| Table Name | Physical Table | Type |
|------------|---------------|------|
| Sheet1 | `[Sheet1$]` | table |
| Extract | `[Extract].[Extract]` | table (extract cache) |

---

## 2. Calculated Fields (Sample - Superstore Datasource)

### Date & Time Period Fields

#### Order Date 2017
```
[Order Date] + 365
```
> Shifts all order dates forward by 365 days (normalizes dates to a 2017 baseline).

#### Days in Range
```
DATEDIFF('day', [Parameters].[Parameter 4], [Parameters].[Start Date (copy)]) + 1
```
> Calculates the number of days between the Minimum Date and Maximum Date parameters.

#### Date Filter CP (Current Period)
```
[Calculation_200128753919090690] >= [Parameters].[Parameter 4]
AND [Calculation_200128753919090690] <= [Parameters].[Start Date (copy)]
```
> Boolean: TRUE when the adjusted order date falls within the current period (between Minimum Date and Maximum Date).

#### Date Filter PP (Prior Period)
```
[Calculation_200128753919090690] >= [Parameters].[Parameter 4] - [Calculation_200128753924255750]
AND [Calculation_200128753919090690] <= [Parameters].[Start Date (copy)] - [Calculation_200128753924255750]
```
> Boolean: TRUE when the adjusted order date falls within the prior period (shifted back by the comparison offset).

#### Date Comparison
```
CASE [Parameters].[Parameter 6]
WHEN "Prior Period" THEN [Calculation_200128753922646019]
WHEN "Prior Year" THEN 365
END
```
> Returns the number of days to shift back: either the length of the current range (Prior Period) or 365 days (Prior Year).

#### Date Equalizer
```
IF [Calculation_200128753923452932] = True THEN [Calculation_200128753919090690]
ELSEIF [Calculation_200128753923641349] = True THEN [Calculation_200128753919090690] + [Calculation_200128753924255750]
ELSE NULL
END
```
> Aligns prior period dates to current period dates for trend comparison.

#### Date Equalizer with Granularity
```
DATE(CASE [Parameters].[Parameter 5]
WHEN "Week" THEN DATETRUNC('week', [Calculation_200128753924587527])
WHEN "Month" THEN DATETRUNC('month', [Calculation_200128753924587527])
WHEN "Quarter" THEN DATETRUNC('quarter', [Calculation_200128753924587527])
WHEN "Year" THEN DATETRUNC('year', [Calculation_200128753924587527])
END)
```
> Truncates the equalized date to the selected granularity (Week, Month, Quarter, or Year).

### Current Period (CP) KPI Fields

#### CP Sales
```
SUM(IF [Calculation_200128753923452932] = True THEN [Sales] END)
```

#### CP Profit
```
SUM(IF [Calculation_200128753923452932] = True THEN [Profit] END)
```

#### CP Quantity
```
SUM(IF [Calculation_200128753923452932] = True THEN [Quantity] END)
```

#### CP Discount
```
AVG(IF [Calculation_200128753923452932] = True THEN [Discount] END)
```

#### CP Returns
```
COUNTD(IF [Calculation_200128753923452932] = True AND [Returned] = "Yes" THEN [Order ID] END)
```

#### CP Days To Ship
```
AVG(IF [Calculation_200128753923452932] = True THEN [Calculation_633318743171944449] END)
```

#### CP Profit Ratio
```
[CP Sales (copy 3)] / [Calculation_200128753924792328]
```
> Ratio of CP Profit to CP Sales.

### Prior Period (PP) KPI Fields

#### PP Sales
```
SUM(IF [Calculation_200128753923641349] = True THEN [Sales] END)
```

#### PP Profit
```
SUM(IF [Calculation_200128753923641349] = True THEN [Profit] END)
```

#### PP Quantity
```
SUM(IF [Calculation_200128753923641349] = True THEN [Quantity] END)
```

#### PP Discount
```
AVG(IF [Calculation_200128753923641349] = True THEN [Discount] END)
```

#### PP Returns
```
COUNTD(IF [Calculation_200128753923641349] = True AND [Returned] = "Yes" THEN [Order ID] END)
```

#### PP Days To Ship
```
AVG(IF [Calculation_200128753923641349] = True THEN [Calculation_633318743171944449] END)
```

#### PP Profit Ratio
```
[CP Profit (copy 2)] / [CP Sales (copy)]
```
> Ratio of PP Profit to PP Sales.

### Difference Fields (CP minus PP)

#### Sales Difference
```
[Calculation_200128753924792328] - [CP Sales (copy)]
```

#### Profit Difference
```
[CP Sales (copy 3)] - [CP Profit (copy 2)]
```

#### Quantity Difference
```
[CP Profit (copy)] - [CP Quantity (copy)]
```

#### Discount Difference
```
[CP Sales (copy 2)] - [CP Discount (copy 2)]
```

#### Returns Difference
```
[CP Sales (copy 4)] - [CP Returns (copy)]
```

#### Days to Ship Difference
```
[CP Discount (copy)] - [CP Days To Ship (copy)]
```

#### Profit Ratio Difference
```
([Calculation_408983187798249472] - [CP Profit Ratio (copy)]) * 100
```

### Base Measure Fields

#### Days to Ship
```
[Ship Date] - [Order Date]
```

#### Returns
```
COUNTD(IF [Returned] = "Yes" THEN [Order ID] END)
```

#### Profit Ratio
```
SUM([Profit]) / SUM([Sales])
```

#### Number of Records
```
1
```

#### Profit (bin)
```
[Profit]
```
> Binned version of the Profit field.

### Dynamic Axis / KPI Switcher Fields

#### Y-Axis
```
CASE [Parameters].[Parameter 7]
WHEN "Sales" THEN [Calculation_200128753924792328]
WHEN "Profit" THEN [CP Sales (copy 3)]
WHEN "Quantity" THEN [CP Profit (copy)]
WHEN "Discount" THEN [CP Sales (copy 2)] * 100
WHEN "Returns" THEN [CP Sales (copy 4)]
WHEN "Days to Ship" THEN [CP Discount (copy)]
WHEN "Profit Ratio" THEN [Calculation_408983187798249472] * 100
END
```
> Dynamically selects the Y-Axis measure based on the Y-Axis parameter.

#### X-Axis
```
CASE [Parameters].[Y-Axis (copy)]
WHEN "Sales" THEN [Calculation_200128753924792328]
WHEN "Profit" THEN [CP Sales (copy 3)]
WHEN "Quantity" THEN [CP Profit (copy)]
WHEN "Discount" THEN [CP Sales (copy 2)] * 100
WHEN "Returns" THEN [CP Sales (copy 4)]
WHEN "Days to Ship" THEN [CP Discount (copy)]
WHEN "Profit Ratio" THEN [Calculation_408983187798249472] * 100
END
```
> Dynamically selects the X-Axis measure based on the X-Axis parameter.

#### Map KPI
```
CASE [Parameters].[Y-Axis (copy 2)]
WHEN "Sales" THEN [Calculation_200128753924792328]
WHEN "Profit" THEN [CP Sales (copy 3)]
WHEN "Quantity" THEN [CP Profit (copy)]
WHEN "Discount" THEN [CP Sales (copy 2)] * 100
WHEN "Returns" THEN [CP Sales (copy 4)]
WHEN "Days to Ship" THEN [CP Discount (copy)]
WHEN "Profit Ratio" THEN [Calculation_408983187798249472] * 100
END
```
> Dynamically selects the Map KPI measure.

#### Map KPI Difference
```
CASE [Parameters].[Y-Axis (copy 2)]
WHEN "Sales" THEN [Calculation_705094862170877952]
WHEN "Profit" THEN [Sales Difference (copy 3)]
WHEN "Quantity" THEN [Profit Difference (copy)]
WHEN "Discount" THEN -[Profit Difference (copy 2)]
WHEN "Returns" THEN -[Profit Difference (copy 3)]
WHEN "Days to Ship" THEN -[Sales Difference (copy)]
WHEN "Profit Ratio" THEN [Sales Difference (copy 2)]
END
```
> Dynamically selects the Map KPI difference; negates Discount, Returns, and Days to Ship so "up" is always good.

#### Map KPI Difference >= 0
```
[Map KPI (copy)] >= 0
```
> Boolean for color-coding: TRUE when the KPI difference is non-negative.

### Label / Formatting Fields

#### Y-Axis Label
```
UPPER([Parameters].[Parameter 7])
```

#### X-Axis Label
```
UPPER([Parameters].[Y-Axis (copy)])
```

#### Y-Axis Prefix
```
CASE [Parameters].[Parameter 7]
WHEN "Sales" THEN "$"
WHEN "Profit" THEN "$"
WHEN "Quantity" THEN ""
WHEN "Discount" THEN ""
WHEN "Returns" THEN ""
WHEN "Days to Ship" THEN ""
WHEN "Profit Ratio" THEN ""
END
```

#### Y-Axis Suffix
```
CASE [Parameters].[Parameter 7]
WHEN "Sales" THEN ""
WHEN "Profit" THEN ""
WHEN "Quantity" THEN ""
WHEN "Discount" THEN "%"
WHEN "Returns" THEN ""
WHEN "Days to Ship" THEN ""
WHEN "Profit Ratio" THEN "%"
END
```

#### X-Axis Prefix
```
CASE [Parameters].[Y-Axis (copy)]
WHEN "Sales" THEN "$"
WHEN "Profit" THEN "$"
WHEN "Quantity" THEN ""
WHEN "Discount" THEN ""
WHEN "Returns" THEN ""
WHEN "Days to Ship" THEN ""
WHEN "Profit Ratio" THEN ""
END
```

#### X-Axis Suffix
```
CASE [Parameters].[Y-Axis (copy)]
WHEN "Sales" THEN ""
WHEN "Profit" THEN ""
WHEN "Quantity" THEN ""
WHEN "Discount" THEN "%"
WHEN "Returns" THEN ""
WHEN "Days to Ship" THEN ""
WHEN "Profit Ratio" THEN "%"
END
```

#### Map KPI Prefix
```
CASE [Parameters].[Y-Axis (copy 2)]
WHEN "Sales" THEN "$"
WHEN "Profit" THEN "$"
WHEN "Quantity" THEN ""
WHEN "Discount" THEN ""
WHEN "Returns" THEN ""
WHEN "Days to Ship" THEN ""
WHEN "Profit Ratio" THEN ""
END
```

#### Map KPI Suffix
```
CASE [Parameters].[Y-Axis (copy 2)]
WHEN "Sales" THEN ""
WHEN "Profit" THEN ""
WHEN "Quantity" THEN ""
WHEN "Discount" THEN "%"
WHEN "Returns" THEN ""
WHEN "Days to Ship" THEN ""
WHEN "Profit Ratio" THEN "%"
END
```

### Region / Map Fields

#### Region = Region Parameter
```
[Region] = [Parameters].[Parameter 3]
```
> Boolean: TRUE when the row's region matches the selected Region Parameter.

#### Region Filter
```
[Parameters].[Parameter 3] = "US" OR [Parameters].[Parameter 3] = [Region]
```
> Boolean: TRUE when "US" is selected (show all) or when the region matches.

#### Region Abbreviation
```
LEFT([Region], 1)
```

#### US Map Color
```
CASE [Parameters].[Parameter 3]
WHEN "US" THEN "US"
ELSE "NULL"
END
```
> Returns "US" when the full US map is selected; otherwise "NULL" for region-specific coloring.

#### Filtered State
```
IF [Calculation_200128753908195328] THEN [State]
ELSE NULL
END
```
> Shows the state name only when Region = Region Parameter is TRUE.

#### Filtered State (US)
```
IF [Parameters].[Parameter 3] = "US" THEN [State]
ELSE NULL
END
```
> Shows the state name only when the "US" region is selected.

#### CLICK TO HIGHLIGHT
```
IF [Calculation_200128753908195328] THEN "CLICK TO HIGHLIGHT"
ELSE NULL
END
```

#### CLICK TO HIGHLIGHT (US)
```
IF [Parameters].[Parameter 3] = "US" THEN "CLICK TO HIGHLIGHT"
ELSE NULL
END
```

### Scatter Plot Fields

#### Scatter Plot Breakdown
```
CASE [Parameters].[Parameter 8]
WHEN "Segment" THEN [Segment]
WHEN "Ship Mode" THEN [Ship Mode]
WHEN "Customer Name" THEN [Customer Name]
WHEN "Category" THEN [Category]
WHEN "Sub-Category" THEN [Sub-Category]
WHEN "Product Name" THEN [Product Name]
END
```
> Dynamically selects the detail dimension for the scatter plot based on the Scatter Plot Detail parameter.

### Calculated Fields in Annotations Datasource

#### Insight 1
```
[Parameters].[Parameter 9]
```

#### Insight 1 Indicator
```
CASE [Parameters].[Parameter 10]
WHEN "Positive" THEN "1"
WHEN "Neutral" THEN "2"
WHEN "Negative" THEN "3"
END
```

#### Insight 2
```
[Parameters].[Insight 1 (copy)]
```

#### Insight 2 Indicator
```
CASE [Parameters].[Insight 1 Indicator (copy)]
WHEN "Positive" THEN "1"
WHEN "Neutral" THEN "2"
WHEN "Negative" THEN "3"
END
```

#### Insight 3
```
[Parameters].[Insight 2 (copy)]
```

#### Insight 3 Indicator
```
CASE [Parameters].[Insight 2 Indicator (copy)]
WHEN "Positive" THEN "1"
WHEN "Neutral" THEN "2"
WHEN "Negative" THEN "3"
END
```

---

## 3. Parameters

### Date Parameters

| Caption | Data Type | Default Value | Allowed Values |
|---------|-----------|---------------|----------------|
| Minimum Date | date | `#2016-09-01#` | Any |
| Maximum Date | date | `#2017-03-31#` | Any |
| Date Granularity | string | Month | Week, Month, Quarter, Year |
| Date Comparison | string | Prior Period | Prior Period, Prior Year |

### KPI / Axis Parameters

| Caption | Data Type | Default Value | Allowed Values |
|---------|-----------|---------------|----------------|
| Y-Axis | string | Sales | Days to Ship, Discount, Profit, Profit Ratio, Quantity, Returns, Sales |
| X-Axis | string | Discount | Days to Ship, Discount, Profit, Profit Ratio, Quantity, Returns, Sales |
| Map KPI | string | Sales | Days to Ship, Discount, Profit, Profit Ratio, Quantity, Returns, Sales |

### Region & Scatter Plot Parameters

| Caption | Data Type | Default Value | Allowed Values |
|---------|-----------|---------------|----------------|
| Region Parameter | string | East | US, Central, East, South, West |
| Scatter Plot Detail | string | Sub-Category | Segment, Ship Mode, Customer Name, Category, Sub-Category, Product Name |

### Annotation Parameters

| Caption | Data Type | Default Value | Allowed Values |
|---------|-----------|---------------|----------------|
| Insight 1 | string | "Sales are up on average nationwide across all three of our product categories. How can we improve profitablility of Furniture?" | Any |
| Insight 1 Indicator | string | Positive | Positive, Neutral, Negative |
| Insight 2 | string | "Let's keep an eye on days to ship as it was up slightly in three of four regions. Standard Class shipping appears to be the culprit." | Any |
| Insight 2 Indicator | string | Neutral | Positive, Neutral, Negative |
| Insight 3 | string | "Profit ratio was down in the East driven by a -134% profit ratio in Machines; this needs to be addressed immediately." | Any |
| Insight 3 Indicator | string | Negative | Positive, Neutral, Negative |

---

## 4. Filters

### Unique Filter Patterns

The workbook uses several recurring filter patterns across its worksheets. Below are the distinct filter types (deduplicated):

#### Region Filters

| Filter Column | Type | Condition |
|--------------|------|-----------|
| `[Sample - Superstore].[none:Region:nk]` | categorical / except | Exclude NULL members from Region |
| `[Sample - Superstore].[none:Region:nk]` | categorical / member | = "Central" |
| `[Sample - Superstore].[none:Region:nk]` | categorical / member | = "East" |
| `[Sample - Superstore].[none:Region:nk]` | categorical / member | = "South" |
| `[Sample - Superstore].[none:Region:nk]` | categorical / member | = "West" |

#### Region Filter (Calculated)

| Filter Column | Type | Condition |
|--------------|------|-----------|
| `[Sample - Superstore].[none:Calculation_705094862173495298:nk]` | categorical / member | = true |
> This is the "Region Filter" calculated field: `[Parameters].[Parameter 3] = "US" OR [Parameters].[Parameter 3] = [Region]`. Applied across many worksheets to show only rows matching the selected region.

#### Dashboard Filter (Annotations datasource)

| Filter Column | Type | Condition |
|--------------|------|-----------|
| `[federated...].[none:Dashboard:nk]` | categorical / member | = "ANNOTATIONS" |
| `[federated...].[none:Dashboard:nk]` | categorical / member | = "DESCRIPTIVE" |
| `[federated...].[none:Dashboard:nk]` | categorical / member | = "PRESCRIPTIVE" |

#### Action Filters (generated by dashboard actions)

These are crossjoin filters generated by Tableau dashboard highlight/select actions:

| Filter Column | Levels |
|--------------|--------|
| `Action (Region = Region Parameter, Country, State)` | [Region = Region Parameter], [Country], [State] |
| `Action (Region = Region Parameter, Region Abbreviation, Region)` | [Region = Region Parameter], [Region Abbreviation], [Region] |
| `Action (Region = Region Parameter, State)` | [Region = Region Parameter], [State] |
| `Action (US Map Color, State)` | [US Map Color], [State] |
| `Action (Country, State)` | [Country], [State] |
| `Action (State)` | [State] |
| `Action (Insight 2 Indicator)` | [Insight 1 Indicator (copy)] |

#### Measure Names Filters

| Filter Column | Allowed Measures |
|--------------|-----------------|
| `[:Measure Names]` | CP Discount, CP Days To Ship |
| `[:Measure Names]` | CP Profit Ratio, PP Profit Ratio |
| `[:Measure Names]` | CP Sales, PP Sales |

---

## 5. Worksheets and Dashboards

### Worksheets (31 total)

| # | Worksheet Name |
|---|---------------|
| 1 | All US Map |
| 2 | Annotations Button: Active |
| 3 | Annotations Button: Inactive |
| 4 | Central Map |
| 5 | Days to Ship Bullet |
| 6 | Days to Ship Difference |
| 7 | Days to Ship Region Comp |
| 8 | Days to Ship Trend |
| 9 | Descriptive Button: Active |
| 10 | Descriptive Button: Inactive |
| 11 | East Map |
| 12 | Insight 1 |
| 13 | Insight 1 Indicator |
| 14 | Insight 2 |
| 15 | Insight 3 |
| 16 | Insight Indicator 2 |
| 17 | Insight Indicator 3 |
| 18 | Prescriptive Button: Active |
| 19 | Prescriptive Button: Inactive |
| 20 | Prescriptive Map |
| 21 | Prescriptive Scatter Plot |
| 22 | Profit Ratio Bullet |
| 23 | Profit Ratio Callout |
| 24 | Profit Ratio Region Comp |
| 25 | Profit Ratio Trend |
| 26 | Sales Bullet |
| 27 | Sales Callout |
| 28 | Sales Region Comp |
| 29 | Sales Trend |
| 30 | South Map |
| 31 | West Map |
| 32 | X-Axis Label |

### Dashboards (3 total)

| # | Dashboard Name |
|---|---------------|
| 1 | Super: Annotations |
| 2 | Super: Descriptive |
| 3 | Super: Prescriptive |

---

## 6. Architecture Summary

This workbook implements a multi-dashboard analytical tool for the Sample Superstore dataset with three dashboard modes:

1. **Super: Descriptive** -- Traditional KPI dashboard with trend lines, bullet charts, regional comparisons, and maps for Sales, Profit Ratio, and Days to Ship. Uses dynamic date range filtering with current period vs. prior period comparison.

2. **Super: Prescriptive** -- Interactive scatter plot view with dynamic X-axis and Y-axis KPI selection, a detail-level dimension selector, and a geographic map with a parameterized KPI overlay.

3. **Super: Annotations** -- Insight annotation layer where analysts can input three text insights with sentiment indicators (Positive/Neutral/Negative) via parameters.

### Key Design Patterns

- **Dynamic KPI switching** via CASE statements on parameters (Y-Axis, X-Axis, Map KPI) that select among 7 measures: Sales, Profit, Quantity, Discount, Returns, Days to Ship, and Profit Ratio.
- **Current Period / Prior Period comparison** using boolean calculated fields that filter rows into CP or PP buckets, with support for both "Prior Period" (same-length offset) and "Prior Year" (365-day offset) modes.
- **Date normalization** by adding 365 days to all order dates, allowing the 2014-2017 source data to be analyzed as if it were a more recent period.
- **Region drill-down** via a Region Parameter that supports "US" (all regions) plus the four individual regions (Central, East, South, West), with separate map worksheets for each.
- **Dashboard navigation** using Active/Inactive button worksheet pairs for each dashboard mode, controlled by dashboard actions.
- **Prefix/Suffix formatting** fields that dynamically add "$" or "%" symbols based on the selected KPI.
