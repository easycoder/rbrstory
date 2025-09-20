# `channels.py` - channel management

The code is as follows:
```python
import asyncio,machine,time
from espnow import ESPNow

class Channels():
    def __init__(self,espComms):
        print('Starting Channels')
        self.espComms=espComms
        self.config=espComms.config
        self.channels=[1,6,11]
        self.myMaster=self.config.getMyMaster()
        if self.config.isMaster():
            self.ssid=self.config.getSSID()
            self.password=self.config.getPassword()
            asyncio.create_task(self.checkRouterChannel())
        self.resetCounters()
    
    def setupSlaveTasks(self):
        asyncio.create_task(self.findMyMaster())
        asyncio.create_task(self.countMissingMessages())

    def resetCounters(self):
        print('Resetting counters')
        self.messageCount=0
        self.idleCount=0

    async def findMyMaster(self):
        if self.myMaster:
            if await self.ping(): return
        else:
            print('Waiting for master')
            for count in range(100):
                self.myMaster=self.config.getMyMaster()
                if self.myMaster:
                    print('Found master',self.myMaster,'on channel',self.espComms.channel)
                    return
                await asyncio.sleep(.1)
        self.hopToNextChannel()
        asyncio.get_event_loop().stop()
        machine.reset()
    
    async def ping(self):
        peer=bytes.fromhex(self.myMaster)
        self.espComms.espSend(peer,'ping')
        _,msg=self.espComms.e.recv(1000)
        print('Ping response from',self.myMaster,':',msg)
        if msg!=None:# and msg.decode()=='pong':
            print('Found master on channel',self.espComms.channel)
            return True
        return False

    async def countMissingMessages(self):
        print('Count missing messages')
        espComms=self.espComms
        ap=espComms.ap
        e=espComms.e
        while True:
            await asyncio.sleep(1)
            if not self.myMaster: continue

            self.messageCount+=1
            self.idleCount+=1
            
            limit=30
            if self.messageCount>limit and not espComms.config.isMaster():
                print('No messages for 30 seconds')
                # Retry the current channel
                if await self.ping():
                    self.resetCounters()
                    continue
                self.hopToNextChannel()
                channel=self.hopToNextChannel()
                asyncio.get_event_loop().stop()
                machine.reset()

            if self.idleCount>300:
                print('No messages after 3 minutes')
                asyncio.get_event_loop().stop()
                machine.reset()
                
    def hopToNextChannel(self):
        index=-1
        for n,value in enumerate(self.channels):
            if value==self.espComms.channel:
                self.espComms.channel=self.channels[(n+1)%len(self.channels)]
                index=n
                break
        if index==-1: self.espComms.channel=self.channels[0]
        self.config.setChannel(self.espComms.channel)
    
    async def checkRouterChannel(self):
        print('Check the router channel')
        while True:
            await asyncio.sleep(300)
            sta=self.espComms.sta
            sta.disconnect()
            time.sleep(1)
            print('Reconnecting...',end='')
            sta.connect(self.ssid,self.password)
            while not sta.isconnected():
                time.sleep(1)
                print('.',end='')
            self.espComms.restartESPNow()
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
