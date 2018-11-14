
# API Pagination

You've now worked with some API calls, but we have yet to see how to retrieve a more complete dataset in a programmatic manner. Returning to the Yelp API, the [documentation](https://www.yelp.com/developers/documentation/v3/business_search) also provides us details regarding the API limits. These often include details about the number of requests a user is allowed to make within a specified time limit and the maximum number of results to be returned. In this case, we are told that any request has a maximum of 50 results per request and defaults to 20. Furthermore, any search will be limited to a total of 1000 results. To retrieve all 1000 of these results, we would have to page through the results piece by piece, retriving 50 at a time. Processes such as these are often refered to as pagination. Without further ado, let's take a look in practice.


```python
import requests
import pandas as pd
```


```python
# client_id = #your id here
# api_key = #your key here

term = 'Mexican'
location = 'Astoria NY'
SEARCH_LIMIT = 10

url = 'https://api.yelp.com/v3/businesses/search'

headers = {
        'Authorization': 'Bearer {}'.format(api_key),
    }

url_params = {
                'term': term.replace(' ', '+'),
                'location': location.replace(' ', '+'),
                'limit': SEARCH_LIMIT
            }
response = requests.get(url, headers=headers, params=url_params)
print(response)
print(type(response.text))
print(response.text[:1000])
```

    <Response [200]>
    <class 'str'>
    {"businesses": [{"id": "jeWIYbgBho9vBDhc5S1xvg", "alias": "holy-guacamole-astoria", "name": "Holy Guacamole", "image_url": "https://s3-media1.fl.yelpcdn.com/bphoto/pMTc8Y0UnRFNh0zDFERfSg/o.jpg", "is_closed": false, "url": "https://www.yelp.com/biz/holy-guacamole-astoria?adjust_creative=xNHtXRpNa-MXGFJJTHHUvw&utm_campaign=yelp_api_v3&utm_medium=api_v3_business_search&utm_source=xNHtXRpNa-MXGFJJTHHUvw", "review_count": 127, "categories": [{"alias": "mexican", "title": "Mexican"}, {"alias": "bars", "title": "Bars"}], "rating": 4.0, "coordinates": {"latitude": 40.756621, "longitude": -73.929336}, "transactions": ["delivery", "pickup"], "price": "$$", "location": {"address1": "3555 31st St", "address2": "", "address3": "", "city": "Astoria", "zip_code": "11106", "country": "US", "state": "NY", "display_address": ["3555 31st St", "Astoria, NY 11106"]}, "phone": "+19178327261", "display_phone": "(917) 832-7261", "distance": 1290.4274875130448}, {"id": "AUyKmFjpaVLwc3awfUnqgQ", "alias": "chela


# Previewing the Results

As before, let's briefly investigate the top level strucutre of our JSON response.


```python
response.json().keys()
```




    dict_keys(['businesses', 'total', 'region'])



Navigating down a level, we might be curious how many restaurants fit our criteria:


```python
response.json()['total']
```




    671



Now of those, we have only retrieved the first 10 results (the search limit we provided.) As the documentation describes, this defaults to 20, and can be up to 50. Observe:


```python
print(len(response.json()['businesses']))
```

    10


Recall that we can also turn this into our usual Pandas DataFrame:


```python
df = pd.DataFrame(response.json()['businesses'])
print(len(df))
df.head()
```

    10





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
      <th>alias</th>
      <th>categories</th>
      <th>coordinates</th>
      <th>display_phone</th>
      <th>distance</th>
      <th>id</th>
      <th>image_url</th>
      <th>is_closed</th>
      <th>location</th>
      <th>name</th>
      <th>phone</th>
      <th>price</th>
      <th>rating</th>
      <th>review_count</th>
      <th>transactions</th>
      <th>url</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>holy-guacamole-astoria</td>
      <td>[{'alias': 'mexican', 'title': 'Mexican'}, {'a...</td>
      <td>{'latitude': 40.756621, 'longitude': -73.929336}</td>
      <td>(917) 832-7261</td>
      <td>1290.427488</td>
      <td>jeWIYbgBho9vBDhc5S1xvg</td>
      <td>https://s3-media1.fl.yelpcdn.com/bphoto/pMTc8Y...</td>
      <td>False</td>
      <td>{'address1': '3555 31st St', 'address2': '', '...</td>
      <td>Holy Guacamole</td>
      <td>+19178327261</td>
      <td>$$</td>
      <td>4.0</td>
      <td>127</td>
      <td>[delivery, pickup]</td>
      <td>https://www.yelp.com/biz/holy-guacamole-astori...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>chela-and-garnacha-astoria</td>
      <td>[{'alias': 'mexican', 'title': 'Mexican'}, {'a...</td>
      <td>{'latitude': 40.7557171543477, 'longitude': -7...</td>
      <td>(917) 832-6876</td>
      <td>1316.297661</td>
      <td>AUyKmFjpaVLwc3awfUnqgQ</td>
      <td>https://s3-media1.fl.yelpcdn.com/bphoto/ChVbA1...</td>
      <td>False</td>
      <td>{'address1': '33-09 36th Ave', 'address2': '',...</td>
      <td>Chela &amp; Garnacha</td>
      <td>+19178326876</td>
      <td>$$</td>
      <td>4.5</td>
      <td>312</td>
      <td>[delivery, pickup]</td>
      <td>https://www.yelp.com/biz/chela-and-garnacha-as...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>de-mole-astoria-astoria</td>
      <td>[{'alias': 'mexican', 'title': 'Mexican'}]</td>
      <td>{'latitude': 40.7625999, 'longitude': -73.9129...</td>
      <td>(718) 777-1655</td>
      <td>918.092772</td>
      <td>jzVv_21473lAMYXIhVbuTA</td>
      <td>https://s3-media1.fl.yelpcdn.com/bphoto/v8jXvZ...</td>
      <td>False</td>
      <td>{'address1': '4220 30th Ave', 'address2': '', ...</td>
      <td>De Mole Astoria</td>
      <td>+17187771655</td>
      <td>$$</td>
      <td>4.0</td>
      <td>328</td>
      <td>[delivery, pickup]</td>
      <td>https://www.yelp.com/biz/de-mole-astoria-astor...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>mi-espiguita-taqueria-astoria</td>
      <td>[{'alias': 'mexican', 'title': 'Mexican'}]</td>
      <td>{'latitude': 40.7612033639422, 'longitude': -7...</td>
      <td>(718) 777-5648</td>
      <td>714.301080</td>
      <td>yvva7IYpD6J7OfKlCdQrkw</td>
      <td>https://s3-media3.fl.yelpcdn.com/bphoto/_bPuQu...</td>
      <td>False</td>
      <td>{'address1': '32-44 31st St', 'address2': '', ...</td>
      <td>Mi Espiguita Taqueria</td>
      <td>+17187775648</td>
      <td>$</td>
      <td>4.5</td>
      <td>101</td>
      <td>[delivery, pickup]</td>
      <td>https://www.yelp.com/biz/mi-espiguita-taqueria...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>maizal-restaurant-and-tequila-bar-astoria-2</td>
      <td>[{'alias': 'mexican', 'title': 'Mexican'}, {'a...</td>
      <td>{'latitude': 40.759331, 'longitude': -73.926035}</td>
      <td>(718) 406-9431</td>
      <td>900.451091</td>
      <td>QIsFsiOP3H_NkgeWST7GPA</td>
      <td>https://s3-media4.fl.yelpcdn.com/bphoto/VOGwDm...</td>
      <td>False</td>
      <td>{'address1': '3207 34th Ave', 'address2': None...</td>
      <td>Maizal Restaurant &amp; Tequila Bar</td>
      <td>+17184069431</td>
      <td>$$</td>
      <td>4.0</td>
      <td>267</td>
      <td>[delivery, pickup]</td>
      <td>https://www.yelp.com/biz/maizal-restaurant-and...</td>
    </tr>
  </tbody>
</table>
</div>



We could easily change our request slightly to retrieve a larger number at a time.


```python
# client_id = #your id here
# api_key = #your key here

term = 'Mexican'
location = 'Astoria NY'
SEARCH_LIMIT = 50

url = 'https://api.yelp.com/v3/businesses/search'

headers = {
        'Authorization': 'Bearer {}'.format(api_key),
    }

url_params = {
                'term': term.replace(' ', '+'),
                'location': location.replace(' ', '+'),
                'limit': SEARCH_LIMIT
            }
response = requests.get(url, headers=headers, params=url_params)
print(response)
print(type(response.text))
print(response.text[:1000])
```

    <Response [200]>
    <class 'str'>
    {"businesses": [{"id": "jeWIYbgBho9vBDhc5S1xvg", "alias": "holy-guacamole-astoria", "name": "Holy Guacamole", "image_url": "https://s3-media1.fl.yelpcdn.com/bphoto/pMTc8Y0UnRFNh0zDFERfSg/o.jpg", "is_closed": false, "url": "https://www.yelp.com/biz/holy-guacamole-astoria?adjust_creative=xNHtXRpNa-MXGFJJTHHUvw&utm_campaign=yelp_api_v3&utm_medium=api_v3_business_search&utm_source=xNHtXRpNa-MXGFJJTHHUvw", "review_count": 127, "categories": [{"alias": "mexican", "title": "Mexican"}, {"alias": "bars", "title": "Bars"}], "rating": 4.0, "coordinates": {"latitude": 40.756621, "longitude": -73.929336}, "transactions": ["delivery", "pickup"], "price": "$$", "location": {"address1": "3555 31st St", "address2": "", "address3": "", "city": "Astoria", "zip_code": "11106", "country": "US", "state": "NY", "display_address": ["3555 31st St", "Astoria, NY 11106"]}, "phone": "+19178327261", "display_phone": "(917) 832-7261", "distance": 1290.4274875130448}, {"id": "AUyKmFjpaVLwc3awfUnqgQ", "alias": "chela


We still have the same number of matching results, but this time have been given the first 50:


```python
response.json()['total']
```




    671




```python
print(len(response.json()['businesses']))
```

    50


If we want to retrieve more of the results, we page through them by using the offset parameter:


```python
# client_id = #your id here
# api_key = #your key here

term = 'Mexican'
location = 'Astoria NY'
SEARCH_LIMIT = 50

url = 'https://api.yelp.com/v3/businesses/search'

headers = {
        'Authorization': 'Bearer {}'.format(api_key),
    }

url_params = {
                'term': term.replace(' ', '+'),
                'location': location.replace(' ', '+'),
                'limit': SEARCH_LIMIT,
                'offset': 50
            }
response2 = requests.get(url, headers=headers, params=url_params)
print(response2)
print(type(response2.text))
print(response2.text[:1000])
```

    <Response [200]>
    <class 'str'>
    {"businesses": [{"id": "DY2wlHlgC5eHITs-d5rT4w", "alias": "panchos-new-york-2", "name": "Panchos", "image_url": "https://s3-media2.fl.yelpcdn.com/bphoto/vwfG2x08lpXbqe3sybnT-Q/o.jpg", "is_closed": false, "url": "https://www.yelp.com/biz/panchos-new-york-2?adjust_creative=xNHtXRpNa-MXGFJJTHHUvw&utm_campaign=yelp_api_v3&utm_medium=api_v3_business_search&utm_source=xNHtXRpNa-MXGFJJTHHUvw", "review_count": 54, "categories": [{"alias": "mexican", "title": "Mexican"}, {"alias": "breakfast_brunch", "title": "Breakfast & Brunch"}], "rating": 4.5, "coordinates": {"latitude": 40.8015, "longitude": -73.96534}, "transactions": [], "price": "$", "location": {"address1": "964 Amsterdam Ave", "address2": "", "address3": "", "city": "New York", "zip_code": "10025", "country": "US", "state": "NY", "display_address": ["964 Amsterdam Ave", "New York, NY 10025"]}, "phone": "+12123165400", "display_phone": "(212) 316-5400", "distance": 5288.3093583250375}, {"id": "bHJWjEg_sGjrlSHIOX6Mqg", "alias": "two-liz


# Practice

With that, you should have the basics to retrive the full result set!

## API Call Function

Use the example above to write a function to retrieve all of the results (up to the maximum 1000 provided by Yelp) for a given search. Your function should then return the results for these as a single Pandas dataframe.


```python
#Your code here
```


### Pseudocode outline: 

The function should take in url paramaters in some form. From there, the first call should check the number of results and make successive API calls using the offset parameter to cycle through these results. Each response should be stored as a DataFrame which will then be stitched together.

Warning: Making too many API calls can lead to errors. Most APIs require you to slow down your requests to a manageable limit. Make sure to use the time.sleep() method from the time package to make some brief pauses (~5 seconds is more then sufficient) between successive calls.

## Neighborhood ______ Restaurants

Use your function above to retrieve all of the restaurants for a particular cuisine in a neighborhood of your choice.


```python
#Your code here
```

## Exploratory Analysis

Take the restaurants from the previous question and do an intial exploratory analysis. At minimum, this should include looking at the distribution of features such as price, rating and number of reviews as well as the relations between these dimensions.


```python
#Your code here
```

## Mapping

Look at the initial Yelp example and try and make a map using Folium of the restaurants you retrieved.


```python
#Your code here
```
