# WEB-SCRAPING


What is web scraping?
Web scraping consists in gathering data available on websites. This can be done manually by a human user or by a bot. The latter can of course gather data much faster than a human user and that is why we are going to focus on this. Is it therefore technically possible to collect all the data of a website in a matter of minutes this kind of bot. The legality of this practice is not well defined however. Websites usually describe in their terms of use and in their robots.txt file if they allow scrapers or not.
How does it work?
Web scrapers gather website data in the same way a human would do it: the scraper goes onto a web page of the website, gets the relevant data, and move forward to the next web page. Every website has a different structure, that is why web scrapers are usually built to explore one website. The two important issues that arise during the implementation of a web scraper are the following:
What is the structure of the web pages that contain relevant data?
How can we get to those web pages?
In order to answer those questions, we need to understand a little how websites work. Websites are created using HTML (Hypertext Markup Language), along with CSS (Cascading Style Sheets) and JavaScript. HTML elements are separated by tags and they directly introduce content to the web page. Here is what a basic HTML document looks like:
Image for post
Basic HTML page
We can see that the content of the first heading is contained between the ‘h1’ tags. The first paragraph is contained between the ‘p’ tags. On a real website, we need to find out between which tags the relevant data is and tell it to our scraper. We also need to specify which links should be explored and where they can be found among the HTML file. With all this information, our scraper should be able to gather the required data.
What tools are we going to use?
In this tutorial we are going to use the Python modules requests and BeautifulSoup.
Requests will allow us to send HTTP requests to get the HTML files.
Link to requests documentation: http://docs.python-requests.org/en/master/
Image for post
Requests module
BeautifulSoup will be used to parse the HTML files. It is one of the most used library for web scraping. Its is quite simple to use and has many features that help gathering websites data efficiently.
Link to BeautifulSoup documentation: https://www.crummy.com/software/BeautifulSoup/bs4/doc/
Image for post
BeautifulSoup module
Prerequisites
python 2.7
requests
beautifulsoup4
pandas
Objective
We want to scrape the data of an online book store: http://books.toscrape.com/
This website is fictional so we can scrape it as much as we want.
In this tutorial we will be gathering the following information about all the products of the website:
book title
price
availability
image
category
rating
Warm-up: get the content of the main page
First let’s use the requests module to get the HTML of the website’s main page.

u'<!DOCTYPE html>\n<!--[if lt IE 7]>      <html lang="en-us" class="no-js lt-ie9 lt-ie8 lt-ie7"> <![endif]-->\n<!--[if IE 7]>         <html lang="en-us" class="no-js lt-ie9 lt-ie8"> <![endif]-->\n<!--[if IE 8]>         <html lang="en-us" class="no-js lt-ie9"> <![endif]-->\n<!--[if gt IE 8]><!--> <html lang="en-us" class="no-js"> <!--<![endif]-->\n    <head>\n        <title>\n    All products | Books to Scrape - Sandbox\n</title>\n\n        <meta http-equiv="content-type" content="text/html; charset=UTF-8" />\n        <meta name="created" content="24th Jun 2016 09:29" />\n        <meta name="description" content="" />\n        <meta name="viewport" content="width=device-width" />\n        <meta name="robots" content="NOARCHIVE,NOCACHE" />\n\n        <!-- Le HTML5 shim, for IE6-8 support of HTML elements -->\n        <!--[if lt IE 9]>\n        <script src="//html5shim.googlecode.com/svn/trunk/html5.js"></script>\n        <![endif]-->\n\n        \n            <link rel="shortcut icon" href="static/oscar/favicon.'
The result is quite messy! Let’s make this more readable:

<html class="no-js" lang="en-us">
 <!--<![endif]-->
 <head>
  <title>
   All products | Books to Scrape - Sandbox
  </title>
  <meta content="text/html; charset=utf-8" http-equiv="content-type"/>
  <meta content="24th Jun 2016 09:29" name="created"/>
  <meta content="" name="description"/>
  <meta content="width=device-width" name="viewport"/>
  <meta content="NOARCHIVE,NOCACHE" name="robots"/>
  <!-- Le HTML5 shim, for IE6-8 support of HTML elements -->
  <!--[if lt IE 9]>
        <script src="//html5shim.googlecode.com/svn/trunk/html5.js"></script>
        <![endif]-->
  <link href="static/oscar/favicon.ico" rel="shortcut icon"/>
  <link href="static/oscar/css/styles.css" rel="stylesheet" type="tex
The function prettify() makes the HTML more readable. However we will not use this directly to explore where the relevant data is.
Let’s define a function to request and parse a HTML web page as we will need this a lot during this tutorial:

Find book URLs on the main page
Now let’s start to dive deeper into the subject. In order to get the book data, we need to be able to access their product page. The first step consist in finding the URL of every book product page.
In your browser, go onto the website main page, right-click on the name of a product and click on inspect. This will show you the HTML part of the web page corresponding to this element. Congratulations, you have found the first book link!
Note the structure of the HTML code:
Image for post
Inspecting HTML code
You can try this with every other product on the page: the structure is always the same. The link of the product corresponds to the ‘href’ attribute of the ‘a’ tag. This one belongs to an ‘article’ tag with the a class value ‘product_pod’. This seems to be a reliable source to spot product URLs.
BeautifulSoup enables us to find those special ‘article’ tags. We can wall the find() function in order to find the first occurence of this tag in the HTML:

<article class="product_pod">\n<div class="image_container">\n<a href="catalogue/a-light-in-the-attic_1000/index.html"><img alt="A Light in the Attic" class="thumbnail" src="media/cache/2c/da/2cdad67c44b002e7ead0cc35693c0e8b.jpg"/></a>\n</div>\n<p class="star-rating Three">\n<i class="icon-star"></i>\n<i class="icon-star"></i>\n<i class="icon-star"></i>\n<i class="icon-star"></i>\n<i class="icon-star"></i>\n</p>\n<h3><a href="catalogue/a-light-in-the-attic_1000/index.html" title="A Light in the Attic">A Light in the ...</a></h3>\n<div class="product_price">\n<p class="price_color">\xc2\xa351.77</p>\n<p class="instock availability">\n<i class="icon-ok"></i>\n    \n        In stock\n    \n</p>\n<form>\n<button class="btn btn-primary btn-block" data-loading-text="Adding..." type="submit">Add to basket</button>\n</form>\n</div>\n</article>
We still have too much information.
Let’s dive deeper in the tree by adding the other child tags:

<a href="catalogue/a-light-in-the-attic_1000/index.html"><img alt="A Light in the Attic" class="thumbnail" src="media/cache/2c/da/2cdad67c44b002e7ead0cc35693c0e8b.jpg"/></a>
Much better! But we only need the URL contained in the ‘href’ value.
We can get this by adding .get(“href”) to the previous instruction:

u'catalogue/a-light-in-the-attic_1000/index.html'
Ok, we managed to get our first product URL with BeautifulSoup.
Now let’s gather all the products URLs on the main web page at once using the findAll() function:

20 fetched products URLs
One example:
u'catalogue/a-light-in-the-attic_1000/index.html'
This function is very handy for finding all the values at once, but you have to check that all the information collected is relevant. Sometimes one same tag can contain completely different data. That is why it is important to be as specific as possible when choosing the tags. Here we decided to rely on the tag ‘article’ with the ‘product_pod’ class because this seems to be a very specific tag and it is unlikely that we can find data other than product data in it.
The previous URLs correspond to their relative path from the main page. In order to make them complete, we just need to add before them the URL of the main page: http://books.toscrape.com/index.html (after removing the index.html part).
Now let’s use this to define a function to retrieve book links on any given page of the website:

Find book categories URLs on the main page
Now let’s try retrieving the URLs corresponding the different product categories:
Image for post
Inspecting HTML code
By inspecting, we can see that they follow the same URL pattern: ‘catalogue/category/books’.
We can tell BeautifulSoup to match the URLs that contain this pattern in order to retrieve easily the categories URLs:

50 fetched categories URLs
Some examples:[u'http://books.toscrape.com/index.htmlcatalogue/category/books/travel_2/index.html',
 u'http://books.toscrape.com/index.htmlcatalogue/category/books/mystery_3/index.html',
 u'http://books.toscrape.com/index.htmlcatalogue/category/books/historical-fiction_4/index.html',
 u'http://books.toscrape.com/index.htmlcatalogue/category/books/sequential-art_5/index.html',
 u'http://books.toscrape.com/index.htmlcatalogue/category/books/classics_6/index.html']
We managed to retrieve the 50 categories URLs successfully!
