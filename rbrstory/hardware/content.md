# The Room By Room hardware #

Room By Room is a distributed appliance that replaces a traditional central heating controller (programmer). Each radiator is fitted with a electrically-operated valve, which in many cases replaces an existing thermostatic radiator valve (TRV). Fitting is just a matter of unscrewing (by hand) the TRV head and replacing it with the electrical head. This has a cable that goes to a small relay box which in turn is wired to a standard mains outlet. The relay box is described below.

Each room also has a thermometer, of which there are currently 2 kinds. One is a self-contained battery-powered device, and the other is mains-powered; a variant of the radiator relay. These are also described below.

To control these devices there is a system controller; a small computer that connects to the devices and to the Internet via the house router.

The system includes a user interface, that runs either locally or remotely over the Internet, or both. The local version, if provided, runs on the system controller, and the remote version is designed to run on a smartphone connected to the RBR website. Both give a view of the current system status, with temperature and timing information for each room, and the ability to configure the system as and when needed.

A Room By Room system comprises one system controller and a number of devices to be controlled, as follows.

### ~tid:controller:The system controller~ ###
This is a small Linux computer. It is possible to use a device such as a Raspberry Pi or similar, but we have found them to be unreliable in the long term if they use an SD card for data storage. If the power is lost unexpectedly the computer can corrupt the SD card, making it unusable without a complete reinstall. For this reason we prefer a mini PC of some kind or other, with SSD storage. If this comes with a display then we can provide a user interface program to run on it; otherwise the only way to control the system is over the Internet with a smartphone or PC.

### ~tid:relay:Relay modules~ ###
These are based on the ESP32 microcontroller. They communicate with the system controller using wifi.

### ~tid:thermometer:Thermometer modules~ ###
These come in two kinds. The battery-powered thermometer communicates with the system using Bluetooth. Batteries usually need to be changed at roughly yearly intervals. The mains-powered version needs no battery changes; it plugs into a nearby mains socket outlet and communicates using wifi.

## System software ##
The system software is distributed among the system controller, the relays and thermometers, the RBR website and a web application that runs on a smartphone. Here are links to the various pages describing each software component.

~sid:software:Introduction~  
~stid:software/controller:The controller software~  
~stid:software/devices:The device software~  
~stid:software/webserver:The web server~  
~stid:software/webapp:The web application~  
~stid:home/pagelist:List of Pages~
