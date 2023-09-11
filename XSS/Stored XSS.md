# Stored XSS

Stored cross-site scripting (second-order or persistent XSS) occurs when an app improperly includes untrusted data in later HTTP responses.

Imagine a website where users can submit comments on blog posts via an HTTP request like this:

```log
POST /post/comment HTTP/1.1
Host: vulnerable-website.com
Content-Length: 100

postId=3&comment=This+post+was+extremely+helpful.&name=Carlos+Montoya&email=carlos%40normal-user.net
```

After this comment has been submitted, any user who visits the blog post will receive the following within the application's response:

```html
<p>This post was extremely helpful.</p>
```

Assuming the application doesn't perform any other processing of the data, an attacker can submit a malicious comment like this:

```html
<script>/* Bad stuff here... */</script>
```

Within the attacker's request, this comment would be URL-encoded as:

```log
comment=%3Cscript%3E%2F*%2BBad%2Bstuff%2Bhere...%2B*%2F%3C%2Fscript%3E
```

Any user who visits the blog post will now receive the following within the application's response:

```html
<p><script>/* Bad stuff here... */</script></p>
```

The script supplied by the attacker will then execute in the victim user's browser, in the context of their session with the application

#### EX: Stored XSS into HTML context with nothing encoded

First, test with this `<img src="1" onerror="alert('asdfghjk123')">` to know where the payload stored or if there are any changes on it

![Screenshot from 2023-09-10 03-17-55](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/375c4aa5-eae1-4f68-8440-35303b2b7370)

Our payload send to the server with POST request in `comment` and `name` parameters

![Screenshot from 2023-09-10 03-19-42](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/8d8eede0-0a16-4e3d-b188-9d0b71c7f31a)

As you can see our payload stored in HTML code in two places in `name` of user that posted this post and `comment` of this user and this place our payload stored without any problem

![Screenshot from 2023-09-10 03-14-06](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/587c3b6a-46cb-4aad-ad39-78555713c7eb)

If you backed to see our comment, the XSS will work![Screenshot from 2023-09-10 03-13-14](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/4d19291c-8b35-481f-8080-332303f3436f)

> [**Python Script for this lab**](https://github.com/MohammedHawary/Solve-Portswigger-Labs-With_py/blob/main/XSS/Stored_XSS_into_HTML_context_with_nothing_encoded.py) 

### Impact of stored XSS attacks

- If an attacker controls a script in the victim's browser, they can fully compromise the user, performing actions like those in reflected XSS vulnerabilities.

- The key difference in exploitability between reflected and stored XSS is that stored XSS attacks are self-contained within the application. The attacker doesn't need external methods to make users trigger their exploit; they insert it into the app and wait for users to encounter it.

- This self-contained nature is especially relevant when an XSS vulnerability only affects logged-in users. In reflected XSS, timing is crucial, as users must be logged in when they encounter the attacker's request. In stored XSS, users are guaranteed to be logged in when they encounter the exploit.

### How to find and test for stored XSS vulnerabilities

Manual testing for stored XSS vulnerabilities is challenging. You should examine all points where you enter attacker-controlled data and places where it may appear in application responses.

Entry points into the application's processing include: 

- URL query string and message body data.

- URL file path.

- Some non-exploitable HTTP request headers.

- Application-specific routes for external data, like emails in webmail, tweets in a Twitter feed, or content from other websites in a news aggregator.

Stored XSS attacks can target all HTTP responses to any app user, irrespective of context.

To start testing for these vulnerabilities, the initial step is pinpointing the links between entry and exit points. These are where data entered at an entry point eventually appears or gets transmitted via an exit point. This task can be challenging due to various factors, including:

- Data from entry points can potentially surface at any exit point. For instance, user-provided display names might show up in a hidden audit log accessible to select users.

- Stored application data can easily be overwritten by other actions. For instance, a search function may replace recent searches as users conduct new ones.

Comprehensively tracing entry-to-exit links means testing each permutation individually: inputting a value, going straight to the exit, and checking if it appears. Yet, in applications with many pages, this becomes impractical.

A practical approach involves systematically testing data entry points, submitting values, and monitoring responses for value appearances. Pay special attention to functions like blog post comments. If a value shows up, investigate if it's stored across requests, not just reflected immediately.

Once you've found entry-to-exit links, test each one for potential stored XSS vulnerabilities. Identify where stored data appears in the response, then test relevant XSS payloads for that context. The testing method is similar to finding reflected XSS vulnerabilities.

### XSS in HTML tag attributes

In some cases, the XSS context occurs within an HTML tag attribute that can create a scriptable context without needing to close the attribute value. For instance, in the href attribute of an anchor tag, you can use the javascript pseudo-protocol to execute script, like this:

```html
<a href="javascript:alert(document.domain)">
```

#### EX: Stored XSS into anchor href attribute with double quotes HTML-encoded

First, test with this `<img src="1" onerror="alert('asdfghjk123')">` to know where the payload stored or if there are any changes on it

![Screenshot from 2023-09-10 11-08-59](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/4eeedb33-da2d-4c20-bc2d-4b71d9b1ad3a)
There are three places our payload stored in page source the important place in the `href` attribute

![Screenshot from 2023-09-10 11-10-16](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/211ef9fe-ccd2-466e-b57a-5aab869f8b9b)
Let's create our payload 

```js
javascript:alert(document.domain)
```

![Screenshot from 2023-09-10 11-11-37](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/2dea4876-640c-44f8-b582-9b73a8f7dde1)
lab solved

![Screenshot from 2023-09-10 11-11-55](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/177fce5a-3ae9-4473-b077-e08781773b7c)
And to run xss press on `test`

![Screenshot from 2023-09-10 11-12-10](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/e01f6108-b1b6-4622-b75e-39b1a84a05c1)

> [**Python Script for this lab**](https://github.com/MohammedHawary/Solve-Portswigger-Labs-With_py/blob/main/XSS/Stored_XSS_into_anchor_href_attribute_with_double_quotes_HTML_encoded.py)

### Making use of HTML-encoding

When XSS occurs within JavaScript code enclosed in a quoted tag attribute, like an event handler, **HTML encoding** can sometimes be utilized to circumvent input filters.

After the browser parses the response and removes HTML tags and attributes, it performs HTML decoding on tag attribute values before proceeding. If the server-side application restricts or sanitizes crucial characters for an XSS attack, you can often evade input validation by employing HTML encoding on those characters.

For instance, if the XSS context is as follows:

```html
<a href="#" onclick="... var input='controllable data here'; ...">
```

If the application blocks or escapes single quote characters, you can use this payload to break out of the JavaScript string and execute your script:

![Screenshot from 2023-09-11 00-29-39](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/9ea45c9e-889f-4610-bd14-f9e931d5584e)
It's stored in page source in four places, the important places in `a` tag this payload that I wrote the `Website` payload

![Screenshot from 2023-09-11 00-31-02](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/4547d3ed-6384-4ae8-aefe-668c718e024e)

Then I wrote this payload

    http://www.asdfghjk1123.com?'alert(document.domain)'

![Screenshot from 2023-09-11 01-21-51](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/c6c68bd1-5a1d-487d-b036-5563338db7d8)
The server escaped the `'` with backslash

![Screenshot from 2023-09-11 01-22-57](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/34c48bdc-d77b-4684-b629-45bde9543474)

Let's try to escape the backslash with backslash

![Screenshot from 20230911 004745](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/67ce419a-17f7-4ea6-bac3-a97d1227138d) 

Not worked

![Screenshot from 20230911 004823](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/7fe23387-f27c-4dc6-a322-d697cae6ec5a)

Then Let's try to use HTML encoding

![Screenshot from 2023-09-11 00-49-15](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/3d8da689-1ff6-4897-a912-4a64612e8128)

And Create our payload 

```html
http://hacker.com?&#x27;-alert()-&#x27;
```

![Screenshot from 2023-09-11 00-49-28](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/107ee0ba-cf6e-4616-9246-09b354c69e6e)
And lab solved

![Screenshot from 2023-09-11 00-49-43](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/b53ff449-6f48-43ec-bf34-2e289d7c9703)
Our payload

![Screenshot from 2023-09-11 00-50-51](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/fa7f0ecc-009b-4f85-8019-e98990af9c20)

To run exploit press the link `hacker`

![Screenshot from 2023-09-11 00-50-04](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/d7f0eb8d-0f13-446e-ace1-426875849bf3)

> [**Python Script for this lab**](https://github.com/MohammedHawary/Solve-Portswigger-Labs-With_py/blob/main/XSS/Stored_XSS_into_onclick_event_with_angle_brackets_and_double_quotes_HTML_encoded_and_single_quotes_and_backslash_escaped.py)
