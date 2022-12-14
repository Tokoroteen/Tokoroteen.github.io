---
title: "Scraping Japanese activity search sites in search of undiscovered activities"
date: 2022-12-13T18:10:10+09:00
# draft: true
categories:
- Blog
tags:
- Python
- scraping
- BeautifulSoup
- Requests
thumbnailImagePosition: left
thumbnailImage: https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2704675/9b146bef-8190-1813-93c9-ab67e5ad1086.png
---
# 1. Outline
I love hands-on activities. I love surfing, going to see Yakusugi cedar trees, making snow globes, preparing perfumes, and so on.
I saw an article, [machine learning from the information in the food log](https://qiita.com/toshiyuki_tsutsui/items/f143946944a428ed105b), that uses machine learning to make ramen recommendations based on information from the food log, and I thought it would be fun to do an activity version of this! I thought it would be fun to do an activity version of this!

When you are looking for activities, I think you follow a three-step below.

1. think about your thoughts about the holiday, such as "I'm stressed out and I want to feel refreshed".
1. think about how you want to spend your holiday, e.g. "I want to go bungee jumping to feel refreshed" etc.
1. find an activity and book it

In this thought flow, I feel that each person's creativity(?) is required to move from 1 to 2. I feel that the transition from 1 to 2 requires creativity from each person. Of course, it's not uncommon for people to make plans based on what they want to do, such as "I want to go bungee jumping," but I think it takes a lot of imagination and creativity to go from a desire for a holiday, such as **"I want to feel refreshed,"** to coming up with a way to spend time, such as **"I think this activity is good for that. I think it is a work that requires imagination and creativity."**
In fact, when I tell my friends about these experiences I've had, it's not uncommon for them to say, "How did you find that?"

So, I'd like to use machine learning to **shape your thoughts on holidays into activities**!
I want to build a model that learns from word-of-mouth information, and when you enter the thought "I want to feel refreshed", it will tell you "I recommend XXX activity for that"!

This time, I'm collecting study data for this purpose.
As a site for researching and booking hands-on activities, [Asoview!](https://www.asoview.com/) is probably the most famous in Japan. As far as I can tell, AssoView! I couldn't find any articles about scraping with it, so I'm going to give it a try!

The finished app is here. (Only Japanese search is available.)

https://lifac.herokuapp.com/

For example, when you want to "fly in the sky" (you can add the sea element after the search and subtract the mountain element!)

![screenshot 2022-11-03 13.50.37.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2704675/9b146bef-8190-1813-93c9-ab67e5ad1086.png)

For example, when you want to be "healed by animals".

![Screenshot 2022-11-03 13.50.48.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2704675/4ce7ede4-a952-443b-59ba-807ceec0f2f2.png)

# 2. Take a walk on the homepage of Asoview!

First, I needed to see how the activity information is listed in Asoview! to see how the activity information is listed.
For example, if you're looking at [Moomin Valley Park](https://www.asoview.com/base/156897/), which is one of the most famous theme park in Saitama prefecture, you'll see  something like this.
![Moomin.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2704675/32389a9d-6ba9-767c-de7e-81f36daa2078.jpeg)

You can see that the yellow circled area contains the name of the facility, a review, the location of the facility, and the genre (in this case, amusement park/theme park). The genres include paragliding, athletics, incense making, and so on. We will attempt to scrape these information later.

The URL for this page is

**https://www.asoview.com/base/156897/**

and
For the other URL is
- tea ceremony class https://www.asoview.com/base/155331/
- Paragliding https://www.asoview.com/base/960/

For each facility
`https://www.asoview.com/base/<<facility unique number>>/`
so I'd like to scrape it as I change the `<<facility unique number>>`.

I'm struggling to figure out what rules are used for the `<<facility unique number>>` here 😅.
Some numbers are 6, some are 155331, and there is a wide range from 1 to 6 digits. I wandered around the homepage to see if the numbers were assigned based on some rules, but I couldn't find any rules after all...

But if I accessed the site while changing the numbers from 1 to 99999999, it would be in trouble.
Then, I found it! [Here](https://www.asoview.com/base/) is the list of facilities!
![List of Locations.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2704675/585daaee-85e9-8481-7f66-0af7a04dd686.jpeg)
I'm going to extract each facility's unique number from this page!

# 3. Extract the list of facilities

First, let's try to get the `<<facility unique number>>` (henceforth referred to as the activity number) from the previous site.
If you right click on the site > Verify, you will see
![kenshou.jpeg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2704675/891ed2af-5fa5-8e67-41b1-947bb207158e.jpeg)

You can see how the site is written in HTML. You can also check the structure of the site element by element here.
![kenshou2.jpeg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2704675/34ede00e-c592-dd51-f559-77bffb1b240e.jpeg)

![base_links.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2704675/22729b4a-2d1d-81f0-5b1c-877aa91a9938.jpeg)

First of all, get all activitiy No. from this a tab.
Import what you need
```python:base_data_scraping.py
import requests
from bs4 import BeautifulSoup
import re
```

By using Requests, you can get HTML, which is then parsed by BeautifulSoup.
```python:base_data_scraping.py
url= "https://www.asoview.com/base/"
n_list = [] #Put the activity no. in this list

res = requests.get(url) #Get the HTML
soup = BeautifulSoup(res.text, "html.parser") #pass to BeautifulSoup
links = soup.find_all("a", class_="page-base__base-link") #All a tabs whose class is "page-base__base-link"

n_list = [int(re.sub("\D*", "", link.get('href'))) for link in links]
```
Get the href in the a tab with `link.get('href')` and extract only the numbers with [regular expression](https://qiita.com/luohao0404/items/7135b2b96f9b0b196bf3).
If you check the contents with `print(n_list)`, you will see that
```
[163, 178, 191, 232, 260, 261, 263, 264, 281, 314, .....]
```
and I could get the activity No. well. From `len(n_list)`, you can see that the number of activity listings is 8786. We will use this `n_list` to get the basic information of each activity.

# 4. Basic information about each activity

If you look at the page of [Tea ceremony class](https://www.asoview.com/base/155331/) in the activity No.155331, you will see that...
![sadou.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2704675/5f6bdee8-e297-8a0a-5235-e77d0e99c221.jpeg)
The name of the facility, number of ratings/evaluations, location > detailed location, genre of the activity, and a description of the facility are listed, so we will retrieve this information for each activity and compile it into one file.

Import the missing stuff, and then run
```python:base_data_scraping.py
import time
import pandas as pd
```
Create a DataFrame from which to add the retrieved information.
```python:base_data_scraping.py
#Create a DataFrame.
columns = ["name", "area", "small_area", "genre_type", "rating_star", "rating_asorepo", "introduction_title", "introduction_text", "No." "url", "review_link"]
df = pd.DataFrame(index=[], columns=columns)
```

I got the ``https://www.asoview.com/base/<Activity No.>/` link and the link to the review page together.

Just like when I got the activity No., for example, if it is the name of the facility, we can see that the class of the h1 tag is "base-name".
![base_name.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2704675/aa0b65f0-be38-7f08-b6d2-c9915de1650a.jpeg)

Thus, you can get the facility name by `soup.find("h1", class_="base-name").text`.
In this way, you will get other values as well.

```python:base_data_scraping.py
for n in n_list: #each activity No.
  try:
    url = f"https://www.asoview.com/base/{n}/"
    res = requests.get(url)
    soup = BeautifulSoup(res.content, "html.parser")
    time.sleep(1)
    res.raise_for_status()
    print(f'There is a URL in No.{n}')

    base_name = soup.find("h1", class_="base-name").text #name of facility
    base_data = soup.find("div", id="base-summary-contents")
    base_data_area = base_data.find("span", class_="base-data__area").text #location
    base_data_small_area = base_data.find("span", class_="base-data__small-area").text #detail location

    #There can be more than one genre.
    base_data_genre_type = base_data.find_all("li", class_="base-data__genre-type") #genre
    base_data_genre_type_list = list(i.text for i in base_data_genre_type)

    review_link = base_data.find("a").get("href") #link to review

    #rating
    base_data_rating = base_data.find("i", class_="base-data__rating")
    base_data_rating_star = float(base_data_rating.contents[3].text) #num of star
    base_data_rating_asorepo = int(int(base_data_rating.contents[5].text.replace(",", ""))) #num of review

    introduction_text_title = soup.find("b", class_="introduction__text-title").text #introduction title
    introduction_text = soup.find("p", class_="introduction__text").text #introduction

    #df to add on
    df_new = pd.DataFrame([[base_name, base_data_area, base_data_small_area, base_data_genre_type_list, base_data_rating_star, base_data_rating_asorepo, introduction_text_title, introduction_text, n, url, review_link]],
                          columns=columns)
    #add on
    df = pd.concat([df, df_new], axis=0)
    count+=1

    #save every 100 pieces
    if count%100 == 0:
      df.to_csv(f'base_data_{count}.csv')
      print("Running time：", time.time() - start)
      print("Rate of progress：", str((count/len(n_list))*100), "%")
      time.sleep(10)

  except requests.exceptions.HTTPError as e: #exception handling
    print("HTTPError:", e)
    continue

  except requests.exceptions.ConnectionError as e:
    print("ConnectionError:", e)
    continue

  except:
    print("Error")
    continue
```
I checked the elements of the site and retrieved them one by one in the same way as when I retrieved the activity No.
If you look at the df retrieved by `df.head()`, you will see that

||name|area|small_area|genre_type|rating_star|rating_asorepo|introduction_title|introduction_text|No.|url|review_link
|:-|:-|:-|:-|:-|:-|:-|:-|:-|:-|:-|:-
0|ガイドラインアウトドアクラブ|北海道|富良野・美瑛・トマム|['ラフティング']|5.0|7|3歳からOKのラフティング！川を全身で遊ぼう！自然を満喫しよう☆|南富良野にある水の綺麗な「シーソラプチ川」と「空知川源流部」にてラフティングを開催。全ツアー...|163|省略|/base/163/asorepo/list/
|1|洞爺ガイドセンター|北海道|洞爺・登別・苫小牧|['レイクカヌー', 'エコツアー･自然体験', 'スノーシュー・スノートレッキング']|0.0|1|美しい洞爺湖を満喫。「旬なプラン」をご提供します！|北海道・洞爺湖で、自然体験アクティビティを開催している洞爺ガイドセンター。美しい自然をご案内...|178|省略|/base/178/asorepo/list/
|2|函館どさんこファーム|北海道|函館・大沼・松前|['乗馬 その他']|0.0|3|どさんこと触れあう函館の旅！函館空港・函館駅から車で20分！|函館どさんこファームは函館空港・函館駅から20分と、アクセス抜群・好立地の乗馬クラブです。\...|191|省略|/base/191/asorepo/list/
|3|リバートリップ北海道|北海道|洞爺・登別・苫小牧|['ラフティング']|0.0|2|女子会プランも大好評！北海道最高峰の激流・鵡川で本格ラフティング！|リバートリップは、北海道最高峰と言われている鵡川で、本格的な激流ラフティングをご提供していま...|232|省略|/base/232/asorepo/list/
|4|知床サイクリングサポート|北海道|網走・北見・知床|['マウンテンバイク（MTB）・ダウンヒル']|0.0|2|ベテランガイドがご案内！知床を肌で感じるネイチャーサイクリングツアー！|知床サイクリングサポートは、世界自然遺産に選ばれた北海道・知床半島を中心に、マウンテンバイク...|260|省略|/base/260/asorepo/list/

I've got some nice data!
Finally, save this.
```python:base_data_scraping.py
df.to_csv('base_data.csv')
```

# 5. Summary

From an activity booking website, I was able to retrieve basic information about each activity.
In the next article, I will use the information I obtained to find out what facilities are listed on Asoview!