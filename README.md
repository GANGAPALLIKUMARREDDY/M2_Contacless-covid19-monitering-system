# M2_Contacless-covid19-monitering-system
#include <Wire.h>
#include <ESP8266WiFi.h>
#include <BlynkSimpleEsp8266.h>
#include "MAX30100_PulseOximeter.h"
//#include <SimpleTimer.h>
#include <ESP8266WiFi.h>
#include <BlynkSimpleEsp8266.h>
#include <LiquidCrystal_I2C.h>
LiquidCrystal_I2C lcd(0x27, 16, 2);

char auth[] = "5fP7c5vxPfCJpZAjI8p1rlyMtCoSQvXr";             // You should get Auth Token in the Blynk App.
char ssid[] = "Covid19";                                     // Your WiFi credentials.
char pass[] = "Jana@600";

#define REPORTING_PERIOD_MS     3000
BlynkTimer timer;

PulseOximeter pox;
uint32_t tsLastReport = 0;

void onBeatDetected()
{
  ;
}

void setup()
{
  Serial.begin(9600);
  Wire.begin(D2, D1);
  
  lcd.init();
  //lcd.begin();
  lcd.backlight();
  lcd.setCursor(3, 0);
  lcd.print("Welcome To");
  lcd.setCursor(0, 1);
  lcd.print("JustDoElectronic");
  delay(3000);
  lcd.clear();
  lcd.setCursor(6, 0);
  lcd.print("IoT");
  lcd.setCursor(1, 1);
  lcd.print("Pulse Oximeter");
  delay(3000);
  lcd.clear();
  Serial.begin(115200);
  Blynk.begin(auth, ssid, pass);
  Serial.print("Initializing pulse oximeter..");


  // Initialize the PulseOximeter instance
  // Failures are generally due to an improper I2C wiring, missing power supply
  // or wrong target chip
  if (!pox.begin()) {
    Serial.println("FAILED");
    for (;;);
  } else {
    Serial.println("SUCCESS");
    digitalWrite(1, HIGH);
  }
  pox.setIRLedCurrent(MAX30100_LED_CURR_24MA);

  // Register a callback for the beat detection
  pox.setOnBeatDetectedCallback(onBeatDetected);

  timer.setInterval(1000L, getSendData);
}

void loop()
{

  timer.run(); // Initiates SimpleTimer
  Blynk.run();
  // Make sure to call update as fast as possible
  pox.update();
  if (millis() - tsLastReport > REPORTING_PERIOD_MS) {


    // to computer serial monitor
    Serial.print("BPM: ");
    Serial.print(pox.getHeartRate());
    //blue.println("\n");

    Serial.print("    SpO2: ");
    Serial.print(pox.getSpO2());
    Serial.print("%");
    Serial.println("\n");

    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("BPM: ");
    lcd.setCursor(10, 0);
    lcd.print(pox.getHeartRate());
    lcd.setCursor(0, 1);
    lcd.print("SpO2:");
    lcd.setCursor(10, 1);
    lcd.print(pox.getSpO2());
    lcd.setCursor(14, 1);
    lcd.print("%");

    Blynk.virtualWrite(V2, pox.getHeartRate() );
    Blynk.virtualWrite(V3, pox.getSpO2());
if(pox.getSpO2()>120&&pox.getHeartRate()>130)
{
  Blynk.notify("Person is Affected by COVID-19");
}
else
{
  Blynk.notify("Person is  not Affected by COVID-19");
}
    tsLastReport = millis();

  }

}

void getSendData()
{
  ;
}#include <Wire.h>
//#include "MAX30100_PulseOximeter.h"
#include <Adafruit_MLX90614.h>
#define BLYNK_PRINT Serial
#include <Blynk.h>
#include <ESP8266WiFi.h>
#include <BlynkSimpleEsp8266.h>
#include "Wire.h"
#include<LiquidCrystal_I2C.h>
LiquidCrystal_I2C lcd(0x27, 16, 2);
//#include "Adafruit_GFX.h"
#define I2C_SDA D2
#define I2C_SCL D1
#define REPORTING_PERIOD_MS 1000
//WidgetLCD lcd(V3);
//uint8_t bm280_address = 0x76;
//uint8_t max30100_address = 0x57;
uint8_t irmlx90614_address = 0x5A;
uint32_t tsLastReport = 0;
char auth[] = "5fP7c5vxPfCJpZAjI8p1rlyMtCoSQvXr";             // You should get Auth Token in the Blynk App.
char ssid[] = "Covid19";                                     // Your WiFi credentials.
char pass[] = "Kumar@45";
Adafruit_MLX90614 mlx = Adafruit_MLX90614();
//PulseOximeter pox;
//float BPM, SpO2;
float A, B ,D;
BlynkTimer timer;

void senddata() {
  Blynk.virtualWrite(V0, millis() / 1000);
    Blynk.virtualWrite(V1,A);
       Blynk.virtualWrite(V4, D);
        // Blynk.virtualWrite(V4,BPM );
      // Blynk.virtualWrite(V5,SpO2 );
}
void setup() {
Serial.begin(9600);
 Blynk.begin(auth, ssid, pass);
Wire.begin();
mlx.begin();
//pox.begin();
lcd.init();
 lcd.backlight();
//delay(2000);
Serial.println();
 timer.setInterval(1000L, senddata);
 pinMode(A0,INPUT);
 pinMode(D6,INPUT);
 pinMode(D5,OUTPUT);
lcd.clear();
lcd.setCursor(4,0);
lcd.print("COVID-19:");
lcd.setCursor(4,1);
lcd.print(" MONITOR ");
  delay(2000);
 
}

void loop() {
  int val=0;
   int val1=0;
   val=analogRead(A0);
   val1=digitalRead(D6);
  Serial.println(val);
  if((val<792)||(val>=797))
  {
  lcd.clear();
lcd.setCursor(0,0);
 lcd.print("MIKE OUTPUT:");
lcd.setCursor(0,1);
 lcd.print("Cough detect");
  Blynk.virtualWrite(V5,val);
  delay(2000);
  }
  else if((val>792)||(val<=796))
  {
     lcd.clear();
lcd.setCursor(0,0);
 lcd.print("MIKE OUTPUT:");
lcd.setCursor(0,1);
 lcd.print("Cough not detect");
 Blynk.virtualWrite(V5,val);
  delay(2000);
  }
  if(D>100&&val>999)
  {
    Blynk.notify(" Person Affected by COVID-19");
  }
  else
  {
   Blynk.notify(" Person not Affected by COVID-19") ;
  }
 if(val1==HIGH)
 {
 digitalWrite(D5,HIGH);
   lcd.clear();
lcd.setCursor(0,0);
 lcd.print("SANITIZER");
lcd.setCursor(7,1);
 lcd.print("ON");
 delay(2000);
 }
 else
 {
  digitalWrite(D5,LOW);
   lcd.clear();
lcd.setCursor(0,0);
 lcd.print("SANITIZER:");
lcd.setCursor(7,1);
 lcd.print("OFF");
  delay(2000);
 }
 delay(1000);
  printTemp();
   Blynk.run();
   timer.run();
}

void printTemp(){
  A=mlx.readAmbientTempC();
  B=mlx.readObjectTempC();
  D=(A*9/5)+42;
Serial.print("Ambient = "); Serial.print(A);
Serial.print("*C\tObject = "); Serial.print(B); Serial.println("*C");
lcd.clear();
lcd.setCursor(0,0);
 lcd.print("Amb Temp:");
lcd.setCursor(9,0);
 lcd.print(A);
  lcd.setCursor(12,0);
 lcd.print("*C");
  lcd.setCursor(0,1);
 lcd.print("Obj Temp:");
 lcd.setCursor(9,1);
 lcd.print(D);
 lcd.setCursor(12,1);
 lcd.print("*F");
 delay(3000);
}
