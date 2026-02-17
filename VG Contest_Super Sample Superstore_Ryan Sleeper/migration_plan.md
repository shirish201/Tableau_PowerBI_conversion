# Migration Plan: Tableau to Power BI Desktop

**Source:** VG Contest - Super Sample Superstore (Ryan Sleeper)
**Reference files:** `tableau_analysis.md`, `dax_formulas.md`, `power_query.md`

---

## Phase 1: Data Model Setup

### 1.1 Import Data Sources

- [ ] Open Power BI Desktop > Get Data > Excel Workbook
- [ ] Load `Sample - Superstore.xlsx`
  - [ ] Select the **Orders** sheet
  - [ ] Select the **Returns** sheet
- [ ] Load the **TOC & Annotations** workbook (or create manually as an Enter Data table)
  - [ ] Columns: Dashboard, Descriptive Value, Prescriptive Value, Predictive Value, Annotations Value
  - [ ] Rows: ANNOTATIONS, DESCRIPTIVE, PRESCRIPTIVE

> Use the Power Query M scripts from `power_query.md` to set column types and add calculated columns (`Order Date 2017`, `Days to Ship`).

### 1.2 Configure Data Model Relationships

- [ ] In Model view, create relationship: `Orders[Order ID]` -> `Returns[Order ID]` (Many-to-One, Single direction)
- [ ] Keep all parameter tables **disconnected** (no relationships)

### 1.3 Create Parameter Tables

Create each as **Enter Data** or use the M scripts from `power_query.md`:

- [ ] **Date Granularity** -- Week, Month, Quarter, Year
- [ ] **Date Comparison** -- Prior Period, Prior Year
- [ ] **Y-Axis** -- Sales, Profit, Quantity, Discount, Returns, Days to Ship, Profit Ratio
- [ ] **X-Axis** -- (same 7 values)
- [ ] **Map KPI** -- (same 7 values)
- [ ] **Region Parameter** -- US, Central, East, South, West
- [ ] **Scatter Plot Detail** -- Segment, Ship Mode, Customer Name, Category, Sub-Category, Product Name
- [ ] **Insight 1 Indicator** -- Positive, Neutral, Negative
- [ ] **Insight 2 Indicator** -- Positive, Neutral, Negative
- [ ] **Insight 3 Indicator** -- Positive, Neutral, Negative

### 1.4 Create Date Dimension Table

- [ ] Use the Date table M script from `power_query.md` (Section 7)
- [ ] Mark it as the Date table in Power BI (Modeling > Mark as Date Table)
- [ ] Relate `DateDim[Date]` to `Orders[Order Date 2017]`

### 1.5 Create DAX Measures

Enter all measures from `dax_formulas.md` in this order (respecting dependencies):

- [ ] **Base measures:** Days in Range, Date Comparison
- [ ] **CP measures:** CP Sales, CP Profit, CP Quantity, CP Discount, CP Returns, CP Days To Ship, CP Profit Ratio
- [ ] **PP measures:** PP Sales, PP Profit, PP Quantity, PP Discount, PP Returns, PP Days To Ship, PP Profit Ratio
- [ ] **Difference measures:** Sales Difference, Profit Difference, Quantity Difference, Discount Difference, Returns Difference, Days to Ship Difference, Profit Ratio Difference
- [ ] **Dynamic switchers:** Y-Axis, X-Axis, Map KPI, Map KPI Difference, Map KPI Positive
- [ ] **Formatting measures:** Y-Axis Label, X-Axis Label, Y/X/Map Prefix and Suffix
- [ ] **Region measures:** Region Matches Parameter, Region Filter, US Map Color, Filtered State, Filtered State US, Click To Highlight, Click To Highlight US
- [ ] **Scatter Plot:** Scatter Plot Breakdown
- [ ] **Annotation measures:** Insight 1/2/3, Insight 1/2/3 Indicator
- [ ] Organize measures into **Display Folders**: Dates, Current Period, Prior Period, Differences, KPI Switchers, Formatting, Region, Scatter Plot, Annotations

---

## Phase 2: Build "Super: Descriptive" Page

This is the main KPI dashboard. Layout: header bar at top, 5 maps in a row, then 3 KPI columns (Sales, Profit Ratio, Days to Ship) each containing a callout, bullet chart, trend line, and regional comparison.

### 2.1 Page Setup

- [ ] Rename Page 1 to **Descriptive**
- [ ] Set page size to 16:9 (1920 x 1080) or match Tableau's automatic sizing
- [ ] Set page background to white (`#FFFFFF`)

### 2.2 Header Bar

- [ ] Add a **rectangle shape** across the top, background `#34657f` (dark teal), height ~70px
- [ ] Add **text box** inside: "SUPER SAMPLE SUPERSTORE" in white, Corbel font
- [ ] Add **text box**: attribution line "Created by Ryan Sleeper..." in white, Corbel 11pt

### 2.3 Navigation Buttons

| Tableau Worksheet | Power BI Equivalent |
|---|---|
| Descriptive Button: Active | **Button** with filled background (active state) |
| Descriptive Button: Inactive | Same button, toggled style |
| Prescriptive Button: Inactive | **Button** with page navigation action to Prescriptive page |
| Annotations Button: Inactive | **Button** with page navigation action to Annotations page |

- [ ] Create 3 **Buttons** (Format > Buttons > Blank) in the header area
- [ ] Label them: DESCRIPTIVE, PRESCRIPTIVE, ANNOTATIONS
- [ ] Set **Action** = Page Navigation, targeting the respective page
- [ ] Style the active button with darker fill; inactive buttons with lighter fill
- [ ] Font: Corbel, white text

### 2.4 Region Maps Row (5 maps side by side)

| Tableau Worksheet | Mark Type | Power BI Visual |
|---|---|---|
| All US Map | Filled Map (Multipolygon) | **Filled Map** or **Shape Map** |
| Central Map | Filled Map (Multipolygon) | **Filled Map** (filtered to Central) |
| West Map | Filled Map (Multipolygon) | **Filled Map** (filtered to West) |
| East Map | Filled Map (Multipolygon) | **Filled Map** (filtered to East) |
| South Map | Filled Map (Multipolygon) | **Filled Map** (filtered to South) |

- [ ] Add 5 **Filled Map** visuals in a horizontal row below the header
- [ ] For each: Location = `State`, Color saturation = none (uniform fill for highlighting)
- [ ] **All US Map**: no region filter, color by `US Map Color` or uniform teal
- [ ] **Central/East/South/West Maps**: add visual-level filter `Region = "Central"` (etc.)
- [ ] Set map style to minimal (no labels, state boundaries only, light wash)
- [ ] Color: `#34657f` teal for the region fill, `#d4d4d4` light gray for non-highlighted states
- [ ] Add **drillthrough** or **cross-filter** action: clicking a map updates the Region Parameter slicer

> **Alternative:** Use a single **Shape Map** or **ArcGIS Map** with a bookmark/button system to toggle regions, reducing 5 visuals to 1.

### 2.5 Date & Region Parameter Controls

- [ ] Add **Slicer** for Region Parameter (Dropdown or Button style), placed near the maps
- [ ] Add **Slicer** for Minimum Date (Date type, single value)
- [ ] Add **Slicer** for Maximum Date (Date type, single value)
- [ ] Add **Slicer** for Date Granularity (Dropdown: Week/Month/Quarter/Year)
- [ ] Add **Slicer** for Date Comparison (Dropdown: Prior Period / Prior Year)
- [ ] Style all slicers: Corbel font, color `#00313c`

### 2.6 Sales Column (left third)

#### Sales Callout

| Tableau | Power BI |
|---|---|
| Sales Callout (large number + difference) | **Card** or **Multi-row Card** |

- [ ] Add **Card** visual showing `[CP Sales]` measure
- [ ] Format: large font (24-30pt), Calibri, `$` prefix, color `#00313c`
- [ ] Add a second **Card** below/beside it for `[Sales Difference]`
- [ ] Use conditional formatting on the difference: positive = teal (`#34657f`), negative = red (`#fc4237`)

#### Sales Bullet Chart

| Tableau | Power BI |
|---|---|
| Sales Bullet (bar with reference line for PP) | **Bullet Chart** (custom visual from AppSource) or **Bar Chart** with constant line |

- [ ] Install the **Bullet Chart by Microsoft** from AppSource (or use a stacked bar)
- [ ] Value = `[CP Sales]`, Target = `[PP Sales]`
- [ ] Color bar by `[Sales Difference]` using diverging palette: teal (#34657f) to red (#fc4237), center at 0
- [ ] Hide axis labels; keep minimal

> **Alternative without custom visual:** Use a **Clustered Bar Chart** with CP Sales and PP Sales side by side, or a bar chart with a **constant line** for PP Sales.

#### Sales Trend

| Tableau | Power BI |
|---|---|
| Sales Trend (dual-line: CP Sales + PP Sales over equalized date) | **Line Chart** with two measures |

- [ ] Add **Line Chart**
- [ ] X-axis = `Date Equalizer with Granularity` (or Date dimension at the selected granularity)
- [ ] Values = `[CP Sales]` (solid line) and `[PP Sales]` (dashed line)
- [ ] CP line color: teal `#34657f`; PP line color: lighter gray
- [ ] Enable markers on both lines
- [ ] Title: "TREND" (Corbel 10pt, `#00313c`)
- [ ] Gridlines: dotted, light gray `#f3f3f3`

> **Power BI granularity note:** Instead of the calculated `Date Equalizer with Granularity`, use the **Date hierarchy** on the X-axis and let the Date Granularity slicer control drill level via bookmarks, or use a `SWITCH` measure that truncates dates.

#### Sales Region Comparison

| Tableau | Power BI |
|---|---|
| Sales Region Comp (circle marks by region, comparing CP vs PP) | **Dot Plot** (custom visual) or **Dumbbell Chart** |

- [ ] Option A: **Scatter Chart** with Region on the Details, X = `[CP Sales]`, dummy Y, color by Region
- [ ] Option B: Install **Dumbbell Chart** from AppSource -- Axis = Region, Value1 = CP Sales, Value2 = PP Sales
- [ ] Option C: **Clustered Bar Chart** with Region on axis, CP and PP as two values
- [ ] Mark as Circle, color by CP vs PP period
- [ ] Show all 4 regions (Central, East, South, West) or filter to selected

### 2.7 Profit Ratio Column (center third)

Repeat the same 4-visual pattern as Sales:

#### Profit Ratio Callout
- [ ] **Card**: `[CP Profit Ratio]` formatted as percentage (0.0%)
- [ ] **Card**: `[Profit Ratio Difference]` with conditional formatting

#### Profit Ratio Bullet
- [ ] **Bullet Chart**: Value = `[CP Profit Ratio]`, Target = `[PP Profit Ratio]`
- [ ] Color by `[Profit Ratio Difference]` (diverging palette)

#### Profit Ratio Trend
- [ ] **Line Chart**: X = Date, Values = `[CP Profit Ratio]` + `[PP Profit Ratio]`
- [ ] Same styling as Sales Trend

#### Profit Ratio Region Comparison
- [ ] Same visual type as Sales Region Comp, using Profit Ratio measures

### 2.8 Days to Ship Column (right third)

Repeat the same 4-visual pattern:

#### Days to Ship Callout (labeled "Difference" in Tableau)
- [ ] **Card**: `[CP Days To Ship]` formatted as decimal (0.0)
- [ ] **Card**: `[Days to Ship Difference]` with conditional formatting

#### Days to Ship Bullet
- [ ] **Bullet Chart**: Value = `[CP Days To Ship]`, Target = `[PP Days To Ship]`
- [ ] Color by `[Days to Ship Difference]` (diverging palette)

#### Days to Ship Trend
- [ ] **Line Chart**: X = Date, Values = `[CP Days To Ship]` + `[PP Days To Ship]`

#### Days to Ship Region Comparison
- [ ] Same visual type as Sales Region Comp, using Days to Ship measures

---

## Phase 3: Build "Super: Prescriptive" Page

Layout: header with nav buttons, left half = map, right half = scatter plot, bottom = insights row.

### 3.1 Page Setup

- [ ] Add new page, rename to **Prescriptive**
- [ ] Copy header bar and navigation buttons from Descriptive (update active/inactive states)

### 3.2 Parameter Slicers

- [ ] Add **Slicer** for Y-Axis (Dropdown)
- [ ] Add **Slicer** for X-Axis (Dropdown)
- [ ] Add **Slicer** for Map KPI (Dropdown)
- [ ] Add **Slicer** for Scatter Plot Detail (Dropdown)
- [ ] Add **Slicer** for Region Parameter (if not already global)
- [ ] Date slicers (Minimum Date, Maximum Date, Date Comparison)
- [ ] Position slicers along the top area below the header or in a side panel

### 3.3 Prescriptive Map (left half)

| Tableau | Power BI |
|---|---|
| Prescriptive Map (filled map, color = Map KPI Difference, size = Map KPI) | **Filled Map** or **Shape Map** |

- [ ] Add **Filled Map** visual, ~50% width
- [ ] Location = `State`
- [ ] Color saturation = `[Map KPI Difference]`
- [ ] Diverging color scale: teal `#34657f` (positive) through gray `#d9d9d9` (zero) to red `#fc4237` (negative)
- [ ] Tooltip: State name, Map KPI value (with prefix/suffix), Map KPI Difference
- [ ] Visual-level filter: Region Filter = TRUE (or apply Region Parameter slicer)

### 3.4 Prescriptive Scatter Plot (right half)

| Tableau | Power BI |
|---|---|
| Prescriptive Scatter Plot (Circle marks, X = X-Axis, Y = Y-Axis, size = X-Axis, color = Map KPI Difference >= 0, detail = Scatter Plot Breakdown, labels on, clusters) | **Scatter Chart** |

- [ ] Add **Scatter Chart**, ~45% width
- [ ] X-axis = `[X-Axis]` measure
- [ ] Y-axis = `[Y-Axis]` measure
- [ ] Size = `[X-Axis]` (bubble size)
- [ ] Legend/Color = `[Map KPI Positive]` (TRUE = teal, FALSE = red)
- [ ] Details = `[Scatter Plot Breakdown]` measure

> **Dynamic dimension challenge:** The `Scatter Plot Breakdown` measure returns text, which works as a dimension on Details. However, the scatter chart needs one data point per breakdown value. In Power BI, the cleanest approach is:
> - Use **Field Parameters** (Modeling > New Parameter > Fields) containing Segment, Ship Mode, Customer Name, Category, Sub-Category, Product Name
> - Place the Field Parameter on the **Details** well of the scatter chart
> - The Scatter Plot Detail slicer selects which field is active

- [ ] Enable **data labels** (show the breakdown dimension name on each bubble)
- [ ] Set mark transparency to ~14% (219/255) to match Tableau
- [ ] Hide axis titles (the dynamic labels are shown via separate text)
- [ ] Gridlines: off
- [ ] Tooltip: Breakdown name, Category, X-Axis value (with prefix/suffix), Y-Axis value (with prefix/suffix)

### 3.5 X-Axis Label

- [ ] Add a **Card** or **Text Box** below the scatter plot showing `[X-Axis Label]` measure
- [ ] Rotated -90 degrees if possible, or placed horizontally below the chart
- [ ] Corbel 10pt, `#00313c`

### 3.6 Insight Row (bottom of page)

Same as the Annotations page insight row (see Phase 4.3 below). This page shares the same 3 insight + 3 indicator visuals at the bottom.

- [ ] Copy the insight cards from the Annotations page (built in Phase 4)

---

## Phase 4: Build "Super: Annotations" Page

Layout: header with nav buttons, large open area for parameter controls, insight rows at the bottom.

### 4.1 Page Setup

- [ ] Add new page, rename to **Annotations**
- [ ] Copy header bar and navigation buttons (update active/inactive states)

### 4.2 Annotation Parameter Controls

- [ ] Add **Slicer** for Insight 1 Indicator (Buttons: Positive / Neutral / Negative)
- [ ] Add **Slicer** for Insight 2 Indicator (same)
- [ ] Add **Slicer** for Insight 3 Indicator (same)

> **Free-text insight input:** Tableau parameters allow free-text entry for Insight 1/2/3 text. Power BI does not natively support this. Options:
> - **Static text boxes** with hardcoded insight text (simplest)
> - **Power Apps visual** embedded for editable text fields
> - **Deneb/HTML Content visual** with a text input form

### 4.3 Insight Display Rows (3 rows)

Each row has an indicator circle on the left and the insight text on the right.

| Tableau Worksheet | Power BI Equivalent |
|---|---|
| Insight 1 Indicator (circle colored by sentiment) | **Card** with conditional formatting icon, or **Shape** with rules |
| Insight 1 (text display) | **Card** showing `[Insight 1]` measure, or **Text Box** |
| Insight Indicator 2 + Insight 2 | Same pattern |
| Insight Indicator 3 + Insight 3 | Same pattern |

For each of the 3 insight rows:

- [ ] Add a small **Card** or **Image** visual on the far left (~3% width)
  - Conditional formatting: background color based on indicator slicer
    - Positive = green/teal
    - Neutral = amber/gray
    - Negative = red
  - Shape: circle (use a KPI visual or conditional icon set)
- [ ] Add a **Card** or **Text Box** to the right (~90% width)
  - Display the insight text
  - Font: Corbel, `#00313c`, left-aligned
- [ ] Repeat for all 3 rows, stacked vertically at the bottom of the page

> **Power BI pattern:** Use 3 **KPI** or **Card** visuals with conditional formatting rules on the background color that reference the Insight Indicator measures (1 = green, 2 = amber, 3 = red).

---

## Phase 5: Interactivity & Cross-Filtering

### 5.1 Cross-Filtering Between Visuals

Tableau uses dashboard actions (highlight/select) to filter multiple worksheets at once. In Power BI:

- [ ] Set **Edit Interactions** for each visual on the Descriptive page:
  - Maps should **filter** the trend, bullet, callout, and region comp visuals
  - Region comp clicking should **highlight** the corresponding map
- [ ] On the Prescriptive page:
  - Map click should **cross-filter** the scatter plot
  - Scatter plot click should **cross-filter** the map

### 5.2 Page Navigation

- [ ] Configure the 3 header navigation buttons on each page:
  - DESCRIPTIVE button > Action = Page Navigation > Descriptive page
  - PRESCRIPTIVE button > Action = Page Navigation > Prescriptive page
  - ANNOTATIONS button > Action = Page Navigation > Annotations page
- [ ] Style the active page's button as filled (`#34657f`), others as lighter

### 5.3 Slicer Sync Across Pages

- [ ] View > Sync Slicers pane
- [ ] Sync these slicers across all 3 pages:
  - Region Parameter
  - Minimum Date
  - Maximum Date
  - Date Granularity
  - Date Comparison
- [ ] Sync Insight Indicator slicers across Annotations and Prescriptive pages
- [ ] Y-Axis, X-Axis, Map KPI, Scatter Plot Detail slicers only on Prescriptive page

### 5.4 Tooltip Customization

- [ ] For each map visual: create a **tooltip page** or use the default tooltip showing State, KPI value, and difference
- [ ] For the scatter plot: create a **tooltip page** showing Breakdown name, Category, X-Axis value, Y-Axis value with prefix/suffix formatting
- [ ] For bullet charts: tooltip showing CP value, PP value, and Difference
- [ ] For trend lines: tooltip showing Date, CP value, PP value

---

## Phase 6: Styling & Polish

### 6.1 Color Theme

Create a Power BI theme JSON file or apply manually:

| Element | Color | Hex |
|---|---|---|
| Primary (headers, active buttons, positive values) | Dark Teal | `#34657f` |
| Dark text | Almost Black | `#00313c` |
| Negative values / alerts | Red | `#fc4237` |
| Borders and dividers | Light Gray | `#f3f3f3` |
| Inactive/neutral | Medium Gray | `#d4d4d4` |
| Background | White | `#ffffff` |
| Secondary panel | Light Gray | `#e6e6e6` |

Diverging color palette for bullet charts and map:

| Negative end | Center | Positive end |
|---|---|---|
| `#fc4237` (red) | `#d9d9d9` (gray) | `#34657f` (teal) |

### 6.2 Typography

- [ ] Set report-level font: **Corbel** for labels and titles
- [ ] Set **Calibri** for data values and numbers
- [ ] Title font size: 10-12pt
- [ ] Data label font size: 9-12pt
- [ ] Text color: `#00313c` throughout

### 6.3 Gridlines & Axes

- [ ] Trend charts: dotted horizontal gridlines (`#f3f3f3`), no vertical gridlines
- [ ] Bullet charts: hide Y-axis, hide gridlines
- [ ] Scatter plot: hide gridlines, hide axis titles
- [ ] All visuals: remove visual borders

### 6.4 Final Layout Adjustments

- [ ] Align all visuals using Power BI's alignment guides (Format > Align)
- [ ] Ensure consistent spacing between the 3 KPI columns on Descriptive
- [ ] Match the 50/50 split on Prescriptive (map left, scatter right)
- [ ] Insight rows at the bottom should span full width with small indicator circles on the left

---

## Phase 7: Testing & Validation

### 7.1 Data Validation

- [ ] Compare CP Sales total (with default parameters: Sep 2016 - Mar 2017, East region) against the Tableau workbook
- [ ] Verify PP values shift correctly when toggling Date Comparison between Prior Period and Prior Year
- [ ] Spot-check Profit Ratio, Days to Ship, and Returns for a specific region
- [ ] Confirm Region Parameter "US" shows all regions, individual selections filter correctly

### 7.2 Interactivity Testing

- [ ] Click each map region and verify KPI columns update
- [ ] Change Date Granularity slicer and confirm trend charts re-aggregate
- [ ] Switch Y-Axis and X-Axis parameters on Prescriptive and verify scatter plot updates
- [ ] Change Scatter Plot Detail and verify bubbles re-group by new dimension
- [ ] Toggle all 3 Insight Indicators and confirm color changes on Annotations page
- [ ] Test all 3 navigation buttons from every page

### 7.3 Formatting Verification

- [ ] Verify "$" prefix appears for Sales and Profit measures
- [ ] Verify "%" suffix appears for Discount and Profit Ratio measures
- [ ] Confirm no prefix/suffix for Quantity, Returns, Days to Ship
- [ ] Check conditional formatting: positive differences in teal, negative in red

---

## Visual Mapping Reference (Quick Lookup)

| Tableau Worksheet | Tableau Type | Power BI Visual | Notes |
|---|---|---|---|
| All US / Central / East / South / West Map | Filled Map (Multipolygon) | **Filled Map** or **Shape Map** | 5 separate visuals with region filters |
| Sales / Profit Ratio / Days to Ship Callout | Text / BAN (Big Ass Number) | **Card** | Large formatted number |
| Sales / Profit Ratio / Days to Ship Bullet | Bar + Reference Line | **Bullet Chart** (AppSource) | Value = CP, Target = PP, color = Difference |
| Sales / Profit Ratio / Days to Ship Trend | Dual Line Chart | **Line Chart** (2 series) | CP solid + PP dashed, markers on |
| Sales / Profit Ratio / Days to Ship Region Comp | Circle Plot by Region | **Dumbbell Chart** or **Clustered Bar** | CP vs PP per region |
| Days to Ship Difference | Text / BAN | **Card** | Formatted difference value |
| Prescriptive Map | Filled Map (color + size) | **Filled Map** | Color = Map KPI Difference |
| Prescriptive Scatter Plot | Scatter (circle, sized, colored, labeled) | **Scatter Chart** | X/Y dynamic, size, color by positive/negative |
| X-Axis Label | Text | **Card** or **Text Box** | Shows UPPER(X-Axis parameter) |
| Insight 1 / 2 / 3 | Text display | **Card** or **Text Box** | Shows insight text |
| Insight 1 / 2 / 3 Indicator | Colored circle | **Card** with conditional background | Green/amber/red based on indicator |
| Descriptive/Prescriptive/Annotations Buttons | Sheet-as-button | **Button** with Page Navigation action | Active = filled, Inactive = muted |

---

## AppSource Visuals Needed

| Visual | Purpose | Required? |
|---|---|---|
| **Bullet Chart by Microsoft** | Bullet charts for CP vs PP comparison | Recommended (or use bar chart + constant line) |
| **Dumbbell Chart** | Region comparison dots | Optional (can substitute with clustered bar) |
| **Deneb** | Custom Vega-Lite visuals for exact Tableau look | Optional (for pixel-perfect recreation) |
| **HTML Content** | Free-text input for annotations | Optional (only if replicating text parameters) |
