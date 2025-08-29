# `ap.py` - the access point #

The code is as follows:
``` Python
import network,binascii,asyncio,random

class AP():
    
    def __init__(self,config):
        self.config=config
        config.setAP(self)
        ap=network.WLAN(network.AP_IF)
        ap.active(True)
        self.ap=ap
        mac=binascii.hexlify(self.ap.config('mac')).decode()
        self.config.setMAC(mac)
        self.ssid=f'RBR-Now-{mac}'
        ap.config(essid=self.ssid,password='00000000')
        ap.config(channel=config.getChannel())
        ap.ifconfig(('192.168.9.1','255.255.255.0','192.168.9.1','8.8.8.8'))
        print(mac,config.getName()) 

    def stop(self):
        password=str(random.randrange(100000,999999))
        print(password)
        self.ap.config(essid='-',password=password)

    def getChannel(self): return self.ap.config('channel')
```

The code sets up a simple access point, using its own MAC address and the password `00000000`, on the wifi channel defined in the config file. The access point is set up when the device first starts, and disables it after 2 minutes. During this time it can be accessed by another device on the network such as a PC, for the purpose of configuration. After the initial 2 minutes, the only way to communicate with the device is by ESP-Now, which requires the sender to know the MAC address of the device.

~tid:devices:The device software~

~stid:home/pagelist:List of Pages~
