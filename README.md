We will be using ARM64 VM in this build.

Requirements: Oracle Cloud Free Tier Account - https://jaredbach.io/creating-a-free-tier-oracle-cloud-account-7bce9ea230b8?gi=c562a2cf276

1. Provision the following VM
https://cybertoffy.com/kasm-workspace-on-oracle-cloud/
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
https://tecadmin.net/convert-ppk-to-pem-using-command/
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

3. Copy the credentials into your password manager, login using IP:443
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

8. Buy domain, add to Cloudflare for WAF, and generate SSL Certificate (recommended).
