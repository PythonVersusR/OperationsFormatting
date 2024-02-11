<img src="https://i.imgur.com/6U6q5jQ.png"/>

# Formatting Dates in Python

It is very common to find dates (some combination of year, month, day of week and time) in data that is collected in real time (and other that organize event information.

Let's see a data frame that comes with dates from an API.


```python
#!pip install sodapy
```


```python
import pandas as pd
from sodapy import Socrata

client = Socrata("data.seattle.gov", None)

results = client.get("kzjm-xkqj", limit=2000)

# Convert to pandas DataFrame
calls911 = pd.DataFrame.from_records(results)
```

Let's check the data types:


```python
calls911.info()
```

Let's get rid of some columns:


```python
calls911=calls911.iloc[:,:7]
```

Let's check the column _datetime_:


```python
calls911.datetime.head()
```


```python
# verify data type
type(calls911.datetime[0])

```

The date and time information is not useful at this time, that is, the information it offers is of limited use, as it is just a string. 

Let's make it useful:


```python
calls911.datetime=pd.to_datetime(calls911.datetime)
calls911.info()
```


```python
calls911
```

Once you have this data type, you can retrieve important information:


```python
calls911['date']=calls911.datetime.dt.date
calls911['year']=calls911.datetime.dt.year
calls911['month']=calls911.datetime.dt.month_name()
calls911['weekday']=calls911.datetime.dt.day_name()
calls911['hour']=calls911.datetime.dt.hour
```


```python
calls911.head()
```

Let's create a new column with what we have. In this case, a boolean where you tell if it is night time (after 8 pm before 6 am):


```python
calls911['nightTime']=((calls911['hour']<=6) | (calls911['hour']>=20))
```

Let's save what we have:


```python
calls911
```

What about data that comes in Spanish?


```python
#!pip install bs4
```


```python
link="https://es.wikipedia.org/wiki/Pandemia_de_COVID-19"

import pandas as pd

covid=pd.read_html(link, flavor="bs4", attrs={"class":"wikitable sortable"})
```

Let me keep the second df:


```python
covidDF=covid[1].copy()
covidDF
```

Notices the presence of some non-English punctuation:


```python
covidDF.columns
```

Let's get rid of those:


```python
#!pip install unidecode
```


```python
import unidecode as ud

[ud.unidecode(c) for c in covidDF.columns]
```


```python
#or
import re

[re.sub('\\s','',ud.unidecode(c)) for c in covidDF.columns]
```


```python
#then
covidDF.columns=[re.sub('\\s','',ud.unidecode(c)) for c in covidDF.columns]
```

Let's  focus on the _Fechadelanalisis_:


```python
# use " a " to split:
covidDF.Fechadelanalisis.str.split(" a ",expand=True)
```


```python
# create the two columns

covidDF[["fecha1","fecha2"]]=covidDF.Fechadelanalisis.str.split(" a ",expand=True)
covidDF
```

Let's format on of those columns:


```python
covidDF.fecha1
```


```python
covidDF.loc[8,'fecha1']='1 de noviembre de 2020'
```


```python
# let's split again:

covidDF.fecha1.str.split(" de ",expand=True)
```

I could create three new columns:


```python
covidDF[['fecha1_dia','fecha1_mes','fecha1_anho']]=covidDF.fecha1.str.split(" de ",expand=True)
covidDF[['fecha1_dia','fecha1_mes','fecha1_anho']]
```

We should use the month number instead of name. Let's prepare a dict of changes:


```python
monthName=('enero','febrero','marzo','abril','mayo','junio','julio','agosto','septiembre','octubre','noviembre','diciembre')
changes={name:number for name,number in zip(monthName,range(1,len(monthName)+1))}
changes
```


```python
covidDF.fecha1_mes.replace(changes,inplace=True)
```

Now we have:


```python
covidDF[['fecha1_dia','fecha1_mes','fecha1_anho']]
```

We will use those columns to create a date:


```python
pd.to_datetime(dict(year=covidDF.fecha1_anho, month=covidDF.fecha1_mes, day=covidDF.fecha1_dia))
```


```python

```
