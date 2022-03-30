# World Weather Analysis

## Overview

The PlanMyTrip app needs a few updates after the Beta testers submitted feedback. They are wanting more weather information and the ability to tailor their destinations to weather preferences. 

## Objective

Using OpenWeather API and Google Nearby and Directions APIs, the app pulled weather and hotel information for random latitude and longitudes pairs.  User prefences narrowed down the locations, and the client then chose four cities for the itinerary. 

## Workflow

1. Create random pairs of latitude / longitudes using numpy and radom libraries
```
lats = np.random.uniform(low=-90.000, high=90.000, size=2000)
lngs = np.random.uniform(low=-180.000, high=180.000, size=2000)
lat_lngs = zip(lats, lngs)
lat_lngs
```

2. Call OpenWeatherMap's API for the city location and weather information.

```
        city_lat = city_weather["coord"]["lat"]
        city_lng = city_weather["coord"]["lon"]
        city_max_temp = city_weather["main"]["temp_max"]
        city_humidity = city_weather["main"]["humidity"]
        city_clouds = city_weather["clouds"]["all"]
        city_wind = city_weather["wind"]["speed"]
        city_country = city_weather["sys"]["country"]
        city_descript = city_weather["weather"][0]["main"]
        city_details = city_weather["weather"][0]["description"]
```

3. Create a Panda's DataFrame and output to CSV file. 
```
new_column_order = ['City', 'Country', 'Lat', 'Lng', 'Max Temp', 'Humidity', 'Cloudiness', 'Wind Speed', 'Description']
city_data_df = pd.DataFrame(city_data)
city_data_df=city_data_df[new_column_order]
output_file="./WeatherPy_Database.csv"
city_data_df.to_csv(output_file, index_label="City_ID")
```

4. Ask for client input on minimum and maximum temperatures and filter map. 

```
min_temp = float(input("What is the minimum temperature you would like for your trip? "))
max_temp = float(input("What is the maximum temperature you would like for your trip? "))
vac_city_data_df = city_data_df.loc[(city_data_df["Max Temp"] <= max_temp) & (city_data_df["Max Temp"]>= min_temp)].copy()
```
5. Use new set of locations to find hotel information within 5,000 meters

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
  
    hotel_data = requests.get(base_url, params = params).json()
    
    try:
        hotel_df.loc[index, "Hotel Name"] = hotel_data["results"][0]["name"]
    except (IndexError):
        print('Hotel not found... skipping')
    
```


6. Plot hotel and weather information on gmaps. 
![Screenshot of map and info box](./WeatherPy_vacation_map.png)
7. Pick four locations to create an itinerary. 
```
vacation_start = vacation_df.loc[vacation_df["City"]=="Lasa"]
vacation_end = vacation_df.loc[vacation_df["City"]=="Lasa"]
vacation_stop1 = vacation_df.loc[vacation_df["City"]=="Salsomaggiore Terme"]
vacation_stop2 = vacation_df.loc[vacation_df["City"]=="Montepulciano"]
vacation_stop3 = vacation_df.loc[vacation_df["City"]=="Crotone"]
```

8. Get driving directions for the 4 cities by using Google's Directions API and plot results. 
```
#Converting Lat, Lng Series into tuples

start = vacation_start["Lat"].to_numpy()[0], vacation_start["Lng"].to_numpy()[0]
end = start
stop1 = vacation_stop1["Lat"].to_numpy()[0], vacation_stop1["Lng"].to_numpy()[0]
stop2 = vacation_stop2["Lat"].to_numpy()[0], vacation_stop2["Lng"].to_numpy()[0]
stop3 = vacation_stop3["Lat"].to_numpy()[0], vacation_stop3["Lng"].to_numpy()[0]

...
#Plotting with gmaps

fig = gmaps.figure()
vacation_itinerary = gmaps.directions_layer(
        start, end, waypoints=[stop1, stop2, stop3],
        travel_mode='DRIVING')

fig.add_layer(vacation_itinerary)
fig
```
![Directions for stops in Italy](./WeatherPy_travel_map.png)

9. Plot results for itinerary

![Four stops in Italy](./WeatherPy_travel_map_markers.png)

## Conclusion
Including these new features in PlanMyTrip is going to be great! The new itinerary is well received and the client is looking forward to their next vacation. 