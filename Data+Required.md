
# Capstone Project - The Battle of Neighborhoods

## Data

Following data will be reuqired to solve the problem statement chosen.

#### 1. Geographic coordinate of Hong Kong cinemas

I need to **compare 5 possible locations with current cinemas** in Hong Kong. Therefore, I need to find a list of Hong Kong cinema and cinemas' geographic coordinates. Luckily, I can find the list and coordinates from the website https://hkmovie6.com/cinema .


```python
# Import necessary library
import json
import pandas as pd
```


```python
# Download the cinema list
!wget -O hk_cinema_list.json https://hkmovie6.com/api/cinemas/lists
```

    --2018-09-27 01:29:16--  https://hkmovie6.com/api/cinemas/lists
    Resolving hkmovie6.com (hkmovie6.com)... 104.27.132.135, 104.27.133.135, 2606:4700:30::681b:8487, ...
    Connecting to hkmovie6.com (hkmovie6.com)|104.27.132.135|:443... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: unspecified [application/json]
    Saving to: ‘hk_cinema_list.json’
    
    hk_cinema_list.json     [ <=>                  ]  51.74K   291KB/s   in 0.2s   
    
    2018-09-27 01:29:17 (291 KB/s) - ‘hk_cinema_list.json’ saved [52984]
    
    


```python
# Convert the JSON data into DataFrmae
cinemas_json = None
with open('hk_cinema_list.json', 'r') as f:
    cinemas_json = json.load(f)
    
cinemas = []
for data in cinemas_json['data']:
    cinemas.append({
        'Name': data['name'],
        'Address': data['address'],
        'Latitude': data['lat'],
        'Longitude': data['lon']
    })
df_cinemas = pd.DataFrame(cinemas, columns=['Name','Address','Latitude','Longitude'])
```


```python
print('There are {} cinemas in Hong Kong'.format(len(df_cinemas)))
```

    There are 68 cinemas in Hong Kong
    


```python
df_cinemas.head()
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
      <th>Name</th>
      <th>Address</th>
      <th>Latitude</th>
      <th>Longitude</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Emperor Cinemas - Entertainment Building</td>
      <td>3/F, Emperor Cinemas Entertainment Building, 3...</td>
      <td>22.281453</td>
      <td>114.154230</td>
    </tr>
    <tr>
      <th>1</th>
      <td>The Coronet @ Emperor Cinemas - Entertainment ...</td>
      <td>3/F, Emperor Cinemas Entertainment Building, 3...</td>
      <td>22.281453</td>
      <td>114.154230</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Emperor Cinemas - Tuen Mun</td>
      <td>3/F, New Town Commercial Arcade, 2 Tuen Lee St...</td>
      <td>22.390776</td>
      <td>113.975983</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Broadway Circuit - CYBERPORT</td>
      <td>Shop L1 - 3, Level 1, The Arcade, 100 Cyberpor...</td>
      <td>22.261067</td>
      <td>114.129825</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Broadway Circuit - PALACE IFC</td>
      <td>Podium L1, IFC Mall, 8 Finance Street, Central</td>
      <td>22.285545</td>
      <td>114.157979</td>
    </tr>
  </tbody>
</table>
</div>



#### 2. Geographic coordinates of 5 possible cinema addresses
I also need to know the geographic coordinates of 5 possible cinemas. I can use Google Map API to find this information


```python
possible_locations = [
    { 'Location': 'L1', 'Address': 'Sau Mau Ping Shopping Centre, Sau Mau Ping'},
    { 'Location': 'L2', 'Address': 'Tuen Mun Ferry, Tuen Mun'},
    { 'Location': 'L3', 'Address': 'Un Chau Shopping Centre, Cheung Sha Wan'},
    { 'Location': 'L4', 'Address': 'Prosperity Millennia Plaza, North Point'},
    { 'Location': 'L5', 'Address': 'Tsuen Fung Centre Shopping Arcade, Tsuen Wan'},
]
```


```python
# install the google map api client library
!pip install -U googlemaps
```

    Collecting googlemaps
      Downloading https://files.pythonhosted.org/packages/5a/3d/13b4230f3c1b8a586cdc8d8179f3c6af771c11247f8de9c166d1ab37f51d/googlemaps-3.0.2.tar.gz
    Requirement not upgraded as not directly required: requests<3.0,>=2.11.1 in /home/jupyterlab/conda/lib/python3.6/site-packages (from googlemaps) (2.18.4)
    Requirement not upgraded as not directly required: chardet<3.1.0,>=3.0.2 in /home/jupyterlab/conda/lib/python3.6/site-packages (from requests<3.0,>=2.11.1->googlemaps) (3.0.4)
    Requirement not upgraded as not directly required: idna<2.7,>=2.5 in /home/jupyterlab/conda/lib/python3.6/site-packages (from requests<3.0,>=2.11.1->googlemaps) (2.6)
    Requirement not upgraded as not directly required: urllib3<1.23,>=1.21.1 in /home/jupyterlab/conda/lib/python3.6/site-packages (from requests<3.0,>=2.11.1->googlemaps) (1.22)
    Requirement not upgraded as not directly required: certifi>=2017.4.17 in /home/jupyterlab/conda/lib/python3.6/site-packages (from requests<3.0,>=2.11.1->googlemaps) (2018.8.24)
    Building wheels for collected packages: googlemaps
      Running setup.py bdist_wheel for googlemaps ... [?25ldone
    [?25h  Stored in directory: /home/jupyterlab/.cache/pip/wheels/3c/3f/25/ce6d7722dba07e5d4a12d27ab38f3d7add65ef43171b02c819
    Successfully built googlemaps
    [31mdistributed 1.21.8 requires msgpack, which is not installed.[0m
    Installing collected packages: googlemaps
    Successfully installed googlemaps-3.0.2
    


```python
google_act = None
with open('google_map_act.json', 'r') as f:
    google_act = json.load(f)
    
GOOGLE_MAP_API_KEY = google_act['api_key']    

import googlemaps
gmaps = googlemaps.Client(key=GOOGLE_MAP_API_KEY)
```


```python
# Retrieve geolocation and create the dataframe of pending cinema addresses
def getLatLng(address):
    latlnt = gmaps.geocode('{}, Hong Kong'.format(address))
    return (latlnt[0]['geometry']['location']['lat'], latlnt[0]['geometry']['location']['lng'])
```

Dataframe of 5 possible locations with geographic coordinates information


```python
for loc in possible_locations:        
    (lat, lng) = getLatLng(loc['Address'])
    loc['Latitude'] = lat
    loc['Longitude'] = lng
    
df_possible_locations = pd.DataFrame(possible_locations, columns=['Location', 'Address', 'Latitude', 'Longitude'])
df_possible_locations
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
      <th>Location</th>
      <th>Address</th>
      <th>Latitude</th>
      <th>Longitude</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>L1</td>
      <td>Sau Mau Ping Shopping Centre, Sau Mau Ping</td>
      <td>22.319503</td>
      <td>114.232187</td>
    </tr>
    <tr>
      <th>1</th>
      <td>L2</td>
      <td>Tuen Mun Ferry, Tuen Mun</td>
      <td>22.371780</td>
      <td>113.966039</td>
    </tr>
    <tr>
      <th>2</th>
      <td>L3</td>
      <td>Un Chau Shopping Centre, Cheung Sha Wan</td>
      <td>22.337280</td>
      <td>114.156457</td>
    </tr>
    <tr>
      <th>3</th>
      <td>L4</td>
      <td>Prosperity Millennia Plaza, North Point</td>
      <td>22.291698</td>
      <td>114.208168</td>
    </tr>
    <tr>
      <th>4</th>
      <td>L5</td>
      <td>Tsuen Fung Centre Shopping Arcade, Tsuen Wan</td>
      <td>22.372112</td>
      <td>114.119317</td>
    </tr>
  </tbody>
</table>
</div>



#### 3. Favorite cinema list of stakeholder

The favorite cinema list is an important information that I can **use it as profile to select the best location**.  
Stakeholder further explains that the rating is range of 1.0 (worst) to 5.0 (best) values


```python
boss_favorite = [
    {'Name': 'Boradway Circuit - MONGKONG', 'Rating': 4.5},
    {'Name': 'Boradway Circuit - The ONE', 'Rating': 4.5},
    {'Name': 'Grand Ocean', 'Rating': 4.3},
    {'Name': 'The Grand Cinema', 'Rating': 3.4},
    {'Name': 'AMC Pacific Place', 'Rating': 2.3},
    {'Name': 'UA IMAX @ Airport', 'Rating': 1.5},
]

df_boss_favorite = pd.DataFrame(boss_favorite, columns=['Name','Rating'])
df_boss_favorite
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
      <th>Name</th>
      <th>Rating</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Boradway Circuit - MONGKONG</td>
      <td>4.5</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Boradway Circuit - The ONE</td>
      <td>4.5</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Grand Ocean</td>
      <td>4.3</td>
    </tr>
    <tr>
      <th>3</th>
      <td>The Grand Cinema</td>
      <td>3.4</td>
    </tr>
    <tr>
      <th>4</th>
      <td>AMC Pacific Place</td>
      <td>2.3</td>
    </tr>
    <tr>
      <th>5</th>
      <td>UA IMAX @ Airport</td>
      <td>1.5</td>
    </tr>
  </tbody>
</table>
</div>



#### 4. Eating, Shopping and Public transportation facility around cinema
The recommended cinema location needs to have many eating and shopping venues nearby. Convenient public transport is also required.  
I can use FourSquare API to find these venues around the location. 

5 minutes walking distance is about 500m. I think it is the suitable distance to search nearby venues.

However, the API provides maximum 50 results only, so it is better to search venues by category. Following categories will be used for finding the target venues. Full list of categories: https://developer.foursquare.com/docs/resources/categories


```python
cinema = df_cinemas.loc[0]
```


```python
print('Use the first cinema "{}" in the list as example to explore venues nearyby'.format(cinema['Name']))
```

    Use the first cinema "Emperor Cinemas - Entertainment Building" in the list as example to explore venues nearyby
    


```python
fs_categories = {
    'Food': '4d4b7105d754a06374d81259',
    'Shop & Service': '4d4b7105d754a06378d81259',
    'Bus Stop': '52f2ab2ebcbc57f1066b8b4f',
    'Metro Station': '4bf58dd8d48988d1fd931735',
    'Nightlife Spot': '4d4b7105d754a06376d81259',
    'Arts & Entertainment': '4d4b7104d754a06370d81259'
}
```


```python
# Install FourSquare client library
!pip install foursquare
```

    Collecting foursquare
      Downloading https://files.pythonhosted.org/packages/7e/9f/21ef283c50eb576eaebb0525d8a988baffe4d59ac2bbb1f9d84434bdf616/foursquare-1%212016.9.12.tar.gz
    Requirement already satisfied: requests>=2.1 in /home/jupyterlab/conda/lib/python3.6/site-packages (from foursquare) (2.18.4)
    Requirement already satisfied: six in /home/jupyterlab/conda/lib/python3.6/site-packages (from foursquare) (1.11.0)
    Requirement already satisfied: chardet<3.1.0,>=3.0.2 in /home/jupyterlab/conda/lib/python3.6/site-packages (from requests>=2.1->foursquare) (3.0.4)
    Requirement already satisfied: idna<2.7,>=2.5 in /home/jupyterlab/conda/lib/python3.6/site-packages (from requests>=2.1->foursquare) (2.6)
    Requirement already satisfied: urllib3<1.23,>=1.21.1 in /home/jupyterlab/conda/lib/python3.6/site-packages (from requests>=2.1->foursquare) (1.22)
    Requirement already satisfied: certifi>=2017.4.17 in /home/jupyterlab/conda/lib/python3.6/site-packages (from requests>=2.1->foursquare) (2018.8.24)
    Building wheels for collected packages: foursquare
      Running setup.py bdist_wheel for foursquare ... [?25ldone
    [?25h  Stored in directory: /home/jupyterlab/.cache/pip/wheels/c1/a4/ff/e07a4f4f02ef7189c5b1e0738a09131f6c5f2de811ce3a39a0
    Successfully built foursquare
    [31mdistributed 1.21.8 requires msgpack, which is not installed.[0m
    Installing collected packages: foursquare
    Successfully installed foursquare-1!2016.9.12
    


```python
fs_act = None
with open('fs_act.json') as json_data:
    fs_act = json.load(json_data)
```


```python
import foursquare
from pandas.io.json import json_normalize # tranform JSON file into a pandas dataframe
fs = foursquare.Foursquare(client_id=fs_act['client_id'], client_secret=fs_act['client_secret'])
```


```python
RADIUS = 500 # 500m, around 5 minutes walking time
```


```python
# Define a function to search nearby information and convert the result as dataframe
def venues_nearby(latitude, longitude, category):    
    results = fs.venues.search(
        params = {
            'query': category, 
            'll': '{},{}'.format(latitude, longitude),
            'radius': RADIUS,
            'categoryId': fs_categories[category]
        }
    )    
    df = json_normalize(results['venues'])
    cols = ['Name','Latitude','Longitude','Tips','Users','Visits']    
    if( len(df) == 0 ):        
        df = pd.DataFrame(columns=cols)
    else:        
        df = df[['name','location.lat','location.lng','stats.tipCount','stats.usersCount','stats.visitsCount']]
        df.columns = cols
    print('{} "{}" venues are found within {}m of location'.format(len(df), category, RADIUS))
    return df
    
```

Find number of MTR station around the cinema


```python
venues_nearby(cinema['Latitude'], cinema['Longitude'], 'Metro Station').head()
```

    2 "Metro Station" venues are found within 500m of location
    




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
      <th>Name</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Tips</th>
      <th>Users</th>
      <th>Visits</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>MTR Central Station (港鐵中環站)</td>
      <td>22.281911</td>
      <td>114.158406</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>MTR Hong Kong Station (港鐵香港站)</td>
      <td>22.284926</td>
      <td>114.158314</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>



Find number of bus station around the cinema


```python
venues_nearby(cinema['Latitude'], cinema['Longitude'], 'Bus Stop').head()
```

    30 "Bus Stop" venues are found within 500m of location
    




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
      <th>Name</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Tips</th>
      <th>Users</th>
      <th>Visits</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Seymour Road / Robinson Road Bus Stop 西摩道／羅便臣道巴士站</td>
      <td>22.280465</td>
      <td>114.150347</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Douglas Street Bus Stop 德忌利士街巴士站</td>
      <td>22.283273</td>
      <td>114.156910</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>HSBC Headquarters Bus Stop 匯豐總行巴士站</td>
      <td>22.280577</td>
      <td>114.159446</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Dr. Sun Yat-Sen Museum Bus Stop 孫中山紀念館巴士站</td>
      <td>22.279132</td>
      <td>114.152743</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Hang Seng Bank Head Office Bus Stop 恒生銀行總行巴士站</td>
      <td>22.283998</td>
      <td>114.156038</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>



Find eating places around the cinema


```python
venues_nearby(cinema['Latitude'], cinema['Longitude'], 'Food').head()
```

    25 "Food" venues are found within 500m of location
    




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
      <th>Name</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Tips</th>
      <th>Users</th>
      <th>Visits</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Mana! Fast Slow Food</td>
      <td>22.282921</td>
      <td>114.154651</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Good Luck Thai Food (鴻運泰國美食)</td>
      <td>22.281165</td>
      <td>114.155296</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Chiu Lung Fast Food (昭隆美食)</td>
      <td>22.282659</td>
      <td>114.156753</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Soul Food</td>
      <td>22.281668</td>
      <td>114.152495</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Sun Hing Fast Food (新興美食)</td>
      <td>22.282521</td>
      <td>114.156717</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>




```python
venues_nearby(cinema['Latitude'], cinema['Longitude'], 'Arts & Entertainment').head()
```

    12 "Arts & Entertainment" venues are found within 500m of location
    




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
      <th>Name</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Tips</th>
      <th>Users</th>
      <th>Visits</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Tai Kwun Centre for Heritage and Arts (大館古蹟及藝術館)</td>
      <td>22.281668</td>
      <td>114.154216</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Wah Tung China Arts Limited (華通陶瓷藝術有限公司)</td>
      <td>22.283046</td>
      <td>114.152723</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Ravenel Fine Arts Limited 睿芙奧</td>
      <td>22.281819</td>
      <td>114.156906</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>KONG Arts Space</td>
      <td>22.281751</td>
      <td>114.153300</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>State Of The Arts</td>
      <td>22.282225</td>
      <td>114.155006</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>



With above data, I can build a **content-based recommender systems** to resolve the problem.  

Combine with FourSquare API on counting how many different venues (Food, Transport, Night Life) and Hong Kong cinema list, a **cinema nearby venues matrix** can be built. Stakeholder's favorite list is the **profile** to combine with cinema nearby venues matrix to become a **weighted matrix of favorite cinema**.

The weighted matrix can be applied on **5 possible locations with venues information** to generate a ranking result. The **the top one** on the ranking list can be recommended to the stakeholder.

