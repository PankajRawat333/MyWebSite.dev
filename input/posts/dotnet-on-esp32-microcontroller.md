Title: Running .Net on ESP32 Microcontroller
Description: Run .NET on microcontroller (ESP32) using nanoFramework.
Published: 06/03/2020
Image: /posts/images/nanoframework.jpg
PrimaryTag: iot
Tags:
  - iot
  - esp32
  - microcontroller
  - dotnet
  - nanoframework
---
**.Net Core** is a cross-platform, fast and lightweight but still, you cannot use it in tiny microcontroller where memory is few kilobytes. Most of the examples are available in C/C++ languages. I'm using .Net and Visual studio since my college time, I don't like to write code in C language, therefore I use MicroPython for embedded device.

**MicroPython** is a lean and efficient implementation of the Python 3 programming language that includes a small subset of the Python standard library and is optimised to run on microcontrollers and in constrained environments.

**ESP32** is a series of low-cost, low-power system on a chip microcontroller with integrated Wi-Fi and dual-mode Bluetooth which makes this my favorite microcontrller.

Thanks to **nanoFramework** team. nanoFramework is a free and open-source platform that enables the writing of managed code applications for constrained embedded devices. It is suitable for many types of projects including IoT sensors, wearables, academic proof of concept, robotics, hobbyist/makers creations or even complex industrial equipment.

### Blink LED using C# nanoFramework on ESP32
- Install and Setup Visual Studio 2017/19 and nanoFramework extension

<img src="/posts/images/nanoframework-extension.jpg">

- Create a new project based on the Blank Application (nanoFramework) template
- Install "nanoFramework.Windows.Devices.Gpio" nuget package.
- Added nanoFramework Preview Feed in Nuget Package manager and update all nuget packages (https://pkgs.dev.azure.com/nanoframework/feed/_packaging/sandbox/nuget/v3/index.json).

<img src="/posts/images/nanoframework-nuget.jpg">

- C# LED blink code

<img src="/posts/images/nanoframework-code.jpg">

- Install the nanoFramework firmware my machine. Open Command Prompt and run "dotnet tool install -g nanoFirmwareFlasher".
- Connect ESP32 on computer and run "nanoff --update --target ESP32_WROOM_32 --serialport COM16" command to flash your device (change COM port number)

<img src="/posts/images/nanoframework-deploy.jpg">

- Select ESP32 on Visual Studio Device Explore then Deploy project from Solution Explore.

<img src="/posts/images/nanoframework-buildoutput.jpg">

Once code deploy successfully, remove and reconnect ESP32. LED start blink. :)

![](/posts/images/nanoframework-led-blink.gif)

### Conclusion
If you are a .Net developer and you want to run C# on microcontroller, nanoFramework is the solution. Most of the core team members and contributors are embedded systems enthusiasts, passionate about coding. As of now nanoFramework community is not very big but they have put enough sample on GitHub to get started. I joined nanoFramework discord channel and got a very quick response when I was facing issue while firmware setup.

Happy coding.