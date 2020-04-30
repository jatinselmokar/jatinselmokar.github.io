---
title: "Unsupervised Learning Approach Towards Neighborhood Segmentation"
excerpt: "Segmentation of San Francisco neighborhoods using K-Means clustering and FourSquare API"
header:
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

For the analysis, we will need the following data

1. San Francisco neighborhood data
2. Geo-Coordinates for neighborhoods
3. Restaurant venue data for each neighborhood


### Neighborhood Data

The neighborhood data is extracted using python's BeautifulSoup package from the Wikipedia website.

``` python
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
from geopy.geocoders import Nominatim

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



For detailed code, please visit [Github]() link.
