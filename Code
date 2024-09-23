import time
import pickle
import pandas as pd
import numpy as np

import matplotlib
import matplotlib.pyplot as plt
#import seaborn as sns
#import folium

import datetime
from datetime import datetime

pd.set_option("display.max_columns", 100)

from stravalib.client import Client
client = Client()

## Read app details from local file
MY_STRAVA_CLIENT_ID, MY_STRAVA_CLIENT_SECRET = open('client.secret').read().strip().split(',')
print ('Client ID and secret read from file client.secret')

print('Client ID:', MY_STRAVA_CLIENT_ID, ', Secret:',MY_STRAVA_CLIENT_SECRET)
#print(MY_STRAVA_CLIENT_SECRET)

#### INITIAL AUTHENTICATION - ONLY NEEDED ONCE

## STEP 1 - get user to approve authosisation request
#client = Client()
#client_id = 117416
#url = client.authorization_url(client_id=client_id,
#                               redirect_uri='http://localhost:8001/authorization',
#                               scope=['read_all','profile:read_all','activity:read_all'])
#print(url)

## STEP 2 - copy the 'code' section from the url in the webpage that opens after clicking the generated URL and paste into 'code'

#code = 'b9f4c57034a430b30ff7fc698894eea05e65b869'

## STEP 3 - receieve access_token and store/dump in local file together with refresh code

#access_token = client.exchange_code_for_token(client_id=MY_STRAVA_CLIENT_ID,
#                                              client_secret=MY_STRAVA_CLIENT_SECRET,
#                                              code=code)
#with open('access_token.pickle', 'wb') as f:
#    pickle.dump(access_token, f)

## Check wheter token needs refreshing
with open('access_token.pickle', 'rb') as f:
    access_token = pickle.load(f)
    
print('Latest access token read from file:')
print(access_token)

if time.time() > access_token['expires_at']:
    print('Token has expired, will refresh')
    refresh_response = client.refresh_access_token(client_id=MY_STRAVA_CLIENT_ID, client_secret=MY_STRAVA_CLIENT_SECRET, refresh_token=access_token['refresh_token'])
    access_token = refresh_response
    with open('access_token.pickle', 'wb') as f:
        pickle.dump(refresh_response, f)
    print('Refreshed token saved to file')
    client.access_token = refresh_response['access_token']
    client.refresh_token = refresh_response['refresh_token']
    client.token_expires_at = refresh_response['expires_at']
        
else:
    print('Token still valid, expires at {}'
          .format(time.strftime("%a, %d %b %Y %H:%M:%S %Z", time.localtime(access_token['expires_at']))))
    client.access_token = access_token['access_token']
    client.refresh_token = access_token['refresh_token']
    client.token_expires_at = access_token['expires_at']

## Fetch activities
activities = client.get_activities()

my_cols =['name',
          'start_date_local',
          'type',
          'sport_type',
          'workout_type',
          'distance',
          'moving_time',
          'elapsed_time',
          'average_speed',
          'max_speed',
          'average_heartrate',
          'max_heartrate']

data = []
for activity in activities:
    my_dict = activity.to_dict()
    data.append([activity.id]+[my_dict.get(x) for x in my_cols])
my_cols.insert(0,'id')

df = pd.DataFrame(data, columns=my_cols)

#Create a distance in km column
df['distance'] = df['distance']/1e3

#Convert mph to kmh
df['average_speed_kmh'] = round(df['distance']/(df['moving_time']/3600),1)

# Convert dates to datetime type
df['start_date_local'] = pd.to_datetime(df['start_date_local'])
df['start_date_local_date'] = pd.to_datetime(df['start_date_local']).dt.date

#df['start_date_local_time'] = df['start_date_local'].time()

# Create a day of the week and month of the year columns
#df['day_of_week'] = df['start_date_local'].dt.day_name()
#df['month_of_year'] = df['start_date_local'].dt.month

# Convert timings to hours for plotting
df['elapsed_time_min'] = round(df['elapsed_time'].astype(int)/(60),1)
df['moving_time_min'] = round(df['moving_time'].astype(int)/(60),1)

df["Afstand"]= ""
df.loc[(df["distance"] > 41) & (df["distance"] <43), ["Afstand"]] = 'M'
df.loc[(df["distance"] > 20) & (df["distance"] <22), ["Afstand"]] = 'HM'
df.loc[(df["distance"] > 15.4) & (df["distance"] <17), ["Afstand"]] = '10EM'
df.loc[(df["distance"] > 14.5) & (df["distance"] <15.5), ["Afstand"]] = '15K'
df.loc[(df["distance"] > 9.5) & (df["distance"] <10.5), ["Afstand"]] = '10K'
df.loc[(df["distance"] > 4.5) & (df["distance"] <5.5), ["Afstand"]] = '5K'
df.loc[(df["distance"] > 2.7) & (df["distance"] <3.3), ["Afstand"]] = '3K'

df.to_csv('Strava_activiteiten.csv')

#Filter = df[df["workout_type"].isin([1,3]) & df["Afstand"].isin(['M','HM','10EM','15K','10K','5K','3K'])]
Filter = df[df["workout_type"].isin([1,3]) & df["Afstand"].isin(['HM','M', '10EM', '15K', '10K'])]

df_pivot = pd.pivot_table(
    Filter, values=["elapsed_time_min"], index=["Afstand","start_date_local_date"]
)

df_pivot.to_csv('Strava_geselecteerde_wedstrijden.csv')

df_pivot.head(20)

ax = df_pivot.plot(kind='barh', title='Wedstrijdtijden Gijsbert', grid='true', figsize=(12,15), rot=0)

ax.bar_label(ax.containers[0])
#ax.yaxis.set_tick_params(pad = 20)
#ax.xaxis.set_tick_params(pad = 20)
plt.legend(['elapsed_time_min'],loc='upper left', title='', prop={'size': 10})
plt.savefig('Wedstrijdtijden.png')