# TOXIC GAS LEAK DETECTION AND ALERT DEVICE USING IOT AND ML:
    Harmful gas leakage is a serious safety hazard. Current gas detection systems are often expensive, complex, and require regular maintenance.This project aims to develop a low-cost, easy-to-deploy, and reliable system for detecting harmful gas leakage without connecting to other devices.The system will use an **ESP8266 module**, **MQ-135** gas sensor, **DHT11** Sensor,and ML algorithm to collect and analyze data from the gas sensor to detect harmful gases and send alerts to the people in the surroundings.

## Features:
Recieves Tempareture and Humidity data from DHT11 Senosr.
Recieves Gas Threshold Value From MQ135 Sensor.
The values are pushed via virtual pins to an iot application called BLynk.
Continous Monitoring of the Gases Threshold Level and Weather.
Future upgradable.

## Requirements
C++/C sharp for working with IDE.
Arduino IDE software
IFTTT Subscriptio free.
Nodemcu ESP8266
Dht11 sensor
Blynk IoT application

## Diagram
<img width="468" alt="image" src="https://github.com/KoduruSanathKumarReddy/WeatherForecast/assets/69503902/6032e99e-e7dd-446a-ac28-81f80ec425e4">

## Program
```c
#include <Arduino.h>
#include <ESP8266WiFi.h>
#include <ESP8266HTTPClient.h>
#include <WiFiClientSecureBearSSL.h>
#include <DHT.h>
#include <BlynkSimpleEsp8266.h>

#define BLYNK_TEMPLATE_ID "TMPL3NLw3Kr-5"
#define BLYNK_TEMPLATE_NAME "AIR Quality"
#define BLYNK_AUTH_TOKEN "Z29qqK8mbQH9sf2iGEojdfhLvmAg6AFN"

const char* ssid = "TRYME";
const char* password = "12345678";
const char* gasIFTTTKey = "bm3rXYTDiawPYVrzmRvBmY";

#define DHTPIN 2    // DHT11 data pin is connected to GPIO 2
#define DHTTYPE DHT11 // DHT type is DHT11

DHT dht(DHTPIN, DHTTYPE);

void setup() {
  Serial.begin(115200);

  // Connect to Wi-Fi
  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);
  Serial.print("Connecting to WiFi ..");
  while (WiFi.status() != WL_CONNECTED) {
    Serial.print('.');
    delay(1000);
  }

  // Initialize Blynk
  Blynk.begin(BLYNK_AUTH_TOKEN);
  dht.begin();
}

void loop() {
  Blynk.run();

  // Check gas value
  int gasValue = analogRead(A0); // Assuming MQ135 sensor is connected to analog pin A0

  if (gasValue > 200) {
    Serial.println("Gas value exceeds threshold. Sending HTTPS request to IFTTT...");

    // Trigger IFTTT event
    sendIFTTTEvent();
  }

  // Read DHT11 values
  float humidity = dht.readHumidity();
  float temperature = dht.readTemperature();

  // Send DHT11 data to Blynk
  Blynk.virtualWrite(V0, temperature);
  Blynk.virtualWrite(V1, humidity);

  // Wait for the next round
  delay(120000);
}

void sendIFTTTEvent() {
  std::unique_ptr<BearSSL::WiFiClientSecure> client(new BearSSL::WiFiClientSecure);
  client->setInsecure();

  HTTPClient https;

  Serial.print("[HTTPS] begin...\n");
  if (https.begin(*client, "https://maker.ifttt.com/trigger/gas_leakd/with/key/" + String(gasIFTTTKey))) {
    Serial.print("[HTTPS] GET...\n");
    int httpCode = https.GET();

    // httpCode will be negative on error
    if (httpCode > 0) {
      Serial.printf("[HTTPS] GET... code: %d\n", httpCode);
      if (httpCode == HTTP_CODE_OK || httpCode == HTTP_CODE_MOVED_PERMANENTLY) {
        String payload = https.getString();
        Serial.println(payload);
      }
    } else {
      Serial.printf("[HTTPS] GET... failed, error: %s\n", https.errorToString(httpCode).c_str());
    }

    https.end();
  } else {
    Serial.printf("[HTTPS] Unable to connect\n");
  }
}
```
## Output:
### Readings on IoT application Blynk:
![image](https://github.com/Pavan-Gv/MiniProject/assets/94827772/d1dc03ad-bfb0-45e3-bcbf-e098dd2361a5)

### External connectivity:
![WhatsApp Image 2023-11-17 at 13 59 47_bb3dc5b4](https://github.com/Pavan-Gv/MiniProject/assets/94827772/21da3f6e-63fe-447c-9191-16428bc58a13)
![WhatsApp Image 2023-11-17 at 13 59 48_108ff893](https://github.com/Pavan-Gv/MiniProject/assets/94827772/af551e6b-0b87-422b-af2d-a97e6cba037e)
![WhatsApp Image 2023-11-17 at 13 59 48_54b5edcd](https://github.com/Pavan-Gv/MiniProject/assets/94827772/700773b7-fbed-49d4-b4b7-8546bc966037)
![WhatsApp Image 2023-11-17 at 13 59 49_8069652f](https://github.com/Pavan-Gv/MiniProject/assets/94827772/3564317a-2800-479f-ba1a-113084a5c461)

## Result
Therefore the Air Quality data from an Sensors is successfully collected and integrated.
The collected data is successfully sent to an IoT application via server.
The virtual pins are used to send the data individually.
