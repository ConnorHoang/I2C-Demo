Sorry. Here is the write-up: Required Implementation Components:
● GPIO’s configured fittingly
● SDA and SCL lines
● Start and stop conditions
● Device addressing
● Data bytes sent (minimum 2 bytes of data)
● R/W bit
● ACK/NACK’s

You don’t need to explain how the I2C write transaction works; we covered this in class and
it’s in a million sources. Moreso, explain your design choices for how you implemented this
bit banging, and why. Some possible topics:
● ACK/NACK bit handling
● Clock generation
● Polling vs. interrupts
● Code structure
● GPIO configuration
● etc.

ACK/NACK bit handling starts by reading in the data line (after the address and start bits). If an ACK is received, the system moves on as planned and starts transmitting data. If a NACK is received, the program terminates its current process and tries again. Once it fully transmits, it waits 5 seconds then begins sending again.

Clock generation was done artificially on the transmitter with delays for ease. The transmitter treats each delay as a half clock cycle while interrupts are used on the receiver. The interrupts are done so that instead of constantly polling to check for transmitted data, the receiver only does its work when it has data to work with. This is also helpful given both systems were on the same board for my testing in this demo.

As such, the transmitter works based on polling, but the receiver is using interrupts.

The code works around using the delay to simulate a clock, with the transmitter following a sequence of stages that advance with each clock transition and the following ACK/NACK as necessary. The logic goes in the standard I2C order, with the logic restarting to the top anytime it receives a NACK. A binary flag determines whether the sequence is for reading or writing.

In I2C, we need space to change the clock, and in between to either have a change while the clock is high, or change to set data while the clock is low. As such the delay time was chosen to be half a clock tick, as such allows for control at each of those stages.

GPIO pin assignment were fairly arbitrary. The two interrupt pins were at the necessary PA pins that supported EXTI, and the transmit pins were GPIO pins I already had wires in for a trivially easier time connecting the transmit and receiver pins. I made sure the chosen transmit pins had access to pull-up resistors.


Besides mixing my data and clock wires, my greatest challenge was failing to properly consider the different receivers I was using. As I originally planned on including the extra credit receiver, my initial testing utilized the receiver and was built with the assumption that communication to the receiver would function. When I plugged this version of the code into the logic analyzer, I found that my code seemed to be only putting alternative highs and lows as opposed to my desired bytes of data. It took me time to realize that as part of the full transceiver system, I had included a behavior that restarted the communication process if the transmitter ever received a NACK. As the logic analyzer never pulled down and thus always sent NACKs, my transmit code was just constantly restarting. Once I removed that line (and remembered the between-message time delay might require me to get a single line of data more than once) my code worked flawlessly.
