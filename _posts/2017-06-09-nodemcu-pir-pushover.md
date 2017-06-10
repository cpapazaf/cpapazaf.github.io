---
layout: post
comments: true
title:  "The simplest, cheapest and easiest home security ever!"
date:   2017-06-09 19:00:02 +0100
categories: iot
tags: iot nodemcu
---

## Challenge
Send push notifications by just using [Pushover][pushover-web], [NodeMcu][nodemcu-web] and a [PIR][pir-web] sensor.

## Prerequisites
* Create an account on [Pushover][pushover-web]. The cost for that is pretty low concidering that you only need to buy their app!
* [NodeMcu][nodemcu-web]
* [PIR][pir-web] sensor
* [Arduino IDE][arduino-ide-web]

## Connecting everything



## Code
```python
#include <ESP8266WiFi.h>
#include <WiFiClientSecure.h>

const char* ssid = "****";
const char* password = "*****";

const char* remoteHost = "api.pushover.net";
String path = "/1/messages.json";
const int httpsPort = 443;
String accessToken = "*****";
String userKey = "*****";
const char* fingerprint = "1ED5B768BB25ADA3E09678A46848084F07E48DAB"; // check the certificate fingerprint from remoteHost

// The message to be sent. Doing the percent encoding myself since pushover requires it
String title = "Motion+Detection";
String device = "NodeMcu";
String message = "******"; //Add your message here. Like, the room id where you installed the sensor
String pubString = "token=" + accessToken + "&user=" + userKey + "&device=" + device + "&title=" + title + "&message=" + message;

#define pirPin D1 // Input for Motion Detector
int pirValue; // variable to store read PIR Value

int checkSensorDelay = 1 * 1000; // seconds to block the loop
int windowCounter = 0;
int waitWindow = 30 * checkSensorDelay;
int allowNewMessage = 0;


WiFiClientSecure client;

void setup() {
  Serial.begin(115200);
  delay(10);

  pinMode(pirPin, INPUT);

  // Connect to WiFi network
  Serial.print("Connecting to ");
  Serial.println(ssid);

  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("");
  Serial.print("WiFi connected with IP: ");
  Serial.println(WiFi.localIP());
}

void sendMessage() {
  Serial.println("Sending alert...");

  // try to connect
  if (!client.connect(remoteHost, httpsPort)) {
    Serial.println("Connection to remoteHost failed!");
    return;
  }

  // verify site
  if (client.verify(fingerprint, remoteHost)) {
    Serial.println("certificate matches");
  } else {
    Serial.println("certificate doesn't match");
  }

  client.print(String("POST ") + path + " HTTP/1.1\r\n" +
               "remoteHost: " + remoteHost + "\r\n" +
               "Content-length: " + String(pubString.length(), DEC) + "\r\n"
               "Content-Type: application/x-www-form-urlencoded\r\n" +
               "Connection: close\r\n\r\n" +
               pubString
              );

  while (client.connected()) {
    String line = client.readStringUntil('\n');
    if (line == "\r") {
      Serial.println("headers received");
      break;
    }
  }

  String line = client.readString();
  if (line.startsWith("{\"status\":1")) {
    Serial.println("Alert sent successfully!");
  } else {
    Serial.print("Alert failure: ");
    Serial.println(line);
  }
}

void loop() {
  pirValue = digitalRead(pirPin);

  if (pirValue == 1 && WiFi.status() == WL_CONNECTED && allowNewMessage == 1) {
    sendMessage();
    windowCounter = 0; // start a new window
    allowNewMessage = 0; // Since we already sent a message do not allow for a new one
  }

  if (windowCounter == waitWindow){ // we reached the max window size
    windowCounter = 0; // restart window
    allowNewMessage = 1; // allow new messages to be sent is the sensor finds something
  }

  delay(checkSensorDelay);  //Check sensor every checkSensorDelay
  windowCounter += checkSensorDelay;
}
```

[nodemcu-web]: http://nodemcu.com/
[pushover-web]: https://pushover.net/
[pir-web]: https://en.wikipedia.org/wiki/Passive_infrared_sensor
[arduino-ide-web]: https://www.arduino.cc/en/Main/Software
