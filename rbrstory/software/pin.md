# `pin.py` - pin utilities #

The code is as follows:
```python
from machine import Pin

class PIN():

    def __init__(self,config,pin,invert=False):
        if pin=='': self.pin=None
        else: self.pin=Pin(int(pin),mode=Pin.OUT)
        self.invert=invert
        self.state = None
        
    def on(self):
        if self.pin!=None:
            self.state = True
            if self.invert: self.pin.off()
            else: self.pin.on()
            return True
        return False
    
    def off(self):
        if self.pin!=None:
            self.state = False
            if self.invert: self.pin.on()
            else: self.pin.off()
            return True
        return False

    def getState(self):
        if self.pin==None: return None
        return self.state            
```

This provides functions to control and interrogate ESP32 I/O pins.

~tid:devices:The device software~

~stid:home/pagelist:List of Pages~
