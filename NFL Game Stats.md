
## Scraping methods


```python
import urllib.request as url
from bs4 import BeautifulSoup 
import csv


def get_game_info(game_id):
    try: 
        web_address = 'http://www.espn.com/nfl/matchup?gameId=' + str(game_id)
        game_webpage = url.urlopen(web_address)
        soup = BeautifulSoup(game_webpage, 'html.parser')
        game_info = soup.title.string
        game_info_split = game_info.split('-')
        teams = game_info_split[0]
        date = game_info_split[2]
        return (teams, date, '')
    except: 
            print('error')
            return('','', web_address)
    



```


```python
#game info test

game_id = 400951552
teams_list = []
date_list = []
ids = []
while game_id < 400951835:
    teams, date = get_game_info(game_id)
    if 'August' not in date and date != '' :
        teams_list.append(teams)
        date_list.append(date)
        ids.append(game_id)
    print(game_id, teams, date)
    game_id += 1
```


```python
# find team stats (independent variables)





def get_game_stats(game_id):
    stats = []
    missed = []
    try:
        web_address = 'http://www.espn.com/nfl/matchup?gameId=' + str(game_id)
        game_webpage = url.urlopen(web_address)
        orig_soup = BeautifulSoup(game_webpage, 'html.parser')
        game_info = orig_soup.title.string              # get matchup and game date
        game_info_split = game_info.split('-')
        teams = game_info_split[0]
        date = game_info_split[2]
        if 'August' not in date:
            stats.append(teams)
            home_away = teams.split('vs.')
            stats.append(date)
            stats.append(home_away[1])
            stats.append(home_away[0])
            soup = orig_soup.find_all('tr')
            for line in soup[4:29]:    #get home and away stats
                index = 1
                while index <= 2:
                    soup = line.find_all('td')
                    value = soup[index].contents[0].strip()
                    try:
                        stats.append(float(value))
                    except:
                        if '-' in value:
                            value = value.split('-')
                            if value[1] != '0':
                                value = round(int(value[0])/int(value[1]),4)
                                stats.append(value)
                            else:
                                stats.append(0)
                        else:
                            top = value.split(':')
                            mins_as_per = round(int(value[0]) / 60,2)
                            secs_as_per = round(int(value[1]) / 3600,2)
                            per_poss = mins_as_per + secs_as_per
                            stats.append(per_poss)
                    index += 1
    except:
            missed.append(web_address)
    return stats, missed

    
    







```


```python
game_id = 400951552
list_stats = []
missed_list = []
num_games = 0

while num_games <= 256:
    if game_id == 400951581:
        game_id += 1        #for Dolphins Vs. Bucs 
    stats, missed = get_game_stats(game_id)
    if stats != []:
        
        list_stats.append(stats)
        game_id += 1
        num_games += 1
    else:
        missed_list.append(missed)
        game_id += 1
    print(num_games)
```


```python

def get_col_heads(game_id):
    variables = []
    variables.append('Matchup')
    variables.append('Date')
    variables.append('Home')
    variables.append('Away')
    web_address = 'http://www.espn.com/nfl/matchup?gameId=' + str(game_id)
    game_webpage = url.urlopen(web_address)
    orig_soup = BeautifulSoup(game_webpage, 'html.parser')
    soup = orig_soup.find_all('tr')
    for line in soup[4:29]:
        soup = line.find_all('td')
        var = soup[0].contents[0].strip()
        variables.append(var + ' ' + 'home')
        variables.append(var + ' ' +  'away')
    return variables

headers = get_col_heads(game_id)

```


```python
#import pandas as pd
#df_2017 = pd.DataFrame(list_stats, columns = headers)


#df_2017 = pd.read_pickle('NFL_2017')
#df_2017.to_pickle('NFL_2017')

#home = df_2017['Away']
#away = df_2017['Home']
#df_2017['Home'] = home
#df_2017['Away'] = away

#df_2017['Date'] = pd.to_datetime(df_2017.Date)
#df_2017 = df_2017.sort_values('Date')

#game_id = 400951779
#num_games = 0
#fix_na = []

#while num_games <= 11:
    #if game_id == 400951581:
        #game_id += 1        #for Dolphins Vs. Bucs 
    #stats, missed = get_game_stats(game_id)
    #if stats != []:
        #num_games += 1
        #fix_na.append(stats)
        #game_id += 1
        #print(num_games)


#for game, idx in enumerate(range(241,253)):
    #df_2017.loc[idx:idx] = fix_na[game]

df_2017.head(n= -30)
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
      <th>Matchup</th>
      <th>Date</th>
      <th>Home</th>
      <th>Away</th>
      <th>1st Downs home</th>
      <th>1st Downs away</th>
      <th>Passing 1st downs home</th>
      <th>Passing 1st downs away</th>
      <th>Rushing 1st downs home</th>
      <th>Rushing 1st downs away</th>
      <th>...</th>
      <th>Turnovers home</th>
      <th>Turnovers away</th>
      <th>Fumbles lost home</th>
      <th>Fumbles lost away</th>
      <th>Interceptions thrown home</th>
      <th>Interceptions thrown away</th>
      <th>Defensive / Special Teams TDs home</th>
      <th>Defensive / Special Teams TDs away</th>
      <th>Possession home</th>
      <th>Possession away</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>14</th>
      <td>Chiefs vs. Patriots</td>
      <td>2017-09-07</td>
      <td>Patriots</td>
      <td>Chiefs</td>
      <td>26.0</td>
      <td>25.0</td>
      <td>17.0</td>
      <td>11.0</td>
      <td>8.0</td>
      <td>10.0</td>
      <td>...</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.05</td>
      <td>0.03</td>
    </tr>
    <tr>
      <th>31</th>
      <td>Raiders vs. Titans</td>
      <td>2017-09-10</td>
      <td>Titans</td>
      <td>Raiders</td>
      <td>22.0</td>
      <td>21.0</td>
      <td>16.0</td>
      <td>15.0</td>
      <td>5.0</td>
      <td>5.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.05</td>
      <td>0.03</td>
    </tr>
    <tr>
      <th>24</th>
      <td>Cardinals vs. Lions</td>
      <td>2017-09-10</td>
      <td>Lions</td>
      <td>Cardinals</td>
      <td>24.0</td>
      <td>19.0</td>
      <td>15.0</td>
      <td>15.0</td>
      <td>4.0</td>
      <td>3.0</td>
      <td>...</td>
      <td>4.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>3.0</td>
      <td>1.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>0.03</td>
      <td>0.05</td>
    </tr>
    <tr>
      <th>55</th>
      <td>Giants vs. Cowboys</td>
      <td>2017-09-10</td>
      <td>Cowboys</td>
      <td>Giants</td>
      <td>13.0</td>
      <td>22.0</td>
      <td>10.0</td>
      <td>12.0</td>
      <td>2.0</td>
      <td>9.0</td>
      <td>...</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.03</td>
      <td>0.05</td>
    </tr>
    <tr>
      <th>22</th>
      <td>Steelers vs. Browns</td>
      <td>2017-09-10</td>
      <td>Browns</td>
      <td>Steelers</td>
      <td>16.0</td>
      <td>20.0</td>
      <td>13.0</td>
      <td>12.0</td>
      <td>1.0</td>
      <td>3.0</td>
      <td>...</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.03</td>
      <td>0.05</td>
    </tr>
    <tr>
      <th>52</th>
      <td>Panthers vs. 49ers</td>
      <td>2017-09-10</td>
      <td>49ers</td>
      <td>Panthers</td>
      <td>20.0</td>
      <td>13.0</td>
      <td>8.0</td>
      <td>10.0</td>
      <td>9.0</td>
      <td>2.0</td>
      <td>...</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.05</td>
      <td>0.03</td>
    </tr>
    <tr>
      <th>20</th>
      <td>Ravens vs. Bengals</td>
      <td>2017-09-10</td>
      <td>Bengals</td>
      <td>Ravens</td>
      <td>17.0</td>
      <td>14.0</td>
      <td>5.0</td>
      <td>8.0</td>
      <td>8.0</td>
      <td>3.0</td>
      <td>...</td>
      <td>1.0</td>
      <td>5.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>4.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.05</td>
      <td>0.03</td>
    </tr>
    <tr>
      <th>48</th>
      <td>Seahawks vs. Packers</td>
      <td>2017-09-10</td>
      <td>Packers</td>
      <td>Seahawks</td>
      <td>12.0</td>
      <td>26.0</td>
      <td>8.0</td>
      <td>16.0</td>
      <td>3.0</td>
      <td>8.0</td>
      <td>...</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.03</td>
      <td>0.05</td>
    </tr>
    <tr>
      <th>18</th>
      <td>Falcons vs. Bears</td>
      <td>2017-09-10</td>
      <td>Bears</td>
      <td>Falcons</td>
      <td>18.0</td>
      <td>20.0</td>
      <td>12.0</td>
      <td>11.0</td>
      <td>4.0</td>
      <td>6.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.05</td>
      <td>0.03</td>
    </tr>
    <tr>
      <th>44</th>
      <td>Colts vs. Rams</td>
      <td>2017-09-10</td>
      <td>Rams</td>
      <td>Colts</td>
      <td>10.0</td>
      <td>19.0</td>
      <td>6.0</td>
      <td>13.0</td>
      <td>3.0</td>
      <td>5.0</td>
      <td>...</td>
      <td>3.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>2.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>4.0</td>
      <td>0.03</td>
      <td>0.05</td>
    </tr>
    <tr>
      <th>15</th>
      <td>Jets vs. Bills</td>
      <td>2017-09-10</td>
      <td>Bills</td>
      <td>Jets</td>
      <td>11.0</td>
      <td>23.0</td>
      <td>8.0</td>
      <td>11.0</td>
      <td>2.0</td>
      <td>10.0</td>
      <td>...</td>
      <td>2.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>2.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.03</td>
      <td>0.05</td>
    </tr>
    <tr>
      <th>28</th>
      <td>Jaguars vs. Texans</td>
      <td>2017-09-10</td>
      <td>Texans</td>
      <td>Jaguars</td>
      <td>19.0</td>
      <td>23.0</td>
      <td>8.0</td>
      <td>11.0</td>
      <td>7.0</td>
      <td>4.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>4.0</td>
      <td>0.0</td>
      <td>3.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>2.0</td>
      <td>0.0</td>
      <td>0.05</td>
      <td>0.03</td>
    </tr>
    <tr>
      <th>39</th>
      <td>Eagles vs. Redskins</td>
      <td>2017-09-10</td>
      <td>Redskins</td>
      <td>Eagles</td>
      <td>19.0</td>
      <td>16.0</td>
      <td>17.0</td>
      <td>12.0</td>
      <td>1.0</td>
      <td>3.0</td>
      <td>...</td>
      <td>2.0</td>
      <td>4.0</td>
      <td>1.0</td>
      <td>3.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>0.05</td>
      <td>0.03</td>
    </tr>
    <tr>
      <th>62</th>
      <td>Chargers vs. Broncos</td>
      <td>2017-09-11</td>
      <td>Broncos</td>
      <td>Chargers</td>
      <td>17.0</td>
      <td>22.0</td>
      <td>12.0</td>
      <td>14.0</td>
      <td>3.0</td>
      <td>7.0</td>
      <td>...</td>
      <td>1.0</td>
      <td>2.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.03</td>
      <td>0.05</td>
    </tr>
    <tr>
      <th>59</th>
      <td>Saints vs. Vikings</td>
      <td>2017-09-11</td>
      <td>Vikings</td>
      <td>Saints</td>
      <td>19.0</td>
      <td>23.0</td>
      <td>14.0</td>
      <td>14.0</td>
      <td>3.0</td>
      <td>5.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.03</td>
      <td>0.05</td>
    </tr>
    <tr>
      <th>67</th>
      <td>Texans vs. Bengals</td>
      <td>2017-09-14</td>
      <td>Bengals</td>
      <td>Texans</td>
      <td>14.0</td>
      <td>12.0</td>
      <td>6.0</td>
      <td>8.0</td>
      <td>6.0</td>
      <td>4.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.05</td>
      <td>0.03</td>
    </tr>
    <tr>
      <th>83</th>
      <td>Eagles vs. Chiefs</td>
      <td>2017-09-17</td>
      <td>Chiefs</td>
      <td>Eagles</td>
      <td>27.0</td>
      <td>16.0</td>
      <td>19.0</td>
      <td>10.0</td>
      <td>6.0</td>
      <td>6.0</td>
      <td>...</td>
      <td>2.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.05</td>
      <td>0.03</td>
    </tr>
    <tr>
      <th>82</th>
      <td>Titans vs. Jaguars</td>
      <td>2017-09-17</td>
      <td>Jaguars</td>
      <td>Titans</td>
      <td>21.0</td>
      <td>20.0</td>
      <td>9.0</td>
      <td>13.0</td>
      <td>12.0</td>
      <td>6.0</td>
      <td>...</td>
      <td>1.0</td>
      <td>3.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>2.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.05</td>
      <td>0.03</td>
    </tr>
    <tr>
      <th>90</th>
      <td>Vikings vs. Steelers</td>
      <td>2017-09-17</td>
      <td>Steelers</td>
      <td>Vikings</td>
      <td>14.0</td>
      <td>22.0</td>
      <td>8.0</td>
      <td>12.0</td>
      <td>5.0</td>
      <td>6.0</td>
      <td>...</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.03</td>
      <td>0.05</td>
    </tr>
    <tr>
      <th>92</th>
      <td>Bears vs. Buccaneers</td>
      <td>2017-09-17</td>
      <td>Buccaneers</td>
      <td>Bears</td>
      <td>19.0</td>
      <td>22.0</td>
      <td>16.0</td>
      <td>13.0</td>
      <td>1.0</td>
      <td>6.0</td>
      <td>...</td>
      <td>4.0</td>
      <td>1.0</td>
      <td>2.0</td>
      <td>1.0</td>
      <td>2.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>2.0</td>
      <td>0.03</td>
      <td>0.05</td>
    </tr>
    <tr>
      <th>79</th>
      <td>Cardinals vs. Colts</td>
      <td>2017-09-17</td>
      <td>Colts</td>
      <td>Cardinals</td>
      <td>17.0</td>
      <td>18.0</td>
      <td>14.0</td>
      <td>11.0</td>
      <td>2.0</td>
      <td>6.0</td>
      <td>...</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.03</td>
      <td>0.05</td>
    </tr>
    <tr>
      <th>126</th>
      <td>Packers vs. Falcons</td>
      <td>2017-09-17</td>
      <td>Falcons</td>
      <td>Packers</td>
      <td>24.0</td>
      <td>19.0</td>
      <td>18.0</td>
      <td>12.0</td>
      <td>4.0</td>
      <td>7.0</td>
      <td>...</td>
      <td>2.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>2.0</td>
      <td>0.05</td>
      <td>0.03</td>
    </tr>
    <tr>
      <th>77</th>
      <td>Bills vs. Panthers</td>
      <td>2017-09-17</td>
      <td>Panthers</td>
      <td>Bills</td>
      <td>10.0</td>
      <td>19.0</td>
      <td>8.0</td>
      <td>12.0</td>
      <td>2.0</td>
      <td>6.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.03</td>
      <td>0.05</td>
    </tr>
    <tr>
      <th>113</th>
      <td>Dolphins vs. Chargers</td>
      <td>2017-09-17</td>
      <td>Chargers</td>
      <td>Dolphins</td>
      <td>21.0</td>
      <td>24.0</td>
      <td>13.0</td>
      <td>19.0</td>
      <td>6.0</td>
      <td>3.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.05</td>
      <td>0.03</td>
    </tr>
    <tr>
      <th>73</th>
      <td>Browns vs. Ravens</td>
      <td>2017-09-17</td>
      <td>Ravens</td>
      <td>Browns</td>
      <td>20.0</td>
      <td>24.0</td>
      <td>12.0</td>
      <td>16.0</td>
      <td>6.0</td>
      <td>7.0</td>
      <td>...</td>
      <td>5.0</td>
      <td>2.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>4.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.03</td>
      <td>0.05</td>
    </tr>
    <tr>
      <th>123</th>
      <td>49ers vs. Seahawks</td>
      <td>2017-09-17</td>
      <td>Seahawks</td>
      <td>49ers</td>
      <td>11.0</td>
      <td>21.0</td>
      <td>4.0</td>
      <td>10.0</td>
      <td>5.0</td>
      <td>9.0</td>
      <td>...</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.03</td>
      <td>0.05</td>
    </tr>
    <tr>
      <th>121</th>
      <td>Redskins vs. Rams</td>
      <td>2017-09-17</td>
      <td>Rams</td>
      <td>Redskins</td>
      <td>21.0</td>
      <td>14.0</td>
      <td>12.0</td>
      <td>9.0</td>
      <td>8.0</td>
      <td>4.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>2.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.05</td>
      <td>0.03</td>
    </tr>
    <tr>
      <th>120</th>
      <td>Cowboys vs. Broncos</td>
      <td>2017-09-17</td>
      <td>Broncos</td>
      <td>Cowboys</td>
      <td>16.0</td>
      <td>26.0</td>
      <td>12.0</td>
      <td>12.0</td>
      <td>1.0</td>
      <td>11.0</td>
      <td>...</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>2.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>2.0</td>
      <td>0.03</td>
      <td>0.05</td>
    </tr>
    <tr>
      <th>86</th>
      <td>Patriots vs. Saints</td>
      <td>2017-09-17</td>
      <td>Saints</td>
      <td>Patriots</td>
      <td>29.0</td>
      <td>20.0</td>
      <td>19.0</td>
      <td>15.0</td>
      <td>8.0</td>
      <td>5.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.05</td>
      <td>0.03</td>
    </tr>
    <tr>
      <th>116</th>
      <td>Jets vs. Raiders</td>
      <td>2017-09-17</td>
      <td>Raiders</td>
      <td>Jets</td>
      <td>17.0</td>
      <td>21.0</td>
      <td>9.0</td>
      <td>13.0</td>
      <td>6.0</td>
      <td>6.0</td>
      <td>...</td>
      <td>2.0</td>
      <td>0.0</td>
      <td>2.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.03</td>
      <td>0.05</td>
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
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>94</th>
      <td>Packers vs. Browns</td>
      <td>2017-12-10</td>
      <td>Browns</td>
      <td>Packers</td>
      <td>24.0</td>
      <td>17.0</td>
      <td>14.0</td>
      <td>12.0</td>
      <td>7.0</td>
      <td>5.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>2.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>2.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.05</td>
      <td>0.03</td>
    </tr>
    <tr>
      <th>96</th>
      <td>49ers vs. Texans</td>
      <td>2017-12-10</td>
      <td>Texans</td>
      <td>49ers</td>
      <td>20.0</td>
      <td>18.0</td>
      <td>12.0</td>
      <td>12.0</td>
      <td>4.0</td>
      <td>4.0</td>
      <td>...</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.05</td>
      <td>0.03</td>
    </tr>
    <tr>
      <th>99</th>
      <td>Seahawks vs. Jaguars</td>
      <td>2017-12-10</td>
      <td>Jaguars</td>
      <td>Seahawks</td>
      <td>17.0</td>
      <td>20.0</td>
      <td>6.0</td>
      <td>12.0</td>
      <td>9.0</td>
      <td>7.0</td>
      <td>...</td>
      <td>3.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>3.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.03</td>
      <td>0.05</td>
    </tr>
    <tr>
      <th>101</th>
      <td>Raiders vs. Chiefs</td>
      <td>2017-12-10</td>
      <td>Chiefs</td>
      <td>Raiders</td>
      <td>15.0</td>
      <td>23.0</td>
      <td>10.0</td>
      <td>12.0</td>
      <td>3.0</td>
      <td>10.0</td>
      <td>...</td>
      <td>3.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>2.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.03</td>
      <td>0.05</td>
    </tr>
    <tr>
      <th>107</th>
      <td>Lions vs. Buccaneers</td>
      <td>2017-12-10</td>
      <td>Buccaneers</td>
      <td>Lions</td>
      <td>21.0</td>
      <td>28.0</td>
      <td>16.0</td>
      <td>19.0</td>
      <td>3.0</td>
      <td>8.0</td>
      <td>...</td>
      <td>3.0</td>
      <td>5.0</td>
      <td>1.0</td>
      <td>3.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.03</td>
      <td>0.05</td>
    </tr>
    <tr>
      <th>109</th>
      <td>Titans vs. Cardinals</td>
      <td>2017-12-10</td>
      <td>Cardinals</td>
      <td>Titans</td>
      <td>14.0</td>
      <td>16.0</td>
      <td>9.0</td>
      <td>9.0</td>
      <td>3.0</td>
      <td>5.0</td>
      <td>...</td>
      <td>2.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>2.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.03</td>
      <td>0.05</td>
    </tr>
    <tr>
      <th>134</th>
      <td>Redskins vs. Chargers</td>
      <td>2017-12-10</td>
      <td>Chargers</td>
      <td>Redskins</td>
      <td>9.0</td>
      <td>24.0</td>
      <td>7.0</td>
      <td>14.0</td>
      <td>1.0</td>
      <td>7.0</td>
      <td>...</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.03</td>
      <td>0.05</td>
    </tr>
    <tr>
      <th>136</th>
      <td>Eagles vs. Rams</td>
      <td>2017-12-10</td>
      <td>Rams</td>
      <td>Eagles</td>
      <td>29.0</td>
      <td>17.0</td>
      <td>18.0</td>
      <td>8.0</td>
      <td>5.0</td>
      <td>6.0</td>
      <td>...</td>
      <td>1.0</td>
      <td>2.0</td>
      <td>0.0</td>
      <td>2.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>2.0</td>
      <td>1.0</td>
      <td>0.05</td>
      <td>0.03</td>
    </tr>
    <tr>
      <th>141</th>
      <td>Ravens vs. Steelers</td>
      <td>2017-12-10</td>
      <td>Steelers</td>
      <td>Ravens</td>
      <td>26.0</td>
      <td>31.0</td>
      <td>13.0</td>
      <td>23.0</td>
      <td>9.0</td>
      <td>6.0</td>
      <td>...</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.03</td>
      <td>0.05</td>
    </tr>
    <tr>
      <th>139</th>
      <td>Cowboys vs. Giants</td>
      <td>2017-12-10</td>
      <td>Giants</td>
      <td>Cowboys</td>
      <td>19.0</td>
      <td>21.0</td>
      <td>12.0</td>
      <td>11.0</td>
      <td>6.0</td>
      <td>6.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>2.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>2.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.03</td>
      <td>0.05</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Vikings vs. Panthers</td>
      <td>2017-12-10</td>
      <td>Panthers</td>
      <td>Vikings</td>
      <td>21.0</td>
      <td>15.0</td>
      <td>14.0</td>
      <td>7.0</td>
      <td>5.0</td>
      <td>6.0</td>
      <td>...</td>
      <td>3.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>2.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.03</td>
      <td>0.05</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Colts vs. Bills</td>
      <td>2017-12-10</td>
      <td>Bills</td>
      <td>Colts</td>
      <td>16.0</td>
      <td>14.0</td>
      <td>5.0</td>
      <td>4.0</td>
      <td>10.0</td>
      <td>10.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>2.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.05</td>
      <td>0.05</td>
    </tr>
    <tr>
      <th>143</th>
      <td>Patriots vs. Dolphins</td>
      <td>2017-12-11</td>
      <td>Dolphins</td>
      <td>Patriots</td>
      <td>14.0</td>
      <td>21.0</td>
      <td>9.0</td>
      <td>13.0</td>
      <td>2.0</td>
      <td>6.0</td>
      <td>...</td>
      <td>2.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>2.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.03</td>
      <td>0.05</td>
    </tr>
    <tr>
      <th>152</th>
      <td>Broncos vs. Colts</td>
      <td>2017-12-14</td>
      <td>Colts</td>
      <td>Broncos</td>
      <td>23.0</td>
      <td>15.0</td>
      <td>11.0</td>
      <td>9.0</td>
      <td>10.0</td>
      <td>4.0</td>
      <td>...</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.05</td>
      <td>0.03</td>
    </tr>
    <tr>
      <th>219</th>
      <td>Chargers vs. Chiefs</td>
      <td>2017-12-16</td>
      <td>Chiefs</td>
      <td>Chargers</td>
      <td>20.0</td>
      <td>20.0</td>
      <td>11.0</td>
      <td>7.0</td>
      <td>7.0</td>
      <td>9.0</td>
      <td>...</td>
      <td>4.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>3.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.03</td>
      <td>0.05</td>
    </tr>
    <tr>
      <th>215</th>
      <td>Bears vs. Lions</td>
      <td>2017-12-16</td>
      <td>Lions</td>
      <td>Bears</td>
      <td>22.0</td>
      <td>18.0</td>
      <td>16.0</td>
      <td>13.0</td>
      <td>4.0</td>
      <td>1.0</td>
      <td>...</td>
      <td>3.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>3.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.05</td>
      <td>0.03</td>
    </tr>
    <tr>
      <th>225</th>
      <td>Ravens vs. Browns</td>
      <td>2017-12-17</td>
      <td>Browns</td>
      <td>Ravens</td>
      <td>23.0</td>
      <td>15.0</td>
      <td>12.0</td>
      <td>8.0</td>
      <td>10.0</td>
      <td>5.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>4.0</td>
      <td>0.0</td>
      <td>2.0</td>
      <td>0.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>0.0</td>
      <td>0.05</td>
      <td>0.03</td>
    </tr>
    <tr>
      <th>243</th>
      <td>Texans vs. Jaguars</td>
      <td>2017-12-17</td>
      <td>Jaguars</td>
      <td>Texans</td>
      <td>9.0</td>
      <td>25.0</td>
      <td>6.0</td>
      <td>14.0</td>
      <td>3.0</td>
      <td>8.0</td>
      <td>...</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.03</td>
      <td>0.05</td>
    </tr>
    <tr>
      <th>228</th>
      <td>Texans vs. Jaguars</td>
      <td>2017-12-17</td>
      <td>Jaguars</td>
      <td>Texans</td>
      <td>9.0</td>
      <td>25.0</td>
      <td>6.0</td>
      <td>14.0</td>
      <td>3.0</td>
      <td>8.0</td>
      <td>...</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.03</td>
      <td>0.05</td>
    </tr>
    <tr>
      <th>221</th>
      <td>Dolphins vs. Bills</td>
      <td>2017-12-17</td>
      <td>Bills</td>
      <td>Dolphins</td>
      <td>21.0</td>
      <td>21.0</td>
      <td>13.0</td>
      <td>10.0</td>
      <td>7.0</td>
      <td>9.0</td>
      <td>...</td>
      <td>3.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>3.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.05</td>
      <td>0.03</td>
    </tr>
    <tr>
      <th>246</th>
      <td>Bengals vs. Vikings</td>
      <td>2017-12-17</td>
      <td>Vikings</td>
      <td>Bengals</td>
      <td>8.0</td>
      <td>18.0</td>
      <td>4.0</td>
      <td>8.0</td>
      <td>2.0</td>
      <td>7.0</td>
      <td>...</td>
      <td>2.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>2.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>2.0</td>
      <td>0.03</td>
      <td>0.05</td>
    </tr>
    <tr>
      <th>253</th>
      <td>Jets vs. Saints</td>
      <td>2017-12-17</td>
      <td>Saints</td>
      <td>Jets</td>
      <td>19.0</td>
      <td>23.0</td>
      <td>7.0</td>
      <td>13.0</td>
      <td>9.0</td>
      <td>7.0</td>
      <td>...</td>
      <td>2.0</td>
      <td>3.0</td>
      <td>0.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.05</td>
      <td>0.03</td>
    </tr>
    <tr>
      <th>224</th>
      <td>Packers vs. Panthers</td>
      <td>2017-12-17</td>
      <td>Panthers</td>
      <td>Packers</td>
      <td>24.0</td>
      <td>29.0</td>
      <td>13.0</td>
      <td>15.0</td>
      <td>9.0</td>
      <td>11.0</td>
      <td>...</td>
      <td>4.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>3.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.03</td>
      <td>0.05</td>
    </tr>
    <tr>
      <th>231</th>
      <td>Bengals vs. Vikings</td>
      <td>2017-12-17</td>
      <td>Vikings</td>
      <td>Bengals</td>
      <td>8.0</td>
      <td>18.0</td>
      <td>4.0</td>
      <td>8.0</td>
      <td>2.0</td>
      <td>7.0</td>
      <td>...</td>
      <td>2.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>2.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>2.0</td>
      <td>0.03</td>
      <td>0.05</td>
    </tr>
    <tr>
      <th>256</th>
      <td>Cardinals vs. Redskins</td>
      <td>2017-12-17</td>
      <td>Redskins</td>
      <td>Cardinals</td>
      <td>19.0</td>
      <td>14.0</td>
      <td>8.0</td>
      <td>10.0</td>
      <td>8.0</td>
      <td>1.0</td>
      <td>...</td>
      <td>2.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.05</td>
      <td>0.03</td>
    </tr>
    <tr>
      <th>255</th>
      <td>Eagles vs. Giants</td>
      <td>2017-12-17</td>
      <td>Giants</td>
      <td>Eagles</td>
      <td>22.0</td>
      <td>27.0</td>
      <td>15.0</td>
      <td>18.0</td>
      <td>5.0</td>
      <td>4.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.05</td>
      <td>0.03</td>
    </tr>
    <tr>
      <th>49</th>
      <td>Vikings vs. Packers</td>
      <td>2017-12-23</td>
      <td>Packers</td>
      <td>Vikings</td>
      <td>15.0</td>
      <td>12.0</td>
      <td>9.0</td>
      <td>6.0</td>
      <td>5.0</td>
      <td>5.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>2.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>2.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.05</td>
      <td>0.03</td>
    </tr>
    <tr>
      <th>43</th>
      <td>Colts vs. Ravens</td>
      <td>2017-12-23</td>
      <td>Ravens</td>
      <td>Colts</td>
      <td>17.0</td>
      <td>23.0</td>
      <td>12.0</td>
      <td>13.0</td>
      <td>5.0</td>
      <td>8.0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.03</td>
      <td>0.05</td>
    </tr>
    <tr>
      <th>118</th>
      <td>Broncos vs. Redskins</td>
      <td>2017-12-24</td>
      <td>Redskins</td>
      <td>Broncos</td>
      <td>18.0</td>
      <td>19.0</td>
      <td>7.0</td>
      <td>12.0</td>
      <td>9.0</td>
      <td>7.0</td>
      <td>...</td>
      <td>3.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.05</td>
      <td>0.03</td>
    </tr>
    <tr>
      <th>129</th>
      <td>Giants vs. Cardinals</td>
      <td>2017-12-24</td>
      <td>Cardinals</td>
      <td>Giants</td>
      <td>12.0</td>
      <td>19.0</td>
      <td>12.0</td>
      <td>13.0</td>
      <td>0.0</td>
      <td>4.0</td>
      <td>...</td>
      <td>3.0</td>
      <td>2.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>0.0</td>
      <td>2.0</td>
      <td>0.03</td>
      <td>0.05</td>
    </tr>
  </tbody>
</table>
<p>227 rows × 54 columns</p>
</div>




```python


def get_game_scores(game_id):
    score = []
    web_address = 'http://www.espn.com/nfl/matchup?gameId=' + str(game_id)
    print(game_id, web_address)
    game_webpage = url.urlopen(web_address)
    orig_soup = BeautifulSoup(game_webpage, 'html.parser')
    game_info = orig_soup.title.string
    game_info_split = game_info.split('-')
    teams = game_info_split[0]
    date = game_info_split[2]
    if 'August' not in date:
        soup = orig_soup.find_all('td', 'final-score')
        home_score = (int(soup[1].contents[0]))
        away_score = (int(soup[0].contents[0]))
        score.append(home_score)
        score.append(away_score)
    return score
        

```


```python
game_id = 400951552
num_games = 0
game_results = []

while  game_id < 400951840:
    if game_id == 400951581:
        game_id += 1        #for Dolphins Vs. Bucs 
    result = get_game_scores(game_id)
    if result != []:
        game_results.append(result)
        num_games += 1
        print(num_games)
    game_id += 1
    #else:
        #missed_list.append(missed)
        #game_id += 1
    

print(game_results)

```

    400951552 http://www.espn.com/nfl/matchup?gameId=400951552
    1
    400951553 http://www.espn.com/nfl/matchup?gameId=400951553
    2
    400951554 http://www.espn.com/nfl/matchup?gameId=400951554
    3
    400951555 http://www.espn.com/nfl/matchup?gameId=400951555
    4
    400951556 http://www.espn.com/nfl/matchup?gameId=400951556
    5
    400951557 http://www.espn.com/nfl/matchup?gameId=400951557
    6
    400951558 http://www.espn.com/nfl/matchup?gameId=400951558
    7
    400951559 http://www.espn.com/nfl/matchup?gameId=400951559
    8
    400951560 http://www.espn.com/nfl/matchup?gameId=400951560
    9
    400951561 http://www.espn.com/nfl/matchup?gameId=400951561
    10
    400951562 http://www.espn.com/nfl/matchup?gameId=400951562
    11
    400951563 http://www.espn.com/nfl/matchup?gameId=400951563
    12
    400951564 http://www.espn.com/nfl/matchup?gameId=400951564
    13
    400951565 http://www.espn.com/nfl/matchup?gameId=400951565
    14
    400951566 http://www.espn.com/nfl/matchup?gameId=400951566
    15
    400951567 http://www.espn.com/nfl/matchup?gameId=400951567
    16
    400951568 http://www.espn.com/nfl/matchup?gameId=400951568
    17
    400951569 http://www.espn.com/nfl/matchup?gameId=400951569
    18
    400951570 http://www.espn.com/nfl/matchup?gameId=400951570
    19
    400951571 http://www.espn.com/nfl/matchup?gameId=400951571
    20
    400951572 http://www.espn.com/nfl/matchup?gameId=400951572
    21
    400951573 http://www.espn.com/nfl/matchup?gameId=400951573
    22
    400951574 http://www.espn.com/nfl/matchup?gameId=400951574
    23
    400951575 http://www.espn.com/nfl/matchup?gameId=400951575
    24
    400951576 http://www.espn.com/nfl/matchup?gameId=400951576
    25
    400951577 http://www.espn.com/nfl/matchup?gameId=400951577
    26
    400951578 http://www.espn.com/nfl/matchup?gameId=400951578
    27
    400951579 http://www.espn.com/nfl/matchup?gameId=400951579
    28
    400951580 http://www.espn.com/nfl/matchup?gameId=400951580
    29
    400951582 http://www.espn.com/nfl/matchup?gameId=400951582
    30
    400951583 http://www.espn.com/nfl/matchup?gameId=400951583
    31
    400951584 http://www.espn.com/nfl/matchup?gameId=400951584
    32
    400951585 http://www.espn.com/nfl/matchup?gameId=400951585
    33
    400951586 http://www.espn.com/nfl/matchup?gameId=400951586
    34
    400951587 http://www.espn.com/nfl/matchup?gameId=400951587
    35
    400951588 http://www.espn.com/nfl/matchup?gameId=400951588
    36
    400951589 http://www.espn.com/nfl/matchup?gameId=400951589
    37
    400951590 http://www.espn.com/nfl/matchup?gameId=400951590
    38
    400951591 http://www.espn.com/nfl/matchup?gameId=400951591
    39
    400951592 http://www.espn.com/nfl/matchup?gameId=400951592
    40
    400951593 http://www.espn.com/nfl/matchup?gameId=400951593
    41
    400951594 http://www.espn.com/nfl/matchup?gameId=400951594
    42
    400951595 http://www.espn.com/nfl/matchup?gameId=400951595
    43
    400951596 http://www.espn.com/nfl/matchup?gameId=400951596
    44
    400951597 http://www.espn.com/nfl/matchup?gameId=400951597
    45
    400951598 http://www.espn.com/nfl/matchup?gameId=400951598
    46
    400951599 http://www.espn.com/nfl/matchup?gameId=400951599
    47
    400951600 http://www.espn.com/nfl/matchup?gameId=400951600
    48
    400951601 http://www.espn.com/nfl/matchup?gameId=400951601
    49
    400951602 http://www.espn.com/nfl/matchup?gameId=400951602
    50
    400951603 http://www.espn.com/nfl/matchup?gameId=400951603
    51
    400951604 http://www.espn.com/nfl/matchup?gameId=400951604
    52
    400951605 http://www.espn.com/nfl/matchup?gameId=400951605
    53
    400951606 http://www.espn.com/nfl/matchup?gameId=400951606
    54
    400951607 http://www.espn.com/nfl/matchup?gameId=400951607
    55
    400951608 http://www.espn.com/nfl/matchup?gameId=400951608
    56
    400951609 http://www.espn.com/nfl/matchup?gameId=400951609
    57
    400951610 http://www.espn.com/nfl/matchup?gameId=400951610
    58
    400951611 http://www.espn.com/nfl/matchup?gameId=400951611
    59
    400951612 http://www.espn.com/nfl/matchup?gameId=400951612
    60
    400951613 http://www.espn.com/nfl/matchup?gameId=400951613
    61
    400951614 http://www.espn.com/nfl/matchup?gameId=400951614
    62
    400951615 http://www.espn.com/nfl/matchup?gameId=400951615
    63
    400951616 http://www.espn.com/nfl/matchup?gameId=400951616
    64
    400951617 http://www.espn.com/nfl/matchup?gameId=400951617
    65
    400951618 http://www.espn.com/nfl/matchup?gameId=400951618
    66
    400951619 http://www.espn.com/nfl/matchup?gameId=400951619
    67
    400951620 http://www.espn.com/nfl/matchup?gameId=400951620
    68
    400951621 http://www.espn.com/nfl/matchup?gameId=400951621
    69
    400951622 http://www.espn.com/nfl/matchup?gameId=400951622
    70
    400951623 http://www.espn.com/nfl/matchup?gameId=400951623
    71
    400951624 http://www.espn.com/nfl/matchup?gameId=400951624
    72
    400951625 http://www.espn.com/nfl/matchup?gameId=400951625
    73
    400951626 http://www.espn.com/nfl/matchup?gameId=400951626
    74
    400951627 http://www.espn.com/nfl/matchup?gameId=400951627
    75
    400951628 http://www.espn.com/nfl/matchup?gameId=400951628
    76
    400951629 http://www.espn.com/nfl/matchup?gameId=400951629
    77
    400951630 http://www.espn.com/nfl/matchup?gameId=400951630
    78
    400951631 http://www.espn.com/nfl/matchup?gameId=400951631
    79
    400951632 http://www.espn.com/nfl/matchup?gameId=400951632
    80
    400951633 http://www.espn.com/nfl/matchup?gameId=400951633
    81
    400951634 http://www.espn.com/nfl/matchup?gameId=400951634
    82
    400951635 http://www.espn.com/nfl/matchup?gameId=400951635
    83
    400951636 http://www.espn.com/nfl/matchup?gameId=400951636
    84
    400951637 http://www.espn.com/nfl/matchup?gameId=400951637
    85
    400951638 http://www.espn.com/nfl/matchup?gameId=400951638
    86
    400951639 http://www.espn.com/nfl/matchup?gameId=400951639
    87
    400951640 http://www.espn.com/nfl/matchup?gameId=400951640
    88
    400951641 http://www.espn.com/nfl/matchup?gameId=400951641
    89
    400951642 http://www.espn.com/nfl/matchup?gameId=400951642
    90
    400951643 http://www.espn.com/nfl/matchup?gameId=400951643
    91
    400951644 http://www.espn.com/nfl/matchup?gameId=400951644
    92
    400951645 http://www.espn.com/nfl/matchup?gameId=400951645
    93
    400951646 http://www.espn.com/nfl/matchup?gameId=400951646
    94
    400951647 http://www.espn.com/nfl/matchup?gameId=400951647
    95
    400951648 http://www.espn.com/nfl/matchup?gameId=400951648
    96
    400951649 http://www.espn.com/nfl/matchup?gameId=400951649
    97
    400951650 http://www.espn.com/nfl/matchup?gameId=400951650
    98
    400951651 http://www.espn.com/nfl/matchup?gameId=400951651
    99
    400951652 http://www.espn.com/nfl/matchup?gameId=400951652
    100
    400951653 http://www.espn.com/nfl/matchup?gameId=400951653
    101
    400951654 http://www.espn.com/nfl/matchup?gameId=400951654
    102
    400951655 http://www.espn.com/nfl/matchup?gameId=400951655
    103
    400951656 http://www.espn.com/nfl/matchup?gameId=400951656
    104
    400951657 http://www.espn.com/nfl/matchup?gameId=400951657
    105
    400951658 http://www.espn.com/nfl/matchup?gameId=400951658
    106
    400951659 http://www.espn.com/nfl/matchup?gameId=400951659
    107
    400951660 http://www.espn.com/nfl/matchup?gameId=400951660
    108
    400951661 http://www.espn.com/nfl/matchup?gameId=400951661
    109
    400951662 http://www.espn.com/nfl/matchup?gameId=400951662
    110
    400951663 http://www.espn.com/nfl/matchup?gameId=400951663
    111
    400951664 http://www.espn.com/nfl/matchup?gameId=400951664
    112
    400951665 http://www.espn.com/nfl/matchup?gameId=400951665
    113
    400951666 http://www.espn.com/nfl/matchup?gameId=400951666
    114
    400951667 http://www.espn.com/nfl/matchup?gameId=400951667
    115
    400951668 http://www.espn.com/nfl/matchup?gameId=400951668
    116
    400951669 http://www.espn.com/nfl/matchup?gameId=400951669
    117
    400951670 http://www.espn.com/nfl/matchup?gameId=400951670
    118
    400951671 http://www.espn.com/nfl/matchup?gameId=400951671
    119
    400951672 http://www.espn.com/nfl/matchup?gameId=400951672
    120
    400951673 http://www.espn.com/nfl/matchup?gameId=400951673
    121
    400951674 http://www.espn.com/nfl/matchup?gameId=400951674
    122
    400951675 http://www.espn.com/nfl/matchup?gameId=400951675
    123
    400951676 http://www.espn.com/nfl/matchup?gameId=400951676
    124
    400951677 http://www.espn.com/nfl/matchup?gameId=400951677
    125
    400951678 http://www.espn.com/nfl/matchup?gameId=400951678
    126
    400951679 http://www.espn.com/nfl/matchup?gameId=400951679
    127
    400951680 http://www.espn.com/nfl/matchup?gameId=400951680
    128
    400951681 http://www.espn.com/nfl/matchup?gameId=400951681
    129
    400951682 http://www.espn.com/nfl/matchup?gameId=400951682
    130
    400951683 http://www.espn.com/nfl/matchup?gameId=400951683
    131
    400951684 http://www.espn.com/nfl/matchup?gameId=400951684
    132
    400951685 http://www.espn.com/nfl/matchup?gameId=400951685
    133
    400951686 http://www.espn.com/nfl/matchup?gameId=400951686
    134
    400951687 http://www.espn.com/nfl/matchup?gameId=400951687
    135
    400951688 http://www.espn.com/nfl/matchup?gameId=400951688
    136
    400951689 http://www.espn.com/nfl/matchup?gameId=400951689
    137
    400951690 http://www.espn.com/nfl/matchup?gameId=400951690
    138
    400951691 http://www.espn.com/nfl/matchup?gameId=400951691
    139
    400951692 http://www.espn.com/nfl/matchup?gameId=400951692
    140
    400951693 http://www.espn.com/nfl/matchup?gameId=400951693
    141
    400951694 http://www.espn.com/nfl/matchup?gameId=400951694
    142
    400951695 http://www.espn.com/nfl/matchup?gameId=400951695
    143
    400951696 http://www.espn.com/nfl/matchup?gameId=400951696
    144
    400951697 http://www.espn.com/nfl/matchup?gameId=400951697
    145
    400951698 http://www.espn.com/nfl/matchup?gameId=400951698
    146
    400951699 http://www.espn.com/nfl/matchup?gameId=400951699
    147
    400951700 http://www.espn.com/nfl/matchup?gameId=400951700
    148
    400951701 http://www.espn.com/nfl/matchup?gameId=400951701
    149
    400951702 http://www.espn.com/nfl/matchup?gameId=400951702
    150
    400951703 http://www.espn.com/nfl/matchup?gameId=400951703
    151
    400951704 http://www.espn.com/nfl/matchup?gameId=400951704
    152
    400951705 http://www.espn.com/nfl/matchup?gameId=400951705
    153
    400951706 http://www.espn.com/nfl/matchup?gameId=400951706
    154
    400951707 http://www.espn.com/nfl/matchup?gameId=400951707
    155
    400951708 http://www.espn.com/nfl/matchup?gameId=400951708
    156
    400951709 http://www.espn.com/nfl/matchup?gameId=400951709
    157
    400951710 http://www.espn.com/nfl/matchup?gameId=400951710
    158
    400951711 http://www.espn.com/nfl/matchup?gameId=400951711
    159
    400951712 http://www.espn.com/nfl/matchup?gameId=400951712
    160
    400951713 http://www.espn.com/nfl/matchup?gameId=400951713
    161
    400951714 http://www.espn.com/nfl/matchup?gameId=400951714
    162
    400951715 http://www.espn.com/nfl/matchup?gameId=400951715
    163
    400951716 http://www.espn.com/nfl/matchup?gameId=400951716
    164
    400951717 http://www.espn.com/nfl/matchup?gameId=400951717
    165
    400951718 http://www.espn.com/nfl/matchup?gameId=400951718
    166
    400951719 http://www.espn.com/nfl/matchup?gameId=400951719
    167
    400951720 http://www.espn.com/nfl/matchup?gameId=400951720
    168
    400951721 http://www.espn.com/nfl/matchup?gameId=400951721
    169
    400951722 http://www.espn.com/nfl/matchup?gameId=400951722
    170
    400951723 http://www.espn.com/nfl/matchup?gameId=400951723
    171
    400951724 http://www.espn.com/nfl/matchup?gameId=400951724
    172
    400951725 http://www.espn.com/nfl/matchup?gameId=400951725
    173
    400951726 http://www.espn.com/nfl/matchup?gameId=400951726
    174
    400951727 http://www.espn.com/nfl/matchup?gameId=400951727
    175
    400951728 http://www.espn.com/nfl/matchup?gameId=400951728
    176
    400951729 http://www.espn.com/nfl/matchup?gameId=400951729
    177
    400951730 http://www.espn.com/nfl/matchup?gameId=400951730
    178
    400951731 http://www.espn.com/nfl/matchup?gameId=400951731
    179
    400951732 http://www.espn.com/nfl/matchup?gameId=400951732
    180
    400951733 http://www.espn.com/nfl/matchup?gameId=400951733
    181
    400951734 http://www.espn.com/nfl/matchup?gameId=400951734
    182
    400951735 http://www.espn.com/nfl/matchup?gameId=400951735
    183
    400951736 http://www.espn.com/nfl/matchup?gameId=400951736
    184
    400951737 http://www.espn.com/nfl/matchup?gameId=400951737
    185
    400951738 http://www.espn.com/nfl/matchup?gameId=400951738
    186
    400951739 http://www.espn.com/nfl/matchup?gameId=400951739
    187
    400951740 http://www.espn.com/nfl/matchup?gameId=400951740
    188
    400951741 http://www.espn.com/nfl/matchup?gameId=400951741
    189
    400951742 http://www.espn.com/nfl/matchup?gameId=400951742
    190
    400951743 http://www.espn.com/nfl/matchup?gameId=400951743
    191
    400951744 http://www.espn.com/nfl/matchup?gameId=400951744
    192
    400951745 http://www.espn.com/nfl/matchup?gameId=400951745
    193
    400951746 http://www.espn.com/nfl/matchup?gameId=400951746
    194
    400951747 http://www.espn.com/nfl/matchup?gameId=400951747
    195
    400951748 http://www.espn.com/nfl/matchup?gameId=400951748
    196
    400951749 http://www.espn.com/nfl/matchup?gameId=400951749
    197
    400951750 http://www.espn.com/nfl/matchup?gameId=400951750
    198
    400951751 http://www.espn.com/nfl/matchup?gameId=400951751
    199
    400951752 http://www.espn.com/nfl/matchup?gameId=400951752
    200
    400951753 http://www.espn.com/nfl/matchup?gameId=400951753
    201
    400951754 http://www.espn.com/nfl/matchup?gameId=400951754
    202
    400951755 http://www.espn.com/nfl/matchup?gameId=400951755
    203
    400951756 http://www.espn.com/nfl/matchup?gameId=400951756
    204
    400951757 http://www.espn.com/nfl/matchup?gameId=400951757
    205
    400951758 http://www.espn.com/nfl/matchup?gameId=400951758
    206
    400951759 http://www.espn.com/nfl/matchup?gameId=400951759
    207
    400951760 http://www.espn.com/nfl/matchup?gameId=400951760
    208
    400951761 http://www.espn.com/nfl/matchup?gameId=400951761
    209
    400951762 http://www.espn.com/nfl/matchup?gameId=400951762
    210
    400951763 http://www.espn.com/nfl/matchup?gameId=400951763
    211
    400951764 http://www.espn.com/nfl/matchup?gameId=400951764
    212
    400951765 http://www.espn.com/nfl/matchup?gameId=400951765
    213
    400951766 http://www.espn.com/nfl/matchup?gameId=400951766
    214
    400951767 http://www.espn.com/nfl/matchup?gameId=400951767
    215
    400951768 http://www.espn.com/nfl/matchup?gameId=400951768
    216
    400951769 http://www.espn.com/nfl/matchup?gameId=400951769
    217
    400951770 http://www.espn.com/nfl/matchup?gameId=400951770
    218
    400951771 http://www.espn.com/nfl/matchup?gameId=400951771
    219
    400951772 http://www.espn.com/nfl/matchup?gameId=400951772
    220
    400951773 http://www.espn.com/nfl/matchup?gameId=400951773
    221
    400951774 http://www.espn.com/nfl/matchup?gameId=400951774
    222
    400951775 http://www.espn.com/nfl/matchup?gameId=400951775
    223
    400951776 http://www.espn.com/nfl/matchup?gameId=400951776
    224
    400951777 http://www.espn.com/nfl/matchup?gameId=400951777
    225
    400951778 http://www.espn.com/nfl/matchup?gameId=400951778
    226
    400951779 http://www.espn.com/nfl/matchup?gameId=400951779
    227
    400951780 http://www.espn.com/nfl/matchup?gameId=400951780
    228
    400951781 http://www.espn.com/nfl/matchup?gameId=400951781
    229
    400951782 http://www.espn.com/nfl/matchup?gameId=400951782
    230
    400951783 http://www.espn.com/nfl/matchup?gameId=400951783
    231
    400951784 http://www.espn.com/nfl/matchup?gameId=400951784
    232
    400951785 http://www.espn.com/nfl/matchup?gameId=400951785
    233
    400951786 http://www.espn.com/nfl/matchup?gameId=400951786
    234
    400951787 http://www.espn.com/nfl/matchup?gameId=400951787
    235
    400951788 http://www.espn.com/nfl/matchup?gameId=400951788
    236
    400951789 http://www.espn.com/nfl/matchup?gameId=400951789
    237
    400951790 http://www.espn.com/nfl/matchup?gameId=400951790
    238
    400951791 http://www.espn.com/nfl/matchup?gameId=400951791
    239
    400951792 http://www.espn.com/nfl/matchup?gameId=400951792
    240
    400951793 http://www.espn.com/nfl/matchup?gameId=400951793
    241
    400951794 http://www.espn.com/nfl/matchup?gameId=400951794
    400951795 http://www.espn.com/nfl/matchup?gameId=400951795
    400951796 http://www.espn.com/nfl/matchup?gameId=400951796
    400951797 http://www.espn.com/nfl/matchup?gameId=400951797
    400951798 http://www.espn.com/nfl/matchup?gameId=400951798
    400951799 http://www.espn.com/nfl/matchup?gameId=400951799
    400951800 http://www.espn.com/nfl/matchup?gameId=400951800
    400951801 http://www.espn.com/nfl/matchup?gameId=400951801
    400951802 http://www.espn.com/nfl/matchup?gameId=400951802
    400951803 http://www.espn.com/nfl/matchup?gameId=400951803
    400951804 http://www.espn.com/nfl/matchup?gameId=400951804
    400951805 http://www.espn.com/nfl/matchup?gameId=400951805
    400951806 http://www.espn.com/nfl/matchup?gameId=400951806
    242
    400951807 http://www.espn.com/nfl/matchup?gameId=400951807
    243
    400951808 http://www.espn.com/nfl/matchup?gameId=400951808
    244
    400951809 http://www.espn.com/nfl/matchup?gameId=400951809
    245
    400951810 http://www.espn.com/nfl/matchup?gameId=400951810
    246
    400951811 http://www.espn.com/nfl/matchup?gameId=400951811
    247
    400951812 http://www.espn.com/nfl/matchup?gameId=400951812
    248
    400951813 http://www.espn.com/nfl/matchup?gameId=400951813
    249
    400951814 http://www.espn.com/nfl/matchup?gameId=400951814
    250
    400951815 http://www.espn.com/nfl/matchup?gameId=400951815
    251
    400951816 http://www.espn.com/nfl/matchup?gameId=400951816
    252
    400951817 http://www.espn.com/nfl/matchup?gameId=400951817
    253
    400951818 http://www.espn.com/nfl/matchup?gameId=400951818
    254
    400951819 http://www.espn.com/nfl/matchup?gameId=400951819
    400951820 http://www.espn.com/nfl/matchup?gameId=400951820
    400951821 http://www.espn.com/nfl/matchup?gameId=400951821
    400951822 http://www.espn.com/nfl/matchup?gameId=400951822
    400951823 http://www.espn.com/nfl/matchup?gameId=400951823
    400951824 http://www.espn.com/nfl/matchup?gameId=400951824
    400951825 http://www.espn.com/nfl/matchup?gameId=400951825
    400951826 http://www.espn.com/nfl/matchup?gameId=400951826
    400951827 http://www.espn.com/nfl/matchup?gameId=400951827
    400951828 http://www.espn.com/nfl/matchup?gameId=400951828
    400951829 http://www.espn.com/nfl/matchup?gameId=400951829
    400951830 http://www.espn.com/nfl/matchup?gameId=400951830
    400951831 http://www.espn.com/nfl/matchup?gameId=400951831
    400951832 http://www.espn.com/nfl/matchup?gameId=400951832
    400951833 http://www.espn.com/nfl/matchup?gameId=400951833
    400951834 http://www.espn.com/nfl/matchup?gameId=400951834
    400951835 http://www.espn.com/nfl/matchup?gameId=400951835
    400951836 http://www.espn.com/nfl/matchup?gameId=400951836
    400951837 http://www.espn.com/nfl/matchup?gameId=400951837
    400951838 http://www.espn.com/nfl/matchup?gameId=400951838
    400951839 http://www.espn.com/nfl/matchup?gameId=400951839
    [[14, 19], [16, 22], [20, 16], [10, 47], [14, 17], [20, 17], [24, 27], [16, 23], [13, 7], [31, 24], [26, 23], [38, 24], [7, 33], [17, 20], [27, 42], [21, 12], [39, 41], [23, 30], [17, 23], [31, 30], [0, 20], [6, 28], [18, 21], [30, 27], [35, 23], [20, 10], [17, 3], [44, 7], [7, 29], [9, 12], [26, 16], [16, 26], [17, 26], [34, 20], [13, 34], [30, 16], [0, 27], [23, 17], [16, 20], [17, 30], [33, 0], [26, 30], [10, 16], [23, 16], [46, 9], [31, 28], [31, 28], [35, 17], [17, 9], [0, 16], [24, 16], [34, 17], [3, 23], [27, 35], [36, 33], [19, 3], [29, 14], [31, 3], [20, 6], [29, 19], [22, 19], [26, 20], [24, 21], [27, 24], [20, 3], [13, 24], [10, 40], [9, 13], [26, 17], [27, 24], [33, 27], [21, 0], [29, 13], [24, 10], [27, 24], [7, 24], [21, 14], [9, 3], [37, 16], [13, 16], [31, 28], [10, 24], [16, 37], [27, 20], [23, 13], [23, 7], [20, 36], [23, 16], [34, 24], [7, 14], [26, 9], [27, 10], [29, 7], [16, 10], [21, 27], [20, 17], [16, 26], [22, 27], [34, 7], [30, 24], [15, 10], [26, 15], [9, 30], [24, 20], [10, 16], [30, 38], [17, 30], [21, 24], [31, 35], [12, 7], [33, 7], [34, 42], [23, 0], [17, 19], [23, 27], [17, 28], [45, 20], [40, 0], [27, 11], [38, 14], [42, 17], [20, 27], [44, 33], [12, 9], [9, 14], [35, 14], [34, 23], [44, 20], [10, 24], [23, 0], [16, 33], [3, 23], [17, 23], [27, 7], [30, 13], [31, 21], [35, 43], [16, 41], [17, 20], [10, 30], [45, 21], [39, 38], [23, 28], [27, 20], [17, 20], [40, 17], [24, 27], [33, 17], [24, 27], [23, 10], [0, 23], [52, 38], [13, 25], [34, 14], [14, 15], [9, 26], [12, 21], [26, 20], [24, 23], [7, 31], [30, 10], [6, 34], [21, 13], [30, 35], [20, 12], [35, 9], [19, 10], [57, 14], [20, 25], [31, 21], [33, 10], [7, 14], [3, 17], [38, 31], [30, 33], [24, 13], [41, 38], [23, 20], [19, 10], [19, 33], [16, 32], [22, 10], [18, 15], [15, 20], [29, 19], [24, 26], [27, 31], [24, 17], [35, 11], [25, 23], [34, 21], [16, 10], [22, 13], [24, 10], [46, 18], [16, 22], [20, 17], [23, 10], [14, 20], [29, 20], [23, 7], [20, 23], [30, 10], [26, 6], [18, 10], [17, 51], [0, 6], [51, 23], [23, 20], [28, 24], [10, 20], [31, 24], [14, 17], [17, 24], [26, 24], [20, 10], [7, 19], [38, 33], [31, 21], [30, 13], [17, 27], [24, 16], [24, 7], [13, 19], [31, 24], [10, 27], [16, 17], [34, 31], [45, 7], [10, 23], [12, 9], [34, 7], [36, 22], [28, 17], [24, 27], [15, 10], [24, 27], [17, 30], [30, 10], [13, 34], [24, 26], [31, 19], [54, 24], [29, 34], [20, 15], [17, 20], [7, 42], [24, 27], [25, 23], [17, 20], [8, 33], [21, 24], [9, 37], [31, 34]]
    


```python
get_game_scores(400951806)
```

    400951806 http://www.espn.com/nfl/matchup?gameId=400951806
    




    [31, 19]


