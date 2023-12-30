# What is server-side template injection?

Server-side template injection is when an attacker injects a malicious payload into a template using native template syntax, leading to its execution on the server. These attacks exploit the concatenation of user input directly into templates instead of treating it as data. By injecting template directives, attackers can manipulate the template engine and gain full control over the server.

## What is the impact of server-side template injection?

Server-side template injection vulnerabilities can lead to various attacks, depending on the template engine and its usage. At its worst, an attacker can achieve remote code execution, gaining full control of the server

Even without full code execution, server-side template injection serves as a foundation for multiple attacks, potentially providing an attacker with read access to sensitive data and arbitrary files on the server.

## How do server-side template injection vulnerabilities arise?

Server-side template injection vulnerabilities occur when user input is directly concatenated into templates instead of being treated as data.

Static templates, which merely have placeholders for dynamic content, are typically not susceptible to server-side template injection. An example is an email template using Twig to greet users by their name.

```php
$output = $twig->render("Dear {first_name},", array("first_name" => $user.first_name) );
```

This is not vulnerable to server-side template injection because the user's first name is merely passed into the template as data.

However, as templates are simply strings, web developers sometimes directly concatenate user input into templates prior to rendering. Let's take a similar example to the one above, but this time, users are able to customize parts of the email before it is sent. For example, they might be able to choose the name that is used:

```php
$output = $twig->render("Dear " . $_GET['name']);
```

In this case, rather than a static value, a portion of the template is dynamically generated using the `GET` parameter `name`  Since template syntax is evaluated server-side, this creates an opportunity for an attacker to insert a server-side template injection payload within the `name` parameter.

```url
http://vulnerable-website.com/?name={{bad-stuff-here}}
```

# Constructing a server-side template injection attack

 Identifying server-side template injection vulnerabilities and crafting a successful attack typically involves the following high-level process. ![ssti-methodology-diagram](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/fc9bd5ad-30eb-4605-bb5c-cfaf5e880387)

### Detect

As with any vulnerability, the first step towards exploitation is being able to find it. Perhaps the simplest initial approach is to try fuzzing the template by injecting a sequence of special characters commonly used in template expressions, such as

`${{<%[%'"}}%\`. If an exception is raised, this indicates that the injected template syntax is potentially being interpreted by the server in some way. This is one sign that a vulnerability to server-side template injection may exist. 

Server-side template injection vulnerabilities occur in two distinct contexts, each of which requires its own detection method. Regardless of the results of your fuzzing attempts, it is important to also try the following context-specific approaches. If fuzzing was inconclusive, a vulnerability may still reveal itself using one of these approaches. Even if fuzzing did suggest a template injection vulnerability, you still need to identify its context in order to exploit it. 

#### Plaintext context

Template languages often permit free content input through HTML tags or the template's native syntax, which is rendered to HTML on the back-end before the HTTP response is sent. For instance, in Freemarker, the line `render('Hello ' + username)` would render as something like `Hello Carlos`.

While this might be mistaken for a simple XSS vulnerability, it can potentially be exploited for XSS. By setting mathematical operations as the parameter value, one can test if this is also a potential entry point for a server-side template injection attack.

For example, consider a template that contains the following vulnerable code:

```java
render('Hello ' + username)
```

During auditing, we might test for server-side template injection by requesting a URL such as:

```url
http://vulnerable-website.com/?username=${7*7}
```

If the output includes `Hello 49`, it indicates the server-side evaluation of the mathematical operation, serving as a proof of concept for a server-side template injection vulnerability.

It's important to note that the syntax needed to execute the mathematical operation successfully varies based on the template engine in use.

#### Code context





























# Tools

- [Tplmap](https://github.com/epinna/tplmap)
