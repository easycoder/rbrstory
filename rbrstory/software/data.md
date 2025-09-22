# RBR data storage #

The RBR system requires a good number of different items of data. Most of these are held in the RBR web server so they can be accessed by more than one control device. The ~stid:hardware/controller:system controller~ also holds essential data linking other items together. The best way to describe the various items is by showing how they are used.

## Database tables ##
Every system managed by RBR has a MAC address. This is a unique address that belongs to the system controller. Every message sent to the database server contains the MAC address of the sender, often plus a password. The first message sent after a reboot is always a `register` request. The server looks up the MAC in a `systems` table. If there is no such record the server creates a password and a new record for the system. It then returns the password to the caller, which saves it in its ramdisk.

The `systems` table, keyed by MAC as above, holds various items of data about the system:
 - the password
 - the IP address of the system controller on its home network
 - the timestamp of record creation
 - the timestamp of the last time the record was accessed
 - a JSON record containing configuration information for all the XR devices in the system
 - a JSON record comprising the system map (see below)
 - a JSON record containing the states of all the sensors in the system
 - a JSON record to hold a request from a remote controller

One of the unusual features of RBR is how a system can be controlled remotely from a smartphone. This is in itself not unique, but the remote user interface permits more than one system to be controlled, allowing the user to switch between them. Any number of systems can be controlled in this way. The systems are identified by their MAC addresses

The `users` table contains a record for every user of the remote interface. To gain access to the control features you must log in to the server by your email address and password. If these are not recognised, the system creates a new record following a dialog with the user. To save having to log in manually each time, your login details are stored by your phone browser as semi-permanent data that will remain until the browser app is removed or an OS update occurs. It's sensible to make a note of these details and store them somewhere that's truly permanent.

Another table, `managed`, holds information about who manages which system(s). Since a user may manage more than one system, the table has a record for each combination of MAC and email.

## Data in the system controller ##
Storage in the system controller is divided between that which is frequently updated and that which is not. The system originally used devices such as the Raspberry Pi as the system controller, and these use SD cards for storage. SD cards wear rapidly if data is constantly rewritten, so volatile data was moved to a ramdisk that is recreated every time the controller restarts. The practice was carried over to the current system, which uses an industrial PC with SSD storage.

The controller's "hard drive" holds all the program files, which are seldom changed. The ramdisk holds the system map, the MAC address and password, information about the current state of the sensors and various other temporary items.

As the main system control script runs, it uses data from the ramdisk to decide which radiators should be on and which should be off, and having sent the necessary commands to make it happen, informs the server of the current state of the system.

## Configuration data ##
Although the configurator can run on the system controller itself, this will rarely happen, not least because the device is unlikely to have a keyboard. Configuration will normally take place on a computer or laptop on the same network, which will use SSH to directly access files on the controller. The XR configurator is able to handle multiple installations, in which case it will hold configuration data for each of the systems. This data is read from the server, but a file at `{home}/.rbr.conf` is created by the configurator to hold the data for all the managed systems

~tid:devices:The device software~  
~stid:home/pagelist:List of Pages~
