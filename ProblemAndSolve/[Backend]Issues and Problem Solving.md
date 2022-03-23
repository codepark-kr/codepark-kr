## [BE] Issues & Problem Solving

> ### [SOLVED] io.jsonwebtoken.security.WeakKeyException
> ![image](https://user-images.githubusercontent.com/68538569/151972553-1cae33ef-1630-41a9-aa64-98af044175a1.png)  
> **\# HttpStatus** : `500 (Internal Server Error)`  
> **\# Trace** : The specified key byte array is 248 bits which is not secure enough for any JWT HMAC-SHA algorithm.  The JWT JWA Specification (RFC 7518, Section 3.2) states that keys used with HMAC-SHA algorithms MUST have a size >= 256 bits.  
> **\# Cause** : The count of characters for secretKey that I generated was too short. If I choose `HS256` as Signature Algorithm, the bytes of secret key must be 256 bytes or larger then that. Also If I choose `HS512`, the secret key must be 512bytes or larger.        
> **\# Solution** : 1. Increase the secret key in `application.yml`. 2. change the Signature Algorithm from `HS512` to `HS256`.