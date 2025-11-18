ADB Proxy & Certificate Manager

<p align="center">
<img src="https://www.google.com/search?q=https://img.shields.io/badge/Android-10%252B-3DDC84%3Fstyle%3Dfor-the-badge%26logo%3Dandroid%26logoColor%3Dwhite" alt="Android 10+">
<img src="https://www.google.com/search?q=https://img.shields.io/badge/Shell-Bash%2520%257C%2520Zsh%2520%257C%2520Sh-4EAA25%3Fstyle%3Dfor-the-badge%26logo%3Dgnu-bash%26logoColor%3Dwhite" alt="Shell Script">
<img src="https://www.google.com/search?q=https://img.shields.io/badge/Root-Required-E34F26%3Fstyle%3Dfor-the-badge" alt="Root Required">
</p>

<p align="center">
<strong>A robust, cross-platform shell script to automate HTTP proxy configuration and System Certificate installation on Android devices.</strong>
</p>

<p align="center">
Designed for penetration testers and developers using
<b>Burp Suite</b>, <b>HTTP Toolkit</b>, <b>MitMProxy</b>, or <b>Charles Proxy</b>.
</p>

🚀 Features

🔌 One-Click Proxy: Toggle HTTP Proxy settings on your device instantly.

🛡️ Android 14+ Support: Bypasses the new read-only APEX certificate stores using nsenter namespace injection.

💉 Live Injection: Installs System Certificates without rebooting the device.

🔧 Chrome Fix: Built-in helper to install User Certificates for browsers that distrust System stores (Certificate Pinning workarounds).

💻 Cross-Platform: Works on macOS, Linux, and Windows (WSL/Git Bash).

🧠 Universal: Auto-detects OS version and applies the correct mounting strategy.

🛠 Prerequisites

Ensure you have the following installed on your host machine:

ADB (Android Debug Bridge) - Part of Android SDK Platform-Tools.

OpenSSL - To calculate certificate hashes.

Curl - To download the certificate from your proxy tool.

Rooted Device/Emulator - su access is required for certificate injection.

Installation

macOS

brew install android-platform-tools openssl curl


Ubuntu/Debian

sudo apt install adb openssl curl


⚙️ Configuration (Extending to Other Proxies)

By default, the script is configured for Burp Suite or HTTP Toolkit running on localhost.

You can easily extend this to work with Charles Proxy, MitMProxy, or a remote server by editing the variables at the top of adb_proxy.sh:

Variable

Description

Example

PROXY_HOST

IP of your host machine relative to the emulator.

10.0.2.2 (Emulator)



192.168.1.x (Physical)

PROXY_PORT

Port your proxy tool is listening on.

8080

CERT_URL

URL to download the CA Certificate.

http://127.0.0.1:8080/cert (Burp)



http://mitm.it/cert/pem (MitMProxy)

📖 Usage

Make the script executable:

chmod +x adb_proxy.sh


Run the script:

./adb_proxy.sh


Follow the interactive menu:

Option

Action

Description

1

Start Proxy

Sets the global HTTP proxy on the device to your configured host/port.

5

Install System Cert

The Magic Button. Injects your proxy's CA into the System Trust Store so apps trust it. Works without rebooting on Android 14.

6

Install User Cert

Opens the Android settings menu to manually install a cert. Required for Chrome on newer Android versions due to specific trust policies.

🧠 How It Works (The Technical Deep Dive)

Installing system certificates became significantly harder in recent Android versions. This script uses advanced Linux namespace manipulation to bypass these restrictions.

The Android 14+ Challenge ("Split-Brain" & Immutable APEX)

In Android 14, system certificates moved from /system/etc/security/cacerts to an immutable APEX container at /apex/com.android.conscrypt/cacerts.

Challenge 1: You cannot remount /apex as Read-Write. It is permanently locked.

Challenge 2: Apps see the APEX path, but some legacy apps and Chrome still check the old /system path.

The Solution: Systemless RAM Overlay & Live Injection

Instead of modifying the actual file system, this script performs a "Live Injection":

RAM Overlay (tmpfs): We create a temporary writable filesystem in the device's RAM (/data/local/tmp/cert_overlay).

Clone & Patch: We copy all original certificates + your new Proxy CA into this RAM disk.

Dual Bind Mount: We use mount --bind to plaster this RAM disk over the top of the read-only system folders. We mount it twice to solve the "Split-Brain" issue:

Mount over /apex/com.android.conscrypt/cacerts (For the OS & Apps).

Mount over /system/etc/security/cacerts (For Chrome compatibility).

Namespace Injection (nsenter):

Android apps run in isolated "Mount Namespaces". A standard mount command in ADB shell is invisible to apps.

We use nsenter to "enter" the namespace of the Zygote process (the parent of all apps). We perform the mount inside Zygote.

This ensures that any new app you launch inherits our modified certificate store.

We also loop through all currently running apps and inject the mount into their live namespaces, so you don't even need to reboot the device.

⚠️ Limitations

[!WARNING]

Temporary Injection > Since this uses a RAM overlay, the injection is lost upon reboot. You must re-run Option 5 every time you restart the device (unless you use a permanent Magisk module).

[!NOTE]

Chrome/WebView Behavior > Chrome has its own aggressive Certificate Pinning and Root Store integrity checks. If Chrome rejects the System cert (showing ERR_CERT_AUTHORITY_INVALID), use Option 6 to install it as a User Certificate as well.
