# Week 1: Lab Setup Virtualbox and kali Linux

**Author:** Nisaur Rohmah
**Program:** Networkwalks Cybersecurity Internship, Batch B084
**Task:** Week 1, WK1-PM1
**Date completed:** 29 October 2026

> A small, private practice lab built on my own laptop. Kali Linux is the attacker machine, and everything runs inside a virtual network that is isolated from other people's systems.

---

## 1. What I Built

I built a virtual lab where I can safely practise security tools later in the program. The lab has one machine for now (Kali Linux), but the network is prepared so that more machines, such as Windows targets, can be added without changing anything.

```
        Windows laptop (host)
        └── VirtualBox
            └── NAT Network "NatNetwork"  10.0.0.0/24
                ├── Kali Linux   10.0.0.2   (attacker)   <- built in this project
                └── future VMs   10.0.0.3 - 10.0.0.99    <- planned
```


---

## 2. Requirements Checklist

| # | Requirement from the task | How I met it | 
|---|---------------------------|--------------|
| 1 | Use VirtualBox (latest recommended) | Installed from virtualbox.org | 
| 2 | Kali Linux as attacker machine | Imported the pre-built Kali VM |
| 3 | Network in subnet 10.0.0.0/24 | Created a NAT Network with that prefix | 
| 4 | Use NAT Network | Adapter 1 attached to `NatNetwork` | 
| 5 | Clipboard and drag/drop enabled | Both set to Bidirectional | 
| 6 | Shared folder from host Downloads | Added Downloads as an auto-mounted shared folder | 
| 7 | Kali IP is 10.0.0.2/24 | Set a manual IPv4 address | 
| 8 | Full Internet access | `ping google.com` replies | 
| 9 | Snapshot of the VM | Took a snapshot after testing | 

---

## 3. My Environment

| Item | Value |
|------|-------|
| Host OS | Windows 11 |
| Host RAM / CPU | 8 GB / Intel Core i3 |
| VirtualBox | 7.2.22 |
| Kali Linux | 2026.3 (pre-built VirtualBox image) |
| VM resources | 2 CPUs, 2048 MB RAM, about 80 GB virtual disk |
| Network adapter | Intel PRO/1000 MT Desktop on NAT Network |

---

## 4. How I Did It

### 4.1 Tools
I installed **7-Zip** (to unpack the Kali download, which comes as a `.7z` file) and **VirtualBox**.

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/81e43cd8-8f9a-4e22-9ea0-78d1e5fa3b29" />


<img width="1920" height="1080" alt="Screenshot (1699)" src="https://github.com/user-attachments/assets/920086dc-3b1f-4510-8d77-c9649a07aa69" />

### 4.2 Creating the network first
In VirtualBox I opened **File > Tools > Network > NAT Networks** and created:

| Setting | Value |
|---------|-------|
| Name | NatNetwork |
| IPv4 prefix | 10.0.0.0/24 |
| DHCP | Enabled |
| IPv6 | Disabled |

I chose a NAT Network instead of plain NAT because machines on a NAT Network can talk to each other and still reach the Internet.

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/d1e9fd4c-ca8e-45fb-8803-b409fe12f059" />

### 4.3 Importing Kali
I extracted the Kali archive into its own folder (not Downloads, so it is not deleted by accident) and opened the `.vbox` file to add it to VirtualBox. Then I set **Network > Adapter 1** to *NAT Network* with the name *NatNetwork*.

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/e16833f5-6819-4f0f-b352-c15f7b3750c7" />


### 4.4 Host and VM integration
With the VM powered off, I changed:

- **General > Advanced:** Shared Clipboard = Bidirectional, Drag'n'Drop = Bidirectional
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/a311d5c6-96e2-4970-b3d3-bda0875bfaae" />

- **Network > Adapter 1:** Attached to = NAT Network, Name = NatNetwork, Promiscuous Mode = Allow All
<img width="1920" height="1080" alt="Screenshot (1707)" src="https://github.com/user-attachments/assets/8cf6a4bd-2fb9-43d7-b53b-66ebc360ae4d" />


- **Shared Folders:** added my host Downloads folder with Auto-mount on and Full access
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/e68692d2-ec9d-4c6d-b981-80691d698187" />

After booting, the folder showed up on the Kali desktop as `sf_Downloads`.

### 4.5 Static IP in Kali
Inside Kali I set the connection **Wired connection 1** to manual addressing:

```bash
sudo nmcli connection modify "Wired connection 1" \
  ipv4.method manual \
  ipv4.addresses 10.0.0.2/24 \
  ipv4.gateway 10.0.0.1 \
  ipv4.dns 8.8.8.8 \
  ipv4.dad-timeout 0
sudo nmcli connection up "Wired connection 1"
```

`ipv4.dad-timeout 0` turns off the duplicate-address wait, which is known to cause missing Internet on VirtualBox 7 with recent Kali versions.

<img width="1920" height="892" alt="Screenshot_2026-10-29_05_37_56" src="https://github.com/user-attachments/assets/1f58b312-4d53-4719-93fe-4a4aa1da5ba7" />

### 4.6 Snapshot
After all tests passed I shut the VM down and took a snapshot named **[fill in snapshot name]**, so I can return to this clean state at any time.

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/2e15904c-8568-459e-bcb2-08a94e9504ef" />

---

## 5. Testing

```bash
ip a
ping -c 4 google.com
```

- `eth0` is **UP** and holds `10.0.0.2/24`
- `google.com` resolved to an address and all pings were answered, so both DNS and Internet access work

<img width="1920" height="892" alt="Screenshot_2026-10-29_05_49_06" src="https://github.com/user-attachments/assets/81e2df6b-b29c-4a23-80b9-fb507979594a" />

---


## 7. Reflection

The biggest lesson was that order matters: the network should exist before the VM is attached to it, and the VM settings should be changed while it is powered off. I also learned that the graphical network editor and `nmcli` do the same job, but the command line is faster to repeat and easier to document. Finally, taking a snapshot only after verifying everything means the snapshot is something I can truly trust.

Next steps: add Windows and other target VMs to the same network and test connectivity between them.

---

## 8. Ethics

This lab is for learning. I only test machines that I own inside this virtual network, or that I have written permission to test.

---

## 9. References

- 7-Zip: https://7-zip.org/download.html
- VirtualBox: https://virtualbox.org/wiki/Downloads
- Kali Linux: https://kali.org/get-kali

*Instructor: Waqas Karim CCIE, Networkwalks*
