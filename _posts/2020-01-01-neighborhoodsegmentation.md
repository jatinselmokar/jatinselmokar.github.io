---
title: "An Unsupervised Learning Approach Towards Neighborhood Segmentation"
excerpt: "Segmentation of San Francisco neighborhoods using K-Means clustering and FourSquare API"
header:
  teaser: /assets/images/clustering/SegmentationHeader.jpg
  overlay_image: /assets/images/clustering/SegmentationHeader.jpg
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
Output:

<img src="/assets/images/clustering/neighborhoodlist.jpg" alt="HTML PIC" style="width:400px;height:300px;">

### Geo-Coordinates

The geo-coordinates for each neighborhood are populated using the google maps API.

``` python
from googlemaps import Client as GoogleMaps

for x in range(len(df)):
    geocode_result = gmaps.geocode(df['Address'][x])
    df['lat'][x] = geocode_result[0]['geometry']['location'] ['lat']
    df['long'][x] = geocode_result[0]['geometry']['location']['lng']
```
Output:

<img src="/assets/images/clustering/geocoordinates.png" alt="Geo-Coordinates" style="width:600px;height:300px;" >

### Restaurant Venues

FourSquare API is used in retrieval of San Francisco restaurant's venue data.

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

sanfran_venues.columns = ['Neighborhood', 'NeighborhoodLatitude', 'NeighborhoodLongitude',
'VenueName', 'VenueLatitude', 'VenueLongitude', 'VenueCategory']
sanfran_venues.head(3)

```
Output:

<img src="/assets/images/clustering/venues.png" alt="HTMLPIC">

## Data Preprocessing & Exploration

### Exploring Different Restaurant Categories
```python
# #Explore
placetoexplore = 'Restaurant'
print('There are ',len(sanfran_venues['VenueCategory'].unique()),' venue categories around San Francisco')

#How many restaurant categories
uniquerestaurants = sanfran_venues[sanfran_venues['VenueCategory'].str.contains('{}'.format(placetoexplore))]['VenueCategory'].unique().tolist()
print('There are', len(uniquerestaurants), ' unique restaurants in SF area')

##Explore Other Categories
sanfran_venues['VenueCategory'].unique()

##Create a df for restaurants
restaurantdf = sanfran_venues[sanfran_venues['VenueCategory'].str.contains('{}'.format(placetoexplore))]
restaurantdf.head()
```
Output:

<img src="/assets/images/clustering/uniquecategories.png" alt="Unique Categories">


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
Output:

<img src="/assets/images/clustering/top5.png" alt="Top 5 Categories" style="height:400px;">

### One-Hot Encoding Restaurant Categories

As the restaurant categories contains categorical data, it is one-hot encoded for the purpose of K-Means algorithm.  
Panda's "get_dummies" method is used in encoding the venue category column.

```python
#Encode VenueCategory column
sfrestaurant_onehot = pd.get_dummies(restaurantdf[['VenueCategory']], prefix="", prefix_sep="")

# add neighborhood column back to dataframe
sfrestaurant_onehot['Neighborhood'] = restaurantdf['Neighborhood']
```
Output:

<img src="/assets/images/clustering/onehot.png" alt="Encoding">

## K-Means clustering
### Selection Of "K" Value Using Elbow Method

K-Means algorithm is an unsupervised learning algorithm that segregates data into "k" groups as defined by the user. Since the method of randomly selecting "k" value is naive and doesn't always produce sub-optimal results, we use a metric called "Elbow method" in determining the sub-optimal cluster size for the data.

The main idea behind it is to run the K-Means algorithm multiple times on different cluster sizes and calculate the inertia at each step i.e., the sum of squared distances of samples to their closest cluster center. The point at which the inertia starts decreasing in a linear fashion is termed as the "Elbow Point" and the associated cluster size is chosen for the algorithm.

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
Output:

<img src="/assets/images/clustering/elbowmethod.png" alt="Top 5 Categories" style="width:500px;height:400px;">

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
Output:

<img src="/assets/images/clustering/kmeansfit.png" alt="Kmeans Fit" >

### Visualizing Clusters

The clustering algorithm forms 5 clusters which can be further used to
understand the restaurant outlook in San Francisco. They also give
information on the popular restaurants in each of the neighborhood areas.

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

Output:

<img src="/assets/images/clustering/clusters.png" alt="Clusters" >

### Restaurant Categories In Each Cluster

* Cluster 1 (Red) – Japanese, Sushi, and Chinese restaurants
* Cluster 2 (Purple) – Mexican & Southern/Soul restaurants
* Cluster 3 (Blue) – Asian restaurants
* Cluster 4 (Green) – American restaurants
* Cluster 5 (Orange) – Vietnamese, Italian, and New American restaurants

## Few Insights On SF Restaurant Businesses
* Majority of the neighborhoods fall in the Orange cluster where Vietnamese, Italian, and New American are the
popular cuisines
* Wide variety of cuisines available throughout San Francisco
* Asian food is popular in cluster 3 (Sunnyvale area)

Based on one's choice of cuisine, clusters can be further studied to decide the opening location of restaurant. For instance, if the business owner had to open an Indian restaurant, neighborhoods in clusters 2,3, and 4 would be the ideal locations.

Although only venue data is used in analyzing the restaurant outlook, the scope of analysis can be extended to include data like population and average income per household to further bolster the findings.


For the entire code and detailed report, please visit *[Github](https://github.com/jatinselmokar/Neighborhood-Segmentation-Using-Unsupervised-Learning-and-FourSquare-API)* link.
