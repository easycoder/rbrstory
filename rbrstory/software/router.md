# Changing the house router #

Connecting to a new Wi-Fi access point solely by modifying config files is a classic system administration task and is very common on headless servers (like Raspberry Pis) or minimal installations.

Using NetworkManager (Recommended for Desktops)
This is the standard method for most distributions like Ubuntu, Fedora, and Debian with a desktop environment.

### Step 1: Create the Configuration File ###
NetworkManager stores each connection as a file in `/etc/NetworkManager/system-connections/`. The filename should be the SSID of the network, but with special characters escaped. It's safest to use a simple filename and define the ssid inside the file.
Create a new file with the .nmconnection extension. Use sudo:
```
sudo nano /etc/NetworkManager/system-connections/MyNewAP.nmconnection
(Replace MyNewAP with a simple name for your connection)
```
       
### Step 2: Add the Connection Details ###
1 Copy the following template into the file and modify it for your network. This is a basic template for a WPA2-Personal network.
```
[connection]
id=MyNewAP
uuid=generate-a-uuid-here
type=wifi
interface-name=wlan0

[wifi]
mode=infrastructure
ssid=The Real SSID of Your Network

[wifi-security]
key-mgmt=wpa-psk
auth-alg=open
psk=YourWiFiPasswordHere

[ipv4]
method=auto

[ipv6]
method=auto
```
2 Crucial Details:
`uuid`: This must be a unique identifier. You can generate one on the command line with the uuidgen command. You must install this command if you don't have it.
```
$ uuidgen
2a8c2df6-0f4d-4c89-8d40-57a0a7d5a1c1 # Copy this output
```
Paste the generated UUID into the `uuid=` field. Using the same UUID for two connections will cause problems.
`interface-name`: Set this to your wireless interface, found with ip a (e.g., wlan0, wlp3s0). You can omit this line to let NetworkManager use any compatible interface.
`ssid`: This is the actual name of the Wi-Fi network (Access Point) you want to connect to. It must be exactly correct, including case and special characters.
`key-mgmt`: wpa-psk is for typical personal networks (WPA2). For open/unsecured networks, use key-mgmt=none and remove the entire `[wifi-security]` section.
`psk`: The plain text password for the network.
    
### Step 3: Set Correct Permissions ###
NetworkManager will ignore the file if its permissions are too open. Set them to be readable only by the root user:
```
sudo chmod 600 /etc/NetworkManager/system-connections/MyNewAP.nmconnection
```

### Step 4: Tell NetworkManager to Reload and Connect ###

1 Reload NetworkManager so it sees the new file:
```
sudo nmcli connection reload
```
2 Bring the connection up. Use the id from the config file (id=MyNewAP), not the filename.
 ```
sudo nmcli connection up MyNewAP
```
3 At this point, if you are working over SSH you will lose the connection. Reconnect your computer to the same access point as the controller, and login again to the controller.

4 Check that you are connected and have an IP address:
```
nmcli connection show --active
ip a show wlan0
ping -c 4 google.com
```

Troubleshooting Tips
1 Check Logs: If it doesn't work, logs are your best friend. Use journalctl to see what's happening:
```
journalctl -u NetworkManager -f
 ```
2 Double-Check SSID and Password: A single typo will cause failure. For the SSID, case sensitivity matters.

By carefully editing these config files and ensuring the correct permissions and syntax, you can reliably connect your Linux machine to any Wi-Fi network without a GUI.

~stid:hardware/controller:The controller hardware~

~stid:home/pagelist:List of Pages~
