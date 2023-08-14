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

> **Note**
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
> CSRF tokens can be transmitted in various ways, not just as hidden parameters in POST requests. Some applications choose to place CSRF tokens in HTTP headers. The method of token transmission greatly affects the overall security of the mechanism. For more details, refer to the [guidelines on preventing CSRF vulnerabilities](https://portswigger.net/web-security/csrf/preventing)).

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
The server resonse with error that it the method not allowed ,Then let's check if the ssrf token created specialy for this seasion coockie or not by login with another user and take his ssrf token and putting it in burp repeater to attacker user 

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

> **Note**
> 
> The cookie-setting behavior does not even need to exist within the same web application as the `CSRF` vulnerability. Any other application within the same overall `DNS` domain can potentially be leveraged to set cookies in the application that is being targeted, if the cookie that is controlled has suitable scope. For example, a cookie-setting function on `staging.demo.normal-website.com` could be leveraged to place a cookie that is submitted to `secure.normal-website.com`.

#### EX: CSRF where token is tied to non-session cookie

<mark>not ended the Because lab problem</mark>

### CSRF token is simply duplicated in a cookie


