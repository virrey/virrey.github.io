Beautiful Soup is a Python library and is helpful in parsing web pages programmatically. But before we get into the world of web scraping, we need to have an understanding of the *robots.txt* file (robots file).
The robots file is a file located at every web site. Try it at any top level url proceeded by /robots.txt

Examples:
www.google.com/robots.txt
www.linkedin.com/robots.txt

So what does this robots file do? The robots file provides an exclusion standard that web crawlers and robots should, ideally, adhere to. 

The basic format of a robots file reads:

```html
User-agent: *
Disallow: /
```

User-agent is who the rule applies to. Disallow are the folder and files the robot file indicates are not allowed to be crawled, indexed, or scraped. This example is the most aggressive of all robots exclusions, it indicates everyone is disallowed from everything.

So when we're on the topic of BeautifulSoup it's important to remember that you are using a tool that can be used to scrape html and xml, and we have to be considerate of the robots exclusion protocol.

The example we have is using Beautiful Soup to scrape a list of stock tickers as shown on the en.wikipedia.org/wiki/NASDAQ-100 [page](en.wikipedia.org/wiki/NASDAQ-100). Keep in mind that the method I have shown here is only one of possibly hundreds of ways of extracting this information. I'll drop my entire function definition here and then break it down one step at a time.

```python
def buildCompanyDict():
'''
This function will give the user the option of a) loading the company_dictionary from a pickle file, b) scraping the list of items from a wikipedia page or, c) sideload via a csv file comp_dict is a basic 1-key:1-val that contains the Stock Ticker as the key and the name of the company as the value. (e.g. 'INTC':'Intel corp')
'''
    comp_dict = defaultdict(str)
    opt = input("'p' for pickle load/n's' for soup scrape/n'sl' csv sideload")
    if opt == "p":
        comp_dict = myUnpickling("DictCompanies.pkl")    
    elif opt == "s":
        u = "https://en.wikipedia.org/wiki/NASDAQ-100"
        reply_txt = req.get(u).text
        soup_table = bs(reply_txt,'html.parser').\
        	find('div',{'class':'div-col columns column-width'})
        children = soup_table.findChildren('li')
        children = children[25:28] # subset with [x:y] during dev only
        pat = re.compile(pattern=r'\(([a-zA-Z]*)\)')
        for i in children:
            s = re.split(pat,i.text)
            co = s[0]
            tick = s[1]
            if not tick in BANS:
                comp_dict[tick] = co
        myPickling("DictCompanies",comp_dict)
    # sideload a different list via CSV
    elif opt == "sl":
        with open('nasdaq_dict.csv', 'r') as f:
            r = csv.reader(f) 
            for i,v in enumerate(r):
                if i != 0:
                    co = v[1]
                    tick = v[0]
                    if not tick in BANS:
                        comp_dict[tick] = co
        myPickling("DictCompanies",comp_dict)
    return comp_dict
```

This function will give the user the option of a) loading the company_dictionary from a pickle file
b) scraping the list of items from a [wikipedia](en.wikipedia.org) page or c) sideload via a csv file.
comp_dict is a basic 1-key:1-val that contains the Stock Ticker as the key and the name of the
company as the value. (e.g. 'INTC':'Intel corp'). Since our topic of interest is BeautifulSoup, I'll focus only on the part of the code encapsulated by the *elif opt == 's'*.

```python
elif opt == "s":
        u = "https://en.wikipedia.org/wiki/NASDAQ-100"
        reply_txt = req.get(u).text
        soup_table = bs(reply_txt,'html.parser').\
        	find('div',{'class':'div-col columns column-width'})
        children = soup_table.findChildren('li')
        children = children[25:28] # subset with [x:y] during dev only
        pat = re.compile(pattern=r'\(([a-zA-Z]*)\)')
        for i in children:
            s = re.split(pat,i.text)
            co = s[0]
            tick = s[1]
            if not tick in BANS:
                comp_dict[tick] = co
        myPickling("DictCompanies",comp_dict)
```

First we initialize the url we want to grab information from.

The object req is a request object. More information on [request here](https://github.com/requests/requests). Therefore, our *reply_txt* variable will hold our html before being soupified. We use BeautifulSoup which takes the reply text and a parser. Here we use a standard 'html.parser'. There is no specific reason for this particular parser beyond that it works. If for some reason your parsing and branching does  not appear to be working you can try 'html', 'html5lib', and 'lxml', among others. 

```python
from bs4 import BeautifulSoup as bs
import requests as req

u = "https://en.wikipedia.org/wiki/NASDAQ-100"
reply_txt = req.get(u).text
soup_table = bs(reply_txt,'html.parser').\
        	find('div',{'class':'div-col columns column-width'})
```

Now before we get into the *.find()* we need to go to our target page to really understand what is going on here.

Using the inspector in Chrome, we can see this section of the page is in a *div* with a *class* called *'div-col columns column-width'*. Therefore, our *find* method is setup to target this area of the page and store that in our *soup_table* variable.

![chrome inspect]({{site.baseurl}}/images/1539564743518.jpg)

But we are interested in the individual items on the list. We tackle this by observing that each item in our list is encapsulated in *<li></li>* tags. And that is how we arrive at:

```python
import re

children = soup_table.findChildren('li')
pat = re.compile(pattern=r'\(([a-zA-Z]*)\)')
```

Find children will look within the *div class* soup_table we previously extracted and find all instances of the *<li>* tags and break it out as a child of the parent. The re.compile is where we define the regex pattern we will apply to our children to separate the stock ticker from the company name in the subsequent for loop. To use *re* you will need to import it. re allows you to do regular expressions. If you are unfamiliar with regular expressions, check out the wikipedia [here](https://en.wikipedia.org/wiki/Regular_expression).

```python
for i in children:
    s = re.split(pat,i.text)
    co = s[0]
    tick = s[1]
    comp_dict[tick] = co
```

Fundamentally our for loop does 2 things. First it takes the regex pattern we previously defined and applies it to the string while simultaneously splitting on it. Second, it takes the split components 0 and 1 for the creation of our dictionary.

As an example of the re.split in action in conjunction with the re.compile pattern, "Activision Blizzard (ATVI)" becomes *s[0]* = 'Activision Blizzard' and *s[1]* becomes 'ATVI'. Now that we have our stock ticker and company name we can load them up into our dictionary with *comp_dict[tick] = co*. Note that here I am setting the stock ticker as the key not the company name, effectively in reverse of how they were displayed on the page which makes more sense if I am going to be indexing on the stock ticker in the future.

I hope you enjoyed reading and remember:

Stay Chaotic – Stay Neutral

[ARI](mailto:ari.virrey@gmail.com)