---

---

## Username enumeration via different responses

Website error messages are great resources for collating this information to build our list of valid usernames.

How Enumerate UserNames From Login Page

1. Create User form create a new 

2. then try to login with userName can't be any one sign up with it and uncorrect password Like `UserName`:`ASdqweasds`|`Password`:`12346745` 

3. copy the error Message and save it in text file 

4. and Login With The User Account that You Create It but with Uncorrect Password

5. copy the error Message and save it in text file

6. compare between them if there are any different between them then this user is already existing 
   
   EX:
   
   first Message       `user Name or password was incorrect.`
   
   second Message `user Name or password was incorrect`
   
   the different between two error Message is the dot `.`
   
   you Can Brute Force It With `Burp Intruder` or with `ffuf`
   
   ```bash
   ffuf -w /usr/share/wordlists/seclists/Usernames/Names/names.txt -X POST -d "username=FUZZ&email=x&password=x&cpassword=x" -H "Content-Type: application/x-www-form-urlencoded" -u http://10.10.127.105/customers/signup -mr "username already exists"ffuf -w /usr/share/wordlists/seclists/Usernames/Names/names.txt -X POST -d "username=FUZZ&email=x&password=x&cpassword=x" -H "Content-Type: application/x-www-form-urlencoded" -u http://10.10.127.105/customers/signup -mr "username already exists"
   ```

### <mark>HINT</mark>

Identify that the `X-Forwarded-For` header is supported, which allows you to spoof your IP address and bypass the IP-based brute-force protection

## Username enumeration via response timing

 Notice that one of the response times was significantly longer than the others. Repeat this request a few times to make sure it consistently takes longer, then make a note of this username. 

## Vulnerabilities in multi-factor authentication (2FA)

- Log in to your own account. This is the first step to establish a baseline for what the normal login process looks like.

- Your 2FA verification code will be sent to you by email. Click the Email client button to access your emails. This is simulating the process of receiving a 2-factor authentication code via email, which is a common method used by many online platforms.

- Go to your account page and make a note of the URL. This is to help participants understand how the URL of the account page is structured, which will be important for the next step.
  
  EX: `https://testing.test.com/my-account?id=Hawary`

- Log out of your account.

- Log in using the victim's credentials. This is the key step where participants attempt to gain unauthorized access to another user's account by using stolen credentials (ex: username:password).

- When prompted for the verification code, manually change the URL to navigate to /my-account. This is where the 2FA phishing technique comes into play. By changing the URL to a different page in the account system (in this case, the "my-account" page), the system may prompt the user to enter their 2FA code again. If the attacker is able to intercept this code (e.g. by using a fake login page or by social engineering the victim), they can use it to gain access to the account.

### Brute Force

Using Hydra

```bash
hydra 10.10.127.105 http-form-post "/customers/login:username=^USER^&password=^PASS^:Invalid Username/Password Combination" -L Names.txt -P /usr/share/wordlists/seclists/Passwords/Common-Credentials/10-million-password-list-top-100.txt -t 10 -w 30
```

Using ffuf

```bash
ffuf -w valid_usernames.txt:W1,password.txt:W2 -X POST -d "username=W1&password=W2" -H "Content-Type: application/x-www-form-urlencoded" -u http://10.10.127.105/customers/login -fc 200
```
