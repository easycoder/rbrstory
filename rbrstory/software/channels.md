# `channels.md` channel management

The code is as follows:
```python
import asyncio,machine,time
from espnow import ESPNow

class Channels():
    def __init__(self,espComms):
        self.espComms=espComms
        self.channels=[1,6,11]
        if self.espComms.config.isMaster():
            self.ssid=espComms.config.getSSID()
            self.password=espComms.config.getPassword()
            asyncio.create_task(self.checkRouterChannel())
        self.resetCounters()
        asyncio.create_task(self.countMissingMessages())

    def resetCounters(self):
        self.messageCount=0
        self.idleCount=0

    async def countMissingMessages(self):
        print('Count missing messages')
        espComms=self.espComms
        ap=espComms.ap
        e=espComms.e
        while True:
            await asyncio.sleep(1)

            self.messageCount+=1
            self.idleCount+=1
            
            limit=30
            if self.messageCount>limit and not espComms.config.isMaster():
                print('No messages for 30 seconds')
#                async with espComms.espnowLock:
#                    for index,value in enumerate(self.channels):
#                        if value==espComms.channel:
#                            espComms.channel=self.channels[(index+1)%len(self.channels)]
#                            break
#                    self.restartESPNow()
#                    print('Switched to channel',espComms.channel)
#                    self.messageCount=0
                index=-1
                # Find the current index
                for n,value in enumerate(self.channels):
                    if value==espComms.channel:
                        espComms.channel=self.channels[(n+1)%len(self.channels)]
                        index=n
                        break
                if index==-1:
                    print('Bad channel number')
                    asyncio.get_event_loop().stop()
                    machine.reset()
                # Go round the loop up to 10 times
                for counter in range[0:30]:
                    await self.restartESPNow()
                    e.send(espComms.sender,'ping')
                    _,msg=self.e.recv(1000)
                    if msg!=None and msg.decode()=='pong':
                        print('Found master on channel',espcomms.channel)
                        self.resetCounters()
                        break
                    else: # Change channels
                        index=(index+1)%len(self.channels)
                        espComms.channel=self.channels[index]
                # Give up
                asyncio.get_event_loop().stop()
                machine.reset()

            if self.idleCount>300:
                print('No messages after 3 minutes')
                asyncio.get_event_loop().stop()
                machine.reset()
    
    async def restartESPNow(self):
        espComms=self.espComms
        ap=espComms.ap
        e=espComms.e
        e.active(False)
        await asyncio.sleep(.2)            
        ap.active(False)
        await asyncio.sleep(.1)
        ap.active(True)
        ap.config(channel=espComms.channel)
        espComms.config.setChannel(espComms.channel)   
        e=ESPNow()
        e.active(True)
        self.peers=[]
        self.messageCount=0
        print('Switched to channel',espComms.channel)

    async def checkRouterChannel(self):
        while True:
            await asyncio.sleep(60)
            sta=self.espComms.sta
            sta.disconnect()
            time.sleep(1)
            print('Reconnecting...',end='')
            sta.connect(self.ssid,self.password)
            while not sta.isconnected():
                time.sleep(1)
                print('.',end='')
            self.restartESPNow()
            channel=sta.config('channel')
            if channel!=self.espComms.channel:
                print(' router changed channel from',self.espComms.channel,'to',channel)
                asyncio.get_event_loop().stop()
                machine.reset()
            print(' no channel change')
```
The job of this module is to handle channel changes. Modern wifi routers maintain on a watch on network congestion, and at times move from a busy channel to one less busy. Computer operating systems deal with this down in their network drivers; warnings are posted by a router before a change so there is minimal disruption when it occurs.

Simple devices like ESP32 are not similarly equipped. when a channel change occurs, all comms go dead. The master device soon detects the channel change and forces a reset, but for the slave it's more difficult. Channel changes are only one way that messages stop arriving and the slave has no way of knowing which it is. One particular case is during system configuration, where normal messaging is halted. The user manages the system manually and most of the slaves stop receiving messages. A strategy is needed to keep the slave from assuming a router channel change. There are 3 ways to deal with it:

 1. Scan through the channels until a transmission is seen from the same device. If messages are infrequent this involves a long period of waiting on each channel before the next one is tried.
 1. Scan the channels and for each one, ping the other device to see if it is occupying that channel. This is potentially quick but involves a lot of deactivating and reactivating the network interface(s) and ESP-Now. It appears that on at least some ESP32 variants, the device holds onto old configuration data and doesn't permit updates, throwing an exception instead.
 1. Save the channel number and the MAC address of the other device, then reset. On power up, ping the other device. This is similar to 2, but has the advantage that no deactivation and reactivation is needed; ust regular message passing.
 
The current code uses option 3, apart for initial (pre-configuration) startup, where the identity of the master device is unknown. Here we have to use option 1.
