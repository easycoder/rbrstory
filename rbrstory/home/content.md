## ~tid:pagelist:List of Pages~ ##

## Introduction ##

This is the story of a project that started in 2000 with a simple idea - to automate my home central heating. I'd discovered some Internet-Of-Things devices made by a company called Shelly, and two of these devices interested me. One was a relay; the other a thermometer, and both of them had wifi so they could be accessed from any computing device on the local network. I figured I could use them to detect the temperature in each room of the house and control radiator valves to maintain the desired temperatures.

At the time I knew little to nothing about home automation. I looked around briefly to see what was out there, and of course Home Assistant came up at the top of the listings. After reading a few articles I developed a gut feeling that it wasn't what I wanted. The emphasis was always on playing with devices such as Hue light bulbs and having music follow me around the house, but these seemed be solutions to problems that didn't really exist. HA is aimed at system builders rather than programmers and required resources I didn't want to commit, such as a computer to run the UI on. My project was to be something I describe as a _distributed appliance_, for a target market that wasn't interested in playing with light bulbs or other technological gadgetry.

I've always had my own private projects, and they've always had one thing in common. I always pretend I'm being commissioned to create something for a real customer. Not just another programmer, but someone who wants to use a product without having to understand how it works. So basing the product on a general-purpose control platform was likely to be sub-optimal as well as no doubt requiring a lot of extra work to force it to behave the way I wanted. At least, that was how I saw it. The model I had in my mind was a good old-fashioned central heating programmer, but spiced up to handle any number of rooms independently.
~img:UI.jpg:right 25%~
Five years on, the project is working and two installations are in use, one in my house and the other in that of my co-worker on the project. Here's a photo of the user interface running on a smartphone and showing one room being heated, the others either off or above their target temperatures. Click the photo to magnify it; click again to shrink it back again.

Since we started, a number of commercial systems have come and gone, none of them having made it as mainstream products, so before describing our own I'll ponder why that might be, given that the cost of heating is an ever-present political issue and that a well-engineered system can save its own cost in a couple of years.

The problem, I believe, is that these systems are too complicated. Not for an engineer, but for ordinary people just wanting to heat their houses. A typical system comprises a number of devices, all of which have to be registered with a hub device via the home internet router. We immediately enter the world of IP addresses and wifi passwords, which is just too much for a large proportion of the potential market. It's like the days of the video recorder; most people knew how to start and stop a recording but few could handle programming the things.

This is why I described my system earlier as a _distributed appliance_. It's asking enough from a customer to connect an electric valve to each radiator, without loading them up with the chore of setting up network addresses and identities. It's just a step too far. To be successful, the system has to be preconfigured before delivery and require only a minimal amount of work on a single device to connect it up. None of the commercial systems present in that way and until they do so I fear the whole concept is a commercial non-starter.

On the next page I'll describe the system architecture, both physical and software, then the book will branch out in various directions.

~tid:architecture:System architecture~

