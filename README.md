# 🔌 Cisco Switch Initial Setup – Physical Home Lab

**Author:** Kaden Baffour | [@KadenCyberNet](https://github.com/KadenCyberNet)  
**Hardware:** Cisco Catalyst Switch – IOS Version 12.2(55)SE7  
**Tools:** PuTTY, Windows Device Manager, Cisco IOS CLI  
**Difficulty:** Beginner  

---

## 📋 Project Overview

This project documents the full process of connecting to and performing initial configuration on a physical Cisco switch using a console cable and PuTTY. This is the foundational skill required before any network configuration can take place — if you can't access the CLI, you can't configure the device.

Everything shown here was performed on real physical hardware, not a simulator.

---

## 🎯 Objectives

- Connect a console cable from a laptop to a physical Cisco switch
- Identify the correct COM port using Windows Device Manager
- Configure PuTTY for serial console access
- Access the Cisco IOS CLI
- Perform initial switch hardening (hostname, enable secret, console password)
- Verify configuration using `show running-config`

---

## 🛠️ Equipment Used

| Item | Details |
|------|---------|
| Cisco Catalyst Switch | IOS Ver 12.2(55)SE7 |
| Console Cable | RJ45 to USB (DIKWAN adapter) |
| Laptop | Lenovo – Windows 11 |
| Terminal Emulator | PuTTY |

---

## 📸 Step-by-Step Walkthrough

---

### Step 1 – Connect the Console Cable to the Switch
![01_console_cable_connected](https://github.com/user-attachments/assets/455f54b6-d557-4692-b2ca-9c1aaf06cba8)

**What I did:**
I physically connected the rollover console cable into the **CONSOLE** port on the back of the Cisco switch.

**Why it matters:**
The console port is the only way to access a Cisco device that has never been configured before. Unlike SSH or Telnet which require a working IP address and network connection, the console port gives you **direct out-of-band access** to the CLI regardless of whether the switch has any configuration at all. This is how network engineers perform first-time setup and emergency recovery on real hardware. Without this connection, you cannot do anything else.

> **Note:** The sticker on the switch shows **IOS VER: 12.2(55)SE7** — always take note of your IOS version. Some commands and features differ between versions and knowing this upfront saves troubleshooting time later.

---

### Step 2 – Connect the USB Adapter to the Laptop

![02_usb_adapter_laptop](https://github.com/user-attachments/assets/59bf9b85-49d8-4e5e-90f4-19cd1890362a)

**What I did:**
I plugged the other end of the console cable into my laptop using a **DIKWAN USB-to-Serial adapter**.

**Why it matters:**
Modern laptops no longer have a built-in serial (DB9) port, which is what traditional Cisco console cables were designed for. This USB adapter bridges that gap by converting the RJ45 console signal into something a modern laptop can understand via USB. Without this adapter, the laptop has no way to communicate with the switch at all. This is a common real-world tool that network engineers carry in their kits. If the adapter does not have its drivers installed, Windows will not detect it and nothing will work — driver installation is always the first troubleshooting step if the device doesn't show up.

---

### Step 3 – Identify the COM Port in Device Manager

![03_device_manager_com3](https://github.com/user-attachments/assets/8e0c1d3c-1fa3-4b30-bad4-87f583f381ae)

**What I did:**
I opened **Windows Device Manager** and expanded **Ports (COM & LPT)** to find which COM port Windows assigned to the USB adapter.

**Why it matters:**
PuTTY needs to know exactly which COM port to communicate through. Windows automatically assigns a COM port number when you plug in a USB serial adapter, but that number is not always the same — it can change depending on which USB port you use or what other devices are connected. In this case Windows assigned **COM3**. If you skip this step and guess the wrong COM port in PuTTY, you will get no response from the switch even though everything is physically connected correctly. Always verify the COM port here before opening PuTTY.

---

### Step 4 – Configure PuTTY for Serial Console Access

![04_putty_serial_config](https://github.com/user-attachments/assets/2fd61156-40b6-4612-b0f0-36f2128d4e98)

**What I did:**
I opened PuTTY, selected **Serial** as the connection type, set the serial line to **COM3**, and set the speed to **9600**.

**Why it matters:**
PuTTY defaults to SSH mode which will not work for a console connection. Switching to Serial mode tells PuTTY we are connecting through a physical cable, not a network. The baud rate of **9600** is the default speed that Cisco switches communicate at on the console port. If the baud rate in PuTTY does not match the switch, you will see scrambled unreadable characters instead of a proper CLI. These settings — 9600 baud, 8 data bits, no parity, 1 stop bit — are standard across virtually all Cisco devices and are important to memorize for the CCNA exam and real-world work.

---

### Step 5 – Gain CLI Access to the Switch

![05_putty_connected](https://github.com/user-attachments/assets/d25c25ff-6e91-4b4c-96b7-b3431ead06df)

**What I did:**
After clicking Open in PuTTY, the terminal window appeared and displayed **"S1 con0 is now available"**. I pressed Enter and received the switch prompt `S1>`.

**Why it matters:**
This screen confirms that the entire physical and software chain is working — the cable is good, the adapter is recognized, the COM port is correct, and PuTTY is configured properly. The message `con0 is now available` means we are connected to **console port 0**, the physical management interface on the switch. The prompt `S1>` indicates we are in **User EXEC mode**, which is the lowest privilege level. From here we can move into Privileged EXEC mode (`enable`) and then Global Configuration mode (`config t`) to start making changes. This moment — seeing that prompt for the first time — is the entry point to everything else in networking.

---

### Step 6 – Initial Switch Configuration & Verification

![06_show_running_config](https://github.com/user-attachments/assets/036e629e-0f09-47a9-9e81-f8075e25b081)

**What I did:**
I entered the following commands to perform the initial hardening of the switch:

```
S1> enable
S1# config t
S1(config)# hostname SW1
SW1(config)# enable secret chicken1234
SW1(config)# line console 0
SW1(config-line)# password chicken1234
SW1(config-line)# login
SW1(config-line)# exit
SW1# write memory
SW1# show running-config
```

**Why it matters:**
An unconfigured switch is a security risk. By default there are no passwords set, meaning anyone who plugs a console cable in has full access to the device. These commands lock that down:

- `hostname SW1` — gives the switch a recognizable identity. In large networks with dozens of devices, a proper hostname tells you immediately which device you are working on.
- `enable secret` — encrypts the privileged mode password using MD5. This is always preferred over `enable password` which stores the password in plaintext and can be read directly from the config file.
- `line console 0` + `password` + `login` — forces anyone connecting via the console port to authenticate with a password before gaining access.
- `write memory` — saves everything to NVRAM so the configuration survives a power cycle. Without this, every change is lost on reboot.
- `show running-config` — displays the active configuration so you can verify every command took effect exactly as intended. In networking, always verify — never assume.

---

## 📝 Key Takeaways

- The console port is the only access method that works on an unconfigured device — it is the most important physical interface on any Cisco device
- Always check **Device Manager** for your COM port before opening PuTTY — guessing wastes time
- PuTTY serial settings must exactly match Cisco defaults: **9600 baud, 8 data bits, no parity, 1 stop bit**
- `enable secret` is always preferred over `enable password` — one is encrypted, one is plaintext
- `write memory` is non-negotiable — unsaved configs are lost on reboot
- `show running-config` should always be run after making changes to confirm they applied correctly

---

## 🔗 Related Projects

- [VLAN Segmentation Lab](https://github.com/KadenCyberNet)
- [Port Security & MAC Address Analysis](https://github.com/KadenCyberNet)
- [SSH Hardening & Secure Remote Management](https://github.com/KadenCyberNet)

---

*Part of the [KadenCyberNet](https://github.com/KadenCyberNet) home lab portfolio.*
