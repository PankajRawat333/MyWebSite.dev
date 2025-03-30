Title: Building an IoT Smart Bin with ESP32 and Azure

Description: Learn how to create a smart bin that automatically alerts the maintenance team when it's full or nearly full using ESP32, ultrasonic sensors, and Azure IoT services.
Lead: Smart Bin IoT Solution using ESP32 and Azure
Published: 09/08/2019
Image: /posts/images/iot-smart-bin.jpg
PrimaryTag: iot
Tags:
  - azure
  - iot
  - esp32
  - microcontroller
---

I recently purchased an ESP32S IoT microcontroller from Amazon to explore its capabilities and gain hands-on experience with microcontrollers and sensors. After spending a few days setting up basic samples—such as LED control, WiFi connectivity, and sending touch sensor data to the cloud using MicroPython—I decided to tackle a fun proof-of-concept (POC) project: building a **Smart Bin**.

The Smart Bin automatically alerts the maintenance team when it’s full or nearly full. To achieve this, I installed the following components inside the bin cover:

- **HC-SR04 Ultrasonic (US) Sensor**: Measures the distance to the bin’s contents.
- **ESP32S Microcontroller**: Processes sensor data and communicates with the cloud.

Using MicroPython, I programmed the ESP32S to read the sensor data every minute and calculate the free space inside the bin. The microcontroller connects to WiFi and sends this distance data to Azure IoT Hub. In Azure, I used Stream Analytics to monitor the data stream and generate alerts if the bin’s free space falls below a threshold (indicating it’s full or nearly full). These alerts are then sent to a queue, triggering an Azure Logic App that emails the maintenance team.

![Smart Bin Architecture](/posts/images/iot-smart-bin-diagram.png)

### How the Ultrasonic Sensor Works

The **HC-SR04 Ultrasonic Sensor** is a four-pin module (Vcc, Trigger, Echo, and Ground) widely used for measuring distance or detecting objects. It features two front-facing components: an ultrasonic transmitter and receiver. The sensor operates using the simple formula:

**Distance = Speed × Time**

The transmitter emits ultrasonic waves that travel through the air, reflect off any object (like the bin’s contents), and return to the receiver. The sensor measures the time it takes for the echo to return and outputs this duration via the Echo pin. Since the speed of sound is approximately 330 m/s at room temperature, the microcontroller uses this time to calculate the distance.

![Ultrasonic Sensor Operation](/posts/images/iot-smart-bin-sensor2.png)

For this project, I mounted the sensor inside the bin cover to measure the distance to the bin’s contents, as shown below:

![Smart Bin Setup](/posts/images/iot-smart-bin-setup.jpg)

I powered the setup using a mobile power bank for convenience. For a detailed circuit diagram and information on using a relay, refer to the [references](#references) section.

### References

- [GitHub Repository](https://github.com/PankajRawat333/SmartBin)
- [Circuit Diagram and Relay Details](https://www.bing.com/search?q=Circuit+diagram+and+relay+details+for+ecs32+and+ultrasonic+sensor&cvid=67d2ad7c489241f7aff06a881d23b23b&aqs=edge..69i57.10806j0j4&FORM=ANAB01&PC=U531)

Happy coding!