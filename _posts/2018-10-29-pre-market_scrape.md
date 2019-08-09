---
title: "Pre-Market Data Web Scraper"
date: 2018-10-29
tags: [Web Scraping]
#header: 
    #image: "/images/Iris/output_3_1.png"
exceprt: "Web Scraping pre-market data from benzinga.com"
---
Simple web scraper built in python to get pre-market data from the benzinga markets website. 

```python
from functions import * #importing functions file (functions for getting & verifying url)
from contextlib import closing
from requests import get
from requests.exceptions import RequestException
from bs4 import BeautifulSoup
```

The first step to obtaining the pre-market data from the bezinga markets website is to find the link to the most recent pre-market-session article published on the site. Once that article is obtained we can scrape through the html to find pre-market movers based on user inputted criteria (the user inputs will be pre-determined for this jupyter notebook).


```python
webpage_html = get_url('https://www.benzinga.com/markets') #get html of benzinga markets website
soup = BeautifulSoup(webpage_html, 'html.parser')
```


```python
#finding link to most recent pre-market-session
for link in soup.find_all('a'):
    url_link = link.get('href')
    if 'pre-market-session' in url_link: #finds link to pre-market movers article
        article_url_link = url_link
        break #once pre-market movers link found, break loop
article_url = 'https://www.benzinga.com' + str(article_url_link)
```


```python
stockLstGainers = [] #empty list to store all stocks up some percentage
stockLstLosers = [] #empty list to store all stocks down some percentage
tickerList = [] #empty list to store all tickers to c/p into brokerage platform
```


```python
raw_html = get_url(article_url) #gets html of most recent stock-movers article
html = BeautifulSoup(raw_html, 'html.parser')
```

I added a few more helper functions to find the data and to get the exact date/time the article was posted. 


```python
#gets string between the given criteria
def find_between(s, first, last):
    try:
        start = s.index(first) + len(first) #starting elements you want to index from
        end = s.index(last, start) #ending element to index from
        return s[start:end]
    except ValueError: #return empty string if ValueError
        return ""

#find article date and time Function
def find_article_date():
    #find span with class id of 'date'
    for span in html.find('span', {'class': 'date'}):
        return str(span)
```

Next we will find all stocks/tickers that meet the user inputted criteria and filter out unnecessary text. 


```python
userPercent = 5
print('\n')
for li in html.find_all('li'): #gets each list element on page
    if 'NASDAQ' in li.text or 'NYSE' in li.text: #gets each list element with NASDAQ in string
        if 'NASDAQ' in li.text:
            new_li = find_between(li.text, 'NASDAQ:', 'trading')
        elif 'NYSE' in li.text:
            new_li = find_between(li.text, 'NYSE:', 'trading')
        new_li = new_li.replace(')', '') #string formatting
        new_li = new_li.replace('in', 'in the') #string formatting
        #clean up strings to keep formatting the same
        if 'shares' in new_li:
            new_li = new_li.replace(' shares', '')
        if 'rose' in new_li: #determine if ticker is up some percentage
            #find percent move of each ticker
            percentMoveUp = find_between(new_li, 'rose', 'percent')
            if float(percentMoveUp) >= float(userPercent):
                stockLstGainers.append(new_li) #add all elements to list
        elif 'fell' in new_li: #determine if ticker is down some percentage
            #find percent move down of each ticker
            percentMoveDown = find_between(new_li, 'fell', 'percent')
            if float(percentMoveDown) >= float(userPercent):
                stockLstLosers.append(new_li)
```
I like to add the article date/time to the .txt file that will be created to make sure the list is from today (Monday) and not last week.

```python
article_date = str(find_article_date()) #get article date from function
article_date_lstrip = article_date.lstrip() #strip only leading white space
article_date = article_date.strip() #strip all leading and ending white space
```

Now that all the data is obtained all that is left to do is write all the data into a txt file listing all pre-market gainers/losers and a list of all tickers displayed (I use the ticker list as a way to copy/paste the pre-market movers into my trading platform watchlist).

```python
with open('Pre-market_movers.txt', 'w') as output: #open file for writing
    output.write('Date and Time article posted: {a}'.format(a=article_date_lstrip) + '\n' + '\n')
    output.write('Stocks Up in Pre-Market:' + '\n' + '\n')
    for stocksUp in stockLstGainers: #write all gainers into file
        q = stocksUp.split('rose', 1)[0].strip()
        tickerList.append(q) #append tickers of all gainers to Ticker List
        output.write(str(stocksUp) + '\n')
    output.write('\n')
    output.write('Stocks Down in Pre-Market:' + '\n' + '\n')
    for stocksDown in stockLstLosers: #write all losers into file
        p = stocksDown.split('fell', 1)[0].strip()
        tickerList.append(p) #append tickers of all losers to Ticker Lis t
        output.write(str(stocksDown) + '\n')
    output.write('\n')
    output.write('List of all tickers displayed:' + '\n' + '\n')
    for ticker in tickerList:
        output.write('{a},'.format(a=ticker))
output.close() #close file
```
