# Moniker Link (CVE-2024-21413) Exploit Demo

This repository contains a demo exploit for the Moniker Link vulnerability (CVE-2024-21413), created as part of course project for Penetration Testing at the University of Sourth Brittany.

The repository includes:
- Links to VM snapshots used for the demo setup
- The Python scripts used to execute the exploit

## Vulnerability Description

CVE-2024-21413 is a vulnerability in Microsoft Office where crafted links inside document files (e.g., `.odt`) can exploit the Windows "Moniker Link" behavior.  
When a user opens a malicious document, it can trigger external network connections (such as to an SMB server) without additional user interaction, potentially leaking sensitive information like NTLM hashes or allowing arbitrary code execution under certain conditions.

## Repository Contents

- `exploits` folder — Python scripts to craft and deliver the malicious payload
- Links to VM snapshots (Windows 10 and Kali Linux)
- Setup instructions

# Setup Instructions

Follow these steps to recreate the environment:

## VirtualBox Network Setup (Allow VMs to Interact)

To allow the Windows and Kali VMs to communicate directly, you need to configure their network settings in VirtualBox.

Option 1: Bridged Adapter
- Open VirtualBox and select your VM.
- Go to Settings -> Network.
- Set "Attached to" as "Bridged Adapter".
- Choose the physical network interface you are using (e.g., Wi-Fi or Ethernet).
- Repeat this for both VMs.
- This will make each VM appear as a separate device on your local network.

Option 2: Host-Only Adapter
- Open VirtualBox and go to File -> Host Network Manager.
- Create a new Host-Only Network (e.g., vboxnet0).
- In each VM:
  - Go to Settings -> Network.
  - Set "Attached to" as "Host-Only Adapter".
  - Choose the created host-only network (e.g., vboxnet0).
- This will allow VMs to communicate with each other and the host, but not with external internet unless you configure NAT separately.

Important:
- After setting up, boot both VMs and check their IP addresses (`ipconfig` on Windows, `ip a` on Kali).
- Ensure they are on the same subnet (e.g., 192.168.137.x).
- If needed, adjust firewall settings to allow ICMP (ping) and SMTP traffic.


## Windows 10 (Victim VM)

1. Edit Hosts File
   - Navigate to: `C:\Windows\System32\drivers\etc\`
   - Open Notepad as Administrator
   - Set file type to "All Files" and open the `hosts` file
   - Add the following line (replace with your Kali VM's IP):
     ```
     192.168.137.10    kali.local
     ```

2. Flush DNS Cache
   ```cmd
   ipconfig /flushdns
   ```
3. Test Connection: Open Command Prompt and run:
    ```cmd
    ping kali.local
    telnet kali.local 25
    ```

4. Prepare the Email Environment
    - Install a vulnerable Microsoft Office version (ensure .odt file handling is enabled).
    - Add a new mail account in Outlook:
        - Choose Advanced Setup -> Manual Configuration
        - Select IMAP as the account type
        - Encryption: None
        - Authentication: None (disable if possible)
        - Incoming Server: `kali.local`, Port 143
        - Outgoing Server (SMTP): `kali.local`, Port 25
        - Username: `testuser`
        - Password: Password set on the Kali server for testuser

## Kali Linux (Attacker VM)

1. Set the Hostname
    - Check hostname:
        ```bash
        hostname
        ```
    - If not kali.local, set it:
        ```bash
        sudo hostnamectl set-hostname kali.local
        ```

2. Update `/etc/hosts`
    - Open the hosts file:
        ```bash
        sudo nano /etc/hosts
        ```
    - Add your current Kali IP address:
        ```bash
        192.168.137.10    kali.local
        ```
    - Then restart networking:
        ```bash
        sudo systemctl restart networking
        ```

3. Configure Postfix
    - Edit Postfix main configuration:
        ```bash
        sudo nano /etc/postfix/main.cf
        ```
    - Update the mynetworks setting:
        ```nginx
        mynetworks = 192.168.137.0/24, 127.0.0.0/8, [::ffff:127.0.0.0]/104, [::1]/128
        ```
        Replace 192.168.137.0/24 with your appropriate network if needed.
    - Restart Postfix and Dovecot:
        ```bash
        sudo systemctl restart postfix
        sudo systemctl restart dovecot
        ```

# Running the Exploit

1. Adjust the Exploit Script
- Open the `exploit.py` script.
- Locate the section where the payload or target IP is defined.
- Replace any hardcoded IP addresses with the current IP address of your Kali VM (e.g., 192.168.137.10).

2. Start Responder
- In a new terminal on Kali, run:
    ```bash
    sudo responder -I eth0
    ```
- Replace `eth0` with your actual network interface if needed (use `ip a` to find it).

3. Execute the Exploit
- In another terminal on Kali, execute the exploit script:
    ```bash
    python3 exploit.py
    ```
- This will send the malicious email to the victim.

4. Capture the Hash
- Monitor the Responder window.
- When the victim interacts with the malicious link, Responder will capture the NTLM hash.


# Disclaimer

This project was developed for educational purposes only as part of an academic pentesting course.
Use responsibly and only in controlled environments where you have permission.
