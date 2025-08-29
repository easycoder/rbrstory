# `dht22.py` - the DHT22 thermometer #

The code is as follows:
```python
import asyncio,dht
from machine import Pin,reset

class DHT22():

    def __init__(self,sensorpin,verbose=False):
        if verbose:
            print('Initialise sensor on pin',sensorpin)
        if sensorpin is '': self.sensor=None
        else: self.sensor=dht.DHT22(Pin(int(sensorpin)))
        self.verbose=verbose
        self.temp=0
        self.errors=0
        self.paused=False

    async def measure(self):
        if self.sensor==None: return
        msg=None
        print('Run the temperature sensor')
        while True:
            if not self.paused:
                try:
                    self.sensor.measure()
                    temperature=self.sensor.temperature()
                    if temperature>50: temperature=0
                    self.temp=round(temperature*10)
                    if self.verbose:
                        print('Temperature:',temperature)
                    self.errors=0
                except OSError as e:
                    if self.verbose:
                        msg=f'Failed to read sensor: {str(e)}'
                        print(msg)
                    self.errors+=1
                    if self.errors>50: reset()
            await asyncio.sleep(5)
    
    def pause(self):
        self.paused=True
    
    def resume(self):
        self.paused=False

    def getTemperature(self):
        return self.temp
```

This handles a DHT22 temperature sensor driven by a pin of the ESP32. The module is initialized by its constructor, then the ~tid:config:config module~ adds a call to `measure()` to the multitasking (`async`) framework, ensuring it runs continuously in the background. The module can be paused and restarted by calls to `pause()` and `resume()`, and `getTemperature()` returns the current temperature.

~tid:devices:The device software~

~stid:home/pagelist:List of Pages~
