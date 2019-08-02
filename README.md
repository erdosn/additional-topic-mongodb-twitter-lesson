
### Questions

### Objectives
YWBAT
- apply twitter's streamlistener to gather data on a continuous basis
- store data as csv format and save as file

### Outline


```python
import json # used to read twitter data
import pandas as pd
import numpy as np
import tweepy as tw

from pprint import pprint
from pymongo import MongoClient

import matplotlib.pyplot as plt
```

### Load our twitter permissions and connect to Twitter using Tweepy


```python
# use 'with' to open file because it automatically closes files for you
with open("/Users/rafael/.ssh/twitter_app00.json") as f:
    d = json.load(f)
print(d.keys())
```

    dict_keys(['consumer_key', 'consumer_secret', 'access_token', 'access_token_secret'])



```python

```


```python
# create your auth
auth = tw.OAuthHandler(consumer_key=d["consumer_key"], consumer_secret=d["consumer_secret"])
```


```python
# pass in key and secret to your auth handler
auth.set_access_token(key=d["access_token"], secret=d["access_token_secret"])
```


```python
# create our api (the thing we use to interact with twitter) by passing in our auth
api = tw.API(auth_handler=auth)
```


```python
# check connection with basic search (uncomment lines below to run)
# for res in api.search("Drake", tweet_mode='extended'):
#     pprint(res._json)
#     break
```

### Let's build a StreamListener to get more and more tweets


```python
# Build our StreamListener
# Twitter requires you to populate it's methods
```


```python
# child class of StreamListener class

class MyStreamListener(tw.StreamListener):

    def on_status(self, status):
        print(status.text)
```


```python
# this is our listener
myStreamListener = MyStreamListener()
myStream = tw.Stream(auth = api.auth, listener=myStreamListener, tweet_mode='extended')
```


```python
# myStream.filter(track=["drake", "Drake", "#carepackage", "#drake", "#Drake"])
```

### Let's store tweets in a mongodb

### Step 1: run `mongod` in terminal and connect to it using pymongo.MongoClient


```python
client = MongoClient(port=27017, host='localhost')
```


```python
# storing names of dbs to a list
db_names_list = client.list_database_names()
```


```python
# mongo is built on json formatting so everything reads like a dictionary
music_tweets = client["music_tweets"]
```


```python
# new collection/table called drake
drake = music_tweets["drake"]
```


```python
# notice we don't see the drake collection yet...why?
# this is because a collection needs a document in order to get built
music_tweets.list_collection_names()
```




    ['kendrickLamar']



### rewrite our on_status method from MyStreamListener to write into our db


```python
# edited for our mongodb child class of StreamListener class

class MyStreamListener(tw.StreamListener):

    def on_status(self, status):
        j = status._json
        drake.insert_one(j)
        print(f'inserted tweet: {j["text"]}')
        pass
```

### Let's store tweets in our drake collection in the music_tweets db



```python
# rerun our myStream object to reset the memory
myStreamListener = MyStreamListener()
myStream = tw.Stream(auth = api.auth, listener=myStreamListener, tweet_mode='extended')
```


```python
myStream.filter(track=["drake", "Drake", "#carepackage", "#drake", "#Drake"])
```

### let's get some numbers on our new data


```python
music_tweets.list_collection_names()
```




    ['drake', 'kendrickLamar']




```python
drake.count_documents(filter={}) #filter={} -> 'select * from drake'
```




    2630




```python
# let's view these tweets
for res in drake.find(filter={}):
    print(res)
    break
# excuse the language
```

    {'_id': ObjectId('5d44694751483225fdac9acc'), 'created_at': 'Fri Aug 02 16:48:02 +0000 2019', 'id': 1157332236661866500, 'id_str': '1157332236661866500', 'text': 'RT @KenTheRuthless: This nigga Drake has me hyped over songs I’ve heard before. 😪 I hate this nigga.', 'source': '<a href="http://twitter.com/download/iphone" rel="nofollow">Twitter for iPhone</a>', 'truncated': False, 'in_reply_to_status_id': None, 'in_reply_to_status_id_str': None, 'in_reply_to_user_id': None, 'in_reply_to_user_id_str': None, 'in_reply_to_screen_name': None, 'user': {'id': 744747281547595776, 'id_str': '744747281547595776', 'name': 'Cash Marqtier 💷💵💶', 'screen_name': 'marqthemilkman', 'location': 'Sandy Springs, GA', 'url': 'http://soundcloud.com/augustmarquis', 'description': 'its like conquistador but Marquis instead 🐎Father of Fashion. Musical artist', 'translator_type': 'none', 'protected': False, 'verified': False, 'followers_count': 136, 'friends_count': 356, 'listed_count': 0, 'favourites_count': 5229, 'statuses_count': 1543, 'created_at': 'Mon Jun 20 04:22:43 +0000 2016', 'utc_offset': None, 'time_zone': None, 'geo_enabled': False, 'lang': None, 'contributors_enabled': False, 'is_translator': False, 'profile_background_color': 'F5F8FA', 'profile_background_image_url': '', 'profile_background_image_url_https': '', 'profile_background_tile': False, 'profile_link_color': '1DA1F2', 'profile_sidebar_border_color': 'C0DEED', 'profile_sidebar_fill_color': 'DDEEF6', 'profile_text_color': '333333', 'profile_use_background_image': True, 'profile_image_url': 'http://pbs.twimg.com/profile_images/1118458235818790913/ze3l5TG-_normal.jpg', 'profile_image_url_https': 'https://pbs.twimg.com/profile_images/1118458235818790913/ze3l5TG-_normal.jpg', 'profile_banner_url': 'https://pbs.twimg.com/profile_banners/744747281547595776/1545879537', 'default_profile': True, 'default_profile_image': False, 'following': None, 'follow_request_sent': None, 'notifications': None}, 'geo': None, 'coordinates': None, 'place': None, 'contributors': None, 'retweeted_status': {'created_at': 'Thu Aug 01 21:56:49 +0000 2019', 'id': 1157047557819949057, 'id_str': '1157047557819949057', 'text': 'This nigga Drake has me hyped over songs I’ve heard before. 😪 I hate this nigga.', 'source': '<a href="http://twitter.com/download/iphone" rel="nofollow">Twitter for iPhone</a>', 'truncated': False, 'in_reply_to_status_id': None, 'in_reply_to_status_id_str': None, 'in_reply_to_user_id': None, 'in_reply_to_user_id_str': None, 'in_reply_to_screen_name': None, 'user': {'id': 326281322, 'id_str': '326281322', 'name': '𝔊𝔯𝔦𝔪 𝔗𝔥𝔦𝔯𝔱𝔢𝔢𝔫', 'screen_name': 'KenTheRuthless', 'location': 'Atlanta', 'url': None, 'description': '☠️ 1397 ☠️', 'translator_type': 'none', 'protected': False, 'verified': False, 'followers_count': 2089, 'friends_count': 815, 'listed_count': 6, 'favourites_count': 3087, 'statuses_count': 89349, 'created_at': 'Wed Jun 29 17:26:57 +0000 2011', 'utc_offset': None, 'time_zone': None, 'geo_enabled': True, 'lang': None, 'contributors_enabled': False, 'is_translator': False, 'profile_background_color': 'C0DEED', 'profile_background_image_url': 'http://abs.twimg.com/images/themes/theme1/bg.png', 'profile_background_image_url_https': 'https://abs.twimg.com/images/themes/theme1/bg.png', 'profile_background_tile': True, 'profile_link_color': 'DD2E44', 'profile_sidebar_border_color': 'FFFFFF', 'profile_sidebar_fill_color': '210CE3', 'profile_text_color': '03FF8E', 'profile_use_background_image': True, 'profile_image_url': 'http://pbs.twimg.com/profile_images/1147530958704402432/sNtJ2whH_normal.jpg', 'profile_image_url_https': 'https://pbs.twimg.com/profile_images/1147530958704402432/sNtJ2whH_normal.jpg', 'profile_banner_url': 'https://pbs.twimg.com/profile_banners/326281322/1554696484', 'default_profile': False, 'default_profile_image': False, 'following': None, 'follow_request_sent': None, 'notifications': None}, 'geo': None, 'coordinates': None, 'place': None, 'contributors': None, 'is_quote_status': False, 'quote_count': 616, 'reply_count': 51, 'retweet_count': 10483, 'favorite_count': 30512, 'entities': {'hashtags': [], 'urls': [], 'user_mentions': [], 'symbols': []}, 'favorited': False, 'retweeted': False, 'filter_level': 'low', 'lang': 'en'}, 'is_quote_status': False, 'quote_count': 0, 'reply_count': 0, 'retweet_count': 0, 'favorite_count': 0, 'entities': {'hashtags': [], 'urls': [], 'user_mentions': [{'screen_name': 'KenTheRuthless', 'name': '𝔊𝔯𝔦𝔪 𝔗𝔥𝔦𝔯𝔱𝔢𝔢𝔫', 'id': 326281322, 'id_str': '326281322', 'indices': [3, 18]}], 'symbols': []}, 'favorited': False, 'retweeted': False, 'filter_level': 'low', 'lang': 'en', 'timestamp_ms': '1564764482070'}


### Now let's load these into a dataframe


```python
document_list = list(drake.find(filter={}))
df = pd.DataFrame(document_list) # list of jsons
df.head()
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
      <th>_id</th>
      <th>contributors</th>
      <th>coordinates</th>
      <th>created_at</th>
      <th>display_text_range</th>
      <th>entities</th>
      <th>extended_entities</th>
      <th>extended_tweet</th>
      <th>favorite_count</th>
      <th>favorited</th>
      <th>...</th>
      <th>quoted_status_permalink</th>
      <th>reply_count</th>
      <th>retweet_count</th>
      <th>retweeted</th>
      <th>retweeted_status</th>
      <th>source</th>
      <th>text</th>
      <th>timestamp_ms</th>
      <th>truncated</th>
      <th>user</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>5d44694751483225fdac9acc</td>
      <td>None</td>
      <td>None</td>
      <td>Fri Aug 02 16:48:02 +0000 2019</td>
      <td>NaN</td>
      <td>{'hashtags': [], 'urls': [], 'user_mentions': ...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>...</td>
      <td>NaN</td>
      <td>0</td>
      <td>0</td>
      <td>False</td>
      <td>{'created_at': 'Thu Aug 01 21:56:49 +0000 2019...</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>RT @KenTheRuthless: This nigga Drake has me hy...</td>
      <td>1564764482070</td>
      <td>False</td>
      <td>{'id': 744747281547595776, 'id_str': '74474728...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>5d44694751483225fdac9acd</td>
      <td>None</td>
      <td>None</td>
      <td>Fri Aug 02 16:48:02 +0000 2019</td>
      <td>NaN</td>
      <td>{'hashtags': [], 'urls': [], 'user_mentions': ...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>...</td>
      <td>NaN</td>
      <td>0</td>
      <td>0</td>
      <td>False</td>
      <td>{'created_at': 'Thu Aug 01 23:33:13 +0000 2019...</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>RT @Jersey_Jinx: I'll also say that Drake's mu...</td>
      <td>1564764482017</td>
      <td>False</td>
      <td>{'id': 2847895388, 'id_str': '2847895388', 'na...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>5d44694751483225fdac9ace</td>
      <td>None</td>
      <td>None</td>
      <td>Fri Aug 02 16:48:02 +0000 2019</td>
      <td>NaN</td>
      <td>{'hashtags': [], 'urls': [], 'user_mentions': ...</td>
      <td>{'media': [{'id': 1157042637473325056, 'id_str...</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>...</td>
      <td>NaN</td>
      <td>0</td>
      <td>0</td>
      <td>False</td>
      <td>{'created_at': 'Fri Aug 02 13:46:54 +0000 2019...</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>RT @willis_cj: bringing this gem back to the T...</td>
      <td>1564764482155</td>
      <td>False</td>
      <td>{'id': 543577551, 'id_str': '543577551', 'name...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>5d44694751483225fdac9acf</td>
      <td>None</td>
      <td>None</td>
      <td>Fri Aug 02 16:48:02 +0000 2019</td>
      <td>NaN</td>
      <td>{'hashtags': [], 'urls': [], 'user_mentions': ...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>...</td>
      <td>NaN</td>
      <td>0</td>
      <td>0</td>
      <td>False</td>
      <td>{'created_at': 'Fri Aug 02 04:18:54 +0000 2019...</td>
      <td>&lt;a href="http://twitter.com/download/android" ...</td>
      <td>RT @big_business_: Drake was giving us toxic s...</td>
      <td>1564764482544</td>
      <td>False</td>
      <td>{'id': 113987040, 'id_str': '113987040', 'name...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5d44694751483225fdac9ad0</td>
      <td>None</td>
      <td>None</td>
      <td>Fri Aug 02 16:48:02 +0000 2019</td>
      <td>NaN</td>
      <td>{'hashtags': [], 'urls': [], 'user_mentions': ...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>0</td>
      <td>False</td>
      <td>...</td>
      <td>NaN</td>
      <td>0</td>
      <td>0</td>
      <td>False</td>
      <td>{'created_at': 'Fri Aug 02 06:07:28 +0000 2019...</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>RT @zephaniiiah: this old drake hitting harder...</td>
      <td>1564764482555</td>
      <td>False</td>
      <td>{'id': 2516793480, 'id_str': '2516793480', 'na...</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 37 columns</p>
</div>



### Assessment


```python

```
