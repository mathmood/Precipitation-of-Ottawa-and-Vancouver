
# Comparison of Precipitation for Ottawa and Vancouver

Is precipitation in Vancouver area is significantly more than Ottawa?

It is generally believed in Canada that Vancouver is significantly darker than other major Canadian cities due to the heavy amount of rainfall. To shed light on this, I collected two precipitation datasets from the Canadian government and compared rainfall and snowfall of Vancouver to the ones of Ottawa (the capital and my current hometown)

The problem in this comparison was that, Ottawa due to its location has a significantly colder weather and therefore, there is more snow than rainfall during colder months (unlike Vancouver). To address this issue, I presented data in three graphs. The top plot presents the monthly average difference of rainfall in two cities Vancouver and Ottawa and the lower presents the monthly average difference of snowfall. Eventually, I used two appropriate 'snow ratios' for Ottawa and Vancouver (considering their average temperature) to present the precipitation of the both cities on the middle graph. As you can see, having these three graphs, we see that in fact Vancouver has more Precipitation.

References: 
https://www.accuweather.com/en/weather-news/what-are-snow-ratios/4786333 
https://en.wikipedia.org/wiki/Precipitation

Unfortunately I cannot provide direct links to the datasets that I used. But the datasets are reachable at: http://climate.weather.gc.ca/historical_data/search_historic_data_e.html Enter "OTTAWA CDA" for the Ottawa dataset and "N VANCOUVER WHARVES" for the Vancouver dataset for the "Name" of the weather station. (There are only one dataset with these specific names).

The provided presentation is made via matlibplot. We start by importing the needed packages.

```
import matplotlib.pyplot as plt
import mplleaflet
import pandas as pd
import numpy as np
import math
import matplotlib.gridspec as gridspec
```

The snow to liquid ratio is a way of describing what makes light, fluffy snow different from heavy, wet snow. The temperature can have a huge impact on the amount of snow as well as how much it weighs. The snow ratio compares the amount of liquid precipitation with the number of inches of snow. In colder weather, snow has more air space so there are more inches of snow. Wet snow that falls at the freezing mark is usually sloppy and heavy. Based on some search I decided to use the following different constants (based on average cold temperature for winter days of each city).

```
p_ottawa = 0.25
p_vancouver = 0.15
```

First let us import our data sets and save them as pandas dataframes. Then we drop the unnecessary columns and rename the required columns so we can reach them easier in the following. The used datasets are uploaded on my dropbox folder as:

https://www.dropbox.com/s/r9s8n5htw0ykudi/Ottawa_1889-2006.csv?dl=0 
https://www.dropbox.com/s/9hjqn64gdas249o/Vancouver_1962-2006.csv?dl=0

``` 
data_ottawa = pd.read_csv('Ottawa_1889-2006.csv', error_bad_lines=False)
data_vancouver = pd.read_csv('Vancouver_1962-2006.csv', error_bad_lines=False)

df_ottawa = pd.DataFrame(data_ottawa)
df_vancouver = pd.DataFrame(data_vancouver)

df_ottawa.set_index("Date/Time")
df_vancouver.set_index("Date/Time")

df_ottawa.drop(["Year","Month","Mean Max Temp (Â°C)","Mean Max Temp Flag","Mean Min Temp (Â°C)","Mean Min Temp Flag",'Mean Temp (Â°C)',"Mean Temp Flag","Extr Max Temp (Â°C)","Extr Max Temp Flag","Extr Min Temp (Â°C)","Extr Min Temp Flag","Total Rain Flag","Total Snow Flag","Total Precip (mm)","Total Precip Flag","Snow Grnd Last Day (cm)","Snow Grnd Last Day Flag","Dir of Max Gust (10's deg)","Dir of Max Gust Flag","Spd of Max Gust (km/h)","Spd of Max Gust Flag"], axis = 1, inplace=True)
df_vancouver.drop(["Year","Month","Mean Max Temp (Â°C)","Mean Max Temp Flag","Mean Min Temp (Â°C)","Mean Min Temp Flag",'Mean Temp (Â°C)',"Mean Temp Flag","Extr Max Temp (Â°C)","Extr Max Temp Flag","Extr Min Temp (Â°C)","Extr Min Temp Flag","Total Rain Flag","Total Snow Flag","Total Precip (mm)","Total Precip Flag","Snow Grnd Last Day (cm)","Snow Grnd Last Day Flag","Dir of Max Gust (10's deg)","Dir of Max Gust Flag","Spd of Max Gust (km/h)","Spd of Max Gust Flag"],axis = 1, inplace=True)

df_ottawa.columns = ['Date', 'Rain (mm) Ottawa','Snow (cm) Ottawa']
df_ottawa.set_index('Date', inplace=True)
df_vancouver.columns = ['Date', 'Rain (mm) Vancouver','Snow (cm) Vancouver']
df_vancouver.set_index('Date', inplace=True)
```

The following we transfer the amount of precipitation to integers and therefore, we convert the amount of snowfall to water so we can compare it with the rainfall. Note that here we need the snow to liquid ratios which we set above.

```
for i in df_ottawa.index:
    if type(df_ottawa.at[i, 'Rain (mm) Ottawa']) == str:
        df_ottawa.at[i, 'Rain (mm) Ottawa'] = int(df_ottawa.at[i, 'Rain (mm) Ottawa'])
    if type(df_ottawa.at[i,'Snow (cm) Ottawa']) == str:
        df_ottawa.at[i, 'Snow (cm) Ottawa'] = int(df_ottawa.at[i, 'Snow (cm) Ottawa'])

sum_var = df_ottawa['Rain (mm) Ottawa'] + 10 * df_ottawa['Snow (cm) Ottawa']* p_ottawa
df_ottawa['darkness Ottawa'] = sum_var

for i in df_vancouver.index:
    if type(df_vancouver.at[i,'Rain (mm) Vancouver']) == str:
        df_vancouver.at[i, 'Rain (mm) Vancouver'] = int(df_vancouver.at[i, 'Rain (mm) Vancouver'])
    if type(df_vancouver.at[i,'Snow (cm) Vancouver']) == str:
        df_vancouver.at[i, 'Snow (cm) Vancouver'] = int(df_vancouver.at[i, 'Snow (cm) Vancouver'])

sum_var = df_vancouver['Rain (mm) Vancouver'] + 10 * df_vancouver['Snow (cm) Vancouver'] * p_vancouver
df_vancouver['darkness Vancouver'] = sum_var
```

Now we need to cancate our data frames together and clean up the outcome and fix the indexing.

```
df = pd.concat([df_ottawa, df_vancouver], axis = 1, join = 'inner')
df = df.dropna()

df['Index'] = np.NaN

for x in df.index:
    df.at[x,'Index'] = int(x[:4]) + (int(x[5:7]) / 100)
```

Now we need to calculate the average amount of rain and snow for each year. Doing so, we are carefully watching the missing data (there are months their imput is NaN).

```
vancouver_darkness = []
ottawa_darkness = []
rainfall = []
snowfall = []

for i in range(1962, 2007):
    ave_ottawa = 0
    ave_vancouver = 0
    count = 0
    rain = 0
    snow = 0
    for x in df.index:
        if math.floor(df.at[x,'Index']) == i:
            ave_ottawa = ave_ottawa + df.at[x, 'darkness Ottawa']
            ave_vancouver = ave_vancouver + df.at[x, 'darkness Vancouver']
            rain = df.at[x,'Rain (mm) Vancouver'] - df.at[x,'Rain (mm) Ottawa']
            snow =10 * p_ottawa * df.at[x,'Snow (cm) Vancouver'] - 10 * p_vancouver * df.at[x,'Snow (cm) Ottawa']
            count = count + 1
    vancouver_darkness.append(ave_vancouver / count)
    ottawa_darkness.append(ave_ottawa / count)
    snowfall.append(snow / count)
    rainfall.append(rain/count)
```

Before plotting the data, we save a copy of our cleaned data fram in a csv file.

```
df_plt = pd.DataFrame(ottawa_darkness, index = range(1962,2007), columns = ['Ottawa'])
df_plt['Vancouver'] = vancouver_darkness
df_plt['Rain'] = rainfall
df_plt['Snow'] = snowfall
df_plt.to_csv('df_plot.csv', encoding='utf-8', index=False)
```

Now we are ready for plotting. We carefully write our axis-labeling to improve the exposition of the graph.

``` 
fig = plt.figure()
gspec = gridspec.GridSpec(7, 3)

top_fig = plt.subplot(gspec[0:2, 0:3])
mid_fig = plt.subplot(gspec[2:5, : ])
lower_fig = plt.subplot(gspec[5:, 0:3])

years = range(1962,2007)
years_base = ['1962-06','1963-06','1964-06','1965-06','1966-06','1967-06','1968-06','1969-06','1970-06','1971-06','1972-06','1973-06','1974-06','1975-06','1976-06','1977-06','1978-06','1979-06','1980-06','1981-06','1982-06','1983-06','1984-06','1985-06','1986-06','1987-06','1988-06','1989-06','1990-06','1991-06','1992-06','1993-06','1994-06','1995-06','1996-06','1997-06','1998-06','1999-06','2000-06','2001-06','2002-06','2003-06','2004-06','2005-06','2006-06']
```

Then we plot three graphs in our grided graph.

```
mid_fig.plot(range(1962,2007), df_plt['Vancouver'], '-.',  c='#FF5789', alpha=0.25, label='Vancouver monthly precipitation (mm)')
mid_fig.plot(range(1962,2007), df_plt['Ottawa'], '-.',  c='#4843E8', alpha=0.25, label='Ottawa monthly precipitation (mm)')
mid_fig.fill_between(range(1962,2007),
                       df_plt['Vancouver'], df_plt['Ottawa'],
                       facecolor ='#57FFCB',
                       alpha=0.15)

top_fig.bar(range(1962,2007), df_plt['Rain'], width =1, color = '#6A5CFF', label='monthly  average rainfall difference  (mm)')
lower_fig.bar(range(1962,2007), df_plt['Snow'],  width =1, color = '#6A97FF', label='monthly  average snowfall difference  (mm)')
top_fig.axhline(0, color='#4E0179', alpha=0.25, linestyle='-');
lower_fig.axhline(0, color='#4E0179', alpha=0.25, linestyle='-');

plt.suptitle('Rain/Snowfall comparioson for Ottawa/Vancouver', fontsize="10")
```

Subsequently, we imporve the visibility of the data by removing unnecessary parts of the graphs.

```
mid_fig.spines['right'].set_visible(False);
mid_fig.spines['top'].set_visible(False);
mid_fig.spines['bottom'].set_visible(False);

top_fig.spines['right'].set_visible(False);
top_fig.spines['top'].set_visible(False);
top_fig.spines['bottom'].set_visible(False);

lower_fig.spines['right'].set_visible(False);
lower_fig.spines['top'].set_visible(False);

lower_fig.legend(loc=1, fontsize=8,   borderaxespad=0., frameon=False)
top_fig.legend(loc=4, fontsize=8,   borderaxespad=0., frameon=False)
mid_fig.legend(loc=2, frameon=False, fontsize=7)

top_fig.tick_params(top='off', bottom='off', left='on', right='off', labelleft='on', labelbottom='off')
mid_fig.tick_params(top='off', bottom='off', left='on', right='off', labelleft='on', labelbottom='off')
lower_fig.tick_params(top='off', bottom='off', left='on', right='off', labelleft='on', labelbottom='on')
```

Finally, we finish the graphing by adding labeling and some adjusments in the plotting.

```
lower_fig.set_yticks([40,20,0,-20,-40], minor=False)
lower_fig.set_yticklabels([40,20,0,20, 40], fontdict=None, minor=False, fontsize=7)

top_fig.set_yticks([40,20,0,-20,-40], minor=False)
top_fig.set_yticklabels([40,20,0,20,40], fontdict=None, minor=False,  fontsize=7)

mid_fig.set_yticks([100, 150, 200], minor=False)
mid_fig.set_yticklabels([100,150,200], fontdict=None, minor=False,  fontsize=7)

lower_fig.set_xticks([1965, 1975, 1985, 1995, 2005], minor=False)
lower_fig.set_xticklabels(["1960's", "1970's", "1980's", "1990's", "2000's"], fontdict=None, minor=False, fontsize=7)

top_fig.text(1960, 2, 'Vancouver',  fontsize=7, rotation = 90, verticalalignment='bottom')
top_fig.text(1960, -32, 'Ottawa',  fontsize=7, rotation = 90, verticalalignment='bottom')
lower_fig.text(1960, 2, 'Vancouver',  fontsize=7, rotation = 90, verticalalignment='bottom')
lower_fig.text(1960, -32, 'Ottawa',  fontsize=7, rotation = 90, verticalalignment='bottom')
```

Now we can plot the graphs and save a copy in a file.

```
plt.show()
fig.savefig('graph_ass4.png')
```

The outcome would be the attached png file. The middle graph obviously implies with these snow to liquid ratios, Vancouver had almost always more precipitation. 
