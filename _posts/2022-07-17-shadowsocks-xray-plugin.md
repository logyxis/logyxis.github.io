## Shadowsocks over WebSocket with TLS Using Xray Plugin on CentOS, Rocky Linux, or AlmaLinux

Here is how to make Shadowsocks use WebSocket and TLS with the Xray Plugin and the Teddysun packages on a CentOS, Rocky Linux, or AlmaLinux server. I assume as a starting point that:

* You already own a domain name
* You have opened an account with [Cloudflare](https://www.cloudflare.com)
* You have added your domain name to your Cloudflare account
* You have rented a server, which is typically a virtual private server (VPS)
* The server runs CentOS, Rocky Linux, or AlmaLinux
* Ports 80 and 443 are open for input on the server
* A DNS record points from the server hostname to the server IP address, with Cloudflare providing DNS services only, not proxying

### 1. Generate Cloudflare API Token

First we're going to obtain a Cloudflare API token so that the Automatic Certificate Management Environment (ACME) client has permission to manage your DNS records at Cloudflare.

1. Log in to your Cloudflare account
2. Go to **My Profile**
3. Select **API Tokens**
4. Click **Create Token**
5. On the line marked **Edit zone DNS**, click **Use template**
6. Under **Permissions**, set **Zone** and **DNS** to **Edit**
7. Under **Zone Resources**, set **Include** and **Specific zone** to your domain name, e.g. `example.com`
8. Click **Continue to summary**
9. Review the summary, and click **Create Token**
10. Copy the displayed token for the Cloudflare API (for security reasons, it will never be shown again!)

The token will be a 40-character string that functions like a password, e.g. `GI1wnt69IjOkVCqSvIv6NqSTetdp7s6OER4WttVp`.

### 2. Obtain TLS Certificate

By default, the xray plugin will look for TLS certificates signed by acme.sh. Therefore we use Acme.sh. This is a shell script that implements the Automatic Certificate Management Environment protocol. The GitHub repo containing the script is located at https://github.com/acmesh-official/acme.sh. 

Use the `export` command to set a shell variable containing the value of the Cloudflare API token. Using our example value from above:

```bash
export CF_Token="GI1wnt69IjOkVCqSvIv6NqSTetdp7s6OER4WttVp"
```

Check that the shell variable is now set to the expected value:

```bash
echo $CF_Token
```

Install the prerequisites for the script:

```bash
yum install wget curl tar socat
```

Download the Acme.sh script:

```bash
wget -O- https://get.acme.sh | sh
```

This creates a directory, `.acme.sh`, containin the script and associated files.

Register your email address with ZeroSSL:

```bash
~/.acme.sh/acme.sh --register-account -m example@gmail.com
```

Get an SSL certificate from ZeroSSL:

```bash
~/.acme.sh/acme.sh --issue --dns dns_cf -d host.example.com
```

The script temporarily adds a DNS `TXT` record named `_acme-challenge` to your domain's Cloudflare DNS entries. This record is removed after verification.

The script places your certificate and keys as follows:

* Certificate `/root/.acme.sh/host.example.com/host.example.com.cer`
* Private key `/root/.acme.sh/host.example.com/host.example.com.key`
* Intermediate CA certificate `/root/.acme.sh/host.example.com/ca.cer`
* Full chain certificate `/root/.acme.sh/host.example.com/fullchain.cer`

The SSL certificates come with ZeroSSL listed as the official certificate authority. You can review the certificates with the `openssl` command. For example:

```bash
openssl x509 -in /root/.acme.sh/host.example.com/host.example.com.cer -noout -text
```

### 3. Install Packages

Add the Extra Packages for Enterprise Linux (EPEL) repository:

```bash
yum install yum-utils epel-release
```

Enable EPEL repository:

```bash
yum-config-manager --enable epel
```

Add the Teddysun repository:

```bash
yum-config-manager --add-repo https://dl.lamp.sh/shadowsocks/teddysun.repo
```

Download and make usable all the metadata for the currently enabled repositories:

```bash
yum makecache
```

List packages from the Teddysun repository:

```bash
yum repo-pkgs teddysun list
```

Install the packages we need for this tutorial:

```bash
yum install shadowsocks-libev xray-plugin
```

Check the version numbers. For shadowsocks-libev, display the help text to see the version number:

```bash
ss-server -h
```

For the xray plugin, you can just display the version number directly:

```bash
xray-plugin -version
```

### 4. Configure Server

Edit the supplied configuration file:

```bash
vi /etc/shadowsocks/shadowsocks-libev-config.json
```

Use the following as a model:

```json
{
        "server":"0.0.0.0",
        "server_port":443,
        "password":"barfoo!",
        "timeout":"600",
        "method":"aes-256-gcm",
        "fast_open":false,
        "mode":"tcp_and_udp",
        "plugin":"xray-plugin",
        "plugin_opts":"server;tls;host=host.example.com"
}
```

### 5. Run Server

Start the service:

```bash
systemctl start shadowsocks-libev-server
```

Enable start after reboot:

```bash
systemctl enable shadowsocks-libev-server
```

Check the status of the service:

```bash
systemctl status shadowsocks-libev-server
```

### 6. Client

The xray plugin is available for 16 platforms:

* xray-plugin-darwin-amd64
* xray-plugin-darwin-arm64
* xray-plugin-freebsd-386
* xray-plugin-freebsd-amd64
* xray-plugin-linux-386
* xray-plugin-linux-amd64
* xray-plugin-linux-arm
* xray-plugin-linux-arm64
* xray-plugin-linux-mips
* xray-plugin-linux-mips64
* xray-plugin-linux-ppc64le
* xray-plugin-linux-s390x
* xray-plugin-windows-386
* xray-plugin-windows-amd64
* xray-plugin-windows-arm
* xray-plugin-windows-arm64

Download Shadowsocks and the xray plugin for your platform, and set the parameters of the client to match those of the server.

We will give details for a Windows client for our example.

You need to pre-install [7-Zip](https://www.7-zip.org) on Windows. Then:

1. Download plugin version `xray-plugin-windows-amd64` from https://github.com/teddysun/xray-plugin/releases
2. Use 7-Zip to extract `xray-plugin-windows-amd64-v1.5.9.tar`
3. Use 7-Zip on the inner archive to extract `xray-plugin_windows_amd64.exe`
4. Download the Shadowsocks client for Windows from https://github.com/shadowsocks/shadowsocks-windows/releases
5. Extract the zip file into folder `Downloads\Shadowsocks-4.4.1.0`
6. Copy `xray-plugin_windows_amd64.exe` into folder `Shadowsocks-4.4.1.0`
7. Start `Shadowsocks.exe`
8. Click **More info**
9. Click **Run anyway**

Whichever platform you are using, you will need to configure your client to match your server. For example:

* Server address `host.example.com` or IP address
* Server port `443`
* Password as per server config file
* Encryption as per server config file e.g. `aes-256-gcm`
* Plugin `xray-plugin_windows_amd64.exe`
* Plugin Options `tls;host=host.example.com`

