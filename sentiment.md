# Stock Website Webscrapping & Sentiment Analysis

<img src="images/sentimentcover.png?raw=true" style="width:500px;"/><br>

**Project description:**  The script below is created by following closely an online <a href="https://www.youtube.com/watch?v=o-zM8onpQZY&ab_channel=TheCodex">Youtue Tutorial</a>. The script parses through finviz.com (a stock financial information site) for news headlines based on the stock tickers that the user's input into the script. NLP alogrithms are applied to these headlines to analyze the sentiment, providing a score. The scores are then visualized through a matplotlib plot and are categorized based on ticker and date.

<br>
<br>
<br>


## 1. Imports

```python
from urllib.request import urlopen, Request
from bs4 import BeautifulSoup
import nltk
from nltk.sentiment.vader import SentimentIntensityAnalyzer
from textblob import TextBlob
import pandas as pd
import matplotlib.pyplot as plt
```
<br>

The above are libraries are imported:
1. urllib.request module for fetching URLs.
2. Beautiful Soup module for parsing HTML and XML documents. It generates a parse tree for parsed pages that can be used to extract data from HTML.
3. nltk module for natural language processing using vader NLP algorithm.
4. textblob another NLP algorithm used for comparison.
5. Pandas for dataframe creation and manipulation.


<br>

## 2. Beautiful Soup Extraction

```python
finviz_url = 'https://finviz.com/quote.ashx?t='
tickers = ['AMZN', 'GOOG', 'FB']

news_tables = {}
for ticker in tickers:
    url = finviz_url + ticker

    req = Request(url=url, headers={'user-agent': 'my-app'})
    response = urlopen(req)

    html = BeautifulSoup(response, features='html.parser')
    news_table = html.find(id='news-table')
    news_tables[ticker] = news_table
```

<img src="images/newstable.png?raw=true" style="width:500px;"/><br>

We create a list for the stock tickers that will be fitted onto the url, to get the tickers pages that we want to scrape our new headlines from. (in this case its AMZN, GOOG & FB). The for loop allows parsing through each ticker page and saving the information on to the news_table dictionary. The html code reveals that the news headlines are contained within the id='news-table'.

<br>

```python
parsed_data = []

for ticker, news_table in news_tables.items():
    for row in news_table.findAll('tr'):
        title = row.a.text
        date_data = row.td.text.split()

        if len(date_data) == 1:
            time = date_data[0]
        else:
            date = date_data[0]
            time = date_data[1]
        
        parsed_data.append([ticker, date, time, title])
```
<img src="images/titletext.png?raw=true" style="width:500px;"/><br>

Similarly, continuing deeper into parsing for the data, we next locate where the title and date information are stored within the html. Html reveals that the news title are within the "a" tags and the dates are within the "td" tags. The date information appear to have 2 forms, 1 with date / time and 1 with only time. The if/else statements categorizes these 2 formats. Finally, the data is then stored within the list in the form of ticker, date, time and the title information.

<br>

## 3. Sentiment Analysis Examples
```python
print(vader.polarity_scores("I think apple will fail")) #Vader Example
```
<img src="images/vaderex.png?raw=true" style="width:500px;"/><br>

```python
feedback1 = "Food is bad"  #TextBlob Example
feedback2 = "Food is good"
print(TextBlob(feedback1).sentiment)
print(TextBlob(feedback2).sentiment)
```
<img src="images/txtblobex.png?raw=true" style="width:500px;"/><br>
The above shows two different NLP algorithms at work (nltk.vader and TextBlob). Both NLP algorithms are able to detect positive and negative sentiments but the way each presents the scoring information and how they are scored are relatively different.

<br>

## 4. Applying Sentiment Analyzer onto dataframe
```python
df = pd.DataFrame(parsed_data, columns=['ticker', 'date', 'time', 'title'])

vader = SentimentIntensityAnalyzer()

f = lambda title: vader.polarity_scores(title)['compound']
df['compound'] = df['title'].apply(f)
df['date'] = pd.to_datetime(df.date).dt.date

b = lambda title:TextBlob(title).sentiment.polarity
df['polarity'] = df['title'].apply(b)
```
<br>

We use a lambda function to apply the sentiment analyzers onto the titles to obtain the sentiment scores. The dates and times are also converted into more readable format for the script.

<br>

## 5. Plotting the data
```python
plt.figure(figsize=(10,8))
mean_df = df.groupby(['ticker', 'date']).mean().unstack()
mean_df = mean_df.xs('compound', axis='columns').transpose()
mean_df.plot(kind='bar')
plt.show()

plt.figure(figsize=(10,8))
mean_df = df.groupby(['ticker', 'date']).mean().unstack()
mean_df = mean_df.xs('polarity', axis='columns').transpose()
mean_df.plot(kind='bar')
plt.show()
```
<img src="images/vaderres.png?raw=true" style="width:300px;"/><br>
<img src="images/textblobres.png?raw=true" style="width:300px;"/><br>

The data is then plotted using matplotlib. The results shows some difference in the how each sentiment analyzer performs. Both sentiments are similar in rating however, vader results seems to show that it can identify sentiment in a headline whereas TextBlob associates that headline to be neutral.

<br>
<br>
<br>

<a href="https://nicsim92.github.io/" style="font-size: 30px">Return to Main Selection Page</a>




