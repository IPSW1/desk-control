# desk-control
Take control of your height adjustable desk!

Several brands for height adjustable desks like Uplift and Ergotopia use components made by Jiecang Linear Motion. Among them is the [JCB35N2](https://en.jiecang.com/product/119.html) or a variant thereof as the control box, in combination with one of their control panels, for example the [JCHT35K9](https://en.jiecang.com/product/125.html).

Documentation on this system basically does not exist and thus custom controllers are not available. This obviously has to change.

**Note: The information in this project is made with the best of intentions. However, we are tinkering with expensive components. You are solely responsible for your stuff and when things break, it is your problem. Please don't try anything without double checking this information and knowing what you do.**


## JCB35N2 Control Box
The JCB35N2 control box is designed for standard desks with 2 motors, with soft-start/soft-stop and anti-collision protection. It has a power port _AC_, two motor ports _M1_ & _M2_ and two control ports _F_ & _HS_. (JCB35N[3,4] are very similar in their design, except for the number of motor ports and missing _F_ port; the _HS_ control port might be exactly the same, but this is not tested!)

The JCHT35K9 control panel has buttons to raise ▲ and lower ▼ the desk, four memory buttons _1_-_4_ to set the height to predefined positions and a setup button _M_, which is used for different settings. A three-digit 7-segment display shows the current height of the desk (probably with configurable units; the Ergotopia Desktopia Pro shows the height in cm).

### _HS_ Port
The _HS_ port is used by the control panels with support for memory buttons. It uses a standard 8 pin RJ45 connector with 5V logic. Pins are active low.

##### Pinout

```
				RJ45 Jack
				
      1   2   3   4   5   6   7   8
    +-+---+---+---+---+---+---+---+-+
    | |   |   |   |   |   |   |   | |
    | +   +   +   +   +   +   +   + |
    |                               |
    |                               |
    |                               |
    +--------+-------------+--------+
             |             |
             +-------------+
```

| Pin | Function                 |
| --- |:------------------------:|
| 1   | M button                 |
| 2   | Height output (UART)     |
| 3   | GND                      |
| 4   | ?                        |
| 5   | VCC (+5V)                |
| 6   | Memory buttons           |
| 7   | Raise + memory buttons   |
| 8   | Lower + memory buttons   |

The function of pin 7 and 8 depends on the input. They can be used alone to manually raise and lower the desk. If they are used in one of the combinations shown below, the respective memory slot is triggered.

Pulling pin 4 low just activates the display of the control panel. Might be the intended function, but it might not be?

##### Memory Buttons

| Button | Pin 6 | Pin 7 | Pin 8 |
| ------ |:-----:|:-----:|:-----:|
| 1      |       | x     | x     |
| 2      | x     |       |       |
| 3      | x     |       | x     |
| 4      | x     | x     |       |

While the box can report the current height, it seems like it is not used by the panel to set the height for the memory slots. The memory function is therefore entirely managed by the control box. Unfortunately, this also means that the soft-start/soft-stop feature is not controllable from the outside.

##### UART height output

The control box can report the current height for the control panel to show. It uses a one-directional UART connection with 9600 baud rate, 8 data, 1 stop, no parity.

* The control box starts a transmission whenever a button is pressed. It begins with two bytes, ``0x81`` and ``0xF5``.
* Data is constantly sent for 10 s with a delay of 15 ms between updates. One update consists of four bytes: the first two are always ``0x01``, the remaining encode the height in a 16-bit value. (This applies specifically to the Ergotopia Desktopia Pro, which shows the value in cm on the display. The value given by the control box is provided in mm.)
* Transmission ends with two bytes, ``0x01`` and ``0x05``.

This example shows one update. The last two bytes combined to a 16-bit value result in a height of 1093 mm, or 109.3 cm.

![](img/height_update.png)

### F Port
?


## Missing / ToDo
* Analyze _F_ port
* Find out all options (anti-collision sensitivity, min/max height, ...)
* Document error codes (missing motor, ...)