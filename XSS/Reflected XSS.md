# Cross-Site Scripting (XSS)

XSS is a web security flaw that enables attackers to compromise user interactions with a vulnerable app, bypassing the same origin policy. This vulnerability allows attackers to impersonate victims, perform user actions, and access their data. In cases of privileged access, it may lead to complete control over the app's functions and data.

### How does XSS work?

Cross-site scripting manipulates a site to deliver malicious JavaScript to users, fully compromising their interaction when the code runs in their browser.

![cross-site-scripting](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/99f548a0-8511-4a56-b584-b8a72ec6fd4f)

### What are the types of XSS attacks?

There are three main types of XSS attacks. These are:

- **Reflected XSS**: Malicious script from the current HTTP request.

- **Stored XSS**: Malicious script from the website's database.

- **DOM-based XSS**: Vulnerability in client-side code, not server-side.

## Reflected cross-site scripting

Reflected XSS: Occurs when data from an HTTP request is included in the immediate response unsafely. Example:

```html
https://insecure-website.com/status?message=All+is+well.
<p>Status: All is well.</p>
```

The application doesn't perform any other processing of the data, so an attacker can easily construct an attack like this:

```html
https://insecure-website.com/status?message=<script>/*+Bad+stuff+here...+*/</script>
<p>Status: <script>/* Bad stuff here... */</script></p>
```

If the user visits the attacker's crafted URL, the attacker's script runs in the user's browser, within their application session. This script can then perform any action and access any data accessible to the user.

#### EX: Reflected XSS into HTML context with nothing encoded

![Screenshot from 20230902 204934](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/3b04dbf8-a6a4-4a1f-96cc-8f7f1998d329)

![Screenshot from 20230902 205013](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/a7b78fd3-432b-4206-944c-818d431825f8)

![Screenshot from 20230902 205130](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/c3dc9797-626a-4979-97c1-bbfa6e84ba20)

![Screenshot from 20230902 205152](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/a79d3c82-1e79-4f52-886c-0779f36d1ee8)

![Screenshot from 20230902 192703](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/6914479e-19de-4425-8c06-fed906d7e16b)

> [**Python Script for this lab**](https://github.com/MohammedHawary/Solve-Portswigger-Labs-With_py/blob/main/XSS/Reflected_XSS_into_HTML_context_with_nothing_encoded.py)

### Impact of reflected XSS attacks

If an attacker can control a script executed in the victim's browser, they can fully compromise the user. This allows the attacker to:

- Perform actions within the application as the user.

- Access the user's viewable information.

- Modify data the user can modify.

- Initiate interactions with other users, including malicious actions appearing as if from the victim.

Attackers may induce victims to make controlled requests for a reflected XSS attack via links on attacker-controlled sites, content-generating sites, or through messages (email, tweets, etc.). These attacks can target specific users or affect any application user.

Reflected XSS is generally less severe than stored XSS since it requires an external delivery mechanism; stored XSS delivers a self-contained attack within the vulnerable app itself.

### How to find and test for reflected XSS vulnerabilities

Testing for reflected XSS manually involves these steps:

- **Test all entry points**: examining data in <u>HTTP requests</u> This includes <u>URL parameters, message body, file path, and HTTP headers</u>. Note that certain HTTP headers may not be practically exploitable for XSS-like behavior.

- **Submit short**: <u>unique, random alphanumeric values</u> at each entry point to check if they appear in the response. <u>Use values around 8 characters long to evade input validation and reduce accidental matches</u>. Employ Burp Intruder's number payloads with randomly generated hex values and its `grep` payloads settings to flag matching responses.

- **Identify reflection context**: For each <u>occurrence of the random value in the response, determine its context</u>, which could be <u>between HTML tags</u>, within quoted tag attributes, in a JavaScript string, etc.

- **Test a candidate payload**: Based on the reflection context, assess an initial XSS payload that could trigger JavaScript execution if reflected unaltered in the response. Send the request to Burp Repeater, insert the candidate payload, issue the request, and check if it worked in the response. Leave the original random value in the request and position the XSS payload before or after it. Use the random value as the search term in Burp Repeater's response view to locate reflections quickly.

- **Test alternative payloads**: If the application modifies or blocks the candidate XSS payload, explore other payloads and techniques to potentially achieve a successful XSS attack, considering the reflection context and input validation in use.

- **Test in a browser**: If a payload appears successful in Burp Repeater, replicate the attack in a real browser by pasting the URL or modifying the request in Burp Proxy's intercept view. Confirm if the injected JavaScript executes as intended. Use simple JavaScript like "alert(document.domain)" to trigger a visible popup upon success.

### Cross-site scripting contexts

When testing for reflected and stored XSS, identify the following:

- The location of attacker-controlled data in the response.

- Application's data processing, such as input validation.

Based on these findings, choose <u>candidate XSS payloads</u> and test their effectiveness.

### XSS between HTML tags

For an XSS context between HTML tags, insert specific HTML tags that trigger JavaScript execution.

Some useful ways of executing JavaScript are: 

```html
<script>alert(document.domain)</script>
<img src=1 onerror=alert(1)>
```

#### EX1: Reflected XSS into HTML context with most tags and attributes blocked

First, I entered sample html code 

![Screenshot from 2023-09-02 23-28-25](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/26e297b0-276b-45e1-93c2-a8273ac17c8b)
The server reply that the <u>Tag is not allowed</u>, let's capture this request with burp

![Screenshot from 2023-09-02 23-28-30](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/9722f478-fcd7-4f95-b75e-161bbbd4a697)
Write uniqe string and numbers to see where it reflected and if there are any changes on it 

![Screenshot from 2023-09-02 23-29-36](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/b0599fe0-4a84-43bc-b5c6-544cf50a18bf)
No changes in string, let's try any html tag and see what will happend

![Screenshot from 2023-09-02 23-30-03](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/0129a292-d3b9-49ed-ae26-d54aef7a9515)
Tag Not allowed, let's show if blocked this signs `<>` 

![Screenshot from 2023-09-02 23-30-33](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/3ca6a23b-9050-418b-9438-97d103e9579b)
The signs allowed then let's send this request to intruder and bruteforce tags to see if there are any tags not blocked and we will use **port-swiger** [sheet cheat](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet) 

![Screenshot from 2023-09-02 23-30-53](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/9c100a16-f48a-4122-a9e4-36ed9f088826)
Press on **Copy tags to clipboard** and past it on burp and start attack

![Screenshot from 2023-09-02 23-31-19](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/8d9e2f3a-3ddf-40ea-b31d-081c890d39b7)

![Screenshot from 2023-09-02 23-31-54](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/c4bec7ee-9f76-474e-b9da-d5e207dd17ac)

There are html tag called **body** and this tag allowed then let's create xss payload with this tage with js event handler

![Screenshot from 2023-09-02 23-32-08](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/2ea8447c-14c4-40df-91b4-37722970d596)

There server response with <u>Attribute is not allowed</u>, Let's brute-force this also with **port-swiger** [sheet cheat](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)![Screenshot from 2023-09-02 23-32-42](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/303cb93c-c164-45ee-bf77-03b70eef9ab1)
![Screenshot from 2023-09-02 23-33-37](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/a445ff36-a14a-4787-b02a-1ef7d67c0f02)

![Screenshot from 2023-09-02 23-33-48](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/d738ee93-c0a7-4efc-9bc3-bb134396d292)

There are some events allowed Then let's create payload with this info

![Screenshot from 2023-09-02 23-34-19](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/b0f98d88-44fb-48cd-b570-2306e9d87038)      

```log
htts://Oaf0000704970765828f01cf002bOOfb.web-securit-academy.net/?search=<body+onresize=alert('asdfghjk123')> 
```

![Screenshot from 2023-09-02 23-34-50](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/1d2e924d-bafc-4d44-a79f-53041bd9a0e6)

Let's test it in browser but don't forget resize page to make our payload work

![Screenshot from 2023-09-02 23-35-06](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/a013edf0-7499-450a-8a2d-d8960fb36ba7)

It's work then let's write payload to solve this lab

![Screenshot from 2023-09-02 23-36-46](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/81bd2ad5-2789-4fe6-907f-b5bea4e6aac0)

write it it body and send to victim

![Screenshot from 2023-09-02 23-37-50](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/66484fc5-ad1e-44ab-9dd6-7a416a687575)
and the lab solved 

![Screenshot from 2023-09-03 00-19-16](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/0f804dc9-6600-412b-ac9f-131cf029b072)

> [**Python Script for this lab**](https://github.com/MohammedHawary/Solve-Portswigger-Labs-With_py/blob/main/XSS/Reflected_XSS_into_HTML_context_with_most_tags_and_attributes_blocked.py)

#### EX2: Reflected XSS into HTML context with all tags blocked except custom ones

First, Let's know About custem tags.

In this example you can use `tabindex=Number` to foucas this tag in browser when press `tab`  button and use `id` with it to add it in url to auto focus this tag

> **NOTE**
> 
> Why we use tabindex?
> 
> because it's support focusing or you can focus on it

![Screenshot from 2023-09-04 03-20-13](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/78fa689a-06a1-46a1-a7d7-6b673aa63d8f)
Like : `http://example.com/index.html#a1`

![Screenshot from 2023-09-04 03-20-28](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/877f5c2a-5c06-490c-b479-c8a7650967b0)

Then in our lab let's i brout-force all html tags but all blocked 

![Screenshot from 2023-09-02 23-28-30](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/9722f478-fcd7-4f95-b75e-161bbbd4a697)

Then I tried custom tag and it's work![Screenshot from 2023-09-04 03-28-51](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/6277ecff-521d-4e6c-b6e4-fecde78beee4)
Then I create a payload with this info

```js
<asdfghjk123 tabindex='1' onfocus=alert('xss') id='xss'>
```

and it's work

![Screenshot from 2023-09-04 03-33-38](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/c30ace3c-8484-4a39-b587-b518d4093e75)
Let's show it in browser

![Screenshot from 2023-09-04 03-33-47](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/fd1de1a3-f443-407b-bed2-81506954a4a7)
And add #xss in url

![Screenshot from 2023-09-04 03-34-15](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/6e518cd3-f2ba-490f-bfeb-6f81687b2b5a)

The XSS worked successfuly

![Screenshot from 2023-09-04 03-34-02](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/1117fae0-2864-4e01-ac34-5f98bb87f9e6)
Let's Create a Payload to solve this lab and send it to victim in exploit server

```html
<script>
    document.location = "https://0a6b00f403f1d59c8027813d00da00fa.web-security-academy.net/?search=%3Casdfghjk1234+tabindex%3d%271%27+onfocus%3dalert(document.cookie)+id%3d%27xss%27%3E#xss"
</script>
```

![Screenshot from 2023-09-04 03-40-14](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/85283ce2-cea6-435d-95b6-d499499e4e20)

![Screenshot from 2023-09-04 04-01-22](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/98840e24-8c98-4c3b-adf3-9b32d4182e28)

![Screenshot from 2023-09-04 03-40-33](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/4221f40e-20bc-4cf6-acdc-9f54f3a950bc)

> [**Python Script for this lab**](https://github.com/MohammedHawary/Solve-Portswigger-Labs-With_py/blob/main/XSS/Reflected_XSS_into_HTML_context_with_all_tags_blocked_except_custom_ones.py)

#### EX3: Reflected XSS with event handlers and href attributes blocked

First, I bruteforce tags to show any tags allowed and there are 5 tags allowed

`a, svg, animate, title, image`  

![Screenshot from 2023-09-04 04-31-28](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/c7c53538-1e1d-415c-b360-9d26580ca3d0)
I searched about svg tags in mozial and this is the usage and as you can see there are 

attribute called `attributeName` and `values` in attributeName you can write `href` attribute and in `values` you can wire `javascript:alert()`

![Screenshot from 2023-09-04 05-31-00](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/73f610fd-7d33-4dc2-8519-0c8ee232a270)

Then Let's Create payload with this info and test it

![Screenshot from 2023-09-04 05-35-39](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/6b24db66-fe6a-4069-8ab1-316052ee10ec)
The `Click me` text didn't appered i tried some tries but didn't work Let's ask chatGPT 

**How write text in svg tag** 

![Screenshot from 2023-09-04 05-35-56](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/818e1b8d-817c-44a6-8561-d06627dab8a8)

chatGPT ansowed that we can use `<text>` tag 

```html
<svg width="200" height="100" xmlns="http://www.w3.org/2000/svg">
  <text x="20" y="40" font-family="Arial" font-size="20" fill="black">Hello, SVG Text!</text>
</svg>
```

![Screenshot from 2023-09-04 05-19-30](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/d76ed343-6413-413a-96d2-f18fb127d22f)

Then let's add it in our payload and test it

![Screenshot from 2023-09-04 05-35-50](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/b0384538-5f13-452f-a317-87a1cebdb674)
![Screenshot from 2023-09-04 05-35-58](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/a36603ec-fdd7-4feb-9fbe-e5a667f0a571)

It's work

![Screenshot from 2023-09-04 05-18-20](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/57e22104-5c3b-441e-8429-c42e13f161b9)
And the lab solved ![Screenshot from 2023-09-04 05-21-22](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/b52c0803-7ad5-43a8-84dd-fa9577ef1a52)

> [**Python Script for this lab**](https://github.com/MohammedHawary/Solve-Portswigger-Labs-With_py/blob/main/XSS/Reflected_XSS_with_event_handlers_and_href_attributes_blocked.py)

#### EX4: Reflected XSS with some SVG markup allowed

First, I entered sample html code

![Screenshot from 20230902 232825](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/26e297b0-276b-45e1-93c2-a8273ac17c8b) The server reply that the <u>Tag is not allowed</u>, let's capture this request with burp

![Screenshot from 20230902 232830](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/9722f478-fcd7-4f95-b75e-161bbbd4a697) Write uniqe string and numbers to see where it reflected and if there are any changes on it

![Screenshot from 2023-09-05 10-46-39](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/212d30eb-afa4-414a-84b9-5c432192a32c)
No changes, Then let's add custom tag to see what will happen `<asdfghjk123>` 

![Screenshot from 2023-09-05 10-46-56](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/3fa99d19-55a3-4c59-ae22-50ce22833881)
Tag not allowed then let's check if server accepted this sign or not `<>`

![Screenshot from 2023-09-05 10-47-09](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/2e1cc7d8-810c-4c03-9eb1-c975f7dbcc64)
The server allowed it then let's brutforce tags to know if there are any tag allowed with port-swiger [cheat-sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)

![Screenshot from 2023-09-05 10-47-25](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/c37bccfe-c513-41e2-a3dc-b7f777141b8f)
![Screenshot from 2023-09-05 10-47-36](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/033f699a-3689-4a30-960d-68b79efb326b)

![Screenshot from 2023-09-05 10-47-52](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/3ef28a44-b20a-4818-b43d-afeb103332f6)

There are 4 tags allowed `svg, title, image, animatetransform` then let's write xss payload with this info

![Screenshot from 2023-09-05 10-48-55](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/64192b41-6795-4db8-8413-d2652ae2a11b)

we can write payload with `svg` and use `animateTransform` tag Animation event attributes to perform our xss payload

![Screenshot from 2023-09-05 10-59-31](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/e1e96943-4476-4b2f-9f0e-a398651e86a9)

![Screenshot from 2023-09-05 11-00-09](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/580ecb94-c138-42a0-9d3b-35d229b1942f)

Let's make chatGPT help us to write example code

![Screenshot from 2023-09-05 11-03-26](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/11f180b6-da61-4b14-a8bf-eb52fdce9125)

```html
<svg width="200" height="200" xmlns="http://www.w3.org/2000/svg">
  <rect width="100" height="100" fill="blue">
    <animateTransform
      attributeName="transform"
      attributeType="XML"
      type="translate"
      values="0,0;100,100;0,0"
      begin="0s"
      dur="3s"
      repeatCount="indefinite"
      onbegin="alert('Animation started!')"
    />
  </rect>
</svg>
```

let's remove additonal things

```html
<svg>
    <animateTransform onbegin="alert('xss')"/>
</svg>
```

Then let's check this payload

![Screenshot from 2023-09-05 11-05-45](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/546d0f0a-621b-4be8-aa93-3999168727f9) 

> **[Python Script for this lab](https://github.com/MohammedHawary/Solve-Portswigger-Labs-With_py/blob/main/XSS/Reflected_XSS_with_some_SVG_markup_allowed.py)**

## XSS in HTML tag attributes

In HTML attribute XSS contexts, you can potentially terminate the attribute, close the tag, and insert a new one. For example:

```html
"><script>alert(document.domain)</script>
```

Typically, angle brackets are blocked or encoded in this scenario, preventing your input from escaping the tag it's within. If you can terminate the attribute value, you can often add a new attribute to create a scriptable context, like an event handler. For example:

```html
" autofocus onfocus=alert(document.domain) x="
```

The payload above generates an `onfocus` event triggering JavaScript when the element gains focus. It includes the `autofocus` attribute to initiate the `onfocus` event without user interaction and appends `x="` **to fix the subsequent** markup gracefully.

In some cases, the XSS context is within an HTML attribute that inherently allows script execution, eliminating the need to terminate the attribute value. For instance, if it's within the href attribute of an anchor tag, you can employ the javascript: pseudo-protocol to execute JavaScript, like this:

```html
<a href="javascript:alert(document.domain)">
```

You may find websites that encode angle brackets but still enable attribute injection. This can occur within tags that typically don't trigger events automatically, like the canonical tag. You can leverage this using access keys and user interaction in Chrome. Access keys enable keyboard shortcuts tied to specific elements. The accesskey attribute lets you specify a letter, which, when pressed with other keys (platform-specific), triggers events.

Some websites encode angle brackets but still permit attribute injection. This is even possible in tags like the canonical tag that don't usually trigger events automatically. You can exploit this using access keys and user interaction on Chrome. Access keys enable keyboard shortcuts tied to specific elements. By defining an `accesskey='x'` attribute with a letter, you can trigger events when that letter is pressed in combination with other keys (platform-specific).

 To assist with your exploit, you can assume that the simulated user will press the following key combinations:

- ALT+SHIFT+X

- CTRL+ALT+X

- Alt+X

#### EX1: Reflected XSS into attribute with angle brackets HTML-encoded

First, test with this `<asdfghjk123>` to know where the payload reflected or if there are any changes on it 

![Screenshot from 2023-09-05 11-56-30](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/4b7905d3-44e7-4186-818b-3e68a6657a3c)
In the page source There are two places reflected in page source the second place in the `input` tag at `value` attribute

![Screenshot from 2023-09-05 11-57-32](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/f6f4ded5-0fb7-461c-a5e8-12ddb78f6f33)
Let's copy this line and write our payload

```html
"autofocus onfocus="alert('xss')
```

![Screenshot from 2023-09-05 12-33-50](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/53d589c2-48c4-42ba-95ff-52e80e28b9dc)

And Let's check it

![Screenshot from 2023-09-05 12-01-49](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/252fdd7b-baf6-468d-8c9a-628aed5b27c1)

> [**Python Script for this lab**](https://github.com/MohammedHawary/Solve-Portswigger-Labs-With_py/blob/main/XSS/Reflected_XSS_into_attribute_with_angle_brackets_HTML_encoded.py)

### EX2: Reflected XSS in canonical link tag

![Screenshot from 2023-09-10 11-58-33](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/ae59c7a5-5c9f-4165-bd2c-60d118adf69f)
![Screenshot from 2023-09-10 11-58-59](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/ece20300-388d-4c41-8e44-db9e905bbc97)
![Screenshot from 2023-09-10 12-01-05](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/e95808e1-50fa-47ab-b6e5-b22c6df5f45d)
![Screenshot from 2023-09-10 12-01-37](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/2862497f-9b84-4afb-9981-d86914cd7dcf)
![Screenshot from 2023-09-10 12-02-20](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/7cd6b5aa-b167-4601-8c4d-7027d77d41c8)
![Screenshot from 2023-09-10 12-03-07](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/b0c73f32-6d93-42cc-84c5-bfa1c911b6f8)
![Screenshot from 2023-09-10 12-04-01](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/f3224732-fdd1-44a4-9e8f-338f5295f24d)
![Screenshot from 2023-09-10 12-07-51](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/0c9e863d-0ff2-44da-99c7-b028271172dc)
![Screenshot from 2023-09-10 12-08-43](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/7e119a13-bcc7-4b34-a290-64d9233463e6)

![Screenshot from 2023-09-10 12-08-33](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/02f7dd9f-3e7a-4ed4-9a3a-d6ddd86e3913)

> [**Python Script for this lab**](https://github.com/MohammedHawary/Solve-Portswigger-Labs-With_py/blob/main/XSS/Reflected_XSS_in_canonical_link_tag.py)

### XSS in hidden input fields

![b9ffb03974c5-article-xss-hidden-input-field-article](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/4074d908-f462-4807-91df-7f77512b31af)

```html
<input type="hidden" name="redacted" value="default" injection="xss" />
```

XSS in hidden inputs is frequently very difficult to exploit because typical JavaScript events like onmouseover and onfocus can't be triggered due to the element being invisible.

You can use  access keys and the onclick event triggers on hidden input when activated via an access key, as confirmed in Firefox. This allows for executing an XSS payload inside a hidden attribute, provided you can convince the victim to press the specified key combination, such as `ALT+SHIFT+X` on Firefox for Windows/Linux or `CTRL+ALT+X` on OS X. You can customize the key combination using a different key in the access key attribute. Here's the vector:

```html
<input type="hidden" accesskey="X" onclick="alert(1)">
```

This vector, while requiring some user interaction, is still superior to expression(), which only functions on `IE<=9`. Be aware that if the reflection is duplicated, the key combination may fail. To address this, you can inject another attribute that disrupts the second reflection, like this: `" accesskey="x" onclick="alert(1)" x='`

This technique now works in Chrome and can be applied to link elements, making previously unexploitable XSS vulnerabilities in such elements exploitable. For instance, if you control only attributes in a link element, like the rel attribute for canonical, injecting the accesskey attribute with an onclick event can lead to XSS.

```html
<link rel="canonical" accesskey="X" onclick="alert(1)" />
```

Poc using link elements (Press ALT+SHIFT+X on Windows) (CTRL+ALT+X on OS X)

## XSS into JavaScript

In an XSS context involving existing JavaScript within the response, various situations can arise, requiring different techniques for a successful exploit, such as script termination.

In the simplest case, you can close the enclosing script tag around existing JavaScript and introduce new HTML tags to trigger JavaScript execution. For instance, in the following XSS context:

```html
<script>
...
var input = 'controllable data here';
...
</script>
```

Then you can use the following payload to break out of the existing JavaScript and execute your own: 

```html
</script><img src=1 onerror=alert(document.domain)>
```

This works because browsers initially parse HTML to identify page elements, including script blocks, before later parsing and executing JavaScript. The payload leaves the original script incomplete, but this doesn't prevent the subsequent script from being parsed and executed normally.

#### EX: Reflected XSS into a JavaScript string with single quote and backslash escaped

First, test with this `<asdfghjk123>` to know where the payload reflected or if there are any changes on it![Screenshot from 2023-09-05 11-56-30](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/4b7905d3-44e7-4186-818b-3e68a6657a3c)In the page source There are two places reflected in page source the second place in JS code in var `searchTerms`![Screenshot from 2023-09-08 11-03-08](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/4fa02d99-b56a-42d7-98f9-a8458012b10e)
![Screenshot from 2023-09-08 11-03-36](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/de1773bb-b890-4c8c-aa76-b409cc9c1068)
Let's close the script tage and write our payload

```html
</script><img src=1 onerror=alert('xss')>
```

![Screenshot from 2023-09-08 11-04-54](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/9627a1d6-9ce5-4dd2-9fd1-42e8d13768a2)

Our payload worked in page source, Let's show it in browser

![Screenshot from 2023-09-08 11-05-06](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/207ff1d1-5427-465d-9821-f037b60c3a57)
It's work

![Screenshot from 2023-09-08 11-05-14](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/20c14ec4-aa5d-4f34-99fd-af4b219f1630)

> [**Python Script for this lab**](https://github.com/MohammedHawary/Solve-Portswigger-Labs-With_py/blob/main/XSS/Reflected_XSS_into_a_JavaScript_string_with_single_quote_and_backslash_escaped.py)

### Breaking out of a JavaScript string

When XSS occurs within a quoted string literal, it can allow direct JavaScript execution. To prevent issues, fix the script following the XSS context, as syntax errors can halt the entire script.

Ways to break out of a string literal include:

```js
'-alert(document.domain)-'
';alert(document.domain)//
```

Some websites hinder XSS by limiting character usage. This occurs at the website or through a **WAF** that blocks your requests. In such cases, explore alternative methods to execute functions and bypass security. Using the `throw` statement with an exception handler is one approach. It allows passing arguments to functions without parentheses. For instance, assigning the `alert()` function to the global exception handler and using the throw statement can trigger the `alert()` function with 1 as an argument.

```js
onerror=alert;throw 1
```

Some apps try to prevent input from breaking out of JavaScript strings by escaping **single quote** characters with a **backslash**. However, they often** forget to escape the backslash** itself, allowing attackers to use their own backslash to nullify the application's escape character.

 For example, suppose that the input: 

```js
';alert(document.domain)//
```

 gets converted to:

```js
\';alert(document.domain)//
```

You can now use the alternative payload:

    \';alert(document.domain)//

which gets converted to:

```js
\\';alert(document.domain)//
```

In this case, the first backslash makes the second one a literal character, not a special one. As a result, the quote is treated as a string terminator, allowing the attack to succeed.

#### EX1: Reflected XSS into a JavaScript string with angle brackets HTML encoded

First, test with this `<asdfghjk123>` to know where the payload reflected or if there are any changes on it

![Screenshot from 2023-09-08 22-18-35](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/cd54b191-922c-4485-87f1-cc1ed50e6381)
In the page source There are two places reflected in page source the second place in JS code in var `searchTerms` and replace this signs `< >` with `&lt &gt`

![Screenshot from 2023-09-08 22-18-57](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/11428011-cfe9-4a25-8cc1-054eac910a55)

![Screenshot from 2023-09-08 22-19-25](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/826ef069-d71b-4c43-94ba-8160468ad154)
Let's create payload with this info

```js
';javascript:alert("XSS")//
            // or 
';alert("XSS");'
            // or 
'-alert("XSS")-'
```

![Screenshot from 2023-09-08 22-21-48](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/4dc96561-ff9d-45c2-8f1a-0c87a6dcbd56)
The script reflected correctly this let's show it in browser

![Screenshot from 2023-09-08 22-22-31](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/43727950-b8bc-4903-95c2-89fc1315d31b)
It's work

![Screenshot from 2023-09-08 22-22-40](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/0dd1efd4-50ff-48a7-88d1-cd10c8ee6f76)
![Screenshot from 2023-09-08 22-22-48](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/bf045492-de11-4d4e-9dae-1f26c35b7bc9)

> [**Python Script for this lab**](https://github.com/MohammedHawary/Solve-Portswigger-Labs-With_py/blob/main/XSS/Reflected_XSS_into_a_JavaScript_string_with_angle_brackets_HTML_encoded.py)

#### EX2: Reflected XSS into a JavaScript string with angle brackets and double quotes HTML-encoded and single quotes escaped

First, test with this `<img src="1" onerror="alert('asdfghjk123')">` to know where the payload reflected or if there are any changes on it![Screenshot from 2023-09-08 23-53-33](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/520e9361-2b3d-4690-be90-420b4daef041)

In the page source There are two places reflected in page source the second place in JS code in var `searchTerms` and replace this signs `< >` with `&lt &gt` and `"` with `&quot` but adding `\` after `'`

![Screenshot from 2023-09-08 23-45-40](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/3cb424bd-9623-484b-aa1e-2bb834410512)
![Screenshot from 2023-09-08 23-53-02](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/bd4b9936-eafd-411e-b600-1fa962e731e9)

Let's create our payload with this info

```js
\';alert(document.cookie)//
```

![Screenshot from 2023-09-08 23-54-23](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/4710c4d3-41bf-40c4-bdb5-3ae786835001)

The script reflected correctly this let's show it in browser

![Screenshot from 2023-09-08 23-37-44](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/a9a38d13-7b52-4c3d-a69c-204ce8af1351)
It's work

![Screenshot from 2023-09-08 23-37-51](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/a9e9932d-88ee-4763-8590-b6cc63003e09)
![Screenshot from 2023-09-08 23-37-57](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/da597c50-aad1-4244-b345-a28cd84ed779)

> [**Python Script for this lab**](https://github.com/MohammedHawary/Solve-Portswigger-Labs-With_py/blob/main/XSS/Reflected_XSS_into_a_JavaScript_string_with_angle_brackets_and_double_quotes_HTML_encoded_and_single_quotes_escaped.py)

### XSS without parentheses and semi-colons

![b1def7274037articlexsswithoutparenthesesarticle](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/31bd6c17-8a37-42cb-b546-3dc048129106)

There are a way to invoke JavaScript functions without parentheses using `onerror` and the `throw` statement. It involves setting the `onerror` handler to the target function and using `throw` to pass the argument.

```html
<script>onerror=alert;throw 1337</script>
```

The `onerror` handler triggers whenever a JavaScript exception occurs, and the `throw` statement enables you to generate a custom exception with an expression passed to the `onerror` handler. Since `throw` is a statement, it's typically necessary to add a semicolon after the `onerror` assignment to start a new statement and prevent forming an expression.

There is a website that filters out parentheses and semicolons. The first method is simple: use curly braces to create a block statement with your `onerror` assignment. After the blocking statement, use `throw` without a semicolon (or a newline):

```html
<script>{onerror=alert}throw 1337</script>
```

The block statement was effective, but there are a more intriguing option. Surprisingly, since the `throw` statement accepts an expression, you can place the `onerror` assignment within the `throw` statement. As the final part of the expression goes to the `onerror` handler, it triggers the function with the desired arguments. Here's how it operates:

![03b498b7f41barticleonerror](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/7cfe5cf8-9a35-496e-9184-c3ad706fc724)

```js
<script>throw onerror=alert,'some string',123,'haha'</script>
```

If you've tried running the code you'll notice that **Chrome** prefixes the string sent to the exception handler with `Uncaught`.

![2e3196af1042articleuncaughtalert](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/f6d48f0a-5d68-4972-83a2-5ff4ab036209)

Using `eval` as an exception handler to evaluate strings. To recap, by prefixing your string with `=`, you make the `Uncaught` string a variable, allowing for the execution of arbitrary JavaScript. For instance:

```html
<script>{onerror=eval}throw'=alert\x281337\x29'</script>
```

The string passed to `eval` is `Uncaught=alert(1337)`. It functions correctly in **Chrome**. However, in **Firefox**, the exception is prefixed with the **two-word string** `uncaught exception` leading to a syntax error when evaluated.

The `onerror/throw` technique doesn't function when executing a throw from the console. **This is because the throw statement executed in the console sends its result to the console itself, not the exception handler**.

> **NOTE**
> 
> **Uncaught**: is a term commonly used in JavaScript error messages to indicate that an error occurred, but there was no corresponding error handler to catch and handle it. It essentially means that an exception was not caught by any try...catch block or handled in any other way.

#### EX: Reflected XSS in a JavaScript URL with some characters blocked

![Screenshot from 2023-09-09 04-02-18](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/cac8597b-b719-4320-987b-d9f4b995da72)

> [**Python Script for this lab**](https://github.com/MohammedHawary/Solve-Portswigger-Labs-With_py/blob/main/XSS/Reflected_XSS_in_a_JavaScript_URL_with_some_characters_blocked.py)

### XSS in JavaScript template literals

JavaScript template literals, enclosed in backticks and utilizing `${...}`, embed and evaluate JavaScript expressions within strings, seamlessly merging them with the surrounding text.

For instance, this script prints a welcome message with the user's display name.

```js
document.getElementById('message').innerText = `Welcome, ${user.displayName}.`;
```

In XSS within a JavaScript template literal, no need to end the literal; use ${...} to embed and execute JavaScript expressions. For example, in this XSS context:

```html
<script>
...
var input = `controllable data here`;
...
</script>
```

Then you can use the following payload to execute JavaScript without terminating the template literal: 

```js
${alert(document.domain)}
```

#### EX: Reflected XSS into a template literal with angle brackets, single, double quotes, backslash and backticks Unicode-escaped

First, test with this `<img src="1" onerror="alert('asdfghjk123')">` to know where the payload reflected or if there are any changes on it

![Screenshot from 2023-09-10 02-36-05](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/ad714088-ed9f-422f-9fe7-f02313f140ee)
In the page source There are one place reflected in page source it's reflected in JS code in var `message` and replace this signs `< > " ' \` with  **CSS (ISO)** of it `\u003c \u0022 \u0027 \002f`

![Screenshot from 2023-09-10 02-36-31](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/7bf19c19-6f01-4a61-969b-d4f54db19ab3)
![Screenshot from 2023-09-10 02-53-10](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/10e043dd-43db-41dc-a362-16f326f47f75)

Let's show if Jquery signs blocked or not 

![Screenshot from 2023-09-10 02-41-33](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/ddf0368d-393a-4b85-8f07-8374200c6cbe)
It's allowed thin let's create our payload with this info

```js
${alert(document.cookie)}
```

![Screenshot from 2023-09-10 02-42-38](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/e9ff95da-02b2-4e51-9765-71a7262e201c)
The script reflected correctly this let's show it in browser

![Screenshot from 2023-09-10 02-42-47](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/fee746b1-2be7-41e7-b72e-57b1bf49e44e)
It's work

![Screenshot from 2023-09-10 02-42-55](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/4be55bac-25b2-475c-966f-7752247085e5)
![Screenshot from 2023-09-10 02-42-59](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/20b1fa30-eef0-455e-854e-0438e03f06e7)

> [**Python Script for this lab**](https://github.com/MohammedHawary/Solve-Portswigger-Labs-With_py/blob/main/XSS/Reflected_XSS_into_a_template_literal_with_angle_brackets_single_double_quotes_backslash_and_backticks_Unicode_escaped.py) 

### Common questions about reflected cross-site scripting

**Difference between reflected XSS and stored XSS**: Reflected XSS injects input into the immediate response, while stored XSS stores input and embeds it into a later response.

**Difference between reflected XSS and self-XSS**: Self-XSS resembles reflected XSS in application behavior but requires the victim to submit the XSS payload, often through social engineering. Self-XSS is typically considered low-impact.
