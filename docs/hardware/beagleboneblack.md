---
layout: default
title: BeagleBone Black
---

The BeagleBoneBlack already has Node.js baked into it's OS, so some of these tips are optional.

<div class="doc-callout">
<em>Note:</em> we are soon deprecating Node v0.8 support - these instructions only apply to the
Debian versions of BeagleBoneBlack. <a href="http://beagleboard.org/latest-images">http://beagleboard.org/latest-images</a>
</div>


#### Upgrading Node.js (Optional)

You need Node.js v0.10.x which should be installed by default on the BBB so this step is optional.
To update Node.js on BBB - checkout the instructions halfway down this page [http://elinux.org/Beagleboard:BeagleBoneBlack_Debian](http://elinux.org/Beagleboard:BeagleBoneBlack_Debian)

In particular the lines about adding an updated repo to /etc/apt/sources.list

    sudo sh -c "echo 'deb [arch=armhf] http://repos.rcn-ee.net/debian wheezy main' >> /etc/apt/sources.list"
    sudo sh -c "echo '#deb-src [arch=armhf] http://repos.rcn-ee.net/debian wheezy main' >> /etc/apt/sources.list"

Then update the packages

    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get install npm --reinstall

#### Installing Node-RED

The easiest way to install Node-RED is to use node's package manager, npm:

    sudo npm install -g --unsafe-perm  node-red

_Note_: the reason for using the `--unsafe-perm` option is that when node-gyp tries
to recompile any native libraries it tries to do so as a "nobody" user and often
fails to get access to certain directories. This causes alarming warnings that look
like errors... but only sometimes are errors. Allowing node-gyp to run as root using
this flag avoids this - or rather shows up any real errors instead.

For alternative install options, see the [main installation instructions](../getting-started/installation.html#install-node-red).


#### BBB specific nodes

There are some BBB specific nodes now available in our [node-red-nodes project on Github](https://github.com/node-red/node-red-nodes/tree/master/hardware/BBB).

These give you direct access to the I/O pins in the simplest possible manner. The easiest way to install them is direct from npm

    cd ~/.node-red
    npm install node-red-node-beaglebone

#### Starting Node-RED

Due to the constrained memory available on the BBB, it is advisable to
run Node-RED with the `node-red-pi` command. For details and other options such as auto-starting
on boot, follow the [Running Node-RED](../getting-started/running.html) instructions.

#### Using the Editor

Once Node-RED is started, assuming you haven't changed the hostname, point a
browser to [http://beaglebone.local:1880](http://beaglebone.local:1880).

#### First Flow - Hello World

To run a "hello world" flow that toggles the USR2 and USR3 LEDs, copy the following flow
and paste it into the Import Nodes dialog (*Import From - Clipboard* in the
dropdown menu, or Ctrl-I). After clicking okay, click in the workspace to place
the new nodes.

    [{"id":"184087e6.e7bf78","type":"inject","name":"on","topic":"","payload":"1","repeat":"","once":false,"x":370,"y":188,"z":"345c8adc.cba376","wires":[["919e132f.6e61f"]]},{"id":"25e4d6c.fda1b2a","type":"inject","name":"off","topic":"","payload":"0","repeat":"","once":false,"x":370,"y":228,"z":"345c8adc.cba376","wires":[["919e132f.6e61f"]]},{"id":"6be2c4b9.941d3c","type":"bbb-discrete-out","pin":"USR2","inverting":false,"toggle":false,"defaultState":"0","name":"","x":613,"y":136,"z":"345c8adc.cba376","wires":[[]]},{"id":"919e132f.6e61f","type":"bbb-discrete-out","pin":"USR3","inverting":false,"toggle":false,"defaultState":"0","name":"","x":619,"y":193,"z":"345c8adc.cba376","wires":[[]]},{"id":"1cf2bd40.e30d43","type":"inject","name":"on","topic":"","payload":"1","repeat":"","once":false,"x":368,"y":102,"z":"345c8adc.cba376","wires":[["6be2c4b9.941d3c"]]},{"id":"3e4caa12.c1b356","type":"inject","name":"off","topic":"","payload":"0","repeat":"","once":false,"x":368,"y":142,"z":"345c8adc.cba376","wires":[["6be2c4b9.941d3c"]]}]

Click the deploy button and the flow should start running. The USR2 and USR3 LEDs
can be manually set on or off using the Inject node buttons.

#### Advanced functions

For experts, the `bonescript` module can be made available for use in Function nodes.

To do this, update `settings.js` to add the `bonescript` module to the
Function global context - to do this :

When you run node-red it will print the location of `settings.js` like

    [info] Settings file  : /usr/local/lib/node_modules/node-red/settings.js

Edit this `settings.js` file - you may need to be administrator or sudo to do this. And
there we need to uncomment the bonescript library line.

    functionGlobalContext: {
        // os:require('os'),
        bonescript:require('bonescript'),
        // jfive:require("johnny-five"),
        // j5board:require("johnny-five").Board({repl:false})
    },

The module is then available to any functions you write as `context.global.bonescript`.

An example flow that demonstrates this is below :

    [{"id":"3c3a39ec.c3c5c6","type":"inject","name":"on","topic":"","payload":"1","repeat":"","once":false,"x":226,"y":498,"z":"345c8adc.cba376","wires":[["6d418357.92be7c"]]},{"id":"f9ade3.ff06522","type":"inject","name":"off","topic":"","payload":"0","repeat":"","once":false,"x":226,"y":538,"z":"345c8adc.cba376","wires":[["6d418357.92be7c"]]},{"id":"919022c7.6e6fe","type":"inject","name":"tick","topic":"","payload":"","repeat":"1","once":false,"x":226,"y":438,"z":"345c8adc.cba376","wires":[["7783db44.887c24"]]},{"id":"ec2495b6.13db68","type":"debug","name":"","active":true,"x":666,"y":478,"z":"345c8adc.cba376","wires":[]},{"id":"7783db44.887c24","type":"function","name":"Toggle USR3 LED on input","func":"\nvar pin = \"USR3\"\nvar b = context.global.bonescript;\ncontext.state = context.state || b.LOW;\n\nb.pinMode(pin, b.OUTPUT);\n\n(context.state == b.LOW) ? context.state = b.HIGH : context.state = b.LOW;\nb.digitalWrite(pin, context.state);\n\nreturn msg;","outputs":1,"x":446,"y":458,"z":"345c8adc.cba376","wires":[["ec2495b6.13db68"]]},{"id":"6d418357.92be7c","type":"function","name":"Set USR2 LED on input","func":"\nvar pin = \"USR2\";\nvar b = context.global.bonescript;\n\nb.pinMode(pin, b.OUTPUT);\n\nvar level = (msg.payload === \"1\")?1:0;\nb.digitalWrite(pin, level);\n\nreturn msg;","outputs":1,"x":446,"y":518,"z":"345c8adc.cba376","wires":[["ec2495b6.13db68"]]}]
