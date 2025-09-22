# Relay modules #

~img:relay.jpg:right 25%~Each radiator in an RBR system is controlled by an electrically-operated relay that acccepts commands from the system controller to turn on or off the radiator. Various types of relay are possible but the standard one is our own XR device. Here is one mounted on the wall next to a radiator. Click/tap the picture to enlarge it.

The box houses an ESP32-C3 microcontroller attached to a mains relay. Power enters and goes out to the radiator via cable glands at the bottom. At the top is a green indicator that lights up when the radiator is on, and next to it is a 3-position switch, with positions as follows:

 1. The radiator is controlled by the microcontroller and relay
 1. The radiator is permanently OFF
 1. The radator is permanently ON

The latter 2 positions provide a manual override in case of system failures of one kind or another.

The XR relay communicates with the system controller using ESP-Now, a wifi networking protocol designed for ESP devices that operates independently of HTTP and other wifi traffic. A main benefit of using them is that the system can be preconfigured before delivery, reducing the amount of work to be done on the premises. They also provide information about their current state, signal levels and the time since last being reset, all of which is viewable remotely.

~sid:hardware:The Room By Room hardware~  
~stid:home/pagelist:List of Pages~
