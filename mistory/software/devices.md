# Device Software #

Most of the devices in the system are powered by ESP32 microcontrollers, and this page describes the software. The system does allow the use of other types; this is covered elsewhere in the documentation.

All of the code is Micropython (MP), which is far easier to write than C++. There is of course a trade-off; the performance of MP code is well below that of C++, but this matters little in a system such as this, where things move at human speeds and nothing needs to respond in less than a second. The downside of using C++ is that it's very hard to get it right. Memory leaks occur regularly unless extreme care is taken in the coding, and some of the system libraries themselves are not guaranteed to be memory-safe. As well as being largely immune to this problem, MP performs perfectly well on such modest hardware as an ESP32.

The system comprises a standard bunch of Python scripts that are loaded into each of the devices. Together these provide all the capability required of the system. Where the functional requirements of one device differs from another, the behaviour of each is govered by a configuration file; a JSON script whose content is unique for each device.

The Python scripts are as follows:

~tid:ap:ap.py~ contains all the code for handling a network access point (hotspot).

~tid:blescan:blescan.py~ is a Bluetooth LE (BLE) scanner set up to detect nearby Mijia thermometers.

~tid:boot:boot.py~ is the file that will always be executed first thing on boot, after MP has started.

~tid:config:config.py~ is a central resource that handles the configuration data and has a list of functions available in the system. Any module that requires functions outside itself will call one here. This module is passed around the system on initialization.

~tid:dht22:dht22.py~ handles devices that have a DHT22 thermometer.

~tid:espcomms:espcomms.py~ handles networking, using ESP-Now to implement a chained system that extends the range of a wifi router.

~tid:files:files.py~ contains a set of useful file functions that supplement those supplied by the MP libraries.

~tid:handler:handler.py~ contains the logic to handle requests from the controller.

~tid:main:main.py~ is the main program. It is run by MP immediately after `boot.py`.

~tid:pin:pin.py~ has simple functions to handle ESP32 I/O pins.

~tid:server:server.py~ runs a small webserver to accept requests received by the access point.

~tid:sta:sta.py~ contains all the code for handling station mode (an HTTP client).
