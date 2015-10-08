ttrss_plugin-feediron
=======================

This is a plugin for Tiny Tiny RSS (tt-rss). It allows you to replace an article's contents by the contents of an element on the linked URL's page, i.e. create a "full feed".


Installation
------------

Checkout the directory into your plugins folder like this (from tt-RSS root directory):

```sh
$ cd /var/www/ttrss
$ git clone git://github.com/m42e/ttrss_plugin-feediron.git plugins/feediron
```

Then enable the plugin in preferences.


Configuration
-------------

The configuration is done in JSON format. In the preferences, you'll find a new tab called *FeedIron*. Use the large field to enter/modify the configuration data and click the **Save** button to store it.
You can enter an URL in the field below and click **Test** to get a preview what the result will be if the filter is applied to the url. *Note:* Save before you test :).

You can also select from available predefined configurations. Just select one, click **Add** and save the configuration.

A configuration looks like this:

```json
{

"heise.de": {
	"name": "Heise Newsticker",
	"url": "http://heise.de/ticker/",
    "type": "xpath",
    "xpath": "div[@class='meldung_wrapper']",
    "force_charset": "utf-8"
},
"berlin.de/polizei": {
    "type": "xpath",
    "xpath": "div[@class='bacontent']"
},
"n24.de": {
    "type": "readability",
},
"www.dorkly.com": {
    "type": "xpath",
    "multipage": {
        "xpath": "a[contains(@data-ga-category,'Pagination') and text() = 'Next']",
        "append": true,
        "recursive": true
    },
    "xpath": "div[contains(@class,'post-content')]"
},
"golem0Bde0C": {
    "type": "xpath",
    "xpath": "article",
	"multipage": {
		"xpath": "ol/li/a[contains(@id, 'atoc_')]",
		"append": true
	},
    "reformat": [
        {
            "type": "regex",
            "pattern": "/(?:[a-z0-9A-Z\\/.\\:]*?)golem0Bde0C(.*)0Erss0Bhtml\\/story01.htm/",
            "replace": "http://www.golem.de/$1.html"
        },
        {
            "type": "replace",
            "search": [
                "0A",
                "0C",
                "0B",
                "0E"
            ],
            "replace": [
                "0",
                "/",
                ".",
                "-"
            ]
        }
    ]
},
"oatmeal": {
    "type": "xpath",
    "xpath": "div[@id='comic']"
},
"blog.beetlebum.de": {
    "type": "xpath",
    "xpath": "div[@class='entry-content']",
    "cleanup": [ "header", "footer" ]
},
"sueddeutsche.de": {
    "type": "xpath",
    "xpath": [
        "h2/strong",
        "section[contains(@class,'authors')]"
    ],
    "join_element": "<p>",
    "cleanup": [
        "script"
    ]
},
"www.spiegel.de": {
    "type": "split",
    "steps": [
        {
            "after": "/article-section clearfix\"\\W*>/",
            "before": "/<div\\W*class=\"module-box home-link-box/"
        },
        {
            "before": "/<div\\W*class=\"btwBarInArticles/"
        }
    ],
    "cleanup" : [ "~<script([^<]|<(?!/script))*</script>~msi" ],
    "force_unicode": true
},
"debug": false

}
```

The *array key* is part of the URL of the article links(!). You'll notice the `golem0Bde0C` in the last entry: That's because all their articles link to something like `http://rss.feedsportal.com/c/33374/f/578068/p/1/s/3f6db44e/l/0L0Sgolem0Bde0Cnews0Cthis0Eis0Ean0Eexample0A10Erss0Bhtml/story01.htm` and to have the plugin match that URL and not interfere with other feeds using *feedsportal.com*, I used the part `golem0Bde0C`.

**type** has to be `xpath`, `split` or `readability`.

### xpath
The **xpath** value is the actual Xpath-element to fetch from the linked page. Omit the leading `//` - they will get prepended automatically.
Every xpath can also be an object consisting of an `xpath` element and an `index` element. So you can get the third div by specify: `{ "xpath":"div", "index":3 }`.

There is an additional option **cleanup** available. Its an array of Xpath-elements (relative to the fetched node) to remove from the fetched node. Omit the leading `//` - they will get prepended automatically.

You can also specify multiple xpaths, they are evaluated in the order they are given in the array. They are simply put together. if you specify a `join_element` this is placed between the elements.
There are also `start_element` and `end_element` which are prepended to the concatenated parts.

### split
The **steps** value is an array of actions performed in the given order. If **after** is given the content will be split using the value and the second half is used, if **before** the first half is used. preg_split is used for this action.

There is an additional option **cleanup** available. Its an array of regex that are removed using preg_replace.

### readability
This option makes use of [php-readability]( https://github.com/j0k3r/php-readability ) which is a fork of the [original](http://code.fivefilters.org/php-readability). All the extraction is performed within this module.

## multipage
This option indicates that the article is split into two or more pages (eventually). FeedIron can combine all the parts into the content of the article.
You have to specify a ```xpath``` which identifies the links (&lt;a&gt;) to the pages. If ```append``` is false, only the links are used and the original link is ignored else the links found using the xpath expression are added to the original page link.
You can also specify the ```recursive``` option to parse every following page for more links. To avoid infinite loops
the fetching stops if an url is added twice.

### General options
* **debug** You can activate debugging informations.  (At the moment there are not that much debug informations to be activated), this option must be places at the same level as the site configs.
* **force_charset** allows to override automatic charset detection. If it is omitted, the charset will be parsed from the HTTP headers or loadHTML() will decide on its own.  
* **reformat** is an array of formating rules for the **url** of the full article. The rules are applied before the full article is fetched. There are two possible types: **regex** and **replace**. 
  * **regex** takes a regex in an option called **pattern** and the replacement in **replace**. For details see [preg_replace](http://www.php.net/manual/de/function.preg-replace.php) in the PHP documentation. 
  * **replace** uses the PHP function str_replace, which takes either a string or an array as search and replace value.  
* **modify** is the same as described above but for the content. It is applied after the split/xpath selection.

If you get an error about "Invalid JSON!", you can use [JSONLint](http://jsonlint.com/) to locate the erroneous part. There is a basic error detection implemented.

XPath
-----

### Tools

To test your XPath expressions, you can use these Chrome extensions:

* [XPath Helper](https://chrome.google.com/webstore/detail/xpath-helper/hgimnogjllphhhkhlmebbmlgjoejdpjl)
* [xPath Viewer](https://chrome.google.com/webstore/detail/xpath-viewer/oemacabgcknpcikelclomjajcdpbilpf)
* [xpathOnClick](https://chrome.google.com/webstore/detail/xpathonclick/ikbfbhbdjpjnalaooidkdbgjknhghhbo)


### Examples

Some XPath expressions you could need (the `//` is automatically prepended and must be omitted in the FeedMod configuration):

##### HTML5 &lt;article&gt; tag

```html
<article>…article…</article>
```

```xslt
//article
```

##### DIV inside DIV

```html
<div id="content"><div class="box_content">…article…</div></div>`
```

```xslt
//div[@id='content']/div[@class='box_content']
```

##### Multiple classes

```html
<div class="post-body entry-content xh-highlight">…article…</div>
```

```xslt
//div[starts-with(@class ,'post-body')]
```
or
```xslt
//div[contains(@class, 'entry-content')]
```

##### Image tag
```html
<a><img src='test.png' /></a>
```
```xslt
img/..
```

## Todo
* ~~Make a recipe base here~~
* ~~Add a simple mechanism to add recipes to the configuration~~
* ~~Add a simple mechanism to export recipes~~
* ~~spend some time on a nicer interface design~~
* split init.php in multiple classes
* Make test output collapsable


## Special Thanks
Thanks to [mbirth](https://github.com/mbirth) who wrote [af_feedmod](https://github.com/mbirth/ttrss_plugin-af_feedmod) who gave me a starting base.

