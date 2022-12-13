# KASM Workspace VPS with Cloudflare WAF, Two Factor, SSL Certificate and CrowdSec IPS

## What is KASM Workspace
The Workspaces platform provides enterprise-class orchestration, data loss prevention, and web streaming technology to enable the delivery of containerized workloads to your browser.
- Demo: https://www.kasmweb.com/#dialog-2

### Requirements: 
- Create Oracle Cloud Free Tier Account - https://jaredbach.io/creating-a-free-tier-oracle-cloud-account-7bce9ea230b8?gi=c562a2cf276
- CloudFlare Account and Domain Name - https://developers.cloudflare.com/registrar/get-started/transfer-domain-to-cloudflare/

1. Provision the following ARM64 Virtual Machine.

Note: Some containers are not compatible with ARM64 or don't have a build yet.
- Images: https://kasmweb.com/docs/latest/guide/custom_images.html
- Guide: https://cybertoffy.com/kasm-workspace-on-oracle-cloud/

```
Operating System: Ubuntu 22.04
Shape configuration
Shape: VM.Standard.A1.Flex
OCPU count: 4
Network bandwidth (Gbps): 4
Memory (GB): 24
Local disk: 100GB
```

2. Download SSH Keypair (and convert). SSH into the VM and run the following using key and IP.
https://tecadmin.net/convert-ppk-to-pem-using-command/, then run the following to setup KASM.
```
sudo apt-get update && sudo apt-get upgrade -y
sudo dd if=/dev/zero bs=1M count=2048 of=/mnt/2GiB.swap
sudo chmod 600 /mnt/2GiB.swap
sudo mkswap /mnt/2GiB.swap
sudo swapon /mnt/2GiB.swap
cat /proc/swaps
echo '/mnt/2GiB.swap swap swap defaults 0 0' | sudo tee -a /etc/fstab
cd /tmp
curl -O https://kasm-static-content.s3.amazonaws.com/kasm_release_1.12.0.d4fd8a.tar.gz
tar -xf kasm_release*.tar.gz
sudo bash kasm_release/install.sh
```

3. Copy the credentials into your password manager, login using IP:443. 

- Enable 2FA - https://kasmweb.com/docs/latest/guide/two_factor.html
- Create new administrator account, add to administrator, login and enable 2FA.
- Create new regular non-priv account, login and enable 2FA.
- Delete default administrator and user account.


4. Since we are using ARM64, Kali Linux will not be enabled by default. 
```
docker pull kasmweb/core-kali-rolling:develop-rolling
```
6. In the interface, proceed to Workspaces > Add
7. Follow the step 5 in this guide - https://www.blog.techraj156.com/post/installing-kasm-workspaces-and-setting-up-kali-linux-for-penetration-testing
```
Docker Image: kasmweb/core-kali-rolling:develop-rolling
Description: Kali Linux is an open-source, Debian-based Linux distribution geared towards various information security tasks, such as Penetration Testing, Security Research, Computer Forensics and Reverse Engineering.
Friendly Name: Kali Linux
Thumbnail URL: img/thumbnails/kali.png
Cores: 2
Memory: 3768 GB
GPU Count: 0
CPU Allocation Method: Inherit
Enabled: Yes
Docker Registry: https://index.docker.io/v1/
Volume Mappings: {}
Run Config: {"user":"root","cap_add":["NET_ADMIN"],"devices":["dev/net/tun","/dev/net/tun"],"privileged":true}
Exec Config: 
Categories: 
Security
Desktop
```
8. When running Kali container, run these commands inside the container to fix some depedancies. (or create a custom docker build).

```
sudo apt install iputils-* --y 
sudo apt-get install apt-file --y 
sudo apt install apt-utils --y
sudo apt dist-upgrade
sudo apt update
ping google.ca
```

8. Buy domain, add to Cloudflare for WAF, and generate SSL Certificate (recommended). -
9. Generate Cloudflare Origin CA Certificate - save it as kasm_nginx.pem and kasm_nginx.key. - https://developers.cloudflare.com/ssl/origin-configuration/origin-ca/
10. Copy these files to a tmp folder in your server, and follow this guide. - https://kasmweb.com/docs/latest/how_to/certificates.html. If you are using a reverse proxy use this link instead https://kasmweb.com/docs/latest/how_to/reverse_proxy.html

Stop the Kasm Services
```
sudo /opt/kasm/bin/stop
```
Replace kasm_nginx.crt and kasm_nginx.key files
```
sudo cp kasm_nginx.pem /opt/kasm/current/certs/kasm_nginx.crt
sudo cp kasm_nginx.key /opt/kasm/current/certs/kasm_nginx.key
```
Start the Kasm Services
```
sudo /opt/kasm/bin/start
```
11. Wait for 10 minutes, clear history on your browser and try accessing your https://domain.com. It should now have a full SSL. 
12. As noted, its best to use a reverse Proxy (Nginx Proxy Manager) to apply Authenticated Origin Pull, thus allowing CF only traffic to your VPS. Direct IP access is prohibited from non CF, and prevents IP leaks. - https://developers.cloudflare.com/ssl/origin-configuration/authenticated-origin-pull/
13. Apply CloudFlare WAF Security rules to block specific locations GeoIP, Known Bots, High Risk ASN or IPs.

## Hardening
1. Install Fail2Ban
- Source: https://webdock.io/en/docs/how-guides/security-guides/how-configure-fail2ban-common-services
```
sudo apt install fail2ban -y
sudo systemctl status fail2ban
sudo nano /etc/fail2ban/jail.d/sshd.conf
```

```
[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 120
ignoreip = whitelist-IP
````
Restart SSH Service
```
sudo systemctl restart fail2ban
```
How to Check the Status of Jail
```
sudo fail2ban-client status sshd
```

2. Install CrowdSec and CrowdSec Firewall Bouncer (iptables)
- Related Source: https://www.smarthomebeginner.com/crowdsec-docker-compose-1-fw-bouncer/
```
curl -s https://packagecloud.io/install/repositories/crowdsec/crowdsec/script.deb.sh | sudo bash
sudo apt-get update
sudo apt-get install crowdsec
sudo apt install crowdsec-firewall-bouncer-iptables
```

3. Enroll bouncer to CrowdSec CLI Web Console
- Register Account: https://crowdsec.net
- Click add instance: https://app.crowdsec.net/instances
- Copy the ```sudo cscli console enroll random-string``` to your SSH session

4. Update and restart the process
```
sudo cscli hub upgrade
sudo cscli hub update
sudo systemctl reload crowdsec
```

5. Give the web console 5-30 minutes for all the changes to appear. Bouncer appears to be 0 for a bit.
