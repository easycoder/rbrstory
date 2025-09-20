# `espcomms.py` - the ESP-Now networking module #

The code is as follows:
```python
import asyncio,network,time,random,machine
from espnow import ESPNow

class ESPComms():
    e=ESPNow()
    
    def __init__(self,config):
        self.config=config
        
        if config.isMaster():
            print('Starting as master')
            self.sta=network.WLAN(network.WLAN.IF_STA)
            self.sta.active(True)
            ssid=self.config.getSSID()
            password=self.config.getPassword()
            print(ssid,password)
            print('Connecting...',end='')
            self.sta.connect(ssid,password)
            while not self.sta.isconnected():
                time.sleep(1)
                print('.',end='')
            ipaddr=self.sta.ifconfig()[0]
            self.channel=self.sta.config('channel')
            self.config.setIPAddr(ipaddr)
            print(f'{ipaddr} ch {self.channel}')
        else:
            self.channel=config.getChannel()
            print('Starting as slave on channel',self.channel)
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
        print(config.getName(),mac)
        if not config.isMaster():
            self.sta=network.WLAN(network.WLAN.IF_STA)
            self.sta.active(True)
            self.sta.config(channel=self.channel)
        
        self.e.active(True)
        print('ESP-Now initialised')
        
        self.requestToSend=False
        self.sending=False

    def closeAP(self):
        password=str(random.randrange(100000,999999))
        print('Password:',password)
        self.ap.config(essid='-',password=password)
            
    def addPeer(self,peer):
        h=peer.hex()
        if not hasattr(self,'peers'):
            self.peers=[]
        if h in self.peers:
            return True
        try:
            self.e.add_peer(peer,channel=self.channel)
        except OSError as ex:
            print(f'Failed to add peer {h} to ESP-NOW: {ex}')
            return False
        self.peers.append(h)
        print('Added',h,'to peers on channel',self.channel)
        return True

    def espSend(self,peer,msg):
        if self.addPeer(peer):
            try: self.e.send(peer,msg)
            except Exception as ex: print('espSend:',ex)
    
    def send(self,mac,msg):
#        print(f'Send {msg[0:20]}... to {mac} on channel {self.channel}')
        self.requestToSend=True
        while not self.sending: await asyncio.sleep(.1)
        self.requestToSend=False
        peer=bytes.fromhex(mac)
        if self.addPeer(peer):
            try:
                result=self.e.send(peer,msg)
                if result:
                    result=None
                    counter=100
                    while counter>0:
                        while self.e.any():
                            _,reply=self.e.irecv()
                            if reply:
                                reply=reply.decode()
                                if reply=='ping': continue
#                                print(f"Received reply: {reply}")
                                result=reply
                                break
                        if result: break
                        await asyncio.sleep(.1)
                        counter-=1
                    if counter==0: result='Fail (no reply)'
                    else:
                        print(f'{msg[0:20]} to {mac}: {result}')
                        self.config.resetCounters()
                else: result='Fail (no result)'
            except Exception as ex:
                result=f'Fail ({ex})'
                print(result)
        else: result='Fail (adding peer)'
        self.sending=False
        return result

    async def receive(self):
        print('Starting ESPNow receiver')
        while True:
            while True:
                if self.e.active():
                    while self.e.any():
                        mac,msg=self.e.recv()
                        msg=msg.decode()
                        print('Received',msg)
                        if msg=='ping':
                            try:
                                self.addPeer(mac)
                                self.e.send(mac,'pong')
                            except Exception as ex: print('ping:',ex)
                        else:
                            if msg[0]=='!':
                                # It's a message to be relayed
                                comma=msg.index(',')
                                slave=msg[1:comma]
                                msg=msg[comma+1:]
    #                            print(f'Slave: {slave}, msg: {msg}')
                                response=await self.send(slave,msg)
                            else:
                                # It's a message for me
                                response=self.config.getHandler().handleMessage(msg)
                                response=f'{response} {self.getRSS(mac)}'
                            self.addPeer(mac)
                            try:
                                self.addPeer(mac)
                                self.e.send(mac,response)
                                print(response)
                                self.config.resetCounters()
                                if not self.config.getMyMaster() and not self.config.isMaster():
                                    self.config.setMyMaster(mac.hex())
                            except Exception as ex: print('Can\'t respond',ex)
                    if self.requestToSend:
                        self.sending=True
                        while self.sending: await asyncio.sleep(.1)
                else: print('Not active')
                await asyncio.sleep(.1)
                self.config.kickWatchdog()

    def getRSS(self,mac):
        try: return self.e.peers_table[mac][0]
        except: return 0

    def restartESPNow(self):
        self.e=ESPNow()
        self.e.active(True)
```

This code handles ESP-Now networking, which operates on the lowest levels of the OSI model. It job is to deliver small packets of data to specified MAC addresses, so it does not deal with IP addresses, access point names or any of the other higher layers.

ESP-Now requires the device to be already set up in either Access Point or Station mode. Its communications go unnoticed by higher-level functions such as HTTP, so it can run concurrently with them.

The code in this module is for a specific network scenario, where there is a single master and a number of slaves. The master (or hub) device connects to the house router with its station (STA) interface and the slaves do not (but they still require STA to be active for ESP-Now to work). Both master and slaves use the same code but use it in different ways. In both case an access point (AP) is set up for the first 2 minutes following a reboot, so that an administrator can talk directly to the device. This permits configuration changes to be made over-the-air from a local browser.

The code here was developed with a lot of trial-and-error and the assistance of AI to find relevant examples. There turned out to be none that directly considered the complexities of the scenario; most are far simpler. If anyone else has solved the same problem they hadn't told the world about it at the time this code development was under way.

### `__init__()` ###
This code makes use of a second module, "config", which handles system parameters such as route SSID/password and identifies which I/O pins are used by the device. This data is typically held in a JSON file that can be read and written as necessary. Wherever you see `config.xxxxx()` it's a function in the config module that's being called.

ESP-Now is extremely fusy about configuration. There may be differences between different variants of ESP32; this code is known to work on ESP32-C3, which is a fairly basic model, so it ought to be fine on most other variants. There is a strict sequence to follow, which differs between master and slave.

The master first requires STA to be active. Here it is connected to the house router, which governs the wifi channel the system will use. Then the AP is set up, and finally ESP-Now is activated. This ordering guarantees that ESP-Now will use the STA channel.

The slave does things the other way round, first setting up AP and following that with a stub STA. This ensures that the AP channel will be used. If you don't get this order right, some things will probably not work and strange errors will arise.

### `closeAP()` ###
During the first 2 minutes following boot, the AP can be connected from a browser and will accept requests and commands. This includes rewriting the JSON config file and rebooting the device. After this period the SSID of the AP is changed to `-` and a random password is applied, making it almost impossible to connect. This is a security feature that prevents anyone interfering unless they can physically repower the device.

### `addPeer()` ###
It's very easy to get ESP-Now peer management wrong. The documentation gives few hints as to how fussy the protocol is about handling peers. For example, it fails to mention that a channel change not only invalidates the peers table but makes it impossible to recreate. Only a reset will do that.

Although ESP-Now will happily receive messages from anywhere, sending can only be done to a peer whose MAC address has been registered with `e.add_peer()`. This must be done just once for any given peer or an error will be thrown. So this function manages its own peers table to track what has already been added. Note that the device may have 2 MAC addresses; one for each of its 2 interfaces. When you initiate a send it will usually be to one of these, but when replying to a message it will be the other. Both must be added to the peers table.

### `send()` and `receive()` ###
The `receive()` function runs synchronously in a loop, waiting for messages and replying to them. It might be considered sensible to use a lock so that send and receive can be exclusive, but this gives rise to some horrible issues that are hard to track down. The strategy here is one that avoids such issues.

In the receive loop, when a message arrives it is decoded by the `handleMessage()` function in a companion `Handler` class, which returns a reply to send back to the caller. While this is happening, a `sending` flag is set `False`. Any request to send will block, using `requestToSend`, until `sending` goes `True`, which is done at the bottom of the receive loop. The message can now be sent and the code waits for a reply.

However, this might not come from the device we just sent to. This system is designed to handle router channel changes, and when this happens the master quickly changes to the new channel, but all the slaves know is that messages have stopped arriving (because they're all on the wrong channel). So after a while they issue a 'ping' message to the master. If it replies they're on the right channel, otherwise they change channels and try the ping again.

The result is that one or more 'ping' messages can arrive when the sender is expecting a reply to its own message, so the code watches out for them and discards them. Eventually it sees its own reply message and returns it to the caller, but before doing so it clears `sending` so the receiver can look out for more incoming messages.

Note that this code cannot deal with anything but 'ping' messages from its slaves. Any scenario requiring a more general 2-way send and receive will probably need to use some form of addressing to identify message senders.

The code in `receive()` has one more feature. If the received message starts with `!` it's not for this device, but instead is a request to forward a message to another device. The MAC address of the intended recipient follows the `!` and is terminated by a comma. The code strips these away and is left with a MAC address and a (shorter) message, which it forwards, returning whatever returns back up to its own caller.

## Channel changes ##
The code to follow a router channel change is in the Channels class.

~tid:handler:The message handler~  
~tid:channels:Dealing with channel changes~  
~tid:devices:The device software~

~stid:home/pagelist:List of Pages~
