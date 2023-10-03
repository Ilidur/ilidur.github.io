---
title: Projects blog
tags:
categories:
date: 2023-10-03
lastMod: 2023-10-03
---
# Focus stacking rig #projects

  + Brief: Reuse 3D printer components to learn how to make a basic mechatronic system

  + MVP: A one axis slide for photography

  + Requires:

    + Control board

    + alu frame

  + ## Progress

    + ## 2023-10-03

      + Goal for today was to set up the dev environment for the controller

        + The controller I'm using is the Seeed Xiao ESP32C3

        + The dev environment is a linux VM inside UTM VM environment. This posed a challenge in connecting the ESP32C3 because I had to pipe the USB. This was a setting in the Input tab of the VM config. Then I have to manually connect the USB after each load from the USB dropdown of the VM.

        + Getting examples to work was not straightforward. I was using the SSD1306 OLED which had an IIC address on the back selected with resistor pins, but the OLED had a completely different address of 3C.

        + Success!

        + ![IMG_20231003_233005496.jpg](/assets/img_20231003_233005496_1696372838944_0.jpg)

      + Next up tomorrow is to set up a wifi connection example

    + ### 2023-10-02

      + Disassembled TronXY S5 3d printer and scavenged parts
        
        + ![IMG_20231002_221639851.jpg](/assets/img_20231002_221639851_1696332184660_0.jpg)

      + Ordered some motor controllers

      + Plan for software/control

        + Use ESP32C3 to control both motor and camera via wifi.

        + Use Sony QX1 that I have to keep it light and trigger commands via wifi api from the rig controller.