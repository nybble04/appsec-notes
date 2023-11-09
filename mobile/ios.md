> NOTE: Read Android notes before reading iOS notes.

# iOS Security Architecture
- Hardware Layer
  - Hardware components are signed into the device - replacement with non-signed component will make the phone unusable
  - Each device has a set of two keys: Device Key and Group Key which are signed by the Apple Root Certificate
- Software Layer
  - iOS is Unix based
  - File system has two partitions
    - User partition
    - OS partition
  - All applications are signed by Apple
  - Apple developer profiles
    - Free: For testing purposes. We can sideload the app into the test device.
    - Paid: To publish an app in the App Store.

# iOS File System
- Bundle directory - `/var/containers/Bundle/Application/[RANDOM_UUID]`
- Runtime data storage - `/var/mobile/Containers/Data/[RANDOM_UUID]`
- Special Purpose Directories
  - `Documents/` - User generated contents
  - `Library/` - Non-user generated contents
    - `Library/Application Support/` - Hidden support files
    - `Library/Caches/` - Cache data
    - `Library/Preferences/` - Preference files
  - `tmp/` - Temporary storage
- Data Protection Classes - Defines when keys to decrypt application files are available. 
  - `NSFileProtectionComplete` - Decryption keys are available only when unlock, without any exceptions
  - `NSFileProtectionCompleteUnlessOpen` - Decryption keys are available only when unlocked, with the exception of the files that remain open when locked
  - `NSFileProtectionCompleteUntilFirstUserAuthentication` - Decryption keys are available once the device is booted and unlocked, untill the next reboot and unlock
  - `NSFileProtectionNone` - Decryption keys available always
    
  To see the data protection settings:
  ```
  frida -U --codeshare ay-kay/iosdataprotection <app_name>
  getDataProtectionKeyForPath()
  getDataProtectionKeyForAllPaths()
  ```

# Test environment
- Jailbroken device (use with caution)
  - [Checkra1n exploit](https://checkra.in/) - Cydia
  - iOS 15.x-16.x [Palera1n exploit](https://github.com/palera1n) - Sileo Nightly
- Emulators
  - [Corellium](https://www.corellium.com/) (paid, no trial)
  - [Appetize.io](https://appetize.io/) (paid, has trial)
  - Xcode built in emulator

# Static Analysis
- Application files are in a `.ipa` format (To remember: **i**OS **pa**ckage). It is a zipped file.
- Pulling IPA from App Store (always use a burner AppleID)
  - [AnyTrans by imobie](https://www.imobie.com/anytrans/)
  - [IPATool](https://github.com/majd/ipatool)
- Directory Structure of an unzipped `ipa` file
  **Look for .plist and .json files**
  ```
  [Application].ipa.zip
  |_ iTunesMetadata.plist        <----- Information about the developer
  |_ META-INF/
  |_ Payload/                    <----- Look for .plist files with strings of interest
    |_ [Application_Name].app    <----- The application. (Right Click > Show Package Contents)
       |_ Info.plist             <----- Information about the application (like in AndroidManifest.xml)
       |_ Frameworks/
       |_ assets/                <----- Look for .json files with strings of interest
  ```
- **Entitlements** are permissions of an app
  - `<key>get-task-allow</key>`: This entitlemnt allows other apps to attach to it (like a debugger). This is seen in dev environments and should be disabled in production. 
  ```
  # To enumerate entitlements
  rabin2 -T Payload/[Application_Name].app/[Application_Name]
  ```
- Look for **supported versions**
  - In Info.plist, look for `MinimumOSVersion`
  
# Dynamic Analysis

## Bypassing SSL Pinning

### 1. Objection/Frida
Install Frida and then install Objection (order is important)
```
pip3 install frida-tools
pip3 install objection
```
Patch the iPA that was pulled (downloaded locally)
```
objection patchipa --source <pulled_ipa> -c <developer_profile_code_sign>
```

### 2. Burp Mobile Assistant 
> NOTE: Requires BurpSuite Professional and a jailbroken device

- Follow instructions at [Installing Burp Suite Mobile Assistant]( https://portswigger.net/burp/documentation/desktop/tools/mobile-assistant/installing)

### 3. SSL Killswitch
- [SSL Kill Switch 2](https://github.com/nabla-c0d3/ssl-kill-switch2)

- [SSL Kill Switch for higher versions of iOS](https://github.com/nabla-c0d3/ssl-kill-switch2/issues/98)

## Keychain dumping
Using Objection
- Run the command
  `ios keychain dump --json keychains.json`

## Bypass Jailbreak detection
- Using Objection
  `ios jailbreak disable`

## Hacking WebViews
Types of WebViews
- UIWebView (insecure)
  - JavaScript can't be disabled - good for XSS
  - Can't restrict access to files
  - No SOP for "file://" scheme - `loadHTMLString:baseURL` has NULL origin
  - Runs in-process
  - Authentication: Can see all traffic of the app (even 3rd party authentication)
- WKWebView (more secure)
  - JavaScript can be disabled
  - Can restrict access to files
  - The `loadHTMLString:baseURL` NULL origin vulnerability is fixed
  - Runs out-of-process
  - Authentication: Can see all traffic of the app (even 3rd party authentication)
- SFSafariViewController
  - Authentication: Does not see any data/traffic of the app. This does not provide not a good user experience (user has to login everytime)
  - SFAuthenticationSession - Cookies and website data is shared (deprecated)
  - ASWEbAuthenticationSession - Same as above, but does not save the session cookies
-** XSS in WebViews**: Target the following
    - URL scheme that takes an input to load a webview with JS enabled
    - Document sharing
    - User clicked links
    - User-generated content
 ```
 # To find vulnerable webviews with user interaction
 frida-trace -U -m "*[UIWebView load*]" <app_name>
 ```

# Automated Analysis
- Use [MobSF](https://github.com/MobSF/Mobile-Security-Framework-MobSF)

**References:**
1. [iOS Pentesting by Mantis](https://www.youtube.com/playlist?list=PL5Fxd3nu70eyqiqrVlD9QMoaOARr082TA)
2. [iOS Pentesting by Hacker101](https://www.hacker101.com/playlists/mobile_hacking.html)
3. [iOS Pentesting by Hacktricks](https://book.hacktricks.xyz/mobile-pentesting/ios-pentesting)
