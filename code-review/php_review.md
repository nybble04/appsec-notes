# Secure Coding in PHP

## 1. Properly name files that you want to include
- Do not name included files (like database connection files ) with the `.inc` extension.
- Instead, name it with a `.php` extension.
- If it is named with `.inc` then its contents can be viewed from the browser if directory traversal is allowed.
- This can reveal sensitive information like source code or credentials.

## 2. XSS Protection
- Escape (neutralize) the output using the `htmlspecialchars` [function](https://www.php.net/manual/en/function.htmlspecialchars.php)
```
 htmlspecialchars(
    string $string,
    int $flags = ENT_QUOTES | ENT_SUBSTITUTE | ENT_HTML401,
    ?string $encoding = null,
    bool $double_encode = true
): string
```
- Characters like `<` will be replaced with `&lt` and so on.

## 3. Password Hashing
- MD5 is weak.
- To hash: use ` password_hash(string $password, string|int|null $algo, array $options = []): string` 
[function](https://www.php.net/manual/en/function.password-hash.php).
- To verify: use ` password_verify(string $password, string $hash): bool` 
[function](https://www.php.net/manual/en/function.password-verify.php).

## 4. Directory Listing
For apache
- Create a `.htaccess` files, and add the directive `Options -Indexes` in it.
- We get a Forbidden 403 error.

For nginx
- Disabled by default in `nginx.conf`.
- To explicitly enable for a particular path: Within the `server{}` block, under the appropriate `location` directive add `autoindex on`.

For Tomcat (not PHP, but still worth knowing)
- Disabled by default in `<CATALINA_HOME>\conf\web.xml`.
- Check the default servlet's `listings` parameter.
```
<!-- The default servlet for all web applications, that serves static     -->
<!-- resources.  It processes all requests that are not mapped to other   -->
<!-- servlets with servlet mappings.                                      -->
<servlet>
  <servlet-name>default</servlet-name>
  <servlet-class>org.apache.catalina.servlets.DefaultServlet</servlet-class>
  <init-param>
    <param-name>debug</param-name>
    <param-value>0</param-value>
  </init-param>
  <init-param>
    <param-name>listings</param-name>
    <param-value>false</param-value>
  </init-param>
  <load-on-startup>1</load-on-startup>
</servlet>
     
<!-- The mapping for the default servlet -->
<servlet-mapping>
  <servlet-name>default</servlet-name>
  <url-pattern>/</url-pattern>
</servlet-mapping>
```
## 5. HttpOnly and Secure Cookies
- These cookies cannot be access by the client using JS.
- In the [function](https://www.php.net/manual/en/function.setcookie.php) 
```
 setcookie(
    string $name,
    string $value = "",
    int $expires_or_options = 0,
    string $path = "",
    string $domain = "",
    bool $secure = false,
    bool $httponly = false
): bool
```
set the `$httponly` and `$secure` parameters to `true`.

## 6. What not to store in cookies
- A guessable and modifieable identifier like username, or access control information like isadmin=true.

## 7. CSRF Protection
- Use a framework or a library that has CSRF protection (CSRF tokens).
- Do NOT implement your own mechanism.

## 8. SQL Injection
- What is the problem? An SQL query has two parts - the query language statements and the data. When the s**upplied data is being considered as query language**, then SQL injection occurs. 
- Use prepared statetements
```
$query = $db->prepare("SELECT * FROM users WHERE email = :email");          #Can use a placeholder ':email' or a '?'
$query->execute(['email' => $email]);
```

## 9. Error Reporting
- Do not display detailed errors on the webpage.
- Use logs for detailed errors.
- For webpage, use a generic error page that is usable for the average user.
- Disable globally in `php.ini`. Set the directive - `display_error = Off`
- If this can't be configured globally in `php.ini`, then add `ini_set('display_errors', 'Off')` OR `error_reporting(0)` in the .php files with sensitive info.

**Resources**
1. [PHP Security Playlist](https://www.youtube.com/playlist?list=PLfdtiltiRHWFsPxAGO-SVPGhCbCwKWF_N)
