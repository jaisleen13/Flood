# Flood
Smart Flood Prediction System using iot sensors
The IoT-Based Flood Detection and Prevention System is a smart monitoring solution designed to detect rising water levels and provide early flood warnings. The system uses sensors, a NodeMCU microcontroller, cloud platforms, and mobile applications to monitor water levels in real time. When water reaches a dangerous level, alerts are sent to users and preventive actions such as activating a water pump or alarm system are automatically triggered.

The water level sensor continuously measures the water level.
Sensor data is processed by the NodeMCU.
Data is uploaded to the cloud platform through Wi-Fi.
Users can monitor water levels remotely using the Blynk application.
When the water level exceeds the threshold:
Buzzer alarm is activated.
Notification is sent to the user's smartphone.
Water pump is turned ON automatically through a relay.
The system continues monitoring until the water level becomes safe.


code 
#define BLYNK_TEMPLATE_ID "TMPL3zu8Gik6j"
#define BLYNK_TEMPLATE_NAME "Flood Detection and Prevention"
#define BLYNK_AUTH_TOKEN "136JnvFKwCwwNo7a3Ls73uHKDnwvTLRh"
#include <WiFi.h>
#include <BlynkSimpleEsp32.h>
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>
#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, -1);
char ssid[] = "OPPO Reno8 5G";
char pass[] = "u5z72tc4";
#define TRIG 5
#define ECHO 18
#define RED_LED 19
#define YELLOW_LED 27
#define GREEN_LED 14
#define BUZZER 23
#define MOTOR 25
#define MOTOR2 26   
BlynkTimer timer;
long duration;
float distance;
float fullDistance = 3.0;
float emptyDistance = 12.0;
bool motorState = 0;
bool motor2State = 0;
BLYNK_WRITE(V1)
{
  motorState = param.asInt();

  digitalWrite(MOTOR, motorState ? LOW : HIGH);
}

BLYNK_WRITE(V5)
{
  motor2State = param.asInt();

  digitalWrite(MOTOR2, motor2State ? LOW : HIGH);
}

void sendWaterLevel()
{
  digitalWrite(TRIG, LOW);
  delayMicroseconds(2);

  digitalWrite(TRIG, HIGH);
  delayMicroseconds(10);
  digitalWrite(TRIG, LOW);

  duration = pulseIn(ECHO, HIGH, 30000); // timeout add
  distance = duration * 0.034 / 2;

  float waterLevel = ((emptyDistance - distance) / (emptyDistance - fullDistance)) * 100;

  waterLevel = constrain(waterLevel, 0, 100);

  Blynk.virtualWrite(V0, waterLevel);

  display.clearDisplay();

  if (waterLevel >= 90)
  {
    digitalWrite(GREEN_LED, LOW);
    digitalWrite(YELLOW_LED, LOW);
    digitalWrite(RED_LED, HIGH);

    Blynk.virtualWrite(V2, 0);
    Blynk.virtualWrite(V3, 0);
    Blynk.virtualWrite(V4, 1);

    digitalWrite(BUZZER, HIGH);

    digitalWrite(MOTOR, HIGH);
    motorState = 0;
    Blynk.virtualWrite(V1, 0);

    display.setTextSize(3);
    display.setCursor(10, 20);
    display.println("DANGER");
  }
  else
  {
    digitalWrite(BUZZER, LOW);

    if (waterLevel <= 20)
    {
      digitalWrite(GREEN_LED, HIGH);
      digitalWrite(YELLOW_LED, LOW);
      digitalWrite(RED_LED, LOW);

      Blynk.virtualWrite(V2, 1);
      Blynk.virtualWrite(V3, 0);
      Blynk.virtualWrite(V4, 0);
    }
    else
    {
      digitalWrite(GREEN_LED, LOW);
      digitalWrite(YELLOW_LED, HIGH);
      digitalWrite(RED_LED, LOW);

      Blynk.virtualWrite(V2, 0);
      Blynk.virtualWrite(V3, 1);
      Blynk.virtualWrite(V4, 0);
    }

    display.setTextSize(2);
    display.setCursor(0, 0);
    display.print("WaterLevel");

    display.setCursor(5, 22);
    display.print(waterLevel);
    display.print("%");

    display.setCursor(0, 45);
    display.print(motorState ? "Motor: ON" : "Motor: OFF");
  }

  display.display();
}

void setup()
{
  Serial.begin(115200);

  pinMode(TRIG, OUTPUT);
  pinMode(ECHO, INPUT);

  pinMode(RED_LED, OUTPUT);
  pinMode(YELLOW_LED, OUTPUT);
  pinMode(GREEN_LED, OUTPUT);
  pinMode(BUZZER, OUTPUT);
  pinMode(MOTOR, OUTPUT);
  pinMode(MOTOR2, OUTPUT);

  digitalWrite(MOTOR, HIGH);
  digitalWrite(MOTOR2, HIGH);

  // ESP32 I2C pins
  Wire.begin(21, 22);
  

  if (!display.begin(SSD1306_SWITCHCAPVCC, 0x3C))
  {
    Serial.println("OLED failed");
    while (1);
  }

  display.clearDisplay();
  display.setTextColor(SSD1306_WHITE);
  display.display();

  Blynk.begin(BLYNK_AUTH_TOKEN, ssid, pass);

  timer.setInterval(1000L, sendWaterLevel);
}

void loop()
{
  Blynk.run();
  timer.run();
}
