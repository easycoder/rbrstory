# System Architecture #

This project, called Room By Room Heating, is part of the [EasyCoder](https:/easycoder.github.io) family. It draws together a number of sub-repositories, each handling a different aspect of the overall project. Some of these are better documented than others, so it's the intention of this book to eventually provide a common reference for them all.

The specific subprojects are as follows:

 1. 
~sid:hardware:Room By Room~ is the product itself. It has a website at [https://rbrheating.com](https://rbrheating.com) but this is intended for use mainly by the system itself, as a means to store data and communicate between its various parts. The system comprises a controller PC running Python and a network of devices mostly based on the ESP32 microcontroller running Micropython. The repository is at [https://github.com/easycoder/rbr](https://github.com/easycoder/rbr).

 2. ~sid:EC-PY:EasyCoder for Python~ is a domain-specific programming language (a DSL) designed to ease the coding of projects such as this. The language is written in Python and comprises English-like commands to do general-purpose programming and to control the system hardware. The logic of the system controller is written using this language. Its repository is at [https://github.com/easycoder/easycoder-py](https://github.com/easycoder/easycoder-py).

 3.  ~sid:EC-JS:EasyCoder for JavaScript~ is a similar DSL written in JavaScript. It is designed to replace JavaScript in a browser for most of the more common situations. The Room By Room user interface, which runs on any computer or smartphone, is written entirely in EasyCoder, with no HTML, CSS or JavaScript visible. The EasyCoder website (also written in EasyCoder) includes a programmer's playground and tutorial called the Codex and is at [https://easycoder.github.io](https://easycoder.github.io). Its repository is at [https://github.com/easycoder/easycoder.github.io](https://github.com/easycoder/easycoder.github.io).

 4. ~sid:Storyteller:Storyteller~ is a tool for building documents such as this one. It extends MarkDown with some useful additions, making it easy to create complex multi-page text documents with illustrations. It has a website at [https://easycoder.github.io/storyteller](https://easycoder.github.io/storyteller).

~tid:pagelist:List of Pages~
