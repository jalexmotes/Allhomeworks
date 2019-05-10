
# Import libraries


```python
%matplotlib inline
from matplotlib import style
# style.use('fivethirtyeight')
import matplotlib.pyplot as plt
import seaborn as sns
```


```python
import numpy as np
import pandas as pd
```


```python
import datetime as dt
```

# Reflect Tables into SQLAlchemy ORM


```python
# Python SQL toolkit and Object Relational Mapper

import sqlalchemy
from sqlalchemy.ext.automap import automap_base
from sqlalchemy.orm import Session
from sqlalchemy import create_engine, inspect, func, Date, desc
```


```python
# Create engine using the `hawaii.sqlite` database file

engine = create_engine("sqlite:///../Resources/hawaii.sqlite")
```


```python
# reflect an existing database into a new model

Base = automap_base()

# reflect the tables

Base.prepare(engine, reflect=True)
```


```python
# We can view all of the classes that automap found

Base.classes.keys()
```




    ['measurement', 'station']




```python
# Save object references to each table

Measurement = Base.classes.measurement
Station = Base.classes.station

```


```python
# Create our session (link) from Python to the DB

session = Session(engine)
```

# Climate Analysis


```python
# Inspect database table to get columns and data types

inspector = inspect(engine)

inspector.get_table_names()

# Get a list of column names and types

columns = inspector.get_columns('measurement')

for c in columns:
    print(c['name'], c["type"])

    # columns
```

    id INTEGER
    station TEXT
    date TEXT
    prcp FLOAT
    tobs FLOAT
    


```python
# Display the row's columns and data in dictionary format

first_row_Measurement = session.query(Measurement).first()
first_row_Measurement.__dict__

```




    {'_sa_instance_state': <sqlalchemy.orm.state.InstanceState at 0x1becdf92438>,
     'tobs': 65.0,
     'date': '2010-01-01',
     'id': 1,
     'prcp': 0.08,
     'station': 'USC00519397'}




```python
# Find last date in database

Last_Date = session.query(func.max(Measurement.date)).all()

Last_Date

```




    [('2017-08-23')]




```python
# Last Date of observation all the way back through 12 months

Last_Year_Observation = dt.date(2017, 8, 23) - dt.timedelta(days=7*52)

Last_Year_Observation
```




    datetime.date(2016, 8, 24)




```python
# Design a query to retrieve the last 12 months of precipitation data and plot the results

results = session.query(Measurement.date,Measurement.prcp).filter(Measurement.date > Last_Year_Observation).all()

```


```python
# Save the query results as a Pandas DataFrame and set the index to the date column

df = pd.DataFrame(results)

df.columns =  results[0].keys()

df.set_index('date',inplace = True)

# Sort the dataframe by date

df_sorted = df.sort_values('date')

df_sorted.head(10)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>prcp</th>
    </tr>
    <tr>
      <th>date</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2016-08-25</th>
      <td>0.08</td>
    </tr>
    <tr>
      <th>2016-08-25</th>
      <td>0.00</td>
    </tr>
    <tr>
      <th>2016-08-25</th>
      <td>0.06</td>
    </tr>
    <tr>
      <th>2016-08-25</th>
      <td>0.00</td>
    </tr>
    <tr>
      <th>2016-08-25</th>
      <td>0.08</td>
    </tr>
    <tr>
      <th>2016-08-25</th>
      <td>0.11</td>
    </tr>
    <tr>
      <th>2016-08-25</th>
      <td>0.21</td>
    </tr>
    <tr>
      <th>2016-08-26</th>
      <td>0.00</td>
    </tr>
    <tr>
      <th>2016-08-26</th>
      <td>0.00</td>
    </tr>
    <tr>
      <th>2016-08-26</th>
      <td>0.01</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Use Pandas Plotting with Matplotlib to plot the data

sns.set()

plot1 = df_sorted.plot(figsize = (10, 5))

fig = plot1.get_figure()

plt.title('Precipitation in Honolulu')

plt.xlabel('Date')

plt.ylabel('Precipitation')

plt.legend(["Precipitation"],loc="best")

plt.xticks(rotation=45)

plt.tight_layout()

fig.savefig("../Images/Precipitation_Analysis.png")

plt.show()
```


![png](output_18_0.png)



```python
# Use Pandas to calcualte the summary statistics for the precipitation data

Summary_Stats_DF = df_sorted.describe()

Summary_Stats_DF.rename(columns = {'prcp' : 'Precipitation'})
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Precipitation</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>2009.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>0.172344</td>
    </tr>
    <tr>
      <th>std</th>
      <td>0.452818</td>
    </tr>
    <tr>
      <th>min</th>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>0.020000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>0.130000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>6.700000</td>
    </tr>
  </tbody>
</table>
</div>



# Station Analysis


```python
# Display the row's columns and data in dictionary format

first_row_Station = session.query(Station).first()
first_row_Station.__dict__
```




    {'_sa_instance_state': <sqlalchemy.orm.state.InstanceState at 0x1bed02d48d0>,
     'station': 'USC00519397',
     'longitude': -157.8168,
     'name': 'WAIKIKI 717.2, HI US',
     'id': 1,
     'elevation': 3.0,
     'latitude': 21.2716}




```python
# Setting the dataframe for Station 

results_station = session.query(Station.latitude,Station.longitude,Station.id,Station.elevation,Station.station,Station.name).all()

df_stations = pd.DataFrame(results_station)

df_stations.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>latitude</th>
      <th>longitude</th>
      <th>id</th>
      <th>elevation</th>
      <th>station</th>
      <th>name</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>21.2716</td>
      <td>-157.8168</td>
      <td>1</td>
      <td>3.0</td>
      <td>USC00519397</td>
      <td>WAIKIKI 717.2, HI US</td>
    </tr>
    <tr>
      <th>1</th>
      <td>21.4234</td>
      <td>-157.8015</td>
      <td>2</td>
      <td>14.6</td>
      <td>USC00513117</td>
      <td>KANEOHE 838.1, HI US</td>
    </tr>
    <tr>
      <th>2</th>
      <td>21.5213</td>
      <td>-157.8374</td>
      <td>3</td>
      <td>7.0</td>
      <td>USC00514830</td>
      <td>KUALOA RANCH HEADQUARTERS 886.9, HI US</td>
    </tr>
    <tr>
      <th>3</th>
      <td>21.3934</td>
      <td>-157.9751</td>
      <td>4</td>
      <td>11.9</td>
      <td>USC00517948</td>
      <td>PEARL CITY, HI US</td>
    </tr>
    <tr>
      <th>4</th>
      <td>21.4992</td>
      <td>-158.0111</td>
      <td>5</td>
      <td>306.6</td>
      <td>USC00518838</td>
      <td>UPPER WAHIAWA 874.3, HI US</td>
    </tr>
  </tbody>
</table>
</div>




```python
# How many stations are available in this dataset - through SQL Alchemy

Number_of_Stations = session.query(Station.station).count()
 
print(f"Available Station(s): {Number_of_Stations}")
```

    Available Station(s): 9
    


```python
# What are the most active stations?

Active_Stations = session.query(Station.station,func.count(Measurement.tobs)).filter(Station.station == Measurement.station).\
                  group_by(Station.station).order_by(desc(func.count(Measurement.tobs))).all()

# List the stations and the counts in descending order.

print(f"{Active_Stations[0][0]} has {Active_Stations[0][1]} observations!")

Active_Stations

```

    USC00519281 has 2772 observations!
    




    [('USC00519281', 2772),
     ('USC00519397', 2724),
     ('USC00513117', 2709),
     ('USC00519523', 2669),
     ('USC00516128', 2612),
     ('USC00514830', 2202),
     ('USC00511918', 1979),
     ('USC00517948', 1372),
     ('USC00518838', 511)]




```python
# Using the station id from the previous query, calculate the lowest temperature recorded, 
# highest temperature recorded, and average temperature most active station?

Station_Name = session.query(Station.name).filter(Station.station == Active_Stations[0][0]).all() 

print(Station_Name)

Temp_Stats = session.query(func.min(Measurement.tobs),func.max(Measurement.tobs),func.avg(Measurement.tobs)).\
             filter(Station.station == Active_Stations[0][0]).all()

print(Temp_Stats)
```

    [('WAIHEE 837.5, HI US',)]
    [(53.0, 87.0, 73.09795396419437)]
    


```python
# Choose the station with the highest number of temperature observations.

Station_Name = session.query(Station.name).filter(Station.station == Active_Stations[0][0]).all() 

print(Station_Name)

```

    [('WAIHEE 837.5, HI US',)]
    


```python
# Query the last 12 months of temperature observation data for this station 

results_WAIHEE = session.query(Measurement.date,Measurement.tobs).filter(Measurement.date > Last_Year_Observation).\
                            filter(Station.station == Active_Stations[0][0]).all()

results_WAIHEE_df = pd.DataFrame(results_WAIHEE)

results_WAIHEE_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>date</th>
      <th>tobs</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2016-08-25</td>
      <td>80.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2016-08-26</td>
      <td>79.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2016-08-27</td>
      <td>77.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2016-08-28</td>
      <td>78.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2016-08-29</td>
      <td>78.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Plot the results as a histogram

sns.set()

plt.figure(figsize=(10,5))

plt.hist(results_WAIHEE_df['tobs'],bins=12,color='lightblue')

plt.xlabel('Temperature',weight='bold')

plt.ylabel('Frequency',weight='bold')

plt.title('Station Analysis',weight='bold')

plt.legend(["Temperature Observation"],loc="best")

plt.savefig("../Images/Temperature_Analysis.png")

plt.show()
```


![png](output_28_0.png)

