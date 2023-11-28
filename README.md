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

## External connectivity:

### DHT11: 
![WhatsApp Image 2023-11-17 at 13 59 47_bb3dc5b4](https://github.com/Pavan-Gv/MiniProject/assets/94827772/21da3f6e-63fe-447c-9191-16428bc58a13)

### MQ-135:
![WhatsApp Image 2023-11-17 at 13 59 48_108ff893](https://github.com/Pavan-Gv/MiniProject/assets/94827772/af551e6b-0b87-422b-af2d-a97e6cba037e)

### ESP8266:
![WhatsApp Image 2023-11-17 at 13 59 49_8069652f](https://github.com/Pavan-Gv/MiniProject/assets/94827772/3564317a-2800-479f-ba1a-113084a5c461)

### Final Connectivity:
![WhatsApp Image 2023-11-17 at 13 59 48_54b5edcd](https://github.com/Pavan-Gv/MiniProject/assets/94827772/700773b7-fbed-49d4-b4b7-8546bc966037)

## Program
### Ardunio Code: 
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
### Machine Learning Algorithm:
```python
import pandas as pd
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestRegressor
from sklearn.metrics import mean_squared_error
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import Pipeline
import matplotlib.pyplot as plt
from twilio.rest import Client

# Read data from CSV file
file_path = 'Tox.csv'
data = pd.read_csv(file_path)

account_sid = 'ACf4cc7924e94e56e5e53e5e9e25d36de3'
auth_token = '912bcebdc0203a05df64b34861bfb281'
twilio_phone_number = '+12674154835'
recipient_phone_number = '+919346016760'

# Your Twilio client
client = Client(account_sid, auth_token)

# Threshold for high toxicity (adjust as needed)
high_toxicity_threshold = 0.100
# Split the data into training and testing sets
train_data, test_data, train_toxicity, test_toxicity = train_test_split(
    data[['PPM', 'Humidity', 'Temperature']],
    data['Toxicity'],  # Change 'Leak' to 'Toxicity'
    test_size=0.2,
    random_state=42
)

# Create a simple machine learning pipeline for regression
model = Pipeline([
    ('scaler', StandardScaler()),  # Standardize features
    ('regressor', RandomForestRegressor(random_state=42))  # Use a RandomForestRegressor for regression
])

# Train the model
model.fit(train_data, train_toxicity)

# Make predictions on the test set
predictions = model.predict(test_data)

# Evaluate the model
mse = mean_squared_error(test_toxicity, predictions)
print(f'Mean Squared Error: {mse:.2f}')

# Plot the flow of three values
plt.figure(figsize=(10, 6))
plt.plot(predictions, label='Predictions')
plt.plot(test_toxicity.values, label='Actual Toxicity')
plt.xlabel('Data Points')
plt.ylabel('Toxicity Level')
plt.title('Predicted vs Actual Toxicity')
plt.legend()
plt.show()
plt.figure(figsize=(10, 6))
plt.plot(data['PPM'], label='PPM')
plt.plot(data['Humidity'], label='Humidity')
plt.plot(data['Temperature'], label='Temperature')
plt.xlabel('Data Points')
plt.ylabel('Values')
plt.title('Values from CSV File')
plt.legend()
plt.show()
# Now, you can use this trained model to make predictions on new data
# Replace the following values with your actual sensor readings
new_data = pd.DataFrame({
    'PPM': [500],
    'Humidity': [50],
    'Temperature': [25]
})

predicted_toxicity = model.predict(new_data)[0]

# Classify toxicity level based on specified ranges
if predicted_toxicity < 100:
    toxicity_level = "Low toxicity"
elif 100 <= predicted_toxicity < 500:
    toxicity_level = "Moderate toxicity"
else:
    toxicity_level = "High toxicity"

print(f'Predicted Toxicity: {predicted_toxicity:.2f} - {toxicity_level}')

if predicted_toxicity >= high_toxicity_threshold:
    # Send SMS notification
    message_body = f'Gas toxicity level is high: {predicted_toxicity:.2f}'
    message = client.messages.create(
        body=message_body,
        from_=twilio_phone_number,
        to=recipient_phone_number
    )
    print(f'SMS Sent: {message.sid}')
else:
    print('Gas toxicity level is not high. No SMS sent.')

```
### Basic Trigger HTML Code:
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="refresh" content="0;https://maker.ifttt.com/trigger/gas_leakd/with/key/bm3rXYTDiawPYVrzmRvBmY">
    <title>Redirecting...</title>
</head>
<body>
    <p>If you are not redirected, <a href="URL_TO_REDIRECT">click here</a>.</p>
</body>
</html>
```
### DataSet Transmission Code:
```c++
void sen()
{
// HTTPClient https;
  if (!client.connect(host, httpsPort))
  {
    Serial.println("connection failed");
    return;
  }

  String url = "/macros/s/" + GAS_ID + "/exec?value1="+name1+"&value2="+age1;
  Serial.print("requesting URL: ");
  Serial.println(url);
  client.print(String("GET ") + url + " HTTP/1.1\r\n" +
         "Host: " + host + "\r\n" +
         "User-Agent: BuildFailureDetectorESP8266\r\n" +
         "Connection: close\r\n\r\n");

  Serial.println("request sent");
  while (client.connected()) {
    String line = client.readStringUntil('\n');
    if (line == "\r") {
      Serial.println("headers received");
      break;
    }
  }
  String line = client.readStringUntil('\n');
  if (line.startsWith("{\"state\":\"success\"")) 
  {
    Serial.println("esp8266/Arduino CI successfull!");
  } 
  else 
  {
    Serial.println("esp8266/Arduino CI has failed");
  }
  Serial.print("reply was : ");
  Serial.println(line);
  Serial.println("closing connection");
  Serial.println("==========");
  Serial.println();
```

## Output:
### Blynk Monitering:
![Monitering](https://github.com/Pavan-Gv/MiniProject/assets/94827772/1e5bf288-ef05-4311-b91f-4667c932f6c0)
### Arduino Serial Moniter:
![IDE](https://github.com/Pavan-Gv/MiniProject/assets/94827772/56ef2408-ff0b-4bb7-9bee-1de2477c9796)
### ML Algorithm Predictions:
![Algorithm](https://github.com/Pavan-Gv/MiniProject/assets/94827772/50d11f59-c2d0-4a94-b32e-ac30028bbc5f)

### Graphical Representation:
![Values Graph](https://github.com/Pavan-Gv/MiniProject/assets/94827772/f22b1efe-bdf8-4ebe-a1cb-d23d341bf577)
![A Vs P](https://github.com/Pavan-Gv/MiniProject/assets/94827772/d02cbbfd-6795-4906-825c-4e896530c19d)

### ALERTS PRODUCED:
![Phone Notificaton](https://github.com/Pavan-Gv/MiniProject/assets/94827772/512b5720-74a8-451e-b017-ff30084bfaa9)
![Mail Notification](https://github.com/Pavan-Gv/MiniProject/assets/94827772/e2b250f5-1200-4b93-a948-376e2383c362)



## Result
Therefore the Air Quality data from an Sensors is successfully collected and integrated.
The collected data is successfully sent to an IoT application via server.
The virtual pins are used to send the data individually.
