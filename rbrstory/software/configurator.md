# The RBR-Now configurator #

Although RBR can handle a number of different types of hardware, the one that causes least problems with installation and maintenance is the RBR-Now device. This is an ESP32-based microcontroller driving either a relay or a thermometer (and other variants may appear in the future).

The most important feature of RBR (apart from those the user sees every day) is that it can be mostly configured before delivery. Most home automation products have to be configured during installation, which often means setting up each one to connect to the house router. This is time-consuming and inconvenient. Sometimes the router itself changes, such as when the owner changes network provider, and the entire system has to be reprogrammed.

The RBR-Now family of devices uses the ESP-Now networking protocol to run independently of the house system, can be set up before delivery and can be moved to another router with changes to only one item.

Although the system can be set up manually for each device, doing so provides little to no confirmation that everything is as it should be. To make things simpler we provide the "configurator; an EasyCoder script that runs on a separate computer on the network. The computer must be Linux as it requires features that are not available on Windows. It is possible that a Linux subsystem could be used as there are not many such features; the main one is to scan the local network to find wifi access points. We have no knowledge of using a Mac but something similar might apply.

Here's what the configurator looks like after starting up:
~img:RBRConf.png:center 50%~
On the left is a column of buttons and on the right the information about the system and its devices. The configurator can manage more than one system; some may be on the same router network and others not. The drop-down Systems list lets you choose which one to manage. If you are not currently connected to the network for that system you will not be able to view or alter anything.

Next to this list is a button to perform a scan of the local network, looking for system controllers. If it finds one that already exists in the list, no action is taken. You can also delete a system from the list. Most dangerous commands of this kind will ask for confirmation before going ahead.

In the bottom right-hand corner of the window is a status line that provides information about what the configurator is doing. Generally, green text implies it is ready to accept a command; blue text tells you what it is currently doing and red text warns you that something did not work. The configurator also disables buttons and text fields that cannot be used at any given time.

A newly-discovered system has no devices; the ones you see in the picture are for an existing system. On the left, _Scan for devices_ lets you add devices to the system, one by one. This is a complex procedure but most of the detail is hidden.

When an RBR-Now devices is installed into a product, the first thing is to flash it with Micropython, taking care to pick the version for the particular variant of ESP32. Currently they are all ESP32-C3. See the Micropython website for instructions on how to do this on the command line, or you may find Thonny is able to do it. We provide an EasyCoder script, `flash-device.ecs`, that will do the job for you if the files are all in the right places. If you have downloaded the RBR repository this should be the case. The tool requires adafruit-ampy to be installed:
```
pip install adafruit-ampy
```
This downloads and installs Micropython and all the RBR Python scripts.

Once this is done you can power up the device. If you run it on Thonny you will see messages output as configuration proceeds, but in most cases you can just keep an eye on the status line in the configurator. When the device starts it will wait indefinitely to be configured. While doing so it publishes an access point on `http://192.168.9.1`, with an SSID `RBR-Now-xxxxxxxxxxxx`, where `xxxxxxxxxxxx` is the MAC address of the device. The password is `00000000` and you can log into it from a browser if you wish. There's not much to see, though.

When you click _Scan for devices_ the configurator polls the local network, looking for anything starting with `RBR-Now` that it doesn't already know about. Your new device will appear after a while. Note that this action depends on internal Linux features, and owing to cacheing devices can fail to appear immediately or go on being visible after they have been disconnected. If nothing happens, click again and if it stubbornly refuses to appear after many clicks, close the configurator and restart it. Eventually a small window will pop up showing a list of all the devices found. There should only be one unless you powered up two devices, so to avoid confusion don't do that.

The first device found on a system is designated the master, so be sure you pick the one you want to do that job. It doesn't matter a lot which one it is, so pick one that is most central and has good wifi connections to all the others. Configuration of the master is a lot more complex than that of a slave, so you will see a good deal of activity. From time to time your computer will disconnect from the router and connect to the master device. Normally everything completes proerly but occasionally there are issues which cause a faiture mid-way. Take note that your computer may be connected to the master instead of the router, so if you appear to have lost the Internet go you your control panel and reselect the router before trying again.

Once the master is configured it restarts and powers up again doing its job. The configurator UI will show it in the _Master device_ field and its name, `(none)`, will show in the `Name` field of _Selected device_. You can rename it to whatever you choose, such as the name of the room it's in. You also need to provide pin numbers for the LED and the relay; these are usually 3 and 9 respectively. Some devices require inverted logic for either of these pins; the need will become obvious once the device is running. Now click _Update_, followed by `Reset`, and the device will restart with all the saved settings.

With the master done, it's time to do the slaves. These are much simpler. For each one, power it up, click _Scan for devices_, wait for the popup, then click the device. Once the job is done it will appear in the Slave list, as in the picture above. Give each one a name and set the pin numbers as before.

While the system is under control of the configurator, the system controller is blocked from making any changes. You can turn relays on and off from the configurator, then when you finally exit the program the system controller will take over again and resore them to their proper states.

## The configurator script ##
This is quite a large EasyCoder script, so I'll document it over the coming pages. We'll start at the top:
```
!   rbrconf.ecs

    script RBRConfigurator

    use graphics

!    debug compile

    window HostWindow
    window Window
    window Dialog
    layout MainPanel
    layout LeftPanel
    layout RightPanel
    layout DeviceHPanel
    layout DeviceButtonPanel
    layout OuterLayout
    layout Layout
    layout VLayout
    layout HLayout
    layout Panel
    group Group
    label Label
    label RelayStateLabel
    label StatusLabel
    label UptimeLabel
    label TemperatureLabel
    pushbutton ResetConfigButton
    pushbutton ScanDevicesButton
    pushbutton ClearSystemButton
    pushbutton SetMasterAddressButton
    pushbutton ReconfigureMasterButton
    pushbutton RemoveSlaveButton
    pushbutton UpdateFileButton
    pushbutton UpdateAllFilesButton
    pushbutton DeleteFileButton
    pushbutton ExitButton
    pushbutton ScanSystemsButton
    pushbutton RemoveSystemButton
    pushbutton MasterDeviceButton
    pushbutton RelayOffButton
    pushbutton RelayOnButton
    pushbutton UpdateWidgetDataButton
    pushbutton ResetDeviceButton
    pushbutton TemperatureButton
    pushbutton OKButton
    pushbutton CancelButton
    lineinput HostInput
    lineinput UserInput
    lineinput PasswordInput
    lineinput LineInput
    lineinput DeviceNameInput
    lineinput LEDPinInput
    lineinput RelayPinInput
    lineinput DHT22PinInput
    lineinput PathInput
    listbox ListBox
    listbox SlaveList
    combobox SystemsCombo
    checkbox LEDInvertCheckbox
    checkbox RelayInvertCheckbox
    messagebox MessageBox

!    debug step

    log `Starting`
    init graphics

    create MainPanel type QHBoxLayout

    ! Do the left-hand panel

    create LeftPanel type QVBoxLayout
    add LeftPanel to MainPanel

    add stretch to LeftPanel

    create ResetConfigButton text `Reset everything`
    on click ResetConfigButton go to ResetConfigFileClick
    add ResetConfigButton to LeftPanel

    create ClearSystemButton text `Clear the selected system`
    on click ClearSystemButton go to ClearSystemClick
    add ClearSystemButton to LeftPanel

    create SetMasterAddressButton text `Set the master IP address`
    on click SetMasterAddressButton go to SetMasterAddressClick
    add SetMasterAddressButton to LeftPanel

    create ReconfigureMasterButton text `Reconfigure the master device`
    on click ReconfigureMasterButton go to ReconfigureMasterClick
    add ReconfigureMasterButton to LeftPanel

    add stretch to LeftPanel

    create ScanDevicesButton text `Scan for devices`
    on click ScanDevicesButton go to ScanForDevicesClick
    add ScanDevicesButton to LeftPanel

    create UpdateFileButton text `Update one file`
    on click UpdateFileButton go to UpdateFileClick
    add UpdateFileButton to LeftPanel

    create UpdateAllFilesButton text `Update all files`
    on click UpdateAllFilesButton go to UpdateAllFilesClick
    add UpdateAllFilesButton to LeftPanel

    create DeleteFileButton text `Delete file`
    add DeleteFileButton to LeftPanel
    on click DeleteFileButton go to DeleteFileClick

    create RemoveSlaveButton text `Remove slave`
    add RemoveSlaveButton to LeftPanel
    on click RemoveSlaveButton go to RemoveSlaveClick

    add stretch to LeftPanel

    create ExitButton text `Exit`
    add ExitButton to LeftPanel
    on click ExitButton go to ExitClick

    ! Now do the right-hand panel

    create RightPanel type QVBoxLayout
    add stretch RightPanel to MainPanel

    ! Create the system name group
    create Group title `Systems`
    set the height of Group to 50
    add Group to RightPanel
    create Layout type QHBoxLayout
    add Layout to Group
    create SystemsCombo
    add stretch SystemsCombo to Layout
    create ScanSystemsButton text `System Scan`
    disable ScanSystemsButton
    on click ScanSystemsButton go to ScanSystemsClick
    add ScanSystemsButton to Layout
    create RemoveSystemButton text `Remove`
    disable RemoveSystemButton
    on click RemoveSystemButton go to RemoveSystemClick
    add RemoveSystemButton to Layout
    
    ! Create the Master Device group
    create Group title `Master device`
    set the height of Group to 50
    add Group to RightPanel
    create Layout type QHBoxLayout
    add Layout to Group
    create MasterDeviceButton text ``
    add MasterDeviceButton to Layout
    on click MasterDeviceButton go to MasterDeviceClick
    
    ! Create the Slave Devices group
    create Group title `Slave devices`
    set the height of Group to 150
    add Group to RightPanel
    create Layout type QHBoxLayout
    add Layout to Group
    create SlaveList
    add SlaveList to Layout
    on select SlaveList go to SelectSlaveClick

    ! Create the Selected Device group
    create Group title `Selected device`
    set the height of Group to 150
    add Group to RightPanel

    create DeviceHPanel type QHBoxLayout
    add DeviceHPanel to Group

    create OuterLayout type QVBoxLayout
    add OuterLayout to DeviceHPanel

    create Layout type QHBoxLayout
    add Layout to OuterLayout
    create Label text `Name:`
    add Label to Layout
    create DeviceNameInput size 40
    add DeviceNameInput to Layout
    create Label text `Uptime:`
    add Label to Layout
    create UptimeLabel text `0`
    add UptimeLabel to Layout
    add stretch to Layout
    
    create Layout type QHBoxLayout
    add Layout to OuterLayout
    create Label text `LED Pin:`
    add Label to Layout
    create LEDPinInput size 5
    add LEDPinInput to Layout
    create LEDInvertCheckbox text `Inverted`
    add LEDInvertCheckbox to Layout
    add stretch to Layout
    
    create Layout type QHBoxLayout
    add Layout to OuterLayout
    create Label text `Relay Pin:`
    add Label to Layout
    create RelayPinInput size 5
    add RelayPinInput to Layout
    create RelayInvertCheckbox text `Inverted`
    add RelayInvertCheckbox to Layout
    create RelayOffButton size 5 text `-`
    on click RelayOffButton go to RelayOffClick
    add RelayOffButton to Layout
    create RelayOnButton size 5 text `+`
    on click RelayOnButton go to RelayOnClick
    add RelayOnButton to Layout
    create RelayStateLabel size 5 text `???`
    add RelayStateLabel to Layout
    add stretch to Layout
    
    create Layout type QHBoxLayout
    add Layout to OuterLayout
    create Label text `DHT22 Pin:`
    add Label to Layout
    create DHT22PinInput size 5
    add DHT22PinInput to Layout
    add stretch to Layout
    
    create Layout type QHBoxLayout
    add Layout to OuterLayout
    create Label text `Path:`
    add Label to Layout
    create PathInput size 60
    set the width of PathInput to 250
    add PathInput to Layout
    add stretch to Layout
    
    create Layout type QHBoxLayout
    add Layout to OuterLayout
    create Label text `Temperature:`
    add Label to Layout
    create TemperatureLabel text ``
    add TemperatureLabel to Layout
    add stretch to Layout

    ! The 'Update' and 'Reset' ManageButtons
    create DeviceButtonPanel type QVBoxLayout
    add stretch to DeviceButtonPanel
    create UpdateWidgetDataButton text `Update`
    on click UpdateWidgetDataButton go to UpdateWidgetDataClick
    add UpdateWidgetDataButton to DeviceButtonPanel
    create ResetDeviceButton text `Reset`
    on click ResetDeviceButton go to ResetDeviceButtonClick
    add ResetDeviceButton to DeviceButtonPanel
    create TemperatureButton text `Get temperature`
    on click TemperatureButton go to TemperatureButtonClick
    add TemperatureButton to DeviceButtonPanel
    add stretch to DeviceButtonPanel
    add DeviceButtonPanel to DeviceHPanel

    add stretch to RightPanel

    create StatusLabel text `` align right
    add StatusLabel to RightPanel
    gosub to OK 

    create Window title `RBR-Now configurator` size 700 500
    set the layout of Window to MainPanel
    show Window

    start graphics
```
At the start, the script names itself (for debugging) and requests the graphics subsystem. It then lists all the variables used. EasyCoder is a typed language so you can see the variables groued into different types. EasyCode commands are all-lowercase and variables start with a capital letter; this is not mandatory but it makes programs easier to read.

The job of this part of the script is to set up the graphics environment. Most of it is quite self-evident, especially if you're familiar with Qt graphics, which power this part of EasyCoder. Evey visible component on the screen is named somewhere and many are given values to display. Some items are containers (layouts) and most of the others are items whose purpose is obvious from their types and names. Once this code has run, the UI will appear on the screen of the computer.

~tid:configurator02:The configurator, part 2~

~stid:home/pagelist:List of Pages~
