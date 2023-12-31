# Server-side request forgery (SSRF)

Server-side request forgery (SSRF) is a web security vulnerability that enables attackers to make the server-side application send requests to unintended locations, possibly exposing sensitive data or connecting to internal-only services.

## What is the impact of SSRF attacks?

A successful SSRF attack can lead to unauthorized actions or data access within the organization, either in the vulnerable application or on connected back-end systems. In some cases, attackers can execute arbitrary commands. Additionally, exploiting SSRF to connect with external systems may lead to malicious attacks that seem to come from the vulnerable organization.

## Common SSRF attacks

SSRF attacks exploit trust relationships to escalate the attack and perform unauthorized actions. These trust relationships can be with the server or other back-end systems in the organization.

### SSRF attacks against the server itself

In an `SSRF` attack against the server itself, the attacker induces the application to make an HTTP request back to the server through its `loopback` network interface. This is achieved by supplying a URL with a hostname like `127.0.0.1` or `localhost`, which points to the `loopback` adapter. For example, a shopping application querying back-end REST APIs to check stock information may be vulnerable to such attacks, their browser makes a request like this: 

```log
POST /product/stock HTTP/1.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 118

stockApi=http://stock.weliketoshop.net:8080/product/stock/check%3FproductId%3D6%26storeId%3D1
```

This causes the server to make a request to the specified URL, retrieve the stock status, and return this to the user.

In this situation, an attacker can modify the request to specify a URL local to the server itself. For example: 

```log
POST /product/stock HTTP/1.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 118

stockApi=http://localhost/admin
```

 Here, the server will fetch the contents of the /admin URL and return it to the user. 

The attacker could directly visit the `/admin` URL, but it is typically accessible only to authenticated users. Visiting the URL directly won't show anything of interest. However, if the request to the `/admin `URL comes from the local machine, normal access controls are bypassed. The application grants full access to the administrative functionality because the request seems to originate from a trusted location.

#### EX: Basic SSRF against the local server

![Screenshot from 2023-08-07 07-51-50](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/3c550ff3-28f8-4626-a2f7-4d7ba5143e95)
After capture this stock request 

![Screenshot from 2023-08-07 07-51-57](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/d071f4f2-41fa-470f-9ee6-95dd349e13da)

i show parameter called `stockApi` and have a value url let's try access the local admin page

![Screenshot from 2023-08-07 09-42-06](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/d4f83fc5-3848-4d25-9814-215c716430e6)

It's work we can access the /admin page in localhost

![Screenshot from 2023-08-07 07-52-40](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/8dd25307-9de1-4f1f-9a26-6f71751103ab)
let's delete carlos

![Screenshot from 2023-08-07 07-54-29](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/4465ae9a-18ec-4a58-9fd8-26e1782b23b9)

![Screenshot from 2023-08-07 07-56-34](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/3780ce31-a829-49e7-afab-115361dc458e)

---

Why do applications behave in this way, and implicitly trust requests that come from the local machine? This can arise for various reasons:

1. The access control check might be in a separate component before the application server, and when a connection is made back to the server, the check is bypassed.
2. For disaster recovery, the application allows administrative access without login for any user from the local machine, assuming only trusted users would access the server directly.
3. The administrative interface might be on a different port, making it unreachable for ordinary users.

These trust relationships, treating local machine requests differently, are what make SSRF a critical vulnerability.

### SSRF attacks against other back-end systems

In SSRF, another trust relationship involves the application server interacting with non-routable, internal back-end systems with weaker security. These systems often have private IP addresses and contain sensitive functionality accessible without authentication.

For example, if there's an administrative interface at the back-end URL

```url
https://192.168.0.68/admin
```

an attacker can exploit the SSRF vulnerability to access it by submitting the following request:

```log
POST /product/stock HTTP/1.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 118

stockApi=http://192.168.0.68/admin
```

#### EX: Basic SSRF against another back-end system

![Screenshot from 2023-08-07 07-51-50](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/3c550ff3-28f8-4626-a2f7-4d7ba5143e95)

After capture this stock request 

![Screenshot from 2023-08-07 07-51-57](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/d071f4f2-41fa-470f-9ee6-95dd349e13da)
i show parameter called `stockApi` and have a value url let's try access the local admin page

![Screenshot from 2023-08-07 09-52-23](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/60105895-6eaa-4dc4-b826-aa9b05a69ce8)
dosent work ,Let's check if this `/admin` page hosted in another ip address then let's bruteforce it with burp intruder

![Screenshot from 2023-08-07 09-53-10](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/af030c5a-501c-43ba-9226-d500529e21f6)
and the `/admin` hosted in 192.168.0.108 let's try delete carlos

![Screenshot from 2023-08-07 09-54-03](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/d6272653-4d73-40e6-87fd-a26ca0bb431f)

![Screenshot from 2023-08-07 09-55-28](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/dba2cb57-a3b4-474f-b760-c92cdd52ed79)

## Circumventing common SSRF defenses

 It is common to see applications containing SSRF behavior together with defenses aimed at preventing malicious exploitation. Often, these defenses can be circumvented. 

### SSRF with blacklist-based input filters

Some applications block input containing hostnames like `127.0.0.1` and `localhost`, or sensitive URLs like `/admin`. In this situation, you can often circumvent the filter using various techniques: 

- Using an alternative IP representation of `127.0.0.1`, such as `2130706433`, `017700000001`, or `127.1`.

- Registering your own domain name that resolves to `127.0.0.1`. You can use `spoofed.burpcollaborator.net`  or `127.0.0.1.nip.io` for this purpose.

- Obfuscating blocked strings using URL encoding or case variation. 

- Use a URL under your control that redirects to the target URL. Experiment with various redirect codes and protocols, like switching from `http:` to `https:`, as this can bypass certain `anti-SSRF `filters.

#### EX: SSRF with blacklist-based input filter

![Screenshot from 2023-08-07 10-14-35](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/2c2b1ee6-a64e-4ccc-b1b0-5341be8eba71)
after captuer the request when we press `check stock`

![Screenshot from 2023-08-07 10-14-45](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/0125afe0-eb50-4062-8b5b-78686c32dd44)
i show parameter called `stockApi` and have a value url let's try access the local host
![Screenshot from 2023-08-07 12-43-00](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/1db8f4c7-f300-49c3-a686-4582d7de89c9)

as we can see that the request is blocked ,After some tries bypassed this block by changing the `127.0.0.1` to: `127.1`

![Screenshot from 2023-08-07 12-43-12](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/45dd68ef-38f0-48fe-8ef7-3b90a46307e6)
Let's Access the `/admin` path

![Screenshot from 2023-08-07 12-43-20](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/642e6a6f-52ee-4ff0-8c23-bd8382e93846)
as you can see that the URL is blocked again.let's try to bypass it by double-URL encoding first char `a`  to `%25%36%31` to access the `/admin`

![Screenshot from 2023-08-07 12-43-53](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/1a2d2fde-051f-4baa-84be-51e20d18f523)

as you can see it's work then lets delete user carlos

![Screenshot from 2023-08-07 12-46-23](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/8eb7804e-37be-461c-8aa9-2ae5600f4671)

### SSRF with whitelist-based input filters

Certain applications only accept input that matches or contains whitelisted values. In such cases, you can exploit inconsistencies in URL parsing to bypass the filter. The URL specification contains features that are often overlooked during ad hoc parsing and validation of URLs.

- You can embed credentials in a URL before the hostname, using the `@` character. For example:
  
      https://expected-host:fakepassword@evil-host

- You can use the `#` character to indicate a URL fragment. For example:
  
      https://evil-host#expected-host

- You can leverage the DNS naming hierarchy to place required input into a fully-qualified DNS name that you control. For example:
  
      https://expected-host.evil-host

- URL-encoding characters can confuse URL-parsing code, especially when the filter and back-end HTTP request code treat URL-encoded characters differently. Double-encoding characters may also be attempted, as some servers recursively URL-decode input, creating further discrepancies.

- You can use combinations of these techniques together. 

#### EX: SSRF with whitelist-based input filter

![Screenshot from 2023-08-07 22-46-57](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/e315340a-4396-422d-a16b-8ed5ac2314b6)
after captuer the request when we press `check stock`

![Screenshot from 2023-08-07 22-47-11](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/3706225d-ceed-4576-b848-d5b1cb4ad23e)
i show parameter called `stockApi` and have a value url let's try access the local host

![Screenshot from 2023-08-07 22-47-35](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/6e9b3fd8-73ff-4cc2-9413-660e4c86ac20)
it response with error message that external stock check host must be `stock.weliketoshop.net` let's try bypass it by this payload

```url
http://127.0.0.1@stock.weliketoshop.net
```

<img title="" src="https://github.com/MohammedHawary/Web-Penetration/assets/94152045/6bec917c-d429-46bf-b676-f8488f0babe1" alt="Screenshot from 2023-08-07 22-48-05" data-align="inline">
it's work then add `#` before `@`

![Screenshot from 2023-08-07 22-48-19](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/87d4e2d2-618c-413c-aed0-4d9b8b08d5ea)
does't work then lets try to URL-encoding the `#` or double URL-encoding

![Screenshot from 2023-08-07 22-48-36](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/7e83518e-a50d-4093-b9ad-6006da3511f1)
It' work but with double URL-encoding then lets try to delete user 

with this payload 

    http://127.0.0.1%25%32%33@stock.weliketoshop.net/admin/delete?username=carlos

![Screenshot from 2023-08-07 22-50-08](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/39e82be0-4040-4994-b7de-2a07ab05318b)

### Bypassing SSRF filters via open redirection

Filter defenses can be bypassed via `open redirect` vulnerabilities. In the prior SSRF case, if user-provided URLs are rigorously checked to prevent SSRF attacks, but the allowed app URLs have an `open redirect` flaw, you can manipulate a URL that meets the filter criteria, leading to a redirected request to the intended backend.

For example, suppose the application contains an open redirection vulnerability in which the following URL:

```url
/product/nextProduct?currentProductId=6&path=http://evil-user.net
```

returns a redirection to:

```url
http://evil-user.net
```

You can leverage the open redirection vulnerability to bypass the URL filter, and exploit the SSRF vulnerability as follows:

```log
POST /product/stock HTTP/1.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 118

stockApi=http://weliketoshop.net/product/nextProduct?currentProductId=6&path=http://192.168.0.68/admin
```

#### EX: SSRF with filter bypass via open redirection vulnerability

![Screenshot from 2023-08-08 01-20-30](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/d7c26750-c90f-4946-a2e9-aee5decc71ce)
after captuer the request when we press `check stock` Visit a product, intercept the request in Burp Suite, and send it to Burp Repeater. 

![Screenshot from 2023-08-08 01-20-40](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/d254cf6d-d5f7-4ea9-8f04-4a5b75f060d0)
i show parameter called `stockApi` and have a value url let's try access the local host
![Screenshot from 2023-08-08 01-22-00](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/335c3b90-3725-4cbd-857f-c484fc68b40b)
Error called that the url is invalid, Let's search about open redirect vulnerability to exploit it to bypass ssrf protiction

![Screenshot from 2023-08-08 01-22-09](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/2663bd04-0b46-4fbb-933f-64050798bf0f)
and i found an `path` parameter i think that the parameter vunerable with

`open redirect`![Screenshot from 2023-08-08 01-22-19](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/04f2b1d4-7c35-43a2-8d08-0dd1c2f85e75)
Let's show if it redirect me or not to `google.com`

![Screenshot from 2023-08-08 01-22-46](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/078b89b6-b549-49e0-a399-fc91679217f1)
yes it's redirect me to `google.com` thin the `path` parameter is vulnerable with `open redirection` then let's copy all path with parameter and add it in `stockApi`

![Screenshot from 2023-08-08 01-23-19](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/fe9d4967-c21c-473f-a778-a1e68905b09e)
this error can bypass by URL-encoding

![Screenshot from 2023-08-08 01-24-38](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/2385e984-450e-467a-86f7-ffb7c97b08bf)

It's work then and we can delete carlos

![Screenshot from 2023-08-08 01-24-46](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/f3af9aaf-48a3-4f3e-8b85-4c26d84d0d8c)

### Blind SSRF vulnerabilities

Blind SSRF vulnerabilities arise when an application can be induced to issue a back-end HTTP request to a supplied URL, but the response from the back-end request is not returned in the application's front-end response.

Blind SSRF is generally harder to exploit but can sometimes lead to full remote code execution on the server or other back-end components. 

### How to find and exploit blind SSRF vulnerabilities

The most reliable way to detect `blind SSRF` vulnerabilities is using out-of-band (OAST) techniques. This involves attempting to trigger an HTTP request to an external system that you control, and monitoring for network interactions with that system.

The easiest and most effective way to use out-of-band techniques is using `Burp Collaborator`. You can use `Burp Collaborator` to generate unique domain names, send these in payloads to the application, and monitor for any interaction with those domains. If an incoming HTTP request is observed coming from the application, then it is vulnerable to `SSRF`. 

> **Note**
> 
> Blind SSRF vulnerabilities arise when an application can be induced to issue a back-end HTTP request to a supplied URL, but the response from the back-end request is not returned in the application's front-end response.
> 
> Blind SSRF is generally harder to exploit but can sometimes lead to full remote code execution on the server or other back-end components.  It is common when testing for SSRF vulnerabilities to observe a DNS look-up for the supplied Collaborator domain, but no subsequent HTTP request. This typically happens because the application attempted to make an HTTP request to the domain, which caused the initial DNS lookup, but the actual HTTP request was blocked by network-level filtering. It is relatively common for infrastructure to allow outbound DNS traffic, since this is needed for so many purposes, but block HTTP connections to unexpected destinations. 

#### EX1: Blind SSRF with out-of-band detection

![Screenshot from 2023-08-08 01-58-47](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/d908528b-d737-432d-a80c-8ddaab99b043)
I visit a product, intercept the request in Burp Suite, and send it to Burp Repeater. 

![Screenshot from 2023-08-08 02-00-30](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/e3d8debb-6202-4aab-a409-99460eb67506)
i show the `referer` header and i tried to inject the `burpcolaboritor` and send this request

![Screenshot from 2023-08-08 01-57-28](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/f1ae0a4c-5c18-4a80-af5d-4ffb36ad379e)
it work and the `referer` header vulnerable with blind ssrf
![Screenshot from 2023-08-08 01-58-14](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/b7f0ac0a-3c12-4bfd-9988-d46e95b78733)

---

 Simply identifying a blind `SSRF` vulnerability that can trigger out-of-band HTTP requests doesn't in itself provide a route to exploitability. Since you **cannot view the response from the back-end request**, the behavior **can't be used to explore content on systems that the application server can reach**. However, it can still be leveraged to probe for other vulnerabilities on the server itself or on other back-end systems. **You can blindly sweep the internal IP address space, sending payloads designed to detect well-known vulnerabilities**. **If those payloads also employ blind out-of-band techniques, then you might uncover a critical vulnerability on an unpatched internal server.**

#### EX3: Blind SSRF with Shellshock exploitation

[Lab: Blind SSRF with Shellshock exploitation | Web Security Academy](https://portswigger.net/web-security/ssrf/blind/lab-shellshock-exploitation)

## Cheat sheet

https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Request%20Forgery/README.md
