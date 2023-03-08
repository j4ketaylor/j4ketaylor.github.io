---
layout: post
title:  "Japan popularity map"
date:   2023-03-08
last_modified_at: 2023-03-08
categories: [Python projects]
#tags: [Getting Start]
---
  
In this post, I'll be detailing a short python program that scrapes [r/JapanTravel][r/JapanTravel] on Reddit for information about the frequency about how much different areas are mentioned.

Firstly, we log in to Reddit, and generate a string of n posts

```python
import praw

reddit_read_only = praw.Reddit(client_id="hauy5ee-x3IMJJwoa6JnJQ",
                               client_secret="K35WWmARpqyMDKQR_YY2XdijhvizfQ",
                               user_agent="j4ket1235",
                               check_for_async=False)
 
def JapanTravel(postnum):

    subreddit = reddit_read_only.subreddit("JapanTravel")
    
    postString = "" 
    
    for post in subreddit.top(limit=postnum):
        postString += post.selftext

        
    return postString
```

Next, we parse this information, and compare each word against a csv file of places in Japan.

```python
from pandas import *
from string import *
import scrapeReddit

def getData(post_num):
    string = scrapeReddit.JapanTravel(post_num)
    string = string.title()
    
    data = read_csv("japanstuff.csv")
    cities = data['city'].tolist()
    cities_tracker = [0] * len(cities)
        
    list1 = []
    
    for i in range(0,len(string)-1):
            if string[i] == "/" or string[i] == "." or string[i] == ","
             or string[i] == "-" or string[i] == "(" or string[i] == ")"
              or string[i] == "!" or string[i] == ':'or string[i] == "\n":
                list1.append(" ")
            else:
                list1.append(string[i])
                
    print("25%")
    
    while str("\n") in list1:
        list1.remove("\n")
    
    list2 = "".join(list1)
    list3 = list2.split(' ')
    
    j = 0
    for i in list3:
        if i in cities:
            cities_tracker[cities.index(i)] = cities_tracker[cities.index(i)] + 1
            
    j = 0   
    
    print("75%")
      
    for i in cities_tracker:
        if i > 0:
            print(str(i) + " " + cities[j])
        j += 1
    
    
    for i in range(0,len(cities_tracker)):
        if(cities_tracker[i])/cities_tracker[0] < 0.05:
            data = data.drop(i)  
        else:
            print(cities[i] + " " + str(((cities_tracker[i])/cities_tracker[0])*100) + "%")
            data.at[i,'times_seen']=(cities_tracker[i])/cities_tracker[0]
            
    print("Complete!")
    
    
    data.to_csv("finaljapan.csv", encoding='utf-8', index=False)
    return
```

Finally, we plot a map of everything.

```python
import numpy
import pandas
import matplotlib.pyplot as plt
import geopandas
import gatherData

gatherData.getData(30)



df = pandas.read_csv("finaljapan.csv",usecols=["city","lat","lng","times_seen"])
df.head()

countries = geopandas.read_file(geopandas.datasets.get_path("naturalearth_lowres"))
countries.head()


fig, ax = plt.subplots(figsize=(40,30))

countries[countries["name"] == "Japan"].plot(color="lightgrey",ax=ax)


df.plot(x="lng",y="lat",kind="scatter",c="times_seen",colormap="YlOrRd",ax=ax)

for i, txt in enumerate(df.city):
   ax.annotate(txt, (df.lng[i]+0.05, df.lat[i]))

plt.show()

plt.savefig()

```

Running the final code generates this map:

![Japan Map](/assets/images/japan_map.png)




[r/JapanTravel]: https://www.reddit.com/r/JapanTravel/
