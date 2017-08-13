### Internet-LAN-Over-VPN-Using-OpenVPN


> Configuration

- 1 VPN Server
- 1 Server
- OS: CentOS 7


> Architecture

![MetaStore remote database](https://github.com/gamboabdoulraoufou/Internet-LAN-Over-VPN-Using-OpenVPN/blob/master/img/vpn_archi.png)


> Log as root user `VPN Server`

```sh
sudo su - root
```


> Intall pre-requisites `VPN Server`

```sh
# install the Extra Packages
yum install -y epel-release

# install openvpn
yum install -y openvpn 

# install easy-rsa
yum install -y easy-rsa

# install iptables
yum install -y iptables-services

```

> Configure OpenVPN `VPN Server`

```sh
# copy openvpn configuration file
cp /usr/share/doc/openvpn-*/sample/sample-config-files/server.conf /etc/openvpn

# edit file
vi /etc/openvpn/server.conf

# change the values for the bellow parameter and keep default configuration for the rest

########### START CHANGE INTO server.conf file ##########

# Which TCP/UDP port should OpenVPN listen on?
# If you want to run multiple OpenVPN instances
# on the same machine, use a different port
# number for each one.  You will need to
# open up this port on your firewall.
port 1196

# TCP or UDP server?
proto tcp
;proto udp

# SSL/TLS root certificate (ca), certificate
# (cert), and private key (key).  Each client
# and the server must have their own cert and
# key file.  The server and all clients will
# use the same ca file.
#
# See the "easy-rsa" directory for a series
# of scripts for generating RSA certificates
# and private keys.  Remember to use
# a unique Common Name for the server
# and each of the client certificates.
#
# Any X509 key management system can be used.
# OpenVPN can also use a PKCS #12 formatted key file
# (see "pkcs12" directive in man page).
ca /etc/openvpn/keys/ca.crt
cert /etc/openvpn/keys/server.crt
key /etc/openvpn/keys/server.key  # This file should be kept secret

# Diffie hellman parameters.
# Generate your own with:
#   openssl dhparam -out dh2048.pem 2048
dh /etc/openvpn/keys/dh2048.pem

# If enabled, this directive will configure
# all clients to redirect their default
# network gateway through the VPN, causing
# all IP traffic such as web browsing and
# and DNS lookups to go through the VPN
# (The OpenVPN server machine may need to NAT
# or bridge the TUN/TAP interface to the internet
# in order for this to work properly).
push "redirect-gateway def1"

# Certain Windows-specific network settings
# can be pushed to clients, such as DNS
# or WINS server addresses.  CAVEAT:
# http://openvpn.net/faq.html#dhcpcaveats
# The addresses below refer to the public
# DNS servers provided by opendns.com.
push "dhcp-option DNS 8.8.8.8"
push "dhcp-option DNS 8.8.4.4"

# For extra security beyond that provided
# by SSL/TLS, create an "HMAC firewall"
# to help block DoS attacks and UDP port flooding.
#
# Generate with:
#   openvpn --genkey --secret ta.key
#
# The server and each client must have
# a copy of this key.
# The second parameter should be '0'
# on the server and '1' on the clients.
tls-auth /etc/openvpn/keys/ta.key 0 # This file is secret

# For compression compatible with older clients use comp-lzo
# If you enable it here, you must also
# enable it in the client config file.
comp-lzo

# The maximum number of concurrently connected
# clients we want to allow.
max-clients 20

# It's a good idea to reduce the OpenVPN
# daemon's privileges after initialization.
#
# You can uncomment this out on
# non-Windows systems.
user nobody
group nobody

# Output a short status file showing
# current connections, truncated
# and rewritten every minute.
status /var/log/openvpn/openvpn-status.log

# By default, log messages will go to the syslog (or
# on Windows, if running as a service, they will go to
# the "\Program Files\OpenVPN\log" directory).
# Use log or log-append to override this default.
# "log" will truncate the log file on OpenVPN startup,
# while "log-append" will append to it.  Use one
# or the other (but not both).
log         /var/log/openvpn/openvpn.log
log-append  /var/log/openvpn/openvpn.log

########### END CHANGE INTO server.conf file ##########

# save and quit

```

> Create openvpn log folder `VPN Server`

```sh 
mkdir /var/log/openvpn/
```

> Generating Keys and Certificates `VPN Server`

```sh
# create certificates folder
mkdir -p /etc/openvpn/easy-rsa/keys
mkdir -p /etc/openvpn/keys

# copy the key and certificate generation scripts into the directory
cp -rf /usr/share/easy-rsa/2.0/* /etc/openvpn/easy-rsa

# edit the default values the script and chenge some parameters
vi /etc/openvpn/easy-rsa/vars

# change the values for the bellow parameter and keep default configuration for the rest

########### START CHANGE INTO vars file ##########

# These are the default values for fields
# which will be placed in the certificate.
# Don't leave any of these fields blank.
export KEY_COUNTRY="US"
export KEY_PROVINCE="CA"
export KEY_CITY="SanFrancisco"
export KEY_ORG="Fort-Funston"
export KEY_EMAIL="me@myhost.mydomain"
export KEY_OU="MyOrganizationalUnit"

# X509 Subject Field
export KEY_NAME="server"

# PKCS11 Smart Card
# export PKCS11_MODULE_PATH="/usr/lib/changeme.so"
# export PKCS11_PIN=1234

# If you'd like to sign all keys with the same Common Name, uncomment the KEY_CN export below
# You will also need to make sure your OpenVPN server config has the duplicate-cn option set
# export KEY_CN="openvpn.example.com"

########### END CHANGE INTO vars file ##########

# save and quit

# source file
cd /etc/openvpn/easy-rsa
source ./vars
./clean-all
```

> Generate certificates `VPN Server`

```sh
# generate certificate authority
./build-ca

# server key
./build-key-server server

# generate a Diffie-Hellman key exchange file
./build-dh

# generate client 1 key (the remote server)
./build-key server-client

# generate client 2 key (mac user)
./build-key mac-client

# copy generated keys into key folder
cd /etc/openvpn/easy-rsa/keys
cp keys/dh2048.pem keys/ca.crt keys/server.crt keys/server.key /etc/openvpn/keys/


```

> Copy openssl.cnf file `VPN Server`

```sh
cp /etc/openvpn/easy-rsa/openssl-1.0.0.cnf /etc/openvpn/openssl.cnf
```


> Configure authentication - TLS Authentication
Create an "HMAC firewall" to help block DoS attacks and UDP port flooding

```sh
# create DH parameters file
openssl dhparam -out dh2048.pem 2048

# create key
mkdir -p /etc/openvpn/tls && openvpn --genkey --secret /etc/openvpn/tls/ta.key

# copy key into key folder
cp /etc/openvpn/tls/ta.key /etc/openvpn/keys/

```


> Routing `VPN Server`

```sh
# mask firefall and start iptables
systemctl mask firewalld
systemctl enable iptables
systemctl stop firewalld
systemctl start iptables
iptables --flush

# add a rule to iptables to forward our routing to our OpenVPN subnet, and save this rule
iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE
iptables-save > /etc/sysconfig/iptables

# enable IP forwarding in sysctl
vi /etc/sysctl.conf

# add this parameter
net.ipv4.ip_forward = 1

# restart network service
systemctl restart network.service

```

> Starting OpenVPN `VPN Server`

``` sh
# add it to systemctl
systemctl -f enable openvpn@server.service

# Start OpenVPN
systemctl start openvpn@server.service

# check openvpn status
systemctl status openvpn@server

```

> Configuring a Client 1 `Server (Linix server accessible only via VPN tunnel)`

```sh
# log as root
sudo su - root
```

Copy these files from `VPN Server` to the client server:
- ca.crt
- server-client.crt
- server-client.key

You can use scp

> Intall pre-requisites `Server (Linix server accessible only via VPN tunnel)`

```sh
# install the Extra Packages
yum install -y epel-release

# install openvpn
yum install -y openvpn 

```

> Web application for test `Server (Linix server accessible only via VPN tunnel)`

```sh
# install http server
yum -y install httpd

# start http service
systemctl enable httpd
systemctl start httpd
systemctl status httpd

# configure firewall
firewall-cmd --permanent --add-port=80/tcp
firewall-cmd --reload

# create web page for test
mkdir -p /var/www/html/
echo "Hello world" > /var/www/html/index.html

```

> Configure openvpn `Server (Linix server accessible only via VPN tunnel)`

```sh
# create config folder
mkdir /etc/openvpn-config-files

# copy certificates files download from server into the folder
cp ca.crt /etc/openvpn-config-files/
cp server-client.crt /etc/openvpn-config-files/
cp server-client.key /etc/openvpn-config-files/

# create openvpn configuration folder
vi /etc/openvpn-config-files/client.ovpn

# add the following content
######### START FILE CONTENT #########
client
proto tcp
dev tun
remote 147.135.194.105 1196
resolv-retry infinite
nobind
user nobody
group nobody
persist-key
persist-tun
ca /etc/openvpn-config/ca.crt
cert /etc/openvpn-config/server-client.crt
key /etc/openvpn-config/server-client.key
tls-auth /etc/openvpn-config/ta.key
comp-lzo
verb 3

######### END FILE CONTENT #########

# save and quit

# add execution right
chmod +x /etc/openvpn-config-files/client.ovpn
```

> Start OpenVPN `Server (Linix server accessible only via VPN tunnel)`

```sh
openvpn --config /etc/openvpn-config-files/client.ovpn
```

> Check you VPN tunneling `Server (Linix server accessible only via VPN tunnel)`

```sh
# check config
ifconfig

```

You should see something like this

![MetaStore remote database](https://github.com/gamboabdoulraoufou/Internet-LAN-Over-VPN-Using-OpenVPN/blob/master/img/vpn_server-client-ip.png)


> Configure openvpn for Mac or windows client

Copy these files from `VPN Server` to the client server:
- ca.crt
- mac-client.crt
- mac-client.key

create client.ovpn file with the following content

######### START FILE CONTENT #########

client
proto tcp
dev tun
remote 147.135.194.105 1196
resolv-retry infinite
nobind
user nobody
group nobody
persist-key
persist-tun
ca /Users/agambo/OVH/vpn_config/client/ca.crt
cert /Users/agambo/OVH/vpn_config/client/mac-client.crt
key /Users/agambo/OVH/vpn_config/client/mac-client.key
tls-auth /Users/agambo/OVH/vpn_config/client/ta.key
comp-lzo
verb 3

######### END FILE CONTENT #########


> MAC client configuration

On Mac OS X, the open source application Tunnelblick provides an interface similar to the OpenVPN GUI on Windows, and comes with OpenVPN and the required TUN/TAP drivers. As with Windows, the only step required is to place your .ovpn configuration file into the ~/Library/Application
Support/Tunnelblick/Configurations directory. Or, you can double-click on your .ovpn file.


> Windows client configuration

On Windows, you will need the official OpenVPN Community Edition binaries which come with a GUI. Then, place your .ovpn configuration file into the proper directory, C:\Program Files\OpenVPN\config, and click Connect in the GUI. OpenVPN GUI on Windows must be executed with administrative privileges


> Reach our web aplication try vpn tunnel 
We are using local IP adress of our Web server from windows or mac client

![MetaStore remote database](https://github.com/gamboabdoulraoufou/Internet-LAN-Over-VPN-Using-OpenVPN/blob/master/img/web-app.png)


