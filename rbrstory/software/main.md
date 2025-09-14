# `main.py` - the main program #

The code is as follows:
```python
import asyncio,machine
from config import Config
from pin import PIN

class RBRNow():

    def init(self):
        config=Config()
        self.led=config.getLED()
        self.config=config
        asyncio.create_task(self.startBlink())
        asyncio.create_task(self.closeAP())
        self.config.doFinalInitTasks()

    async def blink(self):
        while True:
            if self.config.resetRequested:
                asyncio.get_event_loop().stop()
                machine.reset()
            self.led.on()
            if self.blinkCycle=='init':
                await asyncio.sleep(0.5)
                self.led.off()
                await asyncio.sleep(0.5)
                self.config.addUptime(1)
            elif self.blinkCycle=='master':
                await asyncio.sleep(0.2)
                self.led.off()
                await asyncio.sleep(0.2)
                self.led.on()
                await asyncio.sleep(0.2)
                self.led.off()
                await asyncio.sleep(4.6)
                self.config.addUptime(5)
            elif self.blinkCycle=='slave':
                await asyncio.sleep(0.2)
                self.led.off()
                await asyncio.sleep(4.8)
                self.config.addUptime(5)

    async def startBlink(self):
        self.uptime=0
        self.blinkCycle='init'
        await self.blink()

    async def closeAP(self):
        while True:
            print('2-min config window')
            await asyncio.sleep(120)
            if self.config.isMaster():
                self.blinkCycle='master'
                break
            if self.config.getMyMaster()!=None:
                self.blinkCycle='slave'
                self.config.closeAP()
                break

if __name__ == '__main__':
    RBRNow().init()
    try: asyncio.get_event_loop().run_forever()
    except: pass
    print('Finished')
    machine.reset()
```

The main program has very little to do beyond running the on-board LED to indicate the system state. When it starts it creates the ~tid:config:`config.py`~ object where most of the real work is done.

On startup, the LED flashes 500mS on, 500mS off for 2 minutes, during which time the AP will be open, allowing the device to be accesed for reprogramming. Since slave devices do not have an operational STA interface, the only way to talk to them after 2 minutes is via ESP-Now, where all messages go through the master device.

After 2 minutes the AP is deactivated (made impossible to connect to, since true deactivation will knock out ESP-Now). After that, the master device blinks the LED twice briefly on a 5-second cycle, while a slave blinks only once. You can tell very easily which is which and which state they are in.

Inside the blink, the code checks for a reset request, and if it is found, closes down the async loop and restarts the device.

~tid:devices:The device software~

~stid:home/pagelist:List of Pages~
