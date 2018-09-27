Welcome to the first posting! I put in in my head to enact a 'one-analysis-per-day' rule. This is the first result. It's been a long day, but it is what it is.
I ventured over to [Data.gov|'data.gov']
```python
# Source: https://catalog.data.gov/dataset?q=recall&groups=safety3175#topic=safety_navigation
import pandas as pd
import matplotlib.pyplot as plt
from datetime import datetime as dt

%matplotlib inline

def get_data(myvar):
    pth = "/Users/ari/ds/gitblog/FSIS Recall Info/r{}.csv"
    df = []
    for i in myvar:
        fullpath = pth.format(i)
        df.append(pd.read_csv(fullpath, encoding = "ISO-8859-1"))
    return pd.concat(df)

yrs = list(range(2005,2015,1))
rdf = get_data(yrs)
```

Text

```python
rdf[pd.to_numeric(rdf['rPounds'], errors='coerce').notnull()].reset_index()
rdf['rDt'] = pd.to_datetime(rdf['rDt'],format='%b %d %Y')
rdf.dropna();
```

Text

```python
rdf['rYr'] = pd.DatetimeIndex(rdf['rDt']).year
rdf['rMo'] = pd.DatetimeIndex(rdf['rDt']).month
```

Text

```python
sDt1 = rdf.rYr.value_counts().sort_index()
sDt1.plot();
```

Text

```python
sDt2 = rdf.rMo.value_counts().sort_index()
sDt2.plot.bar(color='skyblue');
```

Text
