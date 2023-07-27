# WS2812-Double-Ring-Clock
Details how to build and program the WS2812 clock with 2 rings with the use of a 3d printer, 2 ws2812 leds ring (60 led and 24 led) and an esp8266 (like Wemos D1)

The 60-led ring will be used the display the minutes and seconds on the outer ring

![60-Led RIng](https://github.com/Roukie686868/WS2812-Double-Ring-Clock/blob/main/Pictures/WS2812%20Ring60%20small.png)

The hours and half hours will be displayed on the inner ring

![24-Led RIng](https://github.com/Roukie686868/WS2812-Double-Ring-Clock/blob/main/Pictures/WS2812%20Ring24%20small.png)

The end result should look something like this.

![Working clock](https://github.com/Roukie686868/WS2812-Double-Ring-Clock/blob/main/Pictures/60Led%20Clock%20(Custom).jpg)

The outer ring shows the minutes with a RED dot and the seconds with a GREEN dot, when the overlap, the color is purple for that one second. The inner ring shows the hours with a blue dot.
The rectangular box in the middle holds a WEMOS D1 mini and a photo resistor to tune the led brightness depending the brightness in the room. (tune this your self with ```lightm = map(light, 200, 1024, 10 ,250);``` )
By connecting to your Wifi the and pulling the internet time correct by ```timeClient.setTimeOffset(+3600+3600);``` for your time zone and daylight saving the time is shown.

The 3D print is in 2 colors. The numbers are just a 3 layers higher than the rings so stop the print after the rings are done and continue with a different color for the last 3 layers to make the number stand out from the rest.

Big thanks to Werner Rotschopf for his sketch to get the local time including the Daylight Savings Time (DST). His great and simple sketch works best compared with the many complex solutions out on the internet. It looks like he got this from somebody called by noiasca 2020-09-22

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

/* Configuration of NTP . Details found on https://github.com/nayarsystems/posix_tz_db/blob/master/zones.csv */
#define MY_NTP_SERVER "at.pool.ntp.org"           
#define MY_TZ "CET-1CEST,M3.5.0,M10.5.0/3"  // Amsterdam  // Get your from the github page
//#define MY_TZ "WET0WEST,M3.5.0/1,M10.5.0"   // Lisbon

// configure the clock
#define NUM_LEDS 84
#define LED_PIN 2
#define COLOR_ORDER GRB
CRGB leds[NUM_LEDS];
const int LDR = A0;
int light  = 100;  // Initial brightness
int lightm = 100;

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
  lightm = map(light, 0, 1024, 10 ,250);
  FastLED.setBrightness(lightm);

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
