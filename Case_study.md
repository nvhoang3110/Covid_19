
# Covid-19 development in Germany: a data analysist case study using Python

![Image source: Federal Statistical Office (Destatis)](https://www.destatis.de/DE/Presse/Pressemitteilungen/Grafiken/Sonstiges/2021/_Statisch/20210331-corona.png?__blob=normal)

In this case study, we will practice our analysis skills as well as using Python to report and visualise the development of Covid-19 pandemic in Germany. 

### Introduction

On 27 January 2020, the first Corana case in Germany was confirmed near Munich, Bavaria. Over the course of more than four years, the pandemic has spread rapidly across the country, resulted in millions of infections and deaths. The main aim of this case study is to look back at the published data and analyse the development of the pandemic in Germany in order to gain more insights. 

With our goal in mind, we will use the data officially published by the Federal Statistical Office (Destatis). After downloading the data from the site, we can perform some data cleaning and transforming, followed by visualizing our data in different formats. Let's start the project by downloading and exploring the data set. 

### Dataset

Covid-19 data in Germany was collected and published by the Federal Statistical Office (Destatis). Over the course of more than four years, the Corona data platform has developed into one of the largest data collections for science. As a result, it is now being expanded into a central platform for the healthcare sector, which is no longer included from the website of the Federal Statistical Office.

You can download the Covid-19 data and other dataset from the central platform for the healthcare [website](https://www.healthcare-datenplattform.de/). It's free for science and research purposes, however you may need to register in order to get access the data. For our case study, we are only interested in the infections federal states (Infektionen Bundesländer) dataset. Here's the [link](https://www.healthcare-datenplattform.de/dataset/infektionen_bundeslaender) for you to download the data directly from the web page. You can also access to the data from the project's repository.

#### Get the data into our machine

Assumed that you have already downloaded the appropriated dataset, the first step is to load it into our machine. 

``` Python
# Import necessary libraries
import pandas as pd  # For data manipulation and analysis
import numpy as np  # For numerical computations

# Provide a path to the dataset
dat_path = "./bl_infektionen.csv"

# Read the data in Comma separated values (CSV) format
dat = pd.read_csv(path)
```

### Data exploration

Let's take a quick look at what we have at hand. An overview about the dataset could be handy:

```python
# Overview about the dataset. 
dat.info()
```

```python
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 16576 entries, 0 to 16575
Data columns (total 8 columns):
 #   Column           Non-Null Count  Dtype
---  ------           --------------  -----
 0   _id              16576 non-null  int64
 1   ags2             16576 non-null  int64
 2   bundesland       16576 non-null  object
 3   datum            16576 non-null  object
 4   bl_inz           16576 non-null  int64
 5   bl_inz_rate      16576 non-null  float64
 6   bl_inz_fix       16576 non-null  int64
 7   bl_inz_rate_fix  16576 non-null  float64
dtypes: float64(2), int64(4), object(2)
memory usage: 1.0+ MB
```
We have a table consisted of 8 columns with 16,576 entries and no missing values. A description from the [webpage](https://www.healthcare-datenplattform.de/dataset/infektionen_bundeslaender) of the dataset provides us with the information that each column contains:

* ```_id```: an ID columns to keep track of the entries

* ```age2```: a number assigned to each federal state. Ranging from 1 to 10

* ```bundesland```: oficial name of the federal state coresponding to the number in the ```age2``` columns

* ```datum```: date in which the entry was recorded

* ```bl_inz```: Confirmed corona cases in the last 7 days

* ```bl_inz_rate```: Confirmed corona cases in the last 7 days per 100,000 inhabitants (as of 31-12-2020)

* ```bl_inz_fix```: Confirmed corona cases in the last 7 days without taking into account late notifications

* ```bl_inz_rate_fix```: Confirmed corona cases in the last 7 days per 100,000 inhabitants (as of 31-12-2020) without taking into account late registrations

Here are some of the actual entries from our dataset. We extract only the first 5 rows from the table:

```Python
# Inspect some rows from the data
dat.head(5)
```

```python
   _id  ags2          bundesland       datum  bl_inz  bl_inz_rate  bl_inz_fix  bl_inz_rate_fix
0    1     1  Schleswig-Holstein  2020-03-01       2          0.1         -99            -99.0
1    2     1  Schleswig-Holstein  2020-03-02       4          0.1         -99            -99.0
2    3     1  Schleswig-Holstein  2020-03-03       5          0.2         -99            -99.0
3    4     1  Schleswig-Holstein  2020-03-04       5          0.2         -99            -99.0
4    5     1  Schleswig-Holstein  2020-03-05       6          0.2         -99            -99.0
```

Other than two columns ```bundesland``` and ```datum``` which have ```object``` data type, all others are numerics, either ```int64``` or ```float64```. We can get a quick summary statistics about them, which should exclude the ```_id``` and also ```ag2``` columns since they serve as counter and numeric annotation for states, respectively and not actual data:

```python
# Drop index columns and describe
dat.drop(["_id", "ags2"], axis = 1).describe()
```

```python
              bl_inz   bl_inz_rate     bl_inz_fix  bl_inz_rate_fix
count   16576.000000  16576.000000   16576.000000     16576.000000
mean    15756.819196    297.203113   14252.366976       264.275857
std     34360.735485    432.822233   31184.949188       404.816576
min         0.000000      0.000000     -99.000000       -99.000000
25%       725.000000     20.500000     508.000000        13.200000
50%      4201.000000    124.700000    3652.500000       113.000000
75%     14100.750000    358.700000   12815.250000       328.325000
max    316242.000000   2659.500000  289072.000000      2490.000000
```

Here we have some common summary statistics indexes like mean, standard deviation, maximum, minimum, etc... The summary date showed here was aggregated for the entire country over the course of three years. In order to get similar result for each individual state, we need to group the data by federal state first and them apply the summarize:

```python
# Group by federal states and describe
dat.drop(["_id", "ags2"], axis = 1).groupby("bundesland").describe()
```

```python
                        bl_inz                                                      ... bl_inz_rate_fix
                         count          mean           std   min      25%      50%  ...             std   min     25%     50%      75%     max       
bundesland                                                                          ...
Baden-Württemberg       1036.0  33547.194981  49839.000256  21.0  3298.25  15197.0  ...      416.716999 -99.0  14.350  128.20  312.875  1952.6       
Berlin                  1036.0   9461.921815  12956.610303   1.0  1147.50   5333.5  ...      335.091636 -99.0  23.750  130.80  292.650  1863.2       
Brandenburg             1036.0   7376.029923  10403.257909   1.0   371.25   3271.5  ...      394.415265 -99.0   5.500  110.55  358.000  1813.7       
Bremen                  1036.0   2005.150579   2651.751369   1.0   167.75    784.5  ...      384.649548 -99.0  20.550  113.45  362.825  1868.6       
Freistaat Bayern        1036.0  44635.736486  67194.406011   9.0  3510.25  17518.5  ...      481.958646 -99.0  16.175  124.55  354.425  2199.9       
Hamburg                 1036.0   5372.041506   7850.645920   2.0   672.50   2367.5  ...      328.989097 -99.0  18.075  100.50  242.325  1891.4       
Hessen                  1036.0  19229.569498  25578.531530   4.0  1412.25   9212.5  ...      389.100930 -99.0  17.525  136.10  341.400  1684.4       
Mecklenburg-Vorpommern  1036.0   4710.052124   7548.123763   0.0   135.75   1456.5  ...      466.504782 -99.0   3.950   84.10  377.525  2490.0       
Niedersachsen           1036.0  25284.948842  36552.550661   1.0  1490.75   7860.5  ...      411.721801 -99.0  11.000   85.85  371.525  2042.6       
Nordrhein-Westfalen     1036.0  52520.077220  70669.257322  92.0  5358.75  24416.0  ...      354.711601 -99.0  17.625  128.10  330.850  1537.5       
Rheinland-Pfalz         1036.0  11611.961390  16632.765225   2.0   893.50   4911.5  ...      360.409474 -99.0  13.200  107.45  301.825  1747.3       
Saarland                1036.0   3232.532819   4816.923615   0.0   282.75   1196.5  ...      498.057441 -99.0  14.925  110.95  359.625  2287.9       
Sachsen                 1036.0  13045.005792  18643.802228   0.0   626.00   7458.5  ...      400.929817 -99.0   6.775  155.55  331.575  2320.1       
Sachsen-Anhalt          1036.0   6383.706564   9904.593369   0.0   194.25   2988.5  ...      425.072827 -99.0   5.175  123.80  287.750  2055.4       
Schleswig-Holstein      1036.0   7797.683398  11281.830762   2.0   429.25   1917.5  ...      361.283262 -99.0   8.800   62.75  328.550  1574.6       
Thüringen               1036.0   5895.494208   9088.435657   0.0   298.00   2950.5  ...      412.595676 -99.0   6.800  132.80  286.050  2218.3       

[16 rows x 32 columns]
```
The same technique can be applied to get the summary statistics for every single day over the period of roughly three years. Note that the resulted table will also have more rows than the previous one:

```python
# Group by date and describe
dat.drop(["_id", "ags2"], axis = 1).groupby("datum").describe()
```

```python
           bl_inz                                                                        ... bl_inz_rate_fix
            count        mean           std     min      25%     50%       75%      max  ...           count  mean  std   min   25%   50%   75%   max
datum                                                                                    ...
2020-03-01   16.0      8.6875     22.828984     0.0     0.75     2.0      2.50     92.0  ...            16.0 -99.0  0.0 -99.0 -99.0 -99.0 -99.0 -99.0
2020-03-02   16.0     11.3125     26.670130     0.0     1.00     2.0      5.00    107.0  ...            16.0 -99.0  0.0 -99.0 -99.0 -99.0 -99.0 -99.0
2020-03-03   16.0     16.3750     34.403246     0.0     1.75     4.5      9.75    136.0  ...            16.0 -99.0  0.0 -99.0 -99.0 -99.0 -99.0 -99.0
2020-03-04   16.0     25.6875     57.376788     0.0     1.75     5.5     13.25    226.0  ...            16.0 -99.0  0.0 -99.0 -99.0 -99.0 -99.0 -99.0
2020-03-05   16.0     35.8125     78.039493     0.0     2.50     7.0     20.00    307.0  ...            16.0 -99.0  0.0 -99.0 -99.0 -99.0 -99.0 -99.0
...           ...         ...           ...     ...      ...     ...       ...      ...  ...             ...   ...  ...   ...   ...   ...   ...   ...
2022-12-27   16.0  11265.2500  13812.516978  1992.0  3413.50  6310.5  14608.75  55143.0  ...            16.0 -99.0  0.0 -99.0 -99.0 -99.0 -99.0 -99.0
2022-12-28   16.0  10887.3750  13114.723811  1823.0  3205.25  6119.5  14315.00  52122.0  ...            16.0 -99.0  0.0 -99.0 -99.0 -99.0 -99.0 -99.0
2022-12-29   16.0  10166.2500  12083.280749  1683.0  3123.50  5645.5  13519.75  48249.0  ...            16.0 -99.0  0.0 -99.0 -99.0 -99.0 -99.0 -99.0
2022-12-30   16.0   9017.3125  10626.221848  1525.0  2668.50  4879.0  11808.75  42345.0  ...            16.0 -99.0  0.0 -99.0 -99.0 -99.0 -99.0 -99.0
2022-12-31   16.0   8549.8125   9795.118765  1438.0  2605.00  4691.0  11312.00  39029.0  ...            16.0 -99.0  0.0 -99.0 -99.0 -99.0 -99.0 -99.0

[1036 rows x 32 columns]
```

### Data visualization

With our initial understanding about the data, it is time for some data visualization. For the shake of our case study, we will forcus only on the confirmed corona cases in the last 7 days, which is data contained in the ```bl_inz``` column. 

#### 7-day prevalence across the country

We first visualize the development of the pandemic across the whole contry during the period of three years. For that before plotting, we need to aggragate our data by date. A simple line plot would fit our purpose nicely:

```python
# Import libraires for plotting
import matplotlib.pyplot as plt
import matplotlib.dates as mdates
import matplotlib.ticker as ticker
import seaborn as sns
import datetime

# Convert the date column into datetime datatype
dat["datum"] = pd.to_datetime(dat["datum"])

# Aggregate data based on date
dat_daily = dat.iloc[:, 3:5].groupby(["datum"]).sum()

dat_daily.reset_index(inplace = True)

# Other pre-setting
title = "Covid-19 in Germany: 7-day prevalence across the country"
subtitle = "Number of confirmed corona cases in the last 7 days across the country"
footer = "Source: healthcare daten platform, healthcare-datenplattform.de"

quarter_year_locator = mdates.MonthLocator(interval = 3)

sns.set_style("darkgrid")
# Plot
sns.set(rc = {"figure.figsize":(14, 8)})

ax = sns.lineplot(data = dat_daily, x = "datum", y = "bl_inz",
                color = "#a50f15",
                legend = "full", lw = 3)

# Some aesthetic modifications
ax.xaxis.set_major_locator(quarter_year_locator)
ax.yaxis.set_major_formatter(ticker.StrMethodFormatter("{x:,.0f}"))

plt.ylabel("7-day prevalence", weight = "bold")
plt.xlabel("Date (Year-Month)", weight = "bold")

ax.text(x = 0.5, y = 1.05, s = title, 
        fontsize = 15, alpha = 1, ha = "center", va = "bottom", weight = "bold", 
        transform = ax.transAxes)

plt.text(x = 0.5, y = 1.02, s = subtitle, alpha = 0.5, 
        fontsize = 10, ha = "center", va = "bottom", transform = ax.transAxes)

plt.text(x = 0.885, y = -0.11, s = footer, alpha = 0.5, 
        fontsize = 10, ha = "center", va = "bottom", transform = ax.transAxes)

# We also display day with the highest record with a scatter and comment
y_cod = dat_daily["bl_inz"].max()

x_cod_min = dat_daily["datum"].iloc[0]

x_cod_max = dat_daily.loc[dat_daily["bl_inz"] == y_cod, "datum"]

plt.scatter(x = x_cod_max, y = y_cod, s = 55, color = "#a50f15")

ax.hlines(y_cod, xmin = x_cod_min, xmax = x_cod_max, ls = "--")
ax.vlines(x_cod_max, ymin = 0, ymax = y_cod, ls = "--")

ax.text(x = x_cod_max + np.timedelta64(1, "M"), y = 1_590_000, 
        s = "Highest record: More\nthan 1.6 million cases", 
        fontsize = 10, alpha = 0.75, weight = "bold") 

# Tighten the layout and display the result
plt.tight_layout()

plt.show()
```
![Covid-19: 7-day prevalence across the country](https://github.com/nvhoang3110/Covid_19/blob/8400b9283f2f260708da8919dbe69ad172c8675b/Plots/fig2.png)

From our plot, we can see that there are three waves of corona virus: the initial wave occured around the end of 2020 until around June next year. The second wave hit about the same time as the first one, but this time the number of infections soared to a much higher degree and reached the record. The final and last one was relatively mild and short compared to the second, which may likely due to the introduction of multiple type of vaccinations and restriction laws that came into practice. Each wave lasted around 6 to 7 months during the winter time, except for the last one which reached its peak at mid summer and late fall.

#### 7-day prevalence by federal states

Let us see how the pandemic developed in each federal state compared to the common trend. For that, a lineplot can be made for each state:

```python
# Some pre-setting
title = "Covid-19 in Germany: 7-day prevalence by federal states"
subtitle = "Number of confirmed corona cases in the last 7 days by federal state"

sns.set(rc = {"figure.figsize" : (14, 8)})
# Plot
ax = sns.lineplot(data = dat, x = "datum", y = "bl_inz",
                hue = "bundesland", palette = "YlOrRd",
                legend = "full", lw = 3)

# Additional aesthetic modifications
ax.yaxis.set_major_formatter(ticker.StrMethodFormatter("{x:,.0f}"))
ax.xaxis.set_major_locator(quarter_year_locator)

plt.ylabel('7-day prevalence', weight = "bold")
plt.xlabel('Date (Year-Month)', weight = "bold")

ax.legend(bbox_to_anchor = (1, 1), title = "Federal state", 
        title_fontproperties = {"size" : 11, "weight" : "bold"})

plt.ylabel("7-day prevalence")
plt.xlabel("Date (Year-Month)")

ax.text(x = 0.5, y = 1.05, s = title, 
        fontsize = 15, alpha = 1, ha = "center", va = "bottom", weight = "bold", 
        transform = ax.transAxes)

plt.text(x = 0.5, y = 1.02, s = subtitle, alpha = 0.5, 
        fontsize = 10, ha = "center", va = "bottom", transform = ax.transAxes)

plt.text(x = 1.03, y = -0.11, s = footer, alpha = 0.5, 
        fontsize = 10, ha = "center", va = "bottom", transform = ax.transAxes)

# Tighten the layout and display the result
plt.tight_layout()

plt.show()
```

![Covid-19: 7-day prevalence by federal states](https://github.com/nvhoang3110/Covid_19/blob/8400b9283f2f260708da8919dbe69ad172c8675b/Plots/fig1.png)

So indeed, overall the common trend of the pandemic development does hold true for all federal states, just with different size and magnitude. It is because there are states that are bigger than the others, housing a lot more residents and thus hit more severely by the disease. 

One biggest drawback of this plot is that it is really difficult to observe the development of the pandemic over time for each individual state, since the lines are stacked on top of each other. This can be fixed my plotting each line for each state one by one, so we can still compare them with the silhoutte of the other lines:

```python
# Some pre-setting
sns.set(rc = {"figure.figsize":(14, 8)})
# Plot
g = sns.relplot(data = dat, x = "datum", y = "bl_inz",
                col = "bundesland", hue = "bundesland",
                kind = "line", palette = sns.color_palette("YlOrRd", 16),   
                linewidth = 4, zorder = 5,
                col_wrap = 4, height = 3, aspect = 1.5, legend = False)

# Add text and silhouettes
for time, ax in g.axes_dict.items():
        ax.text(0.1, 0.85, time,
                transform = ax.transAxes, fontweight="bold")
        ax.tick_params(axis = "x", rotation = 45)
        sns.lineplot(data = dat, x = "datum", y = "bl_inz", units = "bundesland",
                estimator = None, color= ".7", linewidth = 1, ax = ax) 

g.fig.suptitle(title, x = 0.5, y = 0.97, fontsize = 15, alpha = 1, weight = "bold", ha = "center", va = "bottom")
g.fig.text(s = subtitle, x = 0.5, y = 0.95, alpha = 0.5, fontsize = 10, ha = "center", va = "bottom")
g.fig.text(s = footer, x = 0.885, y = 0.003, alpha = 0.5, fontsize = 10, ha = "center", va = "bottom")

# Additional aesthetic modifications
g.set_titles("")
g.set_axis_labels("Date (Year-Month)", "7-day prevalence", weight = "bold")
ax.yaxis.set_major_formatter(ticker.StrMethodFormatter("{x:,.0f}"))
ax.xaxis.set_major_locator(quarter_year_locator)

# Tighten the layout and display the result
plt.tight_layout(pad = 1.75)

plt.show()
```
![Covid-19: 7-day prevalence by federal states](https://github.com/nvhoang3110/Covid_19/blob/8400b9283f2f260708da8919dbe69ad172c8675b/Plots/fig3.png)

Much better. As we stated before, big states with a lot more people like Nordrhein-Westfalen, Baden-Württemberg or Bayern follow the common trend much closer than smaller states such as Bremen or Saarland. In fact, in these states the sign of the first and last wave of Covid-19 is barely visible at all. One possible reason is that we use the same y-axis to house our lines, and since there are states with more and states with less people, though the common trend may hold it is the difference in magnitude that makes us hard to detect the movement of the line in some areas. 

#### Animated choroplethmap

Another approach to plot our data is through the use of choropleth map, which provides an easy way to visualize how a variable varies across a geographic area or show the level of variability within a region. And since our data is time series data, we can make multiple maps and overlap them to make a short animation. 

For that, we need the spatial data of Germany which corresponds to the number of federal states. There are many places on the internet for you can get the data, but the easiest and most available one are from Database of Global Administrative Areas project (GADM). We also need some additional libraries to work with geospatial data and create choropleth map. Note that since we have accessed to data at state level, geospatial data at state level (level 1) is also sufficient:

```python
# Import additional libraries to work with geospatial data and create map
import geopandas as gpd
from gadm import GADMDownloader

# More libraries to work with operating system and create map
import os
from matplotlib.lines import Line2D
from matplotlib.patches import Circle
from matplotlib.colors import LinearSegmentedColormap

# Download shapefile data for Germany. Make sure that your machine has internet connection
downloader = GADMDownloader(version = "4.0")

map_df = downloader.get_shape_data_by_country_name(country_name = "Germany", ad_level = 1)
```
Since our data is a continuous time-series, it is a good idea to bind them into groups based on intervals the original value falls into. In this format, we can have more control over the number of categories displayed on our map, as well as using colors to visualize with greater consistence and precision. To do that, we can define a helper function to categorize our data and then assign the groups into a new column. For map making purpose, usually spliting continuous data into 5 or 6 intervals is sufficient:

```python
# Define intervals to assign our data into
categories = ["0 - 1,000", "1,000 - 10,000", "10,000 - 50,000", "50,000 - 100,000", "100,000 - 200,000", "> 200,000"]

# Define helper function to categorize our data
def get_group(num):
    if 0 <= num <= 1_000:
        return "0 - 1,000"
    elif 1_000 < num <= 10_000:
        return "1,000 - 10,000"
    elif 10_000 < num <= 50_000:
        return "10,000 - 50,000"
    elif 50_000 < num <= 100_000:
        return "50,000 - 100,000"
    elif 100_000 < num <= 200_000:
        return "100,000 - 200,000"
    else:
        return "> 200,000"

# Apply and add category column
dat["cat"] = dat["bl_inz"].apply(get_group).astype("category")

# Reorder our category
dat["cat"] = dat["cat"].cat.reorder_categories(categories)
```
The next step is to match our time series data with spatial data together based on a common column, then make a map for each day and then combine them together to create an animation in the form of a short GIF: 

```python
# Some spatial data transformations
map_df = map_df[["NAME_1", "geometry"]].rename({"NAME_1": "bundesland"}, axis = "columns")

map_df = map_df.replace("Bayern", "Freistaat Bayern")

# Merge two data set together
df_merged = map_df.merge(dat, left_on = ["bundesland"], right_on =  ["bundesland"])

# Define color palette and color function 
col_plot = ['#fef0d9', '#fdd49e', '#fdbb84', '#fc8d59', '#e34a33', '#b30000']

cmap1 =  LinearSegmentedColormap.from_list("mycmap", col_plot)

# Create the custom legend box
legend_box = []

for a, b in zip(col_plot, categories):
    element = Line2D([0], [0], marker = "o", color = a, label = b, 
                markerfacecolor = a, markersize = 14, linewidth = 0)
    legend_box.append(element)

# Create a date column to filter data by date
date_dat = df_merged["datum"].drop_duplicates().dt.strftime("%Y-%m-%d")

# Provide a path to save the plots
plot_path = "./Plot/"

# Initial counter
k = 1

# Plot repeatly using a loop
for i in date_dat:
    # Filter data by date
    plot_dat = df_merged[df_merged["datum"] == i]

    # Plot
    ax = plot_dat.plot(column = "cat", categorical = True, cmap = cmap1, 
                    figsize = (14, 8), linewidth = 0.6,  
                    edgecolor = "#ffffff", legend = True)

    # Add legend box, text
    ax.legend(title = "7-day prevalence", handles = legend_box, 
        fontsize = 8, frameon = False, bbox_to_anchor = (0, 0.58), 
        title_fontproperties = {'weight':'bold', 'size': 8})
    
    ax.text(x = 0.5, y = 0.99, s = title, 
        fontsize = 12, alpha = 1, ha = 'center', va = 'bottom', weight = "bold", 
        transform = ax.transAxes)
    
    plt.text(x = 0.5, y = 0.965, 
        s = "Number of confirmed corona cases in the last 7 days recorded in {} by federal states".format(str(i)), 
        alpha = 0.5, 
        fontsize = 8, ha = "center", va = 'bottom', transform = ax.transAxes)

    plt.figtext(x = 1.40, y = 0, s = footer, alpha = 0.5,  # 1.07
        fontsize = 8, ha = 'center', va = 'bottom', transform = ax.transAxes)

    # Additional aesthetic modifications
    plt.axis("off")
    plt.tight_layout()

    # Provide a file path and save the plot
    filepath = os.path.join(plot_path, "infection_case{}.png".format(str(i)))
    
    chart = ax.get_figure()
    chart.savefig(filepath, dpi = 100) 
    
    k = k + 1
    plt.close()

    # Additional aesthetic modifications
    plt.axis("off")
    plt.tight_layout()
    
    # Save the plot
    filepath = os.path.join(plot_path, "infection_case{}.png".format(str(i)))
    
    chart = ax.get_figure()
    chart.savefig(filepath, dpi = 100)
    
    # Add an unit to the counter and finish plotting
    k = k + 1

    plt.close()

# Import some libraries to create GIF file
import glob
from PIL import Image

# Create a list of image frames
frames = []
imgs = glob.glob("/Plot/*.png")
imgs.sort(key = os.path.getmtime)

for i in imgs:
    new_frame = Image.open(i)
    frames.append(new_frame)

# Save the frames as a GIF file that loops forever
frames[0].save('./Plot/infection_case.gif', 
            format = 'GIF', 
            append_images = frames[1:], 
            save_all = True,
            duration = 200, loop = 0)
```

Open the GIF file after the machine finished running the code at your designated storage location and we got the following animation. Note that it may take a while to finish one loop since we have quite a lot of frames:

![Covid-19: 7-day prevalence by federal states](https://github.com/nvhoang3110/Covid_19/blob/3ede540ffd0e153473da8cd5ac110515348535af/Plots/infection_case.gif)

```python
# Inspect the amount of frames after creating our animation
print(k)
```

```python
1037 # A total of 1036 frames since we start our counter at 1
```

### Future work

Throughout our analysis, we forcused specifically on confirmed corona cases in the last 7 days which contains in the ```bl_inz``` columns. It is also possible to use other index such as data per 100,000 inhabitants - ```bl_inz_rate```, or data without taking into account late registrations - ```bl_inz_fix```. Next, we can use existing sklearn libraries to perform this timeseries analysis and make prediction.

Our dataset is just one among many that described the development of Covid-19 pandemic in Germany. To see things from other perspective, one could further look at the mortal dataset, hospitalizations dataset, vaccination dataset, etc... and perform similar analysis.

### Summary

In out case study, we went over some data analysis and visualization to get a better understand of the development of the Corona pandemic in Germany. We downloaded the data, performed some cleaning and reshaping and then plot the 7-day prevalence using multiple types of bar plot. We also made a choropleth map to present out data by federal states using different color mappings. Thanks for reading!






















































