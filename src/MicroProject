#include "DHT.h"
#include <WiFi.h>
#include <WebServer.h>

#define DHTPIN 4
#define DHTTYPE DHT11

const char* ssid = "Rachel";
const char* password = "racheldenny";

// Compost quality regression coefficients (from Khandakar dataset)
float beta0 = 103.0169095;
float beta1 = -0.221933838;
float beta2 = -0.906550104;

DHT dht(DHTPIN, DHTTYPE);
WebServer server(80);

// Crop lookup table: name, avg temp, avg humidity, compost advice
struct Crop {
  const char* name;
  float temp;
  float humidity;
  const char* compost;
};

Crop crops[] = {
  {"Rice",         23, 82, "High Nitrogen compost — well decomposed organic matter"},
  {"Maize",        22, 65, "Moderate Nitrogen — mix of green and brown compost"},
  {"Chickpea",     18, 16, "Low Nitrogen, High Phosphorus — aged bone meal compost"},
  {"Kidney Beans", 20, 21, "Moderate NPK — balanced compost"},
  {"Pigeon Peas",  27, 48, "Low Nitrogen — mature dry leaf compost"},
  {"Mung Bean",    28, 85, "Low Nitrogen — light compost application"},
  {"Black Gram",   29, 65, "Low Nitrogen — well-aerated compost"},
  {"Lentil",       19, 64, "Moderate Phosphorus — root-boosting compost"},
  {"Pomegranate",  22, 90, "High Potassium — banana peel and ash compost"},
  {"Banana",       27, 80, "High NPK — rich fully matured compost"},
  {"Mango",        31, 50, "Moderate Potassium — fruit waste compost"},
  {"Grapes",       24, 81, "High Potassium — wood ash compost"},
  {"Watermelon",   25, 85, "High Nitrogen — green waste compost"},
  {"Muskmelon",    28, 92, "High Nitrogen — kitchen waste compost"},
  {"Apple",        21, 92, "High Potassium — leaf mold compost"},
  {"Orange",       23, 92, "Moderate NPK — citrus-safe balanced compost"},
  {"Papaya",       33, 92, "High Nitrogen — fast decomposed green compost"},
  {"Coconut",      27, 94, "High Potassium — coir pith compost"},
  {"Cotton",       24, 79, "High Nitrogen — heavy organic compost"},
  {"Jute",         25, 80, "High Nitrogen — green manure compost"},
  {"Coffee",       25, 58, "Moderate NPK — slightly acidic compost"}
};

int numCrops = 21;

// Find closest matching crop based on live temp and humidity
int recommendCrop(float temp, float hum) {
  int bestIndex = 0;
  float bestScore = 999999;
  for (int i = 0; i < numCrops; i++) {
    float tempDiff = temp - crops[i].temp;
    float humDiff  = hum  - crops[i].humidity;
    float score = tempDiff * tempDiff + humDiff * humDiff;
    if (score < bestScore) {
      bestScore = score;
      bestIndex = i;
    }
  }
  return bestIndex;
}

// Compost quality from regression
float getQuality(float temp, float hum) {
  float q = beta0 + beta1 * temp + beta2 * hum;
  if (q < 0)   q = 0;
  if (q > 100) q = 100;
  return q;
}

String getCompostStatus(float q) {
  if (q < 30) return "Early Stage — Not Ready";
  if (q < 60) return "Active Composting";
  if (q < 80) return "Maturing — Almost Ready";
  return "Ready to Use!";
}

void handleRoot() {
  float h = dht.readHumidity();
  float t = dht.readTemperature();

  if (isnan(h) || isnan(t)) {
    server.send(200, "text/html", "<h2>Sensor error — check DHT11 wiring</h2>");
    return;
  }

  float q = getQuality(t, h);
  String compostStatus = getCompostStatus(q);
  int cropIndex = recommendCrop(t, h);
  String cropName = String(crops[cropIndex].name);
  String compostAdvice = String(crops[cropIndex].compost);

  String html = R"rawhtml(
<!DOCTYPE html>
<html>
<head>
  <meta charset='UTF-8'>
  <meta name='viewport' content='width=device-width,initial-scale=1'>
  <meta http-equiv='refresh' content='5'>
  <title>Smart Compost Monitor</title>
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; }
    body {
      font-family: Arial, sans-serif;
      background: #f0f4e8;
      padding: 20px;
      color: #333;
    }
    h1 {
      text-align: center;
      color: #2e7d32;
      font-size: 1.6em;
      margin-bottom: 6px;
    }
    .subtitle {
      text-align: center;
      color: #888;
      font-size: 0.85em;
      margin-bottom: 20px;
    }
    .grid {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 14px;
      max-width: 600px;
      margin: 0 auto 14px auto;
    }
    .card {
      background: white;
      border-radius: 14px;
      padding: 18px;
      box-shadow: 0 2px 8px rgba(0,0,0,0.08);
      text-align: center;
    }
    .card .icon { font-size: 1.6em; margin-bottom: 6px; }
    .card .val {
      font-size: 2em;
      font-weight: bold;
      color: #2e7d32;
    }
    .card .label {
      font-size: 0.8em;
      color: #999;
      margin-top: 4px;
    }
    .wide {
      max-width: 600px;
      margin: 0 auto 14px auto;
    }
    .crop-card {
      background: #2e7d32;
      color: white;
      border-radius: 14px;
      padding: 20px;
      text-align: center;
      box-shadow: 0 2px 8px rgba(0,0,0,0.15);
    }
    .crop-card .crop-label {
      font-size: 0.85em;
      opacity: 0.8;
      margin-bottom: 6px;
    }
    .crop-card .crop-name {
      font-size: 2em;
      font-weight: bold;
      margin-bottom: 4px;
    }
    .compost-card {
      background: white;
      border-radius: 14px;
      padding: 18px;
      box-shadow: 0 2px 8px rgba(0,0,0,0.08);
    }
    .compost-card .section-title {
      font-size: 0.8em;
      color: #999;
      margin-bottom: 10px;
      text-transform: uppercase;
      letter-spacing: 0.5px;
    }
    .compost-row {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 10px;
    }
    .compost-row .qlabel { font-size: 0.9em; color: #555; }
    .compost-row .qval {
      font-size: 1.3em;
      font-weight: bold;
      color: #2e7d32;
    }
    .status-pill {
      display: inline-block;
      background: #e8f5e9;
      color: #2e7d32;
      border-radius: 20px;
      padding: 4px 14px;
      font-size: 0.85em;
      font-weight: bold;
      margin-bottom: 12px;
    }
    .advice-box {
      background: #f9f9f9;
      border-left: 4px solid #2e7d32;
      border-radius: 6px;
      padding: 10px 14px;
      font-size: 0.88em;
      color: #444;
      text-align: left;
    }
    .footer {
      text-align: center;
      color: #bbb;
      font-size: 0.75em;
      margin-top: 10px;
    }
  </style>
</head>
<body>
  <h1>&#127807; Smart Compost Monitor</h1>
  <p class='subtitle'>Auto-refreshes every 5 seconds</p>

  <!-- Sensor readings -->
  <div class='grid'>
    <div class='card'>
      <div class='icon'>&#127777;</div>
      <div class='val'>)rawhtml";
  html += String(t, 1);
  html += R"rawhtml( &deg;C</div>
      <div class='label'>Temperature</div>
    </div>
    <div class='card'>
      <div class='icon'>&#128167;</div>
      <div class='val'>)rawhtml";
  html += String(h, 1);
  html += R"rawhtml(%</div>
      <div class='label'>Humidity</div>
    </div>
  </div>

  <!-- Crop recommendation -->
  <div class='wide'>
    <div class='crop-card'>
      <div class='crop-label'>&#127807; Recommended crop for current conditions</div>
      <div class='crop-name'>)rawhtml";
  html += cropName;
  html += R"rawhtml(</div>
    </div>
  </div>

  <!-- Compost quality + advice -->
  <div class='wide'>
    <div class='compost-card'>
      <div class='section-title'>Compost Analysis</div>
      <div class='compost-row'>
        <span class='qlabel'>Predicted compost quality</span>
        <span class='qval'>)rawhtml";
  html += String(q, 1);
  html += R"rawhtml(%</span>
      </div>
      <div style='text-align:center; margin-bottom:12px;'>
        <span class='status-pill'>)rawhtml";
  html += compostStatus;
  html += R"rawhtml(</span>
      </div>
      <div class='section-title'>Compost recommendation for )rawhtml";
  html += cropName;
  html += R"rawhtml(</div>
      <div class='advice-box'>)rawhtml";
  html += compostAdvice;
  html += R"rawhtml(</div>
    </div>
  </div>

  <p class='footer'>ESP32 + DHT11 &nbsp;|&nbsp; Linear Regression Model &nbsp;|&nbsp; Crop Recommendation Dataset</p>
</body>
</html>
)rawhtml";

  server.send(200, "text/html", html);
}

void setup() {
  Serial.begin(115200);
  dht.begin();
  WiFi.begin(ssid, password);
  Serial.print("Connecting to WiFi");
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\nConnected! IP address:");
  Serial.println(WiFi.localIP());
  server.on("/", handleRoot);
  server.begin();
}

void loop() {
  server.handleClient();
}
