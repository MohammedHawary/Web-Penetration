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

## Detect

As with any vulnerability, the first step towards exploitation is being able to find it. Perhaps the simplest initial approach is to try fuzzing the template by injecting a sequence of special characters commonly used in template expressions, such as

`${{<%[%'"}}%\`. If an exception is raised, this indicates that the injected template syntax is potentially being interpreted by the server in some way. This is one sign that a vulnerability to server-side template injection may exist. 

Server-side template injection vulnerabilities occur in two distinct contexts, each of which requires its own detection method. Regardless of the results of your fuzzing attempts, it is important to also try the following context-specific approaches. If fuzzing was inconclusive, a vulnerability may still reveal itself using one of these approaches. Even if fuzzing did suggest a template injection vulnerability, you still need to identify its context in order to exploit it. 

    ${{<%[%'"}}%\
    {{7*7}}
    ${7*7}
    <%= 7*7 %>
    ${{7*7}}
    #{7*7}
    *{7*7}

### Plaintext context

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

### Code context

In some instances, user input, as demonstrated in our earlier email example, can expose vulnerabilities when inserted into a template expression. This might involve placing a user-controllable variable name inside a parameter.

```php
greeting = getQueryParameter('greeting')
engine.render("Hello {{"+greeting+"}}", data)
```

On the website, the resulting URL would be something like:

```url
http://vulnerable-website.com/?greeting=data.username
```

This would be rendered in the output to `Hello Carlos`, for example.

This context is often overlooked in assessments as it doesn't lead to evident XSS and closely resembles a straightforward hashmap lookup. To test for server-side template injection here, one approach is to confirm that the parameter lacks a direct XSS vulnerability by injecting arbitrary HTML into the value:

```url
http://vulnerable-website.com/?greeting=data.username<tag>
```

Without XSS, this typically leads to either an empty entry in the output (only "Hello" without the username), encoded tags, or an error message. The subsequent step involves trying to escape the statement using standard templating syntax and injecting arbitrary HTML afterward.

```url
http://vulnerable-website.com/?greeting=data.username}}<tag>
```

If encountering an error or blank output after attempting this indicates either the use of incorrect syntax for the templating language or, if no valid template-style syntax is apparent, server-side template injection is likely impossible. Conversely, if the output displays correctly alongside the injected arbitrary HTML, it strongly suggests the presence of a server-side template injection vulnerability.

```html
Hello Carlos<tag>
```

### Identify

Once you have detected the template injection potential, the next step is to identify the template engine.

Although there are a huge number of templating languages, many of them use very similar syntax that is specifically chosen not to clash with HTML characters. As a result, it can be relatively simple to create probing payloads to test which template engine is being used.

Simply submitting invalid syntax is often enough because the resulting error message will tell you exactly what the template engine is, and sometimes even which version. For example, the invalid expression `<%=foobar%>` triggers the following response from the Ruby-based ERB engine:

```log
(erb):1:in `<main>': undefined local variable or method `foobar' for main:Object (NameError)
from /usr/lib/ruby/2.5.0/erb.rb:876:in `eval'
from /usr/lib/ruby/2.5.0/erb.rb:876:in `result'
from -e:4:in `<main>'
```

Otherwise, manual testing involves trying various language-specific payloads and examining their interpretation by the template engine. Employing a process of elimination based on apparent validity or invalidity of syntax helps narrow down options quickly. A common approach is injecting arbitrary mathematical operations using syntax from different template engines and observing their successful evaluation. For assistance, a decision tree, like the one below, can be beneficial:

![template-decision-tree](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/b0756980-2d94-4951-965f-229f7c045058)

You should be aware that the same payload can sometimes return a successful response in more than one template language. For example, the payload `{{7*'7'}}` returns `49` in `Twig` and `7777777` in `Jinja2`. Therefore, it is important not to jump to conclusions based on a single successful response.

| `${}`       | `{{}}`       | `<%= %>`        |
| ----------- | ------------ | --------------- |
| `${7/0}`    | `{{7/0}}`    | `<%= 7/0 %>`    |
| `${foobar}` | `{{foobar}}` | `<%= foobar %>` |
| `${7*7}`    | `{{7*7}}`    | ``              |

### Exploit

After detecting that a potential vulnerability exists and successfully identifying the template engine, you can begin trying to find ways of exploiting it.

# Exploiting server-side template injection vulnerabilities

Once you discover a server-side template injection vulnerability, and identify the template engine being used, successful exploitation typically involves the following process.

## Read

Unless you're already familiar with the template engine, consulting its documentation is typically the initial step. While not the most thrilling use of time, it's crucial not to underestimate the valuable information that documentation can provide.

### Learn the basic template syntax

Mastering fundamental syntax, essential functions, and variable handling is crucial. Even acquiring the skill to embed native code blocks in a template can swiftly lead to an exploit. For instance, with knowledge that the Python-based `Mako` template engine is in use, achieving remote code execution could be as straightforward as:

```python
<%
    import os
    x=os.popen('id').read()
    %>
    ${x}
```

In an unsandboxed environment, achieving remote code execution and using it to read, edit, or delete arbitrary files is similarly as simple in many common template engines.

#### EX1: Basic server-side template injection

![Screenshot 2023-12-31 094728](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/5f7dd076-a44f-4553-bc9a-38754604b8cc)
![Screenshot 2023-12-31 094759](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/5c051801-4eba-4ae2-8eb6-fc5336879abf)
![Screenshot 2023-12-31 094809](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/d66a1d9b-be19-4fb6-9091-70fa397cbed6)
![Screenshot 2023-12-31 094835](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/8650faee-c3ba-4d2c-bffc-7d4c09ebfa7b)
![Screenshot 2023-12-31 100202](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/9e8999b2-ca4d-4cab-946a-dcd4deb2a700)
![Screenshot 2023-12-31 100253](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/3ee5cc70-e7ea-42e3-8307-f06a0efdd3ea)





###### ERB (Ruby)

- `{{7*7}} = {{7*7}}`
- `${7*7} = ${7*7}`
- `<%= 7*7 %> = 49`
- `<%= foobar %> = Error`

```
<%= system("whoami") %>
<%= Dir.entries('/') %>
<%= File.open('/path/to/file.txt').read %>
```

# Tools

- [Tplmap](https://github.com/epinna/tplmap)
