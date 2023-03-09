# Microsoft Movie Studios Initial Pitch

# ***THIS IS A README PREVIEW, THIS IS NOT THE FINAL PRODUCT***

## Overview
This project reviews data from IMDB and TheNumbers.com to find the potential most equitable actions that can be taken to produce equitable content by looking at director history, reviewing the domestic, foreign, and global markets, and analyzing budgets against profit margins.

## Business Understanding
Microsoft is looking to produce original video content by opening a new studio. The studio is looking to start off strong with high-value titles from the beginning and justify the studio's initial cost to open.

## Data Understanding

IMDB is one of the biggest and detailed database on movie data. We specifically focused on movie_basics, movie_akas, directors, and persons table from IMDB. Budget data was taken from TheNumbers which provided data on production budget, domestic gross earnings, and worldwide gross earnings. 

We beleive that the "success" of a film can be measured by the revenue that it brings in and some big variables that affect the success of a film's directors, the film's market, and the film's budget.

### IMPORTS AND DATA


```python
import pandas as pd
import matplotlib.pyplot as plt
import matplotlib.ticker as mticker
import sqlite3
import numpy as np
import seaborn as sns
%matplotlib inline
```

### Set Plot and Display Defaults


```python
pd.options.display.float_format = '{:20,.2f}'.format
sns.set_theme()
sns.set_palette('colorblind')
```

TheNumbers has an extensive database of movies matched to their budgets and gross revenues.


```python
budgets_df = pd.read_csv('data/tn.movie_budgets.csv.gz')
display(budgets_df.head())
display(budgets_df.info())
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
      <th>id</th>
      <th>release_date</th>
      <th>movie</th>
      <th>production_budget</th>
      <th>domestic_gross</th>
      <th>worldwide_gross</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>Dec 18, 2009</td>
      <td>Avatar</td>
      <td>$425,000,000</td>
      <td>$760,507,625</td>
      <td>$2,776,345,279</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>May 20, 2011</td>
      <td>Pirates of the Caribbean: On Stranger Tides</td>
      <td>$410,600,000</td>
      <td>$241,063,875</td>
      <td>$1,045,663,875</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>Jun 7, 2019</td>
      <td>Dark Phoenix</td>
      <td>$350,000,000</td>
      <td>$42,762,350</td>
      <td>$149,762,350</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>May 1, 2015</td>
      <td>Avengers: Age of Ultron</td>
      <td>$330,600,000</td>
      <td>$459,005,868</td>
      <td>$1,403,013,963</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>Dec 15, 2017</td>
      <td>Star Wars Ep. VIII: The Last Jedi</td>
      <td>$317,000,000</td>
      <td>$620,181,382</td>
      <td>$1,316,721,747</td>
    </tr>
  </tbody>
</table>
</div>


    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 5782 entries, 0 to 5781
    Data columns (total 6 columns):
     #   Column             Non-Null Count  Dtype 
    ---  ------             --------------  ----- 
     0   id                 5782 non-null   int64 
     1   release_date       5782 non-null   object
     2   movie              5782 non-null   object
     3   production_budget  5782 non-null   object
     4   domestic_gross     5782 non-null   object
     5   worldwide_gross    5782 non-null   object
    dtypes: int64(1), object(5)
    memory usage: 271.2+ KB



    None


### IMDB's Database Schema
IMDB has one of the largest databases of movies available.

Data used here include basic movie information (movie_basics), alternative movie titles (movie_akas), and movie directors and their respective names (directors & persons).

![database schema flow chart](./images/db_schema.jpeg)


```python
# REQUIRES UNZIPPING data/im.db.zip AS im.db
conn = sqlite3.connect('data/im.db')
```


```python
recent_imdb_movies = pd.read_sql("""
SELECT * 
FROM movie_basics
WHERE CAST(start_year AS int) BETWEEN 2013 AND 2023
""", conn)
display(recent_imdb_movies.head())
display(recent_imdb_movies.info())
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
      <th>movie_id</th>
      <th>primary_title</th>
      <th>original_title</th>
      <th>start_year</th>
      <th>runtime_minutes</th>
      <th>genres</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>tt0063540</td>
      <td>Sunghursh</td>
      <td>Sunghursh</td>
      <td>2013</td>
      <td>175.00</td>
      <td>Action,Crime,Drama</td>
    </tr>
    <tr>
      <th>1</th>
      <td>tt0066787</td>
      <td>One Day Before the Rainy Season</td>
      <td>Ashad Ka Ek Din</td>
      <td>2019</td>
      <td>114.00</td>
      <td>Biography,Drama</td>
    </tr>
    <tr>
      <th>2</th>
      <td>tt0069049</td>
      <td>The Other Side of the Wind</td>
      <td>The Other Side of the Wind</td>
      <td>2018</td>
      <td>122.00</td>
      <td>Drama</td>
    </tr>
    <tr>
      <th>3</th>
      <td>tt0069204</td>
      <td>Sabse Bada Sukh</td>
      <td>Sabse Bada Sukh</td>
      <td>2018</td>
      <td>nan</td>
      <td>Comedy,Drama</td>
    </tr>
    <tr>
      <th>4</th>
      <td>tt0100275</td>
      <td>The Wandering Soap Opera</td>
      <td>La Telenovela Errante</td>
      <td>2017</td>
      <td>80.00</td>
      <td>Comedy,Drama,Fantasy</td>
    </tr>
  </tbody>
</table>
</div>


    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 107602 entries, 0 to 107601
    Data columns (total 6 columns):
     #   Column           Non-Null Count   Dtype  
    ---  ------           --------------   -----  
     0   movie_id         107602 non-null  object 
     1   primary_title    107602 non-null  object 
     2   original_title   107582 non-null  object 
     3   start_year       107602 non-null  int64  
     4   runtime_minutes  82307 non-null   float64
     5   genres           103491 non-null  object 
    dtypes: float64(1), int64(1), object(4)
    memory usage: 4.9+ MB



    None



```python
movie_akas = pd.read_sql("""
SELECT *
FROM movie_akas
""", conn)
display(movie_akas.head())
display(movie_akas.info())
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
      <th>movie_id</th>
      <th>ordering</th>
      <th>title</th>
      <th>region</th>
      <th>language</th>
      <th>types</th>
      <th>attributes</th>
      <th>is_original_title</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>tt0369610</td>
      <td>10</td>
      <td>Джурасик свят</td>
      <td>BG</td>
      <td>bg</td>
      <td>None</td>
      <td>None</td>
      <td>0.00</td>
    </tr>
    <tr>
      <th>1</th>
      <td>tt0369610</td>
      <td>11</td>
      <td>Jurashikku warudo</td>
      <td>JP</td>
      <td>None</td>
      <td>imdbDisplay</td>
      <td>None</td>
      <td>0.00</td>
    </tr>
    <tr>
      <th>2</th>
      <td>tt0369610</td>
      <td>12</td>
      <td>Jurassic World: O Mundo dos Dinossauros</td>
      <td>BR</td>
      <td>None</td>
      <td>imdbDisplay</td>
      <td>None</td>
      <td>0.00</td>
    </tr>
    <tr>
      <th>3</th>
      <td>tt0369610</td>
      <td>13</td>
      <td>O Mundo dos Dinossauros</td>
      <td>BR</td>
      <td>None</td>
      <td>None</td>
      <td>short title</td>
      <td>0.00</td>
    </tr>
    <tr>
      <th>4</th>
      <td>tt0369610</td>
      <td>14</td>
      <td>Jurassic World</td>
      <td>FR</td>
      <td>None</td>
      <td>imdbDisplay</td>
      <td>None</td>
      <td>0.00</td>
    </tr>
  </tbody>
</table>
</div>


    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 331703 entries, 0 to 331702
    Data columns (total 8 columns):
     #   Column             Non-Null Count   Dtype  
    ---  ------             --------------   -----  
     0   movie_id           331703 non-null  object 
     1   ordering           331703 non-null  int64  
     2   title              331703 non-null  object 
     3   region             278410 non-null  object 
     4   language           41715 non-null   object 
     5   types              168447 non-null  object 
     6   attributes         14925 non-null   object 
     7   is_original_title  331678 non-null  float64
    dtypes: float64(1), int64(1), object(6)
    memory usage: 20.2+ MB



    None


Since the directors table only has movie_id and person_id, we join the directors using their person_id to get their names


```python
imdb_directors = pd.read_sql("""
SELECT DISTINCT d.movie_id, p.person_id, p.primary_name 
FROM persons as p
INNER JOIN directors AS d
    ON d.person_id = p.person_id
""", conn)
```

## Data Preparation

### Matching TheNumbers data to IMDB
Use a list of all IMDB titles to match movie IDs from IMDB to movie titles from TheNumbers.


```python
movie_akas = pd.read_sql("""
SELECT DISTINCT movie_id, title FROM movie_akas
""",conn)
```


```python
movie_akas_list = list(movie_akas['title'])
movie_ids_list = list(movie_akas['movie_id'])
```

Since directors are a key part of the calculation, we need to associate movie_ids to movies in order to pair the two together through IMDB


```python
budgets_df['movie_id'] = [movie_ids_list[movie_akas_list.index( title )] if title in movie_akas_list 
                          else None for title in budgets_df['movie']]
budgets_df
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
      <th>id</th>
      <th>release_date</th>
      <th>movie</th>
      <th>production_budget</th>
      <th>domestic_gross</th>
      <th>worldwide_gross</th>
      <th>movie_id</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>Dec 18, 2009</td>
      <td>Avatar</td>
      <td>$425,000,000</td>
      <td>$760,507,625</td>
      <td>$2,776,345,279</td>
      <td>tt1775309</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>May 20, 2011</td>
      <td>Pirates of the Caribbean: On Stranger Tides</td>
      <td>$410,600,000</td>
      <td>$241,063,875</td>
      <td>$1,045,663,875</td>
      <td>tt1298650</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>Jun 7, 2019</td>
      <td>Dark Phoenix</td>
      <td>$350,000,000</td>
      <td>$42,762,350</td>
      <td>$149,762,350</td>
      <td>tt6565702</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>May 1, 2015</td>
      <td>Avengers: Age of Ultron</td>
      <td>$330,600,000</td>
      <td>$459,005,868</td>
      <td>$1,403,013,963</td>
      <td>tt2395427</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>Dec 15, 2017</td>
      <td>Star Wars Ep. VIII: The Last Jedi</td>
      <td>$317,000,000</td>
      <td>$620,181,382</td>
      <td>$1,316,721,747</td>
      <td>None</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>5777</th>
      <td>78</td>
      <td>Dec 31, 2018</td>
      <td>Red 11</td>
      <td>$7,000</td>
      <td>$0</td>
      <td>$0</td>
      <td>tt7837402</td>
    </tr>
    <tr>
      <th>5778</th>
      <td>79</td>
      <td>Apr 2, 1999</td>
      <td>Following</td>
      <td>$6,000</td>
      <td>$48,482</td>
      <td>$240,495</td>
      <td>None</td>
    </tr>
    <tr>
      <th>5779</th>
      <td>80</td>
      <td>Jul 13, 2005</td>
      <td>Return to the Land of Wonders</td>
      <td>$5,000</td>
      <td>$1,338</td>
      <td>$1,338</td>
      <td>None</td>
    </tr>
    <tr>
      <th>5780</th>
      <td>81</td>
      <td>Sep 29, 2015</td>
      <td>A Plague So Pleasant</td>
      <td>$1,400</td>
      <td>$0</td>
      <td>$0</td>
      <td>tt2107644</td>
    </tr>
    <tr>
      <th>5781</th>
      <td>82</td>
      <td>Aug 5, 2005</td>
      <td>My Date With Drew</td>
      <td>$1,100</td>
      <td>$181,041</td>
      <td>$181,041</td>
      <td>None</td>
    </tr>
  </tbody>
</table>
<p>5782 rows × 7 columns</p>
</div>



get rid of entries without a movie_id - director can't be found without this info

### Cleaning budgets_df


```python
budgets_df.dropna(subset=['movie_id'],inplace=True)
```


```python
budgets_df.drop_duplicates(subset=['movie_id'],inplace=True)
```

Dollar amounts need to be treated as numbers, so we have to remove dollar sign and commas from our data.


```python
budgets_df['production_budget'] = (budgets_df['production_budget'].str.replace('$', '')
                                   .str.replace(',', '').astype(int))
budgets_df['domestic_gross'] = budgets_df['domestic_gross'].str.replace('$', '').str.replace(',', '').astype(int)
budgets_df['worldwide_gross'] = budgets_df['worldwide_gross'].str.replace('$', '').str.replace(',', '').astype(int)
```

### Feature Engineering

We created foreign market analysis and margins to determine budget range for future movies.


```python
budgets_df['foreign_gross'] = budgets_df['worldwide_gross'] - budgets_df['domestic_gross']
budgets_df['domestic_margin'] = budgets_df['domestic_gross'] - budgets_df['production_budget']
budgets_df['worldwide_margin'] = budgets_df['worldwide_gross'] - budgets_df['production_budget']
budgets_df['foreign_margin'] = budgets_df['foreign_gross'] - budgets_df['production_budget']
```


```python
budgets_df.head()
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
      <th>id</th>
      <th>release_date</th>
      <th>movie</th>
      <th>production_budget</th>
      <th>domestic_gross</th>
      <th>worldwide_gross</th>
      <th>movie_id</th>
      <th>foreign_gross</th>
      <th>domestic_margin</th>
      <th>worldwide_margin</th>
      <th>foreign_margin</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>Dec 18, 2009</td>
      <td>Avatar</td>
      <td>425000000</td>
      <td>760507625</td>
      <td>2776345279</td>
      <td>tt1775309</td>
      <td>2015837654</td>
      <td>335507625</td>
      <td>2351345279</td>
      <td>1590837654</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>May 20, 2011</td>
      <td>Pirates of the Caribbean: On Stranger Tides</td>
      <td>410600000</td>
      <td>241063875</td>
      <td>1045663875</td>
      <td>tt1298650</td>
      <td>804600000</td>
      <td>-169536125</td>
      <td>635063875</td>
      <td>394000000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>Jun 7, 2019</td>
      <td>Dark Phoenix</td>
      <td>350000000</td>
      <td>42762350</td>
      <td>149762350</td>
      <td>tt6565702</td>
      <td>107000000</td>
      <td>-307237650</td>
      <td>-200237650</td>
      <td>-243000000</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>May 1, 2015</td>
      <td>Avengers: Age of Ultron</td>
      <td>330600000</td>
      <td>459005868</td>
      <td>1403013963</td>
      <td>tt2395427</td>
      <td>944008095</td>
      <td>128405868</td>
      <td>1072413963</td>
      <td>613408095</td>
    </tr>
    <tr>
      <th>6</th>
      <td>7</td>
      <td>Apr 27, 2018</td>
      <td>Avengers: Infinity War</td>
      <td>300000000</td>
      <td>678815482</td>
      <td>2048134200</td>
      <td>tt4154756</td>
      <td>1369318718</td>
      <td>378815482</td>
      <td>1748134200</td>
      <td>1069318718</td>
    </tr>
  </tbody>
</table>
</div>



### Merging budgets with recent IMDB movies
Bringing together more information from IMDB with budget data from TheNumbers


```python
recent_movies = budgets_df.merge(recent_imdb_movies, on="movie_id", how='inner')
recent_movies.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 1465 entries, 0 to 1464
    Data columns (total 16 columns):
     #   Column             Non-Null Count  Dtype  
    ---  ------             --------------  -----  
     0   id                 1465 non-null   int64  
     1   release_date       1465 non-null   object 
     2   movie              1465 non-null   object 
     3   production_budget  1465 non-null   int64  
     4   domestic_gross     1465 non-null   int64  
     5   worldwide_gross    1465 non-null   int64  
     6   movie_id           1465 non-null   object 
     7   foreign_gross      1465 non-null   int64  
     8   domestic_margin    1465 non-null   int64  
     9   worldwide_margin   1465 non-null   int64  
     10  foreign_margin     1465 non-null   int64  
     11  primary_title      1465 non-null   object 
     12  original_title     1465 non-null   object 
     13  start_year         1465 non-null   int64  
     14  runtime_minutes    1375 non-null   float64
     15  genres             1460 non-null   object 
    dtypes: float64(1), int64(9), object(6)
    memory usage: 194.6+ KB


### Merging together recent movies and directors
Bringing together information from the merged recent movies table and directors for our final tabel for director analysis


```python
movies_directors = recent_movies.merge(imdb_directors, on="movie_id", how='inner')
movies_directors.head()
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
      <th>id</th>
      <th>release_date</th>
      <th>movie</th>
      <th>production_budget</th>
      <th>domestic_gross</th>
      <th>worldwide_gross</th>
      <th>movie_id</th>
      <th>foreign_gross</th>
      <th>domestic_margin</th>
      <th>worldwide_margin</th>
      <th>foreign_margin</th>
      <th>primary_title</th>
      <th>original_title</th>
      <th>start_year</th>
      <th>runtime_minutes</th>
      <th>genres</th>
      <th>person_id</th>
      <th>primary_name</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>3</td>
      <td>Jun 7, 2019</td>
      <td>Dark Phoenix</td>
      <td>350000000</td>
      <td>42762350</td>
      <td>149762350</td>
      <td>tt6565702</td>
      <td>107000000</td>
      <td>-307237650</td>
      <td>-200237650</td>
      <td>-243000000</td>
      <td>Dark Phoenix</td>
      <td>Dark Phoenix</td>
      <td>2019</td>
      <td>113.00</td>
      <td>Action,Adventure,Sci-Fi</td>
      <td>nm1334526</td>
      <td>Simon Kinberg</td>
    </tr>
    <tr>
      <th>1</th>
      <td>4</td>
      <td>May 1, 2015</td>
      <td>Avengers: Age of Ultron</td>
      <td>330600000</td>
      <td>459005868</td>
      <td>1403013963</td>
      <td>tt2395427</td>
      <td>944008095</td>
      <td>128405868</td>
      <td>1072413963</td>
      <td>613408095</td>
      <td>Avengers: Age of Ultron</td>
      <td>Avengers: Age of Ultron</td>
      <td>2015</td>
      <td>141.00</td>
      <td>Action,Adventure,Sci-Fi</td>
      <td>nm0923736</td>
      <td>Joss Whedon</td>
    </tr>
    <tr>
      <th>2</th>
      <td>7</td>
      <td>Apr 27, 2018</td>
      <td>Avengers: Infinity War</td>
      <td>300000000</td>
      <td>678815482</td>
      <td>2048134200</td>
      <td>tt4154756</td>
      <td>1369318718</td>
      <td>378815482</td>
      <td>1748134200</td>
      <td>1069318718</td>
      <td>Avengers: Infinity War</td>
      <td>Avengers: Infinity War</td>
      <td>2018</td>
      <td>149.00</td>
      <td>Action,Adventure,Sci-Fi</td>
      <td>nm0751648</td>
      <td>Joe Russo</td>
    </tr>
    <tr>
      <th>3</th>
      <td>7</td>
      <td>Apr 27, 2018</td>
      <td>Avengers: Infinity War</td>
      <td>300000000</td>
      <td>678815482</td>
      <td>2048134200</td>
      <td>tt4154756</td>
      <td>1369318718</td>
      <td>378815482</td>
      <td>1748134200</td>
      <td>1069318718</td>
      <td>Avengers: Infinity War</td>
      <td>Avengers: Infinity War</td>
      <td>2018</td>
      <td>149.00</td>
      <td>Action,Adventure,Sci-Fi</td>
      <td>nm0751577</td>
      <td>Anthony Russo</td>
    </tr>
    <tr>
      <th>4</th>
      <td>9</td>
      <td>Nov 17, 2017</td>
      <td>Justice League</td>
      <td>300000000</td>
      <td>229024295</td>
      <td>655945209</td>
      <td>tt0974015</td>
      <td>426920914</td>
      <td>-70975705</td>
      <td>355945209</td>
      <td>126920914</td>
      <td>Justice League</td>
      <td>Justice League</td>
      <td>2017</td>
      <td>120.00</td>
      <td>Action,Adventure,Fantasy</td>
      <td>nm0811583</td>
      <td>Zack Snyder</td>
    </tr>
  </tbody>
</table>
</div>



# Data Analysis

## Getting the the top 15 directors sorted by the median of their worldwide margin 
Here, we wanted to look at the median profits generated from directors' recent movies to reduce the influence of outliers.

Some directors have experience in creating highly profitable films. In recent years, Colin Trevorrow (the director of 2 of the recent _Jurassic World_ films) has been able to generate a median of \\$1.4 billion in profit in his films. The next-highest slot in this list was nearly \\$350 million dollars behind him.


```python
director_wwmargin_median = movies_directors.groupby(
    ['primary_name', 'person_id'], as_index=False
)['worldwide_margin'].median().sort_values(by='worldwide_margin', ascending=False)[:15]
director_wwmargin_median.head()
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
      <th>primary_name</th>
      <th>person_id</th>
      <th>worldwide_margin</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>254</th>
      <td>Colin Trevorrow</td>
      <td>nm1119880</td>
      <td>1,433,854,864.00</td>
    </tr>
    <tr>
      <th>235</th>
      <td>Christophe Gans</td>
      <td>nm0304521</td>
      <td>1,099,199,706.00</td>
    </tr>
    <tr>
      <th>668</th>
      <td>Joss Whedon</td>
      <td>nm0923736</td>
      <td>1,072,413,963.00</td>
    </tr>
    <tr>
      <th>737</th>
      <td>Kyle Balda</td>
      <td>nm0049633</td>
      <td>1,023,031,961.50</td>
    </tr>
    <tr>
      <th>999</th>
      <td>Pierre Coffin</td>
      <td>nm1853544</td>
      <td>959,727,750.00</td>
    </tr>
  </tbody>
</table>
</div>




```python
fig, ax = plt.subplots(figsize=(16, 7))

x = director_wwmargin_median['primary_name']
y = director_wwmargin_median['worldwide_margin']

_y_ticks = [(value * 10**8) for value in range(2,14+1,2)]
ax.set(
    title = "Top 15 Worldwide Margin Directors", 
    xlabel = "Director Names",
    xticklabels = [f'{director[0]}. {director.split(" ")[-1]}' for director in x.tolist()],
    ylabel = "Median Worldwide Margin",
    yticks=_y_ticks,
    yticklabels = [f'${int(x/1000000):,}M' for x in _y_ticks]
)

ax.bar(x, y, align='edge')
plt.xticks(rotation=50)
plt.rc('font', size = 25)
"";
```

    <ipython-input-18-23975babd822>:7: UserWarning: FixedFormatter should only be used together with FixedLocator
      ax.set(



    
![Graph of directors sorted by median global profit margin](images/director_v_margin.png)
    


## Market Discovery:
Here, we sought to discover if a foreign market or a domestic market would be a better area to focus. We did this by gathering the median gross sales of all regions and compared them to one another, with an additional focus on the top 50 highest grossing movies worldwide.


```python
market_focus_df = budgets_df.loc[
    (budgets_df['foreign_gross'] > 0) & (budgets_df['domestic_gross'] > 0),
    ['worldwide_gross','domestic_gross','foreign_gross']
]
market_focus = {
    'worldwide': {
        'median': market_focus_df['worldwide_gross'].median(),
        'top_median': market_focus_df['worldwide_gross'][:50].median()
    },
    'domestic': {
        'median': market_focus_df['domestic_gross'].median(),
        'top_median': market_focus_df['domestic_gross'][:50].median()
    },
    'foreign': {
        'median': market_focus_df['foreign_gross'].median(),
        'top_median': market_focus_df['foreign_gross'][:50].median()
    }
}
```


```python
fig, ax = plt.subplots(figsize = (12,14))
sns.set_palette('colorblind')

x = ['All Films','Top 50\nGrossing Films']
_y_ticks = [(value * 10**8) for value in range(0,10+1,2)]

ax.bar(
    x=x,
    height=list(market_focus['worldwide'].values()),
    width=-0.9,
    align='center'
)
ax.bar(
    x=x,
    height=list(market_focus['domestic'].values()),
    width=-0.4,
    align='edge'
)
ax.bar(
    x=x,
    height=list(market_focus['foreign'].values()),
    width=0.4,
    align='edge'
)

ax.set(
    title="Gross Sales Compared across Domestic,\nForeign, and Global Markets\n(in Millions USD)",
    xlabel="Market Measured",
    ylabel="Gross Sales (Millions, USD)",
    yticks=_y_ticks,
    yticklabels = [f'${int(x/1000000):,}M' for x in _y_ticks]
)
ax.legend(['Global','Domestic','Foreign'])
plt.rc('font', size = 25)
'';
```


    
![Graph depicting gross sales in global, domestic, and foreign markets](images/gross_sales_by_market.png)
    


We can see here that the domestic market seem to be slightly better than the foreign market across the entire dataset, but for the movies in the highest-grossing subset, the foreign market seems to bring in almost double the value.

## Budget vs Worldwide Profit Margin per Movie


```python
fig, ax = plt.subplots(figsize=(15, 10))

x = movies_directors['production_budget']
y = movies_directors['worldwide_margin']

ax.scatter(
    x=x,
    y=y,
    c=np.sign(y),
    cmap=plt.cm.coolwarm.reversed()
)

_x_ticks = [value * 10**6 for value in range(50,350+1,50)]
_y_ticks = [value * 10**6 for value in range(-250,1750+1,250)]
ax.set(
    title="Return on Investment Based on Production Budgets\n(in Millions USD)",
    xlabel="Budget",
    ylabel="Worldwide Profit Margin",
    xticks=_x_ticks,
    xticklabels = [f'${int(value/1000000):,}M' for value in _x_ticks],
    yticks=_y_ticks,
    yticklabels = [f'${int(value/1000000):,}M' for value in _y_ticks],
)

z = np.polyfit(x, y, 2)
p = np.poly1d(z)

ax.plot(x,p(x),"r--")

plt.xticks(fontsize=14, rotation=0)
plt.yticks(fontsize=14, rotation=0)
plt.rc('font', size = 25)
'';
```


    
![png](README_TEST_files/README_TEST_44_0.png)
    


The scatter plot shows a slight positive correlation for production budget and worldwide profit margin (return on investment) which supports that the more investment made the more return received. In this range, there doesn't seem to be a point of diminishing returns.

## Results

## Conclusions

## Next Step

- name matching
- no new avatar
- we looked at last 10 years so it disregards the first avatar and other big movies so might be worth to go back more
- get or generate a more complete dataset bc lots of mismatches and missing data such as worldwide gross listed as zero and movies with no ids

## Repository structure

## Thank you
