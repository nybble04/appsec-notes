# Representational State Transfer (REST) API Testing

## Enumerating API endpoints
The first step for API testing is to find the endpoints and their expected behaviour.
- See how endpoints are constructed
  ```
  # Paths:
  /api, /api/v1, /v1, /v2, /v3, /rest, /swagger,
  /swagger.json, /doc, /docs, /graphql, /graphiql, /altair, /playground

  # Subdomains:
  api.target-name.com
  dev.target-name.com/rest
  developer.target-name.com/rest
  test.target-name.com/api
  ```
- Find more endpoints - Passive
  ```
  # Google Dorking
  intitle:"api" site:"site.com"
  inurl:"/api/v1" site:"site.com"
  intitle:"json" site:"site.com"

  # Git Dorking
  Search in github for exposed keys, documentation or any other issues 
  filename:swagger.json

  # Shodan Dorking
  site port:443
  site port:80
  “content-type: application/json” site
  “wp-json” site

  # Wayback machine
  Find old endpoints that could still be used but are hidden - improper asset management
  ```
- Find more endpoints - Active
  ```
  # Amass
  amass enum -active -d site.com | grep api
  
  # Kiterunner
  Find api endpoints - like gobuster but better, especially for API
  
  # Browser developer tools
  Check the network tab for any api endpoints
  ```
- Understand API versioning - Check the documentation - sometimes bugs are fixed in newer API and the old vulnerable APIs are still exposed.
- OWASP ZAP - Use the `Manual Explore` feature. It opens a browser and you can use the site normally with the active scan running in the background. This is better than the Automated Explore feature because it will perform authenticated scans unlike the automated scans.

## Making a Postman collection for testing
- **Method 1:** Use the 'Capture Requests' feature of Postman.
- **Method 2:** Capture requests using mitmweb and create a swagger file using mitmproxy2swagger
  ```
  # MITM web interface is available on port 8081
  # In the mitm web interface perform → File > Save > flows
  
  mitmproxy2swagger -i flows -o spec.yml -p site.com -f flow --examples
  
  # Now edit the spec.yml → remove "ignore:" command from API related endpoints
  # Now view spec.yml using a swagger editor such as editor.swagger.io  to get a better view of what the APIs do
  # Now import spec.yml to Postman as a collection
  ```

### Testing for BOLA and BFLA
- BOLA - Accessing resources that do not belong to you - Look for GET requests that use soem type of ID
- BFLA - Performing unauthorized actions - Look for PUT/POST/DELETE requests that use some type of ID
  
> In the scoping call request for 2 accounts of the same privilege, for all available privileges.

- Find endpoints with IDs in the request Eg: getprofile/1 , getprofile/2.
- With ID=1, try accessing the URI with ID=2.
- Find endpoints that require admin privileges, and access them with guest/lower privileges (by replacing cookies or tokens).

### Testing for Excessive Data Exposure
- Call the API
- Look at the response
- Check if it is disclosing more info than needed
- Extra mile - Try to find other endpoints from the API
- Extra mile - Look for hidden parameters

### Testing for Improper Assets Management
Find Old / Non-production API
- Is the old version responding with a different response code?
- Is the old version rate limited? If otp is 4 digits only and the reset password API is not rate limited we can bruteforce the otp.
- Test the old versions as authorized and unauthorized user
- Use Postman's "Find and Replace" function to replace all v2 with v1. Alternatively you can set a {{ver}} environment variable.

### Testing for Business Logic Errors
- If you see a number, make it really large
- If you see a number, make it negative
- Try to skip some steps (some endpoints being called) in a process flow. Eg: cart -> pay -> complete_orders can be cart -> complete_orders

### Testing for SSRF
Burp Collaborater alternatives for OOB testing
- https[://]ifconfig[.]pro
- https[://]webhook[.]site

### Testing for Injection Attacks
**SQLi**
```
'
''
;%00
--
-- -
""
;
' OR '1
' OR 1 -- -
" OR "" = "
" OR 1 = 1 -- -
' OR '' = '
OR 1=1
```
**NoSQLi**
```
$gt 
{"$gt":""}
{"$gt":-1}
$ne
{"$ne":""}
{"$ne":-1}
 $nin
{"$nin":1}
{"$nin":[1]}
{"$where":  "sleep(1000)"}
```
**OS**
```
|whoami
||whoami
& whoami
&& whoami
'whoami
"whoami
;whoami
'"whoami
```

### Testing for XSS 
- Find an endpoint which takes an input and also reflects it in the response.
- If response is JSON, see how the frontend uses it - could reflect within the client.

### Testing for CSRF
- Find an endpoint that performs an impactful action without an auth-token. This is essentially CSRF.

**Resources**
1. [APISec University](https://www.apisecuniversity.com/#courses)
2. [Katie Paxton-Fear API hacking](https://www.youtube.com/watch?v=qqmyAxfGV9c&list=PLbyncTkpno5HqX1h2MnV6Qt4wvTb8Mpol&index=4)
3. [Katie Paxton-Fear more API hacking](https://www.youtube.com/watch?v=qC8NQFwVOR0)
