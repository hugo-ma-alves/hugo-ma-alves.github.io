---
layout: post
title: Install a private OpenVPN server in your home network using a Raspberry Pi 1B
categories: [raspberry]
tags: [raspberry, vpn, pihole, openvpn]
---

In this tutorial you will learn how to install the OpenVpn server on your home network.
With OpenVPN installed you will have a fully functional VPNServer in your home.

With this setup you can protect your internet traffic when you are connected to public or non trusted wifi networks. For example, if you go on a trip and you need to connect to the hotel public wifi you can encrypt all your network traffic from the hotel to your home. After the traffic reaches your home it will exit through your ISP and come back to you encrypted again.

**Goals:**

1. Install OpenVpn server on raspberry PI
2. Make OpenVpn accessible from outside the home network, i.e., from Internet
3. Connect to VPN using the OpenVPN client from a IOS device and from Windows 10.

### Requirements

1. **Raspberry Pi**  
In this tutorial I used a Raspberry Pi 1B, it should be enough to run OpenVpn for a signle user. But don't expect a high throughput. With my home connection 200/100 Mb/s I could only get 8 Mb/s.
2. **A static IP or a DDNS**  
If your ISP doesn't provide you a static IP address you will need a DDNS.
If you want a free solution you can use [Duck DNS](https://www.duckdns.org). Check the other tutorial on this site about how you can configure DuckDNS on Raspberry.
3. **Open Vpn client**  
You must install OpenVpn client on your client device (IOS, Android, windows etc...)


### Install OpenVPN server

To make our task easier we will use [PiVPN](http://www.pivpn.io). PiVPN is a script that provides an easy way to install and configure a OpenVPN server on Raspberry Pi.

We can start the installation with the following steps:

    curl -L https://install.pivpn.io  > pivpn_install.sh
    chmod +x pivpn_install.sh
    ./pivpn_install.sh

The first thing that PiVPN does is to update the system. After that it downloads OpenVPN Server from the repositories and launches a wizard to configure OpenVPN server.

##### The following figures explain each step of the wizard
---

1. Informs that the PiVPN installer will install OpenVPN server
![pivpn step 1](/assets/posts/pivpn-raspberry/step_1.png)

2. To work correctly OpenVPN server needs a static IP address. You have to ensure that the Raspberry Pi is configured with a static IP address. Or that your DHCP server always announces the same IP address for the raspberry.
![pivpn step 2](/assets/posts/pivpn-raspberry/step_2.png)

3. Check if the IP address and gateway are correct. If not you have to abort the installation and configure the raspberry with the correct IP address
![pivpn step 3](/assets/posts/pivpn-raspberry/step_3.png)

4. Anoter warning about the importance of having a static IP address. If your network has a DHCP server ensure that it always give the same IP address to the raspberry
![pivpn step 4](/assets/posts/pivpn-raspberry/step_4.png)

5. Choose a local user to store the openvpn configurations
![pivpn step 5](/assets/posts/pivpn-raspberry/step_5.png)

6. In this guide I will use the default user "pi"
![pivpn step 6](/assets/posts/pivpn-raspberry/step_6.png)

7. This is an important warning about keeping your raspberry updated. Since you will have your raspberry exposed to the Internet is is **very** important that you keep it updated.  You should enable unattended Upgrades, with this setting enabled the system will periodic check for security updates and apply them.  
This feature does not eliminate the need of manual updating the raspberry regularly. There are some updates that require a restart and because of that cannot be applied automatically.
![pivpn step 7](/assets/posts/pivpn-raspberry/step_7.png)

8. As explained in the previous step, enable unattended Upgrades
![pivpn step 8](/assets/posts/pivpn-raspberry/step_8.png)

9. OpenVPN protocol can either run over UDP or TCP. Normally you should use UDP.   
You should only choose TCP for specific cases. For example, if the networks where you will be using the VPN block UDP traffic you can try to run OpenVPN over TCP.
![pivpn step 9](/assets/posts/pivpn-raspberry/step_9.png)

10. Choose the port where OpenVPN will listed. Here we use the default port 1194.
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
![pivpn step 15](/assets/posts/pivpn-raspberry/step_14_DH.png)



### Generate a client

Now we have OpenVPN server installed and ready to accept connections. But before that we need to generate client certificates for each client that will use the VPN.

The client certificate is an RSA keypair that will be used to authenticate and secure the communication between your device and OpenVPN server.
You will need the client certificate in order to connect to the OpenVPN server.

The client generations is very easy, you just need to run the following command: `pivpn add`


### Speedtest

![pivpn step 15](/assets/posts/pivpn-raspberry/speedtest_home.png)

//TODO show home speedtest and vpn speedtest with 4g
