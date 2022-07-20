## OpenVPN Server on Windows

**THE PROCEDURE IN THIS POST HAS NOT BEEN TESTED**

### 1. Dynamic DNS Service

You probably don't have a static IP address at home. In this step, you'll subscribe to a Dynamic DNS service. That will give your Window computer a fixed DNS host name, even if its IP address changes from time to time.

Open a browser and visit https://www.noip.com.

Sign up for an account. 

Download No-IP's Dynamic Update Client (DUC). Install it on your PC, leaving the boxes checked to launch DUC and run DUC as a system service in the background.

After launching the DUC, sign in with the username and password you chose for the No-IP site.

Select your host name, and click Save. Close the DUC window. The DUC continues to run in the system tray, which is the area at the bottom right of your Windows desktop.

### 2. Windows Defender Firewall with Advanced Security

We are using UDP port 1194 in this example. You have already opened that port in your router and port forwarded it to your PC. Now open that port on your PC.

1. In the Windows search box, type `firewall`.
2. Select **Windows Defender Firewall with Advanced Security**.
3. In the left pane, click **Inbound Rules**.
4. In the right pane, click **New Rule**.
5. Select the type **Port**, and click **Next**.
6. Select the option for the type `UDP` and specific local port `1194`, and click **Next**.
7. Select the action **Allow the connection**, and click **Next**.
8. Leave all three network locations checked, and click **Next**.
9. Type the name `OpenVPN Inbound`, and click **Finish**.
10. Close **Windows Defender Firewall with Advanced Security**.

### 3. Install OpenVPN

Open your browser, and go to the OpenVPN Community Downloads page at https://openvpn.net/community-downloads.

Download the installer for Windows. After the download is complete, run the installer.
When you see the choice between **Install Now** and **Customize**, click **Customize**.

Customize the components to be installed:

* Unselect the OpenVPN GUI
* Select the OpenVPN Service
* Select the OpenSSL Utilities
* Select the EasyRSA3 Certificate Management Scripts

Install with these options.

### 4. Public Key Infrastructure

The EasyRSA scripts are installed in `C:\Program Files\OpenVPN\easy-rsa`.

To use EasyTLS as well as EasyRSA, download `easytls` and `easytls-openssl.cnf` from https://github.com/TinCanTech/easy-tls/releases.

Copy and paste the downloaded EasyTLS files into `C:\Program Files\OpenVPN\easy-rsa`. This needs administrator permissions.

In the Windows search box, type `cmd`. Right-click on **Command Prompt**, and select **Run as administrator**.

In the Command Prompt Window running as administrator, enter the commands:

```bash
cd C:\Program Files\OpenVPN\easy-rsa
```

```bash
EasyRSA-Start.bat
```

This invokes a POSIX-compliant EasyRSA Shell. You will see a # command prompt.

Initialize the folder C:/Program Files/OpenVPN/easy-rsa/pki by entering the command:

```bash
./easyrsa init-pki
```

Build your Certificate Authority by entering the command:

```bash
./easyrsa build-ca nopass
```

Generate a signing request for the server certificate. We will name the server server1:

```bash
./easyrsa gen-req server1
```

Sign the certificate request as type server:

```bash
./easyrsa sign-req server server1
```

Generate a signing request for the client certificate. We will name the client client1:

```bash
./easyrsa gen-req client1
```

Sign the request as the type client:

```bash
./easyrsa sign-req client client1
```

Generate the Diffie-Hellman parameters:

```bash
./easyrsa gen-dh
```

Initialize Easy-TLS:

```bash
./easytls init-tls
```

Create a TLS-crypt key:

```bash
./easytls build-tls-crypt
```

Exit the EasyRSA shell:

```bash
exit
```

Remove the passphrase from the private key of the server, otherwise the service won't start.

```bash
openssl rsa -in pki\private\server1.key -out pki\private\server1nopass.key
```

```bash
copy pki\private\server1nopass.key pki\private\server1.key /y
```

Close the Command Prompt window.

### 5. Server Configuration File

Open Windows Notepad. Using the following as a model for your configuration file. Replace all the values in the template with the actual values that apply in your environment:

```bash
dev tun
proto udp
port 1194
ca "C:\\Program Files\\OpenVPN\\easy-rsa\\pki\\ca.crt"
cert "C:\\Program Files\\OpenVPN\\easy-rsa\\pki\\issued\\server1.crt"
key "C:\\Program Files\\OpenVPN\\easy-rsa\\pki\\private\\server1.key"
dh "C:\\Program Files\\OpenVPN\\easy-rsa\\pki\\dh.pem"
tls-crypt "C:\\Program Files\\OpenVPN\\easy-rsa\\pki\\easytls\\tls-crypt.key"
cipher AES-256-GCM
server 10.8.0.0 255.255.255.0
ifconfig-pool-persist ipp.txt
push "redirect-gateway def1"
push "dhcp-option DNS 8.8.8.8"
push "dhcp-option DNS 1.1.1.1"
push "block-outside-dns"
keepalive 10 60
persist-key
persist-tun
explicit-exit-notify 1
verb 3
```

Save the file as `server1.ovpn`.

Copy and paste the file into `C:\Program Files\OpenVPN\config-auto`. This requires administrator permissions.

### 6. Enable Forwarding

Open the Windows Control Panel app.

Go to the Network and Internet section, then click on Network and Sharing Center. Click the link Change adapter settings.

You will see that the installation has created a new network adapter. It will have a name OpenVPN TAP-Windows6. It is described as TAP-Windows Adapter V9.

In the Windows search box, type `cmd`. Right-click on **Command Prompt**, and select **Run as administrator**.

In the Command Prompt Window running as administrator, enter the command:

```bash
netsh int ipv4 show int
```

Note down the index number of OpenVPN TAP-Windows6. For example, it might be interface index 11. (It will not necessarily be the same number on your computer.)

See if Forwarding is enabled:

```bash
netsh int ipv4 show int 11 | findstr "Forwarding"
```

If Forwarding is disabled, then enable Forwarding by issuing the command:

```bash
netsh int ipv4 set int 11 Forwarding="enabled"
```

Double-check that the results show Forwarding is enabled:

```bash
netsh int ipv4 show int 11 | findstr "Forwarding"
```

Close the Command Prompt window.

### 7. Enable Routing

In the Windows search box, type `regedit`, right-click on **Registry Editor**, and select **Run as administrator**.

Expand the tree in the left pane. Navigate to `HKEY_LOCAL_MACHINE` > `SYSTEM` > `CurrentControlSet` > `Services` > `Tcpip` > `Parameters`.

Find the key `IPEnableRouter` of type **REG_DWORD**.

Set its value to `1`. Click **OK**.

Close the Registry Editor.

### 8. Start Services

In the Windows search box, type `services`, right-click the **Services** app, and select **Run as administrator**.

Find the row for **Routing and Remote Access**.

1. Right-click on it, and select **Properties**
2. Set the **Startup type** to **Automatic**
3. Click **Apply**
4. Click **Start**
5. Click **OK**

Find the row for **OpenVPN Interactive service**.

1. Right-click on it, and select **Properties**
2. Click **Stop**
3. Change its **Startup type** to **Manual**
4. Click **Apply**
5. Click **OK**

Find the row for **OpenVPNService**. Make sure it is **Running** and **Startup type** is **Automatic**. Click the button to **Restart** the service.

Close the **Services** app.

Open File Explorer and navigate to `C:\Program Files\OpenVPN\log`. Check the file `server1.log` for messages. 

If the OpenVPN service encounters fatal errors, it will write them to the event log. This can be viewed in the Windows Event Viewer (`eventvwr`). The OpenVPN events are under **Windows Logs** > **Application** with a source of `OpenVPNService`.

### 9. Share Internet Adapter

Open the Windows Control Panel app.

In the Network and Internet section, click Network and Sharing Center. Click the link Change adapter settings.

On your main adapter (wifi or Ethernet), right-click, and select **Properties**.

1. Select the **Sharing** tab
2. Check the box for **Allow other network users to connect through this computer's Internet connection**
3. Select **OpenVPN TAP-Windows6**
4. Click **OK**

### 10. Disable and Enable TAP-Windows Adapter V9

On the OpenVPN TAP-Windows6 adapter (or whatever your TAP-Windows Adapter V9 is named), right-click, then disable and reenable the adapter.

Close the Network Connections and Network and Sharing Center windows.

### 11. Create Client Configuration

On your Windows computer, open Notepad, and create a configuration file for the client. Use the template below as a model. You will need to change YOUR.DYNAMIC.DNSNAME to match your situation.

```bash
client
dev tun
proto udp
remote YOUR.DYNAMIC.DNSNAME 1194
resolv-retry infinite
nobind
persist-key
persist-tun
<ca>
-----BEGIN CERTIFICATE-----
MIIG...
-----END CERTIFICATE-----
</ca>
<cert>
-----BEGIN CERTIFICATE-----
MIIH...
-----END CERTIFICATE-----
</cert>
<key>
-----BEGIN PRIVATE KEY-----
MIIJ...
-----END PRIVATE KEY-----
</key>
<tls-crypt>
-----BEGIN OpenVPN Static key V1-----
89c8...
-----END OpenVPN Static key V1-----
</tls-crypt>
remote-cert-tls server
cipher AES-256-GCM
verb 3
```

You will need to copy and paste in your values from the files `ca.crt`, `client1.crt`, `client1.key`, and `tls-crypt.key` in the subfolders of `C:\Program Files\OpenVPN\easy-rsa\pki`. Ignoring any descriptive comments in the contents of the files you paste into the configuration.

Save the file in your server's `Downloads` folder as `client1.ovpn`.

### 12. Prevent Computer from Going to Sleep

You do not want your computer to be asleep when you try to connect from the outside world. Therefore configure its power plan to prevent the PC from ever going to sleep.

Right-click on the Windows Start button, and select the Settings app. Under the System section, select Power & sleep. You can set the options so the screen turns off after a few minutes, but configure your computer so that it never goes to sleep.

### 13. Port Forward on Router

You can change the port OpenVPN uses, but in this post we'll use the default, which is UDP port 1194. You need to do a couple of things on your home router.

1. Open your router firewall on UDP port 1194.
2. Configure your router to forward UDP port 1194 from the outside world (often called the wide-area network or WAN) to your PC on the local area network or LAN.

To determine your computer's address on the LAN, open a Windows Command Prompt and issue the command `ipconfig`. Your LAN address will fall in one of the ranges `10.0.0.0` through `10.255.255.255`, `172.16.0.0` through `172.31.255.255`, or `192.168.0.0` through `192.168.255.255`.

The procedures for opening the firewall and configuring port forwarding vary from router to router, so detailed instructions cannot be given here. Consult the documentation for your particular make and model of router.

### 14. Test Client-Server Connection

Install the OpenVPN client app on your client device. It is available for all major platforms.

Securely download the client OVPN file from server to client.

Import the client OVPN Profile.

Test your client-server connection.
