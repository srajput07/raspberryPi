 # Army Surveillance & Threat Detection System
 
A real-time IoT-based smart surveillance system for border/military zones using ESP32, ultrasonic sensors, PIR, microphone, and Raspberry Pi with Flask to classify and display threat levels on a web dashboard.

# Features
ESP32 monitors:
 3× Ultrasonic distances
 1× PIR sensor (motion)
 1× Microphone (analog + digital)
Classifies threats:
 🔴 Immediate (Explosion, heavy movement)
 ⚪ Medium (Group or vehicle)
 🟢 Safe (Low activity or no threat)
Sends JSON data to Raspberry Pi via HTTP
Flask server updates a live web dashboard
Real-time alerts with colored LED indicators


# Component	Description
ESP32 Dev Board	Main controller for sensors
HC-SR04 (3x)	Distance measurement (180° view)
PIR Sensor	Human/animal motion detection
Sound Sensor Module	Analog + Digital audio analysis
LEDs (R, G, W)	Threat level indicators
Raspberry Pi (any model)	Runs Flask server + dashboard

# Circuit Wiring
Pin on ESP32	Component
GPIO 4,5,14	TRIG pins
GPIO 18,19,21	ECHO pins
GPIO 22	PIR Sensor
GPIO 34,35	Mic Analog/Digital
GPIO 25,26,27	LED (G, W, R)

# Web Dashboard Preview
Live web UI shows all sensor values and highlights threat level.
Ultrasonic Sensor 1: 78 cm
Ultrasonic Sensor 2: 115 cm
Ultrasonic Sensor 3: 69 cm
PIR Motion Sensor: Detected
Microphone Analog: 612
Microphone Digital: High
Threat Level: Heavy Vehicle or Explosion

# How It Works
ESP32 reads sensors, classifies threat, updates LEDs.
Sends sensor data via HTTP POST to Raspberry Pi.
Flask server stores & serves data via /data.
HTML page polls /data every second and updates UI.

# Smart Threat Logic
if (micAnalog > 500) → Heavy Vehicle or Explosion
else if (micAnalog > 300) → Light Vehicle/Group
else if (micAnalog > 100) → Single Person
if (motion or dist < 100cm) → Immediate Threat!

# Future Improvements
Add camera support (ESP32-CAM / USB Pi Cam)
Email/SMS alerts for critical threats
Cloud integration or mobile app
Servo motor to rotate camera towards threat


![image](https://github.com/user-attachments/assets/3fbe4494-1c38-440b-a6d3-9a5872a2bb1a)
![image](https://github.com/user-attachments/assets/eebe382a-5183-4b6b-bb0e-03bbd02efdc7)
![image](https://github.com/user-attachments/assets/7b5e91c4-eaed-4e92-9981-84f78442fd1a)
![WhatsApp Image 2025-03-08 at 22 31 37_172fa9e4](https://github.com/user-attachments/assets/7b5488f0-2b96-4774-bdf1-034be7e74d49)




