
# Pyber Ride Sharing



1. Pyber is most popular among drivers and riders in Urban cities, followed by Suburban.
2. Pyber is comparitively cheaper in Urban regions.
3. Number of drivers is correlated to number of rides in the city.


```python
import os
import pandas as pd
import numpy as np
import matplotlib
import matplotlib.pyplot as plt

plt.style.use("seaborn")
```


```python
city_filename = os.path.join('raw_data','city_data.csv')
ride_filename = os.path.join('raw_data','ride_data.csv')
```


```python
city_df = pd.read_csv(city_filename)
ride_df = pd.read_csv(ride_filename)
rideshares_df = ride_df.merge(city_df,on="city")

rideshares_df['date'] = pd.to_datetime(rideshares_df['date'])

```

# City Summary 


```python
city_group = rideshares_df.groupby(["city","driver_count","type"])

city_summary = pd.DataFrame()
city_summary["Total Rides (per City)"] = city_group["ride_id"].count()
city_summary["Average Fare ($)"] = city_group["fare"].mean()
city_summary["Total Fare ($)"] = city_group["fare"].sum()
city_summary = city_summary.reset_index()


city_summary.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>city</th>
      <th>driver_count</th>
      <th>type</th>
      <th>Total Rides (per City)</th>
      <th>Average Fare ($)</th>
      <th>Total Fare ($)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Alvarezhaven</td>
      <td>21</td>
      <td>Urban</td>
      <td>31</td>
      <td>23.928710</td>
      <td>741.79</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Alyssaberg</td>
      <td>67</td>
      <td>Urban</td>
      <td>26</td>
      <td>20.609615</td>
      <td>535.85</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Anitamouth</td>
      <td>16</td>
      <td>Suburban</td>
      <td>9</td>
      <td>37.315556</td>
      <td>335.84</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Antoniomouth</td>
      <td>21</td>
      <td>Urban</td>
      <td>22</td>
      <td>23.625000</td>
      <td>519.75</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Aprilchester</td>
      <td>49</td>
      <td>Urban</td>
      <td>19</td>
      <td>21.981579</td>
      <td>417.65</td>
    </tr>
  </tbody>
</table>
</div>



# Bubble Plot Of Ride Sharing Data


```python
colors = {"Urban":"lightsalmon","Suburban":"deepskyblue","Rural":"yellow"}

scatter_plot_df = city_summary[["Total Rides (per City)","Average Fare ($)","type"]]

urban_set = city_summary[city_summary.type == "Urban"]
suburb_set = city_summary[city_summary.type == "Suburban"]
rural_set = city_summary[city_summary.type == "Rural"]
handles = []
ax = urban_set.plot(kind="scatter",
                   x="Total Rides (per City)",y="Average Fare ($)",
                   edgecolor='k',
                   s=city_summary.driver_count*5,
                   color="lightsalmon",
                   label="Urban")
suburb_set.plot(kind="scatter",
               x="Total Rides (per City)",y="Average Fare ($)",
               edgecolor='k',
               s=city_summary.driver_count*5,
               color="deepskyblue",
               label="Suburban",
               ax=ax)
rural_set.plot(kind="scatter",x="Total Rides (per City)",y="Average Fare ($)",
               edgecolor='k',
               s=city_summary.driver_count*5,
               color="yellow",
               label="Rural",
               ax=ax)

plt.legend(title="City types")

plt.xlabel("Total Rides (per City)")

plt.ylabel("Average Fare ($)")

plt.title("PyBer RideSharing Data(2016)")

plt.text(37,35,"Note:\nCircle size correlates with number of drivers per city")
```




    <matplotlib.text.Text at 0x1de42980b00>




```python
plt.show()
```


![png](output_9_0.png)



```python
city_type_group = city_summary.groupby("type")

city_type_summary = pd.DataFrame()

city_type_summary["Total Rides"] = city_type_group["Total Rides (per City)"].sum()
city_type_summary["Total Drivers"] = city_type_group["driver_count"].sum()
city_type_summary["Total Fare ($)"] = city_type_group["Total Fare ($)"].sum()
city_type_summary.reset_index(inplace=True)
city_type_summary.head()
```




<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>type</th>
      <th>Total Rides</th>
      <th>Total Drivers</th>
      <th>Total Fare ($)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Rural</td>
      <td>125</td>
      <td>104</td>
      <td>4255.09</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Suburban</td>
      <td>657</td>
      <td>638</td>
      <td>20335.69</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Urban</td>
      <td>1625</td>
      <td>2607</td>
      <td>40078.34</td>
    </tr>
  </tbody>
</table>
</div>



# Rides by City Type


```python
explode_dict = {"Urban":0.1,"Suburban":0,"Rural":0}
city_type_summary["explode"] = [explode_dict[x] for x in city_type_summary["type"]]
city_type_summary["colors"] = [colors[x] for x in city_type_summary["type"]]

city_type_summary.plot(kind="pie",y="Total Rides", autopct='%1.1f%%',
 startangle=140, shadow=False,explode=city_type_summary.explode,
                       colors=city_type_summary.colors,labels=city_type_summary.type,
                      wedgeprops = { 'linewidth' : 1 , 'edgecolor' : 'lightgrey'})

plt.axis('off')
plt.axis('equal')

plt.title('% of Total Rides by City Type')

plt.show()
```


![png](output_12_0.png)


# Drivers by City Type


```python
city_type_summary.plot(kind="pie",y="Total Drivers", autopct='%1.1f%%',
 startangle=140, shadow=False,explode=city_type_summary.explode,
                       colors=city_type_summary.colors,labels=city_type_summary.type,
                      wedgeprops = { 'linewidth' : 1 , 'edgecolor' : 'lightgrey'})

plt.axis('off')
plt.axis('equal')

plt.title('% of Total Drivers by City Type')

plt.show()
```


![png](output_14_0.png)


# Fare by City Type


```python
city_type_summary.plot(kind="pie",y="Total Fare ($)", autopct='%1.1f%%',
 startangle=140, shadow=False,explode=city_type_summary.explode,
                       colors=city_type_summary.colors,labels=city_type_summary.type,
                      wedgeprops = { 'linewidth' : 1 , 'edgecolor' : 'lightgrey'})

plt.axis('off')
plt.axis('equal')

plt.title('% of Total Fare ($) by City Type')

plt.show()
```


![png](output_16_0.png)

