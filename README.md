# ADB Proxy & Certificate Manager

> A robust, cross-platform shell script designed to automate HTTP proxy configuration and System Certificate installation on Android devices.

This tool is designed for penetration testers and developers using proxy tools such as **Burp Suite**, **HTTP Toolkit**, **MitMProxy**, or **Charles Proxy**.

---

## 🚀 Features

* **🔌 One-Click Proxy:** Toggle HTTP Proxy settings on your device instantly.
* **🛡️ Android 14+ Support:** Bypasses new read-only APEX certificate stores using `nsenter` namespace injection.
* **💉 Live Injection:** Installs System Certificates without rebooting the device.
* **🔧 Chrome Fix:** Includes a built-in helper to install User Certificates for browsers that distrust System stores.
* **💻 Cross-Platform:** Works on macOS, Linux, and Windows (WSL/Git Bash).
* **🧠 Universal:** Auto-detects the host OS version and applies the correct mounting strategy.

---

## 🛠 Prerequisites

Ensure you have the following installed on your host machine:

* **ADB (Android Debug Bridge):** Part of the Android SDK Platform-Tools.
* **OpenSSL:** To calculate certificate hashes.
* **Curl:** To download the certificate from your proxy tool.
* **Rooted Device/Emulator:** `su` access is required for certificate injection.

---

## ⬇️ Installation

Use your system's package manager to install the required tools:

### macOS
```bash
brew install android-platform-tools openssl curl
```
### Ubuntu/Debian
```bash
sudo apt install adb openssl curl
```
### Part 4: Configuration & Usage
## ⚙️ Configuration

By default, the script is configured for **Burp Suite** or **HTTP Toolkit** running on localhost. You can extend this to work with other proxies by editing the variables at the top of `adb_proxy.sh`:

| Variable | Description | Default |
| :--- | :--- | :--- |
| `PROXY_HOST` | Host machine IP relative to the emulator | `10.0.2.2` |
| `PROXY_PORT` | Port your proxy tool is listening on | `8080` |
| `CERT_URL` | URL to download the CA Certificate | `http://127.0.0.1:8080/cert` |

---

## 📖 Usage

1.  **Make the script executable:**
    ```bash
    chmod +x adb_proxy.sh
    ```

2.  **Run the script:**
    ```bash
    $ ./adb_proxy.sh
    === ADB Proxy & Cert Manager ===
    [INFO] Selected device: emulator-5554
    
    --- Actions ---
    1) Start Proxy
    2) Change Proxy Config
    3) Stop Proxy
    4) Get Proxy Status
    5) Install System Certificate (Live Injection)
    6) Install User Certificate (Chrome Fix Android 14+)
    m) Show menu
    q) Quit
    
    ?#
    ```

3.  **Follow the interactive menu:**
    * **Option 1 (Start Proxy):** Sets the global HTTP proxy on the device.
    * **Option 5 (Install System Cert):** *The Magic Button.* Injects your proxy's CA into the System Trust Store via RAM overlay.
    * **Option 6 (Install User Cert):** Opens Android settings to manually install a cert (Required for Chrome).

---

## 🧠 How It Works

Installing system certificates became significantly harder in recent Android versions. This script uses advanced Linux namespace manipulation to bypass restrictions.

### The Android 14+ Challenge

In Android 14, system certificates moved from `/system/etc/security/cacerts` to an immutable APEX container at `/apex/com.android.conscrypt/cacerts`.

1.  **Challenge 1:** You cannot remount `/apex` as Read-Write.
2.  **Challenge 2:** Apps see the APEX path, but some legacy apps and Chrome still check the old `/system` path (The "Split-Brain" issue).

### The Solution: Systemless RAM Overlay & Live Injection

Instead of modifying the actual file system, this script performs a "Live Injection":

* **RAM Overlay:** Creates a temporary writable filesystem in the device's RAM.
* **Dual Bind Mount:** Mounts this RAM disk over the top of *both* the read-only APEX path (for the OS) and the Legacy path (for Chrome).
* **Namespace Injection:** Uses `nsenter` to "enter" the namespace of the Zygote process (parent of all apps) and injects the mount, ensuring all new and running apps see the certificates immediately.

---

## ⚠️ Limitations

* **Temporary Injection:** Since this uses a RAM overlay, the injection is lost upon reboot. You must re-run **Option 5** every time you restart the device (unless you use a permanent Magisk module).
* **Chrome/WebView:** Chrome has aggressive Certificate Pinning. If Chrome rejects the System cert, use **Option 6** to install it as a User Certificate as well.
