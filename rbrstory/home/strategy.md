# The comms strategy #

One of the key attributes of the RBR system, as outlined in the ~tid:overview:System overview~, is that it is easy to install. A complete system requires three main items in every room to be controlled:

 - Each radiator must be fitted with a thermostatic valve. These normally come with an actuator, and it's this part we replace. The valve itself has a spring-loaded pin that when depressed opens the valve to let water pass through. The thermostatic actuator sits on top and forces the pin down when the temperature is too low, letting it rise again when the temperature reaches the set point. Our electrical actuator does the same job; when mains power is applied it pushes the pin down and when the power is removed allows it to rise again.
 
 - Because the actuator is mains-powered it requires a source of power. We supply a small box to sit close to the radiator, which connects to the actuator and to a nearby power outlet. Inside the box is a relay controlled by a microcontroller that takes commands by wififrom the system controller.
 
 - The temperature in the room is measured by a thermometer module. This may be powered by a battery or from a power outlet, depending on the model. This item also communicates with the system controller to inform it of the temperature in the room.

## Installation ##
Installation involves adding each of these three items to every room being controlled by the system. If a radiator is already equipped with a thermostatic valve and there is a nearby power outlet the job is quite easy. However, the microcontroller and the thermometer have to be configured, identify themselves to the system controller. In most similar sytems this configuration work has to be done when the system is delivered, and frequently involves using a computer to connect to each device and add it to the house wifi network. This is a technical job; not difficult but not something we would ask a customer to undertake. So the RBR system has been designed in such a way that it can be preconfigured before the system is delivered, leaving only a minimal amount of work to be done on site.

## The device network ##
The relay modules achieve this by running ESP-Now, a lightweight networking protocol that does not connect directly to the house network. One of the devices is designated the "master", "hub" device; this does connects to the house router and from there to the system controller. All the other devices are designated "slaves"; these connect to the master using ESP-Now. The outcome of this is that the system can be preconfigured, using the router at the supplier's location, then on delivery the system controller and the master device are switched over to the house network. We have developed a configuration program, that runs on a special installation computer and which performs this task quickly and easily.

## Data entities ##
There are two main types of data held in the system. One is the "system map", which lists the rooms being controlled, the target and actual temperatures for each room, the timing cycle desired and so on. The other is configuration data for the relays and thermometers.

### The system map ###
The map holds one or more room lists, known as "profiles". These are completely independent of each other, so you could in principle have a completely different set of rooms named in two different profiles. This isn't something that's likely to be needed, but it's possible. A more likely use of profiles is to have a timing pattern that is different at weekends to the one that applies on Monday through Friday. This is such a common scenario that the system also implements a weekly calendar, which appears as a table that specifies which profile should apply on each day of the week.
~img:calendar.jpeg:center 40%~
The system map, held in the RBR database for each installation, is used by the system controller to decide when to turn on and off the radiators in the rooms covered by the current profile. Editing facilities are provided on the local and the remote user interfaces, intended for use by the occupants of the property in question. The system controller continually runs a script that examines the map, reads the current temperature data received from the network and sends commands to turn radiators on or off.

### Configuration data ###
A separate record in the database holds information about the physical hardware used to control radiators and read temperatures. This is set up during configuration and will rarely be changed after that. None of the information it holds is of any interest to the occupants of the property. The data is managed using our ~stid:software/configurator:configuration script~. While this script is running, the system controller is prevented from running, so the person doing the configuration has full control over all the devices.

The relationship between these two data records is quite simple. Every device on the ESP-Now network has a name - often that of the room it's in. The devices themselves have unique MAC addresses, so the configuration record links these two things. When the system controller wants to turn on a relay it looks up the MAC address from the device name and sends the request to the master device for onward transmission. This avoids the master having to maintain a list of slave devices; all it ever sees is a MAC address.
