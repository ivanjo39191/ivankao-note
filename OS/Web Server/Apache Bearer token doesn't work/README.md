# Apache Bearer token doesn't work

Created: February 1, 2022 5:22 PM
Tags: OS

```python
RewriteEngine On 
RewriteCond %{HTTP:Authorization} ^(.*) 
RewriteRule .* - [e=HTTP_AUTHORIZATION:%1]
```

[https://stackoverflow.com/questions/40225826/can-apache-prevent-bearer-authentication-header](https://stackoverflow.com/questions/40225826/can-apache-prevent-bearer-authentication-header)

[https://stackoverflow.com/questions/34036781/oauth2-header-authorization-bearer-token-doesnt-work](https://stackoverflow.com/questions/34036781/oauth2-header-authorization-bearer-token-doesnt-work)