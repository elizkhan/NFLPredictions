# NFL Draft Predictions: Part 1 Web Scraping and Data Cleansing 
##### By Elizabeth Khan

![png](web.png)

## Goal: To predict which round a Quarterback, Wide Reciever, or Running Back will be drafted to the NFL  based on the player's performance throughout the regular season. 


### In this notebook I will be covering the web scraping and the data cleansing process for this dataset. If you wish to learn more about the feature engineering and modeling please proceed to the other Jupyter Notebook located here.

### Step 1: Use the Beautiful Soup python package to scrape player statistics for Passing, Receiving, and Rushing from the sports-reference.com website

The function below imports the beautiful soup library and searches for the table object within the html code parsed using the   ```soup = BeautifulSoup(response.content,'html.parser')``` command  the library, it then iterates through the table objects in order and assigns the values to the correct column. The end results of this function is a DataFrame that can be used for analysis. Note: sports-reference also has an API available to query player statistics but you need to have a player-id in order to do so. For simplicity and ease of obtaining the data, we will use the above approach.


```python
def get_stats(urls, cols):
    #Import Libraries
    import pandas as pd
    from requests import get
    from bs4 import BeautifulSoup
    response = get(url)
#     print(response.status_code)
    soup = BeautifulSoup(response.content,'html.parser')
    ncaa = soup.find('tbody')
    stats = ncaa.find_all('td')
    metrics = {}
    for item,col in zip(range(0,len(cols)),cols):
        metrics[col] = [stats[stat].get_text() for stat in range(item,len(stats),len(cols))]
        if col==cols[0]:
            df = pd.DataFrame({col:metrics[col]})
        elif col !=cols[-1]: 
            df[col] = metrics[col]
        else:
            df[col] = metrics[col]
            return df
```

#### Define the urls that we need to pull data from and the columns for each dataframe. For this analysis we will be using datasets from 2017 to 2019


```python
# Column headers for the DataFrames
pass_stats = ['Player','School', 'Conf', 'G','Cmp','Att','Pct','P_Yds','Y/A','AY/A','P_TD','Int','Rate','R_Att','R_Yds','R_Avg','R_TD']
rushing_stats = ['Player','School','Conf','G','Rush_Att','Rush_Yds','Rush_Avg','Rush_TD','Rec','Rec_Yds','Rec_Avg','Rec_TD','Scrim_Plays','Scrim_Yds','Scrim_Avg','Scrim_TD']
receiving_stats = ['Player','School','Conf','G','Rec','Rec_Yds','Rec_Avg','Rec_TD','Rush_Att','Rush_Yds','Rush_Avg','Rush_TD','Scrim_Plays','Scrim_Yds','Scrim_Avg','Scrim_TD']
draft_results = ['Pick','NFLTeam','Player','Pos','Age','To','AP1','PB','St','CarAV','DrAV','G','Cmp','Att','Yds','TD','Int','Att','Yds','TD','Rec','Yds','TD','Solo','Int','Sk','College/Univ','College Stats']
combine_stats = ['Year', 'Player', 'Pos', 'Age', 'AV', 'School', 'College', 'Height','Wt', '40YD', 'Vertical', 'BenchReps', 'Broad Jump', '3Cone', 'Shuttle','Drafted (tm/rnd/yr)']
```

### This code is iterating through the list of years and catgories to create the urls where the data is being pulled from.


```python
# Define Urls
season = ['2017','2018','2019']
category = ['PASSING','RUSHING','RECEIVING']

a = [category,season]

# Use cross product multiplication to create a list of every combination of year and category
import itertools
a = list(itertools.product(*a))

passing_url = []
rushing_url = []
receiving_url = []

for x in range(len(a)):
    if a[x][0] == 'PASSING':
        passing_url.append('https://www.sports-reference.com/cfb/years/'+a[x][1]+'-passing.html')
    elif a[x][0] == 'RUSHING':
         rushing_url.append('https://www.sports-reference.com/cfb/years/'+a[x][1]+'-rushing.html')
    elif a[x][0] == 'RECEIVING':
         receiving_url.append('https://www.sports-reference.com/cfb/years/'+a[x][1]+'-receiving.html')
            
draft_url = ['https://www.pro-football-reference.com/years/'+str(year)+'/draft.htm' for year in range(2018,2021)] 

combine_url = "https://www.pro-football-reference.com/play-index/nfl-combine-results.cgi?request=1&year_min=2018&year_max=2020&pos%5B%5D=QB&pos%5B%5D=WR&pos%5B%5D=RB&show=all&order_by=year_id"

```

###  Finally the dataframes are created and csv files are saved to the default directory as we do not want to continually run the beutiful soup scraping every time this code is run.


```python
# Create Passing DataFrame
for url, year in zip(passing_url, range(2009,2020)):
    if url == passing_url[0]:
        passing = get_stats(url, pass_stats)
        passing['Year'] = year
    else: 
        df = get_stats(url, pass_stats)
        df['Year'] = year
        passing = passing.append(df)
# print(passing.head())
print('Passing Dataframe Created with '+str(len(passing))+' rows')
passing.to_csv('passing_2017_2019.csv',index=None)

# Create Rushing DataFrame
for url, year in zip(rushing_url, range(2009,2020)):
    if url == rushing_url[0]:
        rushing= get_stats(url, rushing_stats)
        rushing['Year'] = year
    else: 
        df = get_stats(url, rushing_stats)
        df['Year'] = year
        rushing = rushing.append(df)
print('Rushing Dataframe Created with '+str(len(rushing))+' rows')
rushing.to_csv('rushing_2017_2019.csv',index=None)


# Create Receiving DataFrame
for url, year in zip(receiving_url, range(2009,2020)):
    if url == receiving_url[0]:
        receiving= get_stats(url, receiving_stats)
        receiving['Year'] = year
    else: 
        df = get_stats(url, receiving_stats)
        df['Year'] = year
        receiving = receiving.append(df)
print('Receiving Dataframe Created with '+str(len(receiving))+' rows')
receiving.to_csv('receiving_2017_2019.csv',index=None)

# Create Draft Results DataFrame
for url, year in zip(draft_url, range(2010,2021)):
    if url == draft_url[0]:
        draft= get_stats(url, draft_results)
        draft['Year'] = year
    else: 
        df = get_stats(url, draft_results)
        df['Year'] = year
        draft = draft.append(df)
print('Draft Dataframe Created with '+str(len(draft))+' rows')
draft.to_csv('draft_2018_2020.csv',index=None)


url = combine_url
combine = get_stats(url, combine_stats)
combine.to_csv('combine_2018_2020.csv')
```

    Passing Dataframe Created with 307 rows
    Rushing Dataframe Created with 853 rows
    Receiving Dataframe Created with 1358 rows
    Draft Dataframe Created with 765 rows
    

### Step 2: Clean up data by removing special characters and leading spaces from Player names, identify Player names that are similar through fuzzy string matching, remove any unmatched player names, and finally replace missing values with 0

#### The data overall is cleaned up for the most part. However when we combine dataframes since there are duplicative columns, columns names, and missing values (i.e. a quarterback probably does not have any receiving stats, so we will want to replace these missing values with 0). Also we  see variations of the player names so the clean up effort will need to account for this as well.


```python
# Import Libraries
import pandas as pd
import numpy as np

# List of Special Characters tp remove
spec_chars = ["!",'"',"#","%","&","'","(",")",
              "*","+",",","-",".","/",":",";","<",
              "=",">","?","@","[","\\","]","^","_",
              "`","{","|","}","~","–","/n"]

# Read scraped file that was saved to local directory
draft = pd.read_csv('draft_2018_2020.csv')
draft = draft[['Pick', 'NFLTeam','Pos', 'Player','Year']]
# remove leading and trailing spaces
draft['Player'] = draft['Player'].str.strip()
combine['Player'] = combine['Player'].str.strip()

passing = pd.read_csv('passing_2017_2019.csv')
# remove special character
passing['Player'] = passing['Player'].str.replace('*', '')
# Drop unecessary columns
passing.drop(columns=['School', 'Conf', 'G'],inplace=True)
passing = passing.add_prefix('qb_')
passing.rename(columns={'qb_Player':'Player','qb_Year':'Year'},inplace=True)


rushing = pd.read_csv('rushing_2017_2019.csv')
rushing['Player'] = rushing['Player'].str.replace('*', '')
rushing['Player'] = rushing['Player'].str.strip()
rb_cols = ['Rush_Att', 'Rush_Yds', 'Rush_Avg','Rush_TD', 'Rec', 'Rec_Yds', 'Rec_Avg', 'Rec_TD', 'Scrim_Plays', 'Scrim_Yds', 'Scrim_Avg', 'Scrim_TD']
# Drop unecessary columns
rushing.drop(columns=['School', 'Conf', 'G'],inplace=True)
# Update columns to include position in prefix
rushing = rushing.add_prefix('rb_')
rushing.rename(columns={'rb_Player':'Player','rb_Year':'Year'},inplace=True)

receiving = pd.read_csv('receiving_2017_2019.csv')
receiving['Player'] = receiving['Player'].str.replace('*', '')
wr_cols = ['Rec', 'Rec_Yds', 'Rec_Avg', 'Rec_TD', 'Rush_Att', 'Rush_Yds', 'Rush_Avg', 'Rush_TD', 'Scrim_Plays', 'Scrim_Yds', 'Scrim_Avg', 'Scrim_TD']
# Drop unecessary columns
receiving.drop(columns=['School', 'Conf', 'G'],inplace=True)

# Update columns to include position in prefix
receiving = receiving.add_prefix('wr_')
receiving.rename(columns={'wr_Player':'Player','wr_Year':'Year'},inplace=True)

draft['draft_year'] = draft.Year
draft['Year'] = draft.draft_year -1

combine['draft_year'] = combine.Year.astype('int')
combine['Year'] = combine.draft_year -1

combine = combine[['Year','draft_year','Player','Pos','School', 'Height', 'Wt', '40YD', 'Vertical', 'BenchReps', 'Broad Jump', '3Cone', 'Shuttle']]

# As an additional sanity check, let's go ahead and remove all possible special characters
# in order to ensure that the PLayer name will work for joins

for char in spec_chars:
    combine['Player'] = combine['Player'].str.replace(char, '')
    draft['Player'] = draft['Player'].str.replace(char, '')
    passing['Player'] = passing['Player'].str.replace(char, '')
    rushing['Player'] = rushing['Player'].str.replace(char, '')
    receiving['Player'] = receiving['Player'].str.replace(char, '')
    
draft = pd.merge(draft, combine, on=['Player','Pos'],how='left')
draft.rename(columns = {'Year_x':'Year','draft_year_x':'draft_year'},inplace=True)
draft.drop(columns = ['Year_y','draft_year_y'],inplace=True)
draft.columns
```




    Index(['Pick', 'NFLTeam', 'Pos', 'Player', 'Year', 'draft_year', 'School',
           'Height', 'Wt', '40YD', 'Vertical', 'BenchReps', 'Broad Jump', '3Cone',
           'Shuttle'],
          dtype='object')




```python
# combine all datasets together
complete = pd.merge(draft, passing, on=['Player','Year'], how='left')
complete = pd.merge(complete, rushing, on=['Player','Year'], how='left')
complete = pd.merge(complete, receiving, on=['Player','Year'], how='left')

complete = complete[complete['Pos'].isin(['QB','WR','RB'])]

complete.head()
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
      <th>Pick</th>
      <th>NFLTeam</th>
      <th>Pos</th>
      <th>Player</th>
      <th>Year</th>
      <th>draft_year</th>
      <th>School</th>
      <th>Height</th>
      <th>Wt</th>
      <th>40YD</th>
      <th>...</th>
      <th>wr_Rec_Avg</th>
      <th>wr_Rec_TD</th>
      <th>wr_Rush_Att</th>
      <th>wr_Rush_Yds</th>
      <th>wr_Rush_Avg</th>
      <th>wr_Rush_TD</th>
      <th>wr_Scrim_Plays</th>
      <th>wr_Scrim_Yds</th>
      <th>wr_Scrim_Avg</th>
      <th>wr_Scrim_TD</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>CLE</td>
      <td>QB</td>
      <td>Baker Mayfield</td>
      <td>2009</td>
      <td>2010</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>NYG</td>
      <td>RB</td>
      <td>Saquon Barkley</td>
      <td>2009</td>
      <td>2010</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>11.7</td>
      <td>3.0</td>
      <td>217.0</td>
      <td>1271.0</td>
      <td>5.9</td>
      <td>18.0</td>
      <td>271.0</td>
      <td>1903.0</td>
      <td>7.0</td>
      <td>21.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>NYJ</td>
      <td>QB</td>
      <td>Sam Darnold</td>
      <td>2009</td>
      <td>2010</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>6</th>
      <td>7</td>
      <td>BUF</td>
      <td>QB</td>
      <td>Josh Allen</td>
      <td>2009</td>
      <td>2010</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>9</th>
      <td>10</td>
      <td>ARI</td>
      <td>QB</td>
      <td>Josh Rosen</td>
      <td>2009</td>
      <td>2010</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 52 columns</p>
</div>



### Let's take a closer look at the dataframe by looking at the summary statistics to see if there are any issues. 
It looks like there is any issue as we have counts of 198 for Pick and the rest qb, rb, and wr stats are much lower. This could be a results of players positions but we will need to investigate this further. Outliers here could be indicative of extremely talented players, so we won't remove any values based on this for no. For example the 5,671 passing yards belong to Joe Burrow who set NCAA records in 2019.


```python
# Summary Statistics of player stats
complete.describe()
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
      <th>Pick</th>
      <th>Year</th>
      <th>draft_year</th>
      <th>qb_Cmp</th>
      <th>qb_Att</th>
      <th>qb_Pct</th>
      <th>qb_P_Yds</th>
      <th>qb_Y/A</th>
      <th>qb_AY/A</th>
      <th>qb_P_TD</th>
      <th>...</th>
      <th>wr_Rec_Avg</th>
      <th>wr_Rec_TD</th>
      <th>wr_Rush_Att</th>
      <th>wr_Rush_Yds</th>
      <th>wr_Rush_Avg</th>
      <th>wr_Rush_TD</th>
      <th>wr_Scrim_Plays</th>
      <th>wr_Scrim_Yds</th>
      <th>wr_Scrim_Avg</th>
      <th>wr_Scrim_TD</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>198.000000</td>
      <td>198.000000</td>
      <td>198.000000</td>
      <td>30.000000</td>
      <td>30.000000</td>
      <td>30.000000</td>
      <td>30.000000</td>
      <td>30.000000</td>
      <td>30.000000</td>
      <td>30.000000</td>
      <td>...</td>
      <td>103.000000</td>
      <td>103.000000</td>
      <td>103.000000</td>
      <td>103.000000</td>
      <td>68.000000</td>
      <td>103.000000</td>
      <td>103.000000</td>
      <td>103.000000</td>
      <td>103.000000</td>
      <td>103.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>127.510101</td>
      <td>2009.989899</td>
      <td>2010.989899</td>
      <td>271.533333</td>
      <td>423.700000</td>
      <td>63.720000</td>
      <td>3548.066667</td>
      <td>8.373333</td>
      <td>8.793333</td>
      <td>27.933333</td>
      <td>...</td>
      <td>14.339806</td>
      <td>7.349515</td>
      <td>36.737864</td>
      <td>209.213592</td>
      <td>5.994118</td>
      <td>2.174757</td>
      <td>95.524272</td>
      <td>1062.631068</td>
      <td>13.483495</td>
      <td>9.524272</td>
    </tr>
    <tr>
      <th>std</th>
      <td>76.862838</td>
      <td>0.824682</td>
      <td>0.824682</td>
      <td>59.384622</td>
      <td>74.102981</td>
      <td>4.827465</td>
      <td>853.700058</td>
      <td>1.467095</td>
      <td>1.964677</td>
      <td>10.612073</td>
      <td>...</td>
      <td>3.760641</td>
      <td>4.209487</td>
      <td>79.433652</td>
      <td>446.268750</td>
      <td>6.409831</td>
      <td>4.680848</td>
      <td>72.684902</td>
      <td>373.411380</td>
      <td>4.190585</td>
      <td>4.136980</td>
    </tr>
    <tr>
      <th>min</th>
      <td>1.000000</td>
      <td>2009.000000</td>
      <td>2010.000000</td>
      <td>152.000000</td>
      <td>270.000000</td>
      <td>53.200000</td>
      <td>1812.000000</td>
      <td>6.500000</td>
      <td>5.800000</td>
      <td>14.000000</td>
      <td>...</td>
      <td>4.800000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>-7.000000</td>
      <td>-3.500000</td>
      <td>0.000000</td>
      <td>25.000000</td>
      <td>324.000000</td>
      <td>4.800000</td>
      <td>1.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>57.000000</td>
      <td>2009.000000</td>
      <td>2010.000000</td>
      <td>234.750000</td>
      <td>371.000000</td>
      <td>60.725000</td>
      <td>2813.500000</td>
      <td>7.325000</td>
      <td>7.475000</td>
      <td>20.500000</td>
      <td>...</td>
      <td>11.750000</td>
      <td>4.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>3.000000</td>
      <td>0.000000</td>
      <td>56.000000</td>
      <td>770.000000</td>
      <td>11.250000</td>
      <td>7.000000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>127.000000</td>
      <td>2010.000000</td>
      <td>2011.000000</td>
      <td>265.000000</td>
      <td>419.500000</td>
      <td>63.450000</td>
      <td>3545.500000</td>
      <td>8.050000</td>
      <td>8.350000</td>
      <td>26.000000</td>
      <td>...</td>
      <td>14.600000</td>
      <td>8.000000</td>
      <td>2.000000</td>
      <td>12.000000</td>
      <td>5.000000</td>
      <td>0.000000</td>
      <td>71.000000</td>
      <td>1083.000000</td>
      <td>14.000000</td>
      <td>9.000000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>198.500000</td>
      <td>2011.000000</td>
      <td>2012.000000</td>
      <td>302.000000</td>
      <td>483.000000</td>
      <td>66.875000</td>
      <td>4083.250000</td>
      <td>9.075000</td>
      <td>9.875000</td>
      <td>32.000000</td>
      <td>...</td>
      <td>16.500000</td>
      <td>10.000000</td>
      <td>10.500000</td>
      <td>62.000000</td>
      <td>7.025000</td>
      <td>1.000000</td>
      <td>99.000000</td>
      <td>1290.000000</td>
      <td>15.750000</td>
      <td>12.000000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>256.000000</td>
      <td>2011.000000</td>
      <td>2012.000000</td>
      <td>402.000000</td>
      <td>560.000000</td>
      <td>76.300000</td>
      <td>5671.000000</td>
      <td>11.600000</td>
      <td>13.000000</td>
      <td>60.000000</td>
      <td>...</td>
      <td>23.100000</td>
      <td>18.000000</td>
      <td>302.000000</td>
      <td>1760.000000</td>
      <td>37.500000</td>
      <td>18.000000</td>
      <td>331.000000</td>
      <td>2038.000000</td>
      <td>23.100000</td>
      <td>21.000000</td>
    </tr>
  </tbody>
</table>
<p>8 rows × 40 columns</p>
</div>



### What is the minimum number of rows we expect for the positional statistics? 
It should be at least be equal to or greater than the number of players in the position as there are some metrics that overlap between positions. As we can see below we are clearly missing stats for all positions


```python
# The Quarterbacks on average are selected earlier in the draft then running back and wide reciever positions. 
complete.groupby(by='Pos')['Pick'].count().reset_index()
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
      <th>Pos</th>
      <th>Pick</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>QB</td>
      <td>37</td>
    </tr>
    <tr>
      <th>1</th>
      <td>RB</td>
      <td>64</td>
    </tr>
    <tr>
      <th>2</th>
      <td>WR</td>
      <td>97</td>
    </tr>
  </tbody>
</table>
</div>




```python
### Using the QB position to verify, it looks like we are missing data for 7 qbs

complete[['Player','qb_Cmp']][complete['Pos']=='QB'].info()

```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 37 entries, 0 to 753
    Data columns (total 2 columns):
     #   Column  Non-Null Count  Dtype  
    ---  ------  --------------  -----  
     0   Player  37 non-null     object 
     1   qb_Cmp  30 non-null     float64
    dtypes: float64(1), object(1)
    memory usage: 888.0+ bytes
    

###  Now I identify the players that are missing stats
I will use this list to try to do fuzzy string matching the player names in the passing, rushing, and receiving dataframes


```python
# Find the players that are missing data due to difference in name between datasets
missing_qbs = list(complete['Player'][(complete['Pos']=='QB')&(complete['qb_Cmp'].isnull())])
missing_wrs = list(complete['Player'][(complete['Pos']=='WR')&(complete['wr_Rec_Yds'].isnull())])
missing_rbs = list(complete['Player'][(complete['Pos']=='RB')&(complete['rb_Rush_Yds'].isnull())])

qb_names = list(passing.Player.drop_duplicates())
wr_names =  list(receiving.Player.drop_duplicates())
rb_names =  list(rushing.Player.drop_duplicates())

```

### Use fuzzy string matching score to identify player name matches from draft data frame to the other dataframes
If we do not find a match in the datasets, we will exclude the players from the model as there is missing data from the website we scraped from

Note: Please see source citations below for resource on this user defined function.


```python
from fuzzywuzzy import fuzz
# Utilizing a function that does fuzzy string matching to return the best name match; 
#if we do not find a match in the datasets 
#we will exclude the players from the model as there is missing data from the website we scraped from
def match_name(name, list_names, min_score=0):
    # -1 score and empty name for missing names
    max_score = -1
    max_name = ""
    for name2 in list_names:
        #Finding fuzzy match score
        score = fuzz.ratio(name, name2)
        # Checking if we are above our threshold and have a better score
        if (score > min_score) & (score > max_score):
            max_name = name2
            max_score = score
    return (max_name, max_score)
```

###  Create a List of Player Name Matches for the players with missing stats


```python
# Determine best matches for all missing players accross dataframes

dict_list = []
dictionary = {}

#Iterating through players that are missing positional stats
for name in missing_qbs:
        # Use our method to find best match, we can set a threshold here
        match = match_name(name, qb_names, 80)

        # New dict for storing data
        dict_ = {}
        dict_.update({"player_name" : name})
        dict_.update({"match_name" : match[0]})
        dict_list.append(dict_)

qb_names_final = pd.DataFrame(dict_list)
names_final = qb_names_final[qb_names_final['match_name']!='']



for name in missing_wrs:
        # Use our method to find best match, we can set a threshold here
        match = match_name(name, wr_names, 80)

        # New dict for storing data
        dict_ = {}
        dict_.update({"player_name" : name})
        dict_.update({"match_name" : match[0]})
        dict_list.append(dict_)

wr_names_final = pd.DataFrame(dict_list)
wr_names_final = wr_names_final[wr_names_final['match_name']!='']
names_final = names_final.append(wr_names_final)



for name in missing_rbs:
        # Use our method to find best match, we can set a threshold here
        match = match_name(name, rb_names, 80)

        # New dict for storing data
        dict_ = {}
        dict_.update({"player_name" : name})
        dict_.update({"match_name" : match[0]})
        dict_list.append(dict_)
        
        

rb_names_final = pd.DataFrame(dict_list)

# Get a list of player names with no matches that should be removed from the final dataset
drop_players = list(rb_names_final['player_name'][rb_names_final['match_name']==''])

# Remove non-matches and use matches for replacing Player name values in the draft dataframe
rb_names_final = rb_names_final[rb_names_final['match_name']!='']
names_final = names_final.append(rb_names_final)
# Remove duplicates
names_final = names_final.drop_duplicates().reset_index()
# Remove incorrect player name match
names_final = names_final[names_final['player_name']!='Justin Watson']
# View list of names that will be used for replacement
names_final
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
      <th>index</th>
      <th>player_name</th>
      <th>match_name</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2</td>
      <td>Gardner Minshew II</td>
      <td>Gardner Minshew</td>
    </tr>
    <tr>
      <th>1</th>
      <td>3</td>
      <td>Tua Tagovailoa</td>
      <td>Tua Tagovailoa</td>
    </tr>
    <tr>
      <th>2</th>
      <td>6</td>
      <td>Nate Stanley</td>
      <td>Nathan Stanley</td>
    </tr>
    <tr>
      <th>4</th>
      <td>12</td>
      <td>Cedrick Wilson</td>
      <td>Cedric Wilson</td>
    </tr>
    <tr>
      <th>5</th>
      <td>14</td>
      <td>Gary Jennings Jr</td>
      <td>Gary Jennings</td>
    </tr>
    <tr>
      <th>6</th>
      <td>15</td>
      <td>Terry Godwin</td>
      <td>Terry Godwin</td>
    </tr>
    <tr>
      <th>7</th>
      <td>17</td>
      <td>Ronald Jones II</td>
      <td>Ronald Jones</td>
    </tr>
    <tr>
      <th>8</th>
      <td>22</td>
      <td>Josh Jacobs</td>
      <td>Joshua Jacobs</td>
    </tr>
    <tr>
      <th>9</th>
      <td>23</td>
      <td>Benny Snell Jr</td>
      <td>Benjamin Snell Jr</td>
    </tr>
    <tr>
      <th>10</th>
      <td>25</td>
      <td>Ty Johnson</td>
      <td>Ty Johnson</td>
    </tr>
    <tr>
      <th>11</th>
      <td>26</td>
      <td>Rodney Anderson</td>
      <td>Rodney Anderson</td>
    </tr>
    <tr>
      <th>12</th>
      <td>28</td>
      <td>Kerrith Whyte Jr</td>
      <td>Kerrith Whyte</td>
    </tr>
    <tr>
      <th>13</th>
      <td>31</td>
      <td>LaMical Perine</td>
      <td>Lamical Perine</td>
    </tr>
    <tr>
      <th>14</th>
      <td>32</td>
      <td>Anthony McFarland Jr</td>
      <td>Anthony McFarland</td>
    </tr>
    <tr>
      <th>15</th>
      <td>33</td>
      <td>DeeJay Dallas</td>
      <td>Deejay Dallas</td>
    </tr>
  </tbody>
</table>
</div>



### Replace existing draft data frame player name with the matched player name for joining purposes


```python
# Replace existing draft data frame player name with the matched player name for joining purposes
import numpy as np
for player, match in zip(list(names_final.player_name),list(names_final.match_name)):
    draft['Player'] = np.where(draft['Player']==player, match, draft['Player'])
```

### Combine dataframes using the update Player names from the fuzzy string matching


```python
# Reattempt the joins between the various dataframes using only the Player name. Note: there will be
rushing.drop(columns='Year',inplace=True)
passing.drop(columns='Year',inplace=True)
receiving.drop(columns='Year',inplace=True)
complete = pd.merge(draft, passing, on=['Player'], how='left')
complete = pd.merge(complete, rushing, on=['Player'], how='left')
complete = pd.merge(complete, receiving, on=['Player'], how='left')

complete = complete[complete['Pos'].isin(['QB','WR','RB'])]

complete.shape
```




    (348, 52)



### Select Most Recent Player Stats
Since I am no longer joining on the Year there are now duplicate records per player. I am going to assume that the most recent year's worth of stats is the most relevant to the draft selection. Therefore, I will use the max index to select the most recent year for each player. I will then fill in missing values with 0.


```python
complete = complete.reset_index()
most_recent_stats = complete.groupby(by='Player')['index'].max().reset_index()
complete = pd.merge(complete, most_recent_stats, on=['Player','index'],how='inner')

# Remove missing players that have no high confidence fuzzy string match
complete = complete[~complete['Player'].isin(drop_players)]

# Fill in missing values with 0. 

complete = complete.fillna(0)

```

### Final Verification of Dataset

The QB, RB, and WR numbers are matching what we expect, there are no missing values for any of the stats. This can be misleading because I replaced missing values with 0, but since I am assuming those missing stats were not related to the player's position it will be acceptable for now. 

### The Dataset looks good for now, let's proceed to the featuring engineering and modeling.


```python
print(complete.groupby('Pos')['Player'].count())
complete.info()
```

    Pos
    QB    33
    RB    56
    WR    91
    Name: Player, dtype: int64
    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 180 entries, 0 to 197
    Data columns (total 53 columns):
     #   Column          Non-Null Count  Dtype  
    ---  ------          --------------  -----  
     0   index           180 non-null    int64  
     1   Pick            180 non-null    int64  
     2   NFLTeam         180 non-null    object 
     3   Pos             180 non-null    object 
     4   Player          180 non-null    object 
     5   Year            180 non-null    int64  
     6   draft_year      180 non-null    int64  
     7   School          180 non-null    object 
     8   Height          180 non-null    object 
     9   Wt              180 non-null    object 
     10  40YD            180 non-null    object 
     11  Vertical        180 non-null    object 
     12  BenchReps       180 non-null    object 
     13  Broad Jump      180 non-null    object 
     14  3Cone           180 non-null    object 
     15  Shuttle         180 non-null    object 
     16  qb_Cmp          180 non-null    float64
     17  qb_Att          180 non-null    float64
     18  qb_Pct          180 non-null    float64
     19  qb_P_Yds        180 non-null    float64
     20  qb_Y/A          180 non-null    float64
     21  qb_AY/A         180 non-null    float64
     22  qb_P_TD         180 non-null    float64
     23  qb_Int          180 non-null    float64
     24  qb_Rate         180 non-null    float64
     25  qb_R_Att        180 non-null    float64
     26  qb_R_Yds        180 non-null    float64
     27  qb_R_Avg        180 non-null    float64
     28  qb_R_TD         180 non-null    float64
     29  rb_Rush_Att     180 non-null    float64
     30  rb_Rush_Yds     180 non-null    float64
     31  rb_Rush_Avg     180 non-null    float64
     32  rb_Rush_TD      180 non-null    float64
     33  rb_Rec          180 non-null    float64
     34  rb_Rec_Yds      180 non-null    float64
     35  rb_Rec_Avg      180 non-null    float64
     36  rb_Rec_TD       180 non-null    float64
     37  rb_Scrim_Plays  180 non-null    float64
     38  rb_Scrim_Yds    180 non-null    float64
     39  rb_Scrim_Avg    180 non-null    float64
     40  rb_Scrim_TD     180 non-null    float64
     41  wr_Rec          180 non-null    float64
     42  wr_Rec_Yds      180 non-null    float64
     43  wr_Rec_Avg      180 non-null    float64
     44  wr_Rec_TD       180 non-null    float64
     45  wr_Rush_Att     180 non-null    float64
     46  wr_Rush_Yds     180 non-null    float64
     47  wr_Rush_Avg     180 non-null    float64
     48  wr_Rush_TD      180 non-null    float64
     49  wr_Scrim_Plays  180 non-null    float64
     50  wr_Scrim_Yds    180 non-null    float64
     51  wr_Scrim_Avg    180 non-null    float64
     52  wr_Scrim_TD     180 non-null    float64
    dtypes: float64(37), int64(4), object(12)
    memory usage: 75.9+ KB
    


```python
complete.to_csv('complete_nfl_draft_predictions_dataset_2017_2019.csv',index=None)
```

<center>  Works Cited </center>
“College Football Statistics and History: College Football at Sports.” Sports-Reference.com, 2020, www.sports-reference.com/cfb/.

Jeannier, Roland. “Combining Datasets with Fuzzy Matching.” Medium, Medium, 18 Aug. 2017, medium.com/@rtjeannier/combining-data-sets-with-fuzzy-matching-17efcb510ab2.

“NFL and AFL Draft History.” Pro-Football-Reference, Sports-Reference, www.pro-football-reference.com/draft/.
