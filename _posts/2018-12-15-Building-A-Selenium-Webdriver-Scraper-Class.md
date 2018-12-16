
This is a follow-up to my previous post regarding scrapping using Selenium and Chromedriver to extact data from the web. When I first started out with Python my code was just a huge block of text. The concept of a class was far and away from my ability to comprehend at the time. I'm sure this can be the case with anyone that starts out learning a programming language. But after some experience and learning how classes work, I decided it was high time to rebuild my scraping project with a `Scraper` object. You can check out [my github public repo](https://github.com/virrey/sandbox_public/) for the `Scraper` class I created which is used in this demo and leverages [Selenium Webdriver](https://www.seleniumhq.org/projects/webdriver/). Having created this class, I noticed the my code was much cleaner and easier to follow and debug. Let us dive in.

Here we import my `Scraper` class and the `robotparser`.


```python
from ariScrape import Scraper
from urllib import robotparser 
```

We will again respectfully check the sites `robots.txt` file to ensure we are being considerate of the sites scrapping policies.


```python
def robot_check(url_parts):
    url = ''.join([str(p) for p in url_parts])
    rp = robotparser.RobotFileParser()
    rp.set_url(url)
    rp.read()
    if rp.can_fetch("*", url):
        print("Robots.txt: User Allowed")
        return True
    else:
        print("Robots.txt: User Disallowed. Please abort.")
        return False
```

In this example we'll be scraping the Intel Corp Income Statement from the [NASDAQ](www.nasdaq.com/symbol/INTC) website. We'll call the `robots_check` function to confirm we are good to go on the scraping side. You will notice here I split the url into 3 parts. From my previous projects it was important to be able to frame urls this way because when you are looping through stock tickers only part of the url will change.


```python
# test ticker symbol @ nasdaq.com
test_ticker = "INTC"
url_prefix = "https://www.nasdaq.com/symbol/"
url_suffix = "/financials"
url_parts = [url_prefix,test_ticker,url_suffix]
robot_check(url_parts)
```

    Robots.txt: User Allowed


My `Scraper` class also allows defining parts of the xpath. The reason for this will be obvious in the next steps.


```python
# xpath of @top left of table @url
# >> //*[@id="financials-iframe-wrap"]/div[1]/table/tbody/tr[1]/td[2]
# xpath of @bottom right of table @url
# >> //*[@id="financials-iframe-wrap"]/div[1]/table/tbody/tr[19]/td[5]
# these are the string parts of the xpath that
# will not change while we iterate
xpath_part1 = '//*[@id="financials-iframe-wrap"]/div[1]/table/tbody/tr['
xpath_part2 = ']/td['
xpath_part3 = ']'
```

When you go into the inspector on Chrome you will notice the xpath of the income statement items are effectively a table of rows and columns. Therefore, our `xpath_column_key` and `xpath_row_key` are an enumeration of those columns and rows. You can then see why the `xpath_parts` assignment is perfect for our needs. We create a list of `xpath_parts` list in a double for loop list comprehension by iterating over our column and row keys.


```python
# init the column and row keys for the example table
xpath_column_key = [i for i in range(2,6,1)] # create column index 2-5
xpath_row_key = [i for i in range(1,20,1)] # create row index 1-19

# make an array of xpath_parts that we're going to iterate over
xpath_parts = [[xpath_part1,r,xpath_part2,c,xpath_part3] for c in xpath_column_key for r in xpath_row_key]
```

Now that we have the `url_parts` and our list of `xpath_parts` we are ready to create our `Scraper` object. Set the scraper object property to True if you would like to see all the internal debugging statements from the class. Otherwise, leave this as False. `getChromeDriverPath` will take a path that is user defined or will look for the chromedriver in some default locations. It also checks if you are on Mac or Win. Once the driver is created, set the URL to be scrapped using `setURLFromParts` then `LoadPage`.


```python
# create an ariScrape object
scrapeObj = Scraper()
scrapeObj.verbose = False
scrapeObj.getChromeDriverPath()
scrapeObj.getChromeDriver()
scrapeObj.setURLFromParts(url_parts)
scrapeObj.LoadPage()
```

Chrome should now open in test mode. If you would like the values being extracted to show leave `val_verbose` set to True. The loop iterates through our `xpath_parts` list to go through each of the column/row combinations and extract the value. Then we close the webdriver.


```python
# set this to True if you want to see the extracted
# values -- useful when obj.verbose is False
val_verbose = True
for xp in xpath_parts:
    scrapeObj.setXPathFromParts(xp)
    scrapeObj.ExtractFloatAtXPath()
    if val_verbose:
        print(scrapeObj.scrapped_item)
scrapeObj.driver.close()
```

If you have any recommendations to improving the code please feel free to reach out. I'm always seeking to learn new and better ways of coding. I hope you enjoyed reading, and remember:

Stay Chaotic – Stay Neutral

[ARI](mailto:ari.virrey@gmail.com)
