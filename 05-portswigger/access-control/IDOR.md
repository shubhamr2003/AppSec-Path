For ecommerce websites search for idor in porducts category
In crm websitres search for idor in users mgmt
- https://youtu.be/lfFfJTEFK4Y
- https://youtu.be/gINAtzdccts

Unprotected Functionality
Lab 1:

Got the admin panel directory just by adding `robots.txt` at the end of the home url
![robots](./images/robots-txt.png)

![admin panel](./images/admin-panel.png)

Lab 2:
In some cases, sensitive functionality is concealed by giving it a less predictable URL. This is an example of so-called "security by obscurity". However, hiding sensitive functionality does not provide effective access control because users might discover the obfuscated URL in a number of ways.

eg- https://insecure-website.com/administrator-panel-yb556

Captured the home page request and sent it to repeater.
And went through the response and found the unique admin url
Pasted it in the browser and got the admin panel access
![unique admin path](./images/unique-admin-path.png)![unique admin](./images/unique-admin-panel.png)

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
![ccokie para](./images/editable-cookie-para.png)![success](success.png)
Or we could do this way as well

![alt](./images/alt-method.png)

Lab 4:
A **Role ID** in an HTTP request is usually an identifier that tells the server **which role/permission set a user or resource is associated with**.

For example, you might see:

```http
POST /api/users/update
Content-Type: application/json
Authorization: Bearer eyJ...

{
  "userId": 123,
  "roleId": 2
}
```

Here:

```text
userId = 123
roleId = 2
```

The server might have a database like:

|roleId|Role|
|--:|---|
|1|Admin|
|2|User|
|3|Manager|

So `roleId: 2` could mean the user has the **User** role.

### Why is this important in API pentesting?

Since you're focusing on **Broken Access Control**, `roleId` is an interesting parameter to test.

Suppose your normal request is:

```http
PUT /api/users/123
```

```json
{
  "name": "Shubham",
  "roleId": 2
}
```

You might investigate what happens if you change:

```json
"roleId": 2
```

to:

```json
"roleId": 1
```

If `1` represents Admin and the server accepts the change **without properly checking whether you're authorized to assign the Admin role**, that could indicate a broken access-control / mass-assignment issue.

### But remember

A `roleId` is **not inherently a security vulnerability**.

The important question is:

> **Does the server properly authorize the action associated with that role ID?**

Also, don't assume `roleId=1` is always Admin. The meaning is application-specific.

In Burp, when you see something like:

```http
roleId=3
```

or:

```json
{
    "roleId": 3
}
```

think:

**"This might be controlling a user's privilege level. I should understand what it represents and whether the server enforces authorization."**

Steps:
Crawl the website looking for something like
{
  "userId": 123,
  "roleId": 2
}

Captured the login request
Captured the item request
Then captured the change email request and found

`{`
  `"username": "wiener",`
  `"email": "wienr@abc.com",`
  `"apikey": "6HzbZcjO8pi0p6f6SAZqZJSGQK067MA4",`
  `"roleid": 2`
`}`

in response and knew immediately i had to change the `roleid:1` to `roleid:2`

`{`
`"email":"wienr@abc.com",`
`"roleid":2`
`}`
![roleid](./images/role-id.png)



#### Broken access control resulting from platform misconfiguration
Lab 5:

Change the URL in the request line to `/` and add the HTTP header `X-Original-URL: /invalid`
`X-Original-URL: /admin`

![override header](./images/override-header.png)![override header2](./images/override-header2.png)

But it says access denied so I change the request to
`X-Original-Url: /admin/delete?username=carlos`

and get a response:

`HTTP/2 400 Bad Request`
`Content-Type: application/json; charset=utf-8`
`X-Frame-Options: SAMEORIGIN`
`Content-Length: 30`

`"Missing parameter 'username'"`

So we just add `?username=carlos` into  the main url

`GET /?username=carlos HTTP/2`
`Host: 0ae700e1030141168026a81e00db0046.web-security-academy.net`
`X-Original-Url: /admin/delete`

And send to follow the redirection
![override header3](./images/override-header3.png)


##### Vertical Privilege escalation
Lab 6:
Method-based access control

So I captured the promotion request of user Carlos 
And login request of user Wiener

And swapped the cookie id of Carlos with Wiener
![cookie-tampering](./images/cookie-tampering.png)

It shows unauthorized, since the user name at the bottom of the request is still Carlos

![carlos](./images/carlos.png)

Now we try to change the user name

![wiener](./images/wiener.png)

We still get 401 unauthorized

Now since the lab contains HTTP method based access control flaw, we'll try to change the method parameter in the request

![method](./images/method.png)

Which means that the request allows methods to be ...
Right click and change the method and change the username to wiener

![change-method](./images/change-method.png)

Copying the request link and opening it in a new tab gives

![solved](./images/solved.png)
