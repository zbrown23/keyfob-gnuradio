# keyfob-gnuradio
This project is an attempt to learn a little bit more about the protocol that drives my car's wireless key fob.
By using data published by the FCC as well as some open source tools, we can learn a decent amount about what's going on.

### Goals
* Recieve, demodulate, and interpret the signals from a car key fob
* Understand a little more about Honda/Acura's rolling code implementation
* (maybe) Identify potential vulnerabilities.

### Hardware
I am using the following pieces of hardware for this research:
* NooElec NESDR Mini 2+ (RTL-SDR)
* Generic UHF monopole antenna from Amazon
* My car key

### Software
* GNURadio - implementing the final decoding pipeline
* Universal Radio Hacker - reverse engineering the protocol
* SDR++ - roughly searching for the fob's signal
* Python - data processing, GNURadio extensions

## Part 1: Background
Modern cars have a few different features that can be accessed by the key fob.
Most can lock and unlock the car, open the trunk, trigger a panic alarm, and some can even remotely start the car.
A simple method of implementing this would be to use a fixed frequency that corresponds to a certain action,
(315.014MHz opens the trunk, etc.) but this is easily defeated, 
just transmit at the frequency that unlocks the car and you're in.
What if instead, we used a specific sequence of bits instead? That's all well and good, 
but unfortunately then all of our cars would be vulnerable to a so-called "replay" attack.
Even if every car had a unique set of bits that needed to be transmitted in order to unlock the car,
we could simply record the sequence for a certain car and "replay" it to get in.
So, how do we protect against that? The answer is to use a "rolling" code. 
In one version of this architecture, the transmitter and reciever share a cryptographically secure pseudo-random number generator (PRNG).
This PRNG is used to generate a cryptographic key which is then used to encrypt the command sent by the transmitter.
The reciever can then use its PRNG, with the same seed as the transmitter, to decrypt the recieved data and act upon it.
The key (haha) to this method's security is the fact that ***after every transmission, the key is regenerated and a used key is made invalid***.
This means that even if you play back a previously used "unlock" message, it will not work since you're reusing a discarded key.


## Part 2: Finding the signal
I started out by exploring the frequencies around 315MHz and trying to track down my car's key fob.
I quickly realized that this was probably going to be pretty difficult, and also discovered that there were a lot
of harmonics being kicked off by the key fob's transmitter as well. 

Eventually I remembered that the FCC actually keeps records of all of the little RF knick-knacks like my key,
*and* I remembered that all devices have to display their FCC ID. Sure enough, there it was, on the bottom of my key.
It's a little hard to read, but under a microscope I got it: [M3N5WY8145](https://fccid.io/M3N5WY8145). In the FCC filing was the exact frequency, 313.850 MHz, and sure enough when I tried it in SDR++ I saw the following:


<img src="docs/sdr_capture.png" width="600">

There's our data! We can see the real and imaginary parts of the signal above and below our center frequency,
and three distinct "packets" of communication. Let's move over to GNURadio.

## Part 3: Universal Radio Hacker
Now that I knew where my signal was, I needed to figure out how it was encoded and what datarate it was. I considered developing a tool in GNURadio to do this, but I 

## Part 4: GNURadio 
The first thing to do was to determine the kind of encoding that the signal used.