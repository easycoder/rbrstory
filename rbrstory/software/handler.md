# `handler.py` - the message handler #

The code is as follows:
```python
import asyncio,machine
from binascii import unhexlify
from files import readFile,writeFile,renameFile,deleteFile,createDirectory

class Handler():
    
    def __init__(self,config):
        self.config=config
        config.setHandler(self)
        self.relay=config.getRelay()
        self.saveError=False

    def handleMessage(self,msg):
#        print('Handler:',msg)
        bleValues=self.config.getBLEValues()
        response=f'OK {self.config.getUptime()} :{bleValues}'
        if msg=='uptime':
            pass
        elif msg == 'on':
            response=f'{response} ON' if self.relay.on() else 'No relay'
        elif msg == 'off':
            response=f'{response} OFF' if self.relay.off() else 'No relay'
        elif msg == 'relay':
            try:
                response=f'OK {self.relay.getState()}'
            except:
                response='No relay'
        elif msg=='reset':
            self.config.reset()
            response='OK'
        elif msg == 'ipaddr':
            response=f'OK {self.config.getIPAddr()}'
        elif msg=='channel':
            response=f'OK {self.config.getChannel()}'
        elif msg[0:8]=='channel=':
            channel=msg[8:]
            self.config.setChannel(channel)
            response=f'OK {channel}'
        elif msg[0:7]=='environ':
            environ=msg[8:]
            print('Set environ to',environ)
            self.config.setEnviron(environ)
            response=f'OK {environ}'
        elif msg=='temp':
            response=f'OK {self.config.getTemperature()}'
        elif msg=='pause':
            self.config.pause()
            response=f'OK paused'
        elif msg=='resume':
            self.config.resume()
            response=f'OK resumed'
        elif msg[0:6]=='delete':
            file=msg[7:]
            response='OK' if deleteFile(file) else 'Fail'
        elif msg[0:4]=='part':
        # Format is part:{n},text:{text}
            part=None
            text=None
            items=msg.split(',')
            for item in items:
                item=item.split(':')
                label=item[0]
                value=item[1]
                if label=='part': part=int(value)
                elif label=='text': text=value
            if part!=None and text!=None:
                text=text.encode('utf-8')
                text=unhexlify(text)
                text=text.decode('utf-8')
                if part==0:
                    self.buffer=[]
                    self.pp=0
                    self.saveError=False
                else:
                    if self.saveError:
                        return 'Save error'
                    else:
                        if part==self.pp+1:
                            self.pp=part
                        else:
                            self.saveError=True
                            return f'Sequence error: {part} {self.pp+1}'
                self.buffer.append(text)
                response=f'{part} {str(len(text))}'
        elif msg[0:4]=='save':
            if len(self.buffer[0])>0:
                file=msg[5:]
                print(f'Save {file}')
                size=0
                f = open(file,'w')
                for n in range(0, len(self.buffer)):
                    f.write(self.buffer[n])
                    size+= len(self.buffer[n])
                f.close()
                # Check the file against the buffer
                if self.checkFile(self.buffer, file): response=str(size) 
                else: response='Bad save'
            else: response='No update'
            text=None
        elif msg[0:5]=='mkdir':
            path=msg[6:]
            print(f'mkdir {path}')
            response='OK' if createDirectory(path) else 'Fail'
        else: response=f'Unknown message: {msg}'
#        print('Handler response:',response)
        return response

    def checkFile(self, buf, file):
        try:
            with open(file, 'r') as f:
                i = 0  # index in lst
                pos = 0  # position in current list item
                while True:
                    c = f.read(1)
                    if not c:
                        # End of file: check if we've also finished the list
                        while i < len(buf) and pos == len(buf[i]):
                            i += 1
                            pos = 0
                        return i == len(buf)
                    if i >= len(buf) or pos >= len(buf[i]) or c != buf[i][pos]:
                        return False
                    pos += 1
                    if pos == len(buf[i]):
                        i += 1
                        pos = 0
        except OSError:
            return False
```
This module handles messages arriving either via AP or from STA on the master device and ESP-Now on a slave. Messages are processed before arriving here, to remove HTTP information or ESP-Now headers, so this module only has to deal with the message content.

The response to most messages is 'OK' plus reply information plus the most recent information broadcast by a Mijia thermometer and detected by this device.

The messages handled are as follows:

`uptime` returns the number of seconds since the device booted.  
`on` turns on the device relay and reports `ON` if it succeeded, otherwise `No relay`.  
`off` turns off the device relay and reports `OFF` if it succeeded, otherwise `No relay`.  
`relay` retuns the current state on the relay (True or False), or `No relay`.  
`reset` requests the device to reset itself.  
`ipaddr` returns the IP adress if the device is a master.  
`channel` returns the channel number currently being used by the device.  
`temp` returns the temperature read by the device, or 0 if there's no thermometer.  
`pause` is an instruction for the thermometer (if present) to pause measuring.  
`resume` is an instruction for the thermometer to resume measuring.  
`delete` is a request to delete the file whose name is given.  
`part` is a message sent by the system configurator, that contains part of a file to be downloaded. The message has 2 components; the part number and the text of the file. Part numbers must arrive in strict ascending sequence or an error is thrown. Each part is added to a list of parts, then a return value is sent comprising the part number and the length of the part. At some point in the future a checksum or even CRC could be added, but so far there is no evidence that data is being corrupted to the point it can't be detected by this simple method.  
`save` is a request sent by the system configurator, to save the collected parts in the file whose name is given. This has to be done carefully, as a faulty file will usually cause the device to crash and become unusable, requiring its box to be opened and the CPU module extracted for reprogramming.

`checkFile()` is a function that reads back the saved file and compares it with the contents of the download list. If it doesn't match, the configurator will retry the entire download up to 10 times before abandoning it.

~tid:devices:The device software~

~stid:home/pagelist:List of Pages~
