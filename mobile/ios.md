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
- **Special Purpose Directories**
  - User generated contents - `Documents/`
  - Non-user generated contents - `Library/`
    - `Library/Application Support/` - Hidden support files
    - `Library/Caches/` - Cache data
    - `Library/Preferences/` - Preference files
  - Temporary storage - `tmp/`
- **Data Protection Classes** - Defines when keys to decrypt application files are available. 
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

# Jail Breaking
The process of removing software restrictions imposed by the manufacturer through kernel patching.
> Use with caution. Only use on test devices without any personal, sensitive or important data.

Types:
1. Untethered - Once jailbroken, the kernel gets patched everytime the device booted without the need for an external computer.
2. Tethered - Once jailbroken, an external computer is required to boot the device. It will not boot without the computer.
3. Semi-tethered - It will boot as a normal (non-jailbroken) device without a computer. But to access the jailbroken features, it should be booted with a computer.
4. Semi-untethered - It will boot as a normal (non-jailbroken) device. An app is used to access the jailbroken features.


# Test environment
- Jailbroken device (use with caution)
  - [Checkra1n exploit](https://checkra.in/) - Cydia
  - iOS 15.x-16.x [Palera1n exploit](https://github.com/palera1n) - Sileo Nightly
- Emulators
  - [Corellium](https://www.corellium.com/) (paid, no trial)
  - [Appetize.io](https://appetize.io/) (paid, has trial)
  - Xcode built in emulator

# Static Analysis

## Mach-O Disassembly
- iOS binaries (Mach-O) cannot be decompiled - they can only be disassembled
  - Use `Otool` which comes inbuilt with Xcode to disassemble the binary
  - If the source is written in Objective-C we can see the classes and methods being used
  - Look for ASLR: `otool -hv <appname>`
  - Look for stack smashing protection: `otool -I -v <appname> | grep stack`
    
## iPA Analysis
- Application files are in a `.ipa` format (To remember: **i**OS **pa**ckage). It is a zipped file.
- Pulling IPA from App Store (always use a burner AppleID)
  - [AnyTrans by imobie](https://www.imobie.com/anytrans/)
  - [IPATool](https://github.com/majd/ipatool)
- ⭐ **Look for .plist and .json files** - Rename .ipa to .zip and extract it  
  ```
  [Application].zip
  |_ iTunesMetadata.plist        <----- Information about the developer
  |_ META-INF/
  |_ Payload/                    <----- Look for .plist files with strings of interest
    |_ [Application_Name].app    <----- The application. (Right Click > Show Package Contents)
       |_ Info.plist             <----- Information about the application (like in AndroidManifest.xml)
       |_ Frameworks/
       |_ assets/                <----- Look for .json files with strings of interest
  ```
- ⭐ Look for **supported versions**
  - In `Info.plist` look for `MinimumOSVersion`
- ⭐ Look for **NSUserDefaults**
  - Persistent state of user preferences and application properties - may contain sensitive information
- ⭐ **Entitlements** are permissions of an app
  - `<key>get-task-allow</key>`: This entitlemnt allows other apps to attach to it (like a debugger). This is seen in dev environments and should be disabled in production. 
  ```
  # To enumerate entitlements
  rabin2 -T Payload/[Application_Name].app/[Application_Name]
  ```
- ⭐ **Database files** 
  - Look for `.db , .sqlite, .sqlite3` files
  - Look for sensitive data. Analyze their data protection class.
  
# Dynamic Analysis

## Bypassing SSL Pinning

### 1. Objection/Frida
Install Frida and then install Objection (order is important)
```
pip3 install frida-tools
pip3 install objection
```

Step 1- Patch the iPA that was pulled (downloaded locally)
```
objection patchipa --source <pulled_ipa> -c <developer_profile_code_sign>
```

OR

Step 1 - For app running in an iDevice connected via USB
```
# Find the application's screen name
frida-ps -Ua

# Use objection 
objection -g <screen_name> explore

# Bypass SSL pinning
ios sslpinning disable
```

### 2. Burp Mobile Assistant 
> NOTE: Requires BurpSuite Professional and a jailbroken device

- Follow instructions at [Installing Burp Suite Mobile Assistant]( https://portswigger.net/burp/documentation/desktop/tools/mobile-assistant/installing)

### 3. SSL Killswitch
- [SSL Kill Switch 2](https://github.com/nabla-c0d3/ssl-kill-switch2)
- [SSL Kill Switch for higher versions of iOS](https://github.com/nabla-c0d3/ssl-kill-switch2/issues/98)

## Common Attacks

⭐⭐ Read these case studies by Cobalt
- [Part 1](https://www.cobalt.io/blog/learning-ios-app-pentesting-and-security-part-1)
  - Improper URL scheme validation
  - Implementation of UIWebView (insecure component)
- [Part 2](https://www.cobalt.io/blog/ios-app-pentesting-and-security-with-real-world-case-studies-part-2)
  - Hardcoded API keys
  - Hardcoded SSH private key
- [Part 3](https://www.cobalt.io/blog/part-3-learning-ios-app-pentesting-and-application-security-with-real-world-case-studies)
  - Vulnerable third party library
  - Improper implementation of SSL pinning

### 1. Bypass Jailbreak detection
- Using Objection
  - `ios jailbreak disable`
- If tools don't work, you will need to patch (reverse engineer) the source and remove the detection logic - Ghidra can be used.

### 2. Fingerprint bypass
- Using Objection
  - `ios UI biometrics_bypass`

### 3. Keychain dumping
- Using Objection
  - `ios keychain dump --json keychains.json`

### 4. Hacking WebViews
Types of WebViews
- ⛔ UIWebView (insecure)
  - JavaScript can't be disabled - good for XSS
  - Can't restrict access to files
  - No SOP for "file://" scheme - `loadHTMLString:baseURL` has NULL origin
  - Runs in-process
  - Authentication: Can see all traffic of the app (even 3rd party authentication)
- ✔️ WKWebView (more secure)
  - JavaScript can be disabled
  - Can restrict access to files
  - The `loadHTMLString:baseURL` NULL origin vulnerability is fixed
  - Runs out-of-process
  - Authentication: Can see all traffic of the app (even 3rd party authentication)
- ✔️ SFSafariViewController
  - Authentication: Does not see any data/traffic of the app. This does not provide not a good user experience (user has to login everytime)
  - SFAuthenticationSession - Cookies and website data is shared (deprecated)
  - ASWEbAuthenticationSession - Same as above, but does not save the session cookies
  
**XSS in WebViews**
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
1. [iOS Pentesting 101 by Cobalt](https://www.cobalt.io/blog/ios-pentesting-101)
2. [iOS Pentesting by Mantis](https://www.youtube.com/playlist?list=PL5Fxd3nu70eyqiqrVlD9QMoaOARr082TA)
3. [iOS Pentesting by Hacker101](https://www.hacker101.com/playlists/mobile_hacking.html)
4. [iOS Pentesting by Hacktricks](https://book.hacktricks.xyz/mobile-pentesting/ios-pentesting)
