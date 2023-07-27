# WS2812-Double-Ring-Clock
Details how to build and program the WS2812 clock with 2 rings with the use of a 3d printer, 2 ws2812 leds ring (60 led and 24 led) and an esp8266 (like Wemos D1)

## End result
The end result should look something like this.  
![Working clock](https://github.com/Roukie686868/WS2812-Double-Ring-Clock/blob/main/Pictures/60Led%20Clock%20(Custom).jpg)  
The outer ring shows the minutes with a RED dot and the seconds with a GREEN dot, when the minute and second overlap, the color is purple for that one second. The inner ring shows the hours with a blue dot. As there are 24 dots in the inner ring is showing half hours as well.

## Parts
The 60-led ring will be used the display the minutes and seconds on the outer ring. Typically these are WS2812B Neo-Pixel placed in a ring  
![60-Led RIng](https://github.com/Roukie686868/WS2812-Double-Ring-Clock/blob/main/Pictures/WS2812%20Ring60%20small.png)  
The hours and half hours will be displayed on the 24-led inner ring  
![24-Led RIng](https://github.com/Roukie686868/WS2812-Double-Ring-Clock/blob/main/Pictures/WS2812%20Ring24%20small.png)  
LDR (Photo resistor 10M) and a normal resistor (10k) to pull it to ground and create a voltage divider. (More explanation on that on this page from [Hackster.io](https://www.hackster.io/najad/ldr-with-arduino-51d709)  
![LDR and resistor](https://github.com/Roukie686868/WS2812-Double-Ring-Clock/blob/main/Pictures/LDR_Resitor.png)  
WEMOS D1 Mini (but any other ESP8255 or ESP32 will do)
![Wemos D1 Mini](https://github.com/Roukie686868/WS2812-Double-Ring-Clock/blob/main/Pictures/WemosD1MiniFrontandBack%20(Custom).png)  

## 3D print
The project consist out of the ring that holds the led and shows the hours and a little rectangular box behind it to hold both the Wemos D1 mini and the photo resistor to tune the led brightness depending the brightness in the room.
Both STL files are in the [Design folder](https://github.com/Roukie686868/WS2812-Double-Ring-Clock/tree/main/DesignFiles)  
The 3D print is in 2 colors. The hour numbers are just a 3 layers higher than the rings so stop the print after the rings are completed and continue with a different color for the last 3 layers to make the number standout from the rest.

## Electric Diagram
As an example the diagram only shows one Neo-Pixel on the right.
![How is it all connected](https://github.com/Roukie686868/WS2812-Double-Ring-Clock/blob/main/Pictures/Breadboard_design%20(Custom).png)  

## Programming
Big thanks to Werner Rotschopf for his sketch explaining how to get the local time including Daylight Savings Time (DST). His great and simple sketch works best compared with the many complex solutions out on the internet. [Werner's Webpage](https://werner.rothschopf.net/202011_arduino_esp8266_ntp_en.htm)  
When you want to adjust ot another timezone than the Central European one visit the next link 
[Different Timezones](https://github.com/nayarsystems/posix_tz_db/blob/master/zones.csv)  
Add the new line to your sketch and remark out the Amsterdam line.
```
// Set the timezone details found on https://github.com/nayarsystems/posix_tz_db/blob/master/zones.csv
#define MY_TZ "CET-1CEST,M3.5.0,M10.5.0/3"  // Amsterdam  // Get your from the github page
//#define MY_TZ "WET0WEST,M3.5.0/1,M10.5.0"   // Lisbon
```

As the Neo-Pixels can be very bright an photo resistor was added to. You may have to play a bit with the map setting to get them right for your day and night situation
```
  light = analogRead(LDR);
  light = map(light, 0, 1024, 10 ,250);
```

Below the final programming
```javascript {.line-numbers}
/* Necessary Includes */
#include <ESP8266WiFi.h>            // we need wifi to get internet access
#include <time.h>                   // time() ctime()
#include <FastLED.h>                // Lib to control the Neo Pixels
#include "credentials.h"            // my credentials locked away in a different file

#ifndef STASSID
#define STASSID mySSID              // set your SSID with "xxxxxx" or store them in the credintials.h file
#define STAPSK  myPASSWORD          // set your wifi password with "xxxxxxx" or store them in the credintials.h file
#endif

// Configuration of NTP .
#define MY_NTP_SERVER "at.pool.ntp.org"           
// Set the timezone details found on https://github.com/nayarsystems/posix_tz_db/blob/master/zones.csv
#define MY_TZ "CET-1CEST,M3.5.0,M10.5.0/3"  // Amsterdam  // Get your from the github page
//#define MY_TZ "WET0WEST,M3.5.0/1,M10.5.0"   // Lisbon

// configure the clock
#define NUM_LEDS 84
#define LED_PIN 2
#define COLOR_ORDER GRB
CRGB leds[NUM_LEDS];
const int LDR = A0;
int light  = 100;  // Initial brightness

/* Globals */
time_t now;                         // this is the epoch
tm tm;                              // the structure tm holds time information in a more convenient way

void showTime() {               // only needed for testing
  time(&now);                       // read the current time
  localtime_r(&now, &tm);           // update the structure tm with the current time
  Serial.print("year:");
  Serial.print(tm.tm_year + 1900);  // years since 1900
  Serial.print("\tmonth:");
  Serial.print(tm.tm_mon + 1);      // January = 0 (!)
  Serial.print("\tday:");
  Serial.print(tm.tm_mday);         // day of month
  Serial.print("\thour:");
  Serial.print(tm.tm_hour);         // hours since midnight  0-23
  Serial.print("\tmin:");
  Serial.print(tm.tm_min);          // minutes after the hour  0-59
  Serial.print("\tsec:");
  Serial.print(tm.tm_sec);          // seconds after the minute  0-61*
  Serial.print("\twday");
  Serial.print(tm.tm_wday);         // days since Sunday 0-6
  if (tm.tm_isdst == 1)             // Daylight Saving Time flag
    Serial.print("\tDST");
  else
    Serial.print("\tstandard");
  Serial.println();
}

void showclock() {
  light = analogRead(LDR);
  light = map(light, 0, 1024, 10 ,250);
  FastLED.setBrightness(light);

    for (int pinNo = 0; pinNo <= NUM_LEDS - 1; pinNo++) {
    leds[pinNo] = CRGB( 0, 0, 0);
    if (pinNo  == tm.tm_sec + 24) {      leds[pinNo] = CRGB(   0, 254,   0);  }
    if (pinNo  == tm.tm_min + 24) {      leds[pinNo] = CRGB( 254,   0,   0);  }
    if (tm.tm_min == tm.tm_sec)   {      leds[tm.tm_sec + 24] = CRGB( 254 ,  0, 254); }
    
    if (tm.tm_hour > 11) {
      if (pinNo  == ( tm.tm_hour -12) *2 +(tm.tm_min/30)) {      leds[pinNo] = CRGB(   0,   0, 254);  }
      }
    if (tm.tm_hour <= 11){
      if (pinNo  == ( tm.tm_hour) *2     +(tm.tm_min/30)) {      leds[pinNo] = CRGB(   0,   0, 254);  }
      }
  }
  FastLED.show();

}
void setup() {
  Serial.begin(115200);
  Serial.println("\nNTP TZ DST - bare minimum");

  configTime(MY_TZ, MY_NTP_SERVER); // --> Here is the IMPORTANT ONE LINER needed in your sketch!

  // start network
  WiFi.persistent(false);
  WiFi.mode(WIFI_STA);
  WiFi.begin(STASSID, STAPSK);
  while (WiFi.status() != WL_CONNECTED) {
    delay(200);
    Serial.print ( "." );
  }
  Serial.println("\nWiFi connected");
  // by default, the NTP will be started after 60 secs

  // setup the led arrangment
  FastLED.addLeds<WS2812, LED_PIN, COLOR_ORDER>(leds, NUM_LEDS);
  FastLED.setBrightness(light);
}

void loop() {
  showTime();
  showclock();
  delay(1000); // dirty delay
}
```
