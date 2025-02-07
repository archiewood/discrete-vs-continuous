---
title: Discrete vs Continuous Data
---

## Continuous Data

### Scatter chart example

The scatter chart is a type of graph that displays values as individual points plotted along two numerical axes (X and Y). 


```sql daily_stats
select
    date_trunc('week', order_datetime) as week,
    sum(sales) as total_sales,
    sum(case when category = 'Cursed Sporting Goods' then sales else 0 end) as goods_sales
from needful_things.orders
group by all
```

```sql regression
WITH 
coeffs AS (
    SELECT
        regr_slope(total_sales, goods_sales) AS slope,
        regr_intercept(total_sales, goods_sales) AS intercept,
        regr_r2(total_sales, goods_sales) AS r_squared
    FROM ${daily_stats}
)

SELECT 
    min(goods_sales) AS x, 
    max(goods_sales) AS x2, 
    min(goods_sales) * slope + intercept AS y, 
    max(goods_sales) * slope + intercept AS y2, 
    'y = ' || ROUND(slope, 2) || 'x + ' || ROUND(intercept, 0) AS label
FROM coeffs, ${daily_stats}
GROUP BY slope, intercept, r_squared
```

<ScatterPlot
    data={daily_stats}
    y=total_sales
    x=goods_sales
    yGridlines=false
    chartAreaHeight=300
    pointSize=6
>
<ReferenceLine data={regression} x=x y=y x2=x2 y2=y2 label=label fontSize=20/>
</ScatterPlot>






Use scatter charts if you need to:

- Explore relationships between two numeric variables
- Identify patterns, clusters, or outliers
- Visualize cause-and-effect relationships

### Line chart example

The line chart is a type of graph that represents data points connected by lines, typically to show trends over time. 

Use line charts if you need to:

- Track trends over time (e.g., sales, website traffic)
- Compare multiple categories over time
- Identify seasonal patterns and fluctuations
- Analyze the rate of change in data

```sql sales_per_day
select
    date_trunc('day', order_datetime) as day,
    sum(sales) as sales
from orders
where order_datetime > '2021-01-01' 
group by 1
order by 1
```

<LineChart
    data={sales_per_day}
    x=day
    y=sales
    yFmt=usd
    yGridlines=false
    chartAreaHeight=300
/>



### Distribution analysis

Distribution analysis examines how data points are spread across different values. This helps reveal patterns of frequency and variation.

A common way to analyze the distribution of values is with a box plot. Here is an example:

```sql stats_by_category
with sales_per_day as (
    select
        date_trunc('day', order_datetime) as day,
        category,
        sum(sales) as sales
    from orders
    where order_datetime > '2021-01-01' 
    group by all
    order by 1
)

select
    category,
    min(sales) as min,
    max(sales) as max,
    median(sales) as median,
    percentile_cont(0.25) within group (order by sales) as q1,
    percentile_cont(0.75) within group (order by sales) as q3
from sales_per_day
group by 1
order by median desc
```

<BoxPlot
    data={stats_by_category}
    name=category
    min=min
    intervalBottom=q1
    midpoint=median
    intervalTop=q3
    max=max
    yFmt=num0
    yMin=0
    yGridlines=false
    renderer=svg
    chartAreaHeight={400}
/>



### Box plot example

The box plots display the distribution, variability, and outliers of a dataset. 

Use box plots if you need to:

- Understand data distribution and spread
- Identify outliers in a dataset
- Comparing multiple datasets side by side
- Check for symmetry or skewness in a dataset

### Time series data analysis

Time series data analysis helps answer questions like:

Time series data analysis involves examining data points collected over time to discover patterns, trends, and relationships that can help predict future values.


### Area chart example

The area charts are a variation of a line chart where the area beneath the line is shaded to visualize cumulative values and emphasize volume or magnitude changes. 

```sql items_over_time
select 
    item,
    date_trunc('month', order_datetime) as month,
    sum(sales) as sales
from needful_things.orders
group by all
order by sales desc
```

<AreaChart
    data={items_over_time}
    x=month
    y=sales
    series=item
    yGridlines=false
    legend=false
    chartAreaHeight=300
/>



Use area charts if you need to:

- Track trends over time while emphasizing volume
- Represent multiple series
- Display cumulative totals to show dynamics
- Visualize proportions in changing datasets

## Discrete Data


### Bar chart example

The bar charts display categorical or numerical data using rectangular bars. They are very useful for plotting data that is divided into distinct categories, such as product types and customer segments (e.g., age groups). 

```sql bar_data
select
    count(*) as count,
    item
from needful_things.orders
where order_datetime > '2021-12-15'
group by all
limit 8
```

<BarChart
    data={bar_data}
    x=item
    y=count
    yGridlines=false
    yAxisLabels=false
    swapXY
    labels=true
    xAxisTitle=Hello
    chartAreaHeight=300
/>





Use bar charts if you need to:

- Compare values across categories, such as revenue by product
- Show rankings, such as best-selling products in a category
- Display discrete data with distinct groups, such as sales by region
- Handle large numbers of categories clearly

Correlation analysis applies to discrete data as well. You can plot the results of the analysis with heatmaps:

### Heatmap example

The heatmap chart is a 2D matrix with color coding: lighter/paler colors are for pairs with weaker correlation, while darker, more saturated colors are for pairs with stronger correlation. 

```sql heatmap_chart
select
    date_part('month', order_datetime) as month,
    left(strftime('%B', order_datetime),3) AS month_name,
    channel,
    sum(sales) as sales
from needful_things.orders
where date_part('hour',order_datetime) > 22
group by all
order by 1,4 desc
```

<Heatmap
    data={heatmap_chart}
    x=month_name
    y=channel
    value=sales
    valueFmt=num0
    chartAreaHeight=300
/>



Use heatmaps if you need to:

- Identify correlations and relationships
- Highlight density and intensity, such as sales or user activity
- Track geospatial trends, such as weather or population density
- Examine time-based data patterns, e.g., hourly website visits


