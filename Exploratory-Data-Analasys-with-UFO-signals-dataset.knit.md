
<!-- rnb-text-begin -->

---
title: "Exploratory Data Analasys with UFO signals dataset"
output: html_notebook
---

The analysis focuses on uncovering **temporal**, **spatial**, and **behavioral patterns** in UFO reports. It explores when sightings are most frequent (by hour, day, and year), where they occur (by country and city size), how they are described (e.g., shape, presence of images), and how these aspects relate to population, geography, and daylight cycles.

## Description of the data

Database contains structured information about UFO sightings reported to NUFORC, including details about the **time**, **location**, **appearance**, and **context** of each event. It is supported by additional datasets describing **geographic attributes of cities** and **sunlight conditions** based on time and location.

`ufo_sightings.csv`

| variable | class | description |
|:-----------------------|:-----------------------|:-----------------------|
| reported_date_time | datetime | The time and date of the sighting, as it appears in the original NUFORC data. |
| reported_date_time_utc | datetime | The time and date of the sighting, normalized to UTC. |
| posted_date | datetime | The date when the sighting was posted to NUFORC. |
| city | character | The city of the sighting. Some of these have been cleaned from the original data. |
| state | character | The state, province, or similar division of the sighting. |
| country_code | character | The 2-letter country code of the sighting, normalized from the original data. |
| shape | character | The reported shape of the craft. |
| reported_duration | character | The reported duration of the event, in the reporter's words. |
| duration_seconds | double | The duration normalized to seconds using regex. |
| summary | character | The reported summary of the event. |
| has_images | logical | Whether the sighting has images available on NUFORC. |
| day_part | character | The approximate part of the day in which the sighting took place, based on the reported date and time, the place, and data from sunrise-sunset.org. Latitude and longitude were rounded to the 10s digit, and the date was rounded to the week, to match against time points such as "nautical twilight", "sunrise", and "sunset." |

`places.csv`

| variable | class | description |
|:-----------------------|:-----------------------|:-----------------------|
| city | character | Unique cities in which sightings took place. |
| alternate_city_names | character | Comma-separated other names for the city. |
| state | character | The state, province, or similar division of the sighting. |
| country | character | The name of the country. |
| country_code | character | The 2-letter country code of the sighting. |
| latitude | double | The latitude for this city, from geonames.org. |
| longitude | double | The longitude for this city, from geonames.org. |
| timezone | character | The timezone for this city, from geonames.org. |
| population | double | The population for this city, from geonames.org. |
| elevation_m | double | The elevation in meters for this city, from geonames.org. |

`day_parts_map.csv`

| variable | class | description |
|:-----------------------|:-----------------------|:-----------------------|
| rounded_lat | double | Latitudes rounded to the tens digit. |
| rounded_long | double | Longitudes rounded to the tens digit. |
| rounded_date | double | Dates rounded to the nearest week. |
| astronomical_twilight_begin | double | The UTC time of day when astronomical twilight began on this date in this location. Astronomical twilight begins when the sun is 18 degrees below the horizon before sunrise. |
| nautical_twilight_begin | double | The UTC time of day when nautical twilight began on this date (or the next date) in this location. Nautical twilight begins when the sun is 12 degrees below the horizon before sunrise. |
| civil_twilight_begin | double | The UTC time of day when civil twilight began on this date (or the next date) in this location. Civil twilight begins when the sun is 6 degrees below the horizon before sunrise. |
| sunrise | double | The UTC time of day when the sun rose on this date (or the next date) in this location. |
| solar_noon | double | The UTC time of day when the sun was at its zenith on this date (or the next date) in this location. |
| sunset | double | The UTC time of day when the sun set on this date (or the next date) in this location. |
| civil_twilight_end | double | The UTC time of day when civil twilight ended on this date (or the next date) in this location. Civil twilight ends when the sun is 6 degrees below the horizon after sunset. |
| nautical_twilight_end | double | The UTC time of day when nautical twilight ended on this date (or the next date) in this location. Nautical twilight ends when the sun is 12 degrees below the horizon after sunset. |
| astronomical_twilight_end | double | The UTC time of day when astronomical twilight ended on this date (or the next date) in this location. Astronomical twilight ends when the sun is 18 degrees below the horizon after sunset. |

# Load the data and check its structure (variables, missing values, data types)


<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-output-begin eyJkYXRhIjoiXG48IS0tIHJuYi1zb3VyY2UtYmVnaW4gZXlKa1lYUmhJam9pWUdCZ2NseHViR2xpY21GeWVTaDBhV1I1ZG1WeWMyVXBJQ0JjYm14cFluSmhjbmtvYUdWeVpTa2dJQ0FnSUNBZ1hHNXNhV0p5WVhKNUtIZHBkR2h5S1NBZ0lDQWdJRnh1WUdCZ0luMD0gLS0+XG5cbmBgYHJcbmxpYnJhcnkodGlkeXZlcnNlKSAgXG5saWJyYXJ5KGhlcmUpICAgICAgIFxubGlicmFyeSh3aXRocikgICAgICBcbmBgYFxuXG48IS0tIHJuYi1zb3VyY2UtZW5kIC0tPlxuIn0= -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxubGlicmFyeSh0aWR5dmVyc2UpICBcbmxpYnJhcnkoaGVyZSkgICAgICAgXG5saWJyYXJ5KHdpdGhyKSAgICAgIFxuYGBgIn0= -->

```r
library(tidyverse)  
library(here)       
library(withr)      
```

<!-- rnb-source-end -->


<!-- rnb-output-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->



<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-output-begin eyJkYXRhIjoiXG48IS0tIHJuYi1zb3VyY2UtYmVnaW4gZXlKa1lYUmhJam9pWUdCZ2NseHVkV1p2WDNOcFoyaDBhVzVuY3lBOExTQnlaV0ZrY2pvNmNtVmhaRjlqYzNZb0oyaDBkSEJ6T2k4dmNtRjNMbWRwZEdoMVluVnpaWEpqYjI1MFpXNTBMbU52YlM5eVptOXlaR0YwWVhOamFXVnVZMlV2ZEdsa2VYUjFaWE5rWVhrdmJXRnBiaTlrWVhSaEx6SXdNak12TWpBeU15MHdOaTB5TUM5MVptOWZjMmxuYUhScGJtZHpMbU56ZGljcFhHNXdiR0ZqWlhNZ1BDMGdjbVZoWkhJNk9uSmxZV1JmWTNOMktDZG9kSFJ3Y3pvdkwzSmhkeTVuYVhSb2RXSjFjMlZ5WTI5dWRHVnVkQzVqYjIwdmNtWnZjbVJoZEdGelkybGxibU5sTDNScFpIbDBkV1Z6WkdGNUwyMWhhVzR2WkdGMFlTOHlNREl6THpJd01qTXRNRFl0TWpBdmNHeGhZMlZ6TG1OemRpY3BYRzVrWVhsZmNHRnlkSE5mYldGd0lEd3RJSEpsWVdSeU9qcHlaV0ZrWDJOemRpZ25hSFIwY0hNNkx5OXlZWGN1WjJsMGFIVmlkWE5sY21OdmJuUmxiblF1WTI5dEwzSm1iM0prWVhSaGMyTnBaVzVqWlM5MGFXUjVkSFZsYzJSaGVTOXRZV2x1TDJSaGRHRXZNakF5TXk4eU1ESXpMVEEyTFRJd0wyUmhlVjl3WVhKMGMxOXRZWEF1WTNOMkp5bGNibHh1WUdCZ0luMD0gLS0+XG5cbmBgYHJcbnVmb19zaWdodGluZ3MgPC0gcmVhZHI6OnJlYWRfY3N2KCdodHRwczovL3Jhdy5naXRodWJ1c2VyY29udGVudC5jb20vcmZvcmRhdGFzY2llbmNlL3RpZHl0dWVzZGF5L21haW4vZGF0YS8yMDIzLzIwMjMtMDYtMjAvdWZvX3NpZ2h0aW5ncy5jc3YnKVxucGxhY2VzIDwtIHJlYWRyOjpyZWFkX2NzdignaHR0cHM6Ly9yYXcuZ2l0aHVidXNlcmNvbnRlbnQuY29tL3Jmb3JkYXRhc2NpZW5jZS90aWR5dHVlc2RheS9tYWluL2RhdGEvMjAyMy8yMDIzLTA2LTIwL3BsYWNlcy5jc3YnKVxuZGF5X3BhcnRzX21hcCA8LSByZWFkcjo6cmVhZF9jc3YoJ2h0dHBzOi8vcmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbS9yZm9yZGF0YXNjaWVuY2UvdGlkeXR1ZXNkYXkvbWFpbi9kYXRhLzIwMjMvMjAyMy0wNi0yMC9kYXlfcGFydHNfbWFwLmNzdicpXG5cbmBgYFxuXG48IS0tIHJuYi1zb3VyY2UtZW5kIC0tPlxuIn0= -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxudWZvX3NpZ2h0aW5ncyA8LSByZWFkcjo6cmVhZF9jc3YoJ2h0dHBzOi8vcmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbS9yZm9yZGF0YXNjaWVuY2UvdGlkeXR1ZXNkYXkvbWFpbi9kYXRhLzIwMjMvMjAyMy0wNi0yMC91Zm9fc2lnaHRpbmdzLmNzdicpXG5wbGFjZXMgPC0gcmVhZHI6OnJlYWRfY3N2KCdodHRwczovL3Jhdy5naXRodWJ1c2VyY29udGVudC5jb20vcmZvcmRhdGFzY2llbmNlL3RpZHl0dWVzZGF5L21haW4vZGF0YS8yMDIzLzIwMjMtMDYtMjAvcGxhY2VzLmNzdicpXG5kYXlfcGFydHNfbWFwIDwtIHJlYWRyOjpyZWFkX2NzdignaHR0cHM6Ly9yYXcuZ2l0aHVidXNlcmNvbnRlbnQuY29tL3Jmb3JkYXRhc2NpZW5jZS90aWR5dHVlc2RheS9tYWluL2RhdGEvMjAyMy8yMDIzLTA2LTIwL2RheV9wYXJ0c19tYXAuY3N2JylcblxuYGBgIn0= -->

```r
ufo_sightings <- readr::read_csv('https://raw.githubusercontent.com/rfordatascience/tidytuesday/main/data/2023/2023-06-20/ufo_sightings.csv')
places <- readr::read_csv('https://raw.githubusercontent.com/rfordatascience/tidytuesday/main/data/2023/2023-06-20/places.csv')
day_parts_map <- readr::read_csv('https://raw.githubusercontent.com/rfordatascience/tidytuesday/main/data/2023/2023-06-20/day_parts_map.csv')

```

<!-- rnb-source-end -->


<!-- rnb-output-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->


Checking if files have been loaded correctly


<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-output-begin eyJkYXRhIjoiXG48IS0tIHJuYi1zb3VyY2UtYmVnaW4gZXlKa1lYUmhJam9pWUdCZ2NseHVhR1ZoWkNoMVptOWZjMmxuYUhScGJtZHpLVnh1YUdWaFpDaHdiR0ZqWlhNcFhHNW9aV0ZrS0dSaGVWOXdZWEowYzE5dFlYQXBYRzVnWUdBaWZRPT0gLS0+XG5cbmBgYHJcbmhlYWQodWZvX3NpZ2h0aW5ncylcbmhlYWQocGxhY2VzKVxuaGVhZChkYXlfcGFydHNfbWFwKVxuYGBgXG5cbjwhLS0gcm5iLXNvdXJjZS1lbmQgLS0+XG4ifQ== -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxuaGVhZCh1Zm9fc2lnaHRpbmdzKVxuaGVhZChwbGFjZXMpXG5oZWFkKGRheV9wYXJ0c19tYXApXG5gYGAifQ== -->

```r
head(ufo_sightings)
head(places)
head(day_parts_map)
```

<!-- rnb-source-end -->


<!-- rnb-output-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->


Saving data in respective files


<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-output-begin eyJkYXRhIjoiXG48IS0tIHJuYi1zb3VyY2UtYmVnaW4gZXlKa1lYUmhJam9pWUdCZ2NseHVaR2x5TG1OeVpXRjBaU2hvWlhKbEtGd2laR0YwWVZ3aUxDQmNJakl3TWpOY0lpd2dYQ0l5TURJekxUQTJMVEl3WENJcExDQnlaV04xY25OcGRtVWdQU0JVVWxWRkxDQnphRzkzVjJGeWJtbHVaM01nUFNCR1FVeFRSU2xjYmx4dWQzSnBkR1ZmWTNOMktIVm1iMTl6YVdkb2RHbHVaM01zSUdobGNtVW9YQ0prWVhSaFhDSXNJRndpTWpBeU0xd2lMQ0JjSWpJd01qTXRNRFl0TWpCY0lpd2dYQ0oxWm05ZmMybG5hSFJwYm1kekxtTnpkbHdpS1NsY2JuZHlhWFJsWDJOemRpaHdiR0ZqWlhNc0lHaGxjbVVvWENKa1lYUmhYQ0lzSUZ3aU1qQXlNMXdpTENCY0lqSXdNak10TURZdE1qQmNJaXdnWENKd2JHRmpaWE11WTNOMlhDSXBLVnh1ZDNKcGRHVmZZM04yS0dSaGVWOXdZWEowYzE5dFlYQXNJR2hsY21Vb1hDSmtZWFJoWENJc0lGd2lNakF5TTF3aUxDQmNJakl3TWpNdE1EWXRNakJjSWl3Z1hDSmtZWGxmY0dGeWRITmZiV0Z3TG1OemRsd2lLU2xjYm1CZ1lDSjkgLS0+XG5cbmBgYHJcbmRpci5jcmVhdGUoaGVyZShcImRhdGFcIiwgXCIyMDIzXCIsIFwiMjAyMy0wNi0yMFwiKSwgcmVjdXJzaXZlID0gVFJVRSwgc2hvd1dhcm5pbmdzID0gRkFMU0UpXG5cbndyaXRlX2Nzdih1Zm9fc2lnaHRpbmdzLCBoZXJlKFwiZGF0YVwiLCBcIjIwMjNcIiwgXCIyMDIzLTA2LTIwXCIsIFwidWZvX3NpZ2h0aW5ncy5jc3ZcIikpXG53cml0ZV9jc3YocGxhY2VzLCBoZXJlKFwiZGF0YVwiLCBcIjIwMjNcIiwgXCIyMDIzLTA2LTIwXCIsIFwicGxhY2VzLmNzdlwiKSlcbndyaXRlX2NzdihkYXlfcGFydHNfbWFwLCBoZXJlKFwiZGF0YVwiLCBcIjIwMjNcIiwgXCIyMDIzLTA2LTIwXCIsIFwiZGF5X3BhcnRzX21hcC5jc3ZcIikpXG5gYGBcblxuPCEtLSBybmItc291cmNlLWVuZCAtLT5cbiJ9 -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxuZGlyLmNyZWF0ZShoZXJlKFwiZGF0YVwiLCBcIjIwMjNcIiwgXCIyMDIzLTA2LTIwXCIpLCByZWN1cnNpdmUgPSBUUlVFLCBzaG93V2FybmluZ3MgPSBGQUxTRSlcblxud3JpdGVfY3N2KHVmb19zaWdodGluZ3MsIGhlcmUoXCJkYXRhXCIsIFwiMjAyM1wiLCBcIjIwMjMtMDYtMjBcIiwgXCJ1Zm9fc2lnaHRpbmdzLmNzdlwiKSlcbndyaXRlX2NzdihwbGFjZXMsIGhlcmUoXCJkYXRhXCIsIFwiMjAyM1wiLCBcIjIwMjMtMDYtMjBcIiwgXCJwbGFjZXMuY3N2XCIpKVxud3JpdGVfY3N2KGRheV9wYXJ0c19tYXAsIGhlcmUoXCJkYXRhXCIsIFwiMjAyM1wiLCBcIjIwMjMtMDYtMjBcIiwgXCJkYXlfcGFydHNfbWFwLmNzdlwiKSlcbmBgYCJ9 -->

```r
dir.create(here("data", "2023", "2023-06-20"), recursive = TRUE, showWarnings = FALSE)

write_csv(ufo_sightings, here("data", "2023", "2023-06-20", "ufo_sightings.csv"))
write_csv(places, here("data", "2023", "2023-06-20", "places.csv"))
write_csv(day_parts_map, here("data", "2023", "2023-06-20", "day_parts_map.csv"))
```

<!-- rnb-source-end -->


<!-- rnb-output-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->


Checking missing data


<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-output-begin eyJkYXRhIjoiXG48IS0tIHJuYi1zb3VyY2UtYmVnaW4gZXlKa1lYUmhJam9pWUdCZ2NseHVaMnhwYlhCelpTaDFabTlmYzJsbmFIUnBibWR6S1Z4dVoyeHBiWEJ6WlNod2JHRmpaWE1wWEc1bmJHbHRjSE5sS0dSaGVWOXdZWEowYzE5dFlYQXBYRzVnWUdBaWZRPT0gLS0+XG5cbmBgYHJcbmdsaW1wc2UodWZvX3NpZ2h0aW5ncylcbmdsaW1wc2UocGxhY2VzKVxuZ2xpbXBzZShkYXlfcGFydHNfbWFwKVxuYGBgXG5cbjwhLS0gcm5iLXNvdXJjZS1lbmQgLS0+XG4ifQ== -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxuZ2xpbXBzZSh1Zm9fc2lnaHRpbmdzKVxuZ2xpbXBzZShwbGFjZXMpXG5nbGltcHNlKGRheV9wYXJ0c19tYXApXG5gYGAifQ== -->

```r
glimpse(ufo_sightings)
glimpse(places)
glimpse(day_parts_map)
```

<!-- rnb-source-end -->


<!-- rnb-output-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->



<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-output-begin eyJkYXRhIjoiXG48IS0tIHJuYi1zb3VyY2UtYmVnaW4gZXlKa1lYUmhJam9pWUdCZ2NseHVZMjlzVTNWdGN5aHBjeTV1WVNoMVptOWZjMmxuYUhScGJtZHpLU2xjYm1CZ1lDSjkgLS0+XG5cbmBgYHJcbmNvbFN1bXMoaXMubmEodWZvX3NpZ2h0aW5ncykpXG5gYGBcblxuPCEtLSBybmItc291cmNlLWVuZCAtLT5cbiJ9 -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxuY29sU3Vtcyhpcy5uYSh1Zm9fc2lnaHRpbmdzKSlcbmBgYCJ9 -->

```r
colSums(is.na(ufo_sightings))
```

<!-- rnb-source-end -->


<!-- rnb-output-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->



<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-output-begin eyJkYXRhIjoiXG48IS0tIHJuYi1zb3VyY2UtYmVnaW4gZXlKa1lYUmhJam9pWUdCZ2NseHVZMjlzVTNWdGN5aHBjeTV1WVNod2JHRmpaWE1wS1Z4dVlHQmdJbjA9IC0tPlxuXG5gYGByXG5jb2xTdW1zKGlzLm5hKHBsYWNlcykpXG5gYGBcblxuPCEtLSBybmItc291cmNlLWVuZCAtLT5cbiJ9 -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxuY29sU3Vtcyhpcy5uYShwbGFjZXMpKVxuYGBgIn0= -->

```r
colSums(is.na(places))
```

<!-- rnb-source-end -->


<!-- rnb-output-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->



<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-output-begin eyJkYXRhIjoiXG48IS0tIHJuYi1zb3VyY2UtYmVnaW4gZXlKa1lYUmhJam9pWUdCZ2NseHVZMjlzVTNWdGN5aHBjeTV1WVNoa1lYbGZjR0Z5ZEhOZmJXRndLU2xjYm1CZ1lDSjkgLS0+XG5cbmBgYHJcbmNvbFN1bXMoaXMubmEoZGF5X3BhcnRzX21hcCkpXG5gYGBcblxuPCEtLSBybmItc291cmNlLWVuZCAtLT5cbiJ9 -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxuY29sU3Vtcyhpcy5uYShkYXlfcGFydHNfbWFwKSlcbmBgYCJ9 -->

```r
colSums(is.na(day_parts_map))
```

<!-- rnb-source-end -->


<!-- rnb-output-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->


Summary of the data


<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-output-begin eyJkYXRhIjoiXG48IS0tIHJuYi1zb3VyY2UtYmVnaW4gZXlKa1lYUmhJam9pWUdCZ2NseHVaR2x0S0hWbWIxOXphV2RvZEdsdVozTXBYRzV6ZFcxdFlYSjVLSFZtYjE5emFXZG9kR2x1WjNNcFhHNWdZR0FpZlE9PSAtLT5cblxuYGBgclxuZGltKHVmb19zaWdodGluZ3MpXG5zdW1tYXJ5KHVmb19zaWdodGluZ3MpXG5gYGBcblxuPCEtLSBybmItc291cmNlLWVuZCAtLT5cbiJ9 -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxuZGltKHVmb19zaWdodGluZ3MpXG5zdW1tYXJ5KHVmb19zaWdodGluZ3MpXG5gYGAifQ== -->

```r
dim(ufo_sightings)
summary(ufo_sightings)
```

<!-- rnb-source-end -->


<!-- rnb-output-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->



<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-output-begin eyJkYXRhIjoiXG48IS0tIHJuYi1zb3VyY2UtYmVnaW4gZXlKa1lYUmhJam9pWUdCZ2NseHVaR2x0S0hCc1lXTmxjeWxjYm5OMWJXMWhjbmtvY0d4aFkyVnpLVnh1WUdCZ0luMD0gLS0+XG5cbmBgYHJcbmRpbShwbGFjZXMpXG5zdW1tYXJ5KHBsYWNlcylcbmBgYFxuXG48IS0tIHJuYi1zb3VyY2UtZW5kIC0tPlxuIn0= -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxuZGltKHBsYWNlcylcbnN1bW1hcnkocGxhY2VzKVxuYGBgIn0= -->

```r
dim(places)
summary(places)
```

<!-- rnb-source-end -->


<!-- rnb-output-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->



<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-output-begin eyJkYXRhIjoiXG48IS0tIHJuYi1zb3VyY2UtYmVnaW4gZXlKa1lYUmhJam9pWUdCZ2NseHVaR2x0S0dSaGVWOXdZWEowYzE5dFlYQXBYRzV6ZFcxdFlYSjVLR1JoZVY5d1lYSjBjMTl0WVhBcFhHNWdZR0FpZlE9PSAtLT5cblxuYGBgclxuZGltKGRheV9wYXJ0c19tYXApXG5zdW1tYXJ5KGRheV9wYXJ0c19tYXApXG5gYGBcblxuPCEtLSBybmItc291cmNlLWVuZCAtLT5cbiJ9 -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxuZGltKGRheV9wYXJ0c19tYXApXG5zdW1tYXJ5KGRheV9wYXJ0c19tYXApXG5gYGAifQ== -->

```r
dim(day_parts_map)
summary(day_parts_map)
```

<!-- rnb-source-end -->


<!-- rnb-output-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->



<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-output-begin eyJkYXRhIjoiXG48IS0tIHJuYi1zb3VyY2UtYmVnaW4gZXlKa1lYUmhJam9pWUdCZ2NseHVjM1Z0S0dSMWNHeHBZMkYwWldRb2RXWnZYM05wWjJoMGFXNW5jeWtwWEc1Z1lHQWlmUT09IC0tPlxuXG5gYGByXG5zdW0oZHVwbGljYXRlZCh1Zm9fc2lnaHRpbmdzKSlcbmBgYFxuXG48IS0tIHJuYi1zb3VyY2UtZW5kIC0tPlxuIn0= -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxuc3VtKGR1cGxpY2F0ZWQodWZvX3NpZ2h0aW5ncykpXG5gYGAifQ== -->

```r
sum(duplicated(ufo_sightings))
```

<!-- rnb-source-end -->


<!-- rnb-output-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->


# Handle missing observations (fill in or remove them), correct errors, etc.

Due to the large size of the file on which the analysis is performed, only some of the data was used for visual representation.


<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-output-begin eyJkYXRhIjoiXG48IS0tIHJuYi1zb3VyY2UtYmVnaW4gZXlKa1lYUmhJam9pWUdCZ2NseHViR2xpY21GeWVTaDBhV1I1ZG1WeWMyVXBJQ0JjYm14cFluSmhjbmtvYUdWeVpTa2dJQ0FnSUNBZ1hHNXNhV0p5WVhKNUtIZHBkR2h5S1Z4dWJHbGljbUZ5ZVNodVlXNXBZWElwWEc1Z1lHQWlmUT09IC0tPlxuXG5gYGByXG5saWJyYXJ5KHRpZHl2ZXJzZSkgIFxubGlicmFyeShoZXJlKSAgICAgICBcbmxpYnJhcnkod2l0aHIpXG5saWJyYXJ5KG5hbmlhcilcbmBgYFxuXG48IS0tIHJuYi1zb3VyY2UtZW5kIC0tPlxuIn0= -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxubGlicmFyeSh0aWR5dmVyc2UpICBcbmxpYnJhcnkoaGVyZSkgICAgICAgXG5saWJyYXJ5KHdpdGhyKVxubGlicmFyeShuYW5pYXIpXG5gYGAifQ== -->

```r
library(tidyverse)  
library(here)       
library(withr)
library(naniar)
```

<!-- rnb-source-end -->


<!-- rnb-output-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->


## UFO sightings

Contains detailed records of UFO sightings, including the date and time of the event, location (city, state, country), reported shape and duration, and a short summary. It also indicates whether an image is available and includes the estimated part of the day the sighting occurred.


<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-output-begin eyJkYXRhIjoiXG48IS0tIHJuYi1zb3VyY2UtYmVnaW4gZXlKa1lYUmhJam9pWUdCZ2NseHVkV1p2WDNOcFoyaDBhVzVuY3lBbFBpVWdYRzRnSUdSd2JIbHlPanB6YkdsalpWOXpZVzF3YkdVb2JpQTlJREV3TURBcElDVStKU0JjYmlBZ2RtbHpYMjFwYzNNb1kyeDFjM1JsY2lBOUlGUlNWVVVzSUhOdmNuUmZiV2x6Y3lBOUlGUlNWVVVwWEc1Z1lHQWlmUT09IC0tPlxuXG5gYGByXG51Zm9fc2lnaHRpbmdzICU+JSBcbiAgZHBseXI6OnNsaWNlX3NhbXBsZShuID0gMTAwMCkgJT4lIFxuICB2aXNfbWlzcyhjbHVzdGVyID0gVFJVRSwgc29ydF9taXNzID0gVFJVRSlcbmBgYFxuXG48IS0tIHJuYi1zb3VyY2UtZW5kIC0tPlxuIn0= -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxudWZvX3NpZ2h0aW5ncyAlPiUgXG4gIGRwbHlyOjpzbGljZV9zYW1wbGUobiA9IDEwMDApICU+JSBcbiAgdmlzX21pc3MoY2x1c3RlciA9IFRSVUUsIHNvcnRfbWlzcyA9IFRSVUUpXG5gYGAifQ== -->

```r
ufo_sightings %>% 
  dplyr::slice_sample(n = 1000) %>% 
  vis_miss(cluster = TRUE, sort_miss = TRUE)
```

<!-- rnb-source-end -->


<!-- rnb-output-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->


Replacing missing data with most frequent entry and saving cleaned data


<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-output-begin eyJkYXRhIjoiXG48IS0tIHJuYi1zb3VyY2UtYmVnaW4gZXlKa1lYUmhJam9pWUdCZ2NseHViVzl6ZEY5amIyMXRiMjVmWkdGNVgzQmhjblFnUEMwZ2RXWnZYM05wWjJoMGFXNW5jeUFsUGlWY2JpQWdZMjkxYm5Rb1pHRjVYM0JoY25RcElDVStKVnh1SUNCaGNuSmhibWRsS0dSbGMyTW9iaWtwSUNVK0pWeHVJQ0J6YkdsalpTZ3hLU0FsUGlWY2JpQWdjSFZzYkNoa1lYbGZjR0Z5ZENsY2JseHViVzl6ZEY5amIyMXRiMjVmYzJoaGNHVWdQQzBnZFdadlgzTnBaMmgwYVc1bmN5QWxQaVZjYmlBZ1kyOTFiblFvYzJoaGNHVXBJQ1UrSlZ4dUlDQmhjbkpoYm1kbEtHUmxjMk1vYmlrcElDVStKVnh1SUNCemJHbGpaU2d4S1NBbFBpVmNiaUFnY0hWc2JDaHphR0Z3WlNsY2JseHVkV1p2WDJOc1pXRnVJRHd0SUhWbWIxOXphV2RvZEdsdVozTWdKVDRsWEc0Z0lHMTFkR0YwWlNoY2JpQWdJQ0JrWVhsZmNHRnlkQ0E5SUdsbVpXeHpaU2hwY3k1dVlTaGtZWGxmY0dGeWRDa3NJRzF2YzNSZlkyOXRiVzl1WDJSaGVWOXdZWEowTENCa1lYbGZjR0Z5ZENrc1hHNGdJQ0FnYzJoaGNHVWdQU0JwWm1Wc2MyVW9hWE11Ym1Fb2MyaGhjR1VwTENCdGIzTjBYMk52YlcxdmJsOXphR0Z3WlN3Z2MyaGhjR1VwWEc0Z0lDbGNibHh1ZDNKcGRHVmZZM04yS0hWbWIxOWpiR1ZoYml3Z2FHVnlaU2hjSW1SaGRHRmNJaXdnWENJeU1ESXpYQ0lzSUZ3aU1qQXlNeTB3TmkweU1Gd2lMQ0JjSW5WbWIxOWpiR1ZoYmk1amMzWmNJaWtwWEc1Y2JtQmdZQ0o5IC0tPlxuXG5gYGByXG5tb3N0X2NvbW1vbl9kYXlfcGFydCA8LSB1Zm9fc2lnaHRpbmdzICU+JVxuICBjb3VudChkYXlfcGFydCkgJT4lXG4gIGFycmFuZ2UoZGVzYyhuKSkgJT4lXG4gIHNsaWNlKDEpICU+JVxuICBwdWxsKGRheV9wYXJ0KVxuXG5tb3N0X2NvbW1vbl9zaGFwZSA8LSB1Zm9fc2lnaHRpbmdzICU+JVxuICBjb3VudChzaGFwZSkgJT4lXG4gIGFycmFuZ2UoZGVzYyhuKSkgJT4lXG4gIHNsaWNlKDEpICU+JVxuICBwdWxsKHNoYXBlKVxuXG51Zm9fY2xlYW4gPC0gdWZvX3NpZ2h0aW5ncyAlPiVcbiAgbXV0YXRlKFxuICAgIGRheV9wYXJ0ID0gaWZlbHNlKGlzLm5hKGRheV9wYXJ0KSwgbW9zdF9jb21tb25fZGF5X3BhcnQsIGRheV9wYXJ0KSxcbiAgICBzaGFwZSA9IGlmZWxzZShpcy5uYShzaGFwZSksIG1vc3RfY29tbW9uX3NoYXBlLCBzaGFwZSlcbiAgKVxuXG53cml0ZV9jc3YodWZvX2NsZWFuLCBoZXJlKFwiZGF0YVwiLCBcIjIwMjNcIiwgXCIyMDIzLTA2LTIwXCIsIFwidWZvX2NsZWFuLmNzdlwiKSlcblxuYGBgXG5cbjwhLS0gcm5iLXNvdXJjZS1lbmQgLS0+XG4ifQ== -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxubW9zdF9jb21tb25fZGF5X3BhcnQgPC0gdWZvX3NpZ2h0aW5ncyAlPiVcbiAgY291bnQoZGF5X3BhcnQpICU+JVxuICBhcnJhbmdlKGRlc2MobikpICU+JVxuICBzbGljZSgxKSAlPiVcbiAgcHVsbChkYXlfcGFydClcblxubW9zdF9jb21tb25fc2hhcGUgPC0gdWZvX3NpZ2h0aW5ncyAlPiVcbiAgY291bnQoc2hhcGUpICU+JVxuICBhcnJhbmdlKGRlc2MobikpICU+JVxuICBzbGljZSgxKSAlPiVcbiAgcHVsbChzaGFwZSlcblxudWZvX2NsZWFuIDwtIHVmb19zaWdodGluZ3MgJT4lXG4gIG11dGF0ZShcbiAgICBkYXlfcGFydCA9IGlmZWxzZShpcy5uYShkYXlfcGFydCksIG1vc3RfY29tbW9uX2RheV9wYXJ0LCBkYXlfcGFydCksXG4gICAgc2hhcGUgPSBpZmVsc2UoaXMubmEoc2hhcGUpLCBtb3N0X2NvbW1vbl9zaGFwZSwgc2hhcGUpXG4gIClcblxud3JpdGVfY3N2KHVmb19jbGVhbiwgaGVyZShcImRhdGFcIiwgXCIyMDIzXCIsIFwiMjAyMy0wNi0yMFwiLCBcInVmb19jbGVhbi5jc3ZcIikpXG5cbmBgYCJ9 -->

```r
most_common_day_part <- ufo_sightings %>%
  count(day_part) %>%
  arrange(desc(n)) %>%
  slice(1) %>%
  pull(day_part)

most_common_shape <- ufo_sightings %>%
  count(shape) %>%
  arrange(desc(n)) %>%
  slice(1) %>%
  pull(shape)

ufo_clean <- ufo_sightings %>%
  mutate(
    day_part = ifelse(is.na(day_part), most_common_day_part, day_part),
    shape = ifelse(is.na(shape), most_common_shape, shape)
  )

write_csv(ufo_clean, here("data", "2023", "2023-06-20", "ufo_clean.csv"))

```

<!-- rnb-source-end -->


<!-- rnb-output-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->



<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-output-begin eyJkYXRhIjoiXG48IS0tIHJuYi1zb3VyY2UtYmVnaW4gZXlKa1lYUmhJam9pWUdCZ2NseHVkV1p2WDJOc1pXRnVJQ1UrSlNCY2JpQWdaSEJzZVhJNk9uTnNhV05sWDNOaGJYQnNaU2h1SUQwZ01UQXdNQ2tnSlQ0bElGeHVJQ0IyYVhOZmJXbHpjeWhqYkhWemRHVnlJRDBnVkZKVlJTd2djMjl5ZEY5dGFYTnpJRDBnVkZKVlJTbGNibUJnWUNKOSAtLT5cblxuYGBgclxudWZvX2NsZWFuICU+JSBcbiAgZHBseXI6OnNsaWNlX3NhbXBsZShuID0gMTAwMCkgJT4lIFxuICB2aXNfbWlzcyhjbHVzdGVyID0gVFJVRSwgc29ydF9taXNzID0gVFJVRSlcbmBgYFxuXG48IS0tIHJuYi1zb3VyY2UtZW5kIC0tPlxuIn0= -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxudWZvX2NsZWFuICU+JSBcbiAgZHBseXI6OnNsaWNlX3NhbXBsZShuID0gMTAwMCkgJT4lIFxuICB2aXNfbWlzcyhjbHVzdGVyID0gVFJVRSwgc29ydF9taXNzID0gVFJVRSlcbmBgYCJ9 -->

```r
ufo_clean %>% 
  dplyr::slice_sample(n = 1000) %>% 
  vis_miss(cluster = TRUE, sort_miss = TRUE)
```

<!-- rnb-source-end -->


<!-- rnb-output-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->


## Places

Contains geographic and demographic information about cities where UFO sightings occurred. It includes city names, alternate names, state, country, and country code, as well as latitude, longitude, timezone, population, and elevation.


<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-output-begin eyJkYXRhIjoiXG48IS0tIHJuYi1zb3VyY2UtYmVnaW4gZXlKa1lYUmhJam9pWUdCZ2NseHVjR3hoWTJWeklDVStKU0JjYmlBZ1pIQnNlWEk2T25Oc2FXTmxYM05oYlhCc1pTaHVJRDBnTVRBd01Da2dKVDRsSUZ4dUlDQjJhWE5mYldsemN5aGpiSFZ6ZEdWeUlEMGdWRkpWUlN3Z2MyOXlkRjl0YVhOeklEMGdWRkpWUlNsY2JtQmdZQ0o5IC0tPlxuXG5gYGByXG5wbGFjZXMgJT4lIFxuICBkcGx5cjo6c2xpY2Vfc2FtcGxlKG4gPSAxMDAwKSAlPiUgXG4gIHZpc19taXNzKGNsdXN0ZXIgPSBUUlVFLCBzb3J0X21pc3MgPSBUUlVFKVxuYGBgXG5cbjwhLS0gcm5iLXNvdXJjZS1lbmQgLS0+XG4ifQ== -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxucGxhY2VzICU+JSBcbiAgZHBseXI6OnNsaWNlX3NhbXBsZShuID0gMTAwMCkgJT4lIFxuICB2aXNfbWlzcyhjbHVzdGVyID0gVFJVRSwgc29ydF9taXNzID0gVFJVRSlcbmBgYCJ9 -->

```r
places %>% 
  dplyr::slice_sample(n = 1000) %>% 
  vis_miss(cluster = TRUE, sort_miss = TRUE)
```

<!-- rnb-source-end -->


<!-- rnb-output-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->


Replacing missing city name with empty string, elevation with mean value and saving cleaned data


<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-output-begin eyJkYXRhIjoiXG48IS0tIHJuYi1zb3VyY2UtYmVnaW4gZXlKa1lYUmhJam9pWUdCZ2NseHVjR3hoWTJWelgyTnNaV0Z1SUR3dElIQnNZV05sY3lBbFBpVmNiaUFnYlhWMFlYUmxLRnh1SUNBZ0lHRnNkR1Z5Ym1GMFpWOWphWFI1WDI1aGJXVnpJRDBnYVdabGJITmxLR2x6TG01aEtHRnNkR1Z5Ym1GMFpWOWphWFI1WDI1aGJXVnpLU3dnWENJZ1hDSXNJR0ZzZEdWeWJtRjBaVjlqYVhSNVgyNWhiV1Z6S1Z4dUlDQXBYRzVjYm0xbFpHbGhibDlsYkdWMllYUnBiMjRnUEMwZ2JXVmthV0Z1S0hCc1lXTmxjeVJsYkdWMllYUnBiMjVmYlN3Z2JtRXVjbTBnUFNCVVVsVkZLVnh1WEc1d2JHRmpaWE5mWTJ4bFlXNGdQQzBnY0d4aFkyVnpYMk5zWldGdUlDVStKVnh1SUNCdGRYUmhkR1VvWEc0Z0lDQWdaV3hsZG1GMGFXOXVYMjBnUFNCcFptVnNjMlVvYVhNdWJtRW9aV3hsZG1GMGFXOXVYMjBwTENCdFpXUnBZVzVmWld4bGRtRjBhVzl1TENCbGJHVjJZWFJwYjI1ZmJTbGNiaUFnS1Z4dVhHNTNjbWwwWlY5amMzWW9jR3hoWTJWelgyTnNaV0Z1TENCb1pYSmxLRndpWkdGMFlWd2lMQ0JjSWpJd01qTmNJaXdnWENJeU1ESXpMVEEyTFRJd1hDSXNJRndpY0d4aFkyVnpYMk5zWldGdUxtTnpkbHdpS1NsY2JtQmdZQ0o5IC0tPlxuXG5gYGByXG5wbGFjZXNfY2xlYW4gPC0gcGxhY2VzICU+JVxuICBtdXRhdGUoXG4gICAgYWx0ZXJuYXRlX2NpdHlfbmFtZXMgPSBpZmVsc2UoaXMubmEoYWx0ZXJuYXRlX2NpdHlfbmFtZXMpLCBcIiBcIiwgYWx0ZXJuYXRlX2NpdHlfbmFtZXMpXG4gIClcblxubWVkaWFuX2VsZXZhdGlvbiA8LSBtZWRpYW4ocGxhY2VzJGVsZXZhdGlvbl9tLCBuYS5ybSA9IFRSVUUpXG5cbnBsYWNlc19jbGVhbiA8LSBwbGFjZXNfY2xlYW4gJT4lXG4gIG11dGF0ZShcbiAgICBlbGV2YXRpb25fbSA9IGlmZWxzZShpcy5uYShlbGV2YXRpb25fbSksIG1lZGlhbl9lbGV2YXRpb24sIGVsZXZhdGlvbl9tKVxuICApXG5cbndyaXRlX2NzdihwbGFjZXNfY2xlYW4sIGhlcmUoXCJkYXRhXCIsIFwiMjAyM1wiLCBcIjIwMjMtMDYtMjBcIiwgXCJwbGFjZXNfY2xlYW4uY3N2XCIpKVxuYGBgXG5cbjwhLS0gcm5iLXNvdXJjZS1lbmQgLS0+XG4ifQ== -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxucGxhY2VzX2NsZWFuIDwtIHBsYWNlcyAlPiVcbiAgbXV0YXRlKFxuICAgIGFsdGVybmF0ZV9jaXR5X25hbWVzID0gaWZlbHNlKGlzLm5hKGFsdGVybmF0ZV9jaXR5X25hbWVzKSwgXCIgXCIsIGFsdGVybmF0ZV9jaXR5X25hbWVzKVxuICApXG5cbm1lZGlhbl9lbGV2YXRpb24gPC0gbWVkaWFuKHBsYWNlcyRlbGV2YXRpb25fbSwgbmEucm0gPSBUUlVFKVxuXG5wbGFjZXNfY2xlYW4gPC0gcGxhY2VzX2NsZWFuICU+JVxuICBtdXRhdGUoXG4gICAgZWxldmF0aW9uX20gPSBpZmVsc2UoaXMubmEoZWxldmF0aW9uX20pLCBtZWRpYW5fZWxldmF0aW9uLCBlbGV2YXRpb25fbSlcbiAgKVxuXG53cml0ZV9jc3YocGxhY2VzX2NsZWFuLCBoZXJlKFwiZGF0YVwiLCBcIjIwMjNcIiwgXCIyMDIzLTA2LTIwXCIsIFwicGxhY2VzX2NsZWFuLmNzdlwiKSlcbmBgYCJ9 -->

```r
places_clean <- places %>%
  mutate(
    alternate_city_names = ifelse(is.na(alternate_city_names), " ", alternate_city_names)
  )

median_elevation <- median(places$elevation_m, na.rm = TRUE)

places_clean <- places_clean %>%
  mutate(
    elevation_m = ifelse(is.na(elevation_m), median_elevation, elevation_m)
  )

write_csv(places_clean, here("data", "2023", "2023-06-20", "places_clean.csv"))
```

<!-- rnb-source-end -->


<!-- rnb-output-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->



<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-output-begin eyJkYXRhIjoiXG48IS0tIHJuYi1zb3VyY2UtYmVnaW4gZXlKa1lYUmhJam9pWUdCZ2NseHVjR3hoWTJWelgyTnNaV0Z1SUNVK0pTQmNiaUFnWkhCc2VYSTZPbk5zYVdObFgzTmhiWEJzWlNodUlEMGdNVEF3TUNrZ0pUNGxJRnh1SUNCMmFYTmZiV2x6Y3loamJIVnpkR1Z5SUQwZ1ZGSlZSU3dnYzI5eWRGOXRhWE56SUQwZ1ZGSlZSU2xjYm1CZ1lDSjkgLS0+XG5cbmBgYHJcbnBsYWNlc19jbGVhbiAlPiUgXG4gIGRwbHlyOjpzbGljZV9zYW1wbGUobiA9IDEwMDApICU+JSBcbiAgdmlzX21pc3MoY2x1c3RlciA9IFRSVUUsIHNvcnRfbWlzcyA9IFRSVUUpXG5gYGBcblxuPCEtLSBybmItc291cmNlLWVuZCAtLT5cbiJ9 -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxucGxhY2VzX2NsZWFuICU+JSBcbiAgZHBseXI6OnNsaWNlX3NhbXBsZShuID0gMTAwMCkgJT4lIFxuICB2aXNfbWlzcyhjbHVzdGVyID0gVFJVRSwgc29ydF9taXNzID0gVFJVRSlcbmBgYCJ9 -->

```r
places_clean %>% 
  dplyr::slice_sample(n = 1000) %>% 
  vis_miss(cluster = TRUE, sort_miss = TRUE)
```

<!-- rnb-source-end -->


<!-- rnb-output-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->


## Day parts map

Provides astronomical timing data based on geographic coordinates and date. It includes rounded latitude, longitude, and date, along with precise UTC times for sunrise, sunset, solar noon, and different twilight phases (astronomical, nautical, and civil).


<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-output-begin eyJkYXRhIjoiXG48IS0tIHJuYi1zb3VyY2UtYmVnaW4gZXlKa1lYUmhJam9pWUdCZ2NseHVaR0Y1WDNCaGNuUnpYMjFoY0NBbFBpVWdYRzRnSUdSd2JIbHlPanB6YkdsalpWOXpZVzF3YkdVb2JpQTlJREV3TURBcElDVStKU0JjYmlBZ2RtbHpYMjFwYzNNb1kyeDFjM1JsY2lBOUlGUlNWVVVzSUhOdmNuUmZiV2x6Y3lBOUlGUlNWVVVwWEc1Z1lHQWlmUT09IC0tPlxuXG5gYGByXG5kYXlfcGFydHNfbWFwICU+JSBcbiAgZHBseXI6OnNsaWNlX3NhbXBsZShuID0gMTAwMCkgJT4lIFxuICB2aXNfbWlzcyhjbHVzdGVyID0gVFJVRSwgc29ydF9taXNzID0gVFJVRSlcbmBgYFxuXG48IS0tIHJuYi1zb3VyY2UtZW5kIC0tPlxuIn0= -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxuZGF5X3BhcnRzX21hcCAlPiUgXG4gIGRwbHlyOjpzbGljZV9zYW1wbGUobiA9IDEwMDApICU+JSBcbiAgdmlzX21pc3MoY2x1c3RlciA9IFRSVUUsIHNvcnRfbWlzcyA9IFRSVUUpXG5gYGAifQ== -->

```r
day_parts_map %>% 
  dplyr::slice_sample(n = 1000) %>% 
  vis_miss(cluster = TRUE, sort_miss = TRUE)
```

<!-- rnb-source-end -->


<!-- rnb-output-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->


Replacing missing data with mean value and saving cleaned data


<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-output-begin eyJkYXRhIjoiXG48IS0tIHJuYi1zb3VyY2UtYmVnaW4gZXlKa1lYUmhJam9pWUdCZ2NseHVaR0Y1WDNCaGNuUnpYMk5zWldGdUlEd3RJR1JoZVY5d1lYSjBjMTl0WVhBZ0pUNGxYRzRnSUcxMWRHRjBaU2hjYmlBZ0lDQmhjM1J5YjI1dmJXbGpZV3hmZEhkcGJHbG5hSFJmWW1WbmFXNGdQU0JwWm1Wc2MyVW9YRzRnSUNBZ0lDQnBjeTV1WVNoaGMzUnliMjV2YldsallXeGZkSGRwYkdsbmFIUmZZbVZuYVc0cExGeHVJQ0FnSUNBZ2JXVmthV0Z1S0dGemRISnZibTl0YVdOaGJGOTBkMmxzYVdkb2RGOWlaV2RwYml3Z2JtRXVjbTBnUFNCVVVsVkZLU3hjYmlBZ0lDQWdJR0Z6ZEhKdmJtOXRhV05oYkY5MGQybHNhV2RvZEY5aVpXZHBibHh1SUNBZ0lDa3NYRzRnSUNBZ1lYTjBjbTl1YjIxcFkyRnNYM1IzYVd4cFoyaDBYMlZ1WkNBOUlHbG1aV3h6WlNoY2JpQWdJQ0FnSUdsekxtNWhLR0Z6ZEhKdmJtOXRhV05oYkY5MGQybHNhV2RvZEY5bGJtUXBMRnh1SUNBZ0lDQWdiV1ZrYVdGdUtHRnpkSEp2Ym05dGFXTmhiRjkwZDJsc2FXZG9kRjlsYm1Rc0lHNWhMbkp0SUQwZ1ZGSlZSU2tzWEc0Z0lDQWdJQ0JoYzNSeWIyNXZiV2xqWVd4ZmRIZHBiR2xuYUhSZlpXNWtYRzRnSUNBZ0tWeHVJQ0FwWEc1Y2JuZHlhWFJsWDJOemRpaGtZWGxmY0dGeWRITmZZMnhsWVc0c0lHaGxjbVVvWENKa1lYUmhYQ0lzSUZ3aU1qQXlNMXdpTENCY0lqSXdNak10TURZdE1qQmNJaXdnWENKa1lYbGZjR0Z5ZEhOZlkyeGxZVzR1WTNOMlhDSXBLVnh1WUdCZ0luMD0gLS0+XG5cbmBgYHJcbmRheV9wYXJ0c19jbGVhbiA8LSBkYXlfcGFydHNfbWFwICU+JVxuICBtdXRhdGUoXG4gICAgYXN0cm9ub21pY2FsX3R3aWxpZ2h0X2JlZ2luID0gaWZlbHNlKFxuICAgICAgaXMubmEoYXN0cm9ub21pY2FsX3R3aWxpZ2h0X2JlZ2luKSxcbiAgICAgIG1lZGlhbihhc3Ryb25vbWljYWxfdHdpbGlnaHRfYmVnaW4sIG5hLnJtID0gVFJVRSksXG4gICAgICBhc3Ryb25vbWljYWxfdHdpbGlnaHRfYmVnaW5cbiAgICApLFxuICAgIGFzdHJvbm9taWNhbF90d2lsaWdodF9lbmQgPSBpZmVsc2UoXG4gICAgICBpcy5uYShhc3Ryb25vbWljYWxfdHdpbGlnaHRfZW5kKSxcbiAgICAgIG1lZGlhbihhc3Ryb25vbWljYWxfdHdpbGlnaHRfZW5kLCBuYS5ybSA9IFRSVUUpLFxuICAgICAgYXN0cm9ub21pY2FsX3R3aWxpZ2h0X2VuZFxuICAgIClcbiAgKVxuXG53cml0ZV9jc3YoZGF5X3BhcnRzX2NsZWFuLCBoZXJlKFwiZGF0YVwiLCBcIjIwMjNcIiwgXCIyMDIzLTA2LTIwXCIsIFwiZGF5X3BhcnRzX2NsZWFuLmNzdlwiKSlcbmBgYFxuXG48IS0tIHJuYi1zb3VyY2UtZW5kIC0tPlxuIn0= -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxuZGF5X3BhcnRzX2NsZWFuIDwtIGRheV9wYXJ0c19tYXAgJT4lXG4gIG11dGF0ZShcbiAgICBhc3Ryb25vbWljYWxfdHdpbGlnaHRfYmVnaW4gPSBpZmVsc2UoXG4gICAgICBpcy5uYShhc3Ryb25vbWljYWxfdHdpbGlnaHRfYmVnaW4pLFxuICAgICAgbWVkaWFuKGFzdHJvbm9taWNhbF90d2lsaWdodF9iZWdpbiwgbmEucm0gPSBUUlVFKSxcbiAgICAgIGFzdHJvbm9taWNhbF90d2lsaWdodF9iZWdpblxuICAgICksXG4gICAgYXN0cm9ub21pY2FsX3R3aWxpZ2h0X2VuZCA9IGlmZWxzZShcbiAgICAgIGlzLm5hKGFzdHJvbm9taWNhbF90d2lsaWdodF9lbmQpLFxuICAgICAgbWVkaWFuKGFzdHJvbm9taWNhbF90d2lsaWdodF9lbmQsIG5hLnJtID0gVFJVRSksXG4gICAgICBhc3Ryb25vbWljYWxfdHdpbGlnaHRfZW5kXG4gICAgKVxuICApXG5cbndyaXRlX2NzdihkYXlfcGFydHNfY2xlYW4sIGhlcmUoXCJkYXRhXCIsIFwiMjAyM1wiLCBcIjIwMjMtMDYtMjBcIiwgXCJkYXlfcGFydHNfY2xlYW4uY3N2XCIpKVxuYGBgIn0= -->

```r
day_parts_clean <- day_parts_map %>%
  mutate(
    astronomical_twilight_begin = ifelse(
      is.na(astronomical_twilight_begin),
      median(astronomical_twilight_begin, na.rm = TRUE),
      astronomical_twilight_begin
    ),
    astronomical_twilight_end = ifelse(
      is.na(astronomical_twilight_end),
      median(astronomical_twilight_end, na.rm = TRUE),
      astronomical_twilight_end
    )
  )

write_csv(day_parts_clean, here("data", "2023", "2023-06-20", "day_parts_clean.csv"))
```

<!-- rnb-source-end -->


<!-- rnb-output-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->



<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-output-begin eyJkYXRhIjoiXG48IS0tIHJuYi1zb3VyY2UtYmVnaW4gZXlKa1lYUmhJam9pWUdCZ2NseHVaR0Y1WDNCaGNuUnpYMk5zWldGdUlDVStKU0JjYmlBZ1pIQnNlWEk2T25Oc2FXTmxYM05oYlhCc1pTaHVJRDBnTVRBd01Da2dKVDRsSUZ4dUlDQjJhWE5mYldsemN5aGpiSFZ6ZEdWeUlEMGdWRkpWUlN3Z2MyOXlkRjl0YVhOeklEMGdWRkpWUlNsY2JtQmdZQ0o5IC0tPlxuXG5gYGByXG5kYXlfcGFydHNfY2xlYW4gJT4lIFxuICBkcGx5cjo6c2xpY2Vfc2FtcGxlKG4gPSAxMDAwKSAlPiUgXG4gIHZpc19taXNzKGNsdXN0ZXIgPSBUUlVFLCBzb3J0X21pc3MgPSBUUlVFKVxuYGBgXG5cbjwhLS0gcm5iLXNvdXJjZS1lbmQgLS0+XG4ifQ== -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxuZGF5X3BhcnRzX2NsZWFuICU+JSBcbiAgZHBseXI6OnNsaWNlX3NhbXBsZShuID0gMTAwMCkgJT4lIFxuICB2aXNfbWlzcyhjbHVzdGVyID0gVFJVRSwgc29ydF9taXNzID0gVFJVRSlcbmBgYCJ9 -->

```r
day_parts_clean %>% 
  dplyr::slice_sample(n = 1000) %>% 
  vis_miss(cluster = TRUE, sort_miss = TRUE)
```

<!-- rnb-source-end -->


<!-- rnb-output-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->


# Adjust the data format to meet the requirements of your analysis


<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-output-begin eyJkYXRhIjoiXG48IS0tIHJuYi1zb3VyY2UtYmVnaW4gZXlKa1lYUmhJam9pWUdCZ2NseHVYRzUxWm05ZmJXOWtaV3hmWkdGMFlTQThMU0IxWm05ZlkyeGxZVzRnSlQ0bFhHNGdJR1pwYkhSbGNpZ2hhWE11Ym1Fb2MyaGhjR1VwTENBaGFYTXVibUVvY21Wd2IzSjBaV1JmWkhWeVlYUnBiMjRwTENBaGFYTXVibUVvYzNWdGJXRnllU2twWEc1Y2JseHVjR3hoWTJWelgyMXZaR1ZzWDJSaGRHRWdQQzBnY0d4aFkyVnpYMk5zWldGdUlDVStKVnh1SUNCbWFXeDBaWElvSVdsekxtNWhLR0ZzZEdWeWJtRjBaVjlqYVhSNVgyNWhiV1Z6S1NsY2JseHVZR0JnSW4wPSAtLT5cblxuYGBgclxuXG51Zm9fbW9kZWxfZGF0YSA8LSB1Zm9fY2xlYW4gJT4lXG4gIGZpbHRlcighaXMubmEoc2hhcGUpLCAhaXMubmEocmVwb3J0ZWRfZHVyYXRpb24pLCAhaXMubmEoc3VtbWFyeSkpXG5cblxucGxhY2VzX21vZGVsX2RhdGEgPC0gcGxhY2VzX2NsZWFuICU+JVxuICBmaWx0ZXIoIWlzLm5hKGFsdGVybmF0ZV9jaXR5X25hbWVzKSlcblxuYGBgXG5cbjwhLS0gcm5iLXNvdXJjZS1lbmQgLS0+XG4ifQ== -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxuXG51Zm9fbW9kZWxfZGF0YSA8LSB1Zm9fY2xlYW4gJT4lXG4gIGZpbHRlcighaXMubmEoc2hhcGUpLCAhaXMubmEocmVwb3J0ZWRfZHVyYXRpb24pLCAhaXMubmEoc3VtbWFyeSkpXG5cblxucGxhY2VzX21vZGVsX2RhdGEgPC0gcGxhY2VzX2NsZWFuICU+JVxuICBmaWx0ZXIoIWlzLm5hKGFsdGVybmF0ZV9jaXR5X25hbWVzKSlcblxuYGBgIn0= -->

```r

ufo_model_data <- ufo_clean %>%
  filter(!is.na(shape), !is.na(reported_duration), !is.na(summary))


places_model_data <- places_clean %>%
  filter(!is.na(alternate_city_names))

```

<!-- rnb-source-end -->


<!-- rnb-output-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->



<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-output-begin eyJkYXRhIjoiXG48IS0tIHJuYi1zb3VyY2UtYmVnaW4gZXlKa1lYUmhJam9pWUdCZ2NseHVaMnhwYlhCelpTaDFabTlmYlc5a1pXeGZaR0YwWVNsY2JtZHNhVzF3YzJVb2NHeGhZMlZ6WDIxdlpHVnNYMlJoZEdFcFhHNW5iR2x0Y0hObEtHUmhlVjl3WVhKMGMxOWpiR1ZoYmlsY2JtQmdZQ0o5IC0tPlxuXG5gYGByXG5nbGltcHNlKHVmb19tb2RlbF9kYXRhKVxuZ2xpbXBzZShwbGFjZXNfbW9kZWxfZGF0YSlcbmdsaW1wc2UoZGF5X3BhcnRzX2NsZWFuKVxuYGBgXG5cbjwhLS0gcm5iLXNvdXJjZS1lbmQgLS0+XG4ifQ== -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxuZ2xpbXBzZSh1Zm9fbW9kZWxfZGF0YSlcbmdsaW1wc2UocGxhY2VzX21vZGVsX2RhdGEpXG5nbGltcHNlKGRheV9wYXJ0c19jbGVhbilcbmBgYCJ9 -->

```r
glimpse(ufo_model_data)
glimpse(places_model_data)
glimpse(day_parts_clean)
```

<!-- rnb-source-end -->


<!-- rnb-output-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->



<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-output-begin eyJkYXRhIjoiXG48IS0tIHJuYi1zb3VyY2UtYmVnaW4gZXlKa1lYUmhJam9pWUdCZ2NseHVkM0pwZEdWZlkzTjJLSFZtYjE5dGIyUmxiRjlrWVhSaExDQm9aWEpsS0Z3aVpHRjBZVndpTENCY0lqSXdNak5jSWl3Z1hDSXlNREl6TFRBMkxUSXdYQ0lzSUZ3aWRXWnZYMjF2WkdWc1gyUmhkR0V1WTNOMlhDSXBLVnh1ZDNKcGRHVmZZM04yS0hCc1lXTmxjMTl0YjJSbGJGOWtZWFJoTENCb1pYSmxLRndpWkdGMFlWd2lMQ0JjSWpJd01qTmNJaXdnWENJeU1ESXpMVEEyTFRJd1hDSXNJRndpY0d4aFkyVnpYMjF2WkdWc1gyUmhkR0V1WTNOMlhDSXBLVnh1WUdCZ0luMD0gLS0+XG5cbmBgYHJcbndyaXRlX2Nzdih1Zm9fbW9kZWxfZGF0YSwgaGVyZShcImRhdGFcIiwgXCIyMDIzXCIsIFwiMjAyMy0wNi0yMFwiLCBcInVmb19tb2RlbF9kYXRhLmNzdlwiKSlcbndyaXRlX2NzdihwbGFjZXNfbW9kZWxfZGF0YSwgaGVyZShcImRhdGFcIiwgXCIyMDIzXCIsIFwiMjAyMy0wNi0yMFwiLCBcInBsYWNlc19tb2RlbF9kYXRhLmNzdlwiKSlcbmBgYFxuXG48IS0tIHJuYi1zb3VyY2UtZW5kIC0tPlxuIn0= -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxud3JpdGVfY3N2KHVmb19tb2RlbF9kYXRhLCBoZXJlKFwiZGF0YVwiLCBcIjIwMjNcIiwgXCIyMDIzLTA2LTIwXCIsIFwidWZvX21vZGVsX2RhdGEuY3N2XCIpKVxud3JpdGVfY3N2KHBsYWNlc19tb2RlbF9kYXRhLCBoZXJlKFwiZGF0YVwiLCBcIjIwMjNcIiwgXCIyMDIzLTA2LTIwXCIsIFwicGxhY2VzX21vZGVsX2RhdGEuY3N2XCIpKVxuYGBgIn0= -->

```r
write_csv(ufo_model_data, here("data", "2023", "2023-06-20", "ufo_model_data.csv"))
write_csv(places_model_data, here("data", "2023", "2023-06-20", "places_model_data.csv"))
```

<!-- rnb-source-end -->


<!-- rnb-output-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->


# Add new columns to each dataframe


<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-output-begin eyJkYXRhIjoiXG48IS0tIHJuYi1zb3VyY2UtYmVnaW4gZXlKa1lYUmhJam9pWUdCZ2NseHViR2xpY21GeWVTaHNkV0p5YVdSaGRHVXBYRzVzYVdKeVlYSjVLR1J3YkhseUtWeHViR2xpY21GeWVTaG9iWE1wWEc1Z1lHQWlmUT09IC0tPlxuXG5gYGByXG5saWJyYXJ5KGx1YnJpZGF0ZSlcbmxpYnJhcnkoZHBseXIpXG5saWJyYXJ5KGhtcylcbmBgYFxuXG48IS0tIHJuYi1zb3VyY2UtZW5kIC0tPlxuIn0= -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxubGlicmFyeShsdWJyaWRhdGUpXG5saWJyYXJ5KGRwbHlyKVxubGlicmFyeShobXMpXG5gYGAifQ== -->

```r
library(lubridate)
library(dplyr)
library(hms)
```

<!-- rnb-source-end -->


<!-- rnb-output-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->


Adding new columns to "sightings" dataframe


<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-output-begin eyJkYXRhIjoiXG48IS0tIHJuYi1zb3VyY2UtYmVnaW4gZXlKa1lYUmhJam9pWUdCZ2NseHVkV1p2WDIxdlpHVnNYMlJoZEdGZmJYVjBZWFJsWkNBOExTQjFabTlmYlc5a1pXeGZaR0YwWVNBbFBpVmNiaUFnYlhWMFlYUmxLRnh1SUNBZ0lIbGxZWElnUFNCNVpXRnlLSEpsY0c5eWRHVmtYMlJoZEdWZmRHbHRaU2tzWEc0Z0lDQWdiVzl1ZEdnZ1BTQnRiMjUwYUNoeVpYQnZjblJsWkY5a1lYUmxYM1JwYldVcExGeHVJQ0FnSUhkbFpXdGtZWGtnUFNCM1pHRjVLSEpsY0c5eWRHVmtYMlJoZEdWZmRHbHRaU3dnYkdGaVpXd2dQU0JVVWxWRkxDQmhZbUp5SUQwZ1JrRk1VMFVzSUd4dlkyRnNaU0E5SUZ3aVExd2lLU3hjYmlBZ0lDQnBjMTkzWldWclpXNWtJRDBnZDJWbGEyUmhlU0FsYVc0bElHTW9YQ0pUWVhSY0lpd2dYQ0pUZFc1Y0lpa3NYRzRnSUNBZ1kyOTFiblJ5ZVY5MWNIQmxjaUE5SUhSdmRYQndaWElvWTI5MWJuUnllVjlqYjJSbEtTeGNiaUFnSUNCeVpYQnZjblJmYUc5MWNpQTlJR2h2ZFhJb2NtVndiM0owWldSZlpHRjBaVjkwYVcxbEtTeGNiaUFnSUNCamFYUjVYM04wWVhSbElEMGdjR0Z6ZEdVb1kybDBlU3dnYzNSaGRHVXNJSE5sY0NBOUlGd2lMQ0JjSWlrc1hHNGdJQ0FnY21Wd2IzSjBYMlJsYkdGNVgyUmhlWE1nUFNCaGN5NXVkVzFsY21saktHUnBabVowYVcxbEtIQnZjM1JsWkY5a1lYUmxMQ0JoY3k1RVlYUmxLSEpsY0c5eWRHVmtYMlJoZEdWZmRHbHRaU2tzSUhWdWFYUnpJRDBnWENKa1lYbHpYQ0lwS1Z4dUlDQXBYRzVjYm5WbWIxOXRiMlJsYkY5a1lYUmhYMjExZEdGMFpXUmNibUJnWUNKOSAtLT5cblxuYGBgclxudWZvX21vZGVsX2RhdGFfbXV0YXRlZCA8LSB1Zm9fbW9kZWxfZGF0YSAlPiVcbiAgbXV0YXRlKFxuICAgIHllYXIgPSB5ZWFyKHJlcG9ydGVkX2RhdGVfdGltZSksXG4gICAgbW9udGggPSBtb250aChyZXBvcnRlZF9kYXRlX3RpbWUpLFxuICAgIHdlZWtkYXkgPSB3ZGF5KHJlcG9ydGVkX2RhdGVfdGltZSwgbGFiZWwgPSBUUlVFLCBhYmJyID0gRkFMU0UsIGxvY2FsZSA9IFwiQ1wiKSxcbiAgICBpc193ZWVrZW5kID0gd2Vla2RheSAlaW4lIGMoXCJTYXRcIiwgXCJTdW5cIiksXG4gICAgY291bnRyeV91cHBlciA9IHRvdXBwZXIoY291bnRyeV9jb2RlKSxcbiAgICByZXBvcnRfaG91ciA9IGhvdXIocmVwb3J0ZWRfZGF0ZV90aW1lKSxcbiAgICBjaXR5X3N0YXRlID0gcGFzdGUoY2l0eSwgc3RhdGUsIHNlcCA9IFwiLCBcIiksXG4gICAgcmVwb3J0X2RlbGF5X2RheXMgPSBhcy5udW1lcmljKGRpZmZ0aW1lKHBvc3RlZF9kYXRlLCBhcy5EYXRlKHJlcG9ydGVkX2RhdGVfdGltZSksIHVuaXRzID0gXCJkYXlzXCIpKVxuICApXG5cbnVmb19tb2RlbF9kYXRhX211dGF0ZWRcbmBgYFxuXG48IS0tIHJuYi1zb3VyY2UtZW5kIC0tPlxuIn0= -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxudWZvX21vZGVsX2RhdGFfbXV0YXRlZCA8LSB1Zm9fbW9kZWxfZGF0YSAlPiVcbiAgbXV0YXRlKFxuICAgIHllYXIgPSB5ZWFyKHJlcG9ydGVkX2RhdGVfdGltZSksXG4gICAgbW9udGggPSBtb250aChyZXBvcnRlZF9kYXRlX3RpbWUpLFxuICAgIHdlZWtkYXkgPSB3ZGF5KHJlcG9ydGVkX2RhdGVfdGltZSwgbGFiZWwgPSBUUlVFLCBhYmJyID0gRkFMU0UsIGxvY2FsZSA9IFwiQ1wiKSxcbiAgICBpc193ZWVrZW5kID0gd2Vla2RheSAlaW4lIGMoXCJTYXRcIiwgXCJTdW5cIiksXG4gICAgY291bnRyeV91cHBlciA9IHRvdXBwZXIoY291bnRyeV9jb2RlKSxcbiAgICByZXBvcnRfaG91ciA9IGhvdXIocmVwb3J0ZWRfZGF0ZV90aW1lKSxcbiAgICBjaXR5X3N0YXRlID0gcGFzdGUoY2l0eSwgc3RhdGUsIHNlcCA9IFwiLCBcIiksXG4gICAgcmVwb3J0X2RlbGF5X2RheXMgPSBhcy5udW1lcmljKGRpZmZ0aW1lKHBvc3RlZF9kYXRlLCBhcy5EYXRlKHJlcG9ydGVkX2RhdGVfdGltZSksIHVuaXRzID0gXCJkYXlzXCIpKVxuICApXG5cbnVmb19tb2RlbF9kYXRhX211dGF0ZWRcbmBgYCJ9 -->

```r
ufo_model_data_mutated <- ufo_model_data %>%
  mutate(
    year = year(reported_date_time),
    month = month(reported_date_time),
    weekday = wday(reported_date_time, label = TRUE, abbr = FALSE, locale = "C"),
    is_weekend = weekday %in% c("Sat", "Sun"),
    country_upper = toupper(country_code),
    report_hour = hour(reported_date_time),
    city_state = paste(city, state, sep = ", "),
    report_delay_days = as.numeric(difftime(posted_date, as.Date(reported_date_time), units = "days"))
  )

ufo_model_data_mutated
```

<!-- rnb-source-end -->


<!-- rnb-output-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->


-   `year`: Extracts the year from the `reported_date_time`.
-   `month`: Extracts the month (1–12) from the report time-stamp.
-   `weekday`: Returns the weekday name from the date.
-   `is_weekend`: Logical column: `TRUE` if the day is Saturday or Sunday, `FALSE` otherwise.
-   `country_upper`: Converts the `country_code` to uppercase.
-   `report_hour`: Extracts the hour (0–23) from the report time-stamp.
-   `city_state`: Concatenates `city` and `state` into a single string.
-   `report_delay_days`: Calculates the delay in days between when the event was reported and when it was posted.

Adding new columns to "places" dataframe


<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-output-begin eyJkYXRhIjoiXG48IS0tIHJuYi1zb3VyY2UtYmVnaW4gZXlKa1lYUmhJam9pWUdCZ2NseHVjR3hoWTJWelgyMXZaR1ZzWDJSaGRHRmZiWFYwWVhSbFpDQThMU0J3YkdGalpYTmZiVzlrWld4ZlpHRjBZU0FsUGlWY2JpQWdiWFYwWVhSbEtGeHVJQ0FnSUdOcGRIbGZjM1JoZEdVZ1BTQndZWE4wWlNoamFYUjVMQ0J6ZEdGMFpTd2djMlZ3SUQwZ1hDSXNJRndpS1N4Y2JpQWdJQ0JwYzE5MWN5QTlJR052ZFc1MGNubGZZMjlrWlNBOVBTQmNJbFZUWENJc1hHNGdJQ0FnY0c5d2RXeGhkR2x2Ymw5c2IyY2dQU0JzYjJjeGNDaHdiM0IxYkdGMGFXOXVLU3hjYmlBZ0lDQm9aVzFwYzNCb1pYSmxJRDBnYVdabGJITmxLR3hoZEdsMGRXUmxJRDQ5SURBc0lGd2lUbTl5ZEdobGNtNWNJaXdnWENKVGIzVjBhR1Z5Ymx3aUtTeGNiaUFnSUNCcGMxOWpiMkZ6ZEdGc0lEMGdZV0p6S0d4dmJtZHBkSFZrWlNrZ1BDQTRNQ0I4SUdGaWN5aHNiMjVuYVhSMVpHVXBJRDRnTVRJd0xGeHVJQ0FnSUhCdmNGOWpZWFJsWjI5eWVTQTlJR05oYzJWZmQyaGxiaWhjYmlBZ0lDQWdJSEJ2Y0hWc1lYUnBiMjRnUENBeE1EQXdNQ0IrSUZ3aWMyMWhiR3hjSWl4Y2JpQWdJQ0FnSUhCdmNIVnNZWFJwYjI0Z1BDQXhNREF3TURBZ2ZpQmNJbTFsWkdsMWJWd2lMRnh1SUNBZ0lDQWdWRkpWUlNCK0lGd2liR0Z5WjJWY0lseHVJQ0FnSUNrc1hHNGdJQ0FnWld4bGRtRjBhVzl1WDJOaGRHVm5iM0o1SUQwZ1kyRnpaVjkzYUdWdUtGeHVJQ0FnSUNBZ2FYTXVibUVvWld4bGRtRjBhVzl1WDIwcElINGdYQ0oxYm10dWIzZHVYQ0lzWEc0Z0lDQWdJQ0JsYkdWMllYUnBiMjVmYlNBOElERXdNQ0IrSUZ3aWJHOTNYQ0lzWEc0Z0lDQWdJQ0JsYkdWMllYUnBiMjVmYlNBOElEVXdNQ0IrSUZ3aWJXVmthWFZ0WENJc1hHNGdJQ0FnSUNCVVVsVkZJSDRnWENKb2FXZG9YQ0pjYmlBZ0lDQXBMRnh1SUNBZ0lHNWhiV1ZmYkdWdVozUm9JRDBnYm1Ob1lYSW9ZMmwwZVNrc1hHNGdJQ0FnZEdsdFpYcHZibVZmWVhKbFlTQTlJSE5oY0hCc2VTaHpkSEp6Y0d4cGRDaDBhVzFsZW05dVpTd2dYQ0l2WENJcExDQmdXMkFzSURJcFhHNGdJQ2xjYmx4dWNHeGhZMlZ6WDIxdlpHVnNYMlJoZEdGZmJYVjBZWFJsWkZ4dVlHQmdJbjA9IC0tPlxuXG5gYGByXG5wbGFjZXNfbW9kZWxfZGF0YV9tdXRhdGVkIDwtIHBsYWNlc19tb2RlbF9kYXRhICU+JVxuICBtdXRhdGUoXG4gICAgY2l0eV9zdGF0ZSA9IHBhc3RlKGNpdHksIHN0YXRlLCBzZXAgPSBcIiwgXCIpLFxuICAgIGlzX3VzID0gY291bnRyeV9jb2RlID09IFwiVVNcIixcbiAgICBwb3B1bGF0aW9uX2xvZyA9IGxvZzFwKHBvcHVsYXRpb24pLFxuICAgIGhlbWlzcGhlcmUgPSBpZmVsc2UobGF0aXR1ZGUgPj0gMCwgXCJOb3J0aGVyblwiLCBcIlNvdXRoZXJuXCIpLFxuICAgIGlzX2NvYXN0YWwgPSBhYnMobG9uZ2l0dWRlKSA8IDgwIHwgYWJzKGxvbmdpdHVkZSkgPiAxMjAsXG4gICAgcG9wX2NhdGVnb3J5ID0gY2FzZV93aGVuKFxuICAgICAgcG9wdWxhdGlvbiA8IDEwMDAwIH4gXCJzbWFsbFwiLFxuICAgICAgcG9wdWxhdGlvbiA8IDEwMDAwMCB+IFwibWVkaXVtXCIsXG4gICAgICBUUlVFIH4gXCJsYXJnZVwiXG4gICAgKSxcbiAgICBlbGV2YXRpb25fY2F0ZWdvcnkgPSBjYXNlX3doZW4oXG4gICAgICBpcy5uYShlbGV2YXRpb25fbSkgfiBcInVua25vd25cIixcbiAgICAgIGVsZXZhdGlvbl9tIDwgMTAwIH4gXCJsb3dcIixcbiAgICAgIGVsZXZhdGlvbl9tIDwgNTAwIH4gXCJtZWRpdW1cIixcbiAgICAgIFRSVUUgfiBcImhpZ2hcIlxuICAgICksXG4gICAgbmFtZV9sZW5ndGggPSBuY2hhcihjaXR5KSxcbiAgICB0aW1lem9uZV9hcmVhID0gc2FwcGx5KHN0cnNwbGl0KHRpbWV6b25lLCBcIi9cIiksIGBbYCwgMilcbiAgKVxuXG5wbGFjZXNfbW9kZWxfZGF0YV9tdXRhdGVkXG5gYGBcblxuPCEtLSBybmItc291cmNlLWVuZCAtLT5cbiJ9 -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxucGxhY2VzX21vZGVsX2RhdGFfbXV0YXRlZCA8LSBwbGFjZXNfbW9kZWxfZGF0YSAlPiVcbiAgbXV0YXRlKFxuICAgIGNpdHlfc3RhdGUgPSBwYXN0ZShjaXR5LCBzdGF0ZSwgc2VwID0gXCIsIFwiKSxcbiAgICBpc191cyA9IGNvdW50cnlfY29kZSA9PSBcIlVTXCIsXG4gICAgcG9wdWxhdGlvbl9sb2cgPSBsb2cxcChwb3B1bGF0aW9uKSxcbiAgICBoZW1pc3BoZXJlID0gaWZlbHNlKGxhdGl0dWRlID49IDAsIFwiTm9ydGhlcm5cIiwgXCJTb3V0aGVyblwiKSxcbiAgICBpc19jb2FzdGFsID0gYWJzKGxvbmdpdHVkZSkgPCA4MCB8IGFicyhsb25naXR1ZGUpID4gMTIwLFxuICAgIHBvcF9jYXRlZ29yeSA9IGNhc2Vfd2hlbihcbiAgICAgIHBvcHVsYXRpb24gPCAxMDAwMCB+IFwic21hbGxcIixcbiAgICAgIHBvcHVsYXRpb24gPCAxMDAwMDAgfiBcIm1lZGl1bVwiLFxuICAgICAgVFJVRSB+IFwibGFyZ2VcIlxuICAgICksXG4gICAgZWxldmF0aW9uX2NhdGVnb3J5ID0gY2FzZV93aGVuKFxuICAgICAgaXMubmEoZWxldmF0aW9uX20pIH4gXCJ1bmtub3duXCIsXG4gICAgICBlbGV2YXRpb25fbSA8IDEwMCB+IFwibG93XCIsXG4gICAgICBlbGV2YXRpb25fbSA8IDUwMCB+IFwibWVkaXVtXCIsXG4gICAgICBUUlVFIH4gXCJoaWdoXCJcbiAgICApLFxuICAgIG5hbWVfbGVuZ3RoID0gbmNoYXIoY2l0eSksXG4gICAgdGltZXpvbmVfYXJlYSA9IHNhcHBseShzdHJzcGxpdCh0aW1lem9uZSwgXCIvXCIpLCBgW2AsIDIpXG4gIClcblxucGxhY2VzX21vZGVsX2RhdGFfbXV0YXRlZFxuYGBgIn0= -->

```r
places_model_data_mutated <- places_model_data %>%
  mutate(
    city_state = paste(city, state, sep = ", "),
    is_us = country_code == "US",
    population_log = log1p(population),
    hemisphere = ifelse(latitude >= 0, "Northern", "Southern"),
    is_coastal = abs(longitude) < 80 | abs(longitude) > 120,
    pop_category = case_when(
      population < 10000 ~ "small",
      population < 100000 ~ "medium",
      TRUE ~ "large"
    ),
    elevation_category = case_when(
      is.na(elevation_m) ~ "unknown",
      elevation_m < 100 ~ "low",
      elevation_m < 500 ~ "medium",
      TRUE ~ "high"
    ),
    name_length = nchar(city),
    timezone_area = sapply(strsplit(timezone, "/"), `[`, 2)
  )

places_model_data_mutated
```

<!-- rnb-source-end -->


<!-- rnb-output-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->


-   `city_state`: Combines `city` and `state` into a single string.
-   `is_us`: Logical value: `TRUE` if the location is in the United States else `FALSE`.
-   `population_log`: Log-transformed population.
-   `hemisphere`: `"Northern"` if latitude is ≥ 0, `"Southern"` otherwise.
-   `is_coastal`: Logical: `TRUE` if longitude is outside the range [80, 120] in absolute value — a rough coastal proxy.
-   `pop_category`: Categorizes places based on population: `"small"`, `"medium"`, or `"large"`.
-   `elevation_category`: Classifies elevation: `"low"` (\<100 m), `"medium"` (\<500 m), `"high"` (≥500 m), or `"unknown"` if NA.
-   `name_length`: The number of characters in the city name.
-   `timezone_area`: Extracts the second part of the timezone string.

Adding new columns to "day parts" dataframe


<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-output-begin eyJkYXRhIjoiXG48IS0tIHJuYi1zb3VyY2UtYmVnaW4gZXlKa1lYUmhJam9pWUdCZ2NseHVaR0Y1WDNCaGNuUnpYMjF2WkdWc1gyMTFkR0YwWldRZ1BDMGdaR0Y1WDNCaGNuUnpYMk5zWldGdUlDVStKVnh1SUNCdGRYUmhkR1VvWEc0Z0lDQWdaR0Y1YkdsbmFIUmZaSFZ5WVhScGIyNGdQU0JoY3k1dWRXMWxjbWxqS0hOMWJuTmxkQ0F0SUhOMWJuSnBjMlVzSUhWdWFYUnpJRDBnWENKelpXTnpYQ0lwTEZ4dUlDQWdJR2x6WDI1dmNuUm9aWEp1WDJobGJXbHpjR2hsY21VZ1BTQnliM1Z1WkdWa1gyeGhkQ0ErUFNBd0xGeHVJQ0FnSUhOMWJuSnBjMlZmYUc5MWNpQTlJR2h2ZFhJb2MzVnVjbWx6WlNrc1hHNGdJQ0FnYzNWdWMyVjBYMmh2ZFhJZ1BTQm9iM1Z5S0hOMWJuTmxkQ2tzWEc0Z0lDQWdhWE5mWkdGNVgzTm9iM0owSUQwZ1pHRjViR2xuYUhSZlpIVnlZWFJwYjI0Z1BDQXpOakF3TUN3Z0l5QnRibWxsYWlCdWFjVzhJREV3YUZ4dUlDQWdJSFIzYVd4cFoyaDBYMlIxY21GMGFXOXVJRDBnWVhNdWJuVnRaWEpwWXloaGMzUnliMjV2YldsallXeGZkSGRwYkdsbmFIUmZaVzVrSUMwZ1lYTjBjbTl1YjIxcFkyRnNYM1IzYVd4cFoyaDBYMkpsWjJsdUxDQjFibWwwY3lBOUlGd2ljMlZqYzF3aUtTeGNiaUFnSUNCcGMxOXNiMjVuWDNSM2FXeHBaMmgwSUQwZ2RIZHBiR2xuYUhSZlpIVnlZWFJwYjI0Z1BpQTFOREF3TENBaklERXVOV2hjYmlBZ0lDQnpkVzV5YVhObFgyMXBiblYwWlhNZ1BTQm9iM1Z5S0hOMWJuSnBjMlVwSUNvZ05qQWdLeUJ0YVc1MWRHVW9jM1Z1Y21selpTa3NYRzRnSUNBZ2MyOXNZWEpmYm05dmJsOXRhVzUxZEdWeklEMGdhRzkxY2loemIyeGhjbDl1YjI5dUtTQXFJRFl3SUNzZ2JXbHVkWFJsS0hOdmJHRnlYMjV2YjI0cExGeHVJQ0FnSUhOMWJuTmxkRjl0YVc1MWRHVnpJRDBnYUc5MWNpaHpkVzV6WlhRcElDb2dOakFnS3lCdGFXNTFkR1VvYzNWdWMyVjBLVnh1SUNBcFhHNWNibVJoZVY5d1lYSjBjMTl0YjJSbGJGOXRkWFJoZEdWa1hHNWdZR0FpZlE9PSAtLT5cblxuYGBgclxuZGF5X3BhcnRzX21vZGVsX211dGF0ZWQgPC0gZGF5X3BhcnRzX2NsZWFuICU+JVxuICBtdXRhdGUoXG4gICAgZGF5bGlnaHRfZHVyYXRpb24gPSBhcy5udW1lcmljKHN1bnNldCAtIHN1bnJpc2UsIHVuaXRzID0gXCJzZWNzXCIpLFxuICAgIGlzX25vcnRoZXJuX2hlbWlzcGhlcmUgPSByb3VuZGVkX2xhdCA+PSAwLFxuICAgIHN1bnJpc2VfaG91ciA9IGhvdXIoc3VucmlzZSksXG4gICAgc3Vuc2V0X2hvdXIgPSBob3VyKHN1bnNldCksXG4gICAgaXNfZGF5X3Nob3J0ID0gZGF5bGlnaHRfZHVyYXRpb24gPCAzNjAwMCwgIyBtbmllaiBuacW8IDEwaFxuICAgIHR3aWxpZ2h0X2R1cmF0aW9uID0gYXMubnVtZXJpYyhhc3Ryb25vbWljYWxfdHdpbGlnaHRfZW5kIC0gYXN0cm9ub21pY2FsX3R3aWxpZ2h0X2JlZ2luLCB1bml0cyA9IFwic2Vjc1wiKSxcbiAgICBpc19sb25nX3R3aWxpZ2h0ID0gdHdpbGlnaHRfZHVyYXRpb24gPiA1NDAwLCAjIDEuNWhcbiAgICBzdW5yaXNlX21pbnV0ZXMgPSBob3VyKHN1bnJpc2UpICogNjAgKyBtaW51dGUoc3VucmlzZSksXG4gICAgc29sYXJfbm9vbl9taW51dGVzID0gaG91cihzb2xhcl9ub29uKSAqIDYwICsgbWludXRlKHNvbGFyX25vb24pLFxuICAgIHN1bnNldF9taW51dGVzID0gaG91cihzdW5zZXQpICogNjAgKyBtaW51dGUoc3Vuc2V0KVxuICApXG5cbmRheV9wYXJ0c19tb2RlbF9tdXRhdGVkXG5gYGBcblxuPCEtLSBybmItc291cmNlLWVuZCAtLT5cbiJ9 -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxuZGF5X3BhcnRzX21vZGVsX211dGF0ZWQgPC0gZGF5X3BhcnRzX2NsZWFuICU+JVxuICBtdXRhdGUoXG4gICAgZGF5bGlnaHRfZHVyYXRpb24gPSBhcy5udW1lcmljKHN1bnNldCAtIHN1bnJpc2UsIHVuaXRzID0gXCJzZWNzXCIpLFxuICAgIGlzX25vcnRoZXJuX2hlbWlzcGhlcmUgPSByb3VuZGVkX2xhdCA+PSAwLFxuICAgIHN1bnJpc2VfaG91ciA9IGhvdXIoc3VucmlzZSksXG4gICAgc3Vuc2V0X2hvdXIgPSBob3VyKHN1bnNldCksXG4gICAgaXNfZGF5X3Nob3J0ID0gZGF5bGlnaHRfZHVyYXRpb24gPCAzNjAwMCwgIyBtbmllaiBuacW8IDEwaFxuICAgIHR3aWxpZ2h0X2R1cmF0aW9uID0gYXMubnVtZXJpYyhhc3Ryb25vbWljYWxfdHdpbGlnaHRfZW5kIC0gYXN0cm9ub21pY2FsX3R3aWxpZ2h0X2JlZ2luLCB1bml0cyA9IFwic2Vjc1wiKSxcbiAgICBpc19sb25nX3R3aWxpZ2h0ID0gdHdpbGlnaHRfZHVyYXRpb24gPiA1NDAwLCAjIDEuNWhcbiAgICBzdW5yaXNlX21pbnV0ZXMgPSBob3VyKHN1bnJpc2UpICogNjAgKyBtaW51dGUoc3VucmlzZSksXG4gICAgc29sYXJfbm9vbl9taW51dGVzID0gaG91cihzb2xhcl9ub29uKSAqIDYwICsgbWludXRlKHNvbGFyX25vb24pLFxuICAgIHN1bnNldF9taW51dGVzID0gaG91cihzdW5zZXQpICogNjAgKyBtaW51dGUoc3Vuc2V0KVxuICApXG5cbmRheV9wYXJ0c19tb2RlbF9tdXRhdGVkXG5gYGAifQ== -->

```r
day_parts_model_mutated <- day_parts_clean %>%
  mutate(
    daylight_duration = as.numeric(sunset - sunrise, units = "secs"),
    is_northern_hemisphere = rounded_lat >= 0,
    sunrise_hour = hour(sunrise),
    sunset_hour = hour(sunset),
    is_day_short = daylight_duration < 36000, # mniej niż 10h
    twilight_duration = as.numeric(astronomical_twilight_end - astronomical_twilight_begin, units = "secs"),
    is_long_twilight = twilight_duration > 5400, # 1.5h
    sunrise_minutes = hour(sunrise) * 60 + minute(sunrise),
    solar_noon_minutes = hour(solar_noon) * 60 + minute(solar_noon),
    sunset_minutes = hour(sunset) * 60 + minute(sunset)
  )

day_parts_model_mutated
```

<!-- rnb-source-end -->


<!-- rnb-output-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->


-   `daylight_duration`: The length of the day in seconds — difference between `sunset` and `sunrise`.
-   `is_northern_hemisphere`: Logical: `TRUE` if the location is in the Northern Hemisphere.
-   `sunrise_hour`: The hour (0–23) when the sun rises.
-   `sunset_hour`: The hour (0–23) when the sun sets.
-   `is_day_short`: Logical: `TRUE` if the day is shorter than 10 hours
-   `twilight_duration`: Duration of astronomical twilight in seconds — time between `astronomical_twilight_begin` and `end`.
-   `is_long_twilight`: Logical: `TRUE` if twilight duration is longer than 1.5 hours
-   `sunrise_minutes`: Sunrise time in total minutes from midnight.
-   `solar_noon_minutes`: Solar noon time in minutes from midnight.
-   `sunset_minutes`: Sunset time in minutes from midnight.


<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-output-begin eyJkYXRhIjoiXG48IS0tIHJuYi1zb3VyY2UtYmVnaW4gZXlKa1lYUmhJam9pWUdCZ2NseHVaMnhwYlhCelpTaDFabTlmYlc5a1pXeGZaR0YwWVY5dGRYUmhkR1ZrS1Z4dVoyeHBiWEJ6WlNod2JHRmpaWE5mYlc5a1pXeGZaR0YwWVY5dGRYUmhkR1ZrS1Z4dVoyeHBiWEJ6WlNoa1lYbGZjR0Z5ZEhOZlkyeGxZVzRwWEc1Z1lHQWlmUT09IC0tPlxuXG5gYGByXG5nbGltcHNlKHVmb19tb2RlbF9kYXRhX211dGF0ZWQpXG5nbGltcHNlKHBsYWNlc19tb2RlbF9kYXRhX211dGF0ZWQpXG5nbGltcHNlKGRheV9wYXJ0c19jbGVhbilcbmBgYFxuXG48IS0tIHJuYi1zb3VyY2UtZW5kIC0tPlxuIn0= -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxuZ2xpbXBzZSh1Zm9fbW9kZWxfZGF0YV9tdXRhdGVkKVxuZ2xpbXBzZShwbGFjZXNfbW9kZWxfZGF0YV9tdXRhdGVkKVxuZ2xpbXBzZShkYXlfcGFydHNfY2xlYW4pXG5gYGAifQ== -->

```r
glimpse(ufo_model_data_mutated)
glimpse(places_model_data_mutated)
glimpse(day_parts_clean)
```

<!-- rnb-source-end -->


<!-- rnb-output-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->


# Explore data with charts


<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-output-begin eyJkYXRhIjoiXG48IS0tIHJuYi1zb3VyY2UtYmVnaW4gZXlKa1lYUmhJam9pWUdCZ2NseHViR2xpY21GeWVTaG5aM0JzYjNReUtWeHViR2xpY21GeWVTaGtjR3g1Y2lsY2JteHBZbkpoY25rb2MyWXBYRzVzYVdKeVlYSjVLSEp1WVhSMWNtRnNaV0Z5ZEdncFhHNXNhV0p5WVhKNUtISnVZWFIxY21Gc1pXRnlkR2hrWVhSaEtWeHVZR0JnSW4wPSAtLT5cblxuYGBgclxubGlicmFyeShnZ3Bsb3QyKVxubGlicmFyeShkcGx5cilcbmxpYnJhcnkoc2YpXG5saWJyYXJ5KHJuYXR1cmFsZWFydGgpXG5saWJyYXJ5KHJuYXR1cmFsZWFydGhkYXRhKVxuYGBgXG5cbjwhLS0gcm5iLXNvdXJjZS1lbmQgLS0+XG4ifQ== -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxubGlicmFyeShnZ3Bsb3QyKVxubGlicmFyeShkcGx5cilcbmxpYnJhcnkoc2YpXG5saWJyYXJ5KHJuYXR1cmFsZWFydGgpXG5saWJyYXJ5KHJuYXR1cmFsZWFydGhkYXRhKVxuYGBgIn0= -->

```r
library(ggplot2)
library(dplyr)
library(sf)
library(rnaturalearth)
library(rnaturalearthdata)
```

<!-- rnb-source-end -->


<!-- rnb-output-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->


## Number of sightings per day

To observe the overall volume and evolution of UFO reports over time, and detect historical patterns or anomalies.


<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-output-begin eyJkYXRhIjoiXG48IS0tIHJuYi1zb3VyY2UtYmVnaW4gZXlKa1lYUmhJam9pWUdCZ2NseHVkV1p2WDIxdlpHVnNYMlJoZEdGZmJYVjBZWFJsWkNBbFBpVmNiaUFnWTI5MWJuUW9aR0YwWlNBOUlHRnpMa1JoZEdVb2NtVndiM0owWldSZlpHRjBaVjkwYVcxbEtTa2dKVDRsWEc0Z0lHZG5jR3h2ZENoaFpYTW9lQ0E5SUdSaGRHVXNJSGtnUFNCdUtTa2dLMXh1SUNCblpXOXRYMnhwYm1Vb1kyOXNiM0lnUFNCY0luTjBaV1ZzWW14MVpWd2lLU0FyWEc0Z0lHeGhZbk1vZEdsMGJHVWdQU0JjSWs1MWJXSmxjaUJ2WmlCemFXZG9kR2x1WjNNZ2NHVnlJR1JoZVZ3aUxDQjRJRDBnWENKRVlYUmxYQ0lzSUhrZ1BTQmNJazUxYldKbGNpQnZaaUJ6YVdkb2RHbHVaM05jSWlsY2JtQmdZQ0o5IC0tPlxuXG5gYGByXG51Zm9fbW9kZWxfZGF0YV9tdXRhdGVkICU+JVxuICBjb3VudChkYXRlID0gYXMuRGF0ZShyZXBvcnRlZF9kYXRlX3RpbWUpKSAlPiVcbiAgZ2dwbG90KGFlcyh4ID0gZGF0ZSwgeSA9IG4pKSArXG4gIGdlb21fbGluZShjb2xvciA9IFwic3RlZWxibHVlXCIpICtcbiAgbGFicyh0aXRsZSA9IFwiTnVtYmVyIG9mIHNpZ2h0aW5ncyBwZXIgZGF5XCIsIHggPSBcIkRhdGVcIiwgeSA9IFwiTnVtYmVyIG9mIHNpZ2h0aW5nc1wiKVxuYGBgXG5cbjwhLS0gcm5iLXNvdXJjZS1lbmQgLS0+XG4ifQ== -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxudWZvX21vZGVsX2RhdGFfbXV0YXRlZCAlPiVcbiAgY291bnQoZGF0ZSA9IGFzLkRhdGUocmVwb3J0ZWRfZGF0ZV90aW1lKSkgJT4lXG4gIGdncGxvdChhZXMoeCA9IGRhdGUsIHkgPSBuKSkgK1xuICBnZW9tX2xpbmUoY29sb3IgPSBcInN0ZWVsYmx1ZVwiKSArXG4gIGxhYnModGl0bGUgPSBcIk51bWJlciBvZiBzaWdodGluZ3MgcGVyIGRheVwiLCB4ID0gXCJEYXRlXCIsIHkgPSBcIk51bWJlciBvZiBzaWdodGluZ3NcIilcbmBgYCJ9 -->

```r
ufo_model_data_mutated %>%
  count(date = as.Date(reported_date_time)) %>%
  ggplot(aes(x = date, y = n)) +
  geom_line(color = "steelblue") +
  labs(title = "Number of sightings per day", x = "Date", y = "Number of sightings")
```

<!-- rnb-source-end -->


<!-- rnb-output-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->


**Interpretation:**

The chart shows daily UFO sightings over time. Sightings were rare before 1960, gradually increased through the 1990s, and peaked between 2000 and 2015. After 2015, the number of reports declined sharply. This suggests that UFO sightings may be influenced by media, public interest, or reporting practices.

## Annual Trend of UFO Sightings

Helps visualize how the frequency of UFO sightings has changed over time, revealing historical peaks and long-term trends.


<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-output-begin eyJkYXRhIjoiXG48IS0tIHJuYi1zb3VyY2UtYmVnaW4gZXlKa1lYUmhJam9pWUdCZ2NseHVkV1p2WDIxdlpHVnNYMlJoZEdGZmJYVjBZWFJsWkNBbFBpVmNiaUFnYlhWMFlYUmxLSGxsWVhJZ1BTQnNkV0p5YVdSaGRHVTZPbmxsWVhJb2NtVndiM0owWldSZlpHRjBaVjkwYVcxbEtTa2dKVDRsWEc0Z0lHTnZkVzUwS0hsbFlYSXBJQ1UrSlZ4dUlDQm5aM0JzYjNRb1lXVnpLSGdnUFNCNVpXRnlMQ0I1SUQwZ2Jpa3BJQ3RjYmlBZ1oyVnZiVjlzYVc1bEtHTnZiRzl5SUQwZ1hDSmtZWEpyWW14MVpWd2lLU0FyWEc0Z0lHZGxiMjFmYzIxdmIzUm9LSE5sSUQwZ1JrRk1VMFVzSUdOdmJHOXlJRDBnWENKeVpXUmNJaXdnYldWMGFHOWtJRDBnWENKc2IyVnpjMXdpS1NBclhHNGdJR3hoWW5Nb1hHNGdJQ0FnZEdsMGJHVWdQU0JjSWtGdWJuVmhiQ0JVY21WdVpDQnZaaUJWUms4Z1UybG5hSFJwYm1kelhDSXNYRzRnSUNBZ2VDQTlJRndpV1dWaGNsd2lMRnh1SUNBZ0lIa2dQU0JjSWs1MWJXSmxjaUJ2WmlCVGFXZG9kR2x1WjNOY0lseHVJQ0FwSUN0Y2JpQWdkR2hsYldWZmJXbHVhVzFoYkNncFhHNWNibUJnWUNKOSAtLT5cblxuYGBgclxudWZvX21vZGVsX2RhdGFfbXV0YXRlZCAlPiVcbiAgbXV0YXRlKHllYXIgPSBsdWJyaWRhdGU6OnllYXIocmVwb3J0ZWRfZGF0ZV90aW1lKSkgJT4lXG4gIGNvdW50KHllYXIpICU+JVxuICBnZ3Bsb3QoYWVzKHggPSB5ZWFyLCB5ID0gbikpICtcbiAgZ2VvbV9saW5lKGNvbG9yID0gXCJkYXJrYmx1ZVwiKSArXG4gIGdlb21fc21vb3RoKHNlID0gRkFMU0UsIGNvbG9yID0gXCJyZWRcIiwgbWV0aG9kID0gXCJsb2Vzc1wiKSArXG4gIGxhYnMoXG4gICAgdGl0bGUgPSBcIkFubnVhbCBUcmVuZCBvZiBVRk8gU2lnaHRpbmdzXCIsXG4gICAgeCA9IFwiWWVhclwiLFxuICAgIHkgPSBcIk51bWJlciBvZiBTaWdodGluZ3NcIlxuICApICtcbiAgdGhlbWVfbWluaW1hbCgpXG5cbmBgYFxuXG48IS0tIHJuYi1zb3VyY2UtZW5kIC0tPlxuIn0= -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxudWZvX21vZGVsX2RhdGFfbXV0YXRlZCAlPiVcbiAgbXV0YXRlKHllYXIgPSBsdWJyaWRhdGU6OnllYXIocmVwb3J0ZWRfZGF0ZV90aW1lKSkgJT4lXG4gIGNvdW50KHllYXIpICU+JVxuICBnZ3Bsb3QoYWVzKHggPSB5ZWFyLCB5ID0gbikpICtcbiAgZ2VvbV9saW5lKGNvbG9yID0gXCJkYXJrYmx1ZVwiKSArXG4gIGdlb21fc21vb3RoKHNlID0gRkFMU0UsIGNvbG9yID0gXCJyZWRcIiwgbWV0aG9kID0gXCJsb2Vzc1wiKSArXG4gIGxhYnMoXG4gICAgdGl0bGUgPSBcIkFubnVhbCBUcmVuZCBvZiBVRk8gU2lnaHRpbmdzXCIsXG4gICAgeCA9IFwiWWVhclwiLFxuICAgIHkgPSBcIk51bWJlciBvZiBTaWdodGluZ3NcIlxuICApICtcbiAgdGhlbWVfbWluaW1hbCgpXG5cbmBgYCJ9 -->

```r
ufo_model_data_mutated %>%
  mutate(year = lubridate::year(reported_date_time)) %>%
  count(year) %>%
  ggplot(aes(x = year, y = n)) +
  geom_line(color = "darkblue") +
  geom_smooth(se = FALSE, color = "red", method = "loess") +
  labs(
    title = "Annual Trend of UFO Sightings",
    x = "Year",
    y = "Number of Sightings"
  ) +
  theme_minimal()

```

<!-- rnb-source-end -->


<!-- rnb-output-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->


**Interpretation:**

The chart shows a clear rise in UFO sightings from the 1980s to around 2012, with a peak near 2014. After that, there’s a sharp decline in reports. The red loess curve highlights a long-term upward trend followed by a recent downward shift. This may reflect changes in reporting behavior, public interest, or data availability over time.

## Number of sightings depending on the day of the week

To investigate whether UFO sightings are more frequent on certain days, especially weekends when people are more likely to be outdoors.


<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-output-begin eyJkYXRhIjoiXG48IS0tIHJuYi1zb3VyY2UtYmVnaW4gZXlKa1lYUmhJam9pWUdCZ2NseHVkV1p2WDIxdlpHVnNYMlJoZEdGZmJYVjBZWFJsWkNBbFBpVmNiaUFnWTI5MWJuUW9kMlZsYTJSaGVTa2dKVDRsWEc0Z0lHZG5jR3h2ZENoaFpYTW9lQ0E5SUhkbFpXdGtZWGtzSUhrZ1BTQnVLU2tnSzF4dUlDQm5aVzl0WDJOdmJDaG1hV3hzSUQwZ1hDSnZjbUZ1WjJWY0lpa2dLMXh1SUNCc1lXSnpLSFJwZEd4bElEMGdYQ0pUYVdkb2RHbHVaM01nWkdWd1pXNWthVzVuSUc5dUlIUm9aU0JrWVhrZ2IyWWdkR2hsSUhkbFpXdGNJaXdnZUNBOUlGd2lSR0Y1SUc5bUlIUm9aU0IzWldWclhDSXNJSGtnUFNCY0lrNTFiV0psY2lCdlppQnphV2RvZEdsdVozTmNJaWxjYm1CZ1lDSjkgLS0+XG5cbmBgYHJcbnVmb19tb2RlbF9kYXRhX211dGF0ZWQgJT4lXG4gIGNvdW50KHdlZWtkYXkpICU+JVxuICBnZ3Bsb3QoYWVzKHggPSB3ZWVrZGF5LCB5ID0gbikpICtcbiAgZ2VvbV9jb2woZmlsbCA9IFwib3JhbmdlXCIpICtcbiAgbGFicyh0aXRsZSA9IFwiU2lnaHRpbmdzIGRlcGVuZGluZyBvbiB0aGUgZGF5IG9mIHRoZSB3ZWVrXCIsIHggPSBcIkRheSBvZiB0aGUgd2Vla1wiLCB5ID0gXCJOdW1iZXIgb2Ygc2lnaHRpbmdzXCIpXG5gYGBcblxuPCEtLSBybmItc291cmNlLWVuZCAtLT5cbiJ9 -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxudWZvX21vZGVsX2RhdGFfbXV0YXRlZCAlPiVcbiAgY291bnQod2Vla2RheSkgJT4lXG4gIGdncGxvdChhZXMoeCA9IHdlZWtkYXksIHkgPSBuKSkgK1xuICBnZW9tX2NvbChmaWxsID0gXCJvcmFuZ2VcIikgK1xuICBsYWJzKHRpdGxlID0gXCJTaWdodGluZ3MgZGVwZW5kaW5nIG9uIHRoZSBkYXkgb2YgdGhlIHdlZWtcIiwgeCA9IFwiRGF5IG9mIHRoZSB3ZWVrXCIsIHkgPSBcIk51bWJlciBvZiBzaWdodGluZ3NcIilcbmBgYCJ9 -->

```r
ufo_model_data_mutated %>%
  count(weekday) %>%
  ggplot(aes(x = weekday, y = n)) +
  geom_col(fill = "orange") +
  labs(title = "Sightings depending on the day of the week", x = "Day of the week", y = "Number of sightings")
```

<!-- rnb-source-end -->


<!-- rnb-output-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->


**Interpretation:**

The number of UFO sightings varies by day of the week. The highest counts occur on Saturdays and Sundays, while Tuesdays have the fewest reports. This suggests people are more likely to notice and report sightings during weekends, possibly due to having more free time or being outdoors more often.

## Hourly distribution of sightings

To analyze what time of day sightings occur most often, revealing strong nocturnal patterns in reported events.


<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-output-begin eyJkYXRhIjoiXG48IS0tIHJuYi1zb3VyY2UtYmVnaW4gZXlKa1lYUmhJam9pWUdCZ2NseHVkV1p2WDIxdlpHVnNYMlJoZEdGZmJYVjBZWFJsWkNBbFBpVmNiaUFnYlhWMFlYUmxLR2h2ZFhJZ1BTQm9iM1Z5S0hKbGNHOXlkR1ZrWDJSaGRHVmZkR2x0WlNrcElDVStKVnh1SUNCamIzVnVkQ2hvYjNWeUtTQWxQaVZjYmlBZ1oyZHdiRzkwS0dGbGN5aDRJRDBnYUc5MWNpd2dlU0E5SUc0cEtTQXJYRzRnSUdkbGIyMWZZMjlzS0dacGJHd2dQU0JjSW5CMWNuQnNaVndpS1NBclhHNGdJR3hoWW5Nb2RHbDBiR1VnUFNCY0lraHZkWEpzZVNCa2FYTjBjbWxpZFhScGIyNGdiMllnYzJsbmFIUnBibWR6WENJc0lIZ2dQU0JjSWtodmRYSWdiMllnZEdobElHUmhlVndpTENCNUlEMGdYQ0pPZFcxaVpYSWdiMllnYzJsbmFIUnBibWR6WENJcFhHNWdZR0FpZlE9PSAtLT5cblxuYGBgclxudWZvX21vZGVsX2RhdGFfbXV0YXRlZCAlPiVcbiAgbXV0YXRlKGhvdXIgPSBob3VyKHJlcG9ydGVkX2RhdGVfdGltZSkpICU+JVxuICBjb3VudChob3VyKSAlPiVcbiAgZ2dwbG90KGFlcyh4ID0gaG91ciwgeSA9IG4pKSArXG4gIGdlb21fY29sKGZpbGwgPSBcInB1cnBsZVwiKSArXG4gIGxhYnModGl0bGUgPSBcIkhvdXJseSBkaXN0cmlidXRpb24gb2Ygc2lnaHRpbmdzXCIsIHggPSBcIkhvdXIgb2YgdGhlIGRheVwiLCB5ID0gXCJOdW1iZXIgb2Ygc2lnaHRpbmdzXCIpXG5gYGBcblxuPCEtLSBybmItc291cmNlLWVuZCAtLT5cbiJ9 -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxudWZvX21vZGVsX2RhdGFfbXV0YXRlZCAlPiVcbiAgbXV0YXRlKGhvdXIgPSBob3VyKHJlcG9ydGVkX2RhdGVfdGltZSkpICU+JVxuICBjb3VudChob3VyKSAlPiVcbiAgZ2dwbG90KGFlcyh4ID0gaG91ciwgeSA9IG4pKSArXG4gIGdlb21fY29sKGZpbGwgPSBcInB1cnBsZVwiKSArXG4gIGxhYnModGl0bGUgPSBcIkhvdXJseSBkaXN0cmlidXRpb24gb2Ygc2lnaHRpbmdzXCIsIHggPSBcIkhvdXIgb2YgdGhlIGRheVwiLCB5ID0gXCJOdW1iZXIgb2Ygc2lnaHRpbmdzXCIpXG5gYGAifQ== -->

```r
ufo_model_data_mutated %>%
  mutate(hour = hour(reported_date_time)) %>%
  count(hour) %>%
  ggplot(aes(x = hour, y = n)) +
  geom_col(fill = "purple") +
  labs(title = "Hourly distribution of sightings", x = "Hour of the day", y = "Number of sightings")
```

<!-- rnb-source-end -->


<!-- rnb-output-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->


**Interpretation:**

UFO sightings are most frequently reported between 8 PM and 3 AM, peaking around 2 AM. Sightings are least common during midday hours. This pattern suggests that sightings are more likely to occur—or at least be noticed and reported—at night, when the sky is dark and unusual lights are more visible.

## Heatmap: day of the week vs hour of the day

To combine daily and hourly trends, showing exactly when during the week sightings peak—particularly during late-night weekend hours.


<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-output-begin eyJkYXRhIjoiXG48IS0tIHJuYi1zb3VyY2UtYmVnaW4gZXlKa1lYUmhJam9pWUdCZ2NseHVkV1p2WDIxdlpHVnNYMlJoZEdGZmJYVjBZWFJsWkNBbFBpVmNiaUFnYlhWMFlYUmxLRnh1SUNBZ0lHaHZkWElnUFNCb2IzVnlLSEpsY0c5eWRHVmtYMlJoZEdWZmRHbHRaU2tzWEc0Z0lDQWdkMlZsYTJSaGVTQTlJR1pqZEY5eVpXeGxkbVZzS0hkbFpXdGtZWGtzSUdNb1hDSk5iMjVjSWl3Z1hDSlVkV1ZjSWl3Z1hDSlhaV1JjSWl3Z1hDSlVhSFZjSWl3Z1hDSkdjbWxjSWl3Z1hDSlRZWFJjSWl3Z1hDSlRkVzVjSWlrcFhHNGdJQ2tnSlQ0bFhHNGdJR052ZFc1MEtIZGxaV3RrWVhrc0lHaHZkWElwSUNVK0pWeHVJQ0JuWjNCc2IzUW9ZV1Z6S0hnZ1BTQm9iM1Z5TENCNUlEMGdkMlZsYTJSaGVTd2dabWxzYkNBOUlHNHBLU0FyWEc0Z0lHZGxiMjFmZEdsc1pTaGpiMnh2Y2lBOUlGd2lkMmhwZEdWY0lpa2dLMXh1SUNCelkyRnNaVjltYVd4c1gzWnBjbWxrYVhOZll5Z3BJQ3RjYmlBZ2JHRmljeWgwYVhSc1pTQTlJRndpU0dWaGRHMWhjRG9nWkdGNUlHOW1JSFJvWlNCM1pXVnJJSFp6SUdodmRYSWdiMllnZEdobElHUmhlVndpTENCNElEMGdYQ0pJYjNWeUlHOW1JSFJvWlNCa1lYbGNJaXdnZVNBOUlGd2lSR0Y1SUc5bUlIUm9aU0IzWldWclhDSXNJR1pwYkd3Z1BTQmNJazUxYldKbGNpQnZaaUJ6YVdkb2RHbHVaM05jSWlsY2JseHVZR0JnSW4wPSAtLT5cblxuYGBgclxudWZvX21vZGVsX2RhdGFfbXV0YXRlZCAlPiVcbiAgbXV0YXRlKFxuICAgIGhvdXIgPSBob3VyKHJlcG9ydGVkX2RhdGVfdGltZSksXG4gICAgd2Vla2RheSA9IGZjdF9yZWxldmVsKHdlZWtkYXksIGMoXCJNb25cIiwgXCJUdWVcIiwgXCJXZWRcIiwgXCJUaHVcIiwgXCJGcmlcIiwgXCJTYXRcIiwgXCJTdW5cIikpXG4gICkgJT4lXG4gIGNvdW50KHdlZWtkYXksIGhvdXIpICU+JVxuICBnZ3Bsb3QoYWVzKHggPSBob3VyLCB5ID0gd2Vla2RheSwgZmlsbCA9IG4pKSArXG4gIGdlb21fdGlsZShjb2xvciA9IFwid2hpdGVcIikgK1xuICBzY2FsZV9maWxsX3ZpcmlkaXNfYygpICtcbiAgbGFicyh0aXRsZSA9IFwiSGVhdG1hcDogZGF5IG9mIHRoZSB3ZWVrIHZzIGhvdXIgb2YgdGhlIGRheVwiLCB4ID0gXCJIb3VyIG9mIHRoZSBkYXlcIiwgeSA9IFwiRGF5IG9mIHRoZSB3ZWVrXCIsIGZpbGwgPSBcIk51bWJlciBvZiBzaWdodGluZ3NcIilcblxuYGBgXG5cbjwhLS0gcm5iLXNvdXJjZS1lbmQgLS0+XG4ifQ== -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxudWZvX21vZGVsX2RhdGFfbXV0YXRlZCAlPiVcbiAgbXV0YXRlKFxuICAgIGhvdXIgPSBob3VyKHJlcG9ydGVkX2RhdGVfdGltZSksXG4gICAgd2Vla2RheSA9IGZjdF9yZWxldmVsKHdlZWtkYXksIGMoXCJNb25cIiwgXCJUdWVcIiwgXCJXZWRcIiwgXCJUaHVcIiwgXCJGcmlcIiwgXCJTYXRcIiwgXCJTdW5cIikpXG4gICkgJT4lXG4gIGNvdW50KHdlZWtkYXksIGhvdXIpICU+JVxuICBnZ3Bsb3QoYWVzKHggPSBob3VyLCB5ID0gd2Vla2RheSwgZmlsbCA9IG4pKSArXG4gIGdlb21fdGlsZShjb2xvciA9IFwid2hpdGVcIikgK1xuICBzY2FsZV9maWxsX3ZpcmlkaXNfYygpICtcbiAgbGFicyh0aXRsZSA9IFwiSGVhdG1hcDogZGF5IG9mIHRoZSB3ZWVrIHZzIGhvdXIgb2YgdGhlIGRheVwiLCB4ID0gXCJIb3VyIG9mIHRoZSBkYXlcIiwgeSA9IFwiRGF5IG9mIHRoZSB3ZWVrXCIsIGZpbGwgPSBcIk51bWJlciBvZiBzaWdodGluZ3NcIilcblxuYGBgIn0= -->

```r
ufo_model_data_mutated %>%
  mutate(
    hour = hour(reported_date_time),
    weekday = fct_relevel(weekday, c("Mon", "Tue", "Wed", "Thu", "Fri", "Sat", "Sun"))
  ) %>%
  count(weekday, hour) %>%
  ggplot(aes(x = hour, y = weekday, fill = n)) +
  geom_tile(color = "white") +
  scale_fill_viridis_c() +
  labs(title = "Heatmap: day of the week vs hour of the day", x = "Hour of the day", y = "Day of the week", fill = "Number of sightings")

```

<!-- rnb-source-end -->


<!-- rnb-output-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->


**Interpretation:**

The heatmap shows the distribution of UFO sightings by hour of the day and day of the week. Most sightings occur after midnight on Sunday, peaking between 1–3 AM. Other late-night hours, especially on weekends, also show elevated counts. This pattern reinforces that sightings are more frequent during late-night weekend hours, when people are likely to be awake and outdoors in dark conditions.

## Sightins with images vs no image

To highlight the rarity of photographic evidence in UFO reports, emphasizing reliance on witness descriptions.


<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-output-begin eyJkYXRhIjoiXG48IS0tIHJuYi1zb3VyY2UtYmVnaW4gZXlKa1lYUmhJam9pWUdCZ2NseHVkV1p2WDIxdlpHVnNYMlJoZEdGZmJYVjBZWFJsWkNBbFBpVmNiaUFnYlhWMFlYUmxLR2x0WVdkbFgzTjBZWFIxY3lBOUlHbG1aV3h6WlNob1lYTmZhVzFoWjJWekxDQmNJa2hoY3lCaGJpQnBiV0ZuWlZ3aUxDQmNJa2hoY3lCdWJ5QnBiV0ZuWlZ3aUtTa2dKVDRsWEc0Z0lHTnZkVzUwS0dsdFlXZGxYM04wWVhSMWN5a2dKVDRsWEc0Z0lHZG5jR3h2ZENoaFpYTW9lQ0E5SUZ3aVhDSXNJSGtnUFNCdUxDQm1hV3hzSUQwZ2FXMWhaMlZmYzNSaGRIVnpLU2tnSzF4dUlDQm5aVzl0WDJOdmJDaDNhV1IwYUNBOUlERXBJQ3RjYmlBZ1kyOXZjbVJmY0c5c1lYSW9kR2hsZEdFZ1BTQmNJbmxjSWlrZ0sxeHVJQ0JzWVdKektIUnBkR3hsSUQwZ1hDSlRhV2RvZEdsdWN5QjNhWFJvSUdsdFlXZGxjeUIyY3lCdWJ5QnBiV0ZuWlZ3aUxDQm1hV3hzSUQwZ1hDSkpiV0ZuWlNCbGVHbHpkR1Z1WTJWY0lpa2dLMXh1SUNCMGFHVnRaVjkyYjJsa0tDa2dLMXh1SUNCelkyRnNaVjltYVd4c1gyMWhiblZoYkNoMllXeDFaWE1nUFNCaktGd2lTR0Z6SUdGdUlHbHRZV2RsWENJZ1BTQmNJaU0yTmtKQ05rRmNJaXdnWENKSVlYTWdibThnYVcxaFoyVmNJaUE5SUZ3aUkwVkdOVE0xTUZ3aUtTbGNibUJnWUNKOSAtLT5cblxuYGBgclxudWZvX21vZGVsX2RhdGFfbXV0YXRlZCAlPiVcbiAgbXV0YXRlKGltYWdlX3N0YXR1cyA9IGlmZWxzZShoYXNfaW1hZ2VzLCBcIkhhcyBhbiBpbWFnZVwiLCBcIkhhcyBubyBpbWFnZVwiKSkgJT4lXG4gIGNvdW50KGltYWdlX3N0YXR1cykgJT4lXG4gIGdncGxvdChhZXMoeCA9IFwiXCIsIHkgPSBuLCBmaWxsID0gaW1hZ2Vfc3RhdHVzKSkgK1xuICBnZW9tX2NvbCh3aWR0aCA9IDEpICtcbiAgY29vcmRfcG9sYXIodGhldGEgPSBcInlcIikgK1xuICBsYWJzKHRpdGxlID0gXCJTaWdodGlucyB3aXRoIGltYWdlcyB2cyBubyBpbWFnZVwiLCBmaWxsID0gXCJJbWFnZSBleGlzdGVuY2VcIikgK1xuICB0aGVtZV92b2lkKCkgK1xuICBzY2FsZV9maWxsX21hbnVhbCh2YWx1ZXMgPSBjKFwiSGFzIGFuIGltYWdlXCIgPSBcIiM2NkJCNkFcIiwgXCJIYXMgbm8gaW1hZ2VcIiA9IFwiI0VGNTM1MFwiKSlcbmBgYFxuXG48IS0tIHJuYi1zb3VyY2UtZW5kIC0tPlxuIn0= -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxudWZvX21vZGVsX2RhdGFfbXV0YXRlZCAlPiVcbiAgbXV0YXRlKGltYWdlX3N0YXR1cyA9IGlmZWxzZShoYXNfaW1hZ2VzLCBcIkhhcyBhbiBpbWFnZVwiLCBcIkhhcyBubyBpbWFnZVwiKSkgJT4lXG4gIGNvdW50KGltYWdlX3N0YXR1cykgJT4lXG4gIGdncGxvdChhZXMoeCA9IFwiXCIsIHkgPSBuLCBmaWxsID0gaW1hZ2Vfc3RhdHVzKSkgK1xuICBnZW9tX2NvbCh3aWR0aCA9IDEpICtcbiAgY29vcmRfcG9sYXIodGhldGEgPSBcInlcIikgK1xuICBsYWJzKHRpdGxlID0gXCJTaWdodGlucyB3aXRoIGltYWdlcyB2cyBubyBpbWFnZVwiLCBmaWxsID0gXCJJbWFnZSBleGlzdGVuY2VcIikgK1xuICB0aGVtZV92b2lkKCkgK1xuICBzY2FsZV9maWxsX21hbnVhbCh2YWx1ZXMgPSBjKFwiSGFzIGFuIGltYWdlXCIgPSBcIiM2NkJCNkFcIiwgXCJIYXMgbm8gaW1hZ2VcIiA9IFwiI0VGNTM1MFwiKSlcbmBgYCJ9 -->

```r
ufo_model_data_mutated %>%
  mutate(image_status = ifelse(has_images, "Has an image", "Has no image")) %>%
  count(image_status) %>%
  ggplot(aes(x = "", y = n, fill = image_status)) +
  geom_col(width = 1) +
  coord_polar(theta = "y") +
  labs(title = "Sightins with images vs no image", fill = "Image existence") +
  theme_void() +
  scale_fill_manual(values = c("Has an image" = "#66BB6A", "Has no image" = "#EF5350"))
```

<!-- rnb-source-end -->


<!-- rnb-output-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->


**Interpretation:**

The chart shows that almost all UFO sightings lack images. Sightings with images are extremely rare, suggesting that reports are usually text-based or anecdotal. This indicates a strong reliance on witness testimony rather than visual evidence in the dataset.

Making sure if above piechart is correct


<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-output-begin eyJkYXRhIjoiXG48IS0tIHJuYi1zb3VyY2UtYmVnaW4gZXlKa1lYUmhJam9pWUdCZ2NseHVjM1Z0S0hWbWIxOXRiMlJsYkY5a1lYUmhYMjExZEdGMFpXUWthR0Z6WDJsdFlXZGxjeUFoUFNCR1FVeFRSU3dnYm1FdWNtMGdQU0JVVWxWRktWeHVZR0JnSW4wPSAtLT5cblxuYGBgclxuc3VtKHVmb19tb2RlbF9kYXRhX211dGF0ZWQkaGFzX2ltYWdlcyAhPSBGQUxTRSwgbmEucm0gPSBUUlVFKVxuYGBgXG5cbjwhLS0gcm5iLXNvdXJjZS1lbmQgLS0+XG4ifQ== -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxuc3VtKHVmb19tb2RlbF9kYXRhX211dGF0ZWQkaGFzX2ltYWdlcyAhPSBGQUxTRSwgbmEucm0gPSBUUlVFKVxuYGBgIn0= -->

```r
sum(ufo_model_data_mutated$has_images != FALSE, na.rm = TRUE)
```

<!-- rnb-source-end -->


<!-- rnb-output-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->


## Number of sightings per shape

To identify which UFO shapes are most commonly reported, offering insight into visual patterns or public perception.


<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-output-begin eyJkYXRhIjoiXG48IS0tIHJuYi1zb3VyY2UtYmVnaW4gZXlKa1lYUmhJam9pWUdCZ2NseHVkV1p2WDIxdlpHVnNYMlJoZEdGZmJYVjBZWFJsWkNBbFBpVmNiaUFnWTI5MWJuUW9jMmhoY0dVcElDVStKVnh1SUNCblozQnNiM1FvWVdWektIZ2dQU0J5Wlc5eVpHVnlLSE5vWVhCbExDQnVLU3dnZVNBOUlHNHBLU0FyWEc0Z0lHZGxiMjFmWTI5c0tHWnBiR3dnUFNCY0luTnJlV0pzZFdWY0lpa2dLMXh1SUNCamIyOXlaRjltYkdsd0tDa2dLMXh1SUNCc1lXSnpLSFJwZEd4bElEMGdYQ0pPZFcxaVpYSWdiMllnYzJsbmFIUnBibWR6SUhCbGNpQnphR0Z3WlZ3aUxDQjRJRDBnWENKVGFHRndaVndpTENCNUlEMGdYQ0pPZFcxaVpYSWdiMllnYzJsbmFIUnBibWR6WENJcFhHNWdZR0FpZlE9PSAtLT5cblxuYGBgclxudWZvX21vZGVsX2RhdGFfbXV0YXRlZCAlPiVcbiAgY291bnQoc2hhcGUpICU+JVxuICBnZ3Bsb3QoYWVzKHggPSByZW9yZGVyKHNoYXBlLCBuKSwgeSA9IG4pKSArXG4gIGdlb21fY29sKGZpbGwgPSBcInNreWJsdWVcIikgK1xuICBjb29yZF9mbGlwKCkgK1xuICBsYWJzKHRpdGxlID0gXCJOdW1iZXIgb2Ygc2lnaHRpbmdzIHBlciBzaGFwZVwiLCB4ID0gXCJTaGFwZVwiLCB5ID0gXCJOdW1iZXIgb2Ygc2lnaHRpbmdzXCIpXG5gYGBcblxuPCEtLSBybmItc291cmNlLWVuZCAtLT5cbiJ9 -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxudWZvX21vZGVsX2RhdGFfbXV0YXRlZCAlPiVcbiAgY291bnQoc2hhcGUpICU+JVxuICBnZ3Bsb3QoYWVzKHggPSByZW9yZGVyKHNoYXBlLCBuKSwgeSA9IG4pKSArXG4gIGdlb21fY29sKGZpbGwgPSBcInNreWJsdWVcIikgK1xuICBjb29yZF9mbGlwKCkgK1xuICBsYWJzKHRpdGxlID0gXCJOdW1iZXIgb2Ygc2lnaHRpbmdzIHBlciBzaGFwZVwiLCB4ID0gXCJTaGFwZVwiLCB5ID0gXCJOdW1iZXIgb2Ygc2lnaHRpbmdzXCIpXG5gYGAifQ== -->

```r
ufo_model_data_mutated %>%
  count(shape) %>%
  ggplot(aes(x = reorder(shape, n), y = n)) +
  geom_col(fill = "skyblue") +
  coord_flip() +
  labs(title = "Number of sightings per shape", x = "Shape", y = "Number of sightings")
```

<!-- rnb-source-end -->


<!-- rnb-output-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->


**Interpretation:**

The most commonly reported UFO shapes are light, circle, and triangle. Unusual shapes like cube, star, and cross are very rare. This suggests that most sightings describe simple or glowing forms, possibly influenced by visibility, perception, or common cultural imagery.

## Number of sightings per country

To examine geographical reporting patterns and reveal the strong dominance of the U.S. in the dataset.


<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-output-begin eyJkYXRhIjoiXG48IS0tIHJuYi1zb3VyY2UtYmVnaW4gZXlKa1lYUmhJam9pWUdCZ2NseHVkV1p2WDIxdlpHVnNYMlJoZEdGZmJYVjBZWFJsWkNBbFBpVmNiaUFnWTI5MWJuUW9ZMjkxYm5SeWVWOWpiMlJsS1NBbFBpVmNiaUFnWm1sc2RHVnlLRzRnUGowZ01UQXdLU0FsUGlWY2JpQWdaMmR3Ykc5MEtHRmxjeWg0SUQwZ2NtVnZjbVJsY2loamIzVnVkSEo1WDJOdlpHVXNJRzRwTENCNUlEMGdiaWtwSUN0Y2JpQWdaMlZ2YlY5aVlYSW9jM1JoZENBOUlGd2lhV1JsYm5ScGRIbGNJaXdnWm1sc2JDQTlJRndpYzJ0NVlteDFaVndpS1NBclhHNGdJR3hoWW5Nb1hHNGdJQ0FnZEdsMGJHVWdQU0JjSWs1MWJXSmxjaUJ2WmlCemFXZG9kR2x1WjNNZ2NHVnlJR052ZFc1MGNubGNJaXhjYmlBZ0lDQjRJRDBnWENKRGIzVnVkSEo1SUdOdlpHVmNJaXhjYmlBZ0lDQjVJRDBnWENKT2RXMWlaWElnYjJZZ2MybG5hSFJwYm1kelhDSmNiaUFnS1NBclhHNGdJSFJvWlcxbEtHRjRhWE11ZEdWNGRDNTRJRDBnWld4bGJXVnVkRjkwWlhoMEtHRnVaMnhsSUQwZ05EVXNJR2hxZFhOMElEMGdNU2twWEc1Z1lHQWlmUT09IC0tPlxuXG5gYGByXG51Zm9fbW9kZWxfZGF0YV9tdXRhdGVkICU+JVxuICBjb3VudChjb3VudHJ5X2NvZGUpICU+JVxuICBmaWx0ZXIobiA+PSAxMDApICU+JVxuICBnZ3Bsb3QoYWVzKHggPSByZW9yZGVyKGNvdW50cnlfY29kZSwgbiksIHkgPSBuKSkgK1xuICBnZW9tX2JhcihzdGF0ID0gXCJpZGVudGl0eVwiLCBmaWxsID0gXCJza3libHVlXCIpICtcbiAgbGFicyhcbiAgICB0aXRsZSA9IFwiTnVtYmVyIG9mIHNpZ2h0aW5ncyBwZXIgY291bnRyeVwiLFxuICAgIHggPSBcIkNvdW50cnkgY29kZVwiLFxuICAgIHkgPSBcIk51bWJlciBvZiBzaWdodGluZ3NcIlxuICApICtcbiAgdGhlbWUoYXhpcy50ZXh0LnggPSBlbGVtZW50X3RleHQoYW5nbGUgPSA0NSwgaGp1c3QgPSAxKSlcbmBgYFxuXG48IS0tIHJuYi1zb3VyY2UtZW5kIC0tPlxuIn0= -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxudWZvX21vZGVsX2RhdGFfbXV0YXRlZCAlPiVcbiAgY291bnQoY291bnRyeV9jb2RlKSAlPiVcbiAgZmlsdGVyKG4gPj0gMTAwKSAlPiVcbiAgZ2dwbG90KGFlcyh4ID0gcmVvcmRlcihjb3VudHJ5X2NvZGUsIG4pLCB5ID0gbikpICtcbiAgZ2VvbV9iYXIoc3RhdCA9IFwiaWRlbnRpdHlcIiwgZmlsbCA9IFwic2t5Ymx1ZVwiKSArXG4gIGxhYnMoXG4gICAgdGl0bGUgPSBcIk51bWJlciBvZiBzaWdodGluZ3MgcGVyIGNvdW50cnlcIixcbiAgICB4ID0gXCJDb3VudHJ5IGNvZGVcIixcbiAgICB5ID0gXCJOdW1iZXIgb2Ygc2lnaHRpbmdzXCJcbiAgKSArXG4gIHRoZW1lKGF4aXMudGV4dC54ID0gZWxlbWVudF90ZXh0KGFuZ2xlID0gNDUsIGhqdXN0ID0gMSkpXG5gYGAifQ== -->

```r
ufo_model_data_mutated %>%
  count(country_code) %>%
  filter(n >= 100) %>%
  ggplot(aes(x = reorder(country_code, n), y = n)) +
  geom_bar(stat = "identity", fill = "skyblue") +
  labs(
    title = "Number of sightings per country",
    x = "Country code",
    y = "Number of sightings"
  ) +
  theme(axis.text.x = element_text(angle = 45, hjust = 1))
```

<!-- rnb-source-end -->


<!-- rnb-output-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->


**Interpretation:**

The vast majority of UFO sightings come from the United States, with over 80,000 reports. Other countries like Canada and Great Britain have significantly fewer sightings. This suggests that the dataset is strongly US-centric, possibly due to better reporting infrastructure, public interest, or data source bias.

## Density map of sightings

To visualize the global distribution of sightings and how they relate to city population size and location.


<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-output-begin eyJkYXRhIjoiXG48IS0tIHJuYi1zb3VyY2UtYmVnaW4gZXlKa1lYUmhJam9pWUdCZ2NseHVkMjl5YkdRZ1BDMGdibVZmWTI5MWJuUnlhV1Z6S0hOallXeGxJRDBnWENKdFpXUnBkVzFjSWl3Z2NtVjBkWEp1WTJ4aGMzTWdQU0JjSW5ObVhDSXBYRzVjYm5Cc1lXTmxjMTlqYkdWaGJpQWxQaVZjYmlBZ1ptbHNkR1Z5S0NGcGN5NXVZU2hzWVhScGRIVmtaU2tzSUNGcGN5NXVZU2hzYjI1bmFYUjFaR1VwS1NBbFBpVmNiaUFnYlhWMFlYUmxLRnh1SUNBZ0lIQnZjRjlqWVhSbFoyOXllU0E5SUdOaGMyVmZkMmhsYmloY2JpQWdJQ0FnSUhCdmNIVnNZWFJwYjI0Z1BDQXhNREF3TUNBZ2ZpQmNJbk50WVd4c1hDSXNYRzRnSUNBZ0lDQndiM0IxYkdGMGFXOXVJRHdnTVRBd01EQXdJSDRnWENKdFpXUnBkVzFjSWl4Y2JpQWdJQ0FnSUZSU1ZVVWdJQ0FnSUNBZ0lDQWdJQ0FnSUNBZ2ZpQmNJbXhoY21kbFhDSmNiaUFnSUNBcFhHNGdJQ2tnSlQ0bFhHNGdJR2RuY0d4dmRDZ3BJQ3RjYmlBZ1oyVnZiVjl6Wmloa1lYUmhJRDBnZDI5eWJHUXNJR1pwYkd3Z1BTQmNJbXhwWjJoMFozSmhlVndpTENCamIyeHZjaUE5SUZ3aVlteGhZMnRjSWlrZ0t5QWdJeUJFYjJSaGFtVnRlU0J0WVhERW1WeHVJQ0JuWlc5dFgzQnZhVzUwS0dGbGN5aGNiaUFnSUNCNElEMGdiRzl1WjJsMGRXUmxMQ0I1SUQwZ2JHRjBhWFIxWkdVc1hHNGdJQ0FnYzJsNlpTQTlJSEJ2Y0hWc1lYUnBiMjRzSUdOdmJHOXlJRDBnY0c5d1gyTmhkR1ZuYjNKNVhHNGdJQ2tzSUdGc2NHaGhJRDBnTUM0MktTQXJYRzRnSUhOallXeGxYM05wZW1Vb2NtRnVaMlVnUFNCaktERXNJRFlwTENCbmRXbGtaU0E5SUZ3aWJtOXVaVndpS1NBclhHNGdJR3hoWW5Nb1hHNGdJQ0FnZEdsMGJHVWdQU0JjSWtOcGRHbGxjeUIzYVhSb0lGVkdUeUJ6YVdkb2RHbHVaM05jSWl4Y2JpQWdJQ0J6ZFdKMGFYUnNaU0E5SUZ3aVVHOXBiblFnYzJsNlpTQitJSEJ2Y0hWc1lYUnBiMjRzSUdOdmJHOXlJSDRnY0c5d2RXeGhkR2x2YmlCallYUmxaMjl5ZVZ3aUxGeHVJQ0FnSUhnZ1BTQmNJa3hoZEdsMGRXUmxYQ0lzWEc0Z0lDQWdlU0E5SUZ3aVFXeDBhWFIxWkdWY0lpeGNiaUFnSUNCamIyeHZjaUE5SUZ3aVVHOXdkV3hoZEdsdmJpQmpZWFJsWjI5eWVWd2lYRzRnSUNrZ0sxeHVJQ0IwYUdWdFpWOXRhVzVwYldGc0tDbGNibUJnWUNKOSAtLT5cblxuYGBgclxud29ybGQgPC0gbmVfY291bnRyaWVzKHNjYWxlID0gXCJtZWRpdW1cIiwgcmV0dXJuY2xhc3MgPSBcInNmXCIpXG5cbnBsYWNlc19jbGVhbiAlPiVcbiAgZmlsdGVyKCFpcy5uYShsYXRpdHVkZSksICFpcy5uYShsb25naXR1ZGUpKSAlPiVcbiAgbXV0YXRlKFxuICAgIHBvcF9jYXRlZ29yeSA9IGNhc2Vfd2hlbihcbiAgICAgIHBvcHVsYXRpb24gPCAxMDAwMCAgfiBcInNtYWxsXCIsXG4gICAgICBwb3B1bGF0aW9uIDwgMTAwMDAwIH4gXCJtZWRpdW1cIixcbiAgICAgIFRSVUUgICAgICAgICAgICAgICAgfiBcImxhcmdlXCJcbiAgICApXG4gICkgJT4lXG4gIGdncGxvdCgpICtcbiAgZ2VvbV9zZihkYXRhID0gd29ybGQsIGZpbGwgPSBcImxpZ2h0Z3JheVwiLCBjb2xvciA9IFwiYmxhY2tcIikgKyAgIyBEb2RhamVteSBtYXDEmVxuICBnZW9tX3BvaW50KGFlcyhcbiAgICB4ID0gbG9uZ2l0dWRlLCB5ID0gbGF0aXR1ZGUsXG4gICAgc2l6ZSA9IHBvcHVsYXRpb24sIGNvbG9yID0gcG9wX2NhdGVnb3J5XG4gICksIGFscGhhID0gMC42KSArXG4gIHNjYWxlX3NpemUocmFuZ2UgPSBjKDEsIDYpLCBndWlkZSA9IFwibm9uZVwiKSArXG4gIGxhYnMoXG4gICAgdGl0bGUgPSBcIkNpdGllcyB3aXRoIFVGTyBzaWdodGluZ3NcIixcbiAgICBzdWJ0aXRsZSA9IFwiUG9pbnQgc2l6ZSB+IHBvcHVsYXRpb24sIGNvbG9yIH4gcG9wdWxhdGlvbiBjYXRlZ29yeVwiLFxuICAgIHggPSBcIkxhdGl0dWRlXCIsXG4gICAgeSA9IFwiQWx0aXR1ZGVcIixcbiAgICBjb2xvciA9IFwiUG9wdWxhdGlvbiBjYXRlZ29yeVwiXG4gICkgK1xuICB0aGVtZV9taW5pbWFsKClcbmBgYFxuXG48IS0tIHJuYi1zb3VyY2UtZW5kIC0tPlxuIn0= -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxud29ybGQgPC0gbmVfY291bnRyaWVzKHNjYWxlID0gXCJtZWRpdW1cIiwgcmV0dXJuY2xhc3MgPSBcInNmXCIpXG5cbnBsYWNlc19jbGVhbiAlPiVcbiAgZmlsdGVyKCFpcy5uYShsYXRpdHVkZSksICFpcy5uYShsb25naXR1ZGUpKSAlPiVcbiAgbXV0YXRlKFxuICAgIHBvcF9jYXRlZ29yeSA9IGNhc2Vfd2hlbihcbiAgICAgIHBvcHVsYXRpb24gPCAxMDAwMCAgfiBcInNtYWxsXCIsXG4gICAgICBwb3B1bGF0aW9uIDwgMTAwMDAwIH4gXCJtZWRpdW1cIixcbiAgICAgIFRSVUUgICAgICAgICAgICAgICAgfiBcImxhcmdlXCJcbiAgICApXG4gICkgJT4lXG4gIGdncGxvdCgpICtcbiAgZ2VvbV9zZihkYXRhID0gd29ybGQsIGZpbGwgPSBcImxpZ2h0Z3JheVwiLCBjb2xvciA9IFwiYmxhY2tcIikgKyAgIyBEb2RhamVteSBtYXDEmVxuICBnZW9tX3BvaW50KGFlcyhcbiAgICB4ID0gbG9uZ2l0dWRlLCB5ID0gbGF0aXR1ZGUsXG4gICAgc2l6ZSA9IHBvcHVsYXRpb24sIGNvbG9yID0gcG9wX2NhdGVnb3J5XG4gICksIGFscGhhID0gMC42KSArXG4gIHNjYWxlX3NpemUocmFuZ2UgPSBjKDEsIDYpLCBndWlkZSA9IFwibm9uZVwiKSArXG4gIGxhYnMoXG4gICAgdGl0bGUgPSBcIkNpdGllcyB3aXRoIFVGTyBzaWdodGluZ3NcIixcbiAgICBzdWJ0aXRsZSA9IFwiUG9pbnQgc2l6ZSB+IHBvcHVsYXRpb24sIGNvbG9yIH4gcG9wdWxhdGlvbiBjYXRlZ29yeVwiLFxuICAgIHggPSBcIkxhdGl0dWRlXCIsXG4gICAgeSA9IFwiQWx0aXR1ZGVcIixcbiAgICBjb2xvciA9IFwiUG9wdWxhdGlvbiBjYXRlZ29yeVwiXG4gICkgK1xuICB0aGVtZV9taW5pbWFsKClcbmBgYCJ9 -->

```r
world <- ne_countries(scale = "medium", returnclass = "sf")

places_clean %>%
  filter(!is.na(latitude), !is.na(longitude)) %>%
  mutate(
    pop_category = case_when(
      population < 10000  ~ "small",
      population < 100000 ~ "medium",
      TRUE                ~ "large"
    )
  ) %>%
  ggplot() +
  geom_sf(data = world, fill = "lightgray", color = "black") +  # Dodajemy mapę
  geom_point(aes(
    x = longitude, y = latitude,
    size = population, color = pop_category
  ), alpha = 0.6) +
  scale_size(range = c(1, 6), guide = "none") +
  labs(
    title = "Cities with UFO sightings",
    subtitle = "Point size ~ population, color ~ population category",
    x = "Latitude",
    y = "Altitude",
    color = "Population category"
  ) +
  theme_minimal()
```

<!-- rnb-source-end -->


<!-- rnb-output-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->


**Interpretation:**

Sightings are most densely clustered in North America and Europe, especially in large urban areas. This suggests that population density and infrastructure may influence reporting frequency. Other regions show fewer reports, which could reflect lower reporting access or less data availability.

## UFO Sightings by City Population Category

Explores whether UFO sightings are more common in small, medium, or large cities, helping to assess the impact of urban scale on reporting frequency.


<!-- rnb-text-end -->


<!-- rnb-chunk-begin -->


<!-- rnb-output-begin eyJkYXRhIjoiXG48IS0tIHJuYi1zb3VyY2UtYmVnaW4gZXlKa1lYUmhJam9pWUdCZ2NseHVkV1p2WDIxdlpHVnNYMlJoZEdGZmJYVjBZWFJsWkNBbFBpVmNiaUFnYkdWbWRGOXFiMmx1S0hCc1lXTmxjMTl0YjJSbGJGOWtZWFJoWDIxMWRHRjBaV1FnSlQ0bElITmxiR1ZqZENoamFYUjVYM04wWVhSbExDQndiM0JmWTJGMFpXZHZjbmtwTENCaWVTQTlJRndpWTJsMGVWOXpkR0YwWlZ3aUtTQWxQaVZjYmlBZ1kyOTFiblFvY0c5d1gyTmhkR1ZuYjNKNUtTQWxQaVZjYmlBZ1oyZHdiRzkwS0dGbGN5aDRJRDBnY0c5d1gyTmhkR1ZuYjNKNUxDQjVJRDBnYml3Z1ptbHNiQ0E5SUhCdmNGOWpZWFJsWjI5eWVTa3BJQ3RjYmlBZ1oyVnZiVjlqYjJ3b2MyaHZkeTVzWldkbGJtUWdQU0JHUVV4VFJTa2dLMXh1SUNCc1lXSnpLRnh1SUNBZ0lIUnBkR3hsSUQwZ1hDSlZSazhnVTJsbmFIUnBibWR6SUdKNUlFTnBkSGtnVUc5d2RXeGhkR2x2YmlCRFlYUmxaMjl5ZVZ3aUxGeHVJQ0FnSUhnZ1BTQmNJbEJ2Y0hWc1lYUnBiMjRnUTJGMFpXZHZjbmxjSWl4Y2JpQWdJQ0I1SUQwZ1hDSk9kVzFpWlhJZ2IyWWdVMmxuYUhScGJtZHpYQ0pjYmlBZ0tTQXJYRzRnSUhOallXeGxYMlpwYkd4ZmJXRnVkV0ZzS0haaGJIVmxjeUE5SUdNb1hDSnpiV0ZzYkZ3aUlEMGdYQ0lqT1RGaVptUmlYQ0lzSUZ3aWJXVmthWFZ0WENJZ1BTQmNJaU5tWkdGbE5qRmNJaXdnWENKc1lYSm5aVndpSUQwZ1hDSWpaRGN6TURJM1hDSXBLU0FyWEc0Z0lIUm9aVzFsWDIxcGJtbHRZV3dvS1Z4dVlHQmdJbjA9IC0tPlxuXG5gYGByXG51Zm9fbW9kZWxfZGF0YV9tdXRhdGVkICU+JVxuICBsZWZ0X2pvaW4ocGxhY2VzX21vZGVsX2RhdGFfbXV0YXRlZCAlPiUgc2VsZWN0KGNpdHlfc3RhdGUsIHBvcF9jYXRlZ29yeSksIGJ5ID0gXCJjaXR5X3N0YXRlXCIpICU+JVxuICBjb3VudChwb3BfY2F0ZWdvcnkpICU+JVxuICBnZ3Bsb3QoYWVzKHggPSBwb3BfY2F0ZWdvcnksIHkgPSBuLCBmaWxsID0gcG9wX2NhdGVnb3J5KSkgK1xuICBnZW9tX2NvbChzaG93LmxlZ2VuZCA9IEZBTFNFKSArXG4gIGxhYnMoXG4gICAgdGl0bGUgPSBcIlVGTyBTaWdodGluZ3MgYnkgQ2l0eSBQb3B1bGF0aW9uIENhdGVnb3J5XCIsXG4gICAgeCA9IFwiUG9wdWxhdGlvbiBDYXRlZ29yeVwiLFxuICAgIHkgPSBcIk51bWJlciBvZiBTaWdodGluZ3NcIlxuICApICtcbiAgc2NhbGVfZmlsbF9tYW51YWwodmFsdWVzID0gYyhcInNtYWxsXCIgPSBcIiM5MWJmZGJcIiwgXCJtZWRpdW1cIiA9IFwiI2ZkYWU2MVwiLCBcImxhcmdlXCIgPSBcIiNkNzMwMjdcIikpICtcbiAgdGhlbWVfbWluaW1hbCgpXG5gYGBcblxuPCEtLSBybmItc291cmNlLWVuZCAtLT5cbiJ9 -->


<!-- rnb-source-begin eyJkYXRhIjoiYGBgclxudWZvX21vZGVsX2RhdGFfbXV0YXRlZCAlPiVcbiAgbGVmdF9qb2luKHBsYWNlc19tb2RlbF9kYXRhX211dGF0ZWQgJT4lIHNlbGVjdChjaXR5X3N0YXRlLCBwb3BfY2F0ZWdvcnkpLCBieSA9IFwiY2l0eV9zdGF0ZVwiKSAlPiVcbiAgY291bnQocG9wX2NhdGVnb3J5KSAlPiVcbiAgZ2dwbG90KGFlcyh4ID0gcG9wX2NhdGVnb3J5LCB5ID0gbiwgZmlsbCA9IHBvcF9jYXRlZ29yeSkpICtcbiAgZ2VvbV9jb2woc2hvdy5sZWdlbmQgPSBGQUxTRSkgK1xuICBsYWJzKFxuICAgIHRpdGxlID0gXCJVRk8gU2lnaHRpbmdzIGJ5IENpdHkgUG9wdWxhdGlvbiBDYXRlZ29yeVwiLFxuICAgIHggPSBcIlBvcHVsYXRpb24gQ2F0ZWdvcnlcIixcbiAgICB5ID0gXCJOdW1iZXIgb2YgU2lnaHRpbmdzXCJcbiAgKSArXG4gIHNjYWxlX2ZpbGxfbWFudWFsKHZhbHVlcyA9IGMoXCJzbWFsbFwiID0gXCIjOTFiZmRiXCIsIFwibWVkaXVtXCIgPSBcIiNmZGFlNjFcIiwgXCJsYXJnZVwiID0gXCIjZDczMDI3XCIpKSArXG4gIHRoZW1lX21pbmltYWwoKVxuYGBgIn0= -->

```r
ufo_model_data_mutated %>%
  left_join(places_model_data_mutated %>% select(city_state, pop_category), by = "city_state") %>%
  count(pop_category) %>%
  ggplot(aes(x = pop_category, y = n, fill = pop_category)) +
  geom_col(show.legend = FALSE) +
  labs(
    title = "UFO Sightings by City Population Category",
    x = "Population Category",
    y = "Number of Sightings"
  ) +
  scale_fill_manual(values = c("small" = "#91bfdb", "medium" = "#fdae61", "large" = "#d73027")) +
  theme_minimal()
```

<!-- rnb-source-end -->


<!-- rnb-output-end -->

<!-- rnb-output-begin eyJkYXRhIjoiQsWCxIVkIHcgcG9sZWNlbml1ICd1Zm9fbW9kZWxfZGF0YV9tdXRhdGVkICU+JSBsZWZ0X2pvaW4ocGxhY2VzX21vZGVsX2RhdGFfbXV0YXRlZCAlPiUgJzogXG4gIG5pZSB1ZGHFgm8gc2nEmSB6bmFsZcW6xIcgZnVua2NqaSAnJT4lJ1xuIn0= -->

```
Błąd w poleceniu 'ufo_model_data_mutated %>% left_join(places_model_data_mutated %>% ': 
  nie udało się znaleźć funkcji '%>%'
```



<!-- rnb-output-end -->

<!-- rnb-chunk-end -->


<!-- rnb-text-begin -->


**Interpretation:**

The chart shows that most UFO sightings come from medium-sized cities, followed by large cities, with small towns reporting the fewest. This suggests that mid-sized urban areas may offer a balance of visibility, outdoor activity, and public engagement conducive to sightings. It also reflects where people live and are most likely to report unusual events.

<!-- rnb-text-end -->

