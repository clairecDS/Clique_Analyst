
# ![.right](https://static1.squarespace.com/static/5d4304e2a2ae96000114b729/t/5d430619bdc79700012170c1/1582323798085/?format=1500w)

# Analyst Assessment by Claire Chu, March 2020
### This document details the data exploration and analysis of data regarding Website Analytics for Clique Brands.

### Quick Navigation Links

- [I. INTRODUCTION](#int)

- [II. DATA EXPLORATION](#exp)

- [III. BUSINESS INTERESTS](#bus)

- [IV. CONCLUSIONS](#con)

<hr>


```python
#import libraries
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
from wordcloud import wordcloud
import re
import seaborn as sns
import matplotlib.pyplot as plt
```

### <a name="int"></a> I. INTRODUCTION: The dataset reviewed here contained __ columns/rows
---
Data for this assessment is Google Analytics hit (pageview) level data, by date and by article title. <br>
Field descriptions below:<br>
○ Date - year, month, day of hit.<br>
○ Page Title - article title that was viewed by user.<br>
○ Age - age of user that visited the site.<br>
○ Gender - gender of user that visited the site. <br>
○ Source/Medium - the referral source of the hit or the website that the user was on
before visiting whowhatwear.com. <br>
○ Pageviews - a hit of a url on our site that is being tracked by the analytics.js
tracking code. <br>
○ Unique Pageviews - represents the number of sessions during which that page
was viewed one or more times. <br>


```python
#read in dataset
data_file = pd.read_excel('~/Desktop/analyst_assessment_CC/assessment_ds.xls')
```


```python
#verify data was read in correctly
data_file.head()
data_file.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 32755 entries, 0 to 32754
    Data columns (total 7 columns):
    date                32755 non-null int64
    Page Title          32755 non-null object
    age                 32755 non-null object
    gender              32755 non-null object
    Source / Medium     32755 non-null object
    pageviews           32755 non-null int64
    Unique Pageviews    32755 non-null int64
    dtypes: int64(3), object(4)
    memory usage: 1.7+ MB



```python
#Convert date column to readable nomenclature
data_file['new_date'] = pd.to_datetime(data_file['date'].astype(str), format='%Y%m%d')
data_file['new_date'].head()
data_file.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>date</th>
      <th>Page Title</th>
      <th>age</th>
      <th>gender</th>
      <th>Source / Medium</th>
      <th>pageviews</th>
      <th>Unique Pageviews</th>
      <th>new_date</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>20190606</td>
      <td>Chrissy Teigen Wore the Pleated-Jean Trend | W...</td>
      <td>35-44</td>
      <td>female</td>
      <td>m.facebook.com / referral</td>
      <td>35541</td>
      <td>31670</td>
      <td>2019-06-06</td>
    </tr>
    <tr>
      <th>1</th>
      <td>20190606</td>
      <td>Chrissy Teigen Wore the Pleated-Jean Trend | W...</td>
      <td>25-34</td>
      <td>female</td>
      <td>m.facebook.com / referral</td>
      <td>29730</td>
      <td>26236</td>
      <td>2019-06-06</td>
    </tr>
    <tr>
      <th>2</th>
      <td>20190608</td>
      <td>Chrissy Teigen Wore the Pleated-Jean Trend | W...</td>
      <td>35-44</td>
      <td>female</td>
      <td>m.facebook.com / referral</td>
      <td>21249</td>
      <td>18923</td>
      <td>2019-06-08</td>
    </tr>
    <tr>
      <th>3</th>
      <td>20190606</td>
      <td>Chrissy Teigen Wore the Pleated-Jean Trend | W...</td>
      <td>45-54</td>
      <td>female</td>
      <td>m.facebook.com / referral</td>
      <td>18858</td>
      <td>16825</td>
      <td>2019-06-06</td>
    </tr>
    <tr>
      <th>4</th>
      <td>20190608</td>
      <td>Chrissy Teigen Wore the Pleated-Jean Trend | W...</td>
      <td>25-34</td>
      <td>female</td>
      <td>m.facebook.com / referral</td>
      <td>18157</td>
      <td>15970</td>
      <td>2019-06-08</td>
    </tr>
  </tbody>
</table>
</div>




```python
#look at "Page Title" column and see if we need to manipulate
data_file['Page Title'].tail(25)
#seems like our article title string is getting cut off, expand display
#pd.options.display.max_colwidth = 100
```




    32730    What Causes Hair Damage? The 12 Worst Things Y...
    32731    21 Affordable Things From Chanel, Gucci, and L...
    32732    The 28 Most Classic Nail Colors of All Time | ...
    32733    The Best Miami Summer Trends for Under $100 | ...
    32734    The East-West Engagement Ring Trend Is on the ...
    32735    7 of the Best Brands for Summer Sandals | Who ...
    32736    How to Become a Morning Person | The Latest Fi...
    32737    The 9 Most Daring Celebrity Bikini Trends of 2...
    32738    The Best Morning Routine for Success | The Lat...
    32739    5 Comfortable Workout Outfits That Are so Flat...
    32740    Life Lessons Podcast Episode With Molly McNear...
    32741    Shopping Totes Will Be the Biggest Bag Trend o...
    32742    Shopping Totes Will Be the Biggest Bag Trend o...
    32743    The Best Morning Routine for Success | The Lat...
    32744    Irina Shayk Wore an Under-$100 Ref Dress With ...
    32745    Sophie Turner Wore an Elegant White Dress Befo...
    32746                       Summer Dresses | Who What Wear
    32747    Emma Greenwell's Rules for Buying Dresses and ...
    32748    Jennifer Aniston Wore the Wedge-Heel Shoe Tren...
    32749      Stylish Looks for Summer Nights | Who What Wear
    32750         Summer Workout Outfit Trends | Who What Wear
    32751    The Only Strapless Bra I WearäóîLively | Who W...
    32752    Zoí‚ Kravitz Wore Bike Shorts to Her Paris Wed...
    32753    Zoí‚ Kravitz Wore Bike Shorts to Her Paris Wed...
    32754    Louis Vuitton Bag Prices Around the World | Wh...
    Name: Page Title, dtype: object




```python
#seems like our page title includes some different websites including "who what wear", "who what wear UK", etc...
#let's split this column so we can look at analysis by website, if necessary
new = data_file['Page Title'].str.split("|", n = 1, expand = True) 
#verify column was split correctly
new.tail(25)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
      <th>1</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>32730</th>
      <td>What Causes Hair Damage? The 12 Worst Things Y...</td>
      <td>Who What Wear UK</td>
    </tr>
    <tr>
      <th>32731</th>
      <td>21 Affordable Things From Chanel, Gucci, and L...</td>
      <td>Who What Wear</td>
    </tr>
    <tr>
      <th>32732</th>
      <td>The 28 Most Classic Nail Colors of All Time</td>
      <td>Who What Wear</td>
    </tr>
    <tr>
      <th>32733</th>
      <td>The Best Miami Summer Trends for Under $100</td>
      <td>Who What Wear</td>
    </tr>
    <tr>
      <th>32734</th>
      <td>The East-West Engagement Ring Trend Is on the ...</td>
      <td>Who What Wear</td>
    </tr>
    <tr>
      <th>32735</th>
      <td>7 of the Best Brands for Summer Sandals</td>
      <td>Who What Wear</td>
    </tr>
    <tr>
      <th>32736</th>
      <td>How to Become a Morning Person</td>
      <td>The Latest Fitness, Health, and Wellness Tips</td>
    </tr>
    <tr>
      <th>32737</th>
      <td>The 9 Most Daring Celebrity Bikini Trends of 2...</td>
      <td>Who What Wear</td>
    </tr>
    <tr>
      <th>32738</th>
      <td>The Best Morning Routine for Success</td>
      <td>The Latest Fitness, Health, and Wellness Tips</td>
    </tr>
    <tr>
      <th>32739</th>
      <td>5 Comfortable Workout Outfits That Are so Flat...</td>
      <td>Who What Wear</td>
    </tr>
    <tr>
      <th>32740</th>
      <td>Life Lessons Podcast Episode With Molly McNear...</td>
      <td>Who What Wear</td>
    </tr>
    <tr>
      <th>32741</th>
      <td>Shopping Totes Will Be the Biggest Bag Trend o...</td>
      <td>Who What Wear</td>
    </tr>
    <tr>
      <th>32742</th>
      <td>Shopping Totes Will Be the Biggest Bag Trend o...</td>
      <td>Who What Wear</td>
    </tr>
    <tr>
      <th>32743</th>
      <td>The Best Morning Routine for Success</td>
      <td>The Latest Fitness, Health, and Wellness Tips</td>
    </tr>
    <tr>
      <th>32744</th>
      <td>Irina Shayk Wore an Under-$100 Ref Dress With ...</td>
      <td>Who What Wear</td>
    </tr>
    <tr>
      <th>32745</th>
      <td>Sophie Turner Wore an Elegant White Dress Befo...</td>
      <td>Who What Wear</td>
    </tr>
    <tr>
      <th>32746</th>
      <td>Summer Dresses</td>
      <td>Who What Wear</td>
    </tr>
    <tr>
      <th>32747</th>
      <td>Emma Greenwell's Rules for Buying Dresses and ...</td>
      <td>Who What Wear</td>
    </tr>
    <tr>
      <th>32748</th>
      <td>Jennifer Aniston Wore the Wedge-Heel Shoe Trend</td>
      <td>Who What Wear</td>
    </tr>
    <tr>
      <th>32749</th>
      <td>Stylish Looks for Summer Nights</td>
      <td>Who What Wear</td>
    </tr>
    <tr>
      <th>32750</th>
      <td>Summer Workout Outfit Trends</td>
      <td>Who What Wear</td>
    </tr>
    <tr>
      <th>32751</th>
      <td>The Only Strapless Bra I WearäóîLively</td>
      <td>Who What Wear</td>
    </tr>
    <tr>
      <th>32752</th>
      <td>Zoí‚ Kravitz Wore Bike Shorts to Her Paris Wed...</td>
      <td>Who What Wear</td>
    </tr>
    <tr>
      <th>32753</th>
      <td>Zoí‚ Kravitz Wore Bike Shorts to Her Paris Wed...</td>
      <td>Who What Wear</td>
    </tr>
    <tr>
      <th>32754</th>
      <td>Louis Vuitton Bag Prices Around the World</td>
      <td>Who What Wear</td>
    </tr>
  </tbody>
</table>
</div>




```python
#move new split columns into "data_file"
data_file['Page_Title'] = new[0]
data_file['Site'] = new [1]
data_file.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>date</th>
      <th>Page Title</th>
      <th>age</th>
      <th>gender</th>
      <th>Source / Medium</th>
      <th>pageviews</th>
      <th>Unique Pageviews</th>
      <th>new_date</th>
      <th>Page_Title</th>
      <th>Site</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>20190606</td>
      <td>Chrissy Teigen Wore the Pleated-Jean Trend | W...</td>
      <td>35-44</td>
      <td>female</td>
      <td>m.facebook.com / referral</td>
      <td>35541</td>
      <td>31670</td>
      <td>2019-06-06</td>
      <td>Chrissy Teigen Wore the Pleated-Jean Trend</td>
      <td>Who What Wear</td>
    </tr>
    <tr>
      <th>1</th>
      <td>20190606</td>
      <td>Chrissy Teigen Wore the Pleated-Jean Trend | W...</td>
      <td>25-34</td>
      <td>female</td>
      <td>m.facebook.com / referral</td>
      <td>29730</td>
      <td>26236</td>
      <td>2019-06-06</td>
      <td>Chrissy Teigen Wore the Pleated-Jean Trend</td>
      <td>Who What Wear</td>
    </tr>
    <tr>
      <th>2</th>
      <td>20190608</td>
      <td>Chrissy Teigen Wore the Pleated-Jean Trend | W...</td>
      <td>35-44</td>
      <td>female</td>
      <td>m.facebook.com / referral</td>
      <td>21249</td>
      <td>18923</td>
      <td>2019-06-08</td>
      <td>Chrissy Teigen Wore the Pleated-Jean Trend</td>
      <td>Who What Wear</td>
    </tr>
    <tr>
      <th>3</th>
      <td>20190606</td>
      <td>Chrissy Teigen Wore the Pleated-Jean Trend | W...</td>
      <td>45-54</td>
      <td>female</td>
      <td>m.facebook.com / referral</td>
      <td>18858</td>
      <td>16825</td>
      <td>2019-06-06</td>
      <td>Chrissy Teigen Wore the Pleated-Jean Trend</td>
      <td>Who What Wear</td>
    </tr>
    <tr>
      <th>4</th>
      <td>20190608</td>
      <td>Chrissy Teigen Wore the Pleated-Jean Trend | W...</td>
      <td>25-34</td>
      <td>female</td>
      <td>m.facebook.com / referral</td>
      <td>18157</td>
      <td>15970</td>
      <td>2019-06-08</td>
      <td>Chrissy Teigen Wore the Pleated-Jean Trend</td>
      <td>Who What Wear</td>
    </tr>
  </tbody>
</table>
</div>




```python
#rearrange data_file to include new columns
data_file = data_file[['date','new_date', 'Page_Title', 'Site', 'age', 'gender', 'Source / Medium' , 'pageviews', 'Unique Pageviews']]
data_file.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>date</th>
      <th>new_date</th>
      <th>Page_Title</th>
      <th>Site</th>
      <th>age</th>
      <th>gender</th>
      <th>Source / Medium</th>
      <th>pageviews</th>
      <th>Unique Pageviews</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>20190606</td>
      <td>2019-06-06</td>
      <td>Chrissy Teigen Wore the Pleated-Jean Trend</td>
      <td>Who What Wear</td>
      <td>35-44</td>
      <td>female</td>
      <td>m.facebook.com / referral</td>
      <td>35541</td>
      <td>31670</td>
    </tr>
    <tr>
      <th>1</th>
      <td>20190606</td>
      <td>2019-06-06</td>
      <td>Chrissy Teigen Wore the Pleated-Jean Trend</td>
      <td>Who What Wear</td>
      <td>25-34</td>
      <td>female</td>
      <td>m.facebook.com / referral</td>
      <td>29730</td>
      <td>26236</td>
    </tr>
    <tr>
      <th>2</th>
      <td>20190608</td>
      <td>2019-06-08</td>
      <td>Chrissy Teigen Wore the Pleated-Jean Trend</td>
      <td>Who What Wear</td>
      <td>35-44</td>
      <td>female</td>
      <td>m.facebook.com / referral</td>
      <td>21249</td>
      <td>18923</td>
    </tr>
    <tr>
      <th>3</th>
      <td>20190606</td>
      <td>2019-06-06</td>
      <td>Chrissy Teigen Wore the Pleated-Jean Trend</td>
      <td>Who What Wear</td>
      <td>45-54</td>
      <td>female</td>
      <td>m.facebook.com / referral</td>
      <td>18858</td>
      <td>16825</td>
    </tr>
    <tr>
      <th>4</th>
      <td>20190608</td>
      <td>2019-06-08</td>
      <td>Chrissy Teigen Wore the Pleated-Jean Trend</td>
      <td>Who What Wear</td>
      <td>25-34</td>
      <td>female</td>
      <td>m.facebook.com / referral</td>
      <td>18157</td>
      <td>15970</td>
    </tr>
  </tbody>
</table>
</div>




```python
#look at our dataset
data_file.describe(include = 'all')
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>date</th>
      <th>new_date</th>
      <th>Page_Title</th>
      <th>Site</th>
      <th>age</th>
      <th>gender</th>
      <th>Source / Medium</th>
      <th>pageviews</th>
      <th>Unique Pageviews</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>3.275500e+04</td>
      <td>32755</td>
      <td>32755</td>
      <td>32755</td>
      <td>32755</td>
      <td>32755</td>
      <td>32755</td>
      <td>32755.000000</td>
      <td>32755.000000</td>
    </tr>
    <tr>
      <th>unique</th>
      <td>NaN</td>
      <td>30</td>
      <td>1091</td>
      <td>5</td>
      <td>6</td>
      <td>2</td>
      <td>75</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>top</th>
      <td>NaN</td>
      <td>2019-06-24 00:00:00</td>
      <td>Celebrity Style and Fashion Trend Coverage</td>
      <td>Who What Wear</td>
      <td>25-34</td>
      <td>female</td>
      <td>google / organic</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>freq</th>
      <td>NaN</td>
      <td>1391</td>
      <td>1175</td>
      <td>30933</td>
      <td>10160</td>
      <td>29310</td>
      <td>6560</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>first</th>
      <td>NaN</td>
      <td>2019-06-01 00:00:00</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>last</th>
      <td>NaN</td>
      <td>2019-06-30 00:00:00</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>2.019062e+07</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>385.552526</td>
      <td>318.538238</td>
    </tr>
    <tr>
      <th>std</th>
      <td>8.883106e+00</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>871.837327</td>
      <td>738.081530</td>
    </tr>
    <tr>
      <th>min</th>
      <td>2.019060e+07</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>9.000000</td>
      <td>7.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>2.019061e+07</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>61.000000</td>
      <td>50.000000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>2.019062e+07</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>173.000000</td>
      <td>140.000000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>2.019062e+07</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>369.000000</td>
      <td>303.000000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>2.019063e+07</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>35541.000000</td>
      <td>31670.000000</td>
    </tr>
  </tbody>
</table>
</div>



##### Overall Data Summary
We can see that all counts = 32755, implying that every cell has an entry (no empty cells). 

__'new_date'__
Our 'new_date' column has 30 unique days, this makes sense since there are 30 days in the month of June, with June 24 having the most entries. Does this imply one article was popular on this day or were there multiple articles posted this day?

__ 'Page Title' __
This column has 1091 unique entires, meaning, that there are multiple entries with the same article title. We assume this is because they have different sources, as it is not likely that we would repost the same article multiple times.
Our top result is an article titled "Celebrity Style and Fashion Trend Coverage" so we should look into this entry and the different sources

__'Site'__
We can see that there are 5 different sites that these pages are being posted at, with the most frequent entry being "Who What Wear" occuring 30933 times. That's ~94% of the entires! We will keep this in mind moving forward.

__ 'age' __
We can see from the chart that this column consists of 6 age group categories with the most popular being the group aged "25-34" with 10160 entries, consisting of ~31% of the dataset. We will need to take a deeper look and establish the 6 specific categories.

__'gender' __
There are two unique categories for gender, this makes sense. We see the top category is 'female' with 29310 (~90%) entries distingued as female. 32755 - 29310 = 3445, or about 10% categorized as 'male'.

__'Source / Medium'__
There are 75 unique sources with 'google/organic' being the most popular with 6560 entries (~20%) of the dataset.

__'pageviews'__
Our pageviews, defined as "a hit of a url on our site that is being tracked by the analytics.js tracking code", have a total of 32755 entires. This is good because it means all our urls have tracking code. Our hits range from 9 - 35541, more than 50% being under 173, so it seems like we definitely have some underperforming posts, but we will have to get more information to determine what our threshold should be. Let's look at some graphs to see how the data is skewed later.

__'Unique Pageviews' __
This column "represents the number of sessions during which that page was viewed one or more times". It should be noted that while 'pageviews' and 'unique pageviews' are similar in name, they represent two vastly different metrics. 'Pageviews' represents hits for each URL while 'unique pageviews' represents sessions. We know that one user has the ability to generate multiple pageviews (reloads) and sessions (default duration in Google Analytics is 30 minutes). We will need to keep this in mind as we proceed with our analysis.


```python
#Let's move forward with our data_file
#data_file.head(10)
data_file.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 32755 entries, 0 to 32754
    Data columns (total 9 columns):
    date                32755 non-null int64
    new_date            32755 non-null datetime64[ns]
    Page_Title          32755 non-null object
    Site                32755 non-null object
    age                 32755 non-null object
    gender              32755 non-null object
    Source / Medium     32755 non-null object
    pageviews           32755 non-null int64
    Unique Pageviews    32755 non-null int64
    dtypes: datetime64[ns](1), int64(3), object(5)
    memory usage: 2.2+ MB


### <a name="exp"></a> II. LOOK INTO THE DATA
---

- Date Investigation
- Article Title Exploration
- Article Site Exploration
- Age Investigation
- Gender Investigation
- Source/Medium Exploration

### __Date Investigation__


```python
date = pd.crosstab(index=data_file['date'],columns="count") 
#print(date)
print (date.head())
#date.info()
date = date.reset_index()
```

    col_0     count
    date           
    20190601   1079
    20190602   1228
    20190603   1235
    20190604   1118
    20190605   1218



```python
date_sort = date.sort_values(by=['count'], ascending=False)
date_sort.head()
#date_sort.tail()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>col_0</th>
      <th>date</th>
      <th>count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>23</th>
      <td>20190624</td>
      <td>1391</td>
    </tr>
    <tr>
      <th>5</th>
      <td>20190606</td>
      <td>1299</td>
    </tr>
    <tr>
      <th>27</th>
      <td>20190628</td>
      <td>1291</td>
    </tr>
    <tr>
      <th>9</th>
      <td>20190610</td>
      <td>1262</td>
    </tr>
    <tr>
      <th>2</th>
      <td>20190603</td>
      <td>1235</td>
    </tr>
  </tbody>
</table>
</div>




```python
dates = date['date']
posts = date['count']
plt.figure(figsize=(25,4))
plt.subplot(133)
plt.plot(dates, posts)
plt.title('June Posts by Day')
plt.xlabel('Day in June')
plt.ylabel('Number of Posts Made')
plt.show()

#graph confirmed with min and max results
```


![png](output_17_0.png)


Our graph shows a peak number of posts around June 25 and a pit around June 15. 

### __Article Title Exploration __ 


```python
titles = pd.crosstab(index=data_file['Page_Title'],columns="count") 
titles = titles.reset_index()
titles.head()
#titles.tail()
#By default the data is sorted alphabetically by Page Title
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>col_0</th>
      <th>Page_Title</th>
      <th>count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>'80s Fashion Trends: 35 Iconic Looks From the ...</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>10 "Boring" Basics Every Woman Needs From Amazon</td>
      <td>26</td>
    </tr>
    <tr>
      <th>2</th>
      <td>10 "Boring" Fashion Basics French Girls Always...</td>
      <td>6</td>
    </tr>
    <tr>
      <th>3</th>
      <td>10 Affordable Jewelry Brands We Always Shop</td>
      <td>31</td>
    </tr>
    <tr>
      <th>4</th>
      <td>10 Affordable Stores Like Lulu's</td>
      <td>30</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Let's instead sort by count. This means the number of times the title occured in our dataset. 
#We can see that there were 1175 instances of "Celebrity Style and Fashion Trend Coverage"
title_sort = titles.sort_values(by=['count'], ascending=False)
title_sort.head()
#title_sort.tail()
#title_sort.info()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>col_0</th>
      <th>Page_Title</th>
      <th>count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>466</th>
      <td>Celebrity Style and Fashion Trend Coverage</td>
      <td>1175</td>
    </tr>
    <tr>
      <th>523</th>
      <td>How to Become a Morning Person</td>
      <td>443</td>
    </tr>
    <tr>
      <th>472</th>
      <td>Chrissy Teigen Wore the Pleated-Jean Trend</td>
      <td>373</td>
    </tr>
    <tr>
      <th>997</th>
      <td>These $69 Treggings "Flatter Every Body Type"</td>
      <td>284</td>
    </tr>
    <tr>
      <th>584</th>
      <td>Jennifer Lopez Wore 2019's Biggest Grandma Trend</td>
      <td>280</td>
    </tr>
  </tbody>
</table>
</div>




```python
#let's try plotting this to get a sense of what our data looks like
title_plt = pd.DataFrame(title_sort)
ax = title_plt.plot.bar(x='Page_Title', y='count', rot=0)
ax1 = plt.axes()
x_axis = ax1.axes.get_xaxis()
x_axis.set_visible(False)
plt.show(ax)

#we can see that apart from the 1175 instances of "Celebrity Style and Fashion Trend Coverage",
#The next highest article only has 443 instances. 
#let's try treating this as an outlier to get a better sense of the data
```


![png](output_22_0.png)



```python
#remove the "Celebrity Style and Fashion Trend Coverage" entry
title_sort3 = title_sort.drop([466])
title_sort3.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>col_0</th>
      <th>Page_Title</th>
      <th>count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>523</th>
      <td>How to Become a Morning Person</td>
      <td>443</td>
    </tr>
    <tr>
      <th>472</th>
      <td>Chrissy Teigen Wore the Pleated-Jean Trend</td>
      <td>373</td>
    </tr>
    <tr>
      <th>997</th>
      <td>These $69 Treggings "Flatter Every Body Type"</td>
      <td>284</td>
    </tr>
    <tr>
      <th>584</th>
      <td>Jennifer Lopez Wore 2019's Biggest Grandma Trend</td>
      <td>280</td>
    </tr>
    <tr>
      <th>424</th>
      <td>Affordable Summer Fashion Trends From a Chic H...</td>
      <td>253</td>
    </tr>
  </tbody>
</table>
</div>




```python
#now plot the same variable, now with entry removed
title_plt2 = pd.DataFrame(title_sort3)
ax2 = title_plt2.plot.bar(x='Page_Title', y='count', rot=0)
ax1 = plt.axes()
x_axis = ax1.axes.get_xaxis()
x_axis.set_visible(False)
plt.show(ax2)
```


![png](output_24_0.png)



```python
#seems like removing the one outlier didnt really change our data
title_sort.describe()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>col_0</th>
      <th>count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>1091.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>30.022915</td>
    </tr>
    <tr>
      <th>std</th>
      <td>53.160243</td>
    </tr>
    <tr>
      <th>min</th>
      <td>1.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>4.500000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>14.000000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>40.000000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>1175.000000</td>
    </tr>
  </tbody>
</table>
</div>




```python
#checking our updated data
#it seems like 75% of our dataset has 40 or less instances. 
#wow our data seems heavily skewed. we'll go back to using the entire dataset.
title_sort3.describe()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>col_0</th>
      <th>count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>1090.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>28.972477</td>
    </tr>
    <tr>
      <th>std</th>
      <td>40.294795</td>
    </tr>
    <tr>
      <th>min</th>
      <td>1.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>4.250000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>14.000000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>40.000000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>443.000000</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Let's get some information about the titles being used
#we will use the "page titles" from the original dataset, that way our wordcloud can accurately show word frequencies
#Generate a word cloud
text = " ".join(review for review in data_file.Page_Title)
wordcloud = wordcloud.WordCloud(background_color="white").generate(text)
plt.imshow(wordcloud, interpolation='bilinear')
plt.axis("off")
plt.show()
```


![png](output_27_0.png)


We can see from the word cloud that the popular terms are 

    - celebrity style
    - fashion trend
    - trend coverage
    - style fashion
    - summer trend
    - shoe trend

### __Article Site Exploration__


```python
site = pd.crosstab(index=data_file['Site'],columns="count") 
site = site.reset_index()
print (site)
#It seems like from our results, something is happening with some entires having "Who What Wear | Who What Wear"
#We will investigate these entires and see if we can merge them with our "Who What Wear" entry.
```

    col_0                                            Site  count
    0       The Latest Fitness, Health, and Wellness Tips    807
    1                                           TheThirty    290
    2                                       Who What Wear  30933
    3                                    Who What Wear UK    615
    4                       Who What Wear | Who What Wear    110



```python
#look into entries where we are getting the "Who What Wear | Who What Wear" anomaly

#Search for entries where we have the "Who What Wear | Who What Wear" anomaly
anom = data_file['Site'].str.contains('Who What Wear [|]')
anom.describe()

#Generated a True/False List
#pair down to listing of only true entries
#pull random sample of 10 entires to investigate

anomT = anom[anom]
anomT.head()

##this has been commented out so as not to pull an additional sample at each run
anomTS = anomT.sample(10, random_state=9999)
anomTS

#Checking entries

data_file[27174:27175]
#Outfit Ideas and Everday Fashion Tips
#data_file[30469:30470]
#Shopping Guides to the Latest Fashion Trends
#data_file[30667:30668]
#Outfit Ideas and Everday Fashion Tips
#data_file[26911:26912]
#Outfit Ideas and Everday Fashion Tips
#data_file[28600:28601]
#Shopping Guides to the Latest Fashion Trends
#data_file[30476:30477]
#Shopping Guides to the Latest Fashion Trends
#data_file[27269:27270]
#Shopping Guides to the Latest Fashion Trends
#data_file[26956:26957]
#Outfit Ideas and Everday Fashion Tips
#data_file[26543:26544]
#Outfit Ideas and Everyday Fashion Tips

#It seems like one or two articles were maybe listed incorrectly
#so we will merge the "Who What Wear | Who What Wear" and "Who What Wear" categories without affecting our analysis
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>date</th>
      <th>new_date</th>
      <th>Page_Title</th>
      <th>Site</th>
      <th>age</th>
      <th>gender</th>
      <th>Source / Medium</th>
      <th>pageviews</th>
      <th>Unique Pageviews</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>27174</th>
      <td>20190612</td>
      <td>2019-06-12</td>
      <td>Outfit Ideas and Everyday Fashion Tips</td>
      <td>Who What Wear | Who What Wear</td>
      <td>25-34</td>
      <td>female</td>
      <td>google / organic</td>
      <td>39</td>
      <td>30</td>
    </tr>
  </tbody>
</table>
</div>




```python
#let's try importing our data with two splits, now that we know our data's inconsistencies
data_file = pd.read_excel('~/Desktop/analyst_assessment_CC/assessment_ds.xls')
data_file['new_date'] = pd.to_datetime(data_file['date'].astype(str), format='%Y%m%d')
new = data_file['Page Title'].str.split("|", n = 2, expand = True) 
new.head()
new.describe()

##I realize this is a step backward, and professionally, I would just restart my notebook here, 
##however for the sake of demonstrating my process, I have left the previous cells 
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
      <th>1</th>
      <th>2</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>32755</td>
      <td>32755</td>
      <td>110</td>
    </tr>
    <tr>
      <th>unique</th>
      <td>1091</td>
      <td>5</td>
      <td>1</td>
    </tr>
    <tr>
      <th>top</th>
      <td>Celebrity Style and Fashion Trend Coverage</td>
      <td>Who What Wear</td>
      <td>Who What Wear</td>
    </tr>
    <tr>
      <th>freq</th>
      <td>1175</td>
      <td>30933</td>
      <td>110</td>
    </tr>
  </tbody>
</table>
</div>




```python
#after breaking them out into additional columns at the "|", deleting extra column to try to remove 5th category
#new[2] = new[2].str.replace('Who What Wear', 'NaN')
#new[2] = new[2].astype('category')
#new[2].head()

#ok after spending a lot of time trying to debug this issue, i realized there was an extra space after "Who What Wear" 
#in our anomaly cases, this code rectifies that issue and reduces our categories to 4 
new[1] = new[1].str.replace('Who What Wear ', 'Who What Wear')
new[1].describe()
```




    count              32755
    unique                 4
    top        Who What Wear
    freq               31043
    Name: 1, dtype: object




```python
#remerge columns together
data_file['Page_Title'] = new[0]
data_file['Site'] = new [1]
```


```python
data_file = data_file[['date','new_date', 'Page_Title', 'Site', 'age', 'gender', 
                       'Source / Medium' , 'pageviews', 'Unique Pageviews']]
data_file.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>date</th>
      <th>new_date</th>
      <th>Page_Title</th>
      <th>Site</th>
      <th>age</th>
      <th>gender</th>
      <th>Source / Medium</th>
      <th>pageviews</th>
      <th>Unique Pageviews</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>20190606</td>
      <td>2019-06-06</td>
      <td>Chrissy Teigen Wore the Pleated-Jean Trend</td>
      <td>Who What Wear</td>
      <td>35-44</td>
      <td>female</td>
      <td>m.facebook.com / referral</td>
      <td>35541</td>
      <td>31670</td>
    </tr>
    <tr>
      <th>1</th>
      <td>20190606</td>
      <td>2019-06-06</td>
      <td>Chrissy Teigen Wore the Pleated-Jean Trend</td>
      <td>Who What Wear</td>
      <td>25-34</td>
      <td>female</td>
      <td>m.facebook.com / referral</td>
      <td>29730</td>
      <td>26236</td>
    </tr>
    <tr>
      <th>2</th>
      <td>20190608</td>
      <td>2019-06-08</td>
      <td>Chrissy Teigen Wore the Pleated-Jean Trend</td>
      <td>Who What Wear</td>
      <td>35-44</td>
      <td>female</td>
      <td>m.facebook.com / referral</td>
      <td>21249</td>
      <td>18923</td>
    </tr>
    <tr>
      <th>3</th>
      <td>20190606</td>
      <td>2019-06-06</td>
      <td>Chrissy Teigen Wore the Pleated-Jean Trend</td>
      <td>Who What Wear</td>
      <td>45-54</td>
      <td>female</td>
      <td>m.facebook.com / referral</td>
      <td>18858</td>
      <td>16825</td>
    </tr>
    <tr>
      <th>4</th>
      <td>20190608</td>
      <td>2019-06-08</td>
      <td>Chrissy Teigen Wore the Pleated-Jean Trend</td>
      <td>Who What Wear</td>
      <td>25-34</td>
      <td>female</td>
      <td>m.facebook.com / referral</td>
      <td>18157</td>
      <td>15970</td>
    </tr>
  </tbody>
</table>
</div>




```python
#convert to category type to look at our different categories
data_file['Site'] = data_file['Site'].astype('category')
data_file['Site'].head()
#try to look at a specific cell where we know the anomaly exists
data_file[27173:27176]
#we know "Outfit Ideas and Everyday Fashion Tips" had the | issue, 
#but string wise appears the same as the other cells without
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>date</th>
      <th>new_date</th>
      <th>Page_Title</th>
      <th>Site</th>
      <th>age</th>
      <th>gender</th>
      <th>Source / Medium</th>
      <th>pageviews</th>
      <th>Unique Pageviews</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>27173</th>
      <td>20190612</td>
      <td>2019-06-12</td>
      <td>Courteney Cox and Her Daughter, Coco, Look Lik...</td>
      <td>Who What Wear</td>
      <td>55-64</td>
      <td>female</td>
      <td>news.google.com / referral</td>
      <td>39</td>
      <td>36</td>
    </tr>
    <tr>
      <th>27174</th>
      <td>20190612</td>
      <td>2019-06-12</td>
      <td>Outfit Ideas and Everyday Fashion Tips</td>
      <td>Who What Wear</td>
      <td>25-34</td>
      <td>female</td>
      <td>google / organic</td>
      <td>39</td>
      <td>30</td>
    </tr>
    <tr>
      <th>27175</th>
      <td>20190613</td>
      <td>2019-06-13</td>
      <td>The 16 Best Fashion and Beauty Items at Target...</td>
      <td>Who What Wear</td>
      <td>45-54</td>
      <td>female</td>
      <td>news.google.com / referral</td>
      <td>39</td>
      <td>37</td>
    </tr>
  </tbody>
</table>
</div>




```python
#redo our counts based on our 4 categories
site = pd.crosstab(index=data_file['Site'],columns="count") 
site = site.reset_index()
```


```python
#Here is an updated graph to demonstrate the different site categories
sns.set(style="ticks", color_codes=True)
sns.barplot(x='Site', y='count', data=site);
plt.title('Where Were June Articles Posted?')
plt.ylabel('Posts from Each Site')
plt.show()
#Majority of our data comes from Who What Wear posts, we already knew that, but at least our data is more accurate
```


![png](output_38_0.png)


### __Age Exploration__


```python
data_file['age'] = data_file['age'].astype('category')
age = pd.crosstab(index=data_file['age'],columns="count") 
age = age.reset_index()
```


```python
sns.set(style="ticks", color_codes=True)
sns.barplot(x='age', y='count', data=age);
plt.title('Age Metrics')
plt.xlabel('age categories')
plt.ylabel('visitors in June')
plt.show()
#we can see that the highest age group is 25-34, with the smallest being the 18-24 group
```


![png](output_41_0.png)


### __Audience Gender__


```python
data_file['gender'] = data_file['gender'].astype('category')
gender = pd.crosstab(index=data_file['gender'],columns="count") 
gender = gender.reset_index()
sns.set(style="ticks", color_codes=True)
sns.barplot(x='gender', y='count', data=gender);
plt.title('Gender Metrics')
plt.ylabel('visitors in June')
plt.show()
#female visitors far outweight their male counterparts
```


![png](output_43_0.png)


### __Source Exploration__


```python
source = pd.crosstab(index=data_file['Source / Medium'],columns="count") 
source = source.reset_index()
source_sort = source.sort_values(by=['count'], ascending=False)
#source_sort.head()
#we can see our top 5 sources are google, facebook, flipboard, and the direct newsletter
```


```python
data_file['Source / Medium'] = data_file['Source / Medium'].astype('category')
#data_file['Source / Medium'].cat.categories

#seems like a lot of our entires are designated by a source, a "/", and a medium
#let's split these
```


```python
#splitting the "source" and "medium" columns
new2 = data_file['Source / Medium'].str.split(" / ", n = 1, expand = True) 
new2.head()
data_file['Source'] = new2[0]
data_file['Medium'] = new2 [1]
data_file = data_file[['date','new_date', 'Page_Title', 'Site', 'age', 'gender', 
                       'Source', 'Medium' , 'pageviews', 'Unique Pageviews']]
data_file.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>date</th>
      <th>new_date</th>
      <th>Page_Title</th>
      <th>Site</th>
      <th>age</th>
      <th>gender</th>
      <th>Source</th>
      <th>Medium</th>
      <th>pageviews</th>
      <th>Unique Pageviews</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>20190606</td>
      <td>2019-06-06</td>
      <td>Chrissy Teigen Wore the Pleated-Jean Trend</td>
      <td>Who What Wear</td>
      <td>35-44</td>
      <td>female</td>
      <td>m.facebook.com</td>
      <td>referral</td>
      <td>35541</td>
      <td>31670</td>
    </tr>
    <tr>
      <th>1</th>
      <td>20190606</td>
      <td>2019-06-06</td>
      <td>Chrissy Teigen Wore the Pleated-Jean Trend</td>
      <td>Who What Wear</td>
      <td>25-34</td>
      <td>female</td>
      <td>m.facebook.com</td>
      <td>referral</td>
      <td>29730</td>
      <td>26236</td>
    </tr>
    <tr>
      <th>2</th>
      <td>20190608</td>
      <td>2019-06-08</td>
      <td>Chrissy Teigen Wore the Pleated-Jean Trend</td>
      <td>Who What Wear</td>
      <td>35-44</td>
      <td>female</td>
      <td>m.facebook.com</td>
      <td>referral</td>
      <td>21249</td>
      <td>18923</td>
    </tr>
    <tr>
      <th>3</th>
      <td>20190606</td>
      <td>2019-06-06</td>
      <td>Chrissy Teigen Wore the Pleated-Jean Trend</td>
      <td>Who What Wear</td>
      <td>45-54</td>
      <td>female</td>
      <td>m.facebook.com</td>
      <td>referral</td>
      <td>18858</td>
      <td>16825</td>
    </tr>
    <tr>
      <th>4</th>
      <td>20190608</td>
      <td>2019-06-08</td>
      <td>Chrissy Teigen Wore the Pleated-Jean Trend</td>
      <td>Who What Wear</td>
      <td>25-34</td>
      <td>female</td>
      <td>m.facebook.com</td>
      <td>referral</td>
      <td>18157</td>
      <td>15970</td>
    </tr>
  </tbody>
</table>
</div>




```python
#investigate the source and medium separately
data_file['Source'].describe()
data_file['Source'] = data_file['Source'].astype('category')
source_list = data_file.Source.cat.categories.tolist()
data_file['Medium'].describe()
data_file['Medium'] = data_file['Medium'].astype('category')
medium_list = data_file.Medium.cat.categories.tolist()
source_list
medium_list
medium = pd.crosstab(index=data_file['Medium'],columns="count") 
medium = medium.reset_index()
medium_sort = medium.sort_values(by=['count'], ascending=False)
medium_sort.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>col_0</th>
      <th>Medium</th>
      <th>count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>13</th>
      <td>referral</td>
      <td>16378</td>
    </tr>
    <tr>
      <th>11</th>
      <td>organic</td>
      <td>6560</td>
    </tr>
    <tr>
      <th>14</th>
      <td>social</td>
      <td>3632</td>
    </tr>
    <tr>
      <th>6</th>
      <td>email</td>
      <td>2441</td>
    </tr>
    <tr>
      <th>0</th>
      <td>(none)</td>
      <td>1849</td>
    </tr>
  </tbody>
</table>
</div>




```python
#now that our database appears ready for analysis, we can start looking at subsets of the data
google = data_file[data_file.Source == 'google']
google.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>date</th>
      <th>new_date</th>
      <th>Page_Title</th>
      <th>Site</th>
      <th>age</th>
      <th>gender</th>
      <th>Source</th>
      <th>Medium</th>
      <th>pageviews</th>
      <th>Unique Pageviews</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>141</th>
      <td>20190604</td>
      <td>2019-06-04</td>
      <td>5 Summer Dress Trends That Will Be Everywhere ...</td>
      <td>Who What Wear</td>
      <td>25-34</td>
      <td>female</td>
      <td>google</td>
      <td>organic</td>
      <td>5814</td>
      <td>4881</td>
    </tr>
    <tr>
      <th>152</th>
      <td>20190617</td>
      <td>2019-06-17</td>
      <td>The 16 Top Fashion Brands, According to Editors</td>
      <td>Who What Wear</td>
      <td>25-34</td>
      <td>female</td>
      <td>google</td>
      <td>organic</td>
      <td>5615</td>
      <td>4597</td>
    </tr>
    <tr>
      <th>200</th>
      <td>20190628</td>
      <td>2019-06-28</td>
      <td>The 23 Best Shopping Picks for June 2019</td>
      <td>Who What Wear</td>
      <td>25-34</td>
      <td>female</td>
      <td>google</td>
      <td>organic</td>
      <td>4995</td>
      <td>3924</td>
    </tr>
    <tr>
      <th>237</th>
      <td>20190624</td>
      <td>2019-06-24</td>
      <td>7 Outdated Sandal Trends We're Splitting Up With</td>
      <td>Who What Wear</td>
      <td>55-64</td>
      <td>female</td>
      <td>google</td>
      <td>organic</td>
      <td>4660</td>
      <td>4160</td>
    </tr>
    <tr>
      <th>240</th>
      <td>20190621</td>
      <td>2019-06-21</td>
      <td>9 Summer Outfits Our Editors Wore to the Offic...</td>
      <td>Who What Wear</td>
      <td>25-34</td>
      <td>female</td>
      <td>google</td>
      <td>organic</td>
      <td>4636</td>
      <td>3779</td>
    </tr>
  </tbody>
</table>
</div>




```python
referral = data_file[data_file.Medium == 'referral']
referral.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>date</th>
      <th>new_date</th>
      <th>Page_Title</th>
      <th>Site</th>
      <th>age</th>
      <th>gender</th>
      <th>Source</th>
      <th>Medium</th>
      <th>pageviews</th>
      <th>Unique Pageviews</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>20190606</td>
      <td>2019-06-06</td>
      <td>Chrissy Teigen Wore the Pleated-Jean Trend</td>
      <td>Who What Wear</td>
      <td>35-44</td>
      <td>female</td>
      <td>m.facebook.com</td>
      <td>referral</td>
      <td>35541</td>
      <td>31670</td>
    </tr>
    <tr>
      <th>1</th>
      <td>20190606</td>
      <td>2019-06-06</td>
      <td>Chrissy Teigen Wore the Pleated-Jean Trend</td>
      <td>Who What Wear</td>
      <td>25-34</td>
      <td>female</td>
      <td>m.facebook.com</td>
      <td>referral</td>
      <td>29730</td>
      <td>26236</td>
    </tr>
    <tr>
      <th>2</th>
      <td>20190608</td>
      <td>2019-06-08</td>
      <td>Chrissy Teigen Wore the Pleated-Jean Trend</td>
      <td>Who What Wear</td>
      <td>35-44</td>
      <td>female</td>
      <td>m.facebook.com</td>
      <td>referral</td>
      <td>21249</td>
      <td>18923</td>
    </tr>
    <tr>
      <th>3</th>
      <td>20190606</td>
      <td>2019-06-06</td>
      <td>Chrissy Teigen Wore the Pleated-Jean Trend</td>
      <td>Who What Wear</td>
      <td>45-54</td>
      <td>female</td>
      <td>m.facebook.com</td>
      <td>referral</td>
      <td>18858</td>
      <td>16825</td>
    </tr>
    <tr>
      <th>4</th>
      <td>20190608</td>
      <td>2019-06-08</td>
      <td>Chrissy Teigen Wore the Pleated-Jean Trend</td>
      <td>Who What Wear</td>
      <td>25-34</td>
      <td>female</td>
      <td>m.facebook.com</td>
      <td>referral</td>
      <td>18157</td>
      <td>15970</td>
    </tr>
  </tbody>
</table>
</div>




```python
male = data_file[data_file.gender == 'male']
male.head()
female = data_file[data_file.gender == 'female']
female.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>date</th>
      <th>new_date</th>
      <th>Page_Title</th>
      <th>Site</th>
      <th>age</th>
      <th>gender</th>
      <th>Source</th>
      <th>Medium</th>
      <th>pageviews</th>
      <th>Unique Pageviews</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>20190606</td>
      <td>2019-06-06</td>
      <td>Chrissy Teigen Wore the Pleated-Jean Trend</td>
      <td>Who What Wear</td>
      <td>35-44</td>
      <td>female</td>
      <td>m.facebook.com</td>
      <td>referral</td>
      <td>35541</td>
      <td>31670</td>
    </tr>
    <tr>
      <th>1</th>
      <td>20190606</td>
      <td>2019-06-06</td>
      <td>Chrissy Teigen Wore the Pleated-Jean Trend</td>
      <td>Who What Wear</td>
      <td>25-34</td>
      <td>female</td>
      <td>m.facebook.com</td>
      <td>referral</td>
      <td>29730</td>
      <td>26236</td>
    </tr>
    <tr>
      <th>2</th>
      <td>20190608</td>
      <td>2019-06-08</td>
      <td>Chrissy Teigen Wore the Pleated-Jean Trend</td>
      <td>Who What Wear</td>
      <td>35-44</td>
      <td>female</td>
      <td>m.facebook.com</td>
      <td>referral</td>
      <td>21249</td>
      <td>18923</td>
    </tr>
    <tr>
      <th>3</th>
      <td>20190606</td>
      <td>2019-06-06</td>
      <td>Chrissy Teigen Wore the Pleated-Jean Trend</td>
      <td>Who What Wear</td>
      <td>45-54</td>
      <td>female</td>
      <td>m.facebook.com</td>
      <td>referral</td>
      <td>18858</td>
      <td>16825</td>
    </tr>
    <tr>
      <th>4</th>
      <td>20190608</td>
      <td>2019-06-08</td>
      <td>Chrissy Teigen Wore the Pleated-Jean Trend</td>
      <td>Who What Wear</td>
      <td>25-34</td>
      <td>female</td>
      <td>m.facebook.com</td>
      <td>referral</td>
      <td>18157</td>
      <td>15970</td>
    </tr>
  </tbody>
</table>
</div>




```python
age = data_file[data_file.age == '25-34']
age.head()
age2 = data_file[data_file.age == '35-44']
age2.describe()
age3 = data_file[data_file.age == '18-24']
age3.describe()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>date</th>
      <th>pageviews</th>
      <th>Unique Pageviews</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>1.740000e+03</td>
      <td>1740.000000</td>
      <td>1740.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>2.019062e+07</td>
      <td>263.436207</td>
      <td>213.547701</td>
    </tr>
    <tr>
      <th>std</th>
      <td>8.767048e+00</td>
      <td>395.634307</td>
      <td>333.429927</td>
    </tr>
    <tr>
      <th>min</th>
      <td>2.019060e+07</td>
      <td>10.000000</td>
      <td>10.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>2.019061e+07</td>
      <td>54.000000</td>
      <td>45.000000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>2.019062e+07</td>
      <td>162.000000</td>
      <td>130.000000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>2.019062e+07</td>
      <td>316.000000</td>
      <td>250.000000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>2.019063e+07</td>
      <td>5715.000000</td>
      <td>4780.000000</td>
    </tr>
  </tbody>
</table>
</div>




```python
topsite = data_file[data_file.Site == ' Who What Wear']
topsite.head()
UK = data_file[data_file.Site == ' Who What WearUK']
UK.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>date</th>
      <th>new_date</th>
      <th>Page_Title</th>
      <th>Site</th>
      <th>age</th>
      <th>gender</th>
      <th>Source</th>
      <th>Medium</th>
      <th>pageviews</th>
      <th>Unique Pageviews</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>75</th>
      <td>20190613</td>
      <td>2019-06-13</td>
      <td>The Best Ways to Apply Blush for Every Face Sh...</td>
      <td>Who What WearUK</td>
      <td>35-44</td>
      <td>female</td>
      <td>m.facebook.com</td>
      <td>referral</td>
      <td>7716</td>
      <td>6679</td>
    </tr>
    <tr>
      <th>115</th>
      <td>20190613</td>
      <td>2019-06-13</td>
      <td>The Best Ways to Apply Blush for Every Face Sh...</td>
      <td>Who What WearUK</td>
      <td>45-54</td>
      <td>female</td>
      <td>m.facebook.com</td>
      <td>referral</td>
      <td>6282</td>
      <td>5394</td>
    </tr>
    <tr>
      <th>249</th>
      <td>20190613</td>
      <td>2019-06-13</td>
      <td>The Best Ways to Apply Blush for Every Face Sh...</td>
      <td>Who What WearUK</td>
      <td>55-64</td>
      <td>female</td>
      <td>m.facebook.com</td>
      <td>referral</td>
      <td>4556</td>
      <td>3908</td>
    </tr>
    <tr>
      <th>314</th>
      <td>20190613</td>
      <td>2019-06-13</td>
      <td>The Best Ways to Apply Blush for Every Face Sh...</td>
      <td>Who What WearUK</td>
      <td>25-34</td>
      <td>female</td>
      <td>m.facebook.com</td>
      <td>referral</td>
      <td>4141</td>
      <td>3573</td>
    </tr>
    <tr>
      <th>672</th>
      <td>20190602</td>
      <td>2019-06-02</td>
      <td>Jennifer Aniston Just Nailed Smart Summer Style</td>
      <td>Who What WearUK</td>
      <td>55-64</td>
      <td>female</td>
      <td>from.flipboard.com</td>
      <td>referral</td>
      <td>2578</td>
      <td>1929</td>
    </tr>
  </tbody>
</table>
</div>




```python
#pull the some entires based on our word cloud
term = 'Celebrity'
celeb_titles = [i for i in data_file.Page_Title if term in i] 
celeb_titles[0:5]
#can add to list later on
```




    ['3 Items a Celebrity Stylist Would Remove From Your Suitcase ',
     'The 11 Best Celebrity Outfits of Summer 2019 ',
     '3 Items a Celebrity Stylist Would Remove From Your Suitcase ',
     'The 11 Best Celebrity Outfits of Summer 2019 ',
     'The 9 Most Daring Celebrity Bikini Trends of 2019 ']




```python
JuneTwentyFour = data_file[data_file.new_date == '2019-06-24']
JuneSix = data_file[data_file.new_date == '2019-06-06']
JuneTwentyEight = data_file[data_file.new_date == '2019-06-28']
JuneTwentyEight.describe()
#data_file.new_date.describe()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>date</th>
      <th>pageviews</th>
      <th>Unique Pageviews</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>1291.0</td>
      <td>1291.000000</td>
      <td>1291.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>20190628.0</td>
      <td>358.471727</td>
      <td>293.086754</td>
    </tr>
    <tr>
      <th>std</th>
      <td>0.0</td>
      <td>659.324892</td>
      <td>545.480261</td>
    </tr>
    <tr>
      <th>min</th>
      <td>20190628.0</td>
      <td>10.000000</td>
      <td>10.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>20190628.0</td>
      <td>72.000000</td>
      <td>57.000000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>20190628.0</td>
      <td>166.000000</td>
      <td>135.000000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>20190628.0</td>
      <td>344.500000</td>
      <td>286.000000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>20190628.0</td>
      <td>6402.000000</td>
      <td>5267.000000</td>
    </tr>
  </tbody>
</table>
</div>



### <a name="bus"></a> III. BUSINESS INTERESTS
---

- WHO IS OUR AUDIENCE?
	
- WHAT DEMOGRAPHIC SEGMENT?

- WHAT CONTENT RESONATES WITH THEM?

- WHAT ARE THE ARTICLE THEMES AND FRAMES THAT ARE DRIVING SITE ENGAGEMENT AND PERFORMANCE?

- WHAT ARE THE CHANNELS THAT ARE DRIVING ENGAGEMENT? 

- WHICH TRAFFIC SOURCES SHOULD WE LEAN INTO, WHICH TRAFFIC SOURCES SHOULD WE MOVE AWAY FROM?

- BULLET SUMMARY

__Did the date of posting have a statistical significance on the pageviews or unique pageviews?__


```python
#thou_samps = data_file.sample(n=1000, random_state=9999)
thou_samps_JuneTwentyFour = JuneTwentyFour.sample(n=1000, random_state=9999)
thou_samps_JuneSix = JuneSix.sample(n=1000, random_state=9999)
thou_samps_JuneSix = thou_samps_JuneSix[['pageviews', 'Unique Pageviews']]
thou_samps_JuneSix.describe()
#I've chosen to sample these two days because they both had ~1300 entries so I could pull a sizeable sample
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>pageviews</th>
      <th>Unique Pageviews</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>1000.000000</td>
      <td>1000.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>445.020000</td>
      <td>376.219000</td>
    </tr>
    <tr>
      <th>std</th>
      <td>1764.797615</td>
      <td>1555.475049</td>
    </tr>
    <tr>
      <th>min</th>
      <td>10.000000</td>
      <td>10.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>57.750000</td>
      <td>47.000000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>151.000000</td>
      <td>118.000000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>354.250000</td>
      <td>289.250000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>35541.000000</td>
      <td>31670.000000</td>
    </tr>
  </tbody>
</table>
</div>




```python
thou_samps_JuneTwentyFour = thou_samps_JuneTwentyFour[['pageviews', 'Unique Pageviews']]
#thou_samps_JuneTwentyFour.info()
thou_samps_JuneTwentyFour.describe()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>pageviews</th>
      <th>Unique Pageviews</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>1000.000000</td>
      <td>1000.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>393.013000</td>
      <td>317.930000</td>
    </tr>
    <tr>
      <th>std</th>
      <td>755.245737</td>
      <td>620.374654</td>
    </tr>
    <tr>
      <th>min</th>
      <td>10.000000</td>
      <td>9.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>77.000000</td>
      <td>60.000000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>184.000000</td>
      <td>143.500000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>384.000000</td>
      <td>310.000000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>9286.000000</td>
      <td>7424.000000</td>
    </tr>
  </tbody>
</table>
</div>



H0: thou_samps_JuneTwentyFour = thou_samps_JuneSix

H1: thou_samps_JuneTwentyFour =/= thou_samps_JuneSix


```python
from scipy import stats
date_stats=stats.ttest_rel(thou_samps_JuneSix,thou_samps_JuneTwentyFour)
date_stats
```




    Ttest_relResult(statistic=array([ 0.85739274,  1.10175326]), pvalue=array([ 0.3914335 ,  0.27083433]))



__ Because PVAL > 0.05, we fail to reject the H0 that the samples are statistically different, suggesting that our differences are instead a result of random chance__

---

__Based on the dataset, it seems that our audience consists of mainly females, aged 24-35, in the US, but is this just a coincidence? Let's run some tests and see what our results say __


```python
#male.info()
#female.info()
thou_samps_male = male.sample(n=1000, random_state=9999)
thou_samps_female = female.sample(n=1000, random_state=9999)
thou_samps_male = thou_samps_male[['pageviews', 'Unique Pageviews']]
thou_samps_male.describe()
thou_samps_female = thou_samps_female[['pageviews', 'Unique Pageviews']]
thou_samps_female.describe()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>pageviews</th>
      <th>Unique Pageviews</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>1000.000000</td>
      <td>1000.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>403.501000</td>
      <td>334.481000</td>
    </tr>
    <tr>
      <th>std</th>
      <td>764.099404</td>
      <td>649.303251</td>
    </tr>
    <tr>
      <th>min</th>
      <td>10.000000</td>
      <td>9.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>61.750000</td>
      <td>50.000000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>183.500000</td>
      <td>151.000000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>405.250000</td>
      <td>335.000000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>8718.000000</td>
      <td>7466.000000</td>
    </tr>
  </tbody>
</table>
</div>



H0: thou_samps_male = thou_samps_female

H1: thou_samps_male =/= thou_samps_female


```python
gender_stats = stats.ttest_rel(thou_samps_female,thou_samps_male)
gender_stats
```




    Ttest_relResult(statistic=array([ 8.49477055,  8.62400515]), pvalue=array([  7.12031839e-17,   2.50599408e-17]))



__ Because PVAL < 0.05, we reject the null hypothesis and find that there is statistical evidence that there is a difference between the pageviews and sessions of Males and Females, giving credibility to our theory about our audience and demographics __


```python
#age this is a segment of site visitors that are 25-34
#age2 this is a segment of site visitors that are 35-44
#age3 this is a segment of site visitors that are 18-24

thou_samps_age = age.sample(n=1000, random_state=9999)
thou_samps_age3 = age3.sample(n=1000, random_state=9999)
thou_samps_age = thou_samps_age[['pageviews', 'Unique Pageviews']]
thou_samps_age.describe()
thou_samps_age3 = thou_samps_age3[['pageviews', 'Unique Pageviews']]
thou_samps_age3.describe()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>pageviews</th>
      <th>Unique Pageviews</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>1000.000000</td>
      <td>1000.00000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>274.024000</td>
      <td>222.32000</td>
    </tr>
    <tr>
      <th>std</th>
      <td>383.326431</td>
      <td>322.74467</td>
    </tr>
    <tr>
      <th>min</th>
      <td>10.000000</td>
      <td>10.00000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>58.000000</td>
      <td>48.75000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>174.000000</td>
      <td>138.00000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>329.250000</td>
      <td>262.00000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>3977.000000</td>
      <td>3394.00000</td>
    </tr>
  </tbody>
</table>
</div>



H0: thou_samps_age = thou_samps_age3

H1: thou_samps_age =/= thou_samps_age3


```python
age_stats = stats.ttest_rel(thou_samps_age,thou_samps_age3)
age_stats
```




    Ttest_relResult(statistic=array([ 5.45961694,  5.43497152]), pvalue=array([  6.02072836e-08,   6.88756299e-08]))



__ Because PVAL < 0.05, we reject the null hypothesis and find that there is statistical evidence that there is a difference between the pageviews and sessions of 18-24 yr olds and 25-34 yr olds, giving credibility to our theory about our audience and demographics __


```python
#topsite.info() 
#this is a segment of articles from WHO WHAT WEAR
#UK.info() 
#this is a segment of articles from WHO WHAT WEAR UK, 
#we only have ~600 entries here, so we will use a smaller sample size

thou_samps_topsite = topsite.sample(n=500, random_state=9999)
thou_samps_UK = UK.sample(n=500, random_state=9999)
thou_samps_topsite = thou_samps_topsite[['pageviews', 'Unique Pageviews']]
thou_samps_topsite.describe()
thou_samps_UK = thou_samps_UK[['pageviews', 'Unique Pageviews']]
thou_samps_UK.describe()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>pageviews</th>
      <th>Unique Pageviews</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>500.000000</td>
      <td>500.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>320.942000</td>
      <td>268.334000</td>
    </tr>
    <tr>
      <th>std</th>
      <td>566.769192</td>
      <td>482.432188</td>
    </tr>
    <tr>
      <th>min</th>
      <td>10.000000</td>
      <td>10.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>68.750000</td>
      <td>59.750000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>194.500000</td>
      <td>163.000000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>385.750000</td>
      <td>321.000000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>7716.000000</td>
      <td>6679.000000</td>
    </tr>
  </tbody>
</table>
</div>



H0: thou_samps_topsite = thou_samps_UK

H1: thou_samps_topsite =/= thou_samps_UK


```python
site_stats = stats.ttest_rel(thou_samps_topsite,thou_samps_UK)
site_stats
```




    Ttest_relResult(statistic=array([ 1.90133849,  1.75080972]), pvalue=array([ 0.05783345,  0.08059317]))



__Our results here seem very close. PVAL is right around the significance value. 
I would say these results are inconclusive for the time being. __

Additional steps would be to 
- increase the sample size (maybe use more than one month of data?)
- Possibly raise our critical value to be more exact

__How does the content resonate? I assume that this is indicated by high pageviews, let's take a look...__


```python
#data_file.pageviews.describe()
#Looking at the breakdown of pageviews, our top 25% is between 369.0 - 35541.0
#Our bottom 25% is between 9 - 61
#Let's pull a subset of each and look at the articles included in each.

top_page = data_file[data_file.pageviews >= 369.0]
top_page.info()

bottom_page = data_file[data_file.pageviews <= 61.0]
bottom_page.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 8199 entries, 0 to 8198
    Data columns (total 10 columns):
    date                8199 non-null int64
    new_date            8199 non-null datetime64[ns]
    Page_Title          8199 non-null object
    Site                8199 non-null category
    age                 8199 non-null category
    gender              8199 non-null category
    Source              8199 non-null category
    Medium              8199 non-null category
    pageviews           8199 non-null int64
    Unique Pageviews    8199 non-null int64
    dtypes: category(5), datetime64[ns](1), int64(3), object(1)
    memory usage: 425.1+ KB
    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 8222 entries, 24533 to 32754
    Data columns (total 10 columns):
    date                8222 non-null int64
    new_date            8222 non-null datetime64[ns]
    Page_Title          8222 non-null object
    Site                8222 non-null category
    age                 8222 non-null category
    gender              8222 non-null category
    Source              8222 non-null category
    Medium              8222 non-null category
    pageviews           8222 non-null int64
    Unique Pageviews    8222 non-null int64
    dtypes: category(5), datetime64[ns](1), int64(3), object(1)
    memory usage: 426.3+ KB



```python
#wordcloud library seems faulty, need to load in at each run
from wordcloud import wordcloud
text = " ".join(review for review in top_page.Page_Title)
wordcloud = wordcloud.WordCloud(background_color="white").generate(text)
plt.imshow(wordcloud, interpolation='bilinear')
plt.axis("off")
plt.title("Highest Pageviews")
plt.show()
```


![png](output_78_0.png)



```python
#wordcloud library seems faulty, need to load in at each run
from wordcloud import wordcloud
text = " ".join(review for review in bottom_page.Page_Title)
wordcloud = wordcloud.WordCloud(background_color="white").generate(text)
plt.imshow(wordcloud, interpolation='bilinear')
plt.axis("off")
plt.title("Lowest Pageviews")
plt.show()
```


![png](output_79_0.png)


Interesting results, it seems like even though we saw that "Celebrity" was the most used word in posts earlier, articles with that word did not get the most page views and it in fact, only exists in the word cloud with the lowest pageviews. We can see that in the popular pageviews, there are names of celebrities, like Jennifer Lopez, Chrissy Teigan, Teigan Wore, etc. So while the word "Celebrity", doesn't seem to get pageviews, the actual specific names seem to get better results. We also see popularity with the word "summer" which makes sense since this data is from June. Additionally, we should note that "Fashion Trend" appears frequently in both the top pageviews and the bottom pageviews.

__Article themes that drive engagement and performance?  I assume that this is indicated by high sessions (unique pageviews), let's take a look...__


```python
#data_file.info()
#rename column because the space in the name is making it mad
data_file = data_file.rename(columns={"Unique Pageviews": "Unique_page"})

#Looking at the breakdown of Unique Pageviews, our top 25% is between 303.0 - 31670.0
#Our bottom 25% is between 7 - 50
#Let's pull a subset of each and look at the articles included in each.

top_unq = data_file[data_file.Unique_page >= 303.0]
top_unq.info()

bottom_unq = data_file[data_file.Unique_page <= 50.0]
bottom_unq.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 8191 entries, 0 to 9953
    Data columns (total 10 columns):
    date           8191 non-null int64
    new_date       8191 non-null datetime64[ns]
    Page_Title     8191 non-null object
    Site           8191 non-null category
    age            8191 non-null category
    gender         8191 non-null category
    Source         8191 non-null category
    Medium         8191 non-null category
    pageviews      8191 non-null int64
    Unique_page    8191 non-null int64
    dtypes: category(5), datetime64[ns](1), int64(3), object(1)
    memory usage: 424.7+ KB
    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 8258 entries, 19419 to 32754
    Data columns (total 10 columns):
    date           8258 non-null int64
    new_date       8258 non-null datetime64[ns]
    Page_Title     8258 non-null object
    Site           8258 non-null category
    age            8258 non-null category
    gender         8258 non-null category
    Source         8258 non-null category
    Medium         8258 non-null category
    pageviews      8258 non-null int64
    Unique_page    8258 non-null int64
    dtypes: category(5), datetime64[ns](1), int64(3), object(1)
    memory usage: 428.2+ KB



```python
#wordcloud library seems faulty, need to load in at each run
from wordcloud import wordcloud
text = " ".join(review for review in top_unq.Page_Title)
wordcloud = wordcloud.WordCloud(background_color="white").generate(text)
plt.imshow(wordcloud, interpolation='bilinear')
plt.axis("off")
plt.title("Highest Unique Pageviews")
plt.show()
```


![png](output_83_0.png)



```python
#wordcloud library seems faulty, need to load in at each run
from wordcloud import wordcloud
text = " ".join(review for review in bottom_unq.Page_Title)
wordcloud = wordcloud.WordCloud(background_color="white").generate(text)
plt.imshow(wordcloud, interpolation='bilinear')
plt.axis("off")
plt.title("Lowest Unique Pageviews")
plt.show()
```


![png](output_84_0.png)


As with the Top Pageviews, we are seeing a higher frequency of words like "summer" and specific celebrity names. Additionally, we see that "celebrity" is listed multiple times in the low unique pageview wordcloud, and doesn't exist in the top unique pageciew wordcloud. This is a good indicator to me that article titles should try to have specific celebrity names in the titles and doing so will be more engaging to customers. 
Again we see that "Fashion Trend" is listed in both the highest and the lowest Unique Pageview wordclouds. 

__Where are our main traffic sources coming from? Let's look at the source for our top pageviews and unique pageviews__


```python
top_page.describe(include='all')
top_unq.describe(include='all')

#Our data shows that for the top pageviews and unique pageviews, our source is Facebook mobile.
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>date</th>
      <th>new_date</th>
      <th>Page_Title</th>
      <th>Site</th>
      <th>age</th>
      <th>gender</th>
      <th>Source</th>
      <th>Medium</th>
      <th>pageviews</th>
      <th>Unique Pageviews</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>8.199000e+03</td>
      <td>8199</td>
      <td>8199</td>
      <td>8199</td>
      <td>8199</td>
      <td>8199</td>
      <td>8199</td>
      <td>8199</td>
      <td>8199.000000</td>
      <td>8199.000000</td>
    </tr>
    <tr>
      <th>unique</th>
      <td>NaN</td>
      <td>30</td>
      <td>651</td>
      <td>4</td>
      <td>6</td>
      <td>2</td>
      <td>38</td>
      <td>11</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>top</th>
      <td>NaN</td>
      <td>2019-06-24 00:00:00</td>
      <td>Chrissy Teigen Wore the Pleated-Jean Trend</td>
      <td>Who What Wear</td>
      <td>25-34</td>
      <td>female</td>
      <td>m.facebook.com</td>
      <td>referral</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>freq</th>
      <td>NaN</td>
      <td>357</td>
      <td>146</td>
      <td>7842</td>
      <td>2859</td>
      <td>7939</td>
      <td>2379</td>
      <td>5210</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>first</th>
      <td>NaN</td>
      <td>2019-06-01 00:00:00</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>last</th>
      <td>NaN</td>
      <td>2019-06-30 00:00:00</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>2.019062e+07</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1141.871570</td>
      <td>945.880595</td>
    </tr>
    <tr>
      <th>std</th>
      <td>8.913794e+00</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1497.872842</td>
      <td>1277.085650</td>
    </tr>
    <tr>
      <th>min</th>
      <td>2.019060e+07</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>369.000000</td>
      <td>135.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>2.019061e+07</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>476.000000</td>
      <td>390.000000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>2.019062e+07</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>658.000000</td>
      <td>546.000000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>2.019062e+07</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1163.500000</td>
      <td>966.000000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>2.019063e+07</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>35541.000000</td>
      <td>31670.000000</td>
    </tr>
  </tbody>
</table>
</div>




```python
top_page['Source'] = top_page['Source'].astype('category')
page_source = pd.crosstab(index=top_page['Source'],columns="count") 
page_source = page_source.reset_index()
page_source_sort = page_source.sort_values(by=['count'], ascending=False)
page_source_sort.head()
#the results show the top 5 sources for the top pageviews are from 
##Mobile Facebook, newsletter, Google, Flipboard, and WhoWhatWear.com
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>col_0</th>
      <th>Source</th>
      <th>count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>32</th>
      <td>m.facebook.com</td>
      <td>2379</td>
    </tr>
    <tr>
      <th>40</th>
      <td>newsletter</td>
      <td>1254</td>
    </tr>
    <tr>
      <th>20</th>
      <td>google</td>
      <td>1131</td>
    </tr>
    <tr>
      <th>18</th>
      <td>from.flipboard.com</td>
      <td>805</td>
    </tr>
    <tr>
      <th>65</th>
      <td>www-whowhatwear-com.cdn.ampproject.org</td>
      <td>654</td>
    </tr>
  </tbody>
</table>
</div>



__Channels driving engagment? Let's look at the medium information and see how our customers are getting to our articles. __


```python
top_page.describe(include='all')
top_unq.describe(include='all')

#Our data shows that for the top pageviews and unique pageviews, our source is referral.
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>date</th>
      <th>new_date</th>
      <th>Page_Title</th>
      <th>Site</th>
      <th>age</th>
      <th>gender</th>
      <th>Source</th>
      <th>Medium</th>
      <th>pageviews</th>
      <th>Unique_page</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>8.191000e+03</td>
      <td>8191</td>
      <td>8191</td>
      <td>8191</td>
      <td>8191</td>
      <td>8191</td>
      <td>8191</td>
      <td>8191</td>
      <td>8191.000000</td>
      <td>8191.000000</td>
    </tr>
    <tr>
      <th>unique</th>
      <td>NaN</td>
      <td>30</td>
      <td>651</td>
      <td>4</td>
      <td>6</td>
      <td>2</td>
      <td>38</td>
      <td>11</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>top</th>
      <td>NaN</td>
      <td>2019-06-02 00:00:00</td>
      <td>Chrissy Teigen Wore the Pleated-Jean Trend</td>
      <td>Who What Wear</td>
      <td>25-34</td>
      <td>female</td>
      <td>m.facebook.com</td>
      <td>referral</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>freq</th>
      <td>NaN</td>
      <td>355</td>
      <td>154</td>
      <td>7819</td>
      <td>2828</td>
      <td>7929</td>
      <td>2475</td>
      <td>5239</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>first</th>
      <td>NaN</td>
      <td>2019-06-01 00:00:00</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>last</th>
      <td>NaN</td>
      <td>2019-06-30 00:00:00</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>2.019062e+07</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1141.111708</td>
      <td>947.848614</td>
    </tr>
    <tr>
      <th>std</th>
      <td>8.927721e+00</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1499.182890</td>
      <td>1276.865650</td>
    </tr>
    <tr>
      <th>min</th>
      <td>2.019060e+07</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>309.000000</td>
      <td>303.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>2.019061e+07</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>476.000000</td>
      <td>391.000000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>2.019062e+07</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>659.000000</td>
      <td>547.000000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>2.019062e+07</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1165.500000</td>
      <td>966.500000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>2.019063e+07</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>35541.000000</td>
      <td>31670.000000</td>
    </tr>
  </tbody>
</table>
</div>




```python
top_page['Medium'] = top_page['Medium'].astype('category')
page_med = pd.crosstab(index=top_page['Medium'],columns="count") 
page_med = page_med.reset_index()
page_med_sort = page_med.sort_values(by=['count'], ascending=False)
page_med_sort.head()
#the results show the top 3 medium for the top pageviews are via 
##"referral" by a long shot, "email", and "organic"
##Not sure what "(none)" refers to in the "medium" category...
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>col_0</th>
      <th>Medium</th>
      <th>count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>13</th>
      <td>referral</td>
      <td>5210</td>
    </tr>
    <tr>
      <th>6</th>
      <td>email</td>
      <td>1254</td>
    </tr>
    <tr>
      <th>11</th>
      <td>organic</td>
      <td>1131</td>
    </tr>
    <tr>
      <th>0</th>
      <td>(none)</td>
      <td>268</td>
    </tr>
    <tr>
      <th>14</th>
      <td>social</td>
      <td>161</td>
    </tr>
  </tbody>
</table>
</div>





- WHO IS OUR AUDIENCE? WHAT DEMOGRAPHIC SEGMENT?

Our data for the month of June 2019, shows lots of posts from the domestic Who What Wear site that received engagment from females older than 24. Statistically, we have found evidence that our articles are standing out with females and people in the 25-34 age group.

- WHAT CONTENT RESONATES WITH THEM? WHAT ARE THE ARTICLE THEMES AND FRAMES THAT ARE DRIVING SITE ENGAGEMENT AND PERFORMANCE?

We found that content that has trigger words like "Summer", "Trend", and specific celebrity names were frequently in the top pageview and top sessions. Based on the wordclouds, I would recommend using titles that are as specific as possible. General words like "celebrity" consistently showed up in the lowest pageviews and sessions.

- WHAT ARE THE CHANNELS THAT ARE DRIVING ENGAGEMENT? WHICH TRAFFIC SOURCES SHOULD WE LEAN INTO, WHICH TRAFFIC SOURCES SHOULD WE MOVE AWAY FROM?

Our top pageviews and sessions were driven by mobile facebook referral and seem to be for a specific article about "Chrissy Teigan Wore the Pleated Jean Trend", confirming the theory that more specific articles are more engaging. 


### <a name="con"></a> IV. CONCLUSIONS
---
WHAT DOES IT MEAN IN CONTEXT? <BR>

In June 2019, the articles seems to be attracting female customers older than 24. In terms of content, it seems like the more specific the article the better. Engagement is higher for articles with specific celebrity names (Jennifer Lopez or Chrissy Teigan) and seasons (Summer Trends) versus those with more generalized themes and verbiage. This is interesting to note because the word "celebrity" was one of the most frequent words used in titles from our dataset. While our most popular source in the dataset was "google", the source that had the most engagement was "Facebook Mobile". It would be interesting to compare this months data against other months to see if these themes hold true and constant. 

__bonus! not requested, but color me curious. 
what males are looking at on clique brands?__


```python
male.info()
male.describe()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 3445 entries, 124 to 32753
    Data columns (total 10 columns):
    date                3445 non-null int64
    new_date            3445 non-null datetime64[ns]
    Page_Title          3445 non-null object
    Site                3445 non-null category
    age                 3445 non-null category
    gender              3445 non-null category
    Source              3445 non-null category
    Medium              3445 non-null category
    pageviews           3445 non-null int64
    Unique Pageviews    3445 non-null int64
    dtypes: category(5), datetime64[ns](1), int64(3), object(1)
    memory usage: 179.0+ KB





<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>date</th>
      <th>pageviews</th>
      <th>Unique Pageviews</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>3.445000e+03</td>
      <td>3445.000000</td>
      <td>3445.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>2.019062e+07</td>
      <td>160.653120</td>
      <td>128.984325</td>
    </tr>
    <tr>
      <th>std</th>
      <td>8.862308e+00</td>
      <td>363.789775</td>
      <td>288.561527</td>
    </tr>
    <tr>
      <th>min</th>
      <td>2.019060e+07</td>
      <td>10.000000</td>
      <td>10.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>2.019061e+07</td>
      <td>29.000000</td>
      <td>25.000000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>2.019062e+07</td>
      <td>73.000000</td>
      <td>60.000000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>2.019062e+07</td>
      <td>161.000000</td>
      <td>130.000000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>2.019063e+07</td>
      <td>6093.000000</td>
      <td>4691.000000</td>
    </tr>
  </tbody>
</table>
</div>




```python
#frequent page titles
male_titles = male['Page_Title'].astype('category')
male_titles = pd.crosstab(index=male['Page_Title'],columns="count") 
male_titles = male_titles.reset_index()
male_titles_sort = male_titles.sort_values(by=['count'], ascending=False)
male_titles_sort.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>col_0</th>
      <th>Page_Title</th>
      <th>count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>153</th>
      <td>Celebrity Style and Fashion Trend Coverage</td>
      <td>135</td>
    </tr>
    <tr>
      <th>348</th>
      <td>These $69 Treggings "Flatter Every Body Type"</td>
      <td>133</td>
    </tr>
    <tr>
      <th>155</th>
      <td>Chrissy Teigen Wore the Pleated-Jean Trend</td>
      <td>117</td>
    </tr>
    <tr>
      <th>172</th>
      <td>How to Become a Morning Person</td>
      <td>91</td>
    </tr>
    <tr>
      <th>310</th>
      <td>The Best Miami Summer Trends for Under $100</td>
      <td>74</td>
    </tr>
  </tbody>
</table>
</div>




```python
#
top_page2 = male[male.pageviews >= 130.0]
top_page2.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 1093 entries, 124 to 19043
    Data columns (total 10 columns):
    date                1093 non-null int64
    new_date            1093 non-null datetime64[ns]
    Page_Title          1093 non-null object
    Site                1093 non-null category
    age                 1093 non-null category
    gender              1093 non-null category
    Source              1093 non-null category
    Medium              1093 non-null category
    pageviews           1093 non-null int64
    Unique Pageviews    1093 non-null int64
    dtypes: category(5), datetime64[ns](1), int64(3), object(1)
    memory usage: 57.3+ KB



```python
#wordcloud library seems faulty, need to load in at each run
from wordcloud import wordcloud
text = " ".join(review for review in top_page2.Page_Title)
wordcloud = wordcloud.WordCloud(background_color="white").generate(text)
plt.imshow(wordcloud, interpolation='bilinear')
plt.axis("off")
plt.title("Highest Pageviews for Males")
plt.show()
```


![png](output_98_0.png)


Lol seems like there is some male interest in Flattering Every Body Type and Treggings (what are those?)
