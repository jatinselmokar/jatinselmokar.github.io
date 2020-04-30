---
title: "Unsupervised Learning Approach Towards Neighborhood Segmentation"
excerpt: "Segmentation of San Francisco neighborhoods using K-Means clustering and FourSquare API"
header:
  teaser: /assets/images/SegmentationHeader.jpg
  overlay_image: /assets/images/SegmentationHeader.jpg
  caption: "Photo credit: [**Unsplash**](https://unsplash.com)"
author_profile: true
categories:
  - Data Science
tags:
  - Unsupervised Learning
  - FourSquare API
  - K-Means Clustering
  - Beautiful Soup
classes: wide
---

## Introduction

Opening a restaurant in a densely populated city is always challenging and often requires understanding of the current food preferences, location, competition, and the capital investment associated with it.

San Francisco being one of the most populated cities in the US has plethora of restaurants offering different cuisines. Although, offering great food at a lower cost is one of the success metrics, there are external factors that define restaurant’s success. Thus, in order to establish a prosperous business model, it is imperative for a business owner to understand and gauge the restaurant business in and around San Francisco.

Our main objective in this project is to guide the business client in choosing the perfect location to open a restaurant. This project aims to analyze and provide insights on restaurant businesses around SF neighborhoods using unsupervised learning techniques (K-Means Clustering) so that the business owner can make an informed decision.

## Data Collection

For the analysis, we will need the following data -

1. San Francisco neighborhood data
2. Geo-Coordinates for neighborhoods
3. Restaurant venue data for each neighborhood


### Neighborhood Data

The neighborhood data is extracted using python's BeautifulSoup package from the Wikipedia website.

``` python
from bs4 import BeautifulSoup

url = 'https://en.wikipedia.org/wiki/List_of_neighborhoods_in_San_Francisco'
results_url = requests.get(url).text
soup = BeautifulSoup(results_url, 'lxml' )
neighborhoods = []

for text in soup.find_all('span', class_='mw-headline'):
    neighborhoods.append(text.text)

neighborhoods = neighborhoods[:-4]

df  = pd.DataFrame(data=[neighborhoods]).T
df.columns = ['Neighborhood']
```

### Geo-Coordinates

The geo-coordinates for each neighborhood are populated using the google maps API.

``` python
from googlemaps import Client as GoogleMaps

for x in range(len(df)):
    geocode_result = gmaps.geocode(df['Address'][x])
    df['lat'][x] = geocode_result[0]['geometry']['location'] ['lat']
    df['long'][x] = geocode_result[0]['geometry']['location']['lng']
```

### Restaurant Venues

FourSquare API was used in retrieval of San Francisco restaurants data and their respective locations.

``` python
def getNearbyVenues(neighborhood, latitudes, longitudes, radius=radiustoexplore):

    venues_list=[]
    for neighborhood, lat, lng in zip(neighborhood, latitudes, longitudes):
        print(neighborhood)

        # create the API request URL
        url = 'https://api.foursquare.com/v2/venues/explore?&client_id={}&client_secret={}&v={}&ll={},{}&radius={}&limit={}'.format(
            CLIENT_ID,
            CLIENT_SECRET,
            VERSION,
            lat,
            lng,
            radius,
            limit)

        # make the GET request
        results = requests.get(url).json()["response"]['groups'][0]['items']

        # return only relevant information for each nearby venue
        for v in results:

            venues_list.append((
                neighborhood,
                lat,
                lng,
                v['venue']['name'],
                v['venue']['location']['lat'],
                v['venue']['location']['lng'],  
                v['venue']['categories'][0]['name']))

    nearby_venues = pd.DataFrame(venues_list)

    return(nearby_venues)

sanfran_venues = getNearbyVenues(  neighborhood=df['Neighborhood'],
                                   latitudes=df['lat'],
                                   longitudes=df['long']
                                  )

```

## Data Preprocessing & Exploration

### Exploring Different Restaurant Categories
```python
# #Explore
placetoexplore = 'Restaurant'
print('There are ',len(sanfran_venues['VenueCategory'].unique()),' venue categories around San Francisco')

#How many restaturant categories
uniquerestaurants = sanfran_venues[sanfran_venues['VenueCategory'].str.contains('{}'.format(placetoexplore))]['VenueCategory'].unique().tolist()
print('There are', len(uniquerestaurants), ' unique restaurants in SF area')

##Explore Other Categories
sanfran_venues['VenueCategory'].unique()

##Create a df for restaurants
restaurantdf = sanfran_venues[sanfran_venues['VenueCategory'].str.contains('{}'.format(placetoexplore))]
restaurantdf.head()
```
### Top 5 Restaurants Categories In Each Neighborhood

```python

sfrestaurant_grouped = sfrestaurant_onehot.groupby('Neighborhood').mean().reset_index()

num_top_venues = 5

for hood in sfrestaurant_grouped['Neighborhood']:
    print("----"+hood+"----")
    temp = sfrestaurant_grouped[sfrestaurant_grouped['Neighborhood'] == hood].T.reset_index()
    temp.columns = ['venue','freq']
    temp = temp.iloc[1:]
    temp['freq'] = temp['freq'].astype(float)
    temp = temp.round({'freq': 2})
    print(temp.sort_values('freq', ascending=False).reset_index(drop=True).head(num_top_venues))
    print('\n')
```
### One-Hot Encoding Restaurant Categories

```python
#Encode VenueCategory column
sfrestaurant_onehot = pd.get_dummies(restaurantdf[['VenueCategory']], prefix="", prefix_sep="")

# add neighborhood column back to dataframe
sfrestaurant_onehot['Neighborhood'] = restaurantdf['Neighborhood']
```

## K-Means clustering

### Selection Of K Using Elbow Point Method

```python
sfrestaurant_grouped_clustering = sfrestaurant_grouped.drop('Neighborhood', 1)

cost =[]
for i in range(1,10):
    KM = KMeans(n_clusters = i, max_iter = 500)
    KM.fit(sfrestaurant_grouped_clustering)
    cost.append(KM.inertia_)   # calculates squared error for the clustered points

plt.figure(figsize= (12,8))
plt.plot(range(1, 10), cost, color ='g', linewidth ='2')
plt.xlabel("Value of K")
plt.ylabel("Sqaured Error (Cost)")
plt.show()

```

### Fit Data for 5 clusters

```python
clusters = 5
sfrestaurant_grouped_clustering = sfrestaurant_grouped.drop('Neighborhood', 1)

# run k-means clustering
kmeans = KMeans(n_clusters=clusters, random_state=0).fit(sfrestaurant_grouped_clustering)

# check cluster labels generated for each row in the dataframe
neighborhoods_venues_sorted.insert(0, 'Cluster Labels', kmeans.labels_)
sanfran_merged = restaurantdf

# merge toronto_grouped with toronto_data to add latitude/longitude for each neighborhood
sanfran_merged = sanfran_merged.join(neighborhoods_venues_sorted.set_index('Neighborhood'), on='Neighborhood')
```
### Visualizing Clusters

```python
# create map
map_clusters = folium.Map(location=[latitude, longitude], zoom_start=12)

# set color scheme for the clusters
x = np.arange(clusters)
ys = [i + x + (i*x)**2 for i in range(clusters)]
colors_array = cm.rainbow(np.linspace(0, 1, len(ys)))
rainbow = [colors.rgb2hex(i) for i in colors_array]

# add markers to the map
markers_colors = []
for lat, lon, poi, cluster in zip(sanfran_merged['NeighborhoodLatitude'], sanfran_merged['NeighborhoodLongitude'], sanfran_merged['Neighborhood'], sanfran_merged['Cluster Labels']):
    label = folium.Popup(str(poi) + ' Cluster ' + str(cluster), parse_html=True)
    folium.CircleMarker(
        [lat, lon],
        radius=5,
        popup=label,
        color=rainbow[cluster-1],
        fill=True,
        fill_color=rainbow[cluster-1],
        fill_opacity=0.9).add_to(map_clusters)

map_clusters
```


For detailed code, please visit [Github]() link.
