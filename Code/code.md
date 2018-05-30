
``` c++
/***********************************************************************
* FILENAME :        sensors_blynk_2.ino             
*
* DESCRIPTION :
*        Smart Room Monitoring project source code. This code includes  
*      blynk app connection for pushing and pulling data for suspicious
*      activity detection using PIR and reed relay. Temperature and 
*      humidity sensors are also added to monitor them and also set the 
*      desired levels of them by Blynk app. If out of range a notification
*      is pushed. Sensor conditioning also implemented. Data is also pushed
*      to Thingspeak cloud just to monitor. 
*      
* FUNCTIONS :
*       void setup()
*       void loop()
*       unsigned char BLYNK_CONNECTED()
*       void BLYNK_WRITE(enum virtual pin) 
*       void BLYNK_READ(enum virtual pin) 
*       void dht_sensor_handler(void)
*       void door_sensor_handler(void)
*       void pir_sensor_handler(void)
*       
*       Copyright Smart Room Co. 2018,  All rights reserved.
* 
* AUTHORS :  Eray Eren       
*            Hamit Basgol
*  
*
*/

//*****************************INCLUDE AREA********************************//

// Here needed libraries are added :
// ESP8266 library for wifi module, BlynkSimple8266 for blynk functions of ESP8266
// Math and float libraries for isnan functions
// DHT library for for humidity & temperature sensor
#include <ESP8266WiFi.h>
#include <BlynkSimpleEsp8266.h>
#include <float.h>
#include <math.h>
#include <DHT.h>  // Including library for dht
//**************************END OF INCLUDE AREA****************************//

//******************************CONSTANTS**********************************//
#define BLYNK_PRINT Serial

//Pin declerations according to ESP8266-12E. 3 digital pins are used
#define DHTPIN 13   // DHT11 pin
#define reedpin 14  // Reed relay pin
#define PIRpin 12   // PIR sensor pins

//Device modes
#define away 1            
#define normal 2
#define concentration 3
//****************************END OF CONSTANTS*****************************//

//*******************************VARIABLES*********************************//
char auth[] = "50503b50e313419bbc2e32f712c0e86f";

char ssid[] = "iPhone";
char pass[] = "eray12345555";     

unsigned char optimum_temperature=22;
unsigned char optimum_humidity=50;
unsigned char smart_room_monitor_mode= normal;
unsigned char door_open_flag;
unsigned char door_state;
unsigned char humidity_flag;
unsigned char temperature_flag;
unsigned char pir_movement_flag;
unsigned char pir_state;
unsigned char noise;
float sensorData;
float sensorData2;

DHT dht(DHTPIN, DHT11);

BlynkTimer timer;
//****************************END OF VARIABLES*****************************//

//Initializations
void setup()
{
  pinMode(reedpin, INPUT);
  // Debug console
  Serial.begin(115200);

  // Blynk library functions for connecting ESP module to given access point
  Blynk.begin(auth, ssid, pass);  

  // Blynk has a beautiful OS logic so that it is used with timers.
  // Therefore, we make use of the timers.
  timer.setInterval(3000L, dht_sensor_handler);    //3000ms continous call
  timer.setInterval(1200L, door_sensor_handler);   //1200ms continous call
  timer.setInterval(1000L, pir_sensor_handler);    //1000ms continous call

}

void loop()
{ 
  Blynk.run();
  timer.run(); // BlynkTimer is working...

  //This part used for pulling sensor data to cloud by using virtual pin 11.
  //These data pushed to thingspeak by using webhook widget in the app.
  //Webhook widget is for 3rd party services. HTTP(s) request can be sent.
  //Webhook basicallly does the below code function:
  //  WiFiClient client;
  //  if (client.connect("api.thingspeak.com", 80)) {
  //    client.print("POST /update HTTP/1.1\n");
  //    client.print("Host: api.thingspeak.com\n");
  //    client.print("Connection: close\n");
  //    client.print("X-THINGSPEAKAPIKEY: " + apiKeyThingspeak1 + "\n");
  //    client.print("Content-Type: application/x-www-form-urlencoded\n");
  //    client.print("Content-Length: ");
  //    client.print(postStr.length());
  //    client.print("\n\n");
  //    client.print(postStr);
  //  }
  Blynk.virtualWrite(V11, sensorData, sensorData2, noise,  pir_movement_flag, door_open_flag, smart_room_monitor_mode);
}

/*
 * Function: BLYNK_CONNECTED
 * ----------------------------
 *   This function from blynk library. It is used for syncing offline user choices 
 *   from app when it gets connected to internet.
 *
 *   @params: none
 *   
 *
 *   returns: connection status 0(not connected)/1(connected) 
 */
BLYNK_CONNECTED() {
    //Optimal temperature, humidity configuration and mode selection is pushed from 
    //app to nodeMCU(the board). Therefore, they have to be synced.
    Blynk.syncVirtual(V0,V1,V4);
    BLYNK_WRITE(V0);
    BLYNK_WRITE(V1);
    BLYNK_WRITE(V4);
}

/*
 * Function: BLYNK_READ
 * ----------------------------
 *   This function from blynk library. It is used for sending data to cloud 
 *   It is not used because we encounter too much delay with this function.
 *
 *   @params: enum virtualpin V0
 *   
 *
 *   returns: none 
 */
//BLYNK_READ(V2) // Temperature and humidity data push to cloud
//{ 
//  
//}


/*
 * Function: BLYNK_WRITE
 * ----------------------------
 *   This function is used for sending mode selection from app to nodeMCU
 *   
 *
 *   @params: enum virtualpin V0
 *   
 *
 *   returns: none
 */
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

/*
 * Function: BLYNK_WRITE
 * ----------------------------
 *   This function is used for sending optimal temperature selection from app to nodeMCU
 *   
 *
 *   @params: enum virtualpin V1
 *   
 *
 *   returns: none
 */
BLYNK_WRITE(V1) // Temperature
{
  optimum_temperature = (unsigned char)param.asInt();
  Serial.println("Optimum Temperature Selection: ");
  Serial.print(optimum_temperature);
  Serial.print(" C");
  Serial.println();
}

/*
 * Function: BLYNK_WRITE
 * ----------------------------
 *   This function is used for sending optimal humidity selection from app to nodeMCU
 *   
 *
 *   @params: enum virtualpin V4
 *   
 *
 *   returns: none
 */
BLYNK_WRITE(V4) // Humidity
{
  optimum_humidity = (unsigned char)param.asInt();
  Serial.println("Optimum Humidity Selection: ");
  Serial.print(optimum_humidity);
  Serial.print("%");
  Serial.println();
}


/*
 * Function: dht_sensor_handler
 * ----------------------------
 *   This function is a handler function of dht11 temperature and humidity sensor.
 *   Sensor data is read here. Notifications are sent to app once if they are within 
 *   -+10% limits of given value from the app again. Notifications are sent once if they
 *   are in range again. 
 *
 *   @params: none
 *   
 *   Note: Notification are sent only in the concentration mode
 *
 *   returns: none
 */
void dht_sensor_handler(void)
{
  sensorData = dht.readHumidity();//reading the sensor data of humidity with the help of library
  if((!isnan(sensorData))){
    Blynk.virtualWrite(V2, sensorData); //sending humidity value to Blynk
  }

  //range check for humidity
  if( ((sensorData<(0.9*optimum_humidity)) || (sensorData>(1.1*optimum_humidity))) && (!isnan(sensorData))  ){ 
    if(!humidity_flag)
    {
      humidity_flag = 1;
      if(smart_room_monitor_mode == concentration ){
        Blynk.notify("Hey! Optimal Humidity is out of range " + String(sensorData) ); // notification sending
      }
    }
  }
  else 
  {
    if(humidity_flag && (!isnan(sensorData)) ){
      humidity_flag = 0;
      if(smart_room_monitor_mode == concentration ){
        Blynk.notify("Good! Optimal Humidity is in range " + String(sensorData) ); // notification sending
      }
    }
  }
  Serial.println("humidity loop: ");
  Serial.println(sensorData);

  sensorData2 = dht.readTemperature();//reading the temperature data with the help of DHT library
  if((!isnan(sensorData2))){
    Blynk.virtualWrite(V3, sensorData2); //sending temperature value to Blynk
  }

  //range check for temperature
  if( ((sensorData2<(0.9*optimum_temperature)) || (sensorData2>(1.1*optimum_temperature)))  && (!isnan(sensorData2)) ){
    if(!temperature_flag)
    {
      temperature_flag = 1;
      if(smart_room_monitor_mode == concentration ){
        Blynk.notify("Hey! Optimal Temperature is out of range " + String(sensorData2)); // notification sending
      }
    }
  }
  else 
  {
    if(temperature_flag && (!isnan(sensorData2)) ){
      temperature_flag = 0;
      if(smart_room_monitor_mode == concentration ){
        Blynk.notify("Good! Optimal Temperature is in range " + String(sensorData2) ); // notification sending
      }
    }
  }
  Serial.println("temperature loop: ");
  Serial.println(sensorData2);
  
}


/*
 * Function: door_sensor_handler
 * ----------------------------
 *   This function is a handler function of reed relay which used for door state.
 *   Reed sensor data is read here. Notifications are sent to app once if the
 *   door state(closed/open) chages. Email is sent when the door is opened.
 *   
 *
 *   @params: none
 *   
 *   Note: Notifications and email are sent only in the away mode. 
 *
 *   returns: none
 */
void door_sensor_handler(void)
{
  door_open_flag = digitalRead(reedpin); //reading reed sensor data 
  Serial.println("door open: ");
  Serial.print(door_open_flag);
  if(door_open_flag && (!isnan(door_open_flag))  ){
    Blynk.virtualWrite(V5, "Closed"); //sending door state to Blynk
    if(!door_state){
      door_state = 1;
      if(smart_room_monitor_mode == away ){
        Blynk.notify("Door Closed "); // notification sending
      }
    }
  }
  else{
    if(door_state && (!isnan(door_open_flag))){
      door_state = 0;
      Blynk.virtualWrite(V5, "Open"); //sending door state to Blynk
      if(smart_room_monitor_mode == away ){
        Blynk.notify("Suspicious! Door Opened "); // notification sending
    
    //Email sending - start
   
        Blynk.email("Smart Room Monitoring - Suspicious Activity Detected", "Warning! Suspicious activity detected. Door Opened!");
    
    //Email sending - finish
    
      }
    }
  }
  
}

/*
 * Function: pir_sensor_handler
 * ----------------------------
 *   This function is a handler function of pir sensor which is used for 
 *   movement monitor. Pir sensor data is read here. Notifications are sent 
 *   to app if there is a movement. Notifications are also sent when there is 
 *   no more movement. Email is sent when there is a movement.
 *   
 *
 *   @params: none
 *   
 *   Note: Notifications and email are sent only in the away mode. 
 *
 *   returns: none
 */
void pir_sensor_handler(void)
{
 
  pir_movement_flag = digitalRead(PIRpin); //reading pir sensor data 
  if(pir_movement_flag && (!isnan(pir_movement_flag))  ){
    Blynk.virtualWrite(V6, "Motion"); //sending pir sensor data to Blynk
    if(!pir_state){
      pir_state = 1;
      if(smart_room_monitor_mode == away ){
        Blynk.notify("Suspicious! Motion exists"); // notification sending
    
    //Email sending - start
 
        Blynk.email("Smart Room Monitoring - Suspicious Activity Detected", "Warning! Suspicious activity detected. Motion exists!");
    
    //Email sending - finish
      }
    }
    
  }
  else{
    if(pir_state &&(!isnan(pir_movement_flag))){
      Blynk.virtualWrite(V6, "No motion"); //sending pir sensor data to Blynk
      pir_state = 0;
      if(smart_room_monitor_mode == away ){
        //Blynk.notify("No motion "); // notification sending
      }
    }
  }
  Serial.println("PIR loop: ");
  Serial.println(pir_movement_flag);
  
}
