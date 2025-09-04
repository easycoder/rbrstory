# `espcomms.py` - the ESP-Now networking module #

The code is as follows:
```python
import asyncio,network,time,random,machine
from espnow import ESPNow

class ESPComms():
    e=ESPNow()
    
    def __init__(self,config):
        self.config=config
        
        sta=network.WLAN(network.WLAN.IF_STA)
        sta.active(True)
        if config.isMaster():
            print('Starting as master')
            ssid=self.config.getSSID()
            password=self.config.getPassword()
            print(ssid,password)
            print('Connecting...',end='')
            sta.connect(ssid,password)
            while not sta.isconnected():
                time.sleep(1)
                print('.',end='')
            ipaddr=sta.ifconfig()[0]
            self.channel=sta.config('channel')
            self.config.setIPAddr(ipaddr)
            print(f'{ipaddr} ch {self.channel}')
        else:
            print('Starting as slave')
            self.channel=config.getChannel()
        ap=network.WLAN(network.AP_IF)
        ap.active(True)
        ap.config(channel=self.channel)
        mac=ap.config('mac').hex()
        config.setMAC(mac)
        self.ssid=f'RBR-Now-{mac}'
        ap.config(essid=self.ssid,password='00000000')
        ap.ifconfig(('192.168.9.1','255.255.255.0','192.168.9.1','8.8.8.8'))
        self.ap=ap
        config.setAP(ap)
        print('AP:',mac,config.getName(),'channel',self.channel)
        config.startServer()
        
        self.espnowLock=asyncio.Lock()
        self.e.active(True)
        self.peers=[]
        print('ESP-Now initialised')

    def stopAP(self):
        password=str(random.randrange(100000,999999))
        print('Password:',password)
        self.ap.config(essid='-',password=password)
    
    def checkPeer(self,peer):
        if not peer in self.peers:
            self.peers.append(peer)
            self.e.add_peer(peer,channel=self.channel)

    async def send(self,mac,espmsg):
        peer=bytes.fromhex(mac)
        # Flush any incoming messages
        while self.e.any(): _,_=self.e.irecv()
        self.checkPeer(peer)
        try:
            print(f'Send {espmsg[0:20]}... to {mac} on channel {self.channel}')
            result=self.e.send(peer,espmsg)
#            print(f'Result: {result}')
            if result:
                counter=50
                while counter>0:
                    if self.e.any():
                        mac,response = self.e.irecv()
                        if response:
#                            print(f"Received response: {response.decode()}")
                            result=response.decode()
                            break
                    await asyncio.sleep(.1)
                    counter-=1
                if counter==0:
                    result='Response timeout'
            else: result='Fail (no result)'
        except Exception as e:
            print(e)
            result=f'Fail ({e})'
        return result

    async def receive(self):
        print('Starting ESPNow receiver')
        self.messageCount=0
        self.idleCount=0
        while True:
            if self.e.active() and self.e.any():
                async with self.espnowLock:
                    mac,msg=self.e.recv()
                    sender=mac.hex()
                    msg=msg.decode()
    #                print(f'Message from {sender} to {mac}: {msg[0:30]}...')
                    if msg[0]=='!':
                        # It's a message to be relayed
                        comma=msg.index(',')
                        slave=msg[1:comma]
                        msg=msg[comma+1:]
    #                    print(f'Slave: {slave}, msg: {msg}')
                        response=await self.send(slave,msg)
                    else:
                        # It's a message for me
                        response=self.config.getHandler().handleMessage(msg)
                        response=f'{response} {self.getRSS(sender)}'
                        self.messageCount=0
                        self.idleCount=0
                        self.hopping=False
                    print('Response:',response)
                    self.checkPeer(mac)
                    self.e.send(mac,response)
            await asyncio.sleep(.1)
            self.config.kickWatchdog()

    def getRSS(self,mac):
        peer=bytes.fromhex(mac)
        try: return self.e.peers_table[peer][0]
        except: return 0

    async def checkChannels(self):
        channels=[1,6,11]
        self.hopping=False
        while True:
            await asyncio.sleep(1)
            self.messageCount+=1
            if self.hopping: self.idleCount+=1
            
            limit=30 if self.hopping else 3600
            if self.messageCount>limit:
                async with self.espnowLock:
                    for index,value in enumerate(channels):
                        if value==self.channel:
                            self.channel=channels[(index+1)%len(channels)]
                            break
                    # COMPLETELY reset the connection
                    self.e.active(False)
                    await asyncio.sleep(.2)
                    
                    # Re-initialize the AP interface too for clean slate
                    self.ap.active(False)
                    await asyncio.sleep(.1)
                    self.ap.active(True)
                    self.ap.config(channel=self.channel)
                    self.config.setChannel(self.channel)
                    
                    # Create new ESPNow instance
                    self.e = ESPNow()
                    self.e.active(True)
                    self.peers=[]
                    print('Switched to channel',self.channel)
                    self.messageCount=0
                    self.hopping=True

            if self.idleCount>300:
                print('No messages after 3 minutes')
                asyncio.get_event_loop().stop()
                machine.reset()
```

This code handles ESP-Now networking, which operates on the lowest levels of the OSI model. It job is to deliver small packets of data to specified MAC addresses, so it does not deal with IP addresses, access point names or any of the other higher layers.

ESP-Now requires the device to be already set up in either Access Point or Station mode. Its communications go unnoticed by higher-level functions such as HTTP, so it can run concurrently with them.

`__init__` is the class constructor. It sets up an HTTP client (STA), an access point (AP) and the ESP-Now networking. Different flavours of ESP32 behave differently, so this code attempts to cater for all by only requiring features held by the simpler versions such as the ESP32-C3. Full information is hard to find, but the C3 appears to require both STA and AP to be active for ESP-Now to work fully. Omitting STA appears to prevent transmission but the device will still receive. The constructor also sets up an async lock to prevent concurrent operations from interfering with each other.

`stopAP()` stops the access point. To actually do this would disable ESP-Now, so instead it just changes the SSID to a single dash and resets the password to a random value. The effectively makes it impossible to log into the device and cause any damage, intentionally or otherwise. The function is called 2 minutes after startup; until this point the AP is open and available for system management purposes.

`checkPeer()` checks if the supplied MAC address is registered with ESP-Now on the device, without which it will not send messages. If the address is not already registered it is added.

`send()` is an asynchronous function that sends a message to another ESP-Now device. The message must be less than 240 bytes; in the RBR system they are always 100 bytes or less. Although the ESP-Now `send()` function confirms the message was sent it can't confirm that it was received, so the code here expects a response and wait up to 5 seconds for one. It then returns the response to its own caller, or reports an error.

`receive()` is set up as a concurrent function in ~tid:config:the `config.py` module~. It runs a loop waiting for messages to arrive. For each one it calls the ~tid:handler:`handler.py` module~, which processes the message and returns a response which it sends back to the caller to close the loop. A special case is where the message is flagged as being for another device, this being the mechanism by which the RBR system overcomes the limited range of wifi systems by asking some devices to act as repeaters. In theory this can be used to create a network of any physical size, but the code here is limited to a single repeat.

`getRSS()` is a simple function that returns the signal strength of messages received by the device. It can be used to judge whether this device should be set up using another as a repeater.

`checkChannels()` is another concurrent function; to detect when the house router has changed channels. This happens usually late at night; the router scans the network and picks which channel is least crowded. In the 2.4GHz band there are only 3 non-overlapping channels; 1, 6 and 11. In normal use, the device will receive messages from the system controller every 10 seconds or so. If these stop, then after a set time the device will 'hop' to the next channel and resume waiting. The initial wait time is an hour, but once the device is "channel hopping" it does so every 30 seconds. If it receives a message it reverts to the one hour, since it must now be on the right channel. This feature allows the configurator program to stop the system controller so that devices can be reconfigured, and gives the user an hour to complete the tasks before resuming channel hopping. It's unlikely that the router will change channels during this time, so channel hopping will normally only resume if the user forgets to exit the configurator.

A second timer detects when no messages have been received on any of the 3 channels for 3 minutes, and resets the device.

~tid:devices:The device software~

~stid:home/pagelist:List of Pages~
