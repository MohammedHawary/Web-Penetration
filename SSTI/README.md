# What is server-side template injection?

Server-side template injection is when an attacker injects a malicious payload into a template using native template syntax, leading to its execution on the server. These attacks exploit the concatenation of user input directly into templates instead of treating it as data. By injecting template directives, attackers can manipulate the template engine and gain full control over the server. Server-side template injection is more potent than typical client-side attacks as the payloads are delivered and assessed on the server.

## What is the impact of server-side template injection?

Server-side template injection vulnerabilities can lead to various attacks, depending on the template engine and its usage. While rare circumstances may pose no real risk, most often, the impact is severe. At its worst, an attacker can achieve remote code execution, gaining full control of the back-end server for further attacks on internal infrastructure.

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

 Identifying server-side template injection vulnerabilities and crafting a successful attack typically involves the following high-level process. 

![ssti-methodology-diagram](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/fc9bd5ad-30eb-4605-bb5c-cfaf5e880387)

# Tools

- [Tplmap](https://github.com/epinna/tplmap)
