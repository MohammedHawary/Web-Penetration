# Cross-site request forgery (CSRF)

**CSRF** is a web security vulnerability that tricks users into unknowingly performing unintended actions. It bypasses the same origin policy that prevents websites from interfering with each other.

<img title="" src="https://github.com/MohammedHawary/Web-Penetration/assets/94152045/fcc898ed-4702-4d81-a87a-8fadbcf9d92e" alt="cross-site request forgery" data-align="inline" width="1050">

### What is the impact of a CSRF attack

In a **CSRF** attack, the attacker tricks the victim into performing unintended actions, such as **changing email address**, **password**, or making **unauthorized transfers**. The attacker can **gain control over the user's account**, potentially accessing all data and functionality if the user has privileged access.

### How does CSRF work

For a CSRF attack to be possible, three key conditions must be in place: 

- **A relevant action**. The attacker aims to induce a specific action within the application, which could be a privileged action or any modification to user-specific data, like changing passwords.

- **Cookie-based session handling**. The action is carried out through HTTP requests, and the application solely relies on session cookies to identify the user, lacking any additional mechanisms for session tracking or user request validation.

- **No unpredictable request parameters**. The requests for performing the action do not have any parameters with values that the attacker cannot determine or guess. For instance, if the attacker needs to know the existing password to cause a password change, the function is not vulnerable.

For example, suppose an application contains a function that lets the user change the email address on their account. When a user performs this action, they make an HTTP request like the following: 

```log
POST /email/change HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 30
Cookie: session=yvthwsztyeQkAPzeQ5gHgTvlyxHfsAfE

email=wiener@normal-user.com
```

 This meets the conditions required for **CSRF**: 

- The attacker is interested in changing the email address associated with a user's account. By doing so, they can usually initiate a password reset and gain complete control over the user's account.

- The application uses a **session cookie** to identify which user issued the request. There are no other tokens or mechanisms in place to track user sessions. 

- The attacker can easily determine the values of the request parameters that are needed to perform the action. 

With these conditions in place, the attacker can construct a web page containing the following HTML: 

```html
<html>
    <body>
        <form action="https://vulnerable-website.com/email/change" method="POST">
            <input type="hidden" name="email" value="pwned@evil-user.net" />
        </form>
        <script>
            document.forms[0].submit();
        </script>
    </body>
</html>
```

 If a victim user visits the attacker's web page, the following will happen: 

- The attacker's page will trigger an HTTP request to the vulnerable web site.

- If the user is logged in to the vulnerable web site, their browser will automatically include their session cookie in the request (assuming SameSite cookies are not being used).

- The vulnerable web site will process the request in the normal way, treat it as having been made by the victim user, and change their email address.

> **NOTE**
> 
> Although CSRF is normally described in relation to cookie-based session handling, it also arises in other contexts where the application automatically adds some user credentials to requests, such as HTTP Basic authentication and certificate-based authentication.

### How to construct a CSRF attack

Generating a CSRF exploit manually can be difficult, especially when dealing with numerous parameters or complex request requirements. To simplify the process, you can utilize the built-in CSRF Proof of Concept (PoC) generator in Burp Suite Professional.

![kali Linux EveryThing - VMware Workstation 8_13_2023 2_50_53 AM](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/dc83b129-bbe7-4077-a6ee-b5930ef3eb59) 

![Screenshot from 2023-08-12 17-41-08](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/2adca186-57f8-4c1e-ac91-5c2aa964ec2d)

#### EX: CSRF vulnerability with no defenses

![Screenshot from 2023-08-12 20-29-18](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/b29a7713-45f1-446b-a7c5-dc40af7c19f4)
After Login, I tried to change email and captured the request

![Screenshot from 2023-08-12 20-29-38](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/8f9dd3b1-c9d6-42ae-afcb-180983fb9eac)
Let's create `CSRF`  html code from burp

![Screenshot from 2023-08-12 20-29-53](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/42a519ec-6412-4b7e-bf85-4eebdf3a32ae)
In exploit server, add this `html` code to `body` and send it by press button `Deliver exploit to victim`

![Screenshot from 2023-08-12 20-30-08](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/e0426c45-8f68-4284-b0db-a07552a8e12a)
![Screenshot from 2023-08-12 20-50-20](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/62871d4d-12dd-43aa-ad3c-8b42485837e0)

### How to deliver a CSRF exploit

CSRF attacks use similar delivery mechanisms to `reflected XSS` attacks. The attacker typically places malicious `HTML` on a web page they control and tricks victims into visiting that page. This can be achieved by sharing a link to the malicious page via email or social media. Another approach is to inject the attack into a popular website, such as a user comment, and wait for users to visit the site.

In some cases, simple `CSRF` exploits can be executed using the `GET` method and a single URL on the vulnerable website. This means that the attacker may not need an external site and can directly provide victims with a malicious URL on the vulnerable domain. For example, if the email address change request can be made with the `GET` method, a self-contained attack would look like this:

```html
<img src="https://vulnerable-website.com/email/change?email=pwned@evil-user.net">
```

## Bypassing CSRF token validation

### What is a CSRF token

A **CSRF token** is a **server-generated value** shared with the client. It serves as a unique, secret, and unpredictable identifier. To perform sensitive actions like form submission, the client must provide the correct **CSRF token**. **Without it, the server will reject the requested action**.

A common way to share `CSRF` tokens with the client is to include them as a **hidden** parameter in an `HTML` form, for example: 

```html
<form name="change-email-form" action="/my-account/change-email" method="POST">
    <label>Email</label>
    <input required type="email" name="email" value="example@normal-website.com">
    <input required type="hidden" name="csrf" value="50FaWgdOhi9M9wyna8taR1k3ODOR8d6u">
    <button class='button' type='submit'> Update email </button>
</form>
```

 Submitting this form results in the following request: 

```log
POST /my-account/change-email HTTP/1.1
Host: normal-website.com
Content-Length: 70
Content-Type: application/x-www-form-urlencoded

csrf=50FaWgdOhi9M9wyna8taR1k3ODOR8d6u&email=example@normal-website.com
```

**CSRF tokens**, when properly implemented, defend against `CSRF` attacks by making it challenging for attackers to create a valid request on behalf of the victim. Since the attacker cannot predict the correct `CSRF` token value, they are unable to include it in the malicious request.

> **NOTE**
> 
> CSRF tokens can be transmitted in various ways, not just as hidden parameters in POST requests. Some applications choose to place CSRF tokens in HTTP headers. The method of token transmission greatly affects the overall security of the mechanism. For more details, refer to the [guidelines on preventing CSRF vulnerabilities](https://portswigger.net/web-security/csrf/preventing).

## Common flaws in CSRF token validation

CSRF vulnerabilities often occur due to inadequate validation of CSRF tokens. In this section, we'll discuss common issues that allow attackers to bypass these defenses.

### Validation of CSRF token depends on request method

Some applications correctly validate the token when the request uses the `POST` method but skip the validation when the `GET` method is used.

In this situation, the attacker can switch to the `GET` method to **bypass the validation and deliver a CSRF attack**: 

```log
GET /email/change?email=pwned@evil-user.net HTTP/1.1
Host: vulnerable-website.com
Cookie: session=2yQIDcpia41WrATfjPqvm9tOkDvkMvLm
```

#### EX: CSRF where token validation depends on request method

![Screenshot from 2023-08-12 20-29-18](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/b29a7713-45f1-446b-a7c5-dc40af7c19f4)

After Login, I tried to change email and captured the request

![Screenshot from 2023-08-12 21-41-23](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/4398ee13-90e4-476f-911c-5eafc781b606)
I tried `CSRF` with `POST` method, but there are a validation on this method then let's try to change the request method to `GET` and send it to show if the server supported this method or not

![Screenshot from 2023-08-12 21-41-34](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/7f465c15-2550-4720-b7b0-e0551e7c851f)
As we see the server support `GET` request method then let's retry `CSRF` attack again and copy the `CSRF` HTML code from burp

![Screenshot from 2023-08-12 21-42-01](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/32dfc27c-dfb1-4ab2-93ee-683d031f090e)

In exploit server, add this `html` code to `body` and send it by press button `Deliver exploit to victim`

![Screenshot from 2023-08-12 21-42-35](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/5650cf25-d10c-43ef-8abe-3a9be551c2af)      

![Screenshot from 2023-08-12 21-42-42](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/1a75ca5d-caf0-472c-aea6-dfe9bc63cd6e)

### Validation of CSRF token depends on token being present

Some applications validate the token correctly when it exists but **skip validation if the token removed**.

In this situation, the attacker can remove the entire parameter containing the token (<u>not just its value</u>) to bypass the validation and deliver a `CSRF` attack: 

```log
POST /email/change HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 25
Cookie: session=2yQIDcpia41WrATfjPqvm9tOkDvkMvLm

email=pwned@evil-user.net
```

#### EX: CSRF where token validation depends on token being present

![Screenshot from 20230812 202918](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/b29a7713-45f1-446b-a7c5-dc40af7c19f4)

After Login, I tried to change email and captured the request![Screenshot from 2023-08-12 22-09-47](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/c5ab914e-9d5c-4c62-bb51-55ec38093021)
I saw the CSRF token, Let's remove it and try to send the request

![Screenshot from 2023-08-12 22-10-01](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/6734a47b-a667-47ab-a71f-e940e8bce484)
The server accepted the request without any error, then let's try `CSRF` attack

![Screenshot from 2023-08-12 22-10-15](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/bdcfd2cf-ffd3-467e-a029-33934e8d78b3)
![Screenshot from 2023-08-12 22-11-04](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/35022318-0fe5-449f-a84e-1c2a4f50ed8f)
![Screenshot from 2023-08-12 22-11-33](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/708110ca-3d7b-421d-85f2-dca8d027fe19)

### CSRF token is not tied to the user session

Certain applications fail to verify if the **CSRF token belongs to the same session as the user making the request**. Instead, they maintain a global pool of issued tokens and accept any token found in this pool.

In such cases, an **attacker can log in to the application using their own account, obtain a valid token**, and then provide that token to the victim user in a `CSRF` attack.

#### EX: CSRF where token is not tied to user session

![Screenshot from 2023-08-13 00-14-14](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/009ec9c1-0f0c-41a0-a7e5-3e43ad8a7da3)
After Login, I tried to change email and captured the request

![Screenshot from 2023-08-13 00-14-31](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/4700eb82-4250-4b13-9942-393ff73cb731)
I saw the CSRF token, Let's remove it and try to send the request![Screenshot from 2023-08-13 00-14-44](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/010074bf-e312-4582-9d57-cdff6889511c)
The server response with error that it's messing parameter `CSRF`, let's check if the server allowed GET request method or not to bypass the `CSRF` token validation

![Screenshot from 2023-08-13 00-14-52](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/8adf5d92-1c5b-4864-9789-b19ffe380572)
The server resonse with error that it the method not allowed ,Then let's check if the ssrf token created specialy for this seasion coockie or not by login with another user and take his `ssrf` token and putting it in burp repeater to attacker user 

![CSRF where token is not tied to user session and 5 more pages - Personal - Microsoft​ Edge 8_13_2023 5_35_20 AM](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/521119ec-42a4-45cf-a2ba-b50ec24f8c9e)

It's work then let's try csrf attack

![Screenshot from 2023-08-13 00-16-30](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/ce934411-55d7-441e-b5b4-33eafc5c2df3)
let's create csrf html code and change the value of ssrf with valid ssrf from another account

![Screenshot from 2023-08-13 00-17-30](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/fbb92dad-fe06-4445-90f6-c39d3e02371e)

![Screenshot from 2023-08-13 00-19-18](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/c3beedb1-5197-4f71-be57-69c67d37b90f)
![Screenshot from 2023-08-13 00-19-24](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/eea29b08-d6ad-4273-8766-3586df8d4720)

### CSRF token is tied to a non-session cookie

In a variation of the previous vulnerability, some applications associate the **CSRF token with a cookie that is different from the one used for session tracking**. This situation commonly arises when an application **utilizes separate frameworks for session handling and CSRF protection**, which are not effectively integrated.

```log
POST /email/change HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 68
Cookie: session=pSJYSScWKpmC60LpFOAHKixuFuM4uXWF; csrfKey=rZHCnSzEp8dbI6atzagGoSYyqJqTz5dv

csrf=RhV7yQDO0xcq9gLEah2WVbmuFqyOq7tY&email=wiener@normal-user.com
```

**This situation is harder to exploit** but is **still vulnerable**. **If the web site contains any behavior that allows an attacker to set a cookie in a victim's browser**, then an attack is possible. The attacker can log in to the application using their own account, obtain a valid token and associated cookie, leverage the cookie-setting behavior to place their cookie into the victim's browser, and feed their token to the victim in their CSRF attack. 

> **NOTE**
> 
> The cookie-setting behavior does not even need to exist within the same web application as the `CSRF` vulnerability. Any other application within the same overall `DNS` domain can potentially be leveraged to set cookies in the application that is being targeted, if the cookie that is controlled has suitable scope. For example, a cookie-setting function on `staging.demo.normal-website.com` could be leveraged to place a cookie that is submitted to `secure.normal-website.com`.

#### EX: CSRF where token is tied to non-session cookie

-------------------><mark>not ended the Because lab problem</mark><----------------

### CSRF token is simply duplicated in a cookie

In another form of the vulnerability, certain applications don't keep track of issued tokens on the server-side. Instead, they duplicate each token in a cookie and a request parameter. During validation, the application only checks if the token in the request parameter matches the one in the cookie. This method, known as the "**double submit**" **defense against CSRF**, is favored for its simplicity and avoidance of server-side state:

```log
POST /email/change HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 68
Cookie: session=1DQGdzYbOJQzLP7460tfyiv3do7MjyPw; csrf=R8ov2YBfTYmzFyjit8o2hKBuoIjXXVpa

csrf=R8ov2YBfTYmzFyjit8o2hKBuoIjXXVpa&email=wiener@normal-user.com
```

#### EX: CSRF where token is duplicated in cookie

-------------------><mark>Not ended the Because lab problem</mark><----------------

## Bypassing SameSite cookie restrictions

`SameSite` is a browser security mechanism that determines **when a website's cookies are included in requests originating from other websites**. **`SameSite` cookie restrictions provide partial protection against a variety of cross-site attacks, including `CSRF`, `cross-site leaks`, and some `CORS` exploits**. 

Since 2021, Chrome enforces default `Lax SameSite` restrictions for cookies unless the issuing website explicitly sets its own restriction level.

### What is a site in the context of SameSite cookies

In `SameSite` cookie restrictions, a "site" refers to the **top-level domain** (`TLD`) and one additional level of the domain name, commonly known as `TLD+1`. The URL scheme is also considered when determining if a request is same-site or **cross-site**. For example, a link from `http://app.example.com` to `https://app.example.com`  is typically treated as **cross-site** by most browsers.

![site-definition](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/d3486301-e108-4bed-8f98-4d3c2e0855af)

> **NOTE**
> 
> You may come across the term **effective top-level domain** (`eTLD`). This is just a way of accounting for the reserved multipart suffixes that are treated as **top-level domains** in practice, such as `.co.uk`.

### What's the difference between a site and an origin

A `site` includes **multiple domain** names, while an `origin` refers to a **single domain** name. It is crucial to distinguish between the two as using them interchangeably can lead to significant security risks. Two URLs are considered to have the same `origin` if they share the **same scheme, domain name, and port** (although the port is often implied by the scheme).

![site-vs-origin](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/d153d8a0-d70a-45cb-8644-7c1ffa36dc8e)

| Request from            | Request to                   | Same-site?            | Same-origin?               |
| ----------------------- | ---------------------------- | --------------------- | -------------------------- |
| https://example.com     | https://example.com          | YES                   | Yes                        |
| https://app.example.com | https://intranet.example.com | YES                   | No: mismatched domain name |
| https://example.com     | https://example.com:8080     | YES                   | No: mismatched port        |
| https://example.com     | https://example.co.uk        | No: mismatched eTLD   | No: mismatched domain name |
| https://example.com     | http://example.com           | No: mismatched scheme | No: mismatched scheme      |

This is an important distinction as it means that any vulnerability enabling arbitrary JavaScript execution can be abused to bypass site-based defenses on other domains belonging to the same site.

### How does SameSite work

`SameSite` **prevents browsers from sending cookies to the issuing domain in every request**, even if triggered by a third-party website. **It enables control over which cross-site requests include specific cookies**, reducing the risk of `CSRF` attacks. These attacks rely on a victim's browser triggering harmful actions using a cookie associated with their authenticated session, which **will fail if the browser doesn't include the cookie**.

All major browsers currently support the following `SameSite` **restriction levels**: 

- `Strict`

- `Lax`

- `None`

Developers can set a restriction level for each cookie manually, granting them greater control over cookie usage. They can achieve this by **including the `SameSite` attribute in the `Set-Cookie` response header and specifying their desired value**:

```log
Set-Cookie: session=0F8tgdOhi9ynR1M9wa3ODa; SameSite=Strict
```

 Although this offers some protection against `CSRF` attacks, **none of these restrictions provide guaranteed immunity**

> **NOTE**
> 
> If a website doesn't set a `SameSite` attribute for a cookie, **Chrome applies default** `Lax` **restrictions automatically**. This restricts the cookie to be sent only in certain cross-site requests, even without explicit configuration by developers. Other major browsers are expected to adopt this behavior as it becomes a standard in the future.

### Strict

If a cookie has the `SameSite=Strict` attribute, **browsers won't send it in cross-site requests**. In other words, **if the request's target site doesn't match the site in the browser's address bar, the cookie won't be included**.

**This is recommended for cookies that allow the bearer to modify data or perform sensitive actions, like accessing authenticated-only pages**.

Although this is the most secure option, it can negatively impact the user experience in cases where cross-site functionality is desirable. 

### Lax

`Lax` SameSite restrictions mean that browsers will send the cookie in cross-site requests, but only if both of the following conditions are met: 

- The request uses the `GET` method.

- The request resulted from a top-level navigation by the user, such as clicking on a link.

This means the cookie is excluded from cross-site `POST` requests, which are commonly targeted in `CSRF` attacks due to their potential to modify data or state.

Likewise, the cookie is not included in background requests, such as those initiated by scripts, `iframe`, or references to `images` and other resources. 

### None

If a cookie has the `SameSite=None` attribute, **it disables `SameSite` restrictions entirely, regardless of the browser**. **This means the cookie will be sent in all requests to the issuing site, even if they were triggered by unrelated third-party sites**.

With the exception of **Chrome**, this is the default behavior used by major browsers if no `SameSite` attribute is provided when setting the cookie. 

Disabling `SameSite` can be valid in certain cases, such as when the cookie is used in a third-party context without granting access to sensitive data or functionality (e.g., tracking cookies).

If you come across a cookie set with `SameSite=None` or no explicit restrictions, it's worth investigating its purpose. Initially, Chrome's adoption of "**Lax-by-default**" behavior caused compatibility issues with existing web functionality. As a temporary solution, some websites chose to disable `SameSite` restrictions on all cookies, including potentially sensitive ones.

When using `SameSite=None`, the website must also include the `Secure` attribute to ensure the cookie is only sent over `HTTPS` in encrypted messages. Otherwise, the cookie will be rejected by browsers and not set.

```log
Set-Cookie: trackingId=0F8tgdOhi9ynR1M9wa3ODa; SameSite=None; Secure
```

### Bypassing SameSite Lax restrictions using GET requests

In practice, servers may not always differentiate between `GET` and `POST` requests to a specific endpoint, even when expecting form submissions. If the servers also have `Lax` restrictions on their session cookies, either explicitly or by browser default, it's still possible to carry out a `CSRF` attack by triggering a `GET` request from the victim

 As long as the request involves a top-level navigation, the browser will still include the victim's session cookie. The following is one of the simplest approaches to launching such an attack: 

```html
<script>
    document.location = 'https://vulnerable-website.com/account/transfer-payment?recipient=hacker&amount=1000000';
</script>
```

Even if an ordinary `GET` request isn't allowed, Some frameworks allow overriding the specified method in a `GET` request. For instance, `Symfony` supports the `_method` parameter in forms, which takes precedence over the **default method for routing**.

```html
<form action="https://vulnerable-website.com/account/transfer-payment" method="POST">
    <input type="hidden" name="_method" value="GET">
    <input type="hidden" name="recipient" value="hacker">
    <input type="hidden" name="amount" value="1000000">
</form>
```

 Other frameworks support a variety of similar parameters. 

### EX: SameSite Lax bypass via method override

![Screenshot from 2023-08-14 12-43-34](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/29dfc057-93b2-4f4a-b012-e6eca50f0481)

I login and captured the request 

![Screenshot from 2023-08-14 12-38-37](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/a63b3531-862e-4854-b6e0-a01b15e6817e)

In the login response the website doesn't explicitly specify any `SameSite` restrictions when setting session cookies. As a result, the browser will use the default `Lax` restriction level. 

![Screenshot from 2023-08-14 11-35-17](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/a1e0d142-9da1-41c9-a6ed-d33a1c209b87)
Recognize that this means the session cookie will be sent in cross-site `GET` requests, as long as they involve a top-level navigation. then let's change Request method to `GET` and send
![Screenshot from 2023-08-14 11-49-54](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/87fff7d0-e380-4269-8be5-56d201b5844e)
The server response with `Method Not Allowed` Let's try bypass it with adding this parameter to query `_method` string and send it 

![Screenshot from 2023-08-14 11-50-13](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/3bbd826a-fd35-48e1-935e-83a6c1c5747e)The request have been accepted by the server then let's Create our payload

```html
<script>
    document.location = 'https://vulnerable-website.com/my-account/change-email?email=hacker&_method=POST';
</script>
```

![Screenshot from 2023-08-14 12-32-44](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/8918fb45-8a97-43d3-97b6-ca5c24e70b5f)
let's test it in browser 

![Screenshot from 2023-08-14 11-52-21](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/98b23116-4f09-4866-86d3-ac01062c59fe)
and it's work then let's to exploit server and send it to victim
![Screenshot from 2023-08-14 12-36-02](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/ef7f35ee-2e43-4785-8c4d-02aef4eeea49)

### Bypassing SameSite restrictions using on-site gadgets

If a cookie has the `SameSite=Strict` attribute, it won't be included in cross-site requests. However, there's a way to bypass this restriction by using a gadget like a client-side redirect(`open redirection`). This redirect constructs the target dynamically using input that the attacker controls, such as URL parameters. These redirects are not recognized as redirects by browsers. Instead, the resulting request is treated as a regular standalone request within the same site. This means that all cookies related to the site will be included in the request, regardless of any restrictions in place. By manipulating this gadget to trigger a malicious secondary request, it's possible to completely bypass `SameSite` cookie restrictions.

>  **NOTE** 
> 
> That the equivalent attack is **not possible with server-side redirects**. In this case, browsers recognize that the request to follow the redirect resulted from a cross-site request initially, so they still apply the appropriate cookie restrictions. 

#### EX: SameSite Strict bypass via client-side redirect

![Screenshot from 2023-08-14 17-20-21](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/930dbd9c-0abe-4d12-9c92-79997b69f40b)
I login and captured the request

![Screenshot from 2023-08-14 17-42-45](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/68f5c7de-46ea-4d87-b0c6-33280ebd974c)
In the login response, the website explicitly specify `SameSite=Strict` restrictions when setting session cookies.

![Screenshot from 2023-08-14 17-21-30](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/7926bb86-e998-4138-b657-d9f31463574d)Then I Changed email and capture request

![Screenshot from 2023-08-14 17-51-16](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/14a28b25-4784-45bc-bec9-79ed4c7e2d0e)
To bypass this restriction(SameSite=Strict) we need to find client-side redirect(`open redirection`) After search I found a page called `/post/x`, you can write a comment on this page, and after writing the comment, it will be sent to `/post/comment/confirmation?postId=x` but, after a few seconds, it's taken back to the blog post.

![Screenshot from 2023-08-14 17-52-32](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/254978d3-21fa-412c-9dcc-3593f726e8ab)
![Screenshot from 2023-08-14 20-42-00](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/e7f830ea-5fba-4c8f-8a2d-b99ec1efedd7)

![Screenshot from 2023-08-14 17-52-36](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/65399416-e644-43a6-857e-a9c4e06212dd)
![Screenshot from 2023-08-14 17-53-14](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/1281f5f5-0740-46a3-a749-ab45fbffc7bc)
Let's capture this request

![Screenshot from 2023-08-14 20-43-02](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/db035f68-62d7-4e9a-bcf2-3a82af170b8a)

```js
redirectOnConfirmation = (blogPath) => {
    setTimeout(() => {
        const url = new URL(window.location);
        const postId = url.searchParams.get("postId");
        window.location = blogPath + '/' + postId;
    }, 3000);
}
```

I found in this request some JavaScript code responsable about redirection

in this code there are line 

```js
window.location = blogPath + '/' + postId;
```

you can write any path in `postId` variable and the variable in take his value from url

```log
/post/comment/confirmation?postId=4
```

then let's Try path traversal with this `../../`

```log
https://example.com/post/comment/confirmation?postId=../../
```

![Screenshot from 2023-08-14 17-51-16](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/14a28b25-4784-45bc-bec9-79ed4c7e2d0e)

in this request right click and copy url

![Screenshot from 2023-08-14 20-54-14](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/aa9a8639-0172-4584-9400-729d7fe3e275)
![Screenshot from 2023-08-14 20-54-25](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/f2d05e9f-ba52-426c-acf7-742c3afb92f2)

As you can see it's redirect us to home page then we found `open redirect` Let's check if the change email page support `GET` method
![Screenshot from 2023-08-14 17-55-50](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/d76d6844-cf2d-400b-b7d6-4d2f8a428ee3)
It's support then let's create our payload Like this 

```html
<script>
    document.location = "https://example.com/post/comment/confirmation?postId=1/../../my-account/change-email?email=hacker%40hack.com%26submit=1";
</script>
```

![Screenshot from 2023-08-14 17-56-42](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/308405e2-1bf0-48e3-a91c-42d78f0c8ef7)
After prepar our script let's check it in exploit server
![Screenshot from 2023-08-14 17-58-07](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/14bbdcd9-d4e3-41dd-93c3-dbe83e0e0452)
![Screenshot from 2023-08-14 17-58-15](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/09dba82f-ba54-417d-b1f1-dbdbeed422be)
And It's work

![Screenshot from 2023-08-14 18-02-16](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/cfe04ebb-641a-4c4b-823d-c933a01bc62c)

then let's send it to victim to solve the lab

![Screenshot from 2023-08-14 21-03-59](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/c994aa1a-aed4-403e-a47c-231b98eb5ab1)

### Bypassing SameSite restrictions via vulnerable sibling domains

Whether testing a website or securing one, remember a request can be same-site even if it's cross-origin. Thoroughly audit the attack surface, including sibling domains. Vulnerabilities like **XSS** can compromise defenses, exposing all site domains to cross-site attacks.

Apart from classic CSRF, remember WebSocket support might lead to cross-site WebSocket hijacking (**CSWSH**), like a **CSRF** attack on a **WebSocket handshake**. 

#### EX: SameSite Strict bypass via sibling domain

# To Be Continue
