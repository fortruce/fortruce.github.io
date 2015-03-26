---
title: "Talking to Arduino with Python"
---

Programming your Arduino with the provided Arduino IDE is pretty simple. Using the IDE's serial monitor to watch output from your Arduino is usually everyone's first step to communicating between their computer and Arduino, but that is only one way communication.

What if you want your Arduino to start taking commands from your computer? What about logging output from your Arduino's serial connection and taking some sort of action based on it?

These scenarios require a more robust communication between your Arduino and your computer. This is easy to accomplish using **Python** and many other languages, but I prefer the easy of Python for newer programmers and experienced alike.

##Finding Your Arduino

Using the IDE, it is easy to select among a drop-down list of potential ports for your Arduino. You can continue to use that list of ports to figure out what port has been assigned to your Arduino, or you can use one of the following methods.

Run the following command for your OS before you plug your Arduino in and after it's plugged in. The new entry in the list will be your Arduino's port.

###OS X

{% highlight bash %}
ls /dev/tty.usbserial*
{% endhighlight %}

###Linux

{% highlight bash %}
ls /dev/ttyUSB*
{% endhighlight %}

###Windows

Windows is a little less elegant.

*	Open the Device Manager
	*	`[Windows + r] devmgmt.msc [enter]`
*	Expand `Ports (COM & LPT)`

Compare the listing before and after plugging in your Arduino and see which `COM` port appears.

##Connect with Python

To connect with Python you must first install the correct version of `pyserial` for your Python version. Instructions for this can be found on the [`pyserial`](http://pyserial.sourceforge.net/pyserial.html#installation) page.

Once `pyserial` is installed, we can use it to connect to your Arduino. Let us assume that the code running on the Arduino is simply setting up the `Serial` line for communication at `9600` baud and then sending a `0x41` (A) every second. The code for your Arduino should look like so:

{% highlight c %}
void setup() {
	// setup Serial communication at 9600 baud
	Serial.begin(9600);
	delay(50);
}

void loop() {
	Serial.write(0x41);
	delay(1000);
}
{% endhighlight %}

Now the Python code to connect and read from the Arduino will be:

{% highlight python %}
import serial

# replace with your serial port as first arg
sp = serial.Serial('/dev/tty.usbserial-A603UFMJ', 9600)

while True:
    print sp.read()
{% endhighlight %}

This code should connect to the Arduino and continually read and print out the `0x41` (A) that the Arduino is sending over the serial port.

##Handshaking

This basic form of communication is useful but only minimally so. Once you start to send more sophisticated messages back and forth you need to know how much and when to read. Starting your communication with a simple handshake ensures that both sides are ready to communicate and are synchronized. If you are not already familiar with handshake protocols, I suggest you read up on the canonical example, [tcp/ip](http://en.wikipedia.org/wiki/Transmission_Control_Protocol#Connection_establishment).

Our handshake will be simple. We will assume that the Arduino is already connected and will not suffer from resetting. You could just as easily use that assumption that the Python script will start then you will give power to your Arduino, but I prefer not having to unhook/rehook in my Arduino ever time.

First, the Arduino side of the equation. All of the relevant code for the Arduino side of the handshake will take place in the `setup()` routine.

{% highlight c %}
void setup() {
    Serial.begin(9600);
    delay(50);
    while (!Serial.available()) {
      Serial.write(0x01);
      delay(300);
    }
    // read the byte that Python will send over
    Serial.read();
}
{% endhighlight %}

This will initial a `Serial` connection then constantly write `0x01` to the `Serial` port until it notices that someone else has written to the port. Finally, it read the byte that Python will send over to clear the buffer.

The Python code will do the opposite. It will start by waiting to hear from an Arduino board, then it will echo a response back to acknowledge the board.

{% highlight python %}
import serial
import struct

ser = serial.Serial(PORT, BAUD)

# reset the arduino
ser.setDTR(level=False)
time.sleep(0.5)
# ensure there is no stale data in the buffer
ser.flushInput()
ser.setDTR()
time.sleep(0.5)


print "waiting for arduino..."

# initial handshake w/ arduino
print 'handshake: ' + str(struct.unpack("B", ser.read())[0])
ser.flushInput()
ser.write('\x10')

print "connected to arduino..."
{% endhighlight %}

First, we initialize the `Serial` port. Then we send a `reset` to the Arduino by setting the `DTR` pin low and then high again. This is akin to pressing the reset button on the Arduino itself (Note: this might not work for you depending on your usb serial connection with your Arduino). The `ser.flushInput()` while the Arduino is being reset is to ensure that there isn't data from before the Arduino was reset in the buffer. Finally, we use a blocking `ser.read()` call to wait for the initial contact from an Arduino, then echo back a `0x10` to the Arduino.

##Conclusion

This should give you a good idea of how to connect your Arduino to a computer to accomplish tasks involving back and forth communication. Using the simple handshake to start your communication will make it possible to stay in synch with your Arduino and even conveniently reset it!

###Bonus:

I highly suggest using the Python `struct` module for easily defining a binary protocol to use when communication with your Arduino. This will make parsing your communications back and forth painless. For example, I want to send readings from an accelerometer to the computer. This is read from the accelerometer as three 16 bit integers, one for each axis. On the Arduino side I will simply send the data over the wire like so:

{% highlight c %}
byte vals[6];
// populate vals with data from accel
Serial.write(vals, 6);
{% endhighlight %}

Then to read the same data in Python I can simply unpack it.

{% highlight python %}
// unpack binary data composed of 3 16-bit signed ints
print struct.unpack("hhh", ser.read(6))
{% endhighlight %}

Viola!
