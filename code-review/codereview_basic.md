# Code Review Methodology

**Where to find source code if not given (black box to white box)**
1. Look at client-side code.
2. Desktop or mobile app source code (decompile).
3. Leak code through a vulnerability: path traversal.
4. OSINT: Github, pastebin, etc.

**Source and Sink**
- **Source** - The code that allows the vulnerability to happen. Eg: `command = $_GET['c']`
- **Sink** - The place where the vulnerability takes effect. Eg: `exec(command)`
If data flows from the source to the sink without proper validation, then there is a vulnerability.

**Tips to quickly start**
1. Search for **known dangerous functions** - see if they operate on user input.
2. **Hardcoded credentials** - API keys, encryption keys, database passwords. NOTE: This is vulnerable even on the server side.
3. The use of **weak cryptography and hashing algorithms** - MD5, SHA1, DES.
4. **Outdated dependencies** - Look for dependencies, their versions and if they are associated with any CVE.
5. Look for **revealing developer comments** - Might reveal sensitive info - ip, credentials, other files that might have sensitive info.

**More comprehensive Review**
1. Focus on critical functions first (Authentication, Authorization, PII like payment or shipping, etc.
2. Follow any code that takes in use input.
3. Use SAST, SCA and secrets scanner tools. Then manually verify the results.

**Some concepts covered in Paul's presentation**
1. Input Validation - Whitelist is better than blacklist.
2. Neutralize input - use parameterized/prepared statements. important for backend, to avoid vulns like sqli.
3. Neutralize output - important for frontend, to avoid reflection. 
4. Data encryption
5. Memory safe functions - for C/C++, to avoid vulns like buffer overflow.
6. IDOR - to avoid access control issues.


**Resources**
1. [Vickie Li - OWASP Dev Slop](https://www.youtube.com/watch?v=A8CNysN-lOM)
2. [Paul Ionescu - OWASP Dev Slop](https://www.youtube.com/watch?v=rAwxFw25x3E)
3. [Practice - OWASP Secure Coding Dojo](https://github.com/OWASP/SecureCodingDojo)
