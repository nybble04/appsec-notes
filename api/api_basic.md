# Application Program Interface (API) Testing

## Enumerating APIs
The first step for API testing
- See how endpoints are constructed
- Find more endpoints
- Guess the resource names - we can use a wordlist
- Understand API versioning - sometime bugs are fixed in newer API and the old vulnerable APIs are available

## OWASP API Security Top 10

[2019 API Top 10](https://owasp.org/www-project-api-security/)

1. Broken Object Level Authorization --> Like IDOR
2. Broken User Authentication --> Tokens
3. Excessive Data Exposure --> Information Disclosure
4. Lack of Resources & Rate Limiting
5. Broken Function Level Authorization --> Like IDOR (user level)
6. Mass Assignment --> Being able to change more than what is allowed
7. Security Misconfiguration --> XSS from CORS
8. Injection --> Rarely seen: SQLi
9. Improper Assets Management --> Like keeping v1 around when v6 is available
10. Insufficient Logging & Monitoring

### Excessive Data Exposure
- Call the API
- Look at the response
- Check if it is disclosing more info than needed
- Extra mile - Try to find other endpoints from the API
- Extra mile - Look for hidden parameters

#### IDOR
> In the scoping call request for 2 accounts of the same privilege, for all available privileges.

- Find endpoints with IDs in the request Eg: getprofile/1 , getprofile/2.
- With ID=1, try accessing the URI with ID=2.
- Find endpoints that require admin privileges, and access them with guest/lower privileges (by replacing cookies or tokens).  

### Business Logic Errors
- If you see a number, make it really large
- If you see a number, make it negative
- Try to skip some steps (some endpoints being called) in a process flow. Eg: cart -> pay -> complete_orders can be cart -> complete_orders

### XSS 
- Find an endpoint which takes an input and also reflects it in the response.
- If response is JSON, see how the frontend uses it - could reflect within the client.

### CSRF
- Watch this - [Chrome Updates and CSRF Dies?](https://www.youtube.com/watch?v=DfSZ3VHljPc)
- Find an endpoint that performs an impactful action without a token. This is essentially CSRF.

### Race Condition / No Rate Limiting
- Very common, sometimes out of scope of bug bounty

**Resources**
1. [Katie Paxton-Fear API hacking](https://www.youtube.com/watch?v=qqmyAxfGV9c&list=PLbyncTkpno5HqX1h2MnV6Qt4wvTb8Mpol&index=4)
2. [Katie Paxton-Fear more API hacking](https://www.youtube.com/watch?v=qC8NQFwVOR0)
