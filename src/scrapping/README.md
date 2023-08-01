# Scrapping Part

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

### another way is to use xpath() method:
You need to go your DevTools and right click on the element you want to extract and select Copy > Copy XPath.

```bash
>>> response.xpath("/html/body/div/div[1]/div[1]/h1/a").getall()
['<a href="/" style="text-decoration: none">Quotes to Scrape</a>']
```
**if you add /text() at the end you will get only the text**

```bash
>>> response.xpath("/html/body/div/div[1]/div[1]/h1/a/text()").get()
'Quotes to Scrape'
```

```bash
>>> response.xpath("//title").get()
'<title>Quotes to Scrape</title>'
```

### Repetitive patterns

```html
<div class="quote" itemscope="" itemtype="http://schema.org/CreativeWork">
  <span class="text" itemprop="text">
    “The world as we have created it is a process of our thinking. It cannot be changed without changing our thinking.”
  </span>
  <span>(...)</span>
  <div class="tags">(...)</div>
</div>
```
**In this example we have many span that reapeat with the same class "text", becarefull its simple quote inside xpath!!**
```bash
response.xpath('//span[has-class("text")]/text()').getall()
['“The world as we have created it is a process of our thinking. It cannot be changed without changing our thinking.”',
'“It is our choices, Harry, that show what we truly are, far more than our abilities.”',
'“There are only two ways to live your life. One is as though nothing is a miracle. The other is as though everything is a miracle.”',
...]
```

```python
import scrapy
import json


class QuoteSpider(scrapy.Spider):
    name = "quote"
    allowed_domains = ["quotes.toscrape.com"]
    page = 1
    start_urls = ["https://quotes.toscrape.com/api/quotes?page=1"]

    def parse(self, response):
        data = json.loads(response.text)
        for quote in data["quotes"]:
            yield {"quote": quote["text"]}
        if data["has_next"]:
            self.page += 1
            url = f"https://quotes.toscrape.com/api/quotes?page={self.page}"
            yield scrapy.Request(url=url, callback=self.parse)
```

[How to use Developper tool to find css selector](https://docs.scrapy.org/en/latest/topics/developer-tools.html#topics-developer-tools)


```html
<div class="quote">
    <span class="text">“The world as we have created it is a process of our
    thinking. It cannot be changed without changing our thinking.”</span>
    <span>
        by <small class="author">Albert Einstein</small>
        <a href="/author/Albert-Einstein">(about)</a>
    </span>
    <div class="tags">
        Tags:
        <a class="tag" href="/tag/change/page/1/">change</a>
        <a class="tag" href="/tag/deep-thoughts/page/1/">deep-thoughts</a>
        <a class="tag" href="/tag/thinking/page/1/">thinking</a>
        <a class="tag" href="/tag/world/page/1/">world</a>
    </div>
</div>
```

We get a list of selectors for the quote HTML elements with:
```bash
>>>> response.css("div.quote")
[<Selector query="descendant-or-self::div[@class and contains(concat(' ', normalize-space(@class), ' '), ' quote ')]" data='<div class="quote" itemscope itemtype...'>,
<Selector query="descendant-or-self::div[@class and contains(concat(' ', normalize-space(@class), ' '), ' quote ')]" data='<div class="quote" itemscope itemtype...'>,
...]

>>>> response.css("div.quote")[0]

```
Now, let’s extract ``text``, `author` and the `tags` from that quote using the quote object we just created:

```bash
>>>> text = quote.css("span.text::text").get()
>>>> author = quote.css("small.author::text").get()
>>>> tags = quote.css("div.tags a.tag::text").getall()
``````

```python
import scrapy


class QuotesSpider(scrapy.Spider):
    name = "quotes"
    start_urls = [
        "https://quotes.toscrape.com/page/1/",
        "https://quotes.toscrape.com/page/2/",
    ]

    def parse(self, response):
        for quote in response.css("div.quote"):
            yield {
                "text": quote.css("span.text::text").get(),
                "author": quote.css("small.author::text").get(),
                "tags": quote.css("div.tags a.tag::text").getall(),
            }
```

