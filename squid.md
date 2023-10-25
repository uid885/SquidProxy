#Author: Christo Deale <br>
#Date: 2023-10-25
# Setup Squid Proxy on RHEL 9 & configure with basic username/password authentication

## Step 1: Install Squid:
```
sudo dnf install squid httpd-tools
```

## Step 2:  Create a Password File:
Use the ***htpasswd*** tool to create a password file that will store usernames and encrypted 
passwords for Squid. You can do this for each user you want to grant access to the proxy 
server. 
```
sudo htpasswd -c /etc/squid/passwd username  
```

## Step 3: Configure Squid:
Open the Squid configuration file for editing:
```
sudo vim /etc/squid/squid.conf
```
```
#
# Recommended minimum configuration:
# Anonymise Traffic
forwarded_for off
request_header_access Allow allow all
request_header_access Authorization allow all
request_header_access WWW-Authenticate allow all
request_header_access Proxy-Authorization allow all
request_header_access Proxy-Authenticate allow all
request_header_access Cache-Control allow all
request_header_access Content-Encoding allow all
request_header_access Content-Length allow all
request_header_access Content-Type allow all
request_header_access Date allow all
request_header_access Expires allow all
request_header_access Host allow all
request_header_access If-Modified-Since allow all
request_header_access Last-Modified allow all
request_header_access Location allow all
request_header_access Pragma allow all
request_header_access Accept allow all
request_header_access Accept-Charset allow all
request_header_access Accept-Encoding allow all
request_header_access Accept-Language allow all
request_header_access Content-Language allow all
request_header_access Mime-Version allow all
request_header_access Retry-After allow all
request_header_access Title allow all
request_header_access Connection allow all
request_header_access Proxy-Connection allow all
request_header_access User-Agent allow all
request_header_access Cookie allow all
request_header_access All deny all

# Example rule allowing access from your local networks.
# Adapt to list your (internal) IP networks from where browsing
# should be allowed
acl localnet src 0.0.0.1-0.255.255.255  # RFC 1122 "this" network (LAN)
acl localnet src 10.0.0.0/8             # RFC 1918 local private network (LAN)
acl localnet src 100.64.0.0/10          # RFC 6598 shared address space (CGN)
acl localnet src 169.254.0.0/16         # RFC 3927 link-local (directly plugged) machines
acl localnet src 172.16.0.0/12          # RFC 1918 local private network (LAN)
acl localnet src 192.168.0.0/16         # RFC 1918 local private network (LAN)
acl localnet src fc00::/7               # RFC 4193 local private network range
acl localnet src fe80::/10              # RFC 4291 link-local (directly plugged) machines

acl SSL_ports port 443
acl Safe_ports port 80          # http
acl Safe_ports port 21          # ftp
acl Safe_ports port 443         # https
acl Safe_ports port 70          # gopher
acl Safe_ports port 210         # wais
acl Safe_ports port 1025-65535  # unregistered ports
acl Safe_ports port 280         # http-mgmt
acl Safe_ports port 488         # gss-http
acl Safe_ports port 591         # filemaker
acl Safe_ports port 777         # multiling http

# Block list of domains
acl block dstdomain '/etc/squid/block.txt'
http_access deny block

#
# Recommended minimum Access Permission configuration:
#
# Deny requests to certain unsafe ports
http_access deny !Safe_ports

# Deny CONNECT to other than secure SSL ports
http_access deny CONNECT !SSL_ports

# Only allow cachemgr access from localhost
http_access allow localhost manager
http_access deny manager

# We strongly recommend the following be uncommented to protect innocent
# web applications running on the proxy server who think the only
# one who can access services on "localhost" is a local user
#http_access deny to_localhost

#
# INSERT YOUR OWN RULE(S) HERE TO ALLOW ACCESS FROM YOUR CLIENTS
auth_param basic program /usr/lib64/squid/basic_ncsa_auth /etc/squid/passwd
acl authenticated proxy_auth REQUIRED
http_access allow authenticated

# Example rule allowing access from your local networks.
# Adapt localnet in the ACL section to list your (internal) IP networks
# from where browsing should be allowed
http_access allow localnet
http_access allow localhost

# And finally deny all other access to this proxy
http_access deny all

# Squid normally listens to port 3128
http_port 3128

# Uncomment and adjust the following to add a disk cache directory.
#cache_dir ufs /var/spool/squid 100 16 256

# Leave coredumps in the first cache dir
coredump_dir /var/spool/squid

#
# Add any of your own refresh_pattern entries above these.
#
refresh_pattern ^ftp:           1440    20%     10080
refresh_pattern ^gopher:        1440    0%      1440
refresh_pattern -i (/cgi-bin/|\?) 0     0%      0
refresh_pattern .               0       20%     4320
```

## Step 4: Create blocklist
```
vim /etc/squid/block.txt
```
.facebook.com <br>
.youtube.com   <br>
.twitter.com      <br>

## Step 5: Restart Squid:
After making these changes, restart Squid to apply the settings:
```
sudo systemctl restart squid
```
## Step 6: Check the Current Firewall Status:
To check if the firewall is running, use the following command:
```
sudo systemctl status firewalld
```
## Step 7: Add a Firewall Rule:
To allow traffic on port 3128 (Squid's default port), you can add a rule using the firewall-cmd command:
```
sudo firewall-cmd --zone=public --add-port=3128/tcp --permanent
```
This command adds a rule to the "public" zone that allows incoming TCP traffic on port 3128. Replace 3128 with the actual port if you have configured Squid to listen on a different port.

## Step 8: Reload the Firewall:
After adding the rule, reload the firewall to apply the changes:
```
sudo firewall-cmd --reload
```
## Step 9: Verify the Rule:
You can check the rules in the "public" zone to make sure the port is now allowed:
```
sudo firewall-cmd --list-ports --zone=public
```
