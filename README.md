# ADB Proxy & Certificate Manager

> A robust, cross-platform shell script designed to automate HTTP proxy configuration and System Certificate installation on Android devices.

This tool is designed for penetration testers and developers using proxy tools such as **Burp Suite**, **HTTP Toolkit**, **MitMProxy**, or **Charles Proxy**.

---

## 🚀 Features

* **🔌 One-Click Proxy:** Toggle HTTP Proxy settings on your device instantly.
* **🛡️ Android 14+ Support:** Bypasses new read-only APEX certificate stores using `nsenter` namespace injection.
* **💉 Live Injection:** Installs System Certificates without rebooting the device (Temporary).
* **♾️ Permanent Mode:** Auto-generates and installs a custom Magisk module to persist certificates across reboots.
* **🔧 Chrome Fix:** Includes a built-in helper to install User Certificates for browsers that distrust System stores.
* **📱 Multi-Device:** Easily switch between multiple connected emulators or physical devices.
* **🧠 Universal:** Auto-detects the host OS version (Android 10-14+) and applies the correct mounting strategy.

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
    6) Install System Certificate (Magisk Module)
    7) Install User Certificate (Chrome Fix)
    8) Switch Device
    m) Show menu
    q) Quit
    
    ?#
    ```

3.  **Follow the interactive menu:**
    * **Option 1 (Start Proxy):** Sets the global HTTP proxy on the device.
    * **Option 5 (Install System Cert - Temp):** *The "Live Injection".* Injects your proxy's CA into the System Trust Store via RAM overlay. Works instantly but resets on reboot.
    * **Option 6 (Install System Cert - Permanent):** *The "Magisk Mode".* Generates and installs a Magisk Module that automatically injects the certificate on every boot. Ideal for long-term setups.
    * **Option 7 (Install User Cert):** Opens Android settings to manually install a cert (Required for Chrome/WebView compatibility).
    * **Option 8 (Switch Device):** Toggle between multiple connected devices.

---

## 🧠 How It Works

Installing system certificates became significantly harder in recent Android versions. This script uses two different strategies to bypass restrictions.

### Strategy A: Live Injection (Option 5)
*Best for quick testing.*

* **RAM Overlay:** Creates a temporary writable filesystem in the device's RAM.
* **Dual Bind Mount:** Mounts this RAM disk over the top of both the read-only APEX path (for the OS) and the Legacy path (for Chrome).
* **Namespace Injection:** Uses `nsenter` to "enter" the namespace of the Zygote process and injects the mount into all currently running apps immediately.

### Strategy B: Permanent Magisk Module (Option 6)
*Best for daily driving or persistent setups.*

* **Module Generation:** Creates a custom Magisk module structure directly on the device (`/data/adb/modules/adb_proxy_cert`).
* **Boot Script:** Writes a `post-fs-data.sh` script that runs on every boot.
* **Auto-Sync:** This script detects Android version (Legacy vs. 14+) and mounts the certificate automatically before apps start. It also copies any User Certificates you install manually into the System store.

---

## ⚠️ Limitations

* **Temporary Injection (Option 5):** Uses a RAM overlay, so the injection is lost upon reboot.
* **Permanent Injection (Option 6):** Persists across reboots but requires Magisk.
* **Chrome/WebView:** Chrome has aggressive Certificate Pinning. If Chrome rejects the System cert (showing `ERR_CERT_AUTHORITY_INVALID`), use **Option 7** to install it as a User Certificate as well.
