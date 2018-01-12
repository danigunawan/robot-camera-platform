**What is this?**

  This is the versatile robot platform. I've gave it to possible usecases: a surveillence bot
and a object tracker but there can be more, i'll leave this to your imagination :)


1. Surveillence robot

2. Object follower

3. Building the robot



**1. The first usecase is a surveillence robot that is controlled using an android interface:**

A video demo is available on [youtube](https://youtu.be/6FrEs4C9D-Y)

The robot will stream the video using [UV4l](http://www.linux-projects.org/uv4l/)

The python server will receive commands using mqtt from the android application, and will transmit 
distance in front and behind the robot.

The android application is located in this [repository](https://github.com/danionescu0/android-robot-camera)


**How does it work**

a. The android app shows the uv4l streamming inside a webview.

b. Using controlls inside the android app lights and engines commands are issued to the MQTT server

c. The python server inside the docker container on the raspberry pi listens to MQTT commands and passes them
using serial interface to the arduino board. The arduino board controlls the motors and the lights.

d. The arduino board senses distances in front and back of the robot and sends the data through the serial interface to the 
python server, the python forwards them to the MQTT and they get picked up by the android interface and shown to the user

![ifttt.png](https://github.com/danionescu0/robot-camera-platform/blob/master/resources/diagram1.png)

**Why does an intermediary arduino layer has to exist and not directly the Pi ?**

* it's more modular, you can reuse the arduino robot in another project without the PI
* for safety, it's cheaper to replace a 3$ arduino pro mini than to replace a Pi (35$)
* an arduino it's not intrerupted by the operating system like the pi is, so it's more 
efficient to implement PWM controlls for the mottors, polling the front and back sensors
a few times per second
* if an error might occur in the python script the robot might run forever draining the
batteries and probably damaging it or catching fire if not supervised, in an arduino sketch
a safeguard it's more reliable because it does not depends on an operating system

**ToDo**

* Implement a battery status updater, maby my monitoring the power consumption for the py and arduino.
By knowing the full power of the battery pack an power estimation would be possible.
Email alerts and system shutdown should be in place when power is critical.
* Movement detection with email notification
* Implement a safeguard inside the arduino sketch, the motors should stop if proximity is detected


**Installation**


*Install Uv4l streamming:*
 
````
chmod +x uv4l/install.sh
chmod +x uv4l/start.sh
sh ./uv4l/install.sh 
````
A complete tutorial about uv4l is found here: https://www.instructables.com/id/Raspberry-Pi-Video-Streaming/?ALLSTEPS



*Clone the project in the home folder:*
````
git clone https://github.com/danionescu0/robot-camera-platform
````
The folder location it's important because in docker-compose.yml the location is hardcoded as: /home/pi/robot-camera-platform:/root/debug
If you need to change the location, please change the value in docker-compose too


*Uv4l configuration:*

* by editing uv4l/start.sh you can configure the following aspects of the video streaming: password,
port, framerate, with, height, rotation and some other minor aspects
* edit config.py and replace password with your own password that you've set on the mosquitto server
* optional you can change the baud rate (default 9600) and don't forget to edit that on the arduino-sketch too


*Install docker and docker-compose*

About docker installation: https://www.raspberrypi.org/blog/docker-comes-to-raspberry-pi/
About docker-compose installation: https://www.berthon.eu/2017/getting-docker-compose-on-raspberry-pi-arm-the-easy-way/

*Test uv4l installation*

a. Start it:
````
sh ./uv4l/start.sh 
````
b.test it in the browser

c. Stop it
````
sudo pkill uv4l
````


**Auto starting services on reboot/startup**

a. Copy the files from systemctl folder in systemctl folder to /etc/systemd/system/

b. Enable services:
````
sudo systemctl enable robot-camera.service
sudo systemctl enable robot-camera-video.service
````

c. Reboot

d. Optional, check status:
````
sudo systemctl status robot-camera.service
sudo systemctl status robot-camera-video.service
````



**2. The second usecase is a object following robot**
The robot will follow an object of a specific color (must be unique from background).

A demo video is available on [youtube](https://youtu.be/z9qLmHRMCZY)

In config_navigation.py you'll find:
````
hsv_bounds = (
    (24, 86, 6),
    (77, 255, 255)
)
object_size_threshold = (10, 100)
````

HSV means hue saturation value, and for our color object detection to work it has a lower and
an upper bound, our object color will have to be in this range to be detected.
[Here](https://github.com/jrosebr1/imutils/blob/master/bin/range-detector) you can find a 
visual HSV object threshold detector.

Object size threshold means the smallest and the highest object radius size 
(in percents from width) which will be considered a detection.

Running the object tracking script in vnc graphical interface in a terminal:

```` python3 object_tracking.py --show-video ````

This will enable you to view the video, with a circle drawn over it. The cirle means 
that the object has been detected.

Running the object tracking script with no video output:

```` python3 object_tracking.py ````


**3. Building the robot**
First a bit about the hardware. The arduino sketch can be found in arduino-sketck folder.


** Components **

Fritzing schematic:

![fritzig_sketch.png](https://github.com/danionescu0/robot-camera-platform/blob/master/arduino-sketch/sketch_small.png)

Checklist: 

* A small robot car with two ac motors with gearboxes

* Arduino pro mini

* Led light

* Small NPN tranzistor and 1 k rezistor

* L298N Dual H-Bridge

* Two power supplies on for the motors one for the raspberry pi and arduino

* RaspberryPi 3

* Two analog infrared distance sensors 


Pinout:

Led flashlight: D3

Left motor: PWM (D5), EN1, EN2(A4, A5)

Right motor: PWM (D6), EN1, EN2(A3, A2)

Infrared sensors: Front (A0), Back(A1)

Tx: D11, Rx: D10
