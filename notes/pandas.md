# Pandas

### Dataframes
* Basics
``` python
import numpy as np
import pandas as pd
df = pd.DataFrame(np.random.randn(15).reshape(3, 5), columns=list('abcde'))
df.shape
df.head()
df.info()
df.describe()
df_array = df['col_1'].astype('float64')
df['col_1'].dtype
```
* Value frequency counts by column
``` python
for name in list(df.columns):
    print(df[name].value_counts().to_frame())
```
* Sorting
``` python
df.sort_index(axis='rows', inplace=True) # row-wise by index value
df.sort_values('col_1', ascending=True, inplace=True) # sort row-wise by column
```
* Modifying
``` python
df.loc[df['col_1']==constraint, 'col_2'] = new_col_2 # logical index subsetting replacement
df.add(series, axis='index') # adds a series row-wise
df.drop('col_1', axis='columns', inplace=True) # drop column
df['col_1'][(df['col_2']==a) & (df['col_3']==b)] # multiple logical subsetting
df.reindex(new_index) # returns the subset of df with matching index values and NA values for unknown new indices
df.reset_index(drop=True) # resets the index to auto increment and moves the old index into the first column if drop is false
```
* Handling missing / duplicates
``` python
df.dropna(how=<'any', 'all'>, axis=<'rows, columns'>) # returns a dataframe where any row that has any null record is removed
df.dropna(thres=2) # only keeps rows with at least 2 non-null observations
df.fillna(0, inplace=True)
df.replace([0, 1], -1) # replace the given input values with the output value
df.isnull().sum().sum() # total null records
df.isnull().sum(axis='rows') # null record counts by column
df['col_name'].unique() # returns unique column values
df.drop_duplicates() # returns a de-duped dataframe
df.drop_duplicates(['col_1', 'col_2']) # removes duplicates in the given column key
df = df[(np.abs(df) > 1).any(axis='rows')] # returns columns where any record value satisfies the condition
```

### Loading
* Read and write csv
``` python
df = pd.read_csv('data.csv', header=<None|0>, index_col=<None|0>)
df = pd.read_csv('data.csv', names=['col_1', 'col_2'], index_col='col_1')
df.to_csv('data.csv', header=True, index=False)
```
* Read and write excel
``` python
xlsx = pd.ExcelFile('data.xlsx')
df = pd.read_excel(xlsx, 'Sheet 1')
writer = pd.ExcelWriter('data.xlsx')
df.to_excel(writer, 'Sheet 1')
writer.save()
```
* Reading from MySQL database (need to use 127.0.0.1 vs localhost)
``` python
sudo apt install python3-mysqldb && pip install mysqlclient
df = pd.read_sql('SELECT * FROM temp.table', 'mysql://USER:PASS@127.0.0.1:PORT')
```

### Plotting
* Line graph of the dataframe columns using the index as the x-axis
``` python
import matplotlib.pyplot as plt
df.plot()
plt.show()
```
* Barplots of dataframe on split plot
``` python
fig, axes = plt.subplots(2, 1)
df.plot.bar(ax=axes[0], stacked=True)
df.plot.barh(ax=axes[1])
plt.show()
```
* Histogram and density plots of the given dataframe column
``` python
df['col_1'].plot.hist(bins=50)
df['col_1'].plot.density()
plt.show()
```
*Sseaborn combination histogram and density plot
``` python
import seaborn as sns
series = pd.Series(np.random.randn(100))
series = series.append(pd.Series(5 + 2*np.random.randn(100)))
sns.distplot(series)
plt.show()
```
* Seaborn pair plots
``` python
sns.pairplot(data=pd.DataFrame({'col_1': np.arange(100),
                                'col_2': 2*np.arange(100) + 5*np.random.randn(100)}),
             diag_kind='kde')
plt.show()
```

### Functions
``` python
df['col_name'].count()
df['col_name'].min()
df['col_name'].max()
df['col_name'].nunique()
df['col_name'].unique()
df['col_name'].value_counts()
df['col_name'].describe()
df.index = df.index.map(lambda x: x.lower())
df['col'].map(lambda x: x.max() - x.min()) # column-wise
df.apply(lambda x: x.max() - x.min(), axis='rows') # row-wise
*df.applymap(lambda x: '{0:.2f}'.format(x)) #  element-wise
```

### Merging
* **combine_first** uses df_2 values first, then defaults to df_1 values
``` python
df_1.merge(df_2, left_index=True, right_on='col_name', how='inner')
pd.concat([series_1, series_2], axis='columns')
pd.concat([df_1, df_2], axis='rows', ignore_index=True)
df_1.combine_first(df_2)
```

### Pivoting
* Pivots dataframe from long format to wide format
``` python
raw_df.pivot(index='ID', columns='Metric_Name', values='Metric_Value').fillna(0)
```
* Pivots dataframe from wide format to long format
``` python
df.melt(id_vars=['key'], value_vars=['value_1', 'value_2'], var_name='category', value_name='metric')
```

### Aggregation
* Group-by column
``` python
df.groupby(['col_name_1'])['col_name_2'].mean()
df.groupby(['col_name_1'])['col_name_2'].mean().round(2).to_frame()
df.groupby(['col_name_1', 'col_name_2']).agg({'col_name_3': 'sum', 'col_name_4': ['min', 'max', 'count']})
```
* Dictionary with the unique col_name as key
``` python
df.groupby(['col_name']).groups
```
* Crosstabulation of categorical variable frequencies
``` python
df = pd.DataFrame({'gender': np.random.choice(['male', 'female'], 100),
                   'pet': np.random.choice(['cat', 'dog'], 100)})
print(df.head())
pd.crosstab(df['gender'], df['pet'], margins=True)
```

### Misc
* Export the dataframe as a json array of row objects
``` python
df.to_json('FILENAME.json', orient='records')
```
* Specify how many rows of a dataframe are shown when it's printed
``` python
pd.options.display.max_rows = 10
```
* Iterate over rows in a dataframe
``` python
for index, row in df.iterrows():
  pass
for index_row in [list(l) for l in list(df.itertuples())]: # preserve data types
  pass
```
* Correlation of columns
``` python
df['col_1'].corr(df['col_2'])
df.corr() # correlation of columns with themselves
```
* Iterative chunker
``` python
chunker = pd.read_csv('data.csv', chunksize=1000)
frequencies = pd.Series()
for chunk in chunker:
    frequencies = frequencies.add(chunk['col_1'].value_counts(), fill_value=0)
frequencies = frequencies.sort_values(ascending=False)
```
* Print the dataframe to the console with certain formatting:
``` python
import sys
df.to_csv(sys.stdout, index=False, header=True, sep=' ')
```
* Scrape html and convert tables into dataframes
``` python
dfs = pd.read_html('https://www.w3schools.com/tags/tag_table.asp')
print(len(dfs))
dfs[1]
```
* Bins the given values with the bin_labels according to the boundaries with left closed
``` python
pd.cut(values, bin_boundaries, labels=bin_labels, right=False)
pd.qcut(df['col'], bin_count) # quartile binning
```
* Samples the given dataframe with or without replacement
``` python
df.sample(n=sample_size, replace=True)
```
* Force cast column to datetime
``` python
df['datetime_col'] = pd.to_datetime(df['datetime_col'], format='%Y-%m-%d', errors='coerce')
```
