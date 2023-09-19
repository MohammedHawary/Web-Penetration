# DOM-based XSS

DOM-based XSS vulnerabilities usually arise when JavaScript takes data from an attacker-controllable source, such as the URL, and passes it to a sink that supports dynamic code execution, such as `eval()` or `innerHTML`. This enables attackers to execute malicious JavaScript, which typically allows them to hijack other users' accounts. 

To perform a DOM-based XSS attack, you insert data into a source, triggering the execution of arbitrary JavaScript.

The main source of DOM XSS is usually the URL, accessed via `window.location`. Attackers craft links to send victims to vulnerable pages with payloads in the URL's query string, fragment, or, in some cases, the path (e.g., targeting 404 pages or PHP-powered websites).

## How to test for DOM-based cross-site scripting

You need to work through each available source in turn, and test each one individually. 

#### Testing HTML sinksb

To check for DOM XSS in an HTML sink, insert a random alphanumeric string into the source (e.g., `location.search`) and inspect the HTML using developer tools to locate the string. Note that the browser's `View source` option is ineffective for DOM XSS testing as it doesn't consider HTML changes made by JavaScript. In Chrome's developer tools, use `Control+F` (or `Command+F` on MacOS) to search for your string in the DOM.

**Browsers vary in URL-encoding behavior**: Chrome, Firefox, and Safari encode `location.search` and `location.hash`, but IE11 and Microsoft Edge (pre-Chromium) don't. If your data is URL-encoded before processing, XSS attacks are less likely to succeed.

#### Testing JavaScript execution sinks

**Testing JavaScript execution sinks** for DOM-based XSS is challenging. Unlike other sinks, your input may not directly show up in the DOM, making it difficult to locate. Instead, you must employ the **JavaScript debugger** to trace how your input is utilized by a sink.

To identify potential sources like "location," search through the page's JavaScript code in Chrome's developer tools using Control+Shift+F (or Command+Alt+F on MacOS).

After locating where the source is read, use the JavaScript debugger to set a breakpoint and follow how the source's value is used. It may be assigned to other variables, in which case, use the search function to trace these variables and check if they are sent to a sink. When you identify a sink receiving data from the source, inspect the value by hovering over the variable before it reaches the sink using the debugger. Just like with HTML sinks, refine your input to test for a successful XSS attack.

#### Testing for DOM XSS using DOM Invader

Identifying and exploiting DOM XSS in the wild can be a tedious process, often requiring you to manually trawl through complex, minified JavaScript. If you use Burp's browser, however, you can take advantage of its built-in DOM Invader extension, which does a lot of the hard work for you. 

### Exploiting DOM XSS with different sources and sinks

In essence, a website is vulnerable to DOM-based cross-site scripting if data can flow from source to sink through an executable path. However, the specifics of sources and sinks, along with data processing and validation in scripts, influence exploitability and required techniques.

 The `document.write` sink works with `script` elements, so you can use a simple payload, such as the one below: 

```js
document.write('... <script>alert(document.domain)</script> ...');
```

> **NOTE**
> 
> In certain cases, the content written to `document.write` may contain surrounding context that must be considered in your exploit. This might involve **closing existing elements** before injecting your JavaScript payload.

The `innerHTML` sink on modern browsers doesn't support script elements or trigger `svg` `onload` events. Therefore, you must utilize alternative elements like `img` or `iframe` along with event handlers such as `onload` and `onerror`. For instance:

```js
element.innerHTML='... <img src=1 onerror=alert(document.domain)> ...'
```

#### EX1: DOM XSS in document.write sink using source location.search

First, I entered unique value to make search about it easyer `asdfghjkl123` 

![Screenshot from 2023-09-12 05-12-03](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/852286d6-5398-44d4-9892-8d2d199021f5)
Then open dev tool and open `js` file of this page

![Screenshot from 2023-09-12 05-14-22](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/2f9e6025-47eb-4df0-9e02-8d5a5ad43d66)
Add **break point** and press `f11` to go step by step

![Screenshot from 2023-09-12 05-17-54](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/6519de33-8901-45db-b109-8f6a1009683a)
As we can see there are variable called query stored our input then press `f11` again

![Screenshot from 2023-09-12 05-18-53](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/3411ceab-575f-4471-b229-7d6723328067)

our value passed to `document.write` function write this html

```js
document.write('<img src="resources/images/tracker.gif?searchTerm='+query+'">');
```

![Screenshot from 2023-09-12 05-20-04](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/8c483d9f-76c9-44db-8f2a-13aa80471b48)

And You can saw you value were reflected with search in `inspector`

![Screenshot from 2023-09-13 04-34-16](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/1ecc86db-ad6b-44e3-b9ec-e60c4c0a6a9a)

And this is the html code after `document.write` function

```html
<img src="/resources/images/tracker.gif?searchTerms=asdfghjkl123">
```

Then let's create our payload with this info

```js
"onload="alert()
```

![Screenshot from 2023-09-12 05-56-44](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/c6ff58b6-35c0-4b0d-b0bc-34ab513c9d2c)

It's work

![Screenshot from 2023-09-12 05-57-00](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/ad5a1b35-5802-476d-9dda-12497aafeafb)
![Screenshot from 2023-09-12 05-57-06](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/d9d090e2-e4fe-4af8-936e-43617e03ec2b)

> [**Python Script for this lab**](https://github.com/MohammedHawary/Solve-Portswigger-Labs-With_py/blob/main/XSS/DOM_XSS_in_document.write_sink_using_source_location.search.py)

#### EX2: DOM XSS in document.write sink using source location.search inside a select element

First, when you press the `check stock` it's print the value of product 

![Screenshot from 2023-09-13 05-05-12](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/b2f743b1-68f1-46b3-ba11-0a5be1d919e5)
In Burp there are two parameters sent data `productId` and `storeId`

![Screenshot from 2023-09-13 05-05-34](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/bbf42499-7301-4524-a918-980757d3d507)

In page source, there are a JS code dealing with previous parameters and rewrite it in HTML tags with `document.write`

![Screenshot from 2023-09-13 04-44-44](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/eb84e13f-9e0c-4026-846d-8f3dc892eb81)
And If you add `&storeId=aaaaaaaaaaaaa` in URL, our value wrote in the list of stocks

Let's understand the JS code to know what happened

![Screenshot from 2023-09-13 04-48-11](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/64d058ca-a2da-4ee1-a04a-aa69d0285a86)
Our payload is written between the `<option>` tag and the `<option>` tag inside `<select>`. Then let's create our payload with the closing tag `</option>` and `</select>` and then write our payload:

```html
</option></select><img src=1 onerror="alert()">
```

![Screenshot from 2023-09-13 04-53-54](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/66401b6d-8787-49f9-9f57-3457e7edbf65)
It's Work

![Screenshot from 2023-09-13 04-54-27](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/9bf4dc16-c8bc-4220-9ad3-3b04b5c8d979)
![Screenshot from 2023-09-13 04-54-34](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/04c83500-e367-469c-b36a-14c565868320)

![Screenshot from 2023-09-13 04-55-32](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/a12304a6-db64-4821-8c7b-592a952d8a31)

> [**Python Script for this lab**](https://github.com/MohammedHawary/Solve-Portswigger-Labs-With_py/blob/main/XSS/DOM_XSS_in_document.write_sink_using_source_location.search.py)

#### EX3: DOM XSS in innerHTML sink using source location.search

First, I entered unique value to make search about it easyer `asdfghjkl123`

![Screenshot from 2023-09-13 05-19-47](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/69be472e-c2d6-4c03-9e1a-d58765e98c1b)
It's written betwwen `<span>` tag , and there are js code in page source dealing with our input

![Screenshot from 2023-09-13 05-37-37](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/f4ad3058-5978-4e7a-bc13-688160b4f4a5)

To know the value of 

```js
document.getElementById('searchMessage').innerHTML
```

Copy it and past it in console

![Screenshot from 2023-09-13 05-22-37](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/958f51a4-cd43-466c-bc04-e6a953dbd5ab)
As we can see the value of this code is our input, Then let's do it again with `query` 

![Screenshot from 2023-09-13 05-22-50](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/8dbedc86-ed9f-46b7-b321-74e3ef391539)
Copy it and past it in console to know his value

![Screenshot from 2023-09-13 05-23-07](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/c3e71777-350f-49e6-a215-53183e869451)
It's also equal our input

![Screenshot from 2023-09-13 05-23-14](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/a023da21-8ed7-45fb-a4ac-62e9e7fb9597)

And our input is written with innerHTML.

And the `innerHTML` is support to write html code inside it and the html code will work 

![Screenshot from 2023-09-13 05-51-05](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/259f8907-475b-4957-8f82-aab7a7577474)
Let's create our payload with this info:

```html
<img src=1 onerror="alert('DOM XSS')">
```

![Screenshot from 2023-09-13 05-23-47](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/a39c71b5-56bc-47fc-8faf-50b1173174d6)
It's Work

![Screenshot from 2023-09-13 05-23-56](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/20d55d2c-55ab-4b24-ac6b-32e6c6573f77)
![Screenshot from 2023-09-13 05-24-01](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/988a77be-9237-404e-80f9-ee1a341361eb)

> [**Python Script for this lab**](https://github.com/MohammedHawary/Solve-Portswigger-Labs-With_py/blob/main/XSS/DOM_XSS_in_innerHTML_sink_using_source_location.search.py)

## Sources and sinks in third-party dependencies

Modern web apps often rely on third-party libraries and frameworks, offering added functions. Yet, some can be sources of DOM XSS.

### DOM XSS in jQuery

When using JavaScript libraries like **jQuery**, be cautious of sinks that modify DOM elements. For instance, jQuery's `attr()` function can **alter attributes of DOM elements**. If data from a user-controlled source, like the URL, is passed to `attr()`, it can potentially lead to XSS. For instance, consider this JavaScript that modifies an anchor element's `href` attribute using URL data:

```js
$(function() {
    $('#backLink').attr("href",(new URLSearchParams(window.location.search)).get('returnUrl'));
});
```

Exploiting this involves altering the URL to inject a malicious JavaScript URL into the `location.search` source. Once the page applies this malicious URL to the back link's href, clicking on the back link will execute it.

```js
?returnUrl=javascript:alert(document.domain)
```

Watch for jQuery's `$()` selector as another potential source of DOM XSS. In the past, it posed a risk when combined with `location.hash` for **animations or auto-scrolling**, often implemented through a vulnerable `hashchange` event handler like this:

```js
$(window).on('hashchange', function() {
    var element = $(location.hash);
    element[0].scrollIntoView();
});
```

Since the **hash is user-controlled**, it could be exploited to inject an XSS vector into the `$()` selector. Recent jQuery **versions fixed this** by blocking HTML injection for input starting with a `#` character, but older vulnerable code may still exist.

To exploit this classic vulnerability, you must trigger a `hashchange` event without user interaction. A straightforward method is to deliver your exploit through an `iframe`.

```html
<iframe src="https://vulnerable-website.com#" onload="this.src+='<img src=1 onerror=alert(1)>'">
```

In this example, the `src` attribute points to the vulnerable page with an empty hash. When the `iframe` loads, an XSS vector is added to the hash, triggering the `hashchange` event.

> **NOTE**
> 
> Even newer versions of jQuery can still be vulnerable via the `$()` selector sink, provided you have full control over its input from a source that doesn't require a `#` prefix.

#### EX1: DOM XSS in jQuery anchor href attribute sink using location.search source

First, there is a feedback page and when I write all the fields with a unique value `asdfghjkl123` to make it easier to search for them, they are not reflected. Let's see the page source.

![Screenshot from 2023-09-17 04-49-11](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/b634c5b3-5d88-49ae-812e-55ab533f200d)
mmmm, There are jQuery code 

```js
$(function() {
    $('#backLink').attr("href", (new URLSearchParams(window.location.search)).get('returnPath'));
});
```

![Screenshot from 2023-09-17 04-57-55](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/267800b9-21cd-446a-960b-868697b2ffc3)
If you can't understand this code, you can make ChatGPT help you to understand it![Screenshot from 20230917 051346](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/0f64e959-13d5-4d17-bae9-8b1d0b11cd32)

Then let's search about `backLink` and write in URL this parameter with value test 

`?returnPath=test`

![Screenshot from 2023-09-17 04-58-56](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/5a1f602c-1e7b-464f-804f-2f64bd76387e)
As we can see our value `test` is writen inside `href` attribute

![Screenshot from 2023-09-17 04-59-30](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/31e27ad4-8416-4ce4-8dd5-d0bf92ae7571)
Let's create our XSS payload with this info:

```js
?/returPath=javascript:alert()
```

![Screenshot from 2023-09-17 05-00-31](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/41654bfb-bcfb-432e-9ecd-72f796233fa2)
Lab solved 

![Screenshot from 2023-09-17 05-00-44](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/8e9be907-adde-44a5-a463-e4a917b41953)
If you press in `< Back` link, our XSS payload will execute![Screenshot from 2023-09-17 05-00-50](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/6b9e193c-f787-4e70-b46c-4f599915ab24)

> [**Python Script for this lab**](https://github.com/MohammedHawary/Solve-Portswigger-Labs-With_py/blob/main/XSS/DOM_XSS_in_jQuery_anchor_href_attribute_sink_using_location.search_source.py)

#### EX2: DOM XSS in jQuery selector sink using a hashchange event

First, I saw this page that didn't have anything on it that caught my attention, let's see page source![Screenshot from 2023-09-17 08-47-55](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/299cf587-6d96-4b8f-950e-f73bfb720d9d)

mmmm, There are jQuery code

```js
$(window).on('hashchange', function(){
    var post = $('section.blog-list h2:contains(' + decodeURIComponent(window.location.hash.slice(1)) + ')');
    if (post) post.get(0).scrollIntoView();
});
```

![Screenshot from 2023-09-17 06-48-01](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/35537c2d-0346-4ca4-8173-a6dd30ab1c66)If you can't understand this code, you can make ChatGPT help you to understand it

![Screenshot from 2023-09-17 08-03-14](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/4bfb9938-ae04-409d-94fa-0136b3bcf67f)
Let's search about **section** `blog-list`

![Screenshot from 2023-09-17 08-04-44](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/c34bda76-896a-4a96-9b12-313b79cdb0cd)
This section contains the `h2` that's use it to scrolling to it![Screenshot from 2023-09-17 08-05-17](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/53fe2d75-191e-4e15-b325-2a0f00c9d53d)

Then Let's copy this line from jQuery code

![Screenshot from 2023-09-17 08-03-54](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/17644b5b-291f-4640-9b1d-55585ff7ab9e)

And write any `h2` value from section and add this line in url 

```js
#h2_value
    // exmaple
#Finding Inspirati
```

And after this paste the line that we copied it in console with some functions

```js
$('section.blog-list h2:contains(' + decodeURIComponent(window.location.hash.slice(1)) + ')').get(0).innerHTML
```

![Screenshot from 2023-09-17 08-06-43](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/379de0e5-04b9-44c6-b88d-b441fed2078a)
We get the hashed value that we written it in url

then let's cteate our payload with this info

```html
<iframe src="https://web-security-academy.net/#" onload="this.src+='<img src=1 onerror=print()>';">
```

Test it

![Screenshot from 2023-09-17 08-11-38](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/5bcf6f0f-da2b-4de5-869c-d9452f9ecc3c)

Worked

![Screenshot from 2023-09-17 08-18-12](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/fa46ce9b-9109-42ee-bfa1-5c90054357c2)
Send it to victim

![Screenshot from 2023-09-17 08-19-33](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/86eb46e2-6048-4af7-8ac8-e8ad389412ce)
Lab solved

![Screenshot from 2023-09-17 08-20-37](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/0321fa41-9ead-4902-901c-e108b6c216f1)

> [**Python Script for this lab**](https://github.com/MohammedHawary/Solve-Portswigger-Labs-With_py/blob/main/XSS/DOM_XSS_in_jQuery_selector_sink_using_a_hashchange_event.py)

### DOM XSS in AngularJS

First, I write this sample XSS code to know if there are any changes on it, and it is easy to search about it `<img src=1 onerror="alert('asdfghjk123')">`

![Screenshot from 2023-09-18 10-27-50](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/3e6bcfa1-3242-45d0-bae9-e8108106be86)
Most payload signs replaced

![Screenshot from 2023-09-18 10-28-05](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/c639585b-fd50-4df7-bd52-844bbfc427b1)

But in the page source the `<body>` tag contain a `ng-app` attribute it also known as an **AngularJS directive** .When a directive is added to the HTML code, you can execute JavaScript expressions within double curly braces `{{}}` Then Let's try this :

```js
{{7*7}}
```

![Screenshot from 20230918 102338](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/a6950fae-3d7e-4a87-a378-9419c58b851e)

It's work

![Screenshot from 2023-09-18 10-28-26](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/03e7415b-a2e5-4e9f-8bfc-0875d6403fe2)
Let's try use alert() with this payload

```js
{{alert()}}
```

![Screenshot from 2023-09-18 10-33-18](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/ddf0e6a2-2b7d-4567-a1ff-dd30cd4a8f5a)
Didn't work

![Screenshot from 2023-09-18 10-33-26](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/d695f6be-58fd-4bce-8f02-c27135d75a90)

Let's ask ChatGPT about `ng-app` to know more info about it

![Screenshot from 2023-09-18 10-34-07](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/321597c8-c08d-4dfa-a222-b25afb1bb191)

After search i found this articl discuse this problem

![Screenshot from 2023-09-17 09-48-33](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/7c879fc3-9ccc-47c8-8557-a70e31bab98b)

![Screenshot from 2023-09-17 09-46-38](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/2650b5b7-ab6b-4bc2-80f1-b867529f521f)
i used this payload:

```js
{{this.constructor.constructor('alert("DOM XSS")')()}}
```

![Screenshot from 2023-09-17 09-47-13](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/22402281-2fda-40fa-9034-4fc9038a7429)
It's work

![Screenshot from 2023-09-17 09-47-28](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/7fc6694e-9ae0-4b2f-ab31-34eb6241352b)

> [**Python Script for this lab**](https://github.com/MohammedHawary/Solve-Portswigger-Labs-With_py/blob/main/XSS/DOMd_XSSd_ind_AngularJSd_expressiond_withd_angled_bracketsd_andd_doubled_quotesd_HTML_encoded.py)

### DOM XSS combined with reflected and stored data

Some DOM vulnerabilities stay within a single page. When a script takes data from the URL and puts it into a risky location, it's a client-side issue. However, these vulnerabilities aren't limited to browser-exposed data; websites often include URL parameters in their server's HTML response. This can lead to reflected DOM XSS vulnerabilities, where the server processes request data and echoes it in the response, possibly into a JavaScript string or a DOM element. A script then mishandles this data, sending it to a dangerous location.

```js
eval('var data = "reflected string"');
```

Websites can store and display server-stored data in a stored DOM XSS scenario, where the server takes data from an initial request, saves it, and later includes it in another response. Subsequently, a script in the later response mishandles this data through a sink.

```js
element.innerHTML = comment.author
```

#### EX1: Reflected DOM XSS

First, I write this sample XSS code to know if there are any changes on it, and it is easy to search about it `<img src=1 onerror="alert('asdfghjk123')">`

![Screenshot from 2023-09-18 07-36-14](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/14191c29-3cd2-487e-873f-eb5b852d0253)
After search about our string in inspector our input reflected without any changes, but the payload didn't work then let's show the page source![Screenshot from 2023-09-18 07-37-15](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/9760f53b-955f-4bc6-a5af-d531f4300a8e)

Our input doesn't reflect in the page source, but there is a JS file called `searchResults.js` so let's show it![Screenshot from 2023-09-18 07-51-17](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/d125c9b7-f9fb-4254-bd58-31ad96c9664c)

As you can see, our input written with `innerText` and this function doesn't execute any HTML tags

![Screenshot from 2023-09-18 08-20-57](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/bab4464c-0d1e-4155-98c4-199e486e4a69)

But there are an eval function and The eval() method evaluates or executes an argument.

![Screenshot from 2023-09-18 08-49-11](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/fc100c77-73bd-44a6-886b-499c603d7a8e)

And in the JS file it's execute the `responseText`, Let's capture this request with burp

![Screenshot from 2023-09-18 08-21-08](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/27a75ab8-38a8-4210-b30f-c5574227162a)
If we entered, `test` the server response with JSON data and contains our input

```json
{
    "results":[],
    "searchTerm":"test"
}
```

![Screenshot from 2023-09-18 08-22-57](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/1ee30ade-0e3e-46bc-ac92-22b2d1d95380)
Then  if you tried to go out JSON with `"` the server will escape it with `\` then I tried to bypass it with backslash `"\` and it's worked

![Screenshot from 20230918 082510](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/53de3811-fb03-49f5-b355-157bf546dcfc)

Let's create payload with this info:

```js
\"};console.log('done');//
```

It's Work but let's copy this payload

![Screenshot from 2023-09-18 08-32-41](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/808412ad-58b8-429a-aa63-e76468e75724)

And paste it in search bar and open console to show if our payload work or not

![Screenshot from 2023-09-18 08-32-49](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/e3c0f4e0-5142-4ba6-82b3-915d6ad5fc66)
It's work

![Screenshot from 2023-09-18 08-32-57](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/1b88c216-c1cc-4db1-83ce-2db9af6dfed1)
Let's make it run alert fun:

```js
\"};alert('RDOM XSS');//
```

![Screenshot from 2023-09-18 08-34-41](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/0e35c9d6-b4c5-4fd0-87ef-67475c208ea5)
Its' worked and Lab solved 

![Screenshot from 2023-09-18 08-35-09](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/0df5e5b5-2d00-422c-846f-04dd40865d5d)

> [**Python Script for this lab**](https://github.com/MohammedHawary/Solve-Portswigger-Labs-With_py/blob/main/XSS/Reflected_DOM_XSS.py)

#### EX2: Stored DOM XSS

![Screenshot from 2023-09-18 12-03-02](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/6a034015-c10b-4366-a291-56e74833502d)
![Screenshot from 2023-09-18 12-04-14](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/5a9fdcfe-9030-4816-847f-e95b261b05ad)
![Screenshot from 2023-09-18 12-08-51](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/6755eb9e-929a-4c65-9692-3f26b8b743da)
![Screenshot from 2023-09-18 12-09-18](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/231d51ca-a134-41d8-a9e7-abab82e30ed0)
![Screenshot from 2023-09-18 12-10-46](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/a572e727-e421-4fbe-84e3-3f31681f4f85)
![Screenshot from 2023-09-18 12-10-55](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/8d3dbb1f-0808-44ba-a4f0-9a73405da5e8)
![Screenshot from 2023-09-18 12-12-11](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/65044605-1673-4851-9a26-12c8c6e0d33e)
![Screenshot from 2023-09-18 12-13-09](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/3cf42834-7a65-412c-9df9-33d258e932da)
![Screenshot from 2023-09-18 12-13-24](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/d8d5af00-9f35-426f-8e0b-613d9ddcedfb)
![Screenshot from 2023-09-18 12-16-10](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/43b1d55d-e12f-4afc-a5d2-93793f2380cf)
![Screenshot from 2023-09-18 12-16-32](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/b5a39a73-66b7-4b44-8eb1-4c488e25c919)
![Screenshot from 2023-09-18 12-16-45](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/431f5e52-d4eb-4b82-94fd-405793508d88)
![Screenshot from 2023-09-18 12-16-52](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/ad778d8b-f2ba-4ecd-b4cf-43ffa7f57c19)

> [**Python Script for this lab**](https://github.com/MohammedHawary/Solve-Portswigger-Labs-With_py/blob/main/XSS/Stored_DOM_XSS.py)

### Which sinks can lead to DOM-XSS vulnerabilities?

The following are some of the main sinks that can lead to DOM-XSS vulnerabilities:

```js
document.write()
document.writeln()
document.domain
element.innerHTML
element.outerHTML
element.insertAdjacentHTML
element.onevent
```

The following jQuery functions are also sinks that can lead to DOM-XSS vulnerabilities:

```js
add()
after()
append()
animate()
insertAfter()
insertBefore()
before()
html()
prepend()
replaceAll()
replaceWith()
wrap()
wrapInner()
wrapAll()
has()
constructor()
init()
index()
jQuery.parseHTML()
$.parseHTML()
```
