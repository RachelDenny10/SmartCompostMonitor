A microprocessor-based smart composting assistant built on the ESP32 DevKit. 
The system reads live temperature and humidity data from a DHT11 sensor and 
runs a Multiple Linear Regression model — trained on 452 real composting records 
— directly on the microcontroller to predict compost maturity (0–100%). 

Simultaneously, it matches current environmental conditions against a 22-crop 
lookup table derived from the Kaggle Crop Recommendation Dataset (2200 records) 
to recommend the most suitable crop and its ideal compost NPK profile.

All outputs are served through a self-hosted HTML dashboard on the ESP32 itself — 
no cloud, no app, no internet required. Any device on the same WiFi network can 
access the live dashboard by entering the ESP32's IP address in a browser.

Built with: 
ESP32 · DHT11 · Arduino IDE · Adafruit DHT Library · Linear Regression · WiFi.h · WebServer.h

Datasets used:
1. Compost Maturity and Emission Monitoring Dataset
- Source: Khandakar et al., Qatar University
- URL: https://github.com/hafsa-kibria/Compost-Dataset
- License: CC BY 4.0
- Records used: 452
- Columns used: Temperature, MC (%), Maturity Score

2. Crop Recommendation Dataset
- Source: Atharva Ingle, Kaggle
- URL: https://www.kaggle.com/datasets/atharvaingle/crop-recommendation-dataset
- Records used: 2200
- Columns used: Temperature, Humidity, Label
  
