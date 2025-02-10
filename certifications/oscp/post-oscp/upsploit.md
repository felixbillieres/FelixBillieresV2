---
description: Community Rating:Very hard
---

# Upsploit

So i start with scanning for open ports:

<figure><img src="../../../.gitbook/assets/image (86).png" alt=""><figcaption></figcaption></figure>

Ok so the webpage is an upload file system:

<figure><img src="../../../.gitbook/assets/image (87).png" alt=""><figcaption></figcaption></figure>

I try to upload my reverse shell and visit the uploads folder:

<figure><img src="../../../.gitbook/assets/image (88).png" alt=""><figcaption></figcaption></figure>

So php is not executed, i try LFI to read /etc/passwd file but it does not work

since every file i upload goes in /uploads maybe i can inject a .htaccess and change the backend of the app:

{% embed url="https://thibaud-robin.fr/articles/bypass-filter-upload/" %}

```php
AddType application/x-httpd-php .txt      # Say all file with extension .txt will execute php

php_value zend.multibyte 1                  # Active specific encoding (you will see why after :D)
php_value zend.detect_unicode 1             # Detect if the file have unicode content
php_value display_errors 1                  # Display php errors
```

so i do a reverse.txt file and try to upload but this does not work...&#x20;

<figure><img src="../../../.gitbook/assets/image (89).png" alt=""><figcaption></figcaption></figure>

will come back later
