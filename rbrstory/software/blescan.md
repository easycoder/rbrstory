# `blescan.py` - the BLE device scanner #

The code is as follows:
```python
import asyncio
import aioble
from binascii import hexlify

class BLEScan():
    
    def __init__(self):
        self.values=''

    async def scan(self):
        print("Scanning for BLE devices...")
        while True:
            async with aioble.scan(duration_ms=20000) as scanner:
                async for result in scanner:
                    addr = result.device.addr_hex()
                    if addr.find('a4:c1:38:') == 0:
                        rssi = result.rssi
                        data = hexlify(result.adv_data).decode()
                        if len(data) > 28:
                            temp = int(data[20:24], 16)
                            hum = int(data[24:26], 16)
                            batt = int(data[26:28], 16)
                            print(f'{addr[9:]} {rssi} {temp} {hum} {batt}')
                            self.values = f'{addr[9:]};{rssi};{temp};{hum};{batt}'
#                    else: print(f'{addr} - no')
            await asyncio.sleep(0.1)

    def getValues(self):
        values=self.values
        self.values=''
        return values
```

The code scans for BLE devices and filters out all but those whose MAC address starts with `a4:c1:38:`, this being the identifer of a Mijia thermometeer module flashed with ATC firmware. From each of these it extracts the temperature, humidity and battery level and saves these. When a call is mae to `getValues()` it returns the most recent value, comprising the remainder of the MAC address and the readings from the device.

~tid:devices:The device software~

~stid:home/pagelist:List of Pages~
