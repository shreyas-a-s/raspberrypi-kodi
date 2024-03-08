## Prerequisites

* A micro SD card. Minimum of 4GB (if you plan to install Raspberry Pi OS Lite) or 8GB (if you plan to install Raspberry Pi OS). Preferably above class 4 but any class will do the job.
* An micro SD card reader. To connect it to PC.
* Raspberry Pi OS. Download from [here](https://awesomeopensource.com/project/elangosundar/awesome-README-templates). I used Raspberry Pi OS Lite (Legacy) with Debian Buster (10).
* [Rufus](https://rufus.ie/en/). To install Pi OS to SD card from a windows PC. Other tools like etcher or built-in tools on some linux distros can also be used to flash the OS to SD card. You can even try the command-line option on linux if you feel to.
* TP-Link TL-WN725N. To create an Access Point on Raspberry Pi so that Kodi can be controlled via phone. If you have a router at home and can wire an ethernet connection to Pi, then wifi adapter and access point creation are not needed.
* A microUSB cable and adapter. To power Pi.
* Appropriate cable to connect Pi to a display (Monitor or TV).
* A USB Mouse can come in handy when you might need to enable `Control via HTTP` on Kodi.
* A keyboard (or a smartphone with wifi connectivity to ssh into Pi) to configure things and to control Kodi after installation. Here I am using smartphone as a way to learn about ssh and stuff but using keyboard is simpler.
* An SSH client (eg: [JuiceSSH](https://play.google.com/store/apps/details?id=com.sonelli.juicessh)) and [Kore](https://f-droid.org/en/packages/org.xbmc.kore/) app installed on phone.
* A router to connect to Pi via cable (ethernet or USB).
* Realtek 8188 Driver. Download from [here](https://github.com/aircrack-ng/rtl8188eus). (I used v5.3.9 which was the latest.)
* A pendrive to store the 8188 driver before copying to Pi.

## Procedure

1. Flash OS to SD card.
2. After completion of flash, go to the sd card boot folder (that is the SD card itself) and create an empty text file named ssh. Remove it’s .txt extension.
3. Eject the SD card.
4. Take out the SD card and insert it into SD card slot of Pi.
5. Connect Pi to a display.
6. Connect Pi to router via wired connection and phone to router via wi-fi.
7. Plug TP-Link TL-WN725N to Pi.
8. Connect the microUSB cable and adapter and power it ON.
9. Pi will boot up. Resize the partition. Then reboot automatically.
10. Wait for it to boot into CLI.
11. Open SSH client on phone and connect to Pi using `pi` as username, `raspberry` as password and the Pi’s `IP ADDRESS`.
12. First of all use this command to enter RaspberryPi configuration tool:
```
sudo raspi-config
```
13. Change these:
```
Overclocking to Turbo
GPU Memory to 160MB
Locale to en_IN UTF-8
Language to English
Time Zone to Asia>Kolkata.
```
14. Finish. Say Yes to Reboot.
15. Enter Raspbian Update Mirror configuration via:
```
sudo nano /etc/apt/sources.list
```
Change the URL after deb with an appropriate mirror from
[here](https://www.raspbian.org/RaspbianMirrors). (Don’t worry the site is
safe. You’ll know what I mean.)
I've used [Netherlands mirror](http://mirror.nl.leaseweb.net/raspbian/raspbian).

16. Ctrl + X. Then Y. Then Enter. This saves the sources.list file.
17. Update the programs on Pi:
```
sudo apt-get update && sudo apt-get upgrade
```
18. Reboot Pi:
```
sudo reboot
```
19. After rebooting, again use SSH app to connect to Pi.
20. Install kernel headers and an app called bc which are requirements for compiling a driver required for the Realtek 8188 wifi modem in TP-Link TL-WN725N. This is because by default the correct driver is not available in the Pi OS. Use this command:
```
sudo apt-get install raspberrypi-kernel-headers bc
```
21. Unzip the kernel files from github to a folder named rtl8188eus in
pendrive.
22. Connect pendrive to Pi.
23. Create a folder named usb in media directory of Pi:
```
sudo mkdir /media/usb
```
24. Mount pendrive to this folder:
```
sudo mount /dev/sda1 /media/usb
```
(We are assuming pendrive shows us as sda in lsblk.)

25. Create a folder named rtl8188eus in home directory of Pi:
```
sudo mkdir /home/pi/rtl8188eus
```
26. Copy contents of folder usb to this folder:
```
cp –a /media/usb/rtl8188eus/. /home/pi/rtl8188eus
```
(Not sure but you might need to use sudo.)

27. Umount pendrive:
```
sudo umount /media/usb
```
28. Disconnect pendrive from Pi.
29. Change creation time of driver files in rtl8188eus folder:
```
find /home/pi/rtl8188eus –type f –exec touch {} +
```
30. Go to that folder:
```
cd /home/pi/rtl8188eus
```
31. Do a cleanup of make files:
```
make clean
```
32. Blacklist another driver in order to use this new one:
```
echo 'blacklist r8188eu'|sudo tee -a '/etc/modprobe.d/realtek.conf'
```
33. Reboot Pi. After rebooting, again use SSH app to connect to Pi.
34. Go to driver folder:
```
cd /home/pi/rtl8188eus
```
35. Compile and install new drivers:
```
sudo make && sudo make install
```
It’s gonna take hell of a lot time, hence grab a coffee and chill until itss completed.

36. Reboot Pi. After rebooting, again use SSH app to connect to Pi.
37. Now we are going to set up out Raspberry Pi as wireless access point.
38. To create an access point, we’ll need DNSMasq, HostAPD and Netfilter-Persistent. Install all the required software in one go with this command:
```
sudo apt-get install dnsmasq hostapd netfilter-persistent
```
39. Since the configuration files are not ready yet, we need to stop the new software from running:
```
sudo systemctl stop dnsmasq
sudo systemctl stop hostapd
```
40. To configure the static IP address, edit the dhcpcd configuration file with:
```
sudo nano /etc/dhcpcd.conf
```
Go to the end of the file and add the following:
```
interface wlan0
 static ip_address=192.168.4.1/24
 nohook wpa_supplicant
```
41. Now restart the dhcpcd daemon and set up the new wlan0 configuration:
```
sudo service dhcpcd restart
```
42. The DHCP service is provided by dnsmasq. Let’s backup the old configuration file and then create a new one:
```
sudo mv /etc/dnsmasq.conf /etc/dnsmasq.conf.orig
sudo nano /etc/dnsmasq.conf
```
Type the following information into the dnsmasq configuration fileand save it:
```
interface=wlan0
dhcp-range=192.168.4.2,192.168.4.20,255.255.255.0,24h
```
This will provide IP-addresses between 192.168.4.2 and 192.168.4.20 with a lease time of 24 hours.

43. Now start dnsmasq to use the updated configuration:
```
sudo systemctl start dnsmasq
```
44. Now it is time to configure the access point software:
```
sudo nano /etc/hostapd/hostapd.conf
```
Add the below information to the configuration file:
```
country_code=DE
interface=wlan0
ssid=YOURSSID
channel=9
auth_algs=1
wpa=2
wpa_passphrase=YOURPWD
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP CCMP
rsn_pairwise=CCMP
```
Make sure to change the ssid and wpa_passphrase.
45. We now need to tell the system where to find this configuration file. Open the hostapd file:
```
sudo nano /etc/default/hostapd
```
Find the line with #DAEMON_CONF, and replace it with this:
```
DAEMON_CONF="/etc/hostapd/hostapd.conf"
```
46. Run the following commands to enable and start hostapd:
```
sudo systemctl unmask hostapd
sudo systemctl enable hostapd
sudo systemctl start hostapd
```
47. You may want devices connected to your wireless access point to access the main network and from there the internet. To do so, we need to set up routing and IP masquerading on the Raspberry Pi. We do this by editing the sysctl.conf file:
```
sudo nano /etc/sysctl.conf
```
And uncomment the following line:
```
net.ipv4.ip_forward=1
```
48. Next we need a “masquerade” firewall rule such that the IPaddresses of the wireless clients connected to the Raspberry Pi can be substituted by their own IP address on the local area network. To do so enter:
```
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```
49. Now to save these firewall rules and automatically use them upon boot, we will use the netfilter-persistent service:
```
sudo netfilter-persistent save
```
50. Reboot Pi. After rebooting, again use SSH app to connect to Pi.
51. Install Kodi:
```
sudo apt-get install kodi
```
52. After installation start Kodi:
```
kodi
```
53. Connect a mouse/keyboard to Pi. Navigate to

`Settings>Services>Control` and enable `Allow remote control via HTTP`.

54. It will be a good thing to go to `addons` and update the currently installed addons.
55. Then navigate back to initial sceen and click the power button on screen. Then click `Exit`.
56. Then copy/paste the following to a cli to create a systemd service for auto start:
```
sudo tee -a /lib/systemd/system/kodi.service <<_EOF_
[Unit]
Description = Kodi Media Center
After = remote-fs.target network-online.target
Wants = network-online.target
[Service]
User = pi
Group = pi
Type = simple
ExecStart = /usr/bin/kodi-standalone
Restart = on-abort
RestartSec = 5
[Install]
WantedBy = multi-user.target
_EOF_
```
Press Enter.

57. Then enable the service:
```
sudo systemctl enable kodi.service
```
58. Shutdown Pi:
```
sudo shutdown now
```
59. If Pi was connected to a monitor, disconnect from it and connect to the desired TV. If Pi is connected to desired TV already then skip this step.
60. `Turn OFF` then `Turn ON` Pi’s adapter. Pi should boot directly to Kodi.
61. Go to phone’s wi-fi settings. You will see the access point we have created in prior step. Connect to it using the ``wpa_passphrase`` you have set in that step.
62. Now go to Kore app on phone. Go through the initial setup and the app will be connected to Kodi.
63. Hurray!! We have successfully installed and booted into Kodi on our Raspberry Pi. You can navigate and control Kodi from Kore app.
