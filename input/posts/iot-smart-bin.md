Title: IoT Smart Bin
Description: Smart bin automatically sends an alert to the maintenance team when the bin is full or about to be full.
Lead: Smart Bin IoT Solution using ESP32 and Azure
Published: 09/08/2019
Image: /posts/images/iot-smart-bin.jpg
PrimaryTag: iot
Tags:
  - azure
  - iot
  - esp32
  - microcontroller
  - sensor
---

Recently, I bought ESP32S IoT microcontroller from Amazon to do some POC and get some insight from microcontroller and sensor. I spend 2-3 days to setup and run basic samples (LED on-off, connect WiFI, Send touch sensor data on Cloud etc.) using micro-python. Then I start work on **Smart Bin** POC (just for fun).

Smart bin automatically sends an alert to the maintenance team when the bin is full or about to be full. Inside the bin cover, I put the following device and sensor.

- HC-SR04 Ultrasonic (US) sensor
- ESP32S Microcontroller

I have written Microcontroller code in micro-python to read sensor data in every minute and calculate free space (distance) inside the bin. I used WiFi connectivity to send distance data in Azure IoT hub, where I used Azure Stream Analytics to analyze stream and generate alerts if data (distance value) breach threshold limit continuously (bin full or about to full). Once an alert comes into a queue, Azure Logic app would trigger and send an email to the maintenance team.

<img src="/posts/images/iot-smart-bin-diagram.png">


### How Ultrasonic Sensor Works

As shown in below image **HC-SR04 Ultrasonic (US) sensor** is a 4 pin module, whose pin names are Vcc, Trigger, Echo and Ground respectively. This sensor is a very popular sensor used in many applications where measuring distance or sensing objects are required. The module has two eyes like projects in the front which forms the Ultrasonic transmitter and Receiver. The sensor works with the simple high school formula that

<img src="/posts/images/iot-smart-bin-sensor.png">

**Distance = Speed x Time**

The Ultrasonic transmitter transmits an ultrasonic wave, this wave travels in air and when it gets objected by any material it gets reflected back toward the sensor this reflected wave is observed by the Ultrasonic receiver module as shown in the picture below

<img src="/posts/images/iot-smart-bin-sensor2.png">

Now, to calculate the distance using the above formulae, we should know the Speed and time. Since we are using the Ultrasonic wave we know the universal speed of US wave at room conditions which is 330m/s. The circuitry inbuilt on the module will calculate the time taken for the US wave to come back and turns on the echo pin high for that same particular amount of time, this way we can also know the time taken. Now simply calculate the distance using a microcontroller or microprocessor.

<img src="/posts/images/iot-smart-bin-setup.jpg">

My circuit diagram, Check below reference link for clear circuit diagram and purpose of relay. for now, I used mobile-power-bank to supply power.



### References

[Github](https://github.com/PankajRawat333/SmartBin)

[Circuit diagram and relay details](https://www.bing.com/search?q=Circuit+diagram+and+relay+details+for+ecs32+and+ultrasonic+sensor&cvid=67d2ad7c489241f7aff06a881d23b23b&aqs=edge..69i57.10806j0j4&FORM=ANAB01&PC=U531)


Happy coding.