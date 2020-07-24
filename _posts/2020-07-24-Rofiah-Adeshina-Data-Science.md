---
layout: post
title: "Twitter Analysis of Social Media Influencers Including Government Officials"
date: 2020-07-23
---
## Introduction
Social media campaigns as other kinds are driven towards improving sales of product.
Trends are set each day on social media networks mostly by influencial individuals. This blog post focuses on Twitter as a means to identify these individuals making use of their popularity and reach scores. It also highlights the major hashtags used by these individuals.

## Methodology
#### Web Scrapping

Web scrapping is a process of data extraction. It is extracting data from HTML using the Beautiful Soup, a third party library for parsing HTML and XML documents. For this analysis, I extracted data from the following websites: [Top African Influencers](https://africafreak.com/100-most-influential-twitter-users-in-africa) and [African Top Government Influencers](https://www.atlanticcouncil.org/blogs/africasource/african-leaders-respond-to-coronavirus-on-twitter/#east-africa). 

I made use of this code to scrap the websites:
```
*#data extraction functions*
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
 
```
Pulling data from the top 100 influencers site into a list
```
res = get_elements('https://africafreak.com/100-most-influential-twitter-users-in-africa', tag='h2')
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
```

## Twitter Mining
After compiling the list of handles, the next step is to mine their data from twitter. Here, I will be making use of the tweepy library. Check this documentation [Tweepy API documentation](http://docs.tweepy.org/en/v3.5.0/api.html) to get more information on the use of tweepy. For easy analysis.

In mine twitter data, these libraries are essential
```
import tweepy
from tweepy.streaming import StreamListener
from tweepy import OAuthHandler
from tweepy import Stream
```
These credentials ("secret keys") are from my twitter developer's account. Therefore a twitter developer account is needed to get these credentials  
```
*initialize custom twitter keys*
consumer_key = "secret key"
consumer_secret = "secret key"
access_token = "secret key"
access_token_secret = "secret key"
```
Setting up an API to access information about the Twitter handles I collected from those websites

```
auth = OAuthHandler(consumer_key, consumer_secret)
auth.set_access_token(access_token, access_token_secret)
api = tweepy.API(auth)
```
For this analysis, the data minned are: the followers_count, number of likes, retweet_count, friends_count and hashtags.
```
*getting followers for influencers*
*Calling the get_user function with our parameters*
followers = []
for i in handles:
    try:
        results = api.get_user(id=i)
    except tweepy.TweepError as e:
        continue 
    followers.append(results.followers_count)

*getting no of likes for influencers*
likes = []
for i in handles:
    try:
        results = api.get_user(id=i)
    except tweepy.TweepError as e:
        continue
    likes.append(results.favourites_count)

*getting no of following for influencers*
following = []
for i in handles:
    try:
        results = api.get_user(id=i)
    except tweepy.TweepError as e:
        continue
    following.append(results.friends_count)


*getting retweets*
no_of_retweets = []
for id in handles:
    try:
        tweets = tweepy.Cursor(api.user_timeline, id=i).items()
        for tweet in tweets:
            no_of_retweets.append(tweet.retweet_count)
    except tweepy.TweepError as e:
        continue               
```
 
### Data Visualization
Having the data from twitter, I determine the popularity and reach of each influencer.
To determine each influencer's reach, I made use of the of a formula: Reach Score = Number of followers - Number of following. The results are then sorted to determine the influencers with the highest reach. The same logic is used to determine the popularity however, popularity score = 

#### Reach
```
*Reach Score = followers - following*
reach = pd.concat([total_followers,gov_following], axis=1)
reach['reach_score']= reach["Number of followers"] - reach["Number of following"]
```
![image](https://user-images.githubusercontent.com/65109526/88411765-68997300-cdd0-11ea-97dc-46ecce0b5ce0.png "Government Official with Highest Reach")


#### Popularity
```
*popularity reach = retweets + likes*
popularity = pd.concat([gov_retweets,total_like], axis=1)
popularity["Popularity_score"] = popularity["No of retweets"]+popularity["Number of likes"]
```
![image](https://user-images.githubusercontent.com/65109526/88410966-055b1100-cdcf-11ea-995f-4bdaadc36cee.png "Most Popular Government Officials")

The same can be done for the top 100 Influencers to determine which one of them is the most popular and has the highest reach.
 
#### Hashtags
Hashtags are used to set trends on twitter. To determine the major hashtags that these officials and top influencers use, I made use of the code below:
```
hashtags=[]
if len(fl_handles) > 0:
    for handle in handles:
        value_list = {}
        *print("Getting hashtags for " + handle)
        * this helps avoid Tweepy errors like suspended users or user not found errors
        try:
            for status in tweepy.Cursor(api.user_timeline, id=handle).items():
                if hasattr(status, "entities"):
                    entities = status.entities
                    if 'hashtags' in entities:
                        for ent in entities['hashtags']:
                            if ent is not None:
                                if "text" in ent:
                                    hashtag = ent["text"]
                                    if hashtag is not None:
                                        hashtags.append(hashtag)
        except tweepy.TweepError as e:
            continue
```
Getting the hashtags and count for each hashtag, I was able to derive this plot, which cleary shows #covid19 as the most popular hashtag being used with over 400 count
![image](https://user-images.githubusercontent.com/65109526/88413153-9a133e00-cdd2-11ea-8e99-fb66ae9c1100.png "hashtags")

### Conclusion
The popularity and reach of an individual are major factors that determine the influence they have on social media. This information is useful to companies as it can help them push their brands on social mediaa and increase sales by partnering with these top influencers. 
