# DAX Formula Conversions from Tableau Calculated Fields

**Source:** `tableau_analysis.md` (VG Contest - Super Sample Superstore, Ryan Sleeper)

> **Assumptions:**
> - The Orders table is named `Orders`, the Returns table is named `Returns`
> - Tables are related via `Order ID` in the data model
> - Tableau parameters become Power BI **What-If parameters** (which create single-column tables + a measure). Each parameter is referenced as its own table, e.g. `'Minimum Date'[Minimum Date Value]`
> - Calculated columns are marked **(Calculated Column)**; measures are marked **(Measure)**
> - Where Tableau uses row-level IF logic inside an aggregate (e.g. `SUM(IF ... THEN [Sales] END)`), DAX uses `CALCULATE` with filters or `SUMX` with conditionals

---

## Power BI Parameters (What-If / standalone tables)

Create these as What-If parameters or disconnected tables in Power BI:

| Parameter Name | Type | Default | Values |
|---|---|---|---|
| Minimum Date | Date | 2016-09-01 | Date picker |
| Maximum Date | Date | 2017-03-31 | Date picker |
| Date Granularity | Text | Month | Week, Month, Quarter, Year |
| Date Comparison | Text | Prior Period | Prior Period, Prior Year |
| Y-Axis | Text | Sales | Days to Ship, Discount, Profit, Profit Ratio, Quantity, Returns, Sales |
| X-Axis | Text | Discount | Days to Ship, Discount, Profit, Profit Ratio, Quantity, Returns, Sales |
| Map KPI | Text | Sales | Days to Ship, Discount, Profit, Profit Ratio, Quantity, Returns, Sales |
| Region Parameter | Text | East | US, Central, East, South, West |
| Scatter Plot Detail | Text | Sub-Category | Segment, Ship Mode, Customer Name, Category, Sub-Category, Product Name |
| Insight 1 | Text | (long text) | Free text |
| Insight 1 Indicator | Text | Positive | Positive, Neutral, Negative |
| Insight 2 | Text | (long text) | Free text |
| Insight 2 Indicator | Text | Neutral | Positive, Neutral, Negative |
| Insight 3 | Text | (long text) | Free text |
| Insight 3 Indicator | Text | Negative | Positive, Neutral, Negative |

---

## 1. Date & Time Period Fields

### Order Date 2017 (Calculated Column)

```dax
Order Date 2017 = Orders[Order Date] + 365
```

### Days to Ship (Calculated Column)

```dax
Days to Ship = DATEDIFF(Orders[Order Date], Orders[Ship Date], DAY)
```

### Days in Range (Measure)

```dax
Days in Range =
DATEDIFF(
    SELECTEDVALUE('Minimum Date'[Minimum Date Value]),
    SELECTEDVALUE('Maximum Date'[Maximum Date Value]),
    DAY
) + 1
```

### Date Comparison (Measure)

```dax
Date Comparison =
SWITCH(
    SELECTEDVALUE('Date Comparison'[Date Comparison Value]),
    "Prior Period", [Days in Range],
    "Prior Year", 365
)
```

### Date Filter CP (Calculated Column)

```dax
Date Filter CP =
Orders[Order Date 2017] >= SELECTEDVALUE('Minimum Date'[Minimum Date Value])
    && Orders[Order Date 2017] <= SELECTEDVALUE('Maximum Date'[Maximum Date Value])
```

> **Note:** Since calculated columns are evaluated at refresh time and cannot reference What-If parameters, you will likely need to implement Date Filter CP as a **measure-level filter** instead. See the CP/PP measures below which use `CALCULATE` + `FILTER` for the runtime approach.

### Date Filter PP (Calculated Column)

```dax
Date Filter PP =
VAR _offset = [Date Comparison]
RETURN
    Orders[Order Date 2017] >= (SELECTEDVALUE('Minimum Date'[Minimum Date Value]) - _offset)
        && Orders[Order Date 2017] <= (SELECTEDVALUE('Maximum Date'[Maximum Date Value]) - _offset)
```

> **Same note as above** -- best implemented at the measure level via `CALCULATE`.

### Date Equalizer (Calculated Column)

```dax
Date Equalizer =
IF(
    Orders[Date Filter CP],
    Orders[Order Date 2017],
    IF(
        Orders[Date Filter PP],
        Orders[Order Date 2017] + [Date Comparison],
        BLANK()
    )
)
```

### Date Equalizer with Granularity (Measure)

```dax
Date Equalizer with Granularity =
VAR _dateEq = [Date Equalizer]
VAR _gran = SELECTEDVALUE('Date Granularity'[Date Granularity Value])
RETURN
    SWITCH(
        _gran,
        "Week", DATE(YEAR(_dateEq), MONTH(_dateEq), DAY(_dateEq) - WEEKDAY(_dateEq, 2) + 1),
        "Month", DATE(YEAR(_dateEq), MONTH(_dateEq), 1),
        "Quarter", DATE(YEAR(_dateEq), (QUARTER(_dateEq) - 1) * 3 + 1, 1),
        "Year", DATE(YEAR(_dateEq), 1, 1)
    )
```

> **Tip:** In practice, you'd typically use a separate Date dimension table and apply granularity via the visual axis rather than a calculated date truncation.

---

## 2. Current Period (CP) KPI Measures

These measures filter to rows where `Order Date 2017` falls within the selected date range.

### CP Sales

```dax
CP Sales =
CALCULATE(
    SUM(Orders[Sales]),
    FILTER(
        Orders,
        Orders[Order Date 2017] >= MIN('Minimum Date'[Minimum Date Value])
            && Orders[Order Date 2017] <= MAX('Maximum Date'[Maximum Date Value])
    )
)
```

### CP Profit

```dax
CP Profit =
CALCULATE(
    SUM(Orders[Profit]),
    FILTER(
        Orders,
        Orders[Order Date 2017] >= MIN('Minimum Date'[Minimum Date Value])
            && Orders[Order Date 2017] <= MAX('Maximum Date'[Maximum Date Value])
    )
)
```

### CP Quantity

```dax
CP Quantity =
CALCULATE(
    SUM(Orders[Quantity]),
    FILTER(
        Orders,
        Orders[Order Date 2017] >= MIN('Minimum Date'[Minimum Date Value])
            && Orders[Order Date 2017] <= MAX('Maximum Date'[Maximum Date Value])
    )
)
```

### CP Discount

```dax
CP Discount =
CALCULATE(
    AVERAGE(Orders[Discount]),
    FILTER(
        Orders,
        Orders[Order Date 2017] >= MIN('Minimum Date'[Minimum Date Value])
            && Orders[Order Date 2017] <= MAX('Maximum Date'[Maximum Date Value])
    )
)
```

### CP Returns

```dax
CP Returns =
CALCULATE(
    DISTINCTCOUNT(Orders[Order ID]),
    FILTER(
        Orders,
        Orders[Order Date 2017] >= MIN('Minimum Date'[Minimum Date Value])
            && Orders[Order Date 2017] <= MAX('Maximum Date'[Maximum Date Value])
    ),
    RELATEDTABLE(Returns),
    Returns[Returned] = "Yes"
)
```

> **Alternative** using `CALCULATE` + `FILTER` on the joined model:
>
> ```dax
> CP Returns =
> CALCULATE(
>     DISTINCTCOUNT(Orders[Order ID]),
>     FILTER(
>         Orders,
>         Orders[Order Date 2017] >= MIN('Minimum Date'[Minimum Date Value])
>             && Orders[Order Date 2017] <= MAX('Maximum Date'[Maximum Date Value])
>             && RELATED(Returns[Returned]) = "Yes"
>     )
> )
> ```

### CP Days To Ship

```dax
CP Days To Ship =
CALCULATE(
    AVERAGE(Orders[Days to Ship]),
    FILTER(
        Orders,
        Orders[Order Date 2017] >= MIN('Minimum Date'[Minimum Date Value])
            && Orders[Order Date 2017] <= MAX('Maximum Date'[Maximum Date Value])
    )
)
```

### CP Profit Ratio

```dax
CP Profit Ratio =
DIVIDE([CP Profit], [CP Sales], 0)
```

---

## 3. Prior Period (PP) KPI Measures

These shift the date window backward by the `[Date Comparison]` offset.

### PP Date Filter Helper (reusable variable pattern)

> All PP measures follow this pattern -- the date window is shifted back by either the number of days in the range (Prior Period) or 365 days (Prior Year).

### PP Sales

```dax
PP Sales =
VAR _minDate = MIN('Minimum Date'[Minimum Date Value])
VAR _maxDate = MAX('Maximum Date'[Maximum Date Value])
VAR _offset = [Date Comparison]
RETURN
CALCULATE(
    SUM(Orders[Sales]),
    FILTER(
        Orders,
        Orders[Order Date 2017] >= (_minDate - _offset)
            && Orders[Order Date 2017] <= (_maxDate - _offset)
    )
)
```

### PP Profit

```dax
PP Profit =
VAR _minDate = MIN('Minimum Date'[Minimum Date Value])
VAR _maxDate = MAX('Maximum Date'[Maximum Date Value])
VAR _offset = [Date Comparison]
RETURN
CALCULATE(
    SUM(Orders[Profit]),
    FILTER(
        Orders,
        Orders[Order Date 2017] >= (_minDate - _offset)
            && Orders[Order Date 2017] <= (_maxDate - _offset)
    )
)
```

### PP Quantity

```dax
PP Quantity =
VAR _minDate = MIN('Minimum Date'[Minimum Date Value])
VAR _maxDate = MAX('Maximum Date'[Maximum Date Value])
VAR _offset = [Date Comparison]
RETURN
CALCULATE(
    SUM(Orders[Quantity]),
    FILTER(
        Orders,
        Orders[Order Date 2017] >= (_minDate - _offset)
            && Orders[Order Date 2017] <= (_maxDate - _offset)
    )
)
```

### PP Discount

```dax
PP Discount =
VAR _minDate = MIN('Minimum Date'[Minimum Date Value])
VAR _maxDate = MAX('Maximum Date'[Maximum Date Value])
VAR _offset = [Date Comparison]
RETURN
CALCULATE(
    AVERAGE(Orders[Discount]),
    FILTER(
        Orders,
        Orders[Order Date 2017] >= (_minDate - _offset)
            && Orders[Order Date 2017] <= (_maxDate - _offset)
    )
)
```

### PP Returns

```dax
PP Returns =
VAR _minDate = MIN('Minimum Date'[Minimum Date Value])
VAR _maxDate = MAX('Maximum Date'[Maximum Date Value])
VAR _offset = [Date Comparison]
RETURN
CALCULATE(
    DISTINCTCOUNT(Orders[Order ID]),
    FILTER(
        Orders,
        Orders[Order Date 2017] >= (_minDate - _offset)
            && Orders[Order Date 2017] <= (_maxDate - _offset)
            && RELATED(Returns[Returned]) = "Yes"
    )
)
```

### PP Days To Ship

```dax
PP Days To Ship =
VAR _minDate = MIN('Minimum Date'[Minimum Date Value])
VAR _maxDate = MAX('Maximum Date'[Maximum Date Value])
VAR _offset = [Date Comparison]
RETURN
CALCULATE(
    AVERAGE(Orders[Days to Ship]),
    FILTER(
        Orders,
        Orders[Order Date 2017] >= (_minDate - _offset)
            && Orders[Order Date 2017] <= (_maxDate - _offset)
    )
)
```

### PP Profit Ratio

```dax
PP Profit Ratio =
DIVIDE([PP Profit], [PP Sales], 0)
```

---

## 4. Difference Fields (CP minus PP) -- Measures

### Sales Difference

```dax
Sales Difference = [CP Sales] - [PP Sales]
```

### Profit Difference

```dax
Profit Difference = [CP Profit] - [PP Profit]
```

### Quantity Difference

```dax
Quantity Difference = [CP Quantity] - [PP Quantity]
```

### Discount Difference

```dax
Discount Difference = [CP Discount] - [PP Discount]
```

### Returns Difference

```dax
Returns Difference = [CP Returns] - [PP Returns]
```

### Days to Ship Difference

```dax
Days to Ship Difference = [CP Days To Ship] - [PP Days To Ship]
```

### Profit Ratio Difference

```dax
Profit Ratio Difference = ([CP Profit Ratio] - [PP Profit Ratio]) * 100
```

---

## 5. Base Measure Fields

### Days to Ship (Calculated Column)

```dax
Days to Ship = DATEDIFF(Orders[Order Date], Orders[Ship Date], DAY)
```

> Already defined above in Section 1.

### Returns (Measure)

```dax
Returns =
CALCULATE(
    DISTINCTCOUNT(Orders[Order ID]),
    FILTER(
        RELATEDTABLE(Returns),
        Returns[Returned] = "Yes"
    )
)
```

> **Alternative** if Returns is merged into Orders:
> ```dax
> Returns =
> CALCULATE(
>     DISTINCTCOUNT(Orders[Order ID]),
>     Orders[Returned] = "Yes"
> )
> ```

### Profit Ratio (Measure)

```dax
Profit Ratio = DIVIDE(SUM(Orders[Profit]), SUM(Orders[Sales]), 0)
```

### Number of Records (Calculated Column)

```dax
Number of Records = 1
```

> In DAX you can also just use `COUNTROWS(Orders)` as a measure instead.

### Profit (bin) (Calculated Column)

```dax
Profit Bin =
VAR _binSize = 100  -- adjust bin size as needed
RETURN
    FLOOR(Orders[Profit], _binSize)
```

> The original Tableau field is a bin on Profit. Adjust `_binSize` to match the bin width used in the workbook.

---

## 6. Dynamic Axis / KPI Switcher Measures

### Y-Axis

```dax
Y-Axis =
SWITCH(
    SELECTEDVALUE('Y-Axis'[Y-Axis Value]),
    "Sales", [CP Sales],
    "Profit", [CP Profit],
    "Quantity", [CP Quantity],
    "Discount", [CP Discount] * 100,
    "Returns", [CP Returns],
    "Days to Ship", [CP Days To Ship],
    "Profit Ratio", [CP Profit Ratio] * 100
)
```

### X-Axis

```dax
X-Axis =
SWITCH(
    SELECTEDVALUE('X-Axis'[X-Axis Value]),
    "Sales", [CP Sales],
    "Profit", [CP Profit],
    "Quantity", [CP Quantity],
    "Discount", [CP Discount] * 100,
    "Returns", [CP Returns],
    "Days to Ship", [CP Days To Ship],
    "Profit Ratio", [CP Profit Ratio] * 100
)
```

### Map KPI

```dax
Map KPI =
SWITCH(
    SELECTEDVALUE('Map KPI'[Map KPI Value]),
    "Sales", [CP Sales],
    "Profit", [CP Profit],
    "Quantity", [CP Quantity],
    "Discount", [CP Discount] * 100,
    "Returns", [CP Returns],
    "Days to Ship", [CP Days To Ship],
    "Profit Ratio", [CP Profit Ratio] * 100
)
```

### Map KPI Difference

```dax
Map KPI Difference =
SWITCH(
    SELECTEDVALUE('Map KPI'[Map KPI Value]),
    "Sales", [Sales Difference],
    "Profit", [Profit Difference],
    "Quantity", [Quantity Difference],
    "Discount", -[Discount Difference],
    "Returns", -[Returns Difference],
    "Days to Ship", -[Days to Ship Difference],
    "Profit Ratio", [Profit Ratio Difference]
)
```

> Discount, Returns, and Days to Ship are negated so that positive = improvement.

### Map KPI Difference >= 0

```dax
Map KPI Positive = [Map KPI Difference] >= 0
```

> Use this in conditional formatting rules to color-code positive vs. negative changes.

---

## 7. Label / Formatting Measures

### Y-Axis Label

```dax
Y-Axis Label = UPPER(SELECTEDVALUE('Y-Axis'[Y-Axis Value]))
```

### X-Axis Label

```dax
X-Axis Label = UPPER(SELECTEDVALUE('X-Axis'[X-Axis Value]))
```

### Y-Axis Prefix

```dax
Y-Axis Prefix =
SWITCH(
    SELECTEDVALUE('Y-Axis'[Y-Axis Value]),
    "Sales", "$",
    "Profit", "$",
    ""
)
```

### Y-Axis Suffix

```dax
Y-Axis Suffix =
SWITCH(
    SELECTEDVALUE('Y-Axis'[Y-Axis Value]),
    "Discount", "%",
    "Profit Ratio", "%",
    ""
)
```

### X-Axis Prefix

```dax
X-Axis Prefix =
SWITCH(
    SELECTEDVALUE('X-Axis'[X-Axis Value]),
    "Sales", "$",
    "Profit", "$",
    ""
)
```

### X-Axis Suffix

```dax
X-Axis Suffix =
SWITCH(
    SELECTEDVALUE('X-Axis'[X-Axis Value]),
    "Discount", "%",
    "Profit Ratio", "%",
    ""
)
```

### Map KPI Prefix

```dax
Map KPI Prefix =
SWITCH(
    SELECTEDVALUE('Map KPI'[Map KPI Value]),
    "Sales", "$",
    "Profit", "$",
    ""
)
```

### Map KPI Suffix

```dax
Map KPI Suffix =
SWITCH(
    SELECTEDVALUE('Map KPI'[Map KPI Value]),
    "Discount", "%",
    "Profit Ratio", "%",
    ""
)
```

---

## 8. Region / Map Fields

### Region = Region Parameter (Calculated Column)

```dax
Region Matches Parameter =
Orders[Region] = SELECTEDVALUE('Region Parameter'[Region Parameter Value])
```

> **Note:** This won't work as a calculated column since it references a slicer. Use as a **measure** instead:

```dax
Region Matches Parameter =
IF(
    MAX(Orders[Region]) = SELECTEDVALUE('Region Parameter'[Region Parameter Value]),
    TRUE(),
    FALSE()
)
```

### Region Filter (Measure)

```dax
Region Filter =
VAR _param = SELECTEDVALUE('Region Parameter'[Region Parameter Value])
RETURN
    _param = "US" || _param = MAX(Orders[Region])
```

> In Power BI, you'd typically apply this as a visual-level filter or use `CALCULATE` with this condition rather than creating a standalone boolean measure.

### Region Abbreviation (Calculated Column)

```dax
Region Abbreviation = LEFT(Orders[Region], 1)
```

### US Map Color (Measure)

```dax
US Map Color =
IF(
    SELECTEDVALUE('Region Parameter'[Region Parameter Value]) = "US",
    "US",
    "NULL"
)
```

### Filtered State (Measure)

```dax
Filtered State =
IF(
    [Region Matches Parameter],
    MAX(Orders[State]),
    BLANK()
)
```

### Filtered State US (Measure)

```dax
Filtered State US =
IF(
    SELECTEDVALUE('Region Parameter'[Region Parameter Value]) = "US",
    MAX(Orders[State]),
    BLANK()
)
```

### CLICK TO HIGHLIGHT (Measure)

```dax
Click To Highlight =
IF(
    [Region Matches Parameter],
    "CLICK TO HIGHLIGHT",
    BLANK()
)
```

### CLICK TO HIGHLIGHT US (Measure)

```dax
Click To Highlight US =
IF(
    SELECTEDVALUE('Region Parameter'[Region Parameter Value]) = "US",
    "CLICK TO HIGHLIGHT",
    BLANK()
)
```

---

## 9. Scatter Plot Fields

### Scatter Plot Breakdown (Measure)

```dax
Scatter Plot Breakdown =
SWITCH(
    SELECTEDVALUE('Scatter Plot Detail'[Scatter Plot Detail Value]),
    "Segment", MAX(Orders[Segment]),
    "Ship Mode", MAX(Orders[Ship Mode]),
    "Customer Name", MAX(Orders[Customer Name]),
    "Category", MAX(Orders[Category]),
    "Sub-Category", MAX(Orders[Sub-Category]),
    "Product Name", MAX(Orders[Product Name])
)
```

> **Power BI approach:** Dynamic dimension switching is harder in Power BI than Tableau. The recommended pattern is to use **Field Parameters** (available in Power BI Desktop since 2022). Create a Field Parameter containing Segment, Ship Mode, Customer Name, Category, Sub-Category, and Product Name, then place that parameter on the axis. This is far cleaner than using `SWITCH` + `MAX`.

---

## 10. Annotation Fields

### Insight 1 (Measure)

```dax
Insight 1 = SELECTEDVALUE('Insight 1'[Insight 1 Value])
```

### Insight 1 Indicator (Measure)

```dax
Insight 1 Indicator =
SWITCH(
    SELECTEDVALUE('Insight 1 Indicator'[Insight 1 Indicator Value]),
    "Positive", "1",
    "Neutral", "2",
    "Negative", "3"
)
```

### Insight 2 (Measure)

```dax
Insight 2 = SELECTEDVALUE('Insight 2'[Insight 2 Value])
```

### Insight 2 Indicator (Measure)

```dax
Insight 2 Indicator =
SWITCH(
    SELECTEDVALUE('Insight 2 Indicator'[Insight 2 Indicator Value]),
    "Positive", "1",
    "Neutral", "2",
    "Negative", "3"
)
```

### Insight 3 (Measure)

```dax
Insight 3 = SELECTEDVALUE('Insight 3'[Insight 3 Value])
```

### Insight 3 Indicator (Measure)

```dax
Insight 3 Indicator =
SWITCH(
    SELECTEDVALUE('Insight 3 Indicator'[Insight 3 Indicator Value]),
    "Positive", "1",
    "Neutral", "2",
    "Negative", "3"
)
```

---

## Key Tableau-to-DAX Translation Notes

| Tableau Pattern | DAX Equivalent |
|---|---|
| `SUM(IF condition THEN [field] END)` | `CALCULATE(SUM(table[field]), FILTER(table, condition))` |
| `AVG(IF condition THEN [field] END)` | `CALCULATE(AVERAGE(table[field]), FILTER(table, condition))` |
| `COUNTD(IF condition THEN [field] END)` | `CALCULATE(DISTINCTCOUNT(table[field]), FILTER(table, condition))` |
| `CASE ... WHEN ... END` | `SWITCH(expression, value1, result1, ...)` |
| `IF ... ELSEIF ... ELSE ... END` | `IF(condition, result, IF(condition2, result2, else))` or `SWITCH(TRUE(), ...)` |
| `DATETRUNC('month', date)` | `DATE(YEAR(date), MONTH(date), 1)` |
| `DATEDIFF('day', date1, date2)` | `DATEDIFF(date1, date2, DAY)` |
| `LEFT(string, n)` | `LEFT(string, n)` (same) |
| `UPPER(string)` | `UPPER(string)` (same) |
| Parameter reference `[Parameters].[Name]` | `SELECTEDVALUE('ParamTable'[ParamTable Value])` |
| `NULL` | `BLANK()` |
| Calculated Column with parameter reference | Not possible in DAX -- use a **Measure** instead |
| Dynamic dimension (CASE on parameter returning different columns) | Use **Field Parameters** in Power BI |
