# `espcomms.py` - the ESP-Now networking module #

The code is as follows:
```python
import asyncio
from binascii import hexlify,unhexlify
from espnow import ESPNow as E

class ESPComms():
    
    def __init__(self,config):
        self.config=config
        config.setESPComms(self)
        E().active(True)
        self.peers=[]
        print('ESP-Now initialised')
    
    def checkPeer(self,peer):
        if not peer in self.peers:
            self.peers.append(peer)
            E().add_peer(peer)

    async def send(self,mac,espmsg):
        peer=unhexlify(mac.encode())
        # Flush any incoming messages
        while E().any(): _,_ = E().irecv()
        self.checkPeer(peer)
        try:
            result=E().send(peer,espmsg)
            if result:
                counter=50
                while counter>0:
                    if E().any():
                        mac,response = E().irecv()
                        if response:
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
        print('Starting ESPNow receiver on channel',self.config.getChannel())
        self.waiting=False
        while True:
            if E().any():
                mac,msg=E().recv()
                sender=hexlify(mac).decode()
                msg=msg.decode()
                if msg[0]=='!':
                    # It's a message to be relayed
                    comma=msg.index(',')
                    slave=msg[1:comma]
                    msg=msg[comma+1:]
                    response=await self.send(slave,msg)
                else:
                    # It's a message for me
                    response=self.config.getHandler().handleMessage(msg)
                    response=f'{response} {self.getRSS(sender)}'
                print(f'Response to {sender}: {response}')
                self.checkPeer(mac)
                E().send(mac,response)
            await asyncio.sleep(.1)
            self.config.kickWatchdog()

    def getRSS(self,mac):
        peer=unhexlify(mac.encode())
        try: return E().peers_table[peer][0]
        except: return 0
```

This code handles ESP-Now networking, which operates on the lowest levels of the OSI model. It job is to deliver small packets of data to specified MAC addresses, so it does not deal with IP addresses, access point names or any of the other higher layers.

ESP-Now requires the device to be already set up in either Access Point or Station mode. Its communications go unnoticed by higher-level functions such as HTTP, so it can run concurrently with them.

`checkPeer()` checks if the supplied MAC address is registered with ESP-Now on the device, without which it will not send messages.

`send()` is an asynchronous function that sends a message to another ESP-Now device. The message must be less than 240 bytes; in the RBR system they are always 100 bytes or less. Although the ESP-Now `send()` function confirms the message was sent it can't confirm that it was received, so the code here expects a response and wait up to 5 seconds for one. It then returns the response to its own caller, or reports an error.

`receive()` is set up as a concurrent function in ~tid:main:the `main.py` module~. It runs a loop waiting for messages to arrive and calling the ~tid:handler:`handler.py` module~, which processes the message and returns a response which it sends back to the caller to close the loop. A special case is where the message is flagged as being for another device, this being the mechanism by which the RBR system overcomes the limited range of wifi systems by asking some devices to act as repeaters.

`getRSS()` is a simple function that returns the signal strength of messages received by the device. It can be used to judge whether this device should be set up using another as a repeater.

~tid:devices:The device software~

~stid:home/pagelist:List of Pages~
