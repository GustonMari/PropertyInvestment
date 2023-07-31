#Scrapping Part

## How to create a project

```bash
scrapy startproject <project_name>
```

## How to create a spider

The class must herit from scrapy.Spider and must have a name attribute.
you can set only one name for a spider and it must be unique for the all project.

* `start_requests()`: must return an iterable of Requests (you can return a list of requests or write a generator function) which the Spider will begin to crawl from.

* `parse()`: a method that will be called to handle the response downloaded for each of the requests made. The response parameter is an instance of `TextResponse` that holds the page content and has further helpful methods to handle it.

## How to run our spider

Go to the top level of the project (the one containing the scrapy.cfg file) and run:

```bash
scrapy crawl <spider_name>
```

Instead of implementing a `start_requests()` method that generates `scrapy.Request` objects from URLs, you can just define a `start_urls` class attribute with a list of URLs. This list will then be used by the default implementation of `start_requests()` to create the initial requests for your spider.

```python
from pathlib import Path

import scrapy


class QuotesSpider(scrapy.Spider):
    name = "quotes"
    start_urls = [
        "https://quotes.toscrape.com/page/1/",
        "https://quotes.toscrape.com/page/2/",
    ]

    def parse(self, response):
        page = response.url.split("/")[-2]
        filename = f"quotes-{page}.html"
        Path(filename).write_bytes(response.body)
```

**This happens because parse() is Scrapy’s default callback method, which is called for requests without an explicitly assigned callback.**

## Best way to learn how extract data

```bash
scrapy shell <url>
```

```bash
>>> response.css('title')
[<Selector xpath='descendant-or-self::title' data='<title>Quotes to Scrape</title>'>]
```

```bash
>>> response.css('title').getall()
['<title>Quotes to Scrape</title>']
```

```bash
>>> response.css('title::text').get()
'Quotes to Scrape'
```

**Its better to use getall() instead of get() because it will return a list of all the elements that match the query.**
    
### another way is to use re() method:

```bash
>>> response.css('title::text').re(r'Quotes.*')
['Quotes to Scrape']
>>> response.css('title::text').re(r'Q\w+')
['Quotes']
>>> response.css('title::text').re(r'(\w+) to (\w+)')
['Quotes', 'Scrape']
```

[How to use Developper tool to find css selector](https://docs.scrapy.org/en/latest/topics/developer-tools.html#topics-developer-tools)
# Need to read this previous link 

# Need to continue at< XPath: a brief intro > : https://docs.scrapy.org/en/latest/intro/tutorial.html