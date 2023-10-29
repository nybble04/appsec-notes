# Application Programming Interface (API) Security

## OWASP Top 10 - 2019

1. Broken Object Level Authorization (BOLA) --> Like IDOR (resource level)
2. Broken User Authentication --> Tokens
3. Excessive Data Exposure --> Information Disclosure
4. Lack of Resources & Rate Limiting
5. Broken Function Level Authorization (BFLA) --> Like IDOR (functionality level)
6. Mass Assignment --> Being able to change more than what is allowed or being able to add an extra parameter in the request which will get processed.
7. Security Misconfiguration --> XSS from CORS
8. Injection --> Try NoSQLi
9. Improper Assets Management --> Like keeping v1 around when v6 is available
10. Insufficient Logging & Monitoring
