# Network-Wide-DNS-Filtering-Containerized-Services
### Designed and deployed a containerized DNS filtering service using Docker and AdGuard Home to provide network-wide ad and tracker blocking.

<h1>Introduction</h1>
<b>Docker</b> is a set of products that uses operating-system-level virtualization to deliver software in packages called containers. Docker is not a type 2 hypervisor, though they share some similarities. The key difference is that Docker is a container engine that uses containerization technology for software, while hypervisors create full virtual machines.  
<b>AdGuard Home</b> is a network-wide software for blocking ads and tracking. After you set it up, it'll cover ALL your home devices, and you don't need any client-side software for that. AdGuard also acts as a DNS resolver.
<hr>
<b>Docker will be used to virtualize the AdGuard Home software on a Ubuntu Virtual Machine.</b>
<hr>

<h1>Prerequisites</h1>
This exercise will be done only using virtual machines. <br>
<b>pfSense</b> - Firewall and Gateway. <br>
<b>Ubuntu/Debian</b> - This is where AdGuard will be hosted, I will be using ubuntu. <br>
<b>Docker</b> <br>
<b>AdGuard Home Container</b><br>
<b>Kali Linux</b> - To confirm the network wide coverage. A different distro can be used as you wish, this was just my pick.<br>

<h1>Docker Installation</h1>
Recommended storage should be at least 30GB. 
First step is to start as root user with <pre>sudo su</pre>

Before installation it is recommended to uninstall any unofficial conflicting docker packages that may conflict with the official packages.
<pre>
  sudo apt remove $(dpkg --get-selections docker.io docker-compose docker-compose-v2 docker-doc podman-docker containerd runc | cut -f1) 
</pre>

Now to install the repo using apt;

```bash
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update
```

Installing the latest docker version;
<pre>sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin </pre>
Check status of docker with;
<pre>sudo systemctl status docker</pre>

Note: Before setting up AdGuard, you’d have to set a static IP address as your machine would become a server and it would be terrible if the IP address was to change.

<h1>AdGuard Container Setup</h1>
- Create a directory for the software;

```bash
mkdir adguard
cd adguard
```
- Create a <b><i>compose.yml</i></b> file in the same repository;
<pre>touch compose.yml</pre>
- Then edit with; 
<pre>nano compose.yml</pre>
- Paste this into the file;
```bash
name: adguardhome

services:
  adguardhome:
    image: adguard/adguardhome
    ports:
      - 53:53/tcp # plain dns over tcp
      - 53:53/udp # plain dns over udp
      - 80:80/tcp # http web interface
      - 3000:3000/tcp # initial setup web interface
    volumes:
      - config:/opt/adguardhome/conf # app configuration
      - work:/opt/adguardhome/work # app working directory

volumes:
  config:
    driver: local
  work:
    driver: local
```
This Docker Compose file runs AdGuard Home as a DNS server and web interface with persistent storage and exposed ports for network-wide ad blocking.

- Save and Exit.

<p>
  In the same directory, start AdGuard with;
<pre>docker compose up -d</pre>
This command downloads the adguard image from docker hub and starts the container in the background.

<img src="screenshots/Screenshot 2025-12-26 011245.png">

</p>


You may encounter an error that says “failed to bind host port 0.0.0.0:53/tcp: address already in use”. If you do, head here, to resolve the error.
https://medium.com/@nifise/docker-setup-debugging-failed-to-bind-host-port-0-0-0-0-53-tcp-address-already-in-use-d0c1a5cd31db

- You should be able to access adguard using;
```
http://your_host_ip:3000
```
<img src="screenshots/Screenshot 2026-01-07 135633.png">
- Restart the container with this, if the site does not load.
<pre>docker compose up -d</pre>

<h1>AdGuard Configuration</h1>
<img src="screenshots/Screenshot 2025-12-26 150858.png">
<img src="screenshots/Screenshot 2025-12-26 152540.png">


When doing this in the future in my personal LAN, I will be using a raspberry Pi to host AdGuard.
<h1>Refrences</h1>
- https://medium.com/@codyrwaits/installing-adguard-home-in-my-home-lab-648393f6064f 
- https://frasermclean.com/posts/secure-your-entire-home-network-with-adguard-home 













