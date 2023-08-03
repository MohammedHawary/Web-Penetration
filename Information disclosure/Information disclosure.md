# Information Disclosure & leakage

## How to test for information disclosure vulnerabilities

The following are some examples of high-level techniques and tools that you can use to help identify information disclosure vulnerabilities during testing. 

### Fuzzing

If you identify interesting parameters, you can try submitting unexpected data types and specially crafted fuzz strings to see the effect (e.g. potential differences in response time or encountered error cases). Pay close attention, as responses may hint at the application's behavior more subtly than explicitly disclosing interesting information.

You can automate much of this process using tools such as Burp Intruder. This provides several benefits. Most notably, you can:

- Add payload positions to parameters and use pre-built wordlists of fuzz strings to test a high volume of different inputs in quick succession.

- Easily identify differences in responses by comparing HTTP status codes, response times, lengths, and so on.

- Use grep matching rules to quickly identify occurrences of keywords, such as error, invalid, SELECT, SQL, and so on.

- Apply grep extraction rules to extract and compare the content of interesting items within responses.

### Using Burp Scanner

Burp Scanner will alert you if it finds sensitive information such as private keys, email addresses, and credit card numbers in a response. It will also identify any backup files, directory listings, and so on. 

### Using Burp's engagement tools

Burp provides several engagement tools that you can use to find interesting information in the target website more easily. You can access the engagement tools from the context menu - just right-click on any HTTP message, Burp Proxy entry, or item in the site map and go to "Engagement tools".

The following tools are particularly useful in this context. 

### Search

You can use this tool to look for any expression within the selected item. You can fine-tune the results using various advanced search options, such as regex search or negative search. This is useful for quickly finding occurrences (or absences) of specific keywords of interest. 

### Find comments

You can use this tool to quickly extract any developer comments found in the selected item. It also provides tabs to instantly access the HTTP request/response cycle in which each comment was found.  

### Discover content

You can use this tool to identify additional content and functionality that is not linked from the website's visible content. This can be useful for finding additional directories and files that won't necessarily appear in the site map automatically. 

### Engineering informative responses

Verbose error messages can disclose interesting information during testing. Studying how error messages change based on input can help extract arbitrary data via an error message. Various methods can be used depending on the scenario, such as making the application logic attempt an invalid action on specific data. For example, submitting an invalid parameter value may reveal a stack trace or debug response that contains valuable details. In some cases, error messages may even disclose the desired data value in the response.

# Common sources of information disclosure

## Files for web crawlers

Websites often offer `/robots.txt` and `/sitemap.xml` files to aid crawlers in navigating their site. These files frequently include directories that crawlers should avoid due to sensitive information. Since these files are typically not linked from the website, they may not show up in Burp's site map right away. Therefore, it's worthwhile to attempt manual navigation to `/robots.txt` or `/sitemap.xml` to see if any useful information is present.

## Directory listings

Web servers may automatically list directory contents when no index page is present. This enables attackers to quickly identify and target resources at a given path, including sensitive files not intended for user access, such as temporary files and crash dumps. While directory listings are not necessarily a security vulnerability, improper access control can result in the exposure of sensitive resources. In these cases, leaking the existence and location of such resources is a clear issue.

## Developer comments

During development, HTML comments may be added to markup and removed before deploying changes to production. However, comments can be forgotten, missed, or intentionally left in despite security implications. While comments are not visible on the rendered page, they can be accessed through Burp or the browser's developer tools. Comments may contain useful information for attackers, such as hints about hidden directories or application logic.

## Error messages

Verbose error messages commonly cause information disclosure. Pay close attention to all error messages during auditing as they may reveal input or data type expectations and help narrow down attack vectors. Error messages may also expose technologies used by the website, such as **naming a template engine, database type, or server, along with its version number**. This information is useful for searching for documented exploits, common configuration errors, or dangerous default settings to exploit. Differences between error messages can also indicate different application behavior, essential for techniques such as **SQL injection and username enumeration**. Publicly available source code of **open-source frameworks** used by the website can be a valuable resource for constructing exploits.

### EX: Information disclosure in error messages

**https://example.net/product?productId=1**

In this website link there are a parameter called `productId` this parameter expected `interger` numbers only if we give it `string` value it lead to error message this message contain the `apache` server version 

```log
Internal Server Error: java.lang.NumberFormatException: For input string: "a"
	at java.base/java.lang.NumberFormatException.forInputString(NumberFormatException.java:67)
	at java.base/java.lang.Integer.parseInt(Integer.java:668)
	at java.base/java.lang.Integer.parseInt(Integer.java:786)
	at lab.y.o.s.t.y(Unknown Source)
	at lab.s.s3.e.y.R(Unknown Source)
	at lab.s.s3.v.d.n.m(Unknown Source)
	at lab.s.s3.v.c.lambda$handleSubRequest$0(Unknown Source)
	at l.e.f.c.lambda$null$3(Unknown Source)
	at l.e.f.c.O(Unknown Source)
	at l.e.f.c.lambda$uncheckedFunction$4(Unknown Source)
	at java.base/java.util.Optional.map(Optional.java:260)
	at lab.s.s3.v.c.i(Unknown Source)
	at lab.server.a.w.q.P(Unknown Source)
	at lab.s.s3.d.f(Unknown Source)
	at lab.s.s3.d.P(Unknown Source)
	at lab.server.a.w.n.z.d(Unknown Source)
	at lab.server.a.w.n.h.lambda$handle$0(Unknown Source)
	at lab.y.m.s.a.Q(Unknown Source)
	at lab.server.a.w.n.h.o(Unknown Source)
	at lab.server.a.w.f.x(Unknown Source)
	at l.e.f.c.lambda$null$3(Unknown Source)
	at l.e.f.c.O(Unknown Source)
	at l.e.f.c.lambda$uncheckedFunction$4(Unknown Source)
	at lab.server.l.t(Unknown Source)
	at lab.server.a.w.f.j(Unknown Source)
	at lab.server.a.d.b.C(Unknown Source)
	at lab.server.a.s.O(Unknown Source)
	at lab.server.a.g.O(Unknown Source)
	at lab.server.u.w(Unknown Source)
	at lab.server.u.T(Unknown Source)
	at lab.u.r.lambda$consume$0(Unknown Source)
	at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1136)
	at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:635)
	at java.base/java.lang.Thread.run(Thread.java:833)

Apache Struts 2 2.3.31
```

## Debugging data

For debugging purposes,Websites may generate custom error messages and logs containing extensive information about the application's behavior for debugging purposes. While this information is helpful during development, it is also valuable to attackers if leaked in the production environment.

Debug messages can sometimes contain vital information for developing an attack, including: 

- Values for key session variables that can be manipulated via user input

- Hostnames and credentials for back-end components 

- File and directory names on the server 

- Keys used to encrypt data transmitted via the client 

Debugging information might be logged in a distinct file. Access to this file can serve as a reference for understanding the application's runtime state and provide clues on how to manipulate the application state and control the received information with crafted input.

### EX: Information disclosure on debug page

**https://example.com/**

In this website after Directory Fuzzing using `burpsuite Content discovery` or `dirsearch` I find this fle `cgi-bin/phpinfo.php` this file contain all `phpinfo` and this is some info from this file lik Environment section

```log
Variable	Value
GATEWAY_INTERFACE 	CGI/1.1
SUDO_GID 	10000
REMOTE_HOST 	154.178.220.71
USER 	carlos
HTTP_DNT 	1
SECRET_KEY 	x3paqmht6fz4ok6u8wq5ma9pjbq4s57l
HTTP_TE 	trailers
HTTP_SEC_FETCH_USER 	?1
QUERY_STRING 	no value
HOME 	/home/carlos
HTTP_USER_AGENT 	Mozilla/5.0 (X11; Linux x86_64; rv:91.0) Gecko/20100101 Firefox/91.0
HTTP_UPGRADE_INSECURE_REQUESTS 	1
HTTP_ACCEPT 	text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
SCRIPT_FILENAME 	/home/carlos/cgi-bin/phpinfo.php
HTTP_HOST 	0a1700fd0484b891808ef80300dc006f.web-security-academy.net
SUDO_UID 	10000
LOGNAME 	carlos
SERVER_SOFTWARE 	PortSwiggerHttpServer/1.0
HTTP_SEC_FETCH_MODE 	navigate
TERM 	unknown
HTTP_COOKIE 	session=Q1Cn4sWiPcuqpP6haYGqiZfGcCG7lf5V; session=ZfDu6Im3HwMBshjux3Xldpw639oPoLdP
PATH 	/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin
HTTP_ACCEPT_LANGUAGE 	en-US,en;q=0.5
HTTP_REFERER 	https://0a1700fd0484b891808ef80300dc006f.web-security-academy.net/cgi-bin/
SERVER_PROTOCOL 	HTTP/1.1
HTTP_ACCEPT_ENCODING 	gzip, deflate
SUDO_COMMAND 	/usr/bin/sh -c /usr/bin/php-cgi
SHELL 	/bin/bash
REDIRECT_STATUS 	true
HTTP_SEC_FETCH_DEST 	document
SUDO_USER 	academy
REQUEST_METHOD 	GET
PWD 	/home/carlos/cgi-bin
HTTP_SEC_FETCH_SITE 	same-origin
SERVER_PORT 	443
SCRIPT_NAME 	/cgi-bin/phpinfo.php
SERVER_NAME 	10.0.4.200 
```

## User account pages

In user profiles or account pages, sensitive information like **email addresses, phone numbers, API keys**, etc., are often stored. While users typically have access only to their own accounts, certain websites may have **logic flaws** that can be exploited by attackers to access other users' data. For instance, a website might identify which user's account page to display using a user parameter

```log
GET /user/personal-info?user=carlos
```

Websites generally implement measures to prevent attackers from manipulating the user parameter to access arbitrary account pages. However, the logic for retrieving specific data items may not be as robust. For instance, while an attacker may not be able to fully load another user's account page, the logic for fetching and displaying the registered email address might not verify if the user parameter matches the currently logged-in user. Consequently, by modifying the user parameter, an attacker could display the email addresses of arbitrary users on their own account page.

## Source code disclosure via backup files

Gaining source code access facilitates attackers in understanding the application's behavior and launching severe attacks. Source code may contain sensitive data, including **hard-coded API keys and credentials** for back-end access.

Identifying the use of specific open-source technology provides limited access to the source code.

In rare cases, it is possible to make a website expose its own source code. While mapping out a website, you might come across explicit references to source code files. However, requesting these files typically executes the code instead of revealing it as plain text. Yet, certain situations allow you to trick the website into returning the file's contents. For instance, temporary backup files generated by text editors often have distinct indicators like appending a tilde `~` or using a different file extension. Requesting a code file with a backup file extension sometimes enables reading the file's contents in the response.

### EX: Source code disclosure via backup files

**https://example.com/**

In this website after Directory Fuzzing using `burpsuite Content discovery` or `dirsearch` I find this fle `backup/ProductTemplate.java.bak` this file contain java code for accessing database 

```java
package data.productcatalog;

import common.db.JdbcConnectionBuilder;

import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.Serializable;
import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

public class ProductTemplate implements Serializable
{
    static final long serialVersionUID = 1L;

    private final String id;
    private transient Product product;

    public ProductTemplate(String id)
    {
        this.id = id;
    }

    private void readObject(ObjectInputStream inputStream) throws IOException, ClassNotFoundException
    {
        inputStream.defaultReadObject();

        ConnectionBuilder connectionBuilder = ConnectionBuilder.from(
                "org.postgresql.Driver",
                "postgresql",
                "localhost",
                5432,
                "postgres",
                "postgres",
                "zdc54cd7oaor4dpb6iy7a575734bse3s"
        ).withAutoCommit();
        try
        {
            Connection connect = connectionBuilder.connect(30);
            String sql = String.format("SELECT * FROM products WHERE id = '%s' LIMIT 1", id);
            Statement statement = connect.createStatement();
            ResultSet resultSet = statement.executeQuery(sql);
            if (!resultSet.next())
            {
                return;
            }
            product = Product.from(resultSet);
        }
        catch (SQLException e)
        {
            throw new IOException(e);
        }
    }

    public String getId()
    {
        return id;
    }

    public Product getProduct()
    {
        return product;
    }
}

```

## Information disclosure due to insecure configuration

In other cases, developers might forget to disable various debugging options in the production environment. For example, the HTTP `TRACE` method is designed for diagnostic purposes. If enabled, the web server will respond to requests that use the `TRACE` method by echoing in the response the exact request that was received. This behavior is often harmless, but occasionally leads to information disclosure, such as the name of internal authentication headers that may be appended to requests by reverse proxies. 

### EX: Authentication bypass via information disclosure

**https://example.com/**

In this website after Directory Fuzzing using `burpsuite Content discovery` or `dirsearch` I find this Page `/admin` but it contain message say 

`Admin interface only available to local users `

Let's check if the web server allow the TRACE Method to make web server echoing in the response the exact request that was received and he supported this method and this is the response

```log
HTTP/2 200 OK
Content-Type: message/http
X-Frame-Options: SAMEORIGIN
Content-Length: 668
TRACE /admin HTTP/1.1
Host: 0a7a007703ccc5728260b071006c000b.web-security-academy.net
user-agent: Mozilla/5.0 (X11; Linux x86_64; rv:91.0) Gecko/20100101 Firefox/91.0
accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
accept-language: en-US,en;q=0.5
accept-encoding: gzip, deflate
referer: https://0a7a007703ccc5728260b071006c000b.web-security-academy.net/login
dnt: 1
upgrade-insecure-requests: 1
sec-fetch-dest: document
sec-fetch-mode: navigate
sec-fetch-site: same-origin
sec-fetch-user: ?1
te: trailers
cookie: session=0vsaLOpqAfkvaqJPePQXahPgOsJtLdXm
Content-Length: 0
X-Custom-IP-Authorization: 154.178.220.71
```

and there are a header called `X-Custom-IP-Authorization` this header containing our  IP address,and it was automatically appended to our request. This is used to determine whether or not the request came from the `localhost` IP address,Let's add it in our request and write local ip `X-Custom-IP-Authorization: 127.0.0.1` and after adding this header we succesfully accessing the admin panel without admin credential

## Version control history

Most websites use version control systems like `Git`, which stores its data in a folder called `.git`. Sometimes this folder is exposed in the production environment, allowing access by browsing to `/.git`. To download the entire `.git` directory, there are various methods available, although manually browsing the file structure can be impractical. Once downloaded, the directory can be opened in `Git` to access the website's version control history, including logs of committed changes. While this may not provide access to the full source code, it can allow reading of small code snippets and the potential discovery of sensitive data hard-coded within changed lines.

### EX: Information disclosure in version control history

**https://example.com/**

In this website after Directory Fuzzing using `burpsuite Content discovery` or `dirsearch` I find this Page `/.git` to download this directory use wget tool

```bash
wget -r https://example.com/.git
```

then use `git-cola` tool to open it to install it use this command 

```bash
sudo apt install git-cola
```

and i found this file `admin.conf` contain the admin password

<u>Note</u>:

in `git-cola` there are option to undo the last commit if there are any file deleted  

[Information disclosure in version control history (Video solution, Audio) - YouTube](https://youtu.be/4Zt71Il1omc)
