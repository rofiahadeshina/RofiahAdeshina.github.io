---
layout: post
title: "Major Social Media Influencers Including Government Officials"
date: 2020-07-23
---
### Introduction
Social media campaigns as other kinds are driven towards improving sales of product.
Trends are set each day on social media networks mostly by influencial individuals. This blog post focuses on Twitter as a means to identify these individuals making use of their popularity, reach and relevance scores. It also highlights the major hashtags used by these individuals.

### Methodology
#### Web Scrapping

Web scrapping is a process of data extraction. It is extracting data from HTML using the Beautiful Soup, a third party library for parsing HTML and XML documents. For this analysis, I extracted data from the following websites: [Top African Influencers](https://africafreak.com/100-most-influential-twitter-users-in-africa) and [African Top Government Influencers](https://www.atlanticcouncil.org/blogs/africasource/african-leaders-respond-to-coronavirus-on-twitter/#east-africa). 

I made use of this code to scrap the websites:
```
#data extraction functions
def simple_get(url):
    """
    Attempts to get the content at `url` by making an HTTP GET request.
    If the content-type of response is some kind of HTML/XML, return the
    text content, otherwise return None.
    """
    try:
        with closing(get(url, stream=True)) as resp:
            if is_good_response(resp):
                return resp.content  #.encode(BeautifulSoup.original_encoding)
            else:
                return None

    except RequestException as e:
        log_error('Error during requests to {0} : {1}'.format(url, str(e)))
        return None


def is_good_response(resp):
    """
    Returns True if the response seems to be HTML, False otherwise.
    """
    content_type = resp.headers['Content-Type'].lower()
    return (resp.status_code == 200 
            and content_type is not None 
            and content_type.find('html') > -1)


def log_error(e):
    """
    It is always a good idea to log errors. 
    This function just prints them, but you can
    make it do anything.
    """
    print(e)
    
def get_elements(url, tag='',search={}, fname=None):
    """
    Downloads a page specified by the url parameter
    and returns a list of strings, one per tag element
    """
    
    if isinstance(url,str):
        response = simple_get(url)
    else:
        #if already it is a loaded html page
        response = url

    if response is not None:
        html = BeautifulSoup(response, 'html.parser')
        
        res = []
        if tag:    
            for li in html.select(tag):
                for name in li.text.split('\n'):
                    if len(name) > 0:
                        res.append(name.strip())
                       
                
        if search:
            soup = html            
            
            
            r = ''
            if 'find' in search.keys():
                print('findaing',search['find'])
                soup = soup.find(**search['find'])
                r = soup

                
            if 'find_all' in search.keys():
                print('findaing all of',search['find_all'])
                r = soup.find_all(**search['find_all'])
   
            if r:
                for x in list(r):
                    if len(x) > 0:
                        res.extend(x)
            
        return res

    # Raise an exception if we failed to get any data from the url
    raise Exception('Error retrieving contents at {}'.format(url))    
    
    
if get_ipython().__class__.__name__ == '__main__':
    fire(get_tag_elements)
    
#pulling data from the site into a list
res = get_elements('https://africafreak.com/100-most-influential-twitter-users-in-africa', tag='h2')

#removing irrelevant elements
for i in res[100:]:
    res.remove(i)

#getting handles and names
names_infl = []
handle_infl = []
for r in res:
    split_data = r.split('.',maxsplit=1)[1].rsplit('(',maxsplit=1)
    print(split_data)
    name = split_data[0].split(',')[0].strip()
    handle =  split_data[1].split(')',maxsplit=1)[0]
    names_infl.append(name)
    handle_infl.append(handle)
```

The same process it carried out for the Goverment Officials
```
url= 'https://www.atlanticcouncil.org/blogs/africasource/african-leaders-respond-to-coronavirus-on-twitter/#east-africa'
response = get(url).content
re_gov = get_elements(response, tag='blockquote')
names = []
handles = []
for r in re_gov:
    split_data = r.split('— ',maxsplit=1)[1].rsplit('(',maxsplit=1)
    name = split_data[0].split(',')[0].strip()
    handle =  split_data[1].rsplit(')',maxsplit=1)[0]
    names.append(name)
    handles.append(handle)

nam_handle = f'{name}:{handle}'
```

### Twitter Mining
After compiling the list of handles, the next step is to mine their data from twitter. Here I will be making use of the tweepy library. To know more about the tweepy library, chech this documentation [Tweepy API documentation](http://docs.tweepy.org/en/v3.5.0/api.html). For easy analysis, I combined the lists (ie handles and handle_infl) into one fl_handle.

To mine twitter data, these libraries are essential. I also included a line of code that help to install the tweepy library
```#!pip install tweepy

import tweepy
from tweepy.streaming import StreamListener
from tweepy import OAuthHandler
from tweepy import Stream


```
These credentials ("secret keys") are from my twitter developer's account. Therefore a twitter developer account is needed to get these credentials  
```
#initialize custom twitter keys
consumer_key = "secret key"
consumer_secret = "secret key"
access_token = "secret key"
access_token_secret = "secret key"
```
Set up an API to access information about the Twitter handles I collected from those websites

```
auth = OAuthHandler(consumer_key, consumer_secret)
auth.set_access_token(access_token, access_token_secret)
api = tweepy.API(auth)
```
For this analysis, the data minned are: the followers_count, number of likes, retweet_count, friends_count, mentions and hashtags.
```# getting followers for influencers
# Calling the get_user function with our parameters
followers = []
for i in fl_handles:
    try:
        results = api.get_user(id=i)
    except tweepy.TweepError as e:
        continue 
    followers.append(results.followers_count)
```
```
# getting no of likes for influencers
likes = []
for i in fl_handles:
    try:
        results = api.get_user(id=i)
    except tweepy.TweepError as e:
        continue
    likes.append(results.favourites_count)
```

```
# getting no of following for influencers
following = []
for i in fl_handles:
    try:
        results = api.get_user(id=i)
    except tweepy.TweepError as e:
        continue
    following.append(results.friends_count)


#getting retweets
no_of_retweets = []
for id in fl_handles:
    try:
        tweets = tweepy.Cursor(api.user_timeline, id=i).items()
        for tweet in tweets:
            no_of_retweets.append(tweet.retweet_count)
    except tweepy.TweepError as e:
        continue
        
#mention for gov influencers
count = []
for x in range(0, len(fl_handles)):
    name = fl_handles[x]
    mentions_count = []
    try:
       for status in tweepy.Cursor(api.user_timeline, id=name).items():
         entities = status.entities
         if "user_mentions" in entities:
            for ent in entities["user_mentions"]:
              if ent is not None:
                if "screen_name" in ent:
                  name = ent["screen_name"]
                  if name is not None:
                    mentions_count.append(name)
    except tweepy.TweepError as e:
        continue
    count.append(len(mentions_count))        
        
```
 
### Results
Having the data from twitter, I determine the popularity and reach of each influencer.
To determine each influencer's reach, I made use of the of a formula: Reach Score = Number of followers - Number of following. The results are then sorted to determine the influencers with the highest reach. The same logic is used to determine the popularity however, popularity score = 

#### Reach
```
#Reach Score = followers - following
reach = pd.concat([total_followers,gov_following], axis=1)
reach['reach_score']= reach["Number of followers"] - reach["Number of following"]

```

#### Popularity
```
#popularity reach = retweets + likes
popularity = pd.concat([gov_retweets,total_like], axis=1)
popularity["Popularity_score"] = popularity["No of retweets"]+popularity["Number of likes"]
```
 

 
 

