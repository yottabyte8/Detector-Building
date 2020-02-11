# Detector-Building
Thermal Detector MITBlueprint 2020



#include <LiquidCrystal.h>
#include <Servo.h>
LiquidCrystal lcd(7, 8, 9, 10, 11, 12);

Servo myservo;

int redpin = 6;
int bluepin = 5;
int greenpin = 3;
int redpwr = 0;
int bluepwr = 0;
int greenpwr = 0;

int servoPin(4);

int ThermistorPin = 0;
int Vo;
float R1 = 10000;
float logR2, R2, T, Tc, Tf;
float c1 = 1.009249522e-03, c2 = 2.378405444e-04, c3 = 2.019202697e-07;

void setup() {
  Serial.begin(9600);
  pinMode(redpin, 0);
  pinMode(bluepin, 0);
  pinMode(greenpin, 0);
  lcd.begin(16, 2);
  myservo.attach(servoPin);
}

void loop() {
  delay(500);
  Vo = analogRead(ThermistorPin);
  R2 = R1 * (1023.0 / (float)Vo - 1.0);
  logR2 = log(R2);
  T = (1.0 / (c1 + c2*logR2 + c3*logR2*logR2*logR2));
  Tc = T - 273.15;
  Tf = (Tc * 9.0)/ 5.0 + 32.0;

  Serial.print("Temperature: ");
  Serial.print(Tf);
  Serial.print(" F; ");
  Serial.print(Tc);
  Serial.println(" C");  

  int tempscale = map(Tc, 20, 40, 0, 180);
  myservo.write(180-tempscale);
  int tempcolor = map(Tc, 20, 40, 0, 255);
  lcd.setCursor(0,0);
 
  if(Tc<25){
    setColor(tempcolor, 0, 255 - tempcolor);
    lcd.print("      SAFE      ");
  }
  else if(Tc<30){
    setColor(tempcolor, 0, 255 - tempcolor);
    lcd.print("    Too Hot!    ");
  }
  else{
    setColor(255, 0, 0);
    lcd.print("    Danger!    ");
  }
  redpwr = 0;
  bluepwr = 0;
  greenpwr = 0;
}

void setColor(int red, int green, int blue)
{
  #ifdef COMMON_ANODE
    red = 255 - red;
    green = 255 - green;
    blue = 255 - blue;
  #endif
  analogWrite(redpin, 0);
  analogWrite(greenpin, 0);
  analogWrite(bluepin, 0);
  analogWrite(redpin, red);
  analogWrite(greenpin, green);
  analogWrite(bluepin, blue);  
}
