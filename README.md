# openvpn-manual-setup
## Windows

### Server Setup

❗Only tested **OpenVPN 2.5.7(amd64)** on **Windows Server 2022**

1. Download and install [OpenVPN](https://openvpn.net/community-downloads/). Check all the check boxs during installation.
1. Download and install [Git](https://git-scm.com/) (Actually we only need `openssl.exe` inside it)
1. Copy `C:/Program Files/Git/usr/bin/openssl.exe` into `C:/Program File/OpenVPN/easy-rsa`
1. Download the repository https://github.com/OpenVPN/easy-rsa-old
1. Copy all the files under `easy-rsa-old/easy-rsa/Windows/` into `C:/Program File/OpenVPN/easy-rsa`
1. Open cmd and cd to `C:/Program File/OpenVPN/easy-rsa`
1. Run `init-config`
1. Run `vars`
1. Run `clean-all`
1. Run `build-ca`, you will see something similar to following: 
    ```
    ai:easy-rsa # ./build-ca
    Generating a 1024 bit RSA private key
    ............++++++
    ...........++++++
    writing new private key to 'ca.key'
    -----
    You are about to be asked to enter information that will be incorporated
    into your certificate request.
    What you are about to enter is what is called a Distinguished Name or a DN.
    There are quite a few fields but you can leave some blank
    For some fields there will be a default value,
    If you enter '.', the field will be left blank.
    -----
    Country Name (2 letter code) [KG]:
    State or Province Name (full name) [NA]:
    Locality Name (eg, city) [BISHKEK]:
    Organization Name (eg, company) [OpenVPN-TEST]:
    Organizational Unit Name (eg, section) []:
    Common Name (eg, your name or your server's hostname) []:OpenVPN-CA
    Email Address [me@myhost.mydomain]:
    ```
    Note that in the above sequence, most queried parameters were defaulted to the values set in the varsor vars.bat files. The only parameter which must be explicitly entered is the Common Name. In the example above, I used "OpenVPN-CA".
1. Run `build-key-server server`

    As in the previous step, most parameters can be defaulted. When the Common Name is queried, enter "server". Two other queries require positive responses, "Sign the certificate? [y/n]" and "1 out of 1 certificate requests certified, commit? [y/n]".
1. RUN `build-dh`
1. cd to `C:/Program Files/OpenVPN/bin`
1. Run `openvpn --genkey tls-auth ta.key`
1. Create firewall rules on port 1194 (Allow both TCP & UDP)
1. In Windows **Settings** -> **Network & Internet** -> **Change adapter options**, right click the adapter for internet (Usually named **Ethernet**). In tab **Sharing**, check the box of **Allow other network users to connect through this computer's Internet connection**. Then Select **OpenVPN TAP-Windows6** for **Home networking connection**. Press **OK**.
1. Replace content of "C:/Program File/OpenVpn/sample-config/server.ovpn" with following:
    ```
    port 1194
    proto udp
    dev tun
    ca "C:\\Program Files\\OpenVPN\\easy-rsa\\keys\\ca.crt"
    cert "C:\\Program Files\\OpenVPN\\easy-rsa\\keys\\server.crt"
    key "C:\\Program Files\\OpenVPN\\easy-rsa\\keys\\server.key" 
    dh "C:\\Program Files\\OpenVPN\\easy-rsa\\keys\\dh2048.pem"
    topology subnetserver 10.8.0.0 255.255.255.0
    ifconfig-pool-persist ipp.txt
    push "redirect-gateway autolocal def1 bypass-dhcp"
    keepalive 10 120
    tls-auth "C:\\Program Files\\OpenVPN\\bin\\ta.key" 0
    data-ciphers-fallback AES-256-CBC
    persist-key
    persist-tun
    status openvpn-status.log
    verb 3
    explicit-exit-notify 1
    ```
1. Double click on "C:/Program File/OpenVpn/sample-config/server.ovpn" and confirm the import.
1. Right click the OpenVPN icon in system tray. Select **server** -> **Connect**.

### Create client .open file
1. Open cmd and cd to `C:/Program File/OpenVPN/easy-rsa` 
2. Run `build-key YOUR_CLIENT_NAME`
3. Create the `FILENAME.ovpn` somewhere with following content(Fill in  SERVER_IP_OR_HOSTNAME / content inside xml tag):
```
client
proto udp
remote SERVER_IP_OR_HOSTNAME 1194
dev tun
resolv-retry infinite
nobind
persist-key
persist-tun
remote-cert-tls server
tls-client
data-ciphers-fallback AES-256-CBC
verb 3
<ca>
**Content of C:/Program Files/OpenVPN/easy-rsa/keys/ca.crt**
</ca>
<cert>
**Content of C:/Program Files/OpenVPN/easy-rsa/keys/YOUR_CLIENT_NAME.crt**
</cert>
<key>
**Content of C:/Program Files/OpenVPN/easy-rsa/keys/YOUR_CLIENT_NAME.key**
</key>
<tls-crypt>
**Content of C:/Program Files/OpenVPN/easy-rsa/bin/ta.key**
</tls-crypt>
```