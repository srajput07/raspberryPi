**ESP-32 Code:- **
#include <Arduino.h>
#include <WiFi.h>
#include <HTTPClient.h>
const char* ssid = "Redmi Note 10S";
const char* password = "Red@4321";
const char* serverIP = "http://192.168.6.63:5000/sensor-data";
#define TRIG_1 4
#define ECHO_1 18
#define TRIG_2 5
#define ECHO_2 19
#define TRIG_3 14
#define ECHO_3 21
#define PIR_SENSOR 22
#define MIC_ANALOG 34
#define MIC_DIGITAL 35
#define GREEN_LED 25
#define WHITE_LED 26
#define RED_LED 27
void setup() {
Serial.begin(115200);
WiFi.begin(ssid, password);
Serial.print("Connecting to WiFi");
while (WiFi.status() != WL_CONNECTED) {
Serial.print(".");
delay(500);
}
Serial.println("\nConnected to WiFi");
pinMode(TRIG_1, OUTPUT); pinMode(ECHO_1, INPUT);
pinMode(TRIG_2, OUTPUT); pinMode(ECHO_2, INPUT);
pinMode(TRIG_3, OUTPUT); pinMode(ECHO_3, INPUT);
pinMode(PIR_SENSOR, INPUT);
pinMode(MIC_DIGITAL, INPUT);
pinMode(GREEN_LED, OUTPUT);
pinMode(WHITE_LED, OUTPUT);
pinMode(RED_LED, OUTPUT);
digitalWrite(GREEN_LED, HIGH);
digitalWrite(WHITE_LED, LOW);
digitalWrite(RED_LED, LOW);}
long measureDistance(int trigPin, int echoPin) {
digitalWrite(trigPin, LOW);
delayMicroseconds(2);
digitalWrite(trigPin, HIGH);
delayMicroseconds(10);
digitalWrite(trigPin, LOW);
long duration = pulseIn(echoPin, HIGH, 30000); // 30ms timeout
if (duration == 0) return 400; // No response, assume max distance
return duration * 0.0343 / 2; // Convert to cm}
void updateLEDStatus(bool motion, int micAnalog, long dist1, long dist2, long dist3) {
String threatLevel = "No Threat";
bool highThreat = false, mediumThreat = false;
if (micAnalog > 500) {
threatLevel = "Heavy Vehicle or Explosion";
highThreat = true;
} else if (micAnalog > 300) {
threatLevel = "Light Vehicle or Group Movement";
mediumThreat = true;}
else if (micAnalog > 100) {
threatLevel = "Single Person Movement"
if (motion || dist1 < 100 || dist2 < 100 || dist3 < 100) {
highThreat = true;
threatLevel = "Immediate Threat!";}
else if (dist1 < 200 || dist2 < 200 || dist3 < 200) {
mediumThreat = true;
threatLevel = "Potential Threat!";}
if (highThreat) {
digitalWrite(RED_LED, HIGH);
digitalWrite(GREEN_LED, LOW);
digitalWrite(WHITE_LED, LOW);
} else if (mediumThreat) {
digitalWrite(WHITE_LED, HIGH);
digitalWrite(GREEN_LED, LOW);
digitalWrite(RED_LED, LOW);
} else {
digitalWrite(GREEN_LED, HIGH);
digitalWrite(WHITE_LED, LOW);
digitalWrite(RED_LED, LOW); }
Serial.println("Threat Level: " + threatLevel);}
void sendToRaspberryPi(long dist1, long dist2, long dist3, bool motion, int micAnalog, bool micDigital) {
if (WiFi.status() != WL_CONNECTED) {
Serial.println("WiFi Disconnected! Attempting reconnect...");
WiFi.disconnect();
WiFi.reconnect();
delay(500);
return;}
HTTPClient http;
http.begin(serverIP);
http.addHeader("Content-Type", "application/json");
// Create JSON payload
String jsonPayload = "{";
jsonPayload += "\"distance1\":" + String(dist1) + ",";
jsonPayload += "\"distance2\":" + String(dist2) + ",";
jsonPayload += "\"distance3\":" + String(dist3) + ",";
jsonPayload += "\"motion\":" + String(motion) + ",";
jsonPayload += "\"micAnalog\":" + String(micAnalog) + ",";
jsonPayload += "\"micDigital\":" + String(micDigital);
jsonPayload += "}";
Serial.println("Sending JSON: " + jsonPayload);
// Send HTTP POST request
int httpResponseCode = http.POST(jsonPayload);
if (httpResponseCode > 0) {
String response = http.getString();
Serial.print("Server Response: ");
Serial.println(response);
} else {
Serial.print("Error sending data: ");
Serial.println(httpResponseCode) }
http.end();}
void loop() {
long dist1 = measureDistance(TRIG_1, ECHO_1);
long dist2 = measureDistance(TRIG_2, ECHO_2);
long dist3 = measureDistance(TRIG_3, ECHO_3);
bool motionDetected = digitalRead(PIR_SENSOR);
int micAnalog = analogRead(MIC_ANALOG);
bool micDigital = digitalRead(MIC_DIGITAL);
// Update LED based on threat level
updateLEDStatus(motionDetected, micAnalog, dist1, dist2, dist3);
// Print sensor data to Serial Monitor
Serial.println("===== SENSOR DATA =====");
Serial.printf("Ultrasonic1: %ld cm\n", dist1);
Serial.printf("Ultrasonic2: %ld cm\n", dist2);
Serial.printf("Ultrasonic3: %ld cm\n", dist3);
Serial.printf("PIR Motion: %d\n", motionDetected);
Serial.printf("Mic Analog: %d\n", micAnalog);
Serial.printf("Mic Digital: %d\n", micDigital);
Serial.println("=======================");
// Send data to Raspberry Pi
sendToRaspberryPi(dist1, dist2, dist3, motionDetected, micAnalog, micDigital);
delay(1000); // Wait 1 second}
**HTML Page Code:-** 
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Army surviellance System</title>
<style>
body {
font-family: Arial, sans-serif;
background-color: #1a1a1a;
color: #00ff00;
text-align: center;
padding: 20px;
}
h1 {
border-bottom: 2px solid #00ff00;
display: inline-block;
padding-bottom: 5px;
}
.container {
display: inline-block;
text-align: left;
padding: 20px;
border: 2px solid #00ff00;
border-radius: 10px;
background-color: #222;
}
  .alert {
font-weight: bold;
color: red;
}
</style>
<script>
function fetchData() {
fetch('/data')
.then(response => response.json())
.then(data => {
document.getElementById("dist1").innerText = data.distance1 + " cm";
  document.getElementById("dist2").innerText = data.distance2 + " cm";
document.getElementById("dist3").innerText = data.distance3 + " cm";
document.getElementById("motion").innerText = data.motion ? "Detected" : "No Motion";
document.getElementById("micAnalog").innerText = data.micAnalog;
document.getElementById("micDigital").innerText = data.micDigital ? "High" : "Low";
classifyThreat(data);
});
}
function classifyThreat(data) {
let threatLevel = "No Threat";
if (data.micAnalog > 500) {
threatLevel = "Heavy Vehicle or Explosion";
} else if (data.micAnalog > 300) {
threatLevel = "Light Vehicle or Group Movement";
} else if (data.micAnalog > 100) {
threatLevel = "Single Person Movement";
}
document.getElementById("threat").innerText = threatLevel;
}
setInterval(fetchData, 1000);
</script>
</head>
<body onload="fetchData()">
<h1>Army surviellance System </h1>
<div class="container">
<p><strong>Ultrasonic Sensor 1:</strong> <span id="dist1"></span></p>
  <p><strong>Ultrasonic Sensor 2:</strong> <span id="dist2"></span></p>
<p><strong>Ultrasonic Sensor 3:</strong> <span id="dist3"></span></p>
<p><strong>PIR Motion Sensor:</strong> <span id="motion"></span></p>
<p><strong>Microphone Sensitivity (Analog):</strong> <span id="micAnalog"></span></p>
<p><strong>Microphone Digital Signal:</strong> <span id="micDigital"></span></p>
<p class="alert"><strong>Threat Level:</strong> <span id="threat">No Threat</span></p>
</div>
</body>
</html>
**Raspberry pi Code:-**
from flask import Flask, request, jsonify, render_template
app = Flask(__name__)
# Store sensor data
sensor_data = { "distance1": 0, "distance2": 0, "distance3": 0, "motion": 0, "micAnalog": 0, "micDigital": 0
}
@app.route("/")
def index():
return render_template("index.html", data=sensor_data)
@app.route("/sensor-data", methods=["POST"])
def receive_data():
global sensor_data
sensor_data = request.get_json()
print("Received Data:", sensor_data)
return jsonify({"message": "Data received"}), 200
@app.route("/data")
def get_data():
return jsonify(sensor_data)
if __name__ == "__main__":
app.run(host="0.0.0.0", port=5000, debug=True)

# raspberryPi
![image](https://github.com/user-attachments/assets/3fbe4494-1c38-440b-a6d3-9a5872a2bb1a)
![image](https://github.com/user-attachments/assets/eebe382a-5183-4b6b-bb0e-03bbd02efdc7)
![image](https://github.com/user-attachments/assets/7b5e91c4-eaed-4e92-9981-84f78442fd1a)
![WhatsApp Image 2025-03-08 at 22 31 37_172fa9e4](https://github.com/user-attachments/assets/7b5488f0-2b96-4774-bdf1-034be7e74d49)



