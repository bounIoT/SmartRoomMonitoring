# Smart Room Monitoring

## Description of the project

Smart Room Monitoring project source code. This code includes Blynk app connection for pushing and pulling data for suspicious activity detection using PIR and reed relay. Temperature and humidity sensors are also added to monitor them and also set the  desired levels of them by Blynk app. If out of range a notification is pushed. Sensor conditioning also implemented. Data is also pushed to Thingspeak cloud just to monitor. 

## Team members

-   Eray Eren
-   Hamit Başgöl

## Repository structure

Codes are given under the following folders. 

-   Code: Code for embedded processing and connectivity
-   Blynk: How we used Blynk to connect the device and smartphone (Cloud and UI).
-   Figures
    -   Screenshot(s) of UI
    -   Hardware Image
    -   Flow of data

We used Blynk for smartphone application and cloud connection. Blynk is used by drag and drop method, and works with components named Widget. For this reason, we did not use any independent code for UI and Cloud connection. Necessary explanations are given in the Blynk folder.

## Hardware setup

Components and boards that were used in this project:

-   NodeMCU
-   DHT-11: Temperature and Humidity Sensor
-   Reed Relay
-   PIR Sensor
-   Neodymium Magnet
-   5V AC/DC Adapter
-   11 Jumpers

Hardware connection:

![Hardware Connection](https://preview.ibb.co/iv4hqy/diagram.jpg)


## Flow of data

The data flow summary is given in the figure below. For further details regarding the data flow, please visit http://docdro.id/Ys8vEsS.

![Flowofdata](https://image.ibb.co/keEe5y/flowof.png)

## Development environment

-   Development Operating Systems are Windows 7 and Windows 10.
-   Development Tools are Arduino IDE, Blynk and Thingspeak
