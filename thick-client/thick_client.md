# Thick Client Security

**Introduction**
- AKA - Fat clients, rich clients, thick clients
- Classification based on Tiers
  - Two tier: (Thick client) <----> (Database)
  - Three tier: (Thick client) <----> (App server) <----> (Database)
- Classify based on proxy awareness
  - Proxy aware - Can configure proxy
  - Proxy unaware (difficult to test) - Do not support proxying
- Some applications may implement SSL certificate pinning

**Methodology**
1. Enumeration - Idenitfy language, framework, architecture, intercept traffic, functionality tests
2. Client side checks - Memory analysis, binary analysis
3. Server side attacks - OWASP top 10 if three-tier with app server **(API and web test cases also apply here)**
4. Network attacks - Intercept traffic, examine vulnerabilities, external server interactions

# Windows Thick Clients

## Information Gathering
- **CFF Explorer**
  - File headers and other static attributes help us understand the underlying technology stack
- **Binscope** from microsoft
  - Compiler protection checks prevent BoF attacks
  - Command: `binscope.exe /verbose /html /logfile <results_output> <target_app_path>`
- **Signcheck** from sysinternals
  - Check if binary is signed or not
  - Command: `sigcheck.exe <target_app_path>`
- **Strings** from sysinternals
  - Check for any harcoded sensitive strings
  - Command: `strings.exe <target_app_path>`
- **Regshot**
  - To find sensitive info in registry
    - 1st shot button - before working on the app
    - 2nd shot button - after working on the app
    - Compare button - to see the difference and identify registry changes made by the application (search by application name as it will contain other app's registry entries as well)
- Verbose error messages due to improper error handling

## Reverse Engineering
- **dnSpy** for .Net applications
  - To modify a decompiled code: Right click > edit iL code > make changes > Click OK > File > Save Module > Relaunch the app
- **JDGUI** for Java applications

## Network communication
- **TCPView** from sysinternals
  - Launch the application and we can see communications from different processes running in the system
  - Filter using Search: "DVT.exe"
- **Wireshark**
  - More verbose, not as easy as TCPView to filter
- **MITM Relay + Burp Proxy**
  - To make connections go through your burp proxy: Forward traffic from (Application) -> (OG destination port) -> (mitm port in attacker system) -> (Burp proxy in attacker system)

## DLL highjacking
- DLL search order
  - Application directory
  - Current directory
  - System directory
  - 16-bit System directory
  - Windows directory
  - Directory in PATH variable
- **Process Monitor** from sysinternals
  - Result contains "NAME NOT FOUND"
  - Process Name is "DVT.exe"
  - Path ends with "dll"
  - Create a malicious dll with the name of the missing dll and place it in the Application directory. Restart the application. DLL will execute in context/with privilege of what the application is running

## Electron applications
Refer 
- [Hunting Common Misconfigurations in Electron Apps - Part 1](https://www.cobalt.io/blog/common-misconfigurations-electron-apps-part-1)
- [An Intro To Electron Application Penetration Testing](https://payatu.com/blog/an-intro-to-electron/)

**Resources**
1. [Thick-Client-Pentest-Checklist](https://github.com/Hari-prasaanth/Thick-Client-Pentest-Checklist)
2. [Thick Client Pentest: Modern Approaches and Techniques - Viraj Mota](https://www.youtube.com/watch?v=BBA72uLgcMg)
3. [Cobalt Core Academy: Thick Client Pentesting with Harsh Bothra](https://www.youtube.com/watch?v=q5PuvOlWrCQ)
4. [Hunting Common Misconfigurations in Electron Apps - Part 1](https://www.cobalt.io/blog/common-misconfigurations-electron-apps-part-1)
