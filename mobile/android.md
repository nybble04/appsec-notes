# Android Security Model
- Linux Kernel: If you get a shell on the phone you can use linux commands. SELinux became enabled by default since API level 18.
- Every application runs on virtual machine known as the **Android Runtime (ART)**
- Like _Java_ bytecode is processed by _JVM_, the _Android (Dalvik)_ bytecode is processed by the _Android Runtime_.
- NOTE: Before ART, Dalvik was the runtime VM used. It processed the Dalvik bytecode.

**Compilation Process**

Java src ---(Java compiler)--> Java Bytecode --(Dex compiler)--> Dalvik bytecode --> Android Runtime

**Android Runtime (ART)**
- Each app has its own sandbox environment.
- Each app has its own user in the filesystem. This user is the owner of the application directory.
- The app users have UID between 10000 and 999999.
- There is a root user. This user can access all the files in the device. Rooting a device means running as root user.

**Important file paths for an app**
1. `/data/app/com.example.app` - Generic app data.
2. `/data/data/com.example.app` - Runtime storage of data.
3. `/mnt/sdcard/Android/data/com.example.app` - External storage for runtime.

App-1 and App-2 cannot interact with each other unless explicitly granted by a **Content Provider/Broadcast Receiver**. Example of a Content Provider/Broadcast receiver is the "Open with Chrome/Firefox" option that comes. In this case, the app will interact with another app.

**Android Profiles**

Profiles allow to separate app data for various uses like work or personal or kids.
- Primary User - The user created the first time the phone is started. It is always running. It can only be removed by a factory reset.
- Secondary User - Additional users created/deleted by the Primary User.
- Guest User - There can only be one Guest User at a time. Used for fast access to the phone.

**Reverse engineering android**

Development: Java/Kotlin source --> DEX Bytecode

Reverse Engineering: DEX Bytecode --> SMALi --> Decompiled Java/Kotlin source

**Application Signing**
- Using PKI - Sign using Private, Verify using Public.
- Three APK Signature schemes - v1, v2, v3.
- Google Play Sign: Additional security mechanism where Google adds its signatures to the apps.

# Static Analysis

### ADB Tool - Getting the application data

Install adb (Linux)
```
sudo apt install adb
```

Step 1: Start the emulator and download the application to the AVD

Step 2: Get a shell on the android emulator
```
adb shell
```

Step 3: Search for the application's **package_name**
```
pm list packages | grep <app_name>
```

Step 4: Using the package name, find the **path_name** to the package
```
pm path <package_name>
```

Step 5: Exit the android shell
```
exit
```

Step 6: Pull (download) the apk to the local system for analysis
```
adb pull <path_name> <saveas_name.apk>
```

### Decompile and view files
Using apktool (Linux)
```
sudo apt install apktook
```
Step 7 (alt): Decompile `d` the apk. If the application is too large then do not decompile the resources `-r`.
```
apktool d -r <pulled_apk>
```

OR

Using JADX-GUI (Linux)
```
sudo apt install default-jdk
sudo apt install jadx
```
Step 7: Open JADX-GUI and import the pulled apk. 

:star: **Step 8:** Check `AndroidManifest.xml` in `Resources/res/`. Look for the following

Refer: [Lucideus - Security Review of Android Manifest File Part I](https://medium.com/@lucideus/security-review-of-android-manifest-file-part-i-ecb5ca51eb6a)

- `minSdkVersion`
- `uses-permission` such as `WRITE_EXTERNAL_STORAGE`. 
    - See H1 report [Possible to steal any protected files on Android](https://hackerone.com/reports/161710)
- `export="true"` : These activities can be directly accessed. If an activity with sensitive information or functionality can be exported (Eg: view admin profile), then we can access it directly without having to login. 
    - See H1 report [Vulnerability in exported activity WebView](https://hackerone.com/reports/537670)
- `debug="true"` : This allows us to inject our own code and execute it in the context of application. 
    - See [Exploiting debuggable android applications](https://resources.infosecinstitute.com/topics/application-security/android-hacking-security-part-6-exploiting-debuggable-android-applications/)
- If JS libraries are used, XSS could be possible. 
    - See H1 report [XSS via start ContentActivity](https://hackerone.com/reports/189793)
- Look for any keys that could be sensitive or abused. 
    - See H1 report [Insecure Storage and Overly Permissive API Keys in Android App](https://hackerone.com/reports/753868)
- Look for Firebase URLs. Then test firebase using [this guide](https://cloud.hacktricks.xyz/pentesting-cloud/gcp-pentesting/gcp-services/gcp-databases-enum/gcp-firebase-enum). 
- Look for any AWS storage bucket names. Proceed with tools like [cloud_enum](https://github.com/initstring/cloud_enum)

:star: **Step 9:** Check `Strings.xml` in `Resources/res/resources.arsc/res/values/`. Look for the following
- Hardcoding sensitive strings. AWS ID, storage bucket names.
- Interesting strings (mangled, encoded) that are used in the code. Analyze the code where this is being used. Check if it can be bypassed.

:star: **Step 10:** Other files for static analysis
- Flutter applications - `libapp.so`
- React native applications - `index.android.bundle`

# Dynamic Analysis

## Setting up Burp with Android Studio's AVD Emulator to intecept traffic (will be HTTPS)

In BurpSuite
- Create a new proxy configuration
- Click "Import / export CA certificate"
- Export the certificate in DER format
- Save it with the `.CER` extension
- Drag and drop this certificate to the emulator

In the Android AVD device (inside the phone), import the Burp Certificate
- Settings > Security > Under Credential storage select Install from SD card > select the Burp certiciate added

In Android Studio's AVD settings
- Go to Settings > Proxy
- Uncheck "Use Android Studio HTTP proxy settings" 
- Select "Manual proxy configuration 
- Enter Burp Proxy's configuration into this

## ⚠️ SSL Pinning
- It is a security mechanism
- It prevents man-in-the-middle attack by importing a malicious certificate.
- The app is made to trust only a particular certificate. It won't trust the malicious certificate.

## SSL Pinning Bypass

### 1. Objection/Frida

**Case 1 - Automatically Patching (injecting Frida) Applications using Objection**

Step 1: Install Frida and then install Objection (order is important)
```
pip3 install frida-tools
pip3 install objection
```

Step 2: Patch the APK that was pulled (downloaded locally) using ADB
```
objection patchapk --source <pulled_apk>
```
Patched APK will be saved as `<pulled_apk>.objection.apk`. Sometimes this app will not run. If you get an error `Error: INSTALL_PARSE_FAILED_NO_CERTIFICATES`, you have to patch it manually.

> Note: If you get a "Invalid resource directory name: ...." error. The app might be using Kotlin, and there might be a parsing error. In this case, use `objection patchapk --source <pulled_apk> --useaapt2`

**Case 2 - Manually Patching (injecting Frida) Applications using APKTool, Keytool, jarsigner and zipalign**

**Refer:** [Guide](https://koz.io/using-frida-on-android-without-root/)

Step 1: Decompile using APKTOOL.
```
apktool d -r <pulled_apk>
```

Step 2: Go to the created directory where the APK has been decompiled. In `lib/` go to the folder that has the CPU architecture of the AVD (emulator) that was being used Eg: `x86_64/`.

Step 3: Download the frida gadget  release for the above architecture of Android from [Frida Github](https://github.com/frida/frida/releases). Extract the zip file.

Step 4: Cut-Paste the downloaded frida gadget into the folder mentioned in Step 2.

Step 5: Rename it to `libfrida-gadget.so`

Step 6: Choose an activity that you know is loaded first when the application launches Eg: MainActivity. Go to its SMALi code (look for a `smali` folder). In that go to the `.smali` code of the chosen activity Eg: MainActivity.smali. 

In the smali code, under `.method public constructor <innit>()V` paste the following before `return-void`:
```
const-string v0, "frida-gadget"
invoke-static {v0}, Ljava/lang/System;->loadLibrary(Ljava/lang/String;)V
```

Step 7: Rebuild the application
```
apktool b <pulled_apk_modified> -o <pulled_apk>_patched.apk 
```

Step 8: Sign the patched APK.
```
# Create a keystore
keytool -genkey -v -keystore my.keystore -alias mykeys -keyalg RSA -keysize 2048 -validity 10000

# Sign the APK
jarsigner -sigalg SHA1withRSA -digestalg SHA1 -keystore my.keystore -storepass Pass123 repackaged.apk mykeys

# Verify the signature
jarsigner -verify repackaged.apk

# Zipalign the apk
zipalign 4 repackaged.apk repackaged-final.apk
```

**Case 3 - Patching Split APKs using Objection**

**Split APK:** If the APK is split into multiple smaller APKs.

Step 1: Inject base.apk with objection
```
objection patchapk -s <base.apk> --use-aapt2
```

Step 2: Sign each of the the split_config.apk one by one
```
objection signapk split_config.1.apk
objection signapk split_config.2.apk
```

Step 3: Install the patched APK to the emulator using ` adb install-multiple`
```
adb install-multiple base.objection.apk split_config.1.objection.apk split_config.2.objection.apk
```

### 2. Using iptables
**Refer**: [Guide](https://infosecwriteups.com/bypass-ssl-pinning-with-ip-forwarding-iptables-568171b52b62)

`vboxnet1` - Interface of the android emulator

`wlan0` - Interface of the host machine running Burp Proxy

**In Android device**

Step 1: Configure static IP address to one that is in the vboxnet subnet

Step 2: Set gateway to the vboxnet IP address

Step 3: Set DNS to 8.8.8.8

Step 4: Save and reboot

**In the host machine (Linux)**

Step 1: Start Burp proxy on port 8081

Step 2: Enable IP forwarding
```
echo 1 > /proc/sys/net/ipv4/ip_forward
```

Step 3: Setup port forwarding using iptables
```
sudo iptables -t nat -A POSTROUTING -o wlan0 -j MASQUERADE
sudo iptables -t nat -A PREROUTING -p tcp -i vboxnet1 --dport 80 -j REDIRECT --to-port 8081
sudo iptables -t nat -A PREROUTING -p tcp -i vboxnet1 --dport 443 -j REDIRECT --to-port 8081
```

## Automated Static and Dynamic Analysis
Use [MobSF](https://github.com/MobSF/Mobile-Security-Framework-MobSF)

Points to remember:
- When using MobSF for dynamic analysis, you should not start the AVD from Android Studio.
- You should start it using the `emulator` command.
- To do this, add the Android SDK emulator directory `/home/<user>/Android/Sdk/emulator` to the `PATH`.
- AVD's with playstore enabled will NOT work.

List available AVDs
```
emulator -list-avds
```
Choose the AVD name from the list and run it using the emulator command
```
emulator -avd <name> -writeable-system -no-snapshot
```

# Common Attacks

## 1. Exported Activities
- Exported activities can be accessed directly
  - Authentication/authorization bypass
  - Return sensitive information/results
  - Tapjacking - make user perform unexpected actions
  ```
  adb shell am start -n <package_name>/<activity_name>
  Eg: adb shell am start -n com.example.package.com/com.example.test.ExportedActivity
  ```

## 2. Data Leak through Content Provider 
- Content Providers provide a way for data to be shared between processes
- After decompiling using JADX-GUI or apktool, search for `content://` to see the content providers
- Invoke a content provider URL that seems interesting **as a non-root user** - If data is visible then it is a vulnerability
```
# Using ADB
adb shell content query --url content://com.example.package.CoolApp/secret

# Using Drozer
run app.provider.query content://com.example.package.CoolApp/secret --vertical
```
Check the following content provider attack vectors at [hacktricks](https://book.hacktricks.xyz/mobile-pentesting/android-app-pentesting/drozer-tutorial/exploiting-content-providers#exploiting-content-providers) using Drozer
- File read
- Path Traversal
- SQL Injection

## 3. Insecure Broadcast Receiver
- A broadcast receiver listens for specific Android-broadcast or custom-broadcast messages.
- These broadcast messages are sent out when a specific event occurs. Broadcast receivers are part of a publish-subscribe communication model.
- To test for vulnerabilities
  - Identify broadcast receivers using drozer
    ```
    run app.broadcast.info -a <app_package>
    ```
  - Check the code, specifically the `onCreate` function of the Activity and `onReceive` function of the Broadcast Receiver
  - See how the message/even affects the functionality of the broadcast receiver. Then see if it can be modified to perform an attack.
    - Eg: If the event takes a user supplied URL and opens it in a WebView - we can trigger this broadcast message ourselves and make it render a malicious URL. [Watch video](https://www.youtube.com/watch?v=_Bp6QNDND3s)

### Resources
1. [Android Pentesting by Byte Theories](https://www.youtube.com/playlist?list=PL1f72Oxv5SylOECx9M34pLZlNa7YkJJ14)
2. [Android Pentesting by Hacker101](https://www.hacker101.com/playlists/mobile_hacking.html)
3. [Android Pentesting by HackTricks](https://book.hacktricks.xyz/mobile-pentesting/android-app-pentesting)
4. [Android Pentesting by BitsPlease](https://www.youtube.com/playlist?list=PLgnrksnL_Rn09gGTTLgi-FL7HxPOoDk3R)
5. [Mobile App Pentesting by HackingSimplified](https://www.youtube.com/playlist?list=PLGJe0xGh7cH2lszCZ7qwsqouEK23XCMGp)
