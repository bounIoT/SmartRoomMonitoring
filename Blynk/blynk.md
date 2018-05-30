# How did we use Blynk in our project?

Blynk is a smartphone application providing a large number of functionalities for the Internet of Things applications. These functionalities are used by dragging and dropping some components, which are called Widgets. Widgets communicate with devices by Virtual Pins. Each widget taking data from a device or sending data to the device has one Virtual Pin. This Virtual Pin is written in the code as an argument of functions so that when the function is executed, the required activity is shown in the application.

The Widget Box is shown in the figure below. Widget Box includes widgets that are used to carry out spesific functionalities.

![Widget Box](https://image.ibb.co/c0Y6Hd/3.png)

We used Blynk for mainly three applications:
-   for sending data to the cloud of Blynk
-   for sending data to the Thingspeak
-   for seeing data coming from the device and for controlling variables in the device.

In this guide, we will explain how we used Blynk in our project. 

We used several widgets in this project, as follows:

-   Four Labels, for seeing the current mode of sensors
-   One Superchart, for seeing the values of humidity and temperature in a graphical representation
-   One Menu, for selecting different modes: Away, Normal and Concentration
-   One Tab, to differentiate Monitor and Settings areas in the smartphone application
-   Two Sliders, to select optimal humidity and temperature variables 
-   One Notification, to give users notification at required times
-   One Email, to send users emails at required times
-   One Webhook, to connect Thingspeak for sending data regarding sensors and current mode

Our application was designed as follows. The first picture shows monitoring functions, while the second shows settings.

![OurApplication](https://image.ibb.co/mY0Lcd/1.png)
![OurApplication2](https://image.ibb.co/fiCu3J/2.png)

## LABELS: 

The label widget in the widget box.

![Label](https://image.ibb.co/cSsoVy/4.png)

### Label 1 - Humidity

This label was used to show humidity level. The Virtual Pin was selected as V2. Reading Rate was determined as 30 sec. The humidity label is given in the figure below:

![Humidity](https://image.ibb.co/hmC0cd/5.png)

The properties of other labels are almost same.

```c++ 
Blynk.virtualWrite(V2, sensorData);
```

This function was used to transfer humidity value, which is named sensorData, to Blynk.

### Label 2 - Temperature

This label is used to show temperature. The Virtual Pin was selected as V3. Reading Rate was determined as 30 sec.

```c++ 
Blynk.virtualWrite(V3, sensorData2);
```

This function is used easily to transfer temperature value.

### Label 3 - Movement

This label is used to send the PIR sensor data to Blynk. The Virtual Pin was selected as V6. Reading Rate was determined as PUSH. It had two states in our application:

```c++ 
Blynk.virtualWrite(V6, "Motion");
```

```c++ 
Blynk.virtualWrite(V6, "No Motion");
```

### Label 4 - Door Open

This label is used to send the Reed Relay sensor data to Blynk. The Virtual Pin was selected as V5. Reading Rate was determined as PUSH. It had two states in our application:

```c++ 
Blynk.virtualWrite(V5, "Closed");
```
```c++ 
Blynk.virtualWrite(V5, "Open");
```

## SUPERCHART - Sensor values

Superchart widget in the widget box.

![Superchart](https://image.ibb.co/hheriJ/6.png)

It is used for seeing the values of humidity and temperature in a graphical representation. With this widgets, we can determine various Data Streams to be shown. We selected Humidity and Temperature to show. After selecting this, by a block being at the near of the name, one can select the Input (which is the place data will be taken from). We created two Data Streams of which virtual pins are V2 and V3.

Superchart settings

![Superchart2](https://image.ibb.co/bY5xOJ/7.png)

Adding data stream to a superchart.

![Superchart3](https://image.ibb.co/jb0xOJ/8.png)

## TABS

Tabs widget in the widget box.

![Tabwidgetbox](https://image.ibb.co/bPzfcd/9.png)

We created two tabs for our application, which are MONITOR and SETTINGS. Widgets up to now were shown in the MONITOR tab, whereas others that we will explain were shown in the SETTINGS tab. New tabs are created easily by "Add Tab" button.

Properties of tab widget.

![Tabproperties](https://image.ibb.co/gGefcd/10.png)

## MENU - mode selection

Menu widget in the widget box.

![Menu](https://image.ibb.co/gvyvAy/11.png)

This menu was used to select the current mode of the device in Blynk. The Virtual Pin was selected as V0. Three items were created as Away, Normal and Concentration. 

Properties of menu widget.

![Menuproperties](https://image.ibb.co/he2u3J/12.png)

For synchronization of modes between device and Blynk, the code shown below was used.

```c++ 
BLYNK_CONNECTED() {
    //Optimal temperature, humidity configuration and mode selection is pushed from 
    //app to nodeMCU(the board). Therefore, they have to be synced.
    Blynk.syncVirtual(V0,V1,V4);
    BLYNK_WRITE(V0);
    BLYNK_WRITE(V1);
    BLYNK_WRITE(V4);
} 
```

To send mode selection from Blynk to nodeMCU, we use the code given below:

```c++ 
BLYNK_WRITE(V0) // V0 refers to a LABEL
{
  switch (param.asInt())
  {
    case away: 
    { // Item 1
      smart_room_monitor_mode = away;
      Serial.println("Away selected");
      break;
    }
    case normal: 
    { // Item 2
      smart_room_monitor_mode = normal;
      Serial.println("Normal selected");
      break;
    }   
    case concentration: 
    { // Item 3
      smart_room_monitor_mode = concentration;
      Serial.println("Concentration selected");
      break;
    } 
  }
}
```

## SLIDERS

Sliders that we used in our application.

![Sliders](https://image.ibb.co/ebAgiJ/13.png)

### Slider 1 - Set Optimal Temperature

This slider was used to determine the optimal temperature value for concentration mode. The Virtual Pin was selected as V1. 

The properties of slider widget. Almost same process was carried out for the other slider.

![Sliders2](https://image.ibb.co/gAnGHd/14.png)

The code below was used to set optimum_temperature according to data taking from this slider. After determining the optimum_temperature value, we used it to check whether the current temperature is out of range or not.

```c++ 
BLYNK_WRITE(V1) // Temperature
{
  optimum_temperature = (unsigned char)param.asInt();
  Serial.println("Optimum Temperature Selection: ");
  Serial.print(optimum_temperature);
  Serial.print(" C");
  Serial.println();
}
```

### Slider 2 - Set Optimal Humidity

This slider was used to determine the optimal humidity value for concentration mode. The Virtual Pin was selected as V4. The code below was used to set optimum_humidity according to data taking from this slider. After determining the optimum_humidity value, we used it to check whether the current humidity level is out of range or not.

```c++ 
BLYNK_WRITE(V4) // Humidity
{
  optimum_humidity = (unsigned char)param.asInt();
  Serial.println("Optimum Humidity Selection: ");
  Serial.print(optimum_humidity);
  Serial.print("%");
  Serial.println();
}
```

## NOTIFICATION

Notification and email widgets in the widget box.

![Notificationemail](https://image.ibb.co/ehodVy/16.png)

It was used to notify user instantly when the hardware goes offline. Also, a sound for notification was selected.

The properties of notification widget.

![Notification](https://image.ibb.co/ihgAcd/15.png)

For other notifications such as when a movement is detected in the away mode or an out of range value is detected in the concentration mode, the function named Blynk.notify() was used. For example,


```c++ 
Blynk.notify("Hey! Optimal Humidity is out of range " + String(sensorData) );
```

## EMAIL

It was used to notify user via email when an intrusion is detected in the away mode. Intrusion conditions were selected as the movement and the state of the door. 

The properties of email widget.

![Email](https://image.ibb.co/eYUVcd/17.png)

The email which the notification sent was written in this widget. After running codes below, the email mentioned in the widget receives the email.

```c++ 
Blynk.email("Smart Room Monitoring - Suspicious Activity Detected", "Warning! Suspicious activity detected. Door Opened!");
```
```c++ 
Blynk.email("Smart Room Monitoring - Suspicious Activity Detected", "Warning! Suspicious activity detected. Motion exists!");
```

While the first string before the comma refers to the header of the email, the second string after the comma includes the body of the email. 

## WEBHOOK - Thingspeak

Webhook widget in the widget box.

![Webhook](https://image.ibb.co/kw6yVy/18.png)

It was used to transfer data coming from sensors and the current mode to Thingspeak. The virtual pin selected for this widget was V11. Each webhook needs an URL to connect required web service. For Thingspeak, we used

https://api.thingspeak.com/update?api_key=O5GZX27G2IUZZ00U&field1=/pin[0]/&field2=/pin[1]/&field3=/pin[2]/&field4=/pin[3]/&field5=/pin[4]/&field6=/pin[5]/

Propeties of the webhook widget:

![Webhook](https://image.ibb.co/ezvQAy/19.png)

For all fields in Thingspeak, one should refer to a pin. In the code, we wrote

```c++ 
Blynk.virtualWrite(V11, sensorData, sensorData2, noise,  pir_movement_flag, door_open_flag, smart_room_monitor_mode);
```

While V11 refers to the pin of the webhook, others refer to the values that will be pushed to Thingspeak.


# How did we use Thingspeak in our project?

Thingspeak is a cloud service developed for the Internet of Things applications. It provides an easy usage to show values taken from the device in a graphical representation. In our project, we used Thingspeak through Blynk. To do this, we used a widget name Webhook developed to provide an easy connection to a web service. However, there was something to be done before connecting Thingspeak with Blynk. We will explain it here.

Thingspeak runs with channels consisting of fields. Before sending data to Thingspeak, we created a channel named SmartRoomMonitoring. Each channel has two API Keys, one of them is to read and the other one is to write data. Since we wanted to write data to our channel, we used Write API Key stated in the section named API Keys.

Through Channel Settings section, we created five fields for our application:

-   Field 1 – Humidity
-   Field 2 – Temperature
-   Field 4 – Movement
-   Field 5 – Door Closed State
-   Field 6 – Mode

All fields were shown in a graphical representation in the section named Private View. To receive further detail regarding sending data to Blynk, please look at the explanation of Webhook in “How did we use Blynk in our project?”

Thingspeak

![Thingspeak](https://image.ibb.co/nkob0y/thingspeak.png)
