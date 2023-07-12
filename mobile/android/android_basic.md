# Android Security Model
- Linux Kernel: If you get a shell on the phone you can use linux commands. SELinux became enabled by default since API level 18.
- Every application runs on virtual machine known as the **Android Runtime (ART)**
- Like _Java_ bytecode is processed by _JVM_, the _Android (Dalvik)_ bytecode is processed by the _Android Runtime_.
- NOTE: Before ART, Dalvik was the runtime VM used. It processed the Dalvik bytecode.

Compilation Process: Java src ---(Java compiler)--> Java Bytecode --(Dex compiler)--> Dalvik bytecode --> Android Runtime

## Android Runtime (ART)
- Each app has its own sandbox environment.
- Each app has its own user in the filesystem. This user is the owner of the application directory.
- The app users have UID between 10000 and 999999.
- There is a root user. This user can access all the files in the device. Rooting a device means running as root user.

Important file paths for an app:
1. `/data/app/com.example.app` - Generic app data.
2. `/data/data/com.example.app` - Runtime storage of data.
3. `/mnt/sdcard/Android/data/com.example.app` - External storage for runtime.

App-1 and App-2 cannot interact with each other unless explicitly granted by a **Content Provider/Broadcast Receiver**. Example of a Content Provider/Broadcast receiver is the "Open with Chrome/Firefox" option that comes. In this case, the app will interact with another app.

## Android Profiles
- **Profiles:** Allow to separate app data for various uses like work or personal or kids.
- Primary User - The user created the first time the phone is started. It is always running. It can only be removed by a factory reset.
- Secondary User - Additional users created/deleted by the Primary User.
- Guest User - There can only be one Guest User at a time. Used for fast access to the phone.

## Reverse engineering android
Development: Java/Kotlin source --> DEX Bytecode
Reverse Engineering: DEX Bytecode --> SMALi --> Decompiled Java/Kotlin source

## Application Signing
- Using PKI - Sign using Private, Verify using Public.
- Three APK Signature schemes - v1, v2, v3.
- **Google Play Sign:** Additional security mechanism where Google adds its signatures to the apps.

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

### JADX-GUI - Decompile and view files

Install JADX (Linux)
```
sudo apt install default-jdk
sudo apt install jadx
```
Step 7: Open JADX and import the pulled apk. 

Step 8: Check `AndroidManifest.xml` in `Resources/res/`. Look for the following
- minSdkVersion
- uses-permissions
- export="true" : These activities can be directly accessed. If an activity with sensitive information or functionality can be exported (Eg: view admin profile), then we can access it directly without having to login.
- Look for any keys that could be sensitive or abused.
- Look for Firebase URLs. Then test firebase using [this guide](https://cloud.hacktricks.xyz/pentesting-cloud/gcp-pentesting/gcp-services/gcp-databases-enum/gcp-firebase-enum).
- Look for any AWS storage bucket names. Proceed with tools like [cloud_enum](https://github.com/initstring/cloud_enum)

Step 9: Check `Strings.xml` in `Resources/res/resources.arsc/res/values/`. Look for the following
- Hardcoding sensitive strings. AWS ID, storage bucket names.
- Interesting strings (mangled, encoded) that are used in the code. Analyze the code where this is being used. Check if it can be bypassed.

### APKTOOL - Decompile and view files (alternative for JADX)

Install apktool (Linux)
```
sudo apt install apktook
```
Step 7 (alt): Decompile `d` the apk. If the application is too large then do not decompile the resources `-r`.
```
apktool d -r <pulled_apk>
```

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

## IMPORTANT: SSL Pinning and Bypassing it using Frida
- It is a security mechanism
- It prevents man-in-the-middle attack by importing a malicious certificate.
- The app is made to trust only a particular certificate. It won't trust the malicious certificate.
- **Bypass SSL Pinning** : This is done using Objection and Frida.

### Automatically Patching (injecting Frida) Applications using Objection

Install Frida and then install Objection (order is important)
```
pip3 install frida-tools
pip3 install objection
```

Step 1: Patch the APK that was pulled (downloaded locally) using ADB
```
objection patchapk --source <pulled_apk>
```
Patched APK will be saved as `<pulled_apk>.objection.apk`. Sometimes this app will not run. If you get an error `Error: INSTALL_PARSE_FAILED_NO_CERTIFICATES`, you have to patch it manually.

> Note: If you get a "Invalid resource directory name: ...." error. The app might be using Kotlin, and there might be a parsing error. In this case, use `objection patchapk --source <pulled_apk> --useaapt2`

### Manually Patching (injecting Frida) Applications using APKTool, Keytool, jarsigner and zipalign

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

### Patching Split APKs using Objection

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