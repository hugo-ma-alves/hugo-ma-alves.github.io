---
layout: post
title: Install a private OpenVPN server in your home network using a Raspberry Pi 1B
categories: [raspberry]
tags: [raspberry, vpn, pihole, openvpn]
---

In this tutorial you will learn how to install the OpenVpn server on your home network using only a Raspberry Pi.
With OpenVPN you will have a fully functional VPN Server in your home.

With this setup you can protect your internet traffic when you are connected to public or non trusted WiFi networks. For example, if you go on a trip and you need to connect to the hotel public WiFi you can encrypt all your network traffic from the hotel to your home. After the traffic reaches your home it will exit to the destination through your ISP and come back to you encrypted again.

Another advantage is that you can securely access your home network while you are away. For example, you can watch your movies stored in a local NAS, control smart lights etc... All these without exposing these services ports over the Internet!

**Objectives for this tutorial:**

1. Install OpenVpn server on raspberry PI.

2. Use PiHole as your DNS server.

3. Make OpenVpn accessible from outside the home network, i.e., from Internet.

4. Connect to VPN using the OpenVPN client from a IOS device and from Windows 10.

## Requirements

1. **Raspberry Pi**  
In this tutorial I used a Raspberry Pi 1B, it should be enough to run OpenVpn for a single user. But don't expect very high throughputs. With my home connection 200/100 Mb/s I could only get 8 Mb/s while connected to the VPN.

2. **A static IP or a DDNS**  
If your ISP doesn't provide you a static IP address you will need a DDNS.
If you want a free solution you can use [Duck DNS](https://www.duckdns.org). Check the other tutorial on this site about how you can configure DuckDNS on a Raspberry.

3. **Open Vpn client**  
You must install OpenVpn client on your client device (IOS, Android, windows etc...)


## Install OpenVPN server

To make our task easier we will use [PiVPN](http://www.pivpn.io). PiVPN is a script that provides an easy way to install and configure a OpenVPN server on Raspberry Pi.

Lets connect to Raspberry with SSH and begin the installation with the following commands:

    curl -L https://install.pivpn.io  > pivpn_install.sh
    chmod +x pivpn_install.sh
    ./pivpn_install.sh

The first thing that PiVPN does is to update the system. After that, it downloads OpenVPN Server and launches a wizard to configure OpenVPN server.  
Now you just need to follow the installation wizard to
complete the installation.

##### The following figures explain each step of the wizard:
---

1. Informs that the PiVPN installer will install OpenVPN server
![pivpn step 1](/assets/posts/pivpn-raspberry/step_1.png)

2. To work correctly OpenVPN server needs a static IP address. You have to ensure that the Raspberry Pi is configured with a static IP address. Or that your DHCP server always announces the same IP address for the raspberry.
![pivpn step 2](/assets/posts/pivpn-raspberry/step_2.png)

3. Check if the IP address and gateway are correct. If not you have to abort the installation and configure the raspberry with the correct IP address.
![pivpn step 3](/assets/posts/pivpn-raspberry/step_3.png)

4. Anoter warning about the importance of having a static IP address. If your network has a DHCP server, ensure that it always give the same IP address to the raspberry.
![pivpn step 4](/assets/posts/pivpn-raspberry/step_4.png)

5. Choose a local user to store the openvpn configurations.
![pivpn step 5](/assets/posts/pivpn-raspberry/step_5.png)

6. In this guide I will use the default user "pi".
![pivpn step 6](/assets/posts/pivpn-raspberry/step_6.png)

7. This is an important warning about keeping your raspberry updated. Since you will have your raspberry exposed to the Internet is is **very** important that you keep it updated.  You should enable unattended Upgrades. With this setting enabled the system will periodic check for security updates and apply them.  
This feature does not eliminate the need of manual updating the raspberry regularly. There are some updates that require a restart and because of that cannot be applied automatically.
![pivpn step 7](/assets/posts/pivpn-raspberry/step_7.png)

8. As explained in the previous step, enable unattended Upgrades.
![pivpn step 8](/assets/posts/pivpn-raspberry/step_8.png)

9. OpenVPN can either run over UDP or TCP, normally you should use UDP.
You should only choose TCP for specific cases. For example, if the networks where you will be using the VPN block UDP traffic, in that case you can try to run OpenVPN over TCP.
![pivpn step 9](/assets/posts/pivpn-raspberry/step_9.png)

10. Choose the port where OpenVPN will listen. I used the default port 1194.
![pivpn step 10](/assets/posts/pivpn-raspberry/step_10.png)

11. Confirm the settings.
![pivpn step 11](/assets/posts/pivpn-raspberry/step_11.png)

12. //todo
![pivpn step 12](/assets/posts/pivpn-raspberry/step_12.png)

13. Here you will select the size for the server RSA key, 2048 is a good balance between speed and security.
![pivpn step 13](/assets/posts/pivpn-raspberry/step_13.png)

14. This step will generate the server RSA key pair and the [Diffie-Hellman](https://en.wikipedia.org/wiki/Diffie–Hellman_key_exchange) parameters. This step could take a lot of time... so be patient.
To avoid aborting the process due to PC sleep you should use a terminal multiplexer like [tmux](https://github.com/tmux/tmux/wiki).
![pivpn step 14](/assets/posts/pivpn-raspberry/step_14.png)
![pivpn step 14 result](/assets/posts/pivpn-raspberry/step_14_DH.png)


15. Here you will have to choose how you will connect to the OpenVPN server. You can either use your public IP address or a domain Name. I do not recommend using the public IP address because it will probably change when your ISP renews the DHCP lease.
You should configure a DDNS, you can configure one at [DuckDNS](https://www.duckdns.org), check the DuckDNS tutorial on this blog.
![pivpn step 15](/assets/posts/pivpn-raspberry/step_15.png)

16. I've previously configured this domain at duckdns.org. This domain name will resolve to your public IP address.
![pivpn step 16](/assets/posts/pivpn-raspberry/step_16.png)

17. Just confirm the domain you introduced.
![pivpn step 17](/assets/posts/pivpn-raspberry/step_17.png)


18. In this step you can configure the DNS server that will be used by your devices. We have two options here, either we use a public DNS server, like Google or Cloudflare, or we use a custom one.  
We will use the last option. Our DNS server will be a PiHole running on the local network (in my case it is also on the raspberry). For information about how to install PiHole check [this blog post]({% post_url 2019-03-22-pihole-vodafone %}).
![pivpn step 18](/assets/posts/pivpn-raspberry/step_18_dns_custom_1.png)

19. Since we have chosen to use PiHole as the DNS server we need to fill its IP address in this step. This is the local address, it doesn't need to be a public address.
![pivpn step 18 2](/assets/posts/pivpn-raspberry/step_18_dns_custom_2.png)


20. Make sure that the DNS server 1 field is the private IP address of the machine running the PiHole.
![pivpn step 19](/assets/posts/pivpn-raspberry/step_19.png)

21. Last step! Reboot the raspberry!
![pivpn step 20](/assets/posts/pivpn-raspberry/step_21.png)


## Generate a client certificate

The OpenVPN server is installed and ready to accept connections. But before that, we need to generate the client certificates for each client that will use the VPN.

The client certificate is an RSA keypair that will be used to authenticate and secure the communication between your device and OpenVPN server.
You will need the client certificate in order to connect to the OpenVPN server. This certificate act as your username and password, but more secure.

The client generations is very easy, you just need to run the following command: `pivpn add`

![pivpn add client](/assets/posts/pivpn-raspberry/pivpn_add.png)

To create a client you just need to follow the instructions. You have to give a username and a password to encrypt the private key.

This process will generate the `/home/pi/ovpns/my-vpn-client.ovpn` file:

    client
    dev tun
    proto udp
    remote demo-pivpn-hugo-ma-alves.duckdns.org 1194
    resolv-retry infinite
    nobind
    persist-key
    persist-tun
    key-direction 1
    remote-cert-tls server
    tls-version-min 1.2
    verify-x509-name server_66iVDlc1xRPTTOSa name
    cipher AES-256-CBC
    auth SHA256
    auth-nocache
    verb 3
    <ca>
    -----BEGIN CERTIFICATE-----
    {A private key}
    -----END CERTIFICATE-----
    </ca>
    <cert>
    -----BEGIN CERTIFICATE-----
    {The certificate}
    -----END CERTIFICATE-----
    </cert>
    <key>
    -----BEGIN ENCRYPTED PRIVATE KEY-----
    {A private key}
    -----END ENCRYPTED PRIVATE KEY-----
    </key>
    <tls-auth>
    #
    # 2048 bit OpenVPN static key
    #
    -----BEGIN OpenVPN Static key V1-----
    {server key}
    -----END OpenVPN Static key V1-----
    </tls-auth>

You can use this file with OpenVPN client app to connect to your VPN.
Now you must copy this file to the device from where you will connect to the VPN, for example your laptop or smartphone. You can copy the file with FTP or SFTP. Or, since this file is just text, you can also copy the content of the file and paste it in a new file(my-vpn-client.ovpn) on your device.

## Connect to the VPN

To connect to the VPN you will need a VPN client, there are multiples alternatives.
For Windows, IOS and Android you can use the [Open VPN client](https://openvpn.net/community-downloads/).
For MacOs you can use [Tunnelblick](https://tunnelblick.net/). Despite the one you choose the only thing you will need is the .ovpn file that you generated in the previous step.


## Results

As said before you will not get the full throughput that you normal get in your home. Your speed will be limited by multiple factors:
- You home network Upload speed
- Raspberry Pi processing capacity
- Visited network speed

In this case my home network is 200/100 Mb/s and I tested the VPN from a WiFi network that also provides around 200/100 Mb/s.

With this setup I could get around 10/10 Mb/s running from the Raspberry Pi 1B, this is one of the oldest Raspberry's, with a 700 MHz cpu and 512 MB of ram.

Running the same test with a Raspberry Pi 3B+ I got no noticeable improvements on the speedtest
![pivpn speedtest ](/assets/posts/pivpn-raspberry/speedtest_vpn_3b.png).

The 10/10 Mb/s provided by Raspberry Pi 1B is enough to my daily browsing outside home. It was also enough to stream some series from my NAS transcoded to a Iphone screen.
And the biggest advantage is the [power consumption](https://www.jeffgeerling.com/blogs/jeff-geerling/raspberry-pi-zero-power) of the Raspberry Pi 1B, consuming as low as 0.4W.
