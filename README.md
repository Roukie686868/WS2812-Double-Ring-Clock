# WS2812-Double-Ring-Clock
Details how to build and program the WS2812 clock with 2 rings
with the use of a 3d pringer, ws2812 leds and an esp8266 (like Wemos D1)
![60-Led RIng](https://github.com/Roukie686868/WS2812-Double-Ring-Clock/blob/main/Pictures/WS2812%20Ring60.png)
![24-Led RIng](https://github.com/Roukie686868/WS2812-Double-Ring-Clock/blob/main/Pictures/WS2812%20Ring24.png)


```
#include <Arduino.h>
#include "credentials.h"

#include <ESP8266WiFi.h>
#include <NTPClient.h>
#include <WiFiUdp.h>

#include <FastLED.h>

const char* myssid =  mySSID;     // Replace mySSID for your "WifiNetworkName" make sure to use the " " around your text
const char* mypass =  myPASSWORD; // Replace myPASSWORD for your "WifiPasworde" make sure to use the " " around your text

WiFiUDP ntpUDP;  // Define NTP Client to get time
NTPClient timeClient(ntpUDP, "pool.ntp.org");

#define NUM_LEDS 84
#define LED_PIN 2
#define COLOR_ORDER GRB
CRGB leds[NUM_LEDS];
const int LDR = A0;
int light  = 100;  // Initial brightness
int lightm = 100;

void setup() {
  Serial.begin(115200);   // Initialize Serial Monitor
  Serial.print("Connecting to ");    // Connect to Wi-Fi
  Serial.println(myssid);
  WiFi.begin(myssid, mypass);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  
  timeClient.begin(); // Initialize a NTPClient to get time
  timeClient.setTimeOffset(+3600+3600); // Set offset time in seconds to adjust for your timezone, for example:  (summertime +3600) GMT +1 = 3600   // GMT +8 = 28800   // GMT -1 = -3600   // GMT 0 = 0

  FastLED.addLeds<WS2812, LED_PIN, COLOR_ORDER>(leds, NUM_LEDS);
  FastLED.setBrightness(light);
}

void loop() {
  timeClient.update();

  int currentSecond = timeClient.getSeconds();
  int currentMinute = timeClient.getMinutes();
  int currentHour   = timeClient.getHours();
  
  light = analogRead(LDR);
  lightm = map(light, 200, 1024, 10 ,250);
  FastLED.setBrightness(lightm);
  Serial.print(currentHour); Serial.print(":"); Serial.print(currentMinute); Serial.print(":"); Serial.print(currentSecond); Serial.print("   Light: "); Serial.print(light); Serial.print(" "); Serial.println(lightm);

  for (int pinNo = 0; pinNo <= NUM_LEDS - 1; pinNo++) {
    leds[pinNo] = CRGB( 0, 0, 0);
    if (pinNo  == currentSecond + 24    ) {      leds[pinNo] = CRGB(   0, 254,   0);  }
    if (pinNo  == currentMinute + 24    ) {      leds[pinNo] = CRGB( 254,   0,   0);  }
    if (currentMinute == currentSecond)   {      leds[currentSecond+24] = CRGB( 254 ,  0, 254); }
    
    if (currentHour > 11) {
      if (pinNo  == ( currentHour-12) *2 +(currentMinute/30)) {      leds[pinNo] = CRGB(   0,   0, 254);  }
      }
    if (currentHour <= 11){
      if (pinNo  == ( currentHour) *2     +(currentMinute/30)) {      leds[pinNo] = CRGB(   0,   0, 254);  }
      }
  }

  FastLED.show();
  delay(1000);
  }
```
