---
title: Dectify CS Challenge Walkthrough
published: true
---

Last week a friend of mine suggested a challenge developed by Dectify and asked be to find the vulnerabilities on the application.You can find the link to the challenge [here.](https://github.com/detectify/cs-challenge)

After following the steps provided in the github repository to start the challenge.I was presented with a web page called **Super secure screenshot service**.

![Index page](https://github.com/apoorvaniranjan/apoorvaniranjan.github.io/raw/main/assets/images/dectify-cs/1.png)

What the application does is that it accepts a url and takes a screenshot of the application which can only mean one thing SSRF(Server Side Request Forgery) .

![working screenshot](https://github.com/apoorvaniranjan/apoorvaniranjan.github.io/raw/main/assets/images/dectify-cs/2.png)

Looking at the page there is a tab to go the admin interface of the application clickng on it opened a new tab with a different port which is not avaialble at the moment so my goal is to retreive the screenshot of the admin interface. 

![admin](https://github.com/apoorvaniranjan/apoorvaniranjan.github.io/raw/main/assets/images/dectify-cs/3.png)

So i started by entering the admin interface url to the form which gave me an error that **the domain must contain atleast one letter**

![admin error](https://github.com/apoorvaniranjan/apoorvaniranjan.github.io/raw/main/assets/images/dectify-cs/4.png)

So if the application is expecting a domain name i could bypass it by adding a domain which resolved to localhost do i tried the "bugbounty.dod.network" which resolves to localhost and the application gave me an error stating that "**Domains that resolve to localhost are not allowed**". 

![error2](https://github.com/apoorvaniranjan/apoorvaniranjan.github.io/raw/main/assets/images/dectify-cs/5.png)
![error2](https://github.com/apoorvaniranjan/apoorvaniranjan.github.io/raw/main/assets/images/dectify-cs/6.png)
Which make me think ,the application is expecting an input containg a letter and it should not resolve to localhost so which brought me to the conusion that what if I tried Hexadecimal representation of localhost and it worked i was able to take the screenshot of the application.

![adminpanel](https://github.com/apoorvaniranjan/apoorvaniranjan.github.io/raw/main/assets/images/dectify-cs/7.png)
![adminpanel](https://github.com/apoorvaniranjan/apoorvaniranjan.github.io/raw/main/assets/images/dectify-cs/8.png)
This was the first task the second task was to find a XSS(Cross Site Scripting) vulnerability on the application.The application had only one input so it has to be that input which is vulnerable to XSS. At first i tried the usual xss paylod to check how the application handled that request and the payload was only reflected only on one place which was inside an anchor tag.

![xss1](https://github.com/apoorvaniranjan/apoorvaniranjan.github.io/raw/main/assets/images/dectify-cs/9.png)
![xss1](https://github.com/apoorvaniranjan/apoorvaniranjan.github.io/raw/main/assets/images/dectify-cs/10.png)

As you can see the application converts the tags to html entities and removes the word script, Since it is reflecting inside an anchor tag of the Download Screenshot option we can use the javascript pseudo protocol to trigger an alert and inorder to bypass the regex used in the application the final payload was like this.If we click the hyperlink on the download option our payload is triggered and we will have an alert box.

    javascrSCRIPTipt:alert(1)

![xss1](https://github.com/apoorvaniranjan/apoorvaniranjan.github.io/raw/main/assets/images/dectify-cs/11.png)
![xss1](https://github.com/apoorvaniranjan/apoorvaniranjan.github.io/raw/main/assets/images/dectify-cs/12.png)
![xss1](https://github.com/apoorvaniranjan/apoorvaniranjan.github.io/raw/main/assets/images/dectify-cs/13.png)

The final vulnerability was a bit tricky to find an i had to use the repo to take a look and I found a nginx configuration file containing all the paths used in the application in that i found that the the fire is misconfigured and it is vulnerable to an alias traversal vulnerability.You can find more informaton about this vulnerability [here](https://i.blackhat.com/us-18/Wed-August-8/us-18-Orange-Tsai-Breaking-Parser-Logic-Take-Your-Path-Normalization-Off-And-Pop-0days-Out-2.pdf).

```
server {
listen 80;
root /usr/share/nginx/html;
location / {
include uwsgi_params;
uwsgi_pass vuln-screenshot:5000;
}
location /screenshots {
alias /usr/share/nginx/html/;
}
}
```
The location screenshots is aliased to /usr/share/nginx/html on visiting /screenshots the content inside /html is served in this case it was the default index page of nginx.

![/screenshots](https://github.com/apoorvaniranjan/apoorvaniranjan.github.io/raw/main/assets/images/dectify-cs/14.png)
                       
 Since the location does'nt end with a trailing slash i was able to go back one directory which gave me a 403 error which means that I am now outside the html directory. 

![403](https://github.com/apoorvaniranjan/apoorvaniranjan.github.io/raw/main/assets/images/dectify-cs/15.png)

To exploit such vulnerability one must know the location of the files and folders located inside the directory which can be achieved by bruteforing the location in this exercise the location was app and inside contained the scource code of the screenshot application.

![app.py](https://github.com/apoorvaniranjan/apoorvaniranjan.github.io/raw/main/assets/images/dectify-cs/16.png)

