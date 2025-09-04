# `config.py` - the configuration code #

The code is as follows:
```python
import json,asyncio,machine
from files import readFile,writeFile,fileExists
from pin import PIN
from server import Server
from dht22 import DHT22
from handler import Handler
from blescan import BLEScan
from espcomms import ESPComms

class Config():

    def __init__(self):
        if fileExists('config.json'):
            self.config=json.loads(readFile('config.json'))
        else:
            self.config={}
            self.config['name']='(none)'
            self.config['master']=False
            self.config['path']=''
            self.config['channel']=1
            self.config['pins']={}
            pin={}
            pin['pin']=''
            pin['invert']=False
            self.config['pins']['led']=pin
            pin={}
            pin['pin']=''
            pin['invert']=False
            self.config['pins']['relay']=pin
            pin={}
            pin['pin']=''
            self.config['pins']['dht22']=pin
            writeFile('config.json',json.dumps(self.config))
        self.channel=int(self.config['channel'])
        pin,invert=self.getPinInfo('led')
        self.led=PIN(self,pin,invert)
        pin,invert=self.getPinInfo('relay')
        self.relay=PIN(self,pin,invert)
        pin,_=self.getPinInfo('dht22')
        if pin=='': self.dht22=None
        else:
            self.dht22=DHT22(pin)
            asyncio.create_task(self.dht22.measure())
        self.ipaddr=None
        self.uptime=0
        self.hop=True
        self.resetRequested=False
        self.server=Server(self)
        self.handler=Handler(self)
        self.bleScan=BLEScan()
        print('Config: set up comms')
        self.espComms=ESPComms(self)
        asyncio.create_task(self.runWatchdog())

    async def respond(self,response,writer):
        await self.server.respond(response,writer)
    async def sendDefaultResponse(self,writer):
        await self.server.sendDefaultResponse(writer)
    async def handleClient(self,reader,writer):
        await self.server.handleClient(reader,writer)

    async def send(self,peer,espmsg): return await self.espComms.send(peer,espmsg)
    
    def reset(self):
        print('Reset requested')
        self.resetRequested=True
    
    def pause(self):
        if self.dht22!=None: self.dht22.pause()
    
    def resume(self):
        if self.dht22!=None: self.dht22.resume()
    
    def doFinalInitTasks(self):
        if not self.isMaster():
            asyncio.create_task(self.espComms.receive())
            asyncio.create_task(self.espComms.checkChannels())
        asyncio.create_task(self.bleScan.scan())

    def setAP(self,ap): self.ap=ap
    def setSTA(self,sta): self.sta=sta
    def setESPComms(self,espComms): self.espComms=espComms 
    def setMAC(self,mac): self.mac=mac
    def setIPAddr(self,ipaddr): self.ipaddr=ipaddr
    def setChannel(self,channel):
        self.channel=channel
        self.config['channel']=channel
        writeFile('config.json',json.dumps(self.config))
    def setHandler(self,handler): self.handler=handler
    def addUptime(self,t): self.uptime+=t
    
    def isMaster(self): return self.config['master']
    def getDevice(self): return self.config['device']
    def getName(self): return self.config['name']
    def getSSID(self): return self.config['hostssid']
    def getPassword(self): return self.config['hostpass']
    def getMAC(self): return self.mac
    def stopAP(self): self.espComms.stopAP()
    def startServer(self): self.server.startup()
    def getIPAddr(self): return self.ipaddr
    def getChannel(self): return self.channel
    def getHandler(self): return self.handler
    def getESPComms(self): return self.espComms
    def getBLEScan(self): return self.bleScan
    def getRBRNow(self): return self.rbrNow
    def getPinInfo(self,name):
        pin=self.config['pins'][name]
        print(name,pin)
        if 'invert' in pin: invert=pin['invert']
        else: invert=False
        return pin['pin'],invert
    def getLED(self): return self.led
    def getRelay(self): return self.relay
    def getUptime(self): return int(round(self.uptime))
    def getTemperature(self): return self.dht22.getTemperature()

    async def runWatchdog(self):
        while True:
            self.active=False
            await asyncio.sleep(180)
            if not self.active:
                print('No activity - reset')
                asyncio.get_event_loop().stop()
                machine.reset()

    def kickWatchdog(self):
        self.active=True
```
The code starts by reading the configuration JSON script for the device. On the first run this will not exist, so it sets up a default file. It then sets up several of the other modules using the data from the config script.

There then follows a number of functions, most of which are implemented by calls to similarly named functions in one of the other code modules. Owing to space limitations, comments are rare, but the code is in most cases quite easy to read. Where possible there are descriptions of each function in the relevant parts of this documentation.

The function `runWatchdog()` sets up a loop that waits for 3 minutes. If no activity has taken place in that time it forces the device to reset. The function `kickWatchdog()` is called by various other modules to signal that some activity has taken place; this prevents the watchdog from timing out for another 3 minutes (not cumulative, but starting from that point).

~tid:devices:The device software~

~stid:home/pagelist:List of Pages~
