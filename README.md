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

> Create openvpn log folder

```sh 
mkdir /var/log/openvpn/
```

> Generating Keys and Certificates































