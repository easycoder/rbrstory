# Room By Room software #

The system software uses three mainstream programming languages and two custom languages that are closely related to each other. This is not done for the sake of being perverse but in acknowledgement of the simple fact that no single language can do everything equally well.

For each part of the system, the choice of language was in part forced on us. A program that is to run in a browser pretty well (at that time at least) had to be JavaScript, and one to run on an ESP32 either C++ or Micropython. A second constraint was self-imposed; that whatever was used it had to be as easy as possible to maintain, as the original coder would quite likely not be around to pick up the project years later. So for the ESP32, C++ was rejected in favour of Micropython as the latter is far easier to code.

A computer project of any size gets complex rather quickly. One way of dealing with this is to bring in a framework of some kind to force structure on the project. I'm not in favour of this because frameworks go out of fashion long before the languages in which they are written. So instead I used a tool that were already in my toolkit; a Domain-Specific Language (DSL). In this project there are two versions of this; one is for the system controller and the other is for the web application that runs on any smartphone. The first of these is written in standard Python, the other in an ES6 flavour of JavaScript. Both keep things as simple as possible. They are each described in the following pages.

~tid:controller:The controller software~

~tid:devices:The device software~

~tid:webserver:The web server~

~tid:webapp:The web application~

~tid:configurator:The configuration application~

~stid:home/pagelist:List of Pages~
