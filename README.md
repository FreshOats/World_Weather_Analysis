# World Weather Analysis
### _He likes it cold ..._
#### by Justin R. Papreck
---

## Overview
This project uses the OpenWeatherApp API and Google Maps API to find random locations on the globe, then show the nearest hotels to plan a vacation based on temperature constraints. In this analysis, the input temperature range was 0 degrees to 48 degrees as a maximum temperature. During the summer, it seems like a great time to escape from the heat and visit some of the coldest places that have hotels! To do this, a heatmap layer was added to the map and the nearest_cities search was applied, and subsequently only cities with hotels were kept. After finding the worldwide locations, 4 locations were chosen within driving distance of one another. These locations were shown with the a travel route and then with the pertinent information for the traveler, such as the city and weather. 

---
## Dependencies
pandas, numpy, time, requests, citipy, gmaps, associated api keys

---
## Retrieving the Weather Data
Initially, 2000 random sets of coordinates were created to use. Obviously there will be many global coordinates that are in the middle of the ocean, so there is definitely a need to find only coordinates near civilization. Using the citypy module, unique cities were added to a list if they were found near the coordinates supplied. Using these city names, the weather data was acquired from OpenWeatherApp in json form. Most of the data were simple to acquire, but the current weather description was inside a list within a list within a dictionary, so the code was modified to remove the text and then capitalize it, just to make it look nice. The data for maximum temperature, humidity, cloudiness, wind speed, and current weather were collected and assembled in a dataframe.  

```
  city_weather = requests.get(city_url).json()

  city_lat = city_weather["coord"]["lat"]
  city_lng = city_weather["coord"]["lon"]
  city_max_temp = city_weather["main"]["temp_max"]
  city_humidity = city_weather["main"]["humidity"]
  city_clouds = city_weather["clouds"]["all"]
  city_wind = city_weather["wind"]["speed"]
  city_country = city_weather["sys"]["country"]

  city_description = [i["description"] for i in city_weather["weather"]][0].capitalize()

  city_data.append({"City": city.title(),
                    "Country": city_country,
                    "Lat": city_lat,
                    "Lng": city_lng,
                    "Max Temp": city_max_temp,
                    "Humidity": city_humidity,
                    "Cloudiness": city_clouds,
                    "Wind Speed": city_wind,
                    "Current Description": city_description
                    })
```

This dataframe was then saved as a csv file to be used for the remainder of the analyses.

---
## Creating a Customer Travel Destinations Map
Initially, upon running this code, the customer is prompted to enter the minimum and maximum temperatures that they find ideal for their vacations: 

```
min_temp = float(input("What is the minimum temperature you desire for your trip?  "))
max_temp = float(input("What is the maximum temperature you desire for your trip?  "))

preferred_cities_df = city_data_df.loc[(city_data_df["Max Temp"] <= max_temp) & 
                                       (city_data_df["Max Temp"] >= min_temp)] 
```

At this point, the customer who shalt not be named, expressed their preference for the cold, setting the minimum to 0 degrees and the maximum to 48. This information is necessary to understand why absolutely no typical 'vacation spots' are highlighted on the maps, and only places at very high altitudes or nearing the Arctic or Antarctic circles are present. 

Drawing from the preferred_cities_df dataframe, the data needed to be filtered to find only locations with hotels within 5 km of the city. Hotel information was found using the Google API nearbysearch. A try/except method was used to bypass and inform of any locations without hotels. Once the locations without hotels were removed, these data were saved as a csv file.

```
params = {
    "radius": 5000, 
    "type": "lodging",
    "key": g_key
}


for index, row in hotel_df.iterrows():
    lat = row["Lat"]
    lng = row["Lng"]
    params["location"] = f"{lat},{lng}"
    
    base_url = "https://maps.googleapis.com/maps/api/place/nearbysearch/json"

    hotels = requests.get(base_url, params=params).json()
    
    try:
        hotel_df.loc[index, "Hotel Name"] = hotels["results"][0]["name"]
    except (IndexError):
        print("Vacation Error 404: Hotel not found!")     
```

In order to provide useful information while traversing the map, the City, Country, Hotel Name, and Weather should be shown when the location is clicked on. This was done in html. These formatted templates were then stored in a list. 

```
info_box_template = """
<dl>
<dt><b>Hotel Name</b></dt><dt>{Hotel Name}</dt>
<dt><b>City</b></dt><dt>{City}</dt>
<dt><b>Country</b></dt><dt>{Country}</dt>
<dt><b>Weather Description</b></dt><dt>{Current Description} and {Max Temp} °F</dt>
</dl>
"""

hotel_info = [info_box_template.format(**row) for index, row in clean_hotel_df.iterrows()]
```

Finally the world map with all locations and clickable information was displayed. 

![WeatherPy_vacation_map](https://user-images.githubusercontent.com/33167541/176560303-1f7764a0-17ad-4ef5-b035-4791f2f12a71.png)


---
## Creating a Travel Itinerary Map
From the previous results, the customer decided that the ideal place to travel was the southern tip of South America. The cities selected to visit were Ancud, Rawson, Ushuaia, and Coihaique in southern Argentina and Chile. The route followed that order, starting and ending in Ancud. The map was created with the following code, to include a layer to show the route. 

```
figure_layout = {
    'width': '1500px',
    'height': '1500px'
}

fig = gmaps.figure(layout=figure_layout)

four_stop_trip = gmaps.directions_layer(start, end, waypoints=[stop1, stop2, stop3], travel_mode='DRIVING')
fig.add_layer(four_stop_trip)
fig
```

![WeatherPy_travel_map](https://user-images.githubusercontent.com/33167541/176560852-3f89d3c5-169e-4274-86fb-4eb6cb20e0e3.png)


And finally, the hotels selected are shown along with the other selected information. 

![WeatherPy_travel_map_markers](https://user-images.githubusercontent.com/33167541/176560920-e9d5c8f7-e08d-4144-8e46-d745fc14ee1e.png)


Clearly, the customer is looking forward to the Albatross Hotel the most, with it's high temperature of 37 degrees. 
