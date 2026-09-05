For ecommerce websites search for idor in porducts category
In crm websitres search for idor in users mgmt
https://youtu.be/lfFfJTEFK4Y
https://youtu.be/gINAtzdccts

Unprotected Functionality
Lab 1:

Got the admin panel directory just by adding `robots.txt` at the end of the home url
![[Pasted image 20260905222201.png]]
![[Pasted image 20260905222335.png]]

Lab 2:
In some cases, sensitive functionality is concealed by giving it a less predictable URL. This is an example of so-called "security by obscurity". However, hiding sensitive functionality does not provide effective access control because users might discover the obfuscated URL in a number of ways.

eg- https://insecure-website.com/administrator-panel-yb556

Captured the home page request and sent it to repeater.
And went through the response and found the unique admin url
Pasted it in the browser and got the admin panel access
![[Pasted image 20260905223311.png]]![[Pasted image 20260905223342.png]]

Parameter Based Access Control Methods
Lab 3:
Some applications determine the user's access rights or role at login, and then store this information in a user-controllable location. This could be:

- A hidden field.
- A cookie.
- A preset query string parameter.

The application makes access control decisions based on the submitted value. For example:

`https://insecure-website.com/login/home.jsp?admin=true https://insecure-website.com/login/home.jsp?role=1`

This approach is insecure because a user can modify the value and access functionality they're not authorized to, such as administrative functions.

Steps to Recreate:

Captured the login request and sent it to repeater
Then changed the value in cookie parameter 
`Admin=false` to `Admin=true` and sent the response 
`Cookie: Admin=true; session=SJZocg7aMoXvjAV9QIOrWTHbQu1Fn7YF`

Now the admin panel was visible in the response code so copy the request url and open it in different tab

==Note: If you copy the response url you'll only get the admin panel ui but wont get directed to the admin panel==
![[Pasted image 20260905225545.png]]![[Pasted image 20260905225832.png]]
Or we could do this way as well

![[Pasted image 20260905233541.png]]

Lab 4:
