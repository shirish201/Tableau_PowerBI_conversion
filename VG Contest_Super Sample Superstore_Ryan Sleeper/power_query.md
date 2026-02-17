# Power Query M Scripts for Super Sample Superstore

**Purpose:** Reproduce the Tableau data connections in Power BI using Power Query (M language).

> **File paths:** Update the `FilePath` variables below to point to your local Excel files. The Tableau workbook used two Excel sources:
> 1. `Sample - Superstore.xlsx` (Orders and Returns sheets)
> 2. `Super Sample Superstore TOC.xlsx` (Sheet1 -- annotations/navigation helper)

---

## 1. Orders Table

Loads the `Orders$` sheet from the Superstore Excel file with correct column types.

```m
let
    // UPDATE THIS PATH to your local file location
    FilePath = "C:\Data\Sample - Superstore.xlsx",

    Source = Excel.Workbook(File.Contents(FilePath), null, true),
    Orders_Sheet = Source{[Item="Orders", Kind="Sheet"]}[Data],
    PromotedHeaders = Table.PromoteHeaders(Orders_Sheet, [PromoteAllScalars=true]),

    // Set column types to match the Tableau schema
    TypedColumns = Table.TransformColumnTypes(PromotedHeaders, {
        {"Row ID", Int64.Type},
        {"Order ID", type text},
        {"Order Date", type date},
        {"Ship Date", type date},
        {"Ship Mode", type text},
        {"Customer ID", type text},
        {"Customer Name", type text},
        {"Segment", type text},
        {"Country", type text},
        {"City", type text},
        {"State", type text},
        {"Postal Code", Int64.Type},
        {"Region", type text},
        {"Product ID", type text},
        {"Category", type text},
        {"Sub-Category", type text},
        {"Product Name", type text},
        {"Sales", type number},
        {"Quantity", Int64.Type},
        {"Discount", type number},
        {"Profit", type number}
    }),

    // Add calculated columns that are best done at the query level

    // Order Date 2017: shift dates forward by 365 days (Tableau normalization)
    AddOrderDate2017 = Table.AddColumn(TypedColumns, "Order Date 2017",
        each Date.AddDays([Order Date], 365), type date),

    // Days to Ship: difference between Ship Date and Order Date
    AddDaysToShip = Table.AddColumn(AddOrderDate2017, "Days to Ship",
        each Duration.Days([Ship Date] - [Order Date]), Int64.Type)

in
    AddDaysToShip
```

---

## 2. Returns Table

Loads the `Returns$` sheet. This table will be related to Orders via `Order ID` in the Power BI data model.

```m
let
    // UPDATE THIS PATH (same file as Orders)
    FilePath = "C:\Data\Sample - Superstore.xlsx",

    Source = Excel.Workbook(File.Contents(FilePath), null, true),
    Returns_Sheet = Source{[Item="Returns", Kind="Sheet"]}[Data],
    PromotedHeaders = Table.PromoteHeaders(Returns_Sheet, [PromoteAllScalars=true]),

    TypedColumns = Table.TransformColumnTypes(PromotedHeaders, {
        {"Returned", type text},
        {"Order ID", type text}
    })

in
    TypedColumns
```

---

## 3. Orders with Returns (Merged -- LEFT JOIN)

If you prefer a single flat table matching the Tableau join (LEFT JOIN Orders to Returns on Order ID), use this query instead of separate Orders + Returns tables.

```m
let
    // UPDATE THIS PATH
    FilePath = "C:\Data\Sample - Superstore.xlsx",

    Source = Excel.Workbook(File.Contents(FilePath), null, true),

    // Load Orders
    Orders_Sheet = Source{[Item="Orders", Kind="Sheet"]}[Data],
    OrdersHeaders = Table.PromoteHeaders(Orders_Sheet, [PromoteAllScalars=true]),
    OrdersTyped = Table.TransformColumnTypes(OrdersHeaders, {
        {"Row ID", Int64.Type},
        {"Order ID", type text},
        {"Order Date", type date},
        {"Ship Date", type date},
        {"Ship Mode", type text},
        {"Customer ID", type text},
        {"Customer Name", type text},
        {"Segment", type text},
        {"Country", type text},
        {"City", type text},
        {"State", type text},
        {"Postal Code", Int64.Type},
        {"Region", type text},
        {"Product ID", type text},
        {"Category", type text},
        {"Sub-Category", type text},
        {"Product Name", type text},
        {"Sales", type number},
        {"Quantity", Int64.Type},
        {"Discount", type number},
        {"Profit", type number}
    }),

    // Load Returns
    Returns_Sheet = Source{[Item="Returns", Kind="Sheet"]}[Data],
    ReturnsHeaders = Table.PromoteHeaders(Returns_Sheet, [PromoteAllScalars=true]),
    ReturnsTyped = Table.TransformColumnTypes(ReturnsHeaders, {
        {"Returned", type text},
        {"Order ID", type text}
    }),

    // LEFT JOIN: Orders left join Returns on Order ID
    Merged = Table.NestedJoin(
        OrdersTyped, {"Order ID"},
        ReturnsTyped, {"Order ID"},
        "Returns", JoinKind.LeftOuter
    ),

    // Expand the Returned column from the joined table
    ExpandReturned = Table.ExpandTableColumn(Merged, "Returns", {"Returned"}, {"Returned"}),

    // Add calculated columns
    AddOrderDate2017 = Table.AddColumn(ExpandReturned, "Order Date 2017",
        each Date.AddDays([Order Date], 365), type date),

    AddDaysToShip = Table.AddColumn(AddOrderDate2017, "Days to Ship",
        each Duration.Days([Ship Date] - [Order Date]), Int64.Type)

in
    AddDaysToShip
```

---

## 4. TOC & Annotations Table

Loads the navigation/annotations helper from `Sheet1` of the TOC Excel file.

```m
let
    // UPDATE THIS PATH to your TOC Excel file
    FilePath = "C:\Data\Super Sample Superstore TOC.xlsx",

    Source = Excel.Workbook(File.Contents(FilePath), null, true),
    Sheet1 = Source{[Item="Sheet1", Kind="Sheet"]}[Data],
    PromotedHeaders = Table.PromoteHeaders(Sheet1, [PromoteAllScalars=true]),

    TypedColumns = Table.TransformColumnTypes(PromotedHeaders, {
        {"Dashboard", type text},
        {"Descriptive Value", Int64.Type},
        {"Prescriptive Value", Int64.Type},
        {"Predictive Value", Int64.Type},
        {"Annotations Value", Int64.Type}
    })

in
    TypedColumns
```

---

## 5. Parameter Tables

Power BI uses disconnected tables (or What-If parameters) to replicate Tableau's parameters. Below are Power Query scripts to create each parameter table. After loading, these tables should **not** be related to any other table in the data model.

### Date Granularity

```m
let
    Source = Table.FromRows(
        {{"Week"}, {"Month"}, {"Quarter"}, {"Year"}},
        type table [Date Granularity = text]
    )
in
    Source
```

### Date Comparison

```m
let
    Source = Table.FromRows(
        {{"Prior Period"}, {"Prior Year"}},
        type table [Date Comparison = text]
    )
in
    Source
```

### Y-Axis

```m
let
    Source = Table.FromRows(
        {
            {"Days to Ship"}, {"Discount"}, {"Profit"},
            {"Profit Ratio"}, {"Quantity"}, {"Returns"}, {"Sales"}
        },
        type table [Y-Axis = text]
    )
in
    Source
```

### X-Axis

```m
let
    Source = Table.FromRows(
        {
            {"Days to Ship"}, {"Discount"}, {"Profit"},
            {"Profit Ratio"}, {"Quantity"}, {"Returns"}, {"Sales"}
        },
        type table [X-Axis = text]
    )
in
    Source
```

### Map KPI

```m
let
    Source = Table.FromRows(
        {
            {"Days to Ship"}, {"Discount"}, {"Profit"},
            {"Profit Ratio"}, {"Quantity"}, {"Returns"}, {"Sales"}
        },
        type table [Map KPI = text]
    )
in
    Source
```

### Region Parameter

```m
let
    Source = Table.FromRows(
        {{"US"}, {"Central"}, {"East"}, {"South"}, {"West"}},
        type table [Region Parameter = text]
    )
in
    Source
```

### Scatter Plot Detail

```m
let
    Source = Table.FromRows(
        {
            {"Segment"}, {"Ship Mode"}, {"Customer Name"},
            {"Category"}, {"Sub-Category"}, {"Product Name"}
        },
        type table [Scatter Plot Detail = text]
    )
in
    Source
```

### Insight 1 Indicator

```m
let
    Source = Table.FromRows(
        {{"Positive"}, {"Neutral"}, {"Negative"}},
        type table [Insight 1 Indicator = text]
    )
in
    Source
```

### Insight 2 Indicator

```m
let
    Source = Table.FromRows(
        {{"Positive"}, {"Neutral"}, {"Negative"}},
        type table [Insight 2 Indicator = text]
    )
in
    Source
```

### Insight 3 Indicator

```m
let
    Source = Table.FromRows(
        {{"Positive"}, {"Neutral"}, {"Negative"}},
        type table [Insight 3 Indicator = text]
    )
in
    Source
```

> **Date parameters** (Minimum Date, Maximum Date): Use Power BI's built-in **Date Slicer** controls with a Date table, or create What-If parameters via **Modeling > New Parameter** in Power BI Desktop. These are easier to manage through the UI than via M code.

> **Free-text parameters** (Insight 1, 2, 3 text): Power BI does not natively support free-text parameter input. Options:
> - Use a **text box visual** for static annotations
> - Use **Power Apps visual** embedded in the report for user-editable text
> - Use a **Deneb** or **HTML Content** custom visual

---

## 6. Data Model Relationships

After loading all queries, set up these relationships in Power BI's Model view:

| From | To | Cardinality | Cross-filter | Active |
|---|---|---|---|---|
| `Orders[Order ID]` | `Returns[Order ID]` | Many-to-Many | Single | Yes |

> If using the merged "Orders with Returns" flat table (query #3), no relationship is needed since the data is already joined.

All parameter tables should remain **disconnected** (no relationships). They interact with visuals through `SELECTEDVALUE()` in DAX measures (see `dax_formulas.md`).

---

## 7. Post-Load Setup Checklist

After importing the queries into Power BI:

1. **Mark parameter tables as disconnected** -- verify they have no relationships in Model view
2. **Create slicers** for each parameter table on the appropriate report pages
3. **Set default slicer selections** to match the Tableau defaults:
   - Date Granularity: Month
   - Date Comparison: Prior Period
   - Y-Axis: Sales
   - X-Axis: Discount
   - Map KPI: Sales
   - Region Parameter: East
   - Scatter Plot Detail: Sub-Category
4. **Hide helper columns** like `Row ID`, `Customer ID`, `Product ID`, `Postal Code` from the report view if they are not needed in visuals
5. **Format the Discount column** as percentage in the Orders table
6. **Create a Date dimension table** for proper time intelligence (recommended over the `Order Date 2017` calculated column approach):

```m
let
    StartDate = #date(2014, 1, 1),
    EndDate = #date(2018, 12, 31),
    DateCount = Duration.Days(EndDate - StartDate) + 1,
    DateList = List.Dates(StartDate, DateCount, #duration(1, 0, 0, 0)),
    ToTable = Table.FromList(DateList, Splitter.SplitByNothing(), {"Date"}, null, ExtraValues.Error),
    TypedDate = Table.TransformColumnTypes(ToTable, {{"Date", type date}}),
    AddYear = Table.AddColumn(TypedDate, "Year", each Date.Year([Date]), Int64.Type),
    AddQuarter = Table.AddColumn(AddYear, "Quarter", each "Q" & Text.From(Date.QuarterOfYear([Date])), type text),
    AddMonth = Table.AddColumn(AddQuarter, "Month", each Date.Month([Date]), Int64.Type),
    AddMonthName = Table.AddColumn(AddMonth, "Month Name", each Date.MonthName([Date]), type text),
    AddWeek = Table.AddColumn(AddMonthName, "Week Number", each Date.WeekOfYear([Date]), Int64.Type),
    AddDay = Table.AddColumn(AddWeek, "Day", each Date.Day([Date]), Int64.Type),
    AddDayOfWeek = Table.AddColumn(AddDay, "Day of Week", each Date.DayOfWeekName([Date]), type text),
    AddYearMonth = Table.AddColumn(AddDayOfWeek, "Year-Month",
        each Date.ToText([Date], "yyyy-MM"), type text)
in
    AddYearMonth
```
