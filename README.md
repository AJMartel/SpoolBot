![spoolbot](https://bytebucket.org/pcotret/spoolbot/raw/04d60773e57fd10401488f2545f4207d40ab95d0/img/spoolbot.png?token=046a49eb15fa61cce9db54ad692f7e915b377428)
# How to create a filament counter for a 3D printer for a few bucks
Vladimir Béraud-Peigné, Eric Gastineau, Marie Xérès

CentraleSupélec, Rennes campus, 2016/2017

This project has been realized by Vladimir Béraud-Peigné, Eric Gastineau and Marie Xérès (1st year students 16/17 at CentraleSupélec, Rennes campus) for a short-term project under the supervision of Dr. Pascal Cotret.

## Goals of this project
![k8400](https://bytebucket.org/pcotret/spoolbot/raw/04d60773e57fd10401488f2545f4207d40ab95d0/img/k8400.jpg?token=cb0ebb0afd27bc7b90f6a25c14342b59efdf03bc)
Well, the basic idea of this project came from a little problem. We have a small fablab room with a Velleman K8400 3D printer. Unfortunately, this printer does not have a filament counter:

* We cannot know exactly how much filament has been used (in terms of length).
* As a consequence, we do not how much filament remains on the spool.

By browsing the Internet, we found a cool solution: https://www.hackster.io/binsun148/smart-3d-printer-filament-counter-filamentbot-383ac8

* It is cheap (based on a good old PS/2 mouse).
* It is quite well documented.

However, as this is a student project, the idea was not only to follow a tutorial but to understand how such a project can be achieved. This tutorial will show how we built our own SpoolBot!

## How does a mechanical mouse work?
![souris](https://bytebucket.org/pcotret/spoolbot/raw/04d60773e57fd10401488f2545f4207d40ab95d0/img/mymouse.png?token=f038ebd3f1ab0a9ddc57c8e8fb63569173a4795b)

A good old computer mouse is based on a ball causing the rotation of a small cylinder.This cylinder has an encoder wheel (drilled with several holes) at one of its ends. This wheel is placed between a phototransistor and a LED: counting pulses received by the phototransistor can be used to measure the distance traveled by the mouse ball.

Detecting the ball direction is a bit more tricky... In fact, there are two photo transistors slightly shifted. When we get through a hole of the wheel encoder, we get two detections (one for each transistor). Depending on which one comes first, we can guess the direction of the mouse.

## PS2 protocol

A computer mouse is a device giving ```[X,Y]``` coordinates of a movement and information about ```[left/right]``` clicks. Such information are transmitted through a synchronous serial link with a 9600bd/s baudrate: each frame is based on a classic ```8p1``` layout (8 data bits, 1 parity bit and 1 stop bit).

By default, a PS/2 computer mouse works with a 200 CPI (*Counts Per Inch*) resolution leading to 8 counts/mm (step is 125µm). Movements are coded on 8 bits for a distance of 3.2cm. The mouse is able to transmit up to 40 movements per second which allows to encode movements up to 2m/s (we think that’s quite enough, even for a hardcore gamer!).

Transmitted data is based on 8-bit words (see diagram below), with a start bit (set to 0) as a preamble. Data words are ended by a parity bit and a stop bit (set to 1). The whole message is coded on 11 bits.

![ps2](https://bytebucket.org/pcotret/spoolbot/raw/04d60773e57fd10401488f2545f4207d40ab95d0/img/timing1.png?token=15422884ee736062b049304e7fd98ff4ad30874f)

The mouse transmits information in 3 frames spaced by 350 µs: frames are 3.6ms long and are spaced by 6.4ms at least. As a consequence, the mouse is able to transmit up to 100 frames per second.

![full](https://bytebucket.org/pcotret/spoolbot/raw/04d60773e57fd10401488f2545f4207d40ab95d0/img/ps2_frame.png?token=a9714e559995c5c2ab0a7148a105ee14363fe64f)

The first frame word contains the following information:

* ```L```: left button (active at 1).
* ```R```: right button (active at 1).
* ```Xs```: direction of the horizontalmoving (1 for left, 0 for right).
* ```Ys```: direction of the verticalmoving (1 for low, 0 for high).
* ```Xv```: overflow in X.
* ```Yv```: overflow in Y.

The two other words transmit the moving value:
* ```X[0..7]```: moving along the horizontal axis (positive to the right, negative to the left).
* ```Y[0..7]```: moving along the vertical axis (positive to the bottom, negative to the top).

## PS/2 decoding on Arduino

We were quite lucky, some nice people have already implemented a PS/2 library: https://playground.arduino.cc/ComponentLib/Ps2mouse
The ```mouse.txt``` file contains an Arduino sketch helping us to create a sketch suiting our needs. Let’s focus on the ```mouse_read()``` method:

```C++
char mouse_read(void)
{
  char data=0x00;
  int i;
  char bit=0x01;
  // Set clk and data lines to high
  gohi(MCLK);
  gohi(MDATA);
  // Wait for the first clock cycle
  delayMicroseconds(50);
  while(digitalRead(MCLK)==HIGH);
  delayMicroseconds(5);
  while(digitalRead(MCLK)==LOW);
  // For each clock rising edge, get the data bit
  for(i=0;i<8;i++)
  {
	while(digitalRead(MCLK)==HIGH);
    if(digitalRead(MDATA)==HIGH)
	{
	  data=data|bit;
    }
    while(digitalRead(MCLK)==LOW);
    bit=bit<<1;
  }
  // Wait for two clock cycles (parity and stop bits)
  while(digitalRead(MCLK)==HIGH);
  while(digitalRead(MCLK)==LOW);
  while(digitalRead(MCLK)==HIGH);
  while(digitalRead(MCLK)==LOW);
  // Set clock line to low
  golo(MCLK);
  return data;
}
```

## SpoolBot solution

![full](https://bytebucket.org/pcotret/spoolbot/raw/04d60773e57fd10401488f2545f4207d40ab95d0/img/full_stuff.png?token=6c9982fc5e9d7b6f61caebfe15056ba3ca9ff6fd)


### Arduino code

See xxx

### Wiring diagram

You can also find it at xxx
![fritzing](https://bytebucket.org/pcotret/spoolbot/raw/04d60773e57fd10401488f2545f4207d40ab95d0/img/wiring.png?token=22c62f1425aebe1bae85f0dbd82a0239b2da4b50)

### 3D-printed case

To be completed, we need to fix a few things on 3D files.
![solidworks](https://bytebucket.org/pcotret/spoolbot/raw/04d60773e57fd10401488f2545f4207d40ab95d0/img/boitier.png?token=11cecede539c40f8cf6ec599dee3b1b412942ce1)