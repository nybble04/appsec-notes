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
3. Server side attacks - OWASP top 10 if three-tier with app server
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





**Resources**
1. [Thick-Client-Pentest-Checklist](https://github.com/Hari-prasaanth/Thick-Client-Pentest-Checklist)
2. [Thick Client Pentest: Modern Approaches and Techniques - Viraj Mota](https://www.youtube.com/watch?v=BBA72uLgcMg)
3. [Cobalt Core Academy: Thick Client Pentesting with Harsh Bothra](https://www.youtube.com/watch?v=q5PuvOlWrCQ)
